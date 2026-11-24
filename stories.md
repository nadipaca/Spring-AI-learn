story1:
Situation
"At Macy's we had more than 500 stores across the US. Floor plan changes were handled manually, through emails, spreadsheets, and follow-up calls, and there was no reliable record of who changed what or who approved it. The company moved this to a digital workflow. A store user edits the floor map in the app, which creates a change request. The request goes through two levels of approval, and once it's fully approved, a downstream service publishes a new version of the floor map. At either level, the approver can approve, reject, or send it back asking for more information."

Task
"I owned two pieces. The first was the event history table, which records every step in a floor plan request's lifecycle for audit and for the timeline screen. The second was the approval email communication in both directions: sending notifications to approvers and requestors, and letting approvers act by replying to the email."

Action: design
"I made the history table append-only. Each row stores the request ID, store ID, floor plan version, event type, from and to status, who acted, their role, comments, the channel (portal or email), and a timestamp. When a request got its final approval, we wrote the status change, the history row, and an outbox event in the same transaction. A relay then published the outbox event to the downstream floor map service, so a new version was never published for an approval that hadn't been saved.

For the email, the outgoing notifications went over SMTP. Each email had a unique reply-to address containing the request ID and a one-time token. When an approver replied with APPROVE, REJECT, or MORE INFO plus comments, our mail provider's inbound webhook posted the reply to our API. We verified the webhook signature, checked that the sender was the assigned approver, and ignored duplicate deliveries by tracking the Message-ID. Each reply became a history event and moved the request to its next step. For example, a MORE INFO reply emailed the requestor with the approver's comments, and when the requestor resubmitted, the approver got a new email. Approvers could also use the portal link instead of replying."

Action: the problem
"Everything passed in Dev and QA. But in UAT, when the business users tested with several people at once, they started getting HTTP 500 errors when submitting requests. I took thread dumps and found most of the web server's request threads were stuck sending email. I was sending the SMTP email synchronously, inside the database transaction. In Dev and QA the mail server was a stub that responded instantly. UAT used the real corporate SMTP relay, which was slower, and we hadn't set any timeouts. So each request held a web thread and a database connection while it waited, and under load both pools ran out."

Action: the fix
"I took email off the request path completely:

The submit API saved the request, the history event, and a notification record in one transaction, then returned right away.
Emails were sent asynchronously after the transaction committed, using @TransactionalEventListener(AFTER_COMMIT) with @Async on a dedicated, size-limited thread pool. No email went out for data that had rolled back, and no web thread waited on SMTP.
I set SMTP connect and read timeouts and added retries with backoff. If an email still failed, it was marked FAILED and a scheduled job retried it later.
I applied the same rule to the inbound webhook: save the reply, return 200 immediately, and process it in the background. Otherwise a slow response makes the mail provider retry, which creates duplicates."
Result
"The 500 errors went away, submit response time dropped from [X] to [Y], and the business users signed off on UAT. Approvers could act straight from their inbox, and every decision, whether made by email or in the portal, was in the history table. My takeaway was: never block a request thread on a slow external system or on a person. After this, we added realistic SMTP latency to our test environments and made that rule part of code reviews."

story2:
STAR story 2: PSData session timeout, OOM, and FileNet (about 2.5 minutes)
Replace the [brackets] with your real details, and drop anything you didn't actually do.

Situation
"At PSData, a steel company, clients ordered steel through a long order form. The form was in an external application, and our backend handled the submission into ServiceNow. Clients often took 30 to 60 minutes to finish because they stopped to check specs, pricing, or other work. When they clicked Submit, [our backend session] had already expired. They got a timeout error and lost everything they'd entered, including uploaded documents. Many users complained, and the business asked us to increase the session timeout."

Task
"I owned this change. Before starting, I spoke with a few users and realised the real requirement wasn't just a longer session. It was 'never lose my work.' So my goal was to let users take as long as they needed, without hurting the application's stability or security."

Action

1. Found the real problem
"I increased the timeout to 60 minutes. During a JMeter load test, heap usage kept climbing until we hit an OutOfMemoryError. I captured a heap dump and analysed it in Eclipse MAT. The dominator tree showed a legacy custom session HashMap holding most of the heap. It was originally written for a single server and had no eviction: sessions were removed only on logout, which most users never did. Each session also held the full form and the uploaded files as byte arrays. My rough calculation was [N] sessions × 60 minutes × [~2 MB] ≈ [X GB] on a [Y GB] heap, so the longer timeout guaranteed a crash."

2. Improved the HashMap usage
"I replaced the plain HashMap with a thread-safe store that has a size limit and an expiry, using [a Caffeine cache with expireAfterAccess and maximumSize / a ConcurrentHashMap with a scheduled cleanup job]. Idle sessions were removed automatically, and sessions were cleared on submit and logout. The plain HashMap also wasn't safe for concurrent requests, so this fixed that too."

3. Moved form data out of the session with incremental upserts
"Instead of holding the whole form in memory and saving it once at the end, I changed it to save section by section. In Angular, I listened to the form's valueChanges stream with debounceTime and concatMap, so saves were batched and stayed in order. On the backend, each save was an upsert keyed by draft ID and section, using a [DB2 MERGE statement / JPA with @Version], so retries were safe and two open tabs couldn't overwrite each other. Uploaded files were streamed to FileNet, and only the document IDs were kept in the draft. The session now held only the user's identity and the draft ID."

4. Improved session timeout behaviour
"Instead of one long fixed timeout, I used a sliding idle timeout so active users stayed logged in, an absolute maximum for security, and a warning popup before expiry. If a session did expire, the user logged back in and resumed from their saved draft."

5. Tested and rolled out safely
"I wrote a unit test with a fake clock to confirm sessions were evicted. I made the timeout configurable so it could be changed without a code release, and I reran the same load test before go-live. After the release, I monitored heap and GC metrics."

Result
"In the same load test, heap usage stayed flat at about [Z GB] instead of climbing, and the OOM errors stopped. Timeout complaints dropped from [N per week] to [near zero]. Users stopped losing work: even if a session expired after an hour, their draft was already saved. Submit also became faster and more reliable, because it only validated the saved draft and sent it to ServiceNow once.
My main lesson was that a timeout change is really a capacity change. Anything kept in memory needs a size limit and an expiry, and user data belongs in the database, not in the session."



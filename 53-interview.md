# Interview Guide: 3-Year Full-Stack Java/Angular Developer

This guide is based on the stack used in these repos:

- **Backend:** Spring Boot 3.5 on Java 17/21
- **Workflows:** Activiti 6 BPMN
- **Frontend:** Angular 15 moving to 20, with PrimeNG
- **Data:** DB2 with JPA/Hibernate Envers
- **Platform:** Redis sessions, OAuth2, Vault, Spring Cloud Config and a Spring Cloud Gateway proxy
- **Batch:** Spring Batch with Quartz
- **Testing:** JUnit 5/Mockito and Jasmine

Each question has a short note on what a good answer sounds like.

---

## 1. Core Java (~10 min)

1. **`HashMap` vs `ConcurrentHashMap`. What happens if a key's `hashCode` changes after you insert it?**
   *Good answer:* buckets and hashing, the `equals`/`hashCode` contract, thread safety, and the entry becoming "lost" in the map.
2. **Checked vs unchecked exceptions. When would you write a custom exception?**
   *Good answer:* names a real case, such as a business-rule failure turned into an HTTP 4xx by a handler.
3. **Use Streams to group a list of tasks by assignee and count them.**
   *Good answer:* `Collectors.groupingBy(Task::getAssignee, Collectors.counting())`.
4. **What's new in Java 17/21 that you've used?**
   *Good answer:* records, `switch` expressions, text blocks, `var`, virtual threads (Java 21).
5. **How does `Optional` help, and how is it misused?**
   *Good answer:* not as a field or parameter, and not calling `.get()` without a check.

## 2. Spring Boot and REST (~10 min)

1. **What happens at startup with `@SpringBootApplication`?**
   *Good answer:* component scan, auto-configuration, conditional beans.
2. **`@Component`, `@Service`, `@Repository`, `@Controller`: is there any functional difference?**
   *Good answer:* they're mostly semantic; `@Repository` adds exception translation.
3. **Constructor vs field injection. Which do you prefer and why?**
   *Good answer:* constructor injection for immutability and easier testing.
4. **How do you handle errors globally in a REST API?**
   *Good answer:* `@RestControllerAdvice` and `@ExceptionHandler`, a consistent error body, correct status codes.
5. **How do you validate a request body?**
   *Good answer:* `@Valid`, Bean Validation annotations, handling `MethodArgumentNotValidException`.
6. **`bootstrap.yml` vs `application.yml`. How does config reach the app from a config server or Vault?**
   *Good answer:* knows properties can be externalized and why secrets don't belong in the repo.
7. **What does a Spring Boot 2 → 3 migration involve?**
   *Good answer:* `javax` to `jakarta`, Java 17 baseline, Security 6 (`SecurityFilterChain`, no `WebSecurityConfigurerAdapter`). This codebase has just been through this migration, so it's a good real-world probe.

## 3. JPA, Hibernate and SQL on DB2 (~8 min)

1. **What is the N+1 problem and how do you fix it?**
   *Good answer:* `JOIN FETCH`, `@EntityGraph`, batch size.
2. **Lazy vs eager loading. What causes `LazyInitializationException`?**
3. **How does `@Transactional` work? Why doesn't it apply on a self-invoked method?**
   *Good answer:* proxies, propagation, rollback only on unchecked exceptions by default.
4. **What does Hibernate Envers do?**
   *Good answer:* auditing and history tables. A bonus if they've used it.
5. **SQL exercise:** find the latest status row per request, or find duplicate records.
   *Good answer:* a window function (`ROW_NUMBER() OVER (PARTITION BY ...)`) or a subquery, and `GROUP BY ... HAVING COUNT(*) > 1`.
6. **How do you troubleshoot a slow query?**
   *Good answer:* explain plan, indexes, pagination.

## 4. Workflow / BPMN with Activiti (~8 min)

This is specific to the team. Don't expect deep knowledge at 3 years, but check that they can reason about it.

1. **What are a process definition, a process instance and a task?**
2. **User task vs service task. How is a task assigned?**
   *Good answer:* `assignee` vs `candidateGroups`, such as the AD groups used in the Pershing approval task.
3. **How does an exclusive gateway decide a path?**
   *Good answer:* condition expressions on process variables, like `${submitStatus == "complete"}`.
4. **Execution listener vs task listener. What would you use a `delegateExpression` for?**
5. **Scenario:** a process instance is stuck and no one can see the task in their inbox. How do you investigate?
   *Good answer:* check `ACT_RU_TASK`/`ACT_RU_EXECUTION`, the candidate group lookup values, process variables, and the logs.
6. **If they don't know Activiti:** ask how they'd design a two-step "request → approve/reject" flow with status transitions and an audit trail.

## 5. Security (~5 min)

1. **Authentication vs authorization.**
2. **OAuth2 basics.**
   *Good answer:* access tokens, resource server vs client, JWT vs opaque tokens, how a resource server validates a token.
3. **How do you secure endpoints by role?**
   *Good answer:* `SecurityFilterChain`, `@PreAuthorize`.
4. **How do you handle secrets? How do you deal with a CVE in a transitive dependency?**
   *Good answer:* Vault, and forcing or upgrading the version in Gradle. The build files here have many of these overrides, so it's directly relevant.
5. **What is CORS, and what is CSRF?**

## 6. Microservices and infrastructure (~5 min)

1. **What does an API gateway or proxy do?**
   *Good answer:* routing, auth relay, a single entry point.
2. **Why store HTTP sessions in Redis?**
   *Good answer:* stateless app nodes, horizontal scaling, sticky sessions not needed.
3. **How do services call each other, and how do you handle timeouts and failures?**
4. **Spring Batch basics.**
   *Good answer:* Job → Step → reader/processor/writer, chunk processing, restartability. Also ask what Quartz adds and how to stop a job running twice on multiple nodes.
5. **How do you use Actuator health checks, and how do you read logs to debug a production issue?**

## 7. Angular, PrimeNG and TypeScript (~10 min)

1. **Component lifecycle hooks. Where do you call an API, and why not in the constructor?**
2. **`@Input`/`@Output` vs a shared service for component communication.**
3. **Observables vs Promises. `switchMap` vs `mergeMap`.**
   *Good answer:* `switchMap` cancels the previous request, as in a search-as-you-type field.
4. **How do you avoid memory leaks from subscriptions?**
   *Good answer:* `async` pipe, `takeUntil`, `takeUntilDestroyed`.
5. **HTTP interceptors.**
   *Good answer:* adding the auth token, global error handling, loading spinners.
6. **Route guards and lazy-loaded modules.**
7. **Reactive vs template-driven forms. How do you write a custom or cross-field validator?**
8. **Standalone components, signals, and the new control flow (`@if`/`@for`).** These came between Angular 15 and 20 and are relevant to the upgrade.
9. **Change detection. What does `OnPush` do?**
10. **PrimeNG:** have they built a `p-table` with lazy loading, paging, sorting and filtering against a backend?

## 8. Testing (~5 min)

1. **Unit vs integration tests. `@WebMvcTest` vs `@SpringBootTest`. `@Mock` vs `@MockBean`.**
2. **How do you test a service that calls a repository and an external API?**
3. **How would you test a BPMN file?**
   Point them at the approach in [PershingAddProductCoverageBpmnTest.java](./cminsight-frontend-workflows/src/test/java/com/fifththird/commercial/cm/wf/processes/PershingAddProductCoverageBpmnTest.java): parse the XML and assert flows and conditions, or run the process on an in-memory engine.
4. **Angular:** how do you test a component that uses `HttpClient`?
   *Good answer:* `HttpTestingController`, spies, `TestBed`.

## 9. Scenario and design questions (~10 min)

These questions usually separate stronger candidates best.

1. **Design a feature:** "A user submits a new product coverage request. A group approves or rejects it within 2 business days. The requestor gets an email." Walk through the Angular screen, REST API, DB tables, workflow, notification and security.
2. **Production debugging:** "Users get 401s after login, but only some of the time."
   *Good answer:* token expiry, Redis session, a gateway not relaying the token, clock skew, multiple nodes.
3. **Code review:** show them a 20-line controller with field injection, no validation, a swallowed exception and an N+1 loop. Ask what they'd change.
4. **Git:** merge vs rebase, resolving conflicts, their branching and PR workflow.

---

## Suggested 60-minute plan

| Time | Area |
|---|---|
| 5 min | Intro and what they built recently (dig into their own project) |
| 10 min | Java and Spring Boot |
| 8 min | JPA and SQL |
| 10 min | Angular and RxJS |
| 7 min | Security, microservices, Activiti (pick based on role) |
| 15 min | Design or debug scenario, plus a code review snippet |
| 5 min | Their questions |

## What to expect at 3 years

- **Must have:** solid Java and Spring Boot REST, JPA basics, Angular components, services, RxJS basics, writes unit tests, comfortable with Git.
- **Good to have:** OAuth2 and JWT, Redis, Batch, gateway concepts, Angular upgrades.
- **Bonus:** BPMN or Activiti/Camunda, DB2, Vault, experience fixing CVEs.
- **Red flags:** can't explain what they personally built, only gives textbook answers with no "in my project we…", or has never debugged a production issue.

## Scoring sheet

| Area | Score (1–5) | Notes |
|---|---|---|
| Core Java | | |
| Spring Boot / REST | | |
| JPA / SQL | | |
| Workflow / BPMN | | |
| Security | | |
| Microservices / Infra | | |
| Angular / RxJS | | |
| Testing | | |
| Design / Debugging | | |
| Communication | | |
| **Overall recommendation** | Hire / No hire / Maybe | |

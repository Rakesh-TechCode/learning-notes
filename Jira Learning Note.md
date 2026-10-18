# Jira Fundamentals — Phase 1 Notes

**Target:** 3+ YOE Java Backend Developer  
**Goal:** Understand and use Jira confidently in a real backend development team.

---

## 1. What Is Jira?

**Jira** is a work-tracking tool used by software teams to plan, assign, track, and manage development work, bugs, and tasks.

**Example:** A developer receives a Jira ticket to implement a payment API in Spring Boot, works on it, and updates its status as progress is made.

**Interview answer:** Jira helps software teams organize work, track progress, assign responsibility, and manage bugs and development tasks.

## 2. Why Do Software Teams Use Jira?

Teams use Jira to:

- Track development tasks and bugs.
- Assign work to responsible team members.
- Monitor progress through statuses.
- Record discussions and requirements.
- Coordinate development, code review, and testing.

**Real-world example:** If the Payment API returns HTTP 500 for invalid input, the team creates a bug ticket, assigns it to a developer, tracks the fix, and verifies it through testing.

**Developer perspective:** Keep your tickets updated, communicate blockers, and ensure your work is traceable.

## 3. Jira vs Git/GitHub

| Tool | Main purpose |
|---|---|
| Jira | Tracks work, requirements, and progress |
| Git | Tracks source-code changes and versions |
| GitHub | Hosts Git repositories and supports pull requests and code collaboration |

**Example workflow:**

```text
Jira ticket: PAY-201
       ↓
Git branch: feature/PAY-201-payment-api
       ↓
Write Spring Boot code
       ↓
Commit changes
       ↓
Create Pull Request
       ↓
Code Review → QA → Done
```

**Remember:** Jira tracks *what work needs to be done*; Git tracks *changes to the code*; GitHub provides a platform for collaboration around that code.

## 4. Jira Project

A **Jira project** is a workspace used to organize related work for a product, service, or team.

**Example:** An e-commerce project may track work for Payment, Orders, Cart, and Notifications.

A project contains issues such as stories, tasks, and bugs.

**Remember:** A project is the container for related work; an issue represents a specific piece of work.

## 5. Jira Issue / Ticket

An **issue** (commonly called a ticket) is an individual work item tracked in Jira.

Examples:

- Implement a payment endpoint.
- Fix an order-creation bug.
- Add validation for payment amounts.
- Write unit tests for a service.

A ticket typically has a summary, description, issue type, status, assignee, reporter, and other relevant details.

**Example:**

```text
Key: PAY-201
Summary: Implement Payment API
Type: Task
Status: In Progress
Assignee: Developer responsible for implementation
```

**Remember:** The issue key, such as `PAY-201`, uniquely identifies the ticket within the Jira environment.

## 6. Basic Jira UI / Navigation

As a developer, be comfortable with these areas:

- **Projects:** Open the relevant project.
- **Search / Issues:** Find a ticket using its key or search.
- **Board:** View issues and their workflow statuses.
- **Backlog:** View and organize upcoming work when the team uses one.
- **Issue details:** Read the description, check the assignee and status, review comments, and inspect acceptance criteria.

**Practical exercise:** Open a ticket and identify its issue key, summary, type, status, assignee, description, and acceptance criteria.

**Developer focus:** Know how to find a ticket and understand its current state. You do not need Jira administrator knowledge for this.

## 7. Assignee

The **assignee** is the person currently responsible for working on an issue.

**Example:** A payment API implementation ticket is assigned to the backend developer who will implement the endpoint.

**Interview answer:** The assignee is the person responsible for working on the Jira issue.

## 8. Reporter

The **reporter** is the person who created or reported the issue.

**Example:** A QA engineer reports that invalid payment details cause an HTTP 500 response. The backend developer is assigned the bug.

**Key difference:**

| Assignee | Reporter |
|---|---|
| Responsible for working on the issue | Created or reported the issue |
| May implement a fix | May be QA, a developer, or another team member |

**Remember:** Reporter and assignee can be different people. The reporter does not have to be the person who fixes the issue.

## 9. Labels

**Labels** are tags used to categorize and filter issues.

Examples:

```text
payment
backend
api
production-bug
```

A payment-related ticket could have the labels `payment` and `backend`.

**Why useful?** Labels help teams find related issues without relying only on the ticket title.

**Remember:** A label categorizes an issue; it does not assign responsibility or change the issue's status.

## 10. Comments

**Comments** are used to communicate information, progress, questions, and blockers directly on a Jira issue.

**Example developer comment:**

> Payment API implementation is complete. Unit tests are passing. PR is ready for code review.

**Example blocker comment:**

> Blocked because the payment provider's sandbox credentials are not available yet.

**Best practice:** Keep comments clear, relevant, and actionable. Record important progress and blockers where the team can find them.

## 11. Attachments

**Attachments** are files added to a Jira issue to provide supporting information.

Examples:

- Error screenshots.
- Relevant log files.
- API request/response examples.
- Requirement documents.

**Example:** QA attaches a screenshot and relevant error details to a payment failure bug.

**Remember:** Attachments provide evidence or supporting information; comments explain the context.

Do not upload passwords, access tokens, or other secrets.

## 12. Ticket Description

The **description** explains what work is required, what the problem is, and any relevant context.

**Example:**

```text
Summary:
Implement Payment API

Description:
Create a Spring Boot REST endpoint that accepts
payment requests and initiates payment processing.

The endpoint should validate the request and return
an appropriate response.
```

A good description should be clear enough for the developer to understand the required work.

**Remember:** The summary tells you *what the ticket is about*; the description explains *the details of the work*.

## 13. Acceptance Criteria — Basic Understanding

**Acceptance criteria** are specific, testable conditions that must be satisfied for a requirement to be accepted.

**Example: Payment API**

```text
1. Valid payment requests are processed successfully.
2. Missing required fields are rejected.
3. Invalid payment amounts are rejected.
4. Appropriate HTTP responses are returned.
```

These criteria help the developer understand expected behavior and help QA verify the implementation.

**Description vs Acceptance Criteria**

| Description | Acceptance criteria |
|---|---|
| Explains the work and its context | Defines the conditions for acceptance |
| Describes what needs to be built | Defines how expected behavior can be verified |

**Remember:** A ticket can have a good description but still need clearer acceptance criteria.

---

## Quick Revision — Important Differences

| Concept | Remember |
|---|---|
| Jira vs Git | Work tracking vs source-code version control |
| Project vs Issue | Workspace/container vs individual work item |
| Assignee vs Reporter | Responsible person vs issue creator/reporter |
| Summary vs Description | Short title vs detailed explanation |
| Description vs Acceptance Criteria | Work context vs testable conditions |
| Comments vs Attachments | Discussion/context vs supporting files |
| Labels vs Assignee | Categorization vs responsibility |

## Practical Developer Workflow

```text
Requirement
    ↓
Jira Ticket
    ↓
Understand Description + Acceptance Criteria
    ↓
Check Assignee + Status
    ↓
Create Git Branch
    ↓
Implement Spring Boot Changes
    ↓
Commit + Pull Request
    ↓
Code Review
    ↓
QA Testing
    ↓
Fix Bugs if Required
    ↓
Update Jira Status → Done
```

**Important:** The exact workflow and status names depend on the team's Jira configuration and development process.

## 3+ YOE Interview Quick Answers

**1. What is Jira?**  
Jira is a work-tracking tool used to manage development tasks, bugs, responsibilities, and progress.

**2. How is Jira different from Git?**  
Jira tracks work items and progress, while Git tracks source-code changes and versions.

**3. What is the difference between an assignee and a reporter?**  
The assignee is responsible for working on the issue, whereas the reporter created or reported it.

**4. Why are acceptance criteria important?**  
They define testable conditions that help developers implement the expected behavior and help QA verify it.

**5. What do you do when a Jira ticket is blocked?**  
I communicate the blocker through the team's agreed process, document relevant details in the ticket, and coordinate with the person or team needed to resolve it.

**6. How do you use Jira during development?**  
I review the ticket and acceptance criteria, work on the associated code changes, link or reference the ticket where appropriate, update progress, and coordinate through review and testing until the work is complete.

---

## Phase 1 Completion Tracker

- [x] 1. What is Jira?
- [x] 2. Why software teams use Jira
- [x] 3. Jira vs Git/GitHub
- [x] 4. Jira Project
- [x] 5. Jira Issue/Ticket
- [x] 6. Basic Jira UI/navigation
- [x] 7. Assignee
- [x] 8. Reporter
- [x] 9. Labels
- [x] 10. Comments
- [x] 11. Attachments
- [x] 12. Ticket description
- [x] 13. Acceptance Criteria

**Phase 1: Complete — 13/13**

**Next:** Phase 2 — Jira Issue Types: Epic → Story → Task → Bug → Sub-task. Finish the current Epic quiz before moving to Story.

---
parent: How We Deliver
---
# Environments

- Each has its own purpose and are deployed independently from each other (e.g. separate servers, DBs, etc).
- Each gets a fresh deployment of new code once the automated tests for that particular commit pass.


## CI

|            |                                                                                                                                  |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Git Branch | `main` or `master`                                                                                                               |
| Purpose    | Functional testing of recently merged code in a more production-like environment prior to being included in a release candidate. |
| When       | Testing is at engineer discretion and is recommended if you haven't been able to fully test locally.                             |


## RC

|            |                                                                                                                                        |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Git Branch | `release-candidate`                                                                                                                    |
| Purpose    | Functional testing by engineers prior to production deployment.<br/>Acceptance testing by stakeholders.<br/>Demonstration environment. |
| When       | Testing is **required** before the code can be released.                                                                               |

## Production

|            |                                                                |
| ---------- | -------------------------------------------------------------- |
| Git Branch | `release`                                                      |
| Purpose    | Environment for end users.                                     |
| When       | Occasional smoke testing may be required, particularly after . |
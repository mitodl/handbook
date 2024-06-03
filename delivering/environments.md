---
parent: How We Deliver
---
# Environments

- Each environment is deployed independently (e.g. separate servers, DBs, etc).
- Each environment deploys new code included in the release once the automated tests for that particular release-commit pass successfully.

## Environment Name

|              |                                                      |
|--------------|------------------------------------------------------|
| Git Branch   | The Github branch that the environment has deployed. |
| Purpose      | The purpose of the environment.                      |
| Expectations | Expectations for environment usage and processes.    |


## CI

|              |                                                                                                                                    |
|--------------|------------------------------------------------------------------------------------------------------------------------------------|
| Git Branch   | `main` or `master`                                                                                                                 |
| Purpose      | Testing code from `main` or `master` in a deployed environment prior to being included in a release candidate and release process. |
| Expectations | Code is tested and functionality confirmed shortly after releasing to CI.                                                          |


## RC

|              |                                                                                                                           |
|--------------|---------------------------------------------------------------------------------------------------------------------------|
| Git Branch   | `release-candidate`                                                                                                       |
| Purpose      | Functional testing prior to production deployment.<br/>Acceptance testing by stakeholders.<br/>Demonstration environment. |
| Expectations | Mock data can be created.  Some risk is acceptable by deploying code that cannot be tested locally by a developer.        |

## Production

|              |                                                                                |
|--------------|--------------------------------------------------------------------------------|
| Git Branch   | `release`                                                                      |
| Purpose      | Environment for end users.                                                     |
| Expectations | Limit creation of mock data.  Code and functionality should pose minimal risk. |
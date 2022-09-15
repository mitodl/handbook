---
parent: How To
---
# New Projects

When starting a new project, we have a baseline of what's necessary
from an infrastructure perspective. Here's a checklist:

- [ ] Do you have a README?
- [ ] Can you run the tests for your code? Is that documented in the README?
- [ ] Do you have docker setup for local development?
- [ ] Do you have a LICENSE file?
- [ ] Are the tests running on a CI server like Travis?
- [ ] Do you have relevant linters setup for the languages you're using?


READMEs for your project.

- [ ] Does it include how to install the app?
- [ ] Does it include how to run the tests?
- [ ] Does it contain a build status or code coverage badge?
- [ ] Does it link to relevant documents?
- [ ] Does it mention the software license used?


For Django projects specifically..

- [ ] Does it use the same database backend locally as it will in production?
- [ ] Does it support both python 2.7 and either python 3.4 or 3.5 (pick one)?
- [ ] Does it run pylint & pep8 as linters?

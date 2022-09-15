---
parent: How We Deliver
---
# How We Work
We hire smart and talented people and support them in doing their best work. The core
component of how we think about "getting work done" is that we trust everyone to take
ownership of the tasks/features/projects that they are involved in.

In order to ensure that we maintain a common understanding of what is involved in our
work and a high quality of our output, it is useful to highlight the stages of our work
and the value that each step provides.

## Project Shaping
The first step of any project is to understand the scope and goals of the work to be
done. This typically involves documenting the user-facing value that the work will
provide and collaborating with product managers to ensure that everyone is properly
aligned and understands what needs to be done.

## Architecture
Every piece of functionality for our products has an impact on the architecture of our
platforms, and the design/experience of our end users. To that end, it is important that
we have a common understanding of how we are going to build a piece of functionality and
why we have chosen that approach. For large projects (with “large” being a subjective
term) we write [Request For Comments
(RFC)](https://blog.pragmaticengineer.com/scaling-engineering-teams-via-writing-things-down-rfcs/)
documents that capture our thinking on these questions. These [RFC
documents](https://github.com/mitodl/hq/discussions/categories/rfc) are then shared
widely throughout the team to ensure that everyone has the opportunity to provide input
and learn about the work being done.

## Design
For visual aspects of our work the RFC may be replaced with a graphic design, a set of
wireframes, etc. This will provide a way for engineers who will be implementing a
feature to understand the user flow and provide input on constraints that may influence
the final design.

## Planning
At the tactical level, it is necessary to understand the steps required to complete a
project and the dependencies across those steps. Timely, clear, and effective
communication is the most important tool in this stage. For small, well understood tasks
this will most likely mean creating an issue in GitHub to track progress and coordinate
conversations. For more complex or speculative projects this might mean convening a
meeting with relevant stakeholders, breaking the task into multiple stages and
documenting the overarching roadmap, or opening a discussion thread in a shared channel
(e.g. [GitHub Discussions](https://github.com/mitodl/hq/discussions))

## Implementation
Once the scope of a problem is understood it is necessary to create a way to deliver the
intended functionality. This will typically be done through writing code that is
incorporated into one of our source repositories. Sometimes it will mean using
third-party services or tools. Maybe it requires something else entirely. As the person
solving the problem, you are best situated to know how best to bring it to resolution.

For more details on our development process please refer to the guide [here](/delivering/development-process.html).

## Validate
Once a feature is added to one of our products, we need to ensure that it functions
properly and will continue to do so. This typically involves writing automated tests
using one of our selected frameworks. This may also involve writing a manual test plan
and ensuring that it is executed on a periodic basis.

## Document
A feature is not useful if nobody knows that it exists or how to use it. In order to
ensure that we don’t have to explain something more than once, it is necessary to write
documentation that addresses the needs of our peers and end-users. This documentation
may take many forms, including but not limited to:
- In-line code comments that explain the “why” for a given approach
- Meaningful and descriptive variable, function, and class definitions
- README documentation that lives with the source code. There is nothing preventing the
  presence of multiple README documents if it will provide more value to be located
  closer to the code it addresses.
- In-line documentation (e.g. docstrings, type annotations, code contracts) that
  provides detailed information about how and why a given function/class will be
  used. This allows us to use automated tools (e.g. sphinx) to create documentation
  sites for our projects.
- Architecture diagrams, flow charts, sequence diagrams, etc. to visually represent how
  a project is structured, how logical flows are expected to work, etc.

## Deliver To Production
While you may not be the person who is actually shipping the functionality into a
production environment, it is your responsibility to make sure that it happens. This
includes:
- Ensuring that you get timely feedback on your pull requests
- Personally validating functionality, or helping others to validate it for you
- Working with platform engineering to inform them of any requirements to support your
  functionality (e.g. “I need a new S3 bucket”, “This will require a Redis database and
  expanded IAM permissions”, etc.)

For a more comprehensive list of the application level details involved in something
being ready for production refer to the "[production readiness](/delivering/production-ready.html)"
guide. The steps involved in getting the code delivered to production are documented in
the [release process](/delivering/release-process.html) guide.

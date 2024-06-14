---
parent: How We Deliver
---

# Code Review
A guide for reviewing code and having your code reviewed.

> Peer code reviews are the single biggest thing you can do to improve your code - [Jeff Atwood](http://blog.codinghorror.com/code-reviews-just-do-it/)

# Purpose
Code review is an important part of a team's development process. It helps to:

* disseminate knowledge about the codebase/technology/techniques across teams
* increase awareness of the features being developed
* provide opportunities to educate junior engineers (actually all engineers)
* find alternative solutions to problems
* catch poor architectural decisions and bugs
* develop a sense of team ownership of the code (no more his/her code)

When doing code reviews, **the process is as important as the result**, therefore, some guidelines are presented below to help improve the process of code review.

# Everyone
* Accept that many programming decisions are opinions. Discuss tradeoffs, which
you prefer, and reach a resolution quickly.
* Ask questions; don't make demands. ("What do you think about naming this
`:user_id`?")
* Ask for clarification. ("I didn't understand. Can you clarify?")
* Avoid selective ownership of code. ("mine", "not mine", "yours")
* Avoid using terms that could be seen as referring to personal traits. ("dumb",
"stupid"). Assume everyone is attractive, intelligent, and well-meaning.
* Be explicit. Remember people don't always understand your intentions online.
* Be humble. ("I'm not sure - let's look it up.")
* Don't use hyperbole. ("always", "never", "endlessly", "nothing")
* Don't use sarcasm.
* Talk in person if there are too many "I didn't understand" or "Alternative
solution:" comments. Post a follow-up comment summarizing offline discussion.

# Having Your Code Reviewed
* Review your code before anybody else.
* Provide context about what the change is and why is it important - Write informative commit messages [4] and pull request descriptions.
* Use the pull request description template [3]. You only need to answer the relevant questions. Pro tip: use the bookmarklet [11] or chrome extension [12] for pull request description templates.
* Annotate your code reviews. Add comments anywhere that you feel will help the reviewer understand your code.
* Be grateful for the reviewer's suggestions. ("Good call. I'll make that
change.")
* Don't take it personally. The review is about the code, not you.
* Explain why the code exists. ("It's like that because of these reasons. Would
it be more clear if I rename this class/file/method/variable?")
* Seek to understand the reviewer's perspective.
* Respond to every comment. If you can't or won't change something, let the reviewers know why.
* Wait to merge the branch until someone has :+1:'d your PR and Continuous Integration
tells you the test suite is green in the branch.
* Merge once you feel confident in the code and its impact on the project.
* Delete the backing branch.

# Reviewing Code
* Code review often and for short sessions. The effectiveness of code review drops after around an hour. Set aside time throughout your day to coincide with breaks in order to not disrupt your flow.
* Review at least part of the code, even if you can't do all of it. If you can't do all of it, comment on the PR to indicate this, so the author can get a full review.
* Review the right things, let tools to do the rest. Don't focus on reviewing things that can be reviewed by tools (e.g. eslint, pylint, etc). If the tools don't provide good feedback, change the config or improve the tools.

Understand why the code is necessary (bug, user experience, refactoring). Then:

* Communicate which ideas you feel strongly about and those you don't.
* Identify ways to simplify the code while still solving the problem.
* If discussions turn too philosophical or academic, move the discussion offline. In the meantime, let the
author make the final decision on alternative implementations.
* Offer alternative implementations, but assume the author already considered
them. ("What do you think about using a custom validator here?")
  * If possible, use GitHub's suggestion feature to propose specific changes to a line or two of code.
  * If you want to propose a complex alternative, discuss with the author first, then consider opening a PR with your changes that targets the PR you are reviewing. Don't add commits directly to the branch you are reviewing. This helps to prevent misunderstandings and merge conflicts.
* Seek to understand the author's perspective.
* Sign off on the pull request with a :thumbsup: or "Ready to merge" comment... always!
* Compliment/reinforce good practices.

Some suggestions of things to look for when reviewing code:

* Does the code work? Does it perform its intended function, the logic is correct etc.
* Is all the code easily understood? Is code unnecessarily complex ?
* Can variable/method names be improved?
* Are classes/files/methods becoming too large? Should they be split into smaller pieces?
* Does it conform to the agreed coding conventions?
* Is there any redundant or duplicate code? Is the code as modular as possible?
* Could the design of the solution be improved (e.g. single responsibility principle, open/closed principle) ?
* Is the code testable? (i.e. class design is such that you can easily mock dependencies, etc)
* Are there tests? It's not enough to be testable, code should be tested.
* Things related to your area of expertise

Pro tip: Keep your own checklist of things you look for in a code review as in [9] [7].

# Links
Links from places on the Internet that helped write this:

(The most useful ones are [9], [1], [3] and [2].)

* [1] [Talk - Implementing a Strong Code-Review culture](https://www.youtube.com/watch?v=PJjmw9TRB7s) - the talk that inspired this.
* [2] [Thoughtbot's Guide for Code-Review](https://github.com/thoughtbot/guides/blob/master/code-review/README.md) - straight to the point guidelines on code review. A lot of the content was taken from there.
* [3]  [Pull request templates make code review easier](https://quickleft.com/blog/pull-request-templates-make-code-review-easier/) - introduction to pull request templates.
* [4] some articles on how to write good commit messages - [first](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html), [second](http://www.slideshare.net/TarinGamberini/commit-messages-goodpractices),  [third](https://wiki.openstack.org/wiki/GitCommitMessages)
* [5] [Am I My Code?](http://mfeckie.github.io/Am-I-My-Code/) - this article explores some strategies for giving feedback when reviewing code.
* [6] [Code review at the Dropbox iOS team](http://www.objc.io/issue-22/dropbox.html) - the article goes over their whole process of review. It has some interesting ideas.
* [7] [Code Review Checklist](http://blog.fogcreek.com/increase-defect-detection-with-our-code-review-checklist-example/) - from Fog Creek Software
* [8] [Effective code review tips](http://blog.fogcreek.com/effective-code-reviews-9-tips-from-a-converted-skeptic/) - from Fog Creek Software
* [9] [Code Review Best Practices](http://kevinlondon.com/2015/05/05/code-review-best-practices.html) - really good article on code review best practices
* [10] [11 proven practices to improve code reviews](http://www.ibm.com/developerworks/rational/library/11-proven-practices-for-peer-review/) - nice article backed by data from studies.
* [11] [Pull request template bookmarklet](https://quickleft.com/blog/pull-request-template-bookmarklet/)
* [12] [Pull request template chrome extension](https://github.com/sprintly/pull-request-template-chrome-extension)


Thanks to @marionzualo for compiling this list.
https://gist.github.com/marionzualo/82fbf4e3d20ef253d3ef

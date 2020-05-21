@Meta
title = "Adventures in GSoC 6"
slug = "adventures-in-gsoc-6"
date = "2017-08-23"
author = "Connor"

@Content
During the last weeks, I finished [implementing][17935] custom element upgrades and fixed various
failing tests.

## Pull Requests:
 * Merged [#17935: Custom element upgrades][17935]
 * Merged [#18074: Enqueue attribute changed callback when replacing attr][18074]
 * Merged [#18092: Only run DOMTokenList update steps if token was found][18092]
 * Merged [#18146: Annotate insertAdjacentHTML with CEReactions][18146]

## Overview
As I entered the final few weeks of the project, I began to triage and categorize the remaining
custom element test failures. Some tests fail due to Servo missing an implementation of a different
spec. This includes the Shadow DOM and various missing attributes. These tests are ok to remain
failing as they do not siginify an issue with the custom element implementation.

The triaging process involves picking a specific test failure and looking at the test itself. I look
for any APIs that are used that servo does not implement. If there are none, I then run the test and
find the failing assertion. This typically will give more insight into the problem. For example,
an upgrade reaction may not occur like it should. Once I figure out what assertion is failing, I
step through the Rust implementation and attempt to figure out why or why not certain reactions were
triggered.

The failures found in [#18074][18074] and [#18146][18146] were due to issues with the custom element
implementation. The former failure was due to the attribute change callback not being fired when
an attribute node was being replaced. The latter was due to Element's `insertAdjacentHTML` missing
the `CEReactions` annotation.

## Next Steps
This week is my final week on the project, I have a [few more][test_failures] test failures to look
into. I will also be writing a blog post discussing my GSoC experience and the custom element
implementation. The final goal is to enable custom element by default in Servo.

For more information on my gsoc project, check out my [proposal and roadmap][roadmap].

[17935]:https://github.com/servo/servo/pull/17935
[18074]:https://github.com/servo/servo/pull/18074
[18092]:https://github.com/servo/servo/pull/18092
[18146]:https://github.com/servo/servo/pull/18146
[roadmap]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md#schedule-of-deliverables
[test_failures]:https://github.com/cbrewster/gsoc2017/blob/master/notes/test_failures.md
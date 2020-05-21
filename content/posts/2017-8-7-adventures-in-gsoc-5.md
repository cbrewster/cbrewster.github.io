@Meta
title = "Adventures in GSoC 5"
slug = "adventures-in-gsoc-5"
date = "2017-08-07"
author = "Connor"

@Content
During the last weeks, I finished adding `[CEReactions]` annotations where used in the spec and
[implemented][17935] custom element upgrades.

## Pull Requests:
 * Meregd [#17761: Add [CEReactions] to webidls][17761]
 * Opened [#17935: Custom element upgrades][17935]

## Overview
Custom element upgrades are the last major section of the custom element implementation. Upgrades
handle scenarios where a custom element is created prior to it being defined. I have already written
about custom element upgrades in another [blog post][custom_element_upgrades]. The Servo
implementation was rather straight forward; however, I did encounter an issue with the custom
element spec and filed an issue [here][ce_issue].

## Next Steps
For the next couple weeks I will triage the remaining test failures. These failures are either due
to an issue with my implementation or a missing implementation of a different spec in Servo (this
includes Shadow DOM, the Selection API, and others). I added a [file][test_failures] to the gsoc
repository that categorizes all the test failures. During the next week, I will finish investigating
the test failures and create patches for any tests that are failing due to a mistake in the custom
element implementation.

For more information on my gsoc project, check out my [proposal and roadmap][roadmap].

[17935]:https://github.com/servo/servo/pull/17935
[17761]:https://github.com/servo/servo/pull/17761
[custom_element_upgrades]:https://cbrewster.github.io/2017/06/08/custom-element-upgrades/
[roadmap]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md#schedule-of-deliverables
[ce_issue]:https://github.com/whatwg/html/issues/2876
[test_failures]:https://github.com/cbrewster/gsoc2017/blob/master/notes/test_failures.md
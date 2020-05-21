@Meta
title = "Adventures in GSoC 2"
slug = "adventures-in-gsoc-2"
date = "2017-06-19"
author = "Connor"

@Content
During the last week, I finished up the remaining work on implementing the `[HTMLConstructor]`
WebIDL extended attribute and implemented custom elements into element creation.

## Pull Requests:
 * Merged [#17224: WebIDL HTMLConstructor support][17224]
 * Opened [#17381: Custom element creation ][17381]

## Overview

As I finished up the remaining work on [#17224][17224], I began to focus on the
[_create an element_][create_element_spec] steps which was the original goal for this week and last
week according to my [roadmap].

After `[HTMLConstructor]` was implemented, the implementation for the _create and element_ steps was
straightforward.

The first step was to add a lookup by local name method for the `CustomElementRegistry` and use that
to find any custom element definition if one exists when creating an element.

The next step was to add a method to `CustomElementDefinition` which constructs a new element from
the definition. When creating a new element, this method is used under the following conditions:

 1. The definition is for an _Autonomous Custom Element_.
 2. The _synchronous custom elements flag_ is set.

If those conditions are unmet and a custom element definition was found, a custom element upgrade
reaction is used to setup the custom element. I have not implemented reactions or upgrades yet, so
I left this portion unimplemented.

I also added the beginning support for the `is` attribute and wired up the _synchronous custom
elements flag_.

## Next Steps

According to the [roadmap], the next step is to implement custom element reactions
(June 18 - July 1).

This includes the following:
 1. Validating and storing _lifecycle callbacks_ and _observed attributes_ when a new custom element
 is defined.
 3. Implementing the various custom element reaction queues.
 4. Enqueueing the callback reactions where appropriate.

This does not include implementing upgrade reactions or the `[CEReactions]` WebIDL extended
attribute.

[17224]:https://github.com/servo/servo/pull/17224
[17381]:https://github.com/servo/servo/pull/17381
[create_element_spec]:https://dom.spec.whatwg.org/#concept-create-element
[roadmap]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md#schedule-of-deliverables
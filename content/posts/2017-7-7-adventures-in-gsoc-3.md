@Meta
title = "Adventures in GSoC 3"
slug = "adventures-in-gsoc-3"
date = "2017-07-07"
author = "Connor"

@Content
During the last weeks, I landed the [custom element creation PR][17381] and implemented custom
element callback reactions. I also spent some time in Hawaii! Anyways, back to business...

## Pull Requests:
 * Merged [#17381: Custom element creation][17381]
 * Opened [#17614: Implement custom element reactions][17614]

## Overview
After finishing the remaining work on custom element creation, I began to implement custom element
callback reactions. CE Reactions are complex and I initially split the task into three parts:

 1. Implement callback reactions.
 2. Implement the upgrade reaction.
 3. Implement the `[CEReactions]` WebIDL Attribute.

After reading the spec more closely and begining on an initial implementation, I realized that it
might not be possible to implement steps 1 and 2 without also imeplementing step 3. After more
implementation, I finally concluded that the two can be implemented separately, but step 3 is a
relatively easy step to implement anyways; however, I have decided to keep step 3 separate as it
requires adding `[CEReactions]` in many `webidl` files and would be cleaner to review separately. I
have allocated less than a week for that work.

To understand my initial confusion, we will need to dive into the
[custom element reactions spec][reactions_spec] and understand how reactions actually work. The
implementation requires multiple stacks and queus which each have their own function. I will go over
them briefly.

### Stacks and Queues
#### Custom Element Reaction Stack
This stack is the "master" stack. Each _unit of related similar-origin browsing contexts_ (In
Servo-land, a single script thread) contains one of these. Each item in this stack is an _element
queue_. If the stack is not empty, it has a _current element queue_ and it always has a special
_element queue_ called the _backup element queue_.

#### Element Queue
Each item in this queue is an `Element`. If an element is placed in this queue, it means that the
element has reactions that need to be invoked.

#### Custom Element Reaction Queue
Every `Element` owns a _custom element reaction queue_. Each item in this queue is a reaction which
is an enum with the following variants:

 1. Upgrade Reaction (To be implemented later).
 2. Callback Reaction with an associated callback function and list of arguments.

### `[CEReactions]` and Reaction Flow
All these queues and stacks make it seem like the reactions are overcomplicated; however, it was
designed this way for a reason. When reactions are invoked, user script is also invoked. This could
be very dangerous as the browser may be in the middle of an operation that could be messed up by the
user script. The main goals of this implementation are to:

 1. Make reactions appear to be synchronously run from the perspective of the user script.
 2. Make sure all reactions are invoked in the correct order.
 3. Wait until returning to user script before invoking any reactions.

To maintain both invariants, the idea is to mark all operations in the webidl interfaces that may
trigger reactions. Before any of these operations are ran, a new _element queue_ is pushed onto the
_custom element reaction stack_. Whenever a reactions should be invoked, the reaction is queued on
its associated element's _custom element reaction queue_ and the element is added to the _current
element queue_ of the _custom element reaction stack_. After the operation is finished, the _current
element queue_ is popped and each element in the queue invokes all of the reactions in its queue in
the proper order.

This is where my confusion occurred with the necessity of `[CEReactions]` for step 1. There is one
piece of information I did miss and that is the _backup element queue_. There are some
user-initiated operations that may cause a reaction but do not go through a DOM interface. This
would mean that the _custom element reaction stack_ may not have a _current element queue_;
instead the _backup element queue_ is used and the reactions are invoked via a microtask.

My [current implementation][17614] does not have the `[CEReactions]` attribute added to the webidl
files. It relies on the _backup element queue_ instead. The reactions are also invoked instantly
rather than by a microtask. These changes are large and I think it is best to try to split the work
into smaller chunks.

## Next Steps
I have adjusted my roadmap to account for vacation and the change of order as I plan to finish
`[CEReactions]` before implementing the upgrade reaction.

[17614]:https://github.com/servo/servo/pull/17614
[17381]:https://github.com/servo/servo/pull/17381
[reactions_spec]:https://html.spec.whatwg.org/multipage/#custom-element-reactions
[roadmap]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md#schedule-of-deliverables
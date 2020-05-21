@Meta
title = "Adventures in GSoC 4"
slug = "adventures-in-gsoc-4"
date = "2017-07-18"
author = "Connor"

@Content
During the last week, I landed [custom element reactions][17614] and opened a [PR][17761] to add the
`[CEReactions]` extended attribute to all the webidl files according to their spec.

## Pull Requests:
 * Merged [#17614: Implement custom element reactions][17614]
 * Opened [#17761: Add [CEReactions] to webidls][17761]

## Overview
This week's work has been rather straight-foward. Once [custom element reactions][17614] were
implemented, I needed to sprinkle `[CEReactions]` everywhere in the webidl files. This process
consisted of going through each webidl file in Servo, going to its associated spec and checking
if any of the attributes or methods needed to be annotated with `[CEReactions]`.

The `[CEReactions]` extended attribute marks any attributes or methods that can somehow trigger
a custom element reaction. Extra code is added in the codegen stage that handles the custom element
reactions. This code catches all reactions that should be invoked and waits until Servo is returning
to author script before invoking any of the reactions.

I am now investigating the test failures to see if I missed any places where callback reactions
should be enqueued.

## Next Steps
The next step, implementing the upgrade reactions, is the final large step in implementing custom
elements! The upgrade reaction handles taking an uncustomized element and upgrading it into a custom
element. This can occur if an element using a custom element name is added to a document and later
a custom element definition is added to the registry with that name. The element is then upgraded
using this new definition. See the following example:

```js
class MyCustomElement extends HTMLElement {
    constructor() {
        super()
    }
}

// This creates a regular HTMLElement that has a valid custom element name
var element = document.createElement('my-custom-element')
document.body.append(element)
// Now create a custom element definition with the previously used custom element name.
// This will trigger an upgrade reaction for `element`
customElements.define('my-custom-element', MyCustomElement)
```

Check out my [blog post][custom_element_upgrades] on custom element upgrades for more information on
how the upgrade reaction is implemented.

For more information on my gsoc project, check out my [proposal and roadmap][roadmap].

[17614]:https://github.com/servo/servo/pull/17614
[17761]:https://github.com/servo/servo/pull/17761
[custom_element_upgrades]:https://cbrewster.github.io/2017/06/08/custom-element-upgrades/
[roadmap]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md#schedule-of-deliverables
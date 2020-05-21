@Meta
title = "Adventures in GSoC 1"
slug = "adventures-in-gsoc-1"
date = "2017-06-12"
author = "Connor"

@Content
During the last week, I worked on implementing the `[HTMLConstructor]` WebIDL extended attribute.

## Pull Requests:
 * Opened [#17224: WebIDL HTMLConstructor support][17224]
 * Opened [#17250: Unwrap function before calling IsConstructor][17250]

## Blog Posts
 * [Understanding Custom Element Upgrades][upgrades_blog_post]

## Overview

According to my [roadmap][proposal], I will have the custom element portion of the [_create an element_][create_element_steps] steps implemented by the end of this next week.

I began working on implementing the steps according to the [spec][create_element_steps], but I ran into issues at Step 6.1.2. The call to `Construct` was failing for every custom element constructor that I tried. A javascript exception was always thrown at the line in the constructor that calls the superclass's constructor.

```javascript
class CustomElement extends HTMLElement {
    constructor() {
        super(); // Error: Invalid constructor.
    }
}

customElements.define('custom-element', CustomElement);
```

I scratched my head at this issue, as the spec is clear that the superclass's constructor must be called once in the subclass's constructor. I consulted with other Servo developers on IRC. I found out that HTML elements don't have a constructor exposed to JS; therefore, the call to the superclass's (`HTMLElement` in this case) fails because the `HTMLElement` interface does not expose a constructor.

When Custom Elements were added to the spec, special constructors were added to certain HTML elements via the `[HTMLConstructor]` extended attribute.

Before I could begin work on implementing custom elements in `document.createElement`, I had to implement [`HTMLConstructor`][htmlconstructor_spec].

## Next Steps

During this week I finished the first pass at the implementation and plan to have it polished and merged by the middle of this next week.

Once merged, I will finish the work on implementing the [_create an element_][create_element_steps] steps by the end of this next week.

[17224]:https://github.com/servo/servo/pull/17224
[17250]:https://github.com/servo/servo/pull/17250
[upgrades_blog_post]:https://cbrewster.github.io/2017/06/08/custom-element-upgrades/
[proposal]:https://github.com/cbrewster/gsoc2017/blob/master/proposal.md
[create_element_steps]:https://dom.spec.whatwg.org/#concept-create-element
[htmlconstructor_spec]:https://html.spec.whatwg.org/multipage/dom.html#htmlconstructor
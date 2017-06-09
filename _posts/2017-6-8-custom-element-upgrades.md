---
layout: post
title: Understanding Custom Element Upgrades
---

I am implementing the Custom Element v1 spec in Servo this summer. While preparing for the work I have read over the spec a few times; however, it is not until recently that the upgrade algorithm has begun to make sense. The point of the algorithm is to handle custom elements that have been added to a document before they have been defined.

Example:
```html
<custom-element></custom-element>
<script>
customElements.define("custom-elements", class CustomElement extends HTMLElement { constructor() { super(); } })
</script>
```

In this example, `custom-element` is added to the document before it has been defined, this means the `custom-element` will not have the constructor run and will not behave as expected. To solve this problem, after a custom element definition is added to a registry, all the elements are traversed and any elements that match the local name of the new definition are upgraded.

It seems odd at first because the custom element starts as a HTMLElement but should end up as a CustomElement after the upgrade algorithm. This all needs to be done without removing and readding a new element. We need to actually run the constructor against the already existing element.

## HTMLConstructor
First we need to take a detour and look at the `[HTMLConstructor]` WebIDL extended attribute. Before custom elements, you could not use a class to extend native elements like `HTMLElement` or `HTMLButtonElement`.

Example:
```javascript
class CustomElement extends HTMLElement {
    constructor() {
        super();
    }
}
// Invalid:
new CustomElement();
```

If you try this code you will probably get an error like `Illegal constructor`. This is because native element don't expose a constructor, well at least not until custom elements. `HTMLConstructor` is a new constructor from the custom element spec that adds a constructor for most native HTML elements; however, it is not like a normal constructor.

Note: The call to the `HTMLConstructor` happens when `super()` is called in the subclass constructor.

The above example will only succeed if `CusomtElement` is added to the custom element registry. This is due to the special behavior of the `HTMLConstructor` defined in the spec. I won't go over that too much in detail (see [the spec][HTMLConstructor] for more detail), the important thing to note is that with the custom constructor, we have control over what `super()` actually returns. We can return a newly created element or an element that _already_ exists. The latter is the important part when it comes to custom element upgrades.

## Construction Stack
Every time a custom element is defined, a new custom element definition is added to the registry. This definition owns a construction stack which contains a list of elements or _already constructed markers_. The construction stack allows the magic of upgrades to happen. When upgrading an element we push the element onto the construction stack of the definition and then call `Construct` on the definition's constructor. This will call the user defined constructor which in turn calls `super()` which will call the `HTMLConstructor`. In the `HTMLConstructor` we check the definition's constructor stack if it contains an element, the element replaced by the _already constructed marker_ and the element is returned. From here, the user defined constructor is now run over the existing element.

## `is` Attribute
One oddity when looking at custom elements the first time is the `is` attribute. The attribute is when defining and creating Customized Built-In elements. These elements extend from elements other than `HTMLElement` like `HTMLButtonElement` or `HTMLInputElement`. [Most browsers][caniuse custom elements] do not even currently support Customized Built-In elements; however, I did want to go over why it has to be done this way.

Going back to the beginning of the post, there are cases where custom elements are added to a document prior to them actually being defined. If we could just use the local name without the `is` attribute for the Customized Built-In Elements, we would not know the proper element interface to pick for these undefined custom elements. If we pick wrong, for example, we are unable to go from `HTMLElement` to `HTMLButtonElement`. This is why we must use the `is` attribute because we must know the proper element interface at any time.

While the `is` attribute is not very ergonomic, there does not appear to be any way to get around it other than just not supporting/using it. One way to avoid this is to always use Autonomous Custom Elements and have them contain the custom behavior button, input, etc...

While the upgrade algorithm does seem a bit odd at first, it is actually a really clever way of solving the issue.

[HTMLConstructor]:https://html.spec.whatwg.org/multipage/dom.html#htmlconstructor
[caniuse custom elements]:http://caniuse.com/#feat=custom-elementsv1

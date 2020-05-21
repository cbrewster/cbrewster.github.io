@Meta
title = "Custom Element Upgrades"
slug = "custom-element-upgrades"
date = "2017-06-08"
author = "Connor"

@Content
I am implementing the Custom Element v1 spec in Servo this summer. While preparing for the work I have read over the spec a few times; however, it was not until recently that the upgrade algorithm began to make sense. The point of the algorithm is to handle custom elements that have been added to a document before they have been defined.

Example:
```html
<custom-element></custom-element>
<script>
customElements.define("custom-elements", class CustomElement extends HTMLElement { constructor() { super(); } })
</script>
```

In this example, `custom-element` is added to the document before it has been defined, which means its constructor will not be run and thus will not behave as expected. To solve this problem, after a custom element definition is added to a registry, all the elements are traversed and any elements that match the local name of the new definition are upgraded.

This seems odd at first because the custom element starts out as a HTMLElement, but should end up as a CustomElement after the upgrade algorithm. This all needs to be done without removing and re-adding a new element. We need to actually run the constructor against the already existing element.

## HTMLConstructor
First, we need to take a detour and look at the `[HTMLConstructor]` WebIDL extended attribute. Before custom elements, you couldn't use a class to extend native elements like `HTMLElement` or `HTMLButtonElement`.

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

If you try this code you will probably get an error like `Illegal constructor`. This is because the native element doesn't expose a constructor (or at least not until custom elements came around). `HTMLConstructor` is a new constructor defined in the custom element spec that adds a constructor for most native HTML elements; however, it is not like a normal constructor.

###### Note: The call to the `HTMLConstructor` happens when `super()` is called in the subclass constructor.

The above example will only succeed if `CustomElement` is added to the custom element registry. This is due to the behavior of the `HTMLConstructor` defined in the spec. I won't go over that too much in detail (see [the spec][HTMLConstructor] for more), but the important thing to note is that with the custom constructor, we have control over what `super()` actually returns. We can return a newly created element or an element that _already_ exists. The latter is the important part when it comes to custom element upgrades.

## Construction Stack
Every time a custom element is defined, a new custom element definition is added to the registry. This definition owns a construction stack which contains a list of elements or _already constructed markers_. The construction stack allows the magic of upgrades to happen. When upgrading an element we push the element onto the construction stack of the definition and then call `Construct` on the definition's constructor. This will call the user defined constructor, which in turn calls `super()`, which will then call the `HTMLConstructor`. When the `HTMLConstructor` is invoked by `super()`, it retrieves the proper custom element definition. The constructor checks the last element of the _construction stack_. If an entry exists and it is not the _already constructed marker_, the entry is replace with the marker and the element from the entry is returned. From here, the user-defined constructor is now run on top of the existing element.

## The `is` Attribute
One oddity when looking at custom elements for the first time is the `is` attribute. The attribute is used when defining and creating Customized Built-In elements. These elements extend from elements other than `HTMLElement`, such as `HTMLButtonElement` or `HTMLInputElement`. [Most browsers][caniuse custom elements] do not even currently support Customized Built-In elements.

The attribute exists because custom elements can be added to a document prior to being defined. If the element was defined with the local name without the `is` attribute for the Customized Built-In Elements, the proper element interface would be unknown until the custom element has been defined. If the wrong element interface is picked, the element cannot change from one element interface to another (e.g. `HTMLElement` to `HTMLButtonElement`). For this reason, the `is` attribute along with the local name of the extended element must be used so that the proper element interface for a given local name is always known.

While the `is` attribute is not very ergonomic, there does not appear to be any way to get around it other than just not supporting/using it. One way to avoid this is to use Autonomous Custom Elements and wrap the element that needs to have custom behavior.

While the upgrade algorithm does seem a bit odd at first, it ends up actually being a really clever way of solving the issue.

[HTMLConstructor]:https://html.spec.whatwg.org/multipage/dom.html#htmlconstructor
[caniuse custom elements]:http://caniuse.com/#feat=custom-elementsv1

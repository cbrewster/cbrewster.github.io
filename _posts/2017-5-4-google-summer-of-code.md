---
layout: post
title: Google Summer of Code 2017
---

This year I have been accepted for Google Summer of Code and I will be implementing the Custom Element specification in [Servo](https://servo.org).

## What are Custom Elements?
Custom Elements are a new addition to the html and dom specifications which allow javascript users to define their own elements that can be used via html markup or javascript. Custom Elements allow programmers to write resuable and modular components without the need of larger frameworks.

## Who uses Custom Elements?
New web frameworks are now being built around Custom Elements such as [Polymer](https://www.polymer-project.org) and [A-Frame](https://aframe.io). Both of these frameworks allow users to easily define their own custom elements and come with many prebuilt components that developers can use.

[Polymer](https://www.polymer-project.org) builds on top of Custom Elements and allows users to more easily write custom components. There are many premade custom elements that are built with Polymer. For example, the Paper elements are premade Material Design elements.

[A-Frame](https://aframe.io) is a new WebVR technology supported by Mozilla. With A-Frame, developers can easily create WebVR experienced by using premade components or by creating their own. This has simplified the amount of work required to create a WebVR experience.

## A Piece of the Puzzle
While implementing Custom Elements will pave the road towards supporting these frameworks without polyfills, there are still other features that these frameworks rely on. Custom Elements are just a part of Web Components.

Web Components consist of the following 4 features:
 * Templates
 * Custom Elements
 * Shadow DOM
 * Imports

While I will only be implementing Custom Elements, I hope that it will kick off the movement to support all the needed features for Web Components.

A log of my progress will be on my [gsoc2017 repo](https://github.com/cbrewster/gsoc2017) on GitHub.

There will be more posts to come concerning implementing Custom Elements in Servo!

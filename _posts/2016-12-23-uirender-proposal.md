---
layout: post
title: UIrender Proposal
---

## Preface
I have been a [Servo](https://www.servo.org/) developer for the past few months. While working on
Servo, I have been amazed at the different libraries that have been created as a byproduct of Servo.
One of these libraries is [webrender](https://github.com/servo/webrender). Webrender takes the 
output of the Servo layout engine and utilizes the gpu to render web pages at blazing fast speeds.
While webrender was made for Servo, it can be used independently. This gave me the idea of a cross
platform UI library built using webrender and [Rust](https://www.rust-lang.org).

The Rust ecosystem does not have very many options for a UI library other than using bindings to C
or C++ UI libraries. Not only that, most non-native UI libraries either look horrible or run slowly.
Recently there has been a move towards using web technology for desktop applications. Using web
technology for a desktop application is nice because it is easy to build cross-platform applications
with easy to configure user interfaces. Unfortunately, these applications bear extra over head as
an entire web browser engine and javascript engine are packages with every application. These
applications also can suffer due to low performance and its negative effects on battery life.

I believe we can do better than the available options. UIrender aims to be a new UI framework that
is fast, easy to configure, and cross-platform. Webrender allows Uirender to render interfaces
similar to that of what html and css can construct without the overhead of an entire web browser
engine.

## Goals
UIrender aims to be:
 * **Fast** - UIrender should render at a high frame rate and should never be the performance 
 bottleneck of the application.

 * **Cross-Platform** - There should be very little to none platform specific code. The application 
 should render consistently across all supported platforms: Linux, macOS, and Windows.

 * **Easy to Configure** - The API should be easy to use and the UI should be easy to customize. The 
 layout should be stored in a common format such as XML and it should be easy to build reusable UI
 components.

 * **Rust-first** - The API should be built using idiomatic Rust and should prioritize compisition
 over inheritance. This means the framework will deviate from the typical UI framework.
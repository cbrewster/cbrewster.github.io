---
layout: post
title: UIrender Architecture
---

## Architecture
UIrender is split into different modules:
 1. **Layout Builder** - This module will take the layout XML and parse it into Rust structures.

 2. **Layout Engine** - This module will take the Rust structures from the Layout Builder and
 calculate the position, size, and styles for every element and generate a `DisplayList` that is 
 sent to webrender.

 3. **Event Loop** - The event loop runs in its own thread and it handles interactive UI elements.
 The event loop handles communication between the application and webrender.
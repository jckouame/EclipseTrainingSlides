% title: Eclipse Plug-in Development
% title_class:                  #empty, largeblend[123] or fullblend
% subtitle: Extending the CDT Debugger
% subtitle_class:
% title_slide_class: dark
% title_slide_image:
% author: Marc Khouzam
% author: Marc-Andre Laperle
% thankyou_blend: largeblend3       #largeblend[123] or fullblend
% thankyou_details:
% mail1: marc.khouzam@ericsson.com
% mail2: marc-andre.laperle@ericsson.com
% phone:
% sms:
% lync:
% footer: Ericsson Internal
% footer: Rev PA1
% footer: 2015-12-15
% logoslide: false      # Show a logo slide as the first slide. Also affected by % animate
% useBuilds: true
% animate: true         #animate logoslide (chrome only)
% aspect_ratio: 4:3     #16:9, 16:10 or 4:3

---
title: Agenda
subtitle: Subtitle Placeholder
class: dark

- allo

++Notes++
- pressing 'f' toggle fullscreen
- pressing 'w' toggles widescreen
- pressing 'o' toggles overview mode
- pressing 'p' toggles speaker notes (if any)
- pressing 'h' highlights code snippets
- pressing 'b' toggles blank screen
- pressing 'c' toggles canvas to draw on the slide with the mouse
    - Pressing 'shift' draws arrow
    - Pressing 'alt' draws rectangle
    - Pressing 'shift+alt' draws ellipse
- pressing 'ESC' toggles of these goodies
++++

---
title: Intro
subtitle: Subtitle Placeholder
class: dark


This is a markdown

use askterisk for *italic* and double for **bold**.  You see?
Great for `writing code` and such

- a
- b
    * (http://google.com)
- c

![alt text](marc.png)
```
if (i = 8) {
  printf("allo%d\n", i);
}
```
---
title: Intro
content_class: smaller

Test  |Column 1   |Column2    |Column 3   |Column 4         | mark
------|-----------|-----------|-----------|----------       |---
Row 1 |placeholder|placeholder|placeholder|placeholder      |     
Row 2 |placeholder|placeholder|placeholder|placeholder     |
Row 3 |placeholder|placeholder|placeholder|placeholder     | johnny
Row 4 |placeholder|placeholder|placeholder|placeholder     |
Row 5 |placeholder|placeholder|placeholder|placeholder     |

---
title: Plugin development

- Creating a new plugin
- Adding actions to the UI
- Using extension points
- Creating a new view
- Using Listeners
- Creating a new Launch
- Extending the CDT Debug launch
- Using preferences
- The Adapter pattern
 
---
title: Getting to know DSF
build_lists: true

- The DSF Executor
- The Services model
- Asynchronous programming with DSF
- Using DSF Events
 
---
title: Getting to know DSF-GDB

- The DSF-GDB services
- Adding a new service
- Extending a service
- Sending commands to GDB
- Creating a new command
- The CommandCache
- Extending the FinalLaunchSequence
- The Data Model Contexts and their hierarchy
- GDB/MI events
- Supporting All-Stop and Non-Stop
- GDB version handling
- The CommandFactory
- IDebugContextListener
- Workbench selection

---
title: Missing features

- What is missing to make baseband developers more productive with eclipse
                     2 hours session with invited end-users

---
title: Latest implementations:

- The new CLI towards GDB
- Advanced C/C++ Debugging features
- The Visualizer
- Extending the CDT editor
- Trace Compass basics



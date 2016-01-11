% title: Eclipse Plug-in Development
% title_class:                  #empty, largeblend[123] or fullblend
% subtitle: Extending the CDT Debugger
% subtitle_class:
% title_slide_class:
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
class: nobackground
build_lists: false
content_class:

- DAY 1: Plugin development
    - Module 1:
    - Module 2:
- DAY 2: Getting to know DSF and DSF-GDB
    - Module 3:
    - Module 4:
- DAY 3: Optimizing performance
    - Module 5:
    - Module 6:

---
title: Agenda (cont)
build_lists: false

- DAY 4: Keeping views synchronized
    - Module 7:
    - Module 8:
- DAY 5: More debugging concepts and intro to Trace Compass
    - Module 9:
    - Module 10:
    <br><br>
##<center>Let's get started!</center>
[//]: (This is a comment)

---
title: DAY 1
subtitle: Plugin development

---
title: Module 1

- intro
- very basic explanation of eclipse architecture, use of SWT, single UI thread
- create one plugin for the project
- create a view
- add SWT button that has a state (e.g., toggle or checkbox) to the view content with buttonPressed listener to print something in the view
- replace SWT button with view toolbar button using command framework
- add same action to view menu and context menu

---
title: Module 2

- restarting eclipse -> state of button is forgotten but the view layout and such is remembered
- using preferences to save state; using preference scope
- reference: how to add a UI preference
- Job and UIJob, Display.asyncExec and Display.syncExec by adding a blinking thingy, like a counting label, that uses a Job + syncExec or asyncExec
- the use of final and final arrays in async programming
- Progress Monitor to allow cancelling the blinking job and showing progress

---
title: DAY 2
subtitle: Getting to know DSF and DSF-GDB

---
title: Module 3

- What is DSF?
- The Services model
- ViewModel and DataModel
- Calling into a DSF service using DsfServiceTracker and asynchronous programming (not on executor, not using -ea)
- access to e.g. IStack from the view to do something cool
- Mention how to find services i.e. F4 on IDsfService
- enable assertions (-ea) and try again, then explain DSFExecutor
- Using suspend event to log stack trace on breakpoint hit
- bp hit vs step completed event

---
title: Module 4

- What is DSF-GDB?
- write a new service that gets published directly in the new plugin which will return the time of day
- add a method to that service that will send a command to GDB to count the number of args of a frame using createMIStackListArguments()
- use new service to display the number of args for each stack frame in their view
- extend a DSF-GDB service e.g. IStack with getNumberLocals(IFrameDMContext, DataRequestMonitor<Integer>)
- explain services factory and using criteria to choose a service 'version' e.g., we do that based on GDB version in DSF-GDB
    - create new factory receiving the launchConfig and a param from to choose a service 'version' e.g. depending on chosen simulator
- new delegate that override the servicesFactory.  We should provide the new delegate ourselves with tabs and all

---
title: DAY 3
subtitle: Optimizing performance

---
title: Module 5 Should we skip this?

- show that the new -stack-list-arguments is being sent to GDB many times but nothing has changed
- Adding CommandCache
- Clearing commandCache by listening to resumed event

---
title: Module 6

- Meeting with end-users to discuss requirements
- Discussion with team about requirements just received

---
title: DAY 4
subtitle: Keeping views synchronized

---
title: Module 7

- IDebugContextChangedListener
- add support for changing selection in DV and our view updating properly using DMContext hierarchy to choose proper info to display
- exercise about GDB mi events ...

---
title: Module 8

- extending the FinalLaunchSequence ...

---
title: DAY 5
subtitle: More debugging concepts and intro to Trace Compass

---
title: Module 9

- If there is interest, show Advanced C/C++ debugging presentation, visualizer, new CLI
- Explain all-stop vs non-stop and give a demo
- explain multi-process in preparation for multi-arch support
- Intro to Trace Compass usage
- Intro to Trace Compass basic concepts and classes

---
title: Module 10

- Extending the CDT editor
- writing a Codan check for code style of counting the number of lines in a method
    
++Notes++
- If we have an extra afternoon we can do an advanced exercise:
    - handle both all-stop and non-stop in their new view by logging the stack trace for every thread of the process that stopped in all-stop mode
- Topics to mention but not very long:
    - CommandFactory should called but is rarely change. Example of DSF-GDB
    - adapter pattern
++++


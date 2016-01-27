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

[//]: (Start presentation in Presenter mode by adding ?presentme=true to URL)
[//]: (Make sure pop-ups are allowed, then keep pop-up on my own screen)
[//]: (then press 'p' for presenter mode for my screen while showing)
[//]: (main browser window on projector)
[//]: (turn off by adding ?presentme=false to URL)
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
title: Course Projects
build_lists: true

## Debug Frame Spy
- New view displaying a history of every `function:line` at which the debugger stopped. 
## Enable GDB verbosity
- Extend debugger to enable verbosity in GDB when starting a debug session
## Code Complexity Checker
- Codan code checker warning when a method is too long
- Codan code checker warning of incompatible return type of overriding method

---
title: Debug Frame Spy
    1. Show list of function name and line number of each location program was interrupted
    1. Show time of interrupt for each entry
    1. Show the number of arguments of the function for each entry
    1. Show the number of locals in that function for each entry
    1. For multi-threaded programs, show the list only for the currently selected thread.

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

[//]: (STARTED CLEANUP HERE)
---
title: Module 3

- Eclipse Debug Platform
- What is DSF?
- DSF concepts applied

---
title: Eclipse Debug Platform

- Eclipse provides a foundation for Debugging
    - Debug perspective
    - Debug views
    - Debug Actions, Toolbar, Menus
    - Debug Launching

++Notes++
- Debug perspective manual or automatic selection
- *Windows->Show View->Other... and expand Debug folder*
- Each view has toolbar actions, context-menu, view-menu
- Launch configuration types, launch configurations
++++

---
title: What is DSF?

- Overview
- View Model and Data Model
- Services
- Data Model contexts
- DSF Executor thread
- DSF Session
- Asynchronous (callback) programming

---
title: DSF Overview

- API for integrating debuggers in Eclipse
- Also designed for efficiency (slow or remote targets)
- Figure shows typical debugger integration using DSF

<center><img src=img/DSFOverview.png></center>

---
title: View Model

- View Model provides layer of abstraction for views
    - *User-presentable* structure of the data
- View Model allows to easily modify presentation e.g.,
    - Hide running threads
    - Limit number of stack frames
    - Only show processes if there is more than one

++Notes++
- Show feature that hides running threads
- Show feature that limits number of stack frames
- In practice, we don't change the view model very often.  So this course will not cover it.
++++
---
title: Data Model

- Data Model deals directly with the backend debugger
    - *Natural* or *backend* structure of the data
    - Independent of presentation to user
    - Provides building blocks for the view model
    - Uses common debugger concepts
        - Execution elements (e.g., processes, threads)
        - Formatted values (e.g., variables, registers)
        - etc

++Notes++
- Mention frontend vs backend terminology
++++
---
title: DSF Services

- DSF provides a service API to access the Data Model
- Built on top of OSGi (as Eclipse is)
- Services are entities managing logical subsets of the data model
- Services are used to request information or to perform actions
- For example, the IRunControl service:
    - Provides list of execution elements (e.g., threads, processes)
    - Provides details about such elements (e.g., name, state)
    - Supports step, resume, interrupt, etc
- Other services: IMemory, IRegisters, IExpressions, IDisassembly...
- All services extend *IDsfService* (press *F4* on *IDsfService*)

---
title: Data Model Contexts

- IDMContext class is a 'pointer' to any type of backend data
    - IExecutionDMContext - thread, process, group
    - IFrameDMContext - stack frames
    - IBreakpointDMContext - breakpoint, tracepoint, dprintf
    - All contexts extend *IDMContext* (use *F4*)
- Contexts are hierachical
    - *process* -> *thread* -> *frame* -> *expression*
    - *DMContexts.getAncestorOfType()*
- Contexts are used to retrieve data from services

---
title: DSF Executor thread

- Accessing data from different threads requires synchronization
- DSF uses a single-threaded executor to avoid synchronization
<center>
<img src=img/synchronization_1.png>
<img src=img/synchronization_2.png>
</center>

---
title: DSF Session

- Instances of DSF services are grouped into a DSF session
- There can be multiple sessions running at the same time
- The session provides the DSF Executor (*DsfSession#getExecutor()*)
- A session handles sending events to registered listeners

---
title:  Asynchronous (callback) programming

- Most DSF APIs return void but indicate completion in a callback
- *RequestMonitor* is the main callback class
    - Remember to call *done()* when real work is finished
    - This calls: *handleCompleted()*, *handleSuccess()*, *handleError()*
- *DataRequestMonitor* to "return" a value
    - *getData()* to get that value

---
title: DSF concepts review

1. APIs to integrate a debugger 'more easily' e.g., GDB
1. View Model for presentation layer
1. Data Model to communicate with backend (GDB)
1. Services API to access Data
1. No synchronization: DSF Executor **must** be used to access Data
1. Services for one backend are grouped in a Session
1. Heavy use of asynchronous programming for responsiveness

---
title: DSF practical review

1. Services extend *IDsfService*
2. Contexts extend *IDMContext*
3. Context hierarchy searched with *DMContexts*
4. Executor can be found with *DsfSession#getExecutor()*
5. *RequestMonitor* and *DataRequestMonitor* for callbacks

---
title: DSF Exercise 

- FrameSpy to periodically print "method:line" for current frame 
    - Reset branch to commit starting with
        - **EX1_START** or **EX1_ADVANCED**
    - To test, make sure you launch a C/C++ Debug session first
    - **Go!**
++Notes++
- Show how to launch a C/C++ debug session (use non-stop first)
++++
---
title: DSF Exercise review

- Finding the DSF session using debug context
    - Debug View and Debug Context
    - Adapter pattern
- Calling an existing DSF service
    - Using a DsfServicesTracker for the DSF session
- Call the asynchronous IStack.getTopFrame()
    - Using a new DataRequestMonitor
    - Calling getData() in handledSuccess()
---
title: DSF Exercise review (2)

- IDMContext vs IDMData
    - call IStack.getFrameData()
    - Using a new DataRequestMonitor
    - Calling getData() in handledSuccess()
    - Then finally display "method:line"

---
title: DSF Exercise follow-up

- What if you select the process element?
    - The top frame of which thread should we use?
- For now, just handle the error
    - Reset to **EX1_ANSWERS** if you need
    - Override *handleError()*
    - **Go!**

---
title: DSF Exercise follow-up (2)

- Enable assertions in development eclipse for test eclipse
    - In launch configuration, *Arguments* tab, *VM arguments*
    - Add *-ea*
    - Make sure you have a breakpoint for *AssertionError*
    - Re-launch and try
    - **Go!**

---
title: DSF Exercise follow-up (3)

- Did you use the DSF Executor?
    - Which calls on the Executor, which are not?
- Wrap first call to DSF service in Executor
    - Call *submit()* of the Executor
    - Pass a *DsfRunnable()* whose *run()* does the work
    - **Go!**

---
title: Which DSF concepts did we exercise?
build_lists: true

1. APIs to integrate a debugger 'more easily' e.g., GDB
1. View Model for presentation layer
1. Data Model to communicate with backend (GDB)
1. Services API to access Data
1. No synchronization: DSF Executor **must** be used to access Data
1. Services for one backend are grouped in a Session
1. Heavy use of asynchronous programming for responsiveness

---
title: DSF Events

- DSF uses events to notify listeners of different things e.g.,
    - Thread/Process started/exited
    - Thread/Process suspend/resumed
    - Breakpoint added/updated/removed
    - etc

---
title: DSF Events (2)

- Events are how the Data Model tells the View Model of changes
    - e.g., Thread stops => Update Debug View
    - This is an advanced topic not covered in this course
- Events also notify services of other services' changes
    - e.g., Clearing caches when execution resumes
---
title: DSF Events details

- Most events implement *IDMEvent* which provides an *IDMContext*
    - e.g., When thread suspends, event specifies which thread
- Event types usually found in the different service interfaces e.g.,
    - *IRunControl*:
        - *ISuspendedDMEvent*, *IContainerSuspendedDMEvent*
        - *IResumedDMEvent*, *IContainerResumedDMEvent*
- Not all services trigger events
    - *IStack* has not events
---
title: Sending DSF Events

- To send an event a service calls *DsfSession#dispatchEvent()*
---
title: Receiving DSF Events

- To receive a DSF events a client must:
    - Declare a **public** method of any name
    - Method takes the event of interest as a parameter
    - Annotate method with *@DsfServiceEventHandler*
    - Register with the DSF Session using *DsfSession#addServiceEventListener()*
    - Registration must be done on the Executor
- The method is called on the DSF Executor


++Notes++
- *dispatchEvent()* calls event listeners in a **separate** Runnable on the Executor.
    - This allows sender to finish its work before events are received by the listeners.
    - Event listener methods always called in the Executor thread.
++++
---
title: Receiving event example

- The following method from *SomeClass* will be called for every suspended event

~~~~
    @DsfServiceEventHandler
    public void anyName(ISuspendedDMEvent e) {
        System.out.println("Received " + e.toString());
    }
~~~~

- as long as we register the class with the session

~~~~
    getSession().addServiceEventListener(SomeClass.this, null);
~~~~

- Remember that registration must be done on Executor
---
title: Help with the DSF Executor

- *@ConfinedToDsfExecutor* vs *@ThreadSafe*

---
[//]: (ENDED CLEANUP HERE)
=> replace polling with ISuspendedDMEvent
---
title: DsfSession class

- A session instance is started and ended using:
    - *startSession()*
    - *endSession()*
- DsfSession provides access to all running sessions:
    - *getActiveSessions()*
    - *getSession(id)*
- DsfSession notifies registered listeners of start/end of all sessions
    - *addSessionStartedListener()*, *removeSessionStartedListener()*
    - *addSessionEndedListener()*, *removeSessionEndedListener()*

---
title: DsfSession class (2)

- Allows access to Services through *DsfServicesTracker*
- A session instance forwards debug event to registered listeners
    - *addServiceEventListener()*, *removeServiceEventListener()*
    - *dispatchEvent()* is called by services that want to send event
- Allows to register adapters for all data model contexts
    - *registerModelAdapter()*, *unregisterModelAdapter()*, *getModelAdapter()*
---
title: Module 3 (old)

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


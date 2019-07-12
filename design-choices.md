# DOM overlay design choices


# Overview

There are multiple different interpretations of what it means to show 2D web content in XR, with several orthogonal design choices that produce a multidimensional problem space. These design choices are intended to be fairly general and cover approaches beyond the initial [DOM Overlay Explainer](https://github.com/immersive-web/dom-overlays/blob/master/explainer.md).


# Resources

TPAC 2018 "Presenting 2D Web Content in XR" [slides](https://www.w3.org/2018/10/iwwg-tpac-slides/2d.pdf), [meeting minutes](https://www.w3.org/2018/10/26-immersive-web-minutes.html). This contained a classification of options, included here for reference:


    1. Do nothing


    2. Solve for immersive mode on handheld devices only


    3. Option 2 + a UA-managed workaround for immersive mode on headset devices


    4. Option 2 + quad layers. 


    5. Revisit feasibility of DOM to texture 

The [DOM Overlay Explainer](https://github.com/immersive-web/dom-overlays/blob/master/explainer.md) in the immersive-web repo was based around option 3. (See also [proposal #50](https://github.com/immersive-web/proposals/issues/50), [issue #400](https://github.com/immersive-web/webxr/issues/400) for earlier discussions.)


# Design choices for DOM overlays

## Output compositing

How does DOM content get combined with app-drawn WebGL content and other content such as AR camera images?



*   2D transparent fullscreen overlay quad (smartphone AR)
*   UA-managed single quad (head-locked or floating quad for headset VR/AR)
*   App-managed single quad
*   App-managed multiple quads
*   Application compositing via DOM-to-texture from WebGL


## DOM input event source

To what extent does the UA support generating DOM events for user actions?

A core advantage of WebXR's input events is that they support events that count as user initiated actions for security purposes. An interactive DOM overlay could also provide input events that count as user actions if managed by the user agent, but not for app-generated synthetic events. Synthetic events must not be sent to cross-origin content.



*   No special support
    *   app needs to do its own hit testing by intersecting a touch point or controller pointing ray with the DOM quad to determine the hit location, and dispatch a synthetic DOM event to the appropriate element.
    *   Example for programatically toggling a checkbox: `document.getElementById('check').dispatchEvent(new MouseEvent('click', {view: window, bubbles: true, cancelable: true}));`
*   Synthetic events based on app-generated event locations
    *   Similar to "no special support", but with added UA support to find intersection points and appropriate elements. For security purposes, these events would not count as user initiated.
*   Automatic event generation via UA, i.e. raycasting + controller click
    *   UA-generated events can count as user-initiated actions for security purposes.


## Integration between DOM and WebXR input events

User actions such as screen touches or controller clicks could potentially be interpreted as XR world interactions or as DOM events. How should these be distinguished, and does the UA provide integration to disambiguate them?



*   WebXR input only (app needs to synthesize DOM events)
*   Duplicated events (a controller click or screen touch generates both DOM event and WebXR input event, app needs to disambiguate)
*   DOM input takes precedence (inputs intersecting a DOM quad anywhere don't generate WebXR input)
*   Smart deduplication (inputs that intersect a DOM quad generate WebXR input if they aren't captured by a DOM event handler, or disambiguated based on explicitly or implicitly defined active regions of the DOM quad)

This choice may also depealternatively the application could generate its own WebXR-style input events for screen touches on the fullscreen element's background. 


## Input element types

Interactive DOM elements can cause challenges for integration in immersive environments. Handling basic input for elements fully drawn inside the content quad is comparatively straightforward, but things get trickier for more complex UI elements, especially if the UA normally uses native platform UI elements separate from the DOM to implement some of them. The following list is intended to be roughly in order of difficulty of implementation mainly based on my experience with Chrome on Android, this may be different for other implementations or platforms.. 



*   None (DOM display only, app needs to draw/animate its own UI elements)
*   Simple clicks (buttons, checkboxes)
*   Complex actions (sliders, drag&drop, scrolling)
*   Advanced input (`<select>` pop-ups, "file" dialogs, date picker, and other UI elements that require rendering additional layers)
*   Text input, _contentEditable_ DOM text
*   Assistive input ("[explore by touch](https://support.google.com/accessibility/android/answer/6006598?hl=en)" or similar)
*   Secure browser UI (Payments API, permission prompts) - note that allowing these outside a trusted UI is problematic, so it may be necessary to explicitly disable such functionality during immersive sessions if a trusted UI is not supported.


## DOM output displays

Is DOM content potentially visible in multiple locations at once?



*   Single output destination (all-in-one headset, smartphone AR)
*   Multiple output destinations active simultaneously (desktop VR headsets)

Rendering web content from a single DOM tree to multiple locations has been strongly discouraged by the Chrome team because it violates deep assumptions in DOM rendering. It’s reasonable to assume this also applies to other implementations. Thus, the API design would likely need to find a way to avoid it.


## DOM source

What is the source of the DOM elements shown in a DOM overlay? Are they part of the main document, or does each DOM overlay use a separate _document_ or other rendering context? This overlaps with the "multiple output" issue, and implementation limitations may make it impossible to split rendering of a single DOM tree into multiple distinct output destinations.



*   same rendering context (similar to Fullscreen API)
*   separate rendering contexts (separate document(s))

Using a single rendering context is convenient for developers, for example it allows using the browser's Inspector for viewing elements and styles. Separate documents are harder to work with, for example each document would need its own separate CSS styles.


## Cross-origin content

How do DOM overlays handle cross-origin content such as embedded iframes?



*   blocked completely
*   opt-in via XR-specific CSP or similar mechanism
*   allowed automatically according to existing 2D rules


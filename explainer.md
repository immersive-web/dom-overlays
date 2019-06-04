# **DOM Overlay Explainer**

## Introduction

The current WebXR specification does not include support for inline AR content, the only available AR session mode is `immersive-ar`, and this is intended to exclusively show the real world or camera image combined with application-drawn WebGL content.

It would be very useful to support a hybrid mode where the scene can be augmented (no pun intended) with an overlayed 2D user interface based on HTML text and DOM elements such as buttons. This is convenient for developers, but also helps with features such as internationalization and accessibility that would be difficult to handle correctly in a purely WebGL-based UI. (Accessibility is beyond the scope of this initial proposal, but the goal is to make it possible for user agents to extend existing support to this new API.)

Platforms that support multiple layers at the device compositor level could use that to render the DOM overlay at higher quality than WebGL-drawn content. (See the [layers proposal](https://github.com/immersive-web/layers#why-layers-what-are-the-benefits) for more background on this.) 

An AR experience may want to show messages to the user that are not tied to the 3D scene, for example instructions how to proceed ("Locate a flat surface that is at least 3’x3’").

The application may be using an AR view as part of a more complex experience, for example a model viewer that allows customizing an item's color or other properties.

A smartphone AR game could show onscreen buttons for game controls that are not tied to scene objects. This DOM overlay would also work on a head-mounted AR display using a controller ray for activating buttons, though an application may choose to use more direct controller inputs instead of the DOM overlay where available.  


### Goals

Support showing DOM content during immersive AR/VR sessions to cover common use cases. This includes displaying explanatory text or HUD elements alongside an AR scene, and providing interactive elements such as buttons or sliders that affect the scene.


### Non-goals

This API is not intended to support placing DOM elements in the 3D scene. It does not address use cases such as placing labels directly on 3D objects or world features. 

Although an overlay could include elements that are styled with 3D CSS transforms or positioned in relation to scene content, with the application using DOM position attributes and similar techniques, the DOM Overlay API does not provide any direct support for such positioning. In addition, this is discouraged because it is unlikely to work across devices and form factors, especially between smartphones and headsets.


### Overlay Overview/Feature Summary

The DOM overlay consists of a single rectangular DOM element and its children. It is composited on top of the immersive content by the user agent. The application can style the alpha color channel for elements to leave parts of the overlay transparent, but there is no depth-based occlusion. The environment view and 3D elements in the scene are always covered by non-transparent DOM elements, using alpha or additive blending for partially transparent DOM elements as appropriate for the display technology.

The exact appearance of the DOM overlay is platform dependent, though there will be requirements to ensure developers know what to expect and to ensure interoperability. The user agent should pick a display style that shows the content in a way that is comfortably visible and accessible for interactive input. The "Display modes" section below gives examples.

The DOM overlay needs to be capable of accepting user input and dispatching appropriate DOM events such as click actions on elements. Examples include screen touch events for smartphones and controller-based ray input for head-mounted displays.

The DOM overlay is restricted to a single rectangle at a fixed Z depth chosen by the user agent. There is no support for placing individual DOM elements at specific distances, or for showing different images to the left and right eyes for stereoscopic effects. This simplification is intended to make it easier to implement - the DOM content is conceptually rendered as a simple rectangular block of pixels that is then composed into a combined view.

The application does not get low-level control over placement of the DOM overlay; the placement of the overlay is intentionally left up to the user agent. The goal of the API is to enable common basic UI functionality across a variety of form factors, and it would be counterproductive for applications to make assumptions or attempt to fine-tune configurations in ways that only work on a specific class of devices.


## Display modes

The specific way that the DOM overlay content is displayed depends on the output device and user agent. In general, implementations have more freedom as the level of immersion increases, though there are requirements to help ensure consistent presentation that developers can rely on.

On a smartphone, for example, in immersive-ar mode, the overlay must cover the entire device screen as used for the camera and scene view, excluding system navigation areas if those are occluded or not touchable.

On a head-mounted AR display with a moderately-sized rectangular field of view (FoV), the overlay can be a head-locked UI that fills the renderable viewport. (Seeing the real world around the overlay helps reduce discomfort that could otherwise be caused by head-locked content, and the FoV is small enough to keep eye movements needed to read content in a comfortable range.)

On a large-FoV VR headset, the overlay may appear as a rectangle floating in space that's kept in front of the user, but isn't necessarily strictly head-locked (in order to improve comfort). This rectangle may be smaller than the maximum FoV if necessary to ensure that the corners and edges remain easily visible. (The visible area can depend on the user's eye position and eye-to-lens distance, and extreme angles may be blurry for some headsets.)

The display technology used affects how the overlay is composited. Applications should check the existing <code>[XRSession.XREnvironmentBlendMode](https://immersive-web.github.io/webxr/#xrsession-interface)</code> attribute. A see-through AR headset typically uses the <code>"additive"</code> blend mode, in this case black pixels appear transparent. If the session uses <code>"opaque"</code> or <code>"alpha-blend"</code> mode, the alpha channel is used to control visibility of DOM overlay contents.


## Input

Low-level inputs that intersect the DOM overlay rectangle (including transparent areas) will be forwarded to the overlay's DOM element for processing according to the usual DOM event propagation model, using event x/y coordinates mapped to the DOM overlay rectangle. For example, screen touch or ray inputs are converted to DOM input events including `"click"` events (required) and optionally also mousedown/mousemove/mouseup events if supported by the implementation.

If a WebXR application uses a DOM overlay in conjunction with XR input, it is possible that a user action could be interpreted as both an interaction with a DOM element and a 3D input to the XR application, for example if the user touches an onscreen button, or uses a controller's primary button while pointing at the DOM overlay.

Thus, either the user agent or application will need to ensure that such input is only interpreted in the desired way. Ideally, the user agent would provide this functionality, though we would need to define how it does so. There may be additional motivations and complexity related to security and privacy.

WebXR's [input events](https://github.com/immersive-web/webxr/blob/master/input-explainer.md#input-events) (`"selectstart"`, `"selectend"`, and `"select"`) potentially duplicate DOM events when the user is interacting with a part of the scene covered by the DOM overlay, including transparent areas. It would be useful if the user agent could suppress WebXR input events if they were already handled by the application in a DOM event handler. For example, calling the `"click"` event's `stopPropagation()` or `preventDefault()` method should suppress the WebXR `"select"` event. This may require the user agent to special-case WebXR input events that interact with the DOM overlay, potentially incurring a small processing delay. `"selectstart"`/`"selectend"` events are potentially asymmetric since one of them may happen inside the DOM overlay while the other one is outside.

WebXR also supports non-event-based input. This includes controller poses, button/axis states, and transient XR input sources such as [Screen-based input](https://github.com/immersive-web/webxr/blob/master/input-explainer.md#screen) that creates 3D targeting rays based on 2D screen touch points. These inputs are not affected by DOM overlays and continue to be processed as usual. Applications are responsible for deduplicating these non-event inputs if they overlap with DOM events, though it's recommended to avoid UI designs that depend on this. There are many ambiguous corner cases, for example pointer movement that starts on a DOM element and ends outside it.

If using a DOM overlay in a headset, the implementation should behave the same as for smartphones whenever the primary XR controller's targeting ray intersects the displayed DOM overlay. In other words, generate `"click"` events when the primary trigger is pressed, subject to whatever additional restrictions are specified, such as those for disambiguation.


## References & acknowledgements

This proposal is based on extended discussions in the `#immersive-web` community, and builds on proposals and suggestions by @ddorwin, @JohnPallett, @toji, and many others. This isn't intended to imply that it represents a consensus, it's supposed to be a starting point for further conversations.

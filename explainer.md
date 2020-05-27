# **DOM Overlay Explainer**

## Introduction

The DOM overlays API provides a way to show DOM content such as text and control elements during an immersive AR session. This content is displayed as a transparent-background layer superimposed on the real-world scene and rendered WebGL content.

The current WebXR specification does not include support for inline AR content, the only available AR session mode is `immersive-ar`, and this is intended to exclusively show the real world or camera image combined with application-drawn WebGL content.

It would be very useful to support a hybrid mode where the scene can be augmented (no pun intended) with an overlayed 2D user interface based on HTML text and DOM elements such as buttons. This is convenient for developers, but also helps with features such as internationalization and accessibility that would be difficult to handle correctly in a purely WebGL-based UI. The goal is to make it possible for user agents to leverage existing support for DOM accessibility features so that they can be used for DOM content displayed through this new API.

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

On a large-FoV VR headset, the overlay may appear as a rectangle floating in space that's kept in front of the user, but isn't necessarily strictly head-locked (in order to improve comfort). This rectangle may be smaller than the maximum FoV if necessary to ensure that the corners and edges remain easily visible. (The visible area can depend on the user's eye position and eye-to-lens distance, and extreme angles may be blurry for some headsets.) The user agent may also provide ways for the user to reposition or resize the floating rectangle.

The display technology used affects how the overlay is composited. Applications should check the existing <code>[XRSession.XREnvironmentBlendMode](https://immersive-web.github.io/webxr/#xrsession-interface)</code> attribute. A see-through AR headset typically uses the <code>"additive"</code> blend mode, in this case black pixels appear transparent. If the session uses <code>"opaque"</code> or <code>"alpha-blend"</code> mode, the alpha channel is used to control visibility of DOM overlay contents.


## Input

Low-level inputs that intersect the DOM overlay rectangle (including transparent areas) will be forwarded to the overlay's DOM element for processing according to the usual DOM event propagation model, using event x/y coordinates mapped to the DOM overlay rectangle. For example, screen touch or ray inputs are converted to DOM input events including `"click"` events (required) and optionally also mousedown/mousemove/mouseup events if supported by the implementation.

If a WebXR application uses a DOM overlay in conjunction with XR input, it is possible that a user action could be interpreted as both an interaction with a DOM element and as 3D input to the XR application. For screen-based phone AR, a screen touch represents a ray being cast into the world which the application can use to trigger interactions such as placing a virtual object on the floor. The ambiguity arises when the touched screen location also shows a DOM element in the overlay. In this case, the user intent may be to use a world-interaction ray, for example when touching a transparent part of the DOM overlay, or the intent may be to interact with a DOM UI element such as a button. The application should be able to control if screen touches should be treated as world interactions, as DOM UI interactions, or both.

The same situation also applies for headset-based AR where the DOM overlay is displayed as a floating rectangle along with a tracked motion controller. DOM UI interactions would typically be based on a ray emanating from the controller. Whenever this ray intersects the DOM overlay, the UA emits DOM events at the intersected element, such as a `click` event when the controller's primary trigger is pressed. If the same controller and primary trigger are also used for world interactions, the application needs to be able to disambiguate the input. For example, if the application uses the primary trigger to activate a sculpting tool, this should be temporarily suppressed if the user is trying to click on a button shown in the DOM overlay.

More specifically, WebXR's [input events](https://github.com/immersive-web/webxr/blob/master/input-explainer.md#input-events) (`"selectstart"`, `"selectend"`, and `"select"`) potentially duplicate DOM events when the user is interacting with a part of the scene covered by the DOM overlay, including transparent areas. To help applications disambiguate, the user agent generates a `beforexrselect` event on the clicked/touched DOM element. If the application calls `preventDefault()` on the event, the WebXR "select" events are suppressed. The `beforexrselect` event bubbles, so the application can set an event handler on a container DOM element to prevent XR "select" events for all elements within this container.

```js
document.getElementById('ui-container').addEventListener('beforexrselect', (ev) => {
  ev.preventDefault();
});
```

A typical application would set such a `beforexrselect` handler for regions of the DOM UI that contain touchable UI elements. This often corresponds to opaque regions of the DOM overlay, but can be different. For example, noninteractive text regions could be treated as non-touchable (without a `beforexrselect` handler) so that they don't block the user from world interactions in those areas. Conversely, the application could set a `beforexrselect` handler on a transparent container element that's slightly larger than the UI elements in it, so that a slightly inaccurate touch that barely missed a button doesn't trigger an unexpected world interaction.

WebXR also supports non-event-based input. This includes controller poses, button/axis states, and transient XR input sources such as [Screen-based input](https://github.com/immersive-web/webxr/blob/master/input-explainer.md#screen) that creates 3D targeting rays based on 2D screen touch points. These inputs are not affected by DOM overlays and continue to be processed as usual. Applications are responsible for deduplicating these non-event inputs if they overlap with DOM events, though it's recommended to avoid UI designs that depend on this. There are many ambiguous corner cases, for example pointer movement that starts on a DOM element and ends outside it.

If using a DOM overlay in a headset, the implementation should behave the same as for smartphones whenever the primary XR controller's targeting ray intersects the displayed DOM overlay. In other words, generate `"click"` events when the primary trigger is pressed, subject to whatever additional restrictions are specified, such as those for disambiguation.

## Implementation

A sample implementation of this style of DOM Overlay for handheld AR displays the selected overlay element as a transparent fullscreen view that remains visible during an immersive-ar session. The site requests this mode as an optional or required feature in `requestSession`:

```js
navigator.xr.requestSession(‘immersive-ar’,
   {
     optionalFeatures: ["dom-overlay"],
     domOverlay: {
       root: document.body
     }
   }).then(onSessionStarted, onRequestSessionError);
```

On session start, the specified root element automatically enters fullscreen mode, and remains in this mode for the duration of the session. (Using the [Fullscreen API](https://fullscreen.spec.whatwg.org/) to change the fullscreen view is blocked by the UA.)

![Image of barebones overlay in AR mode](images/barebones-p50.png)
![Image of Lorenz attractor in AR mode](images/lorenz-p50.png)

During the session, the application can use normal DOM APIs to manipulate the content of the overlay element, use DOM event handlers, and also use WebXR rendering and controller input as usual.

By design, there must always be an active fullscreened element while the session is active. Fully exiting fullscreen mode also ends the immersive-ar session. Conversely, ending the immersive-ar session automatically fully exits fullscreen mode to ensure that the user doesn’t end up in an indeterminate state. (The Fullscreen API allows this according to [4. UI](https://fullscreen.spec.whatwg.org/#ui) : “The user agent may end any fullscreen session without instruction from the end user or call to exitFullscreen() whenever the user agent deems it necessary.”)

Input is handled by the fullscreened DOM element as usual, including advanced input such as displaying a keyboard when tapping a text input field. For example, a handheld AR application can use a DOM button for an "exit AR" action:

```js
let xrSession;
function onSessionStarted(session) {
  xrSession = session;
}

document.getElementById('exit-button').onclick = (ev) => {
  xrSession.end();
};
```

A headset-based implementation can also support this mode, this is very similar to showing a floating browser tab as would be used for an in-headset 2D browsing mode. The UA generates DOM input events based on XR controller actions, for example generating a `click` event at the location where the controller's pointer ray intersects the floating DOM content when the controller's primary trigger is used. This is intended to support a compatibility mode making content originally designed for handheld AR usable on a headset.

See also the [design sketch](http://docs/document/d/e/2PACX-1vRpXB5wX1R1QRzniysT5J1LhLQXAE5OMPX0kQiY-ozv8LsdsP22nf3mDyV6F8G92O_m0qAWMswLqOHT/pub) and [i2i entry](https://www.chromestatus.com/feature/6048666307526656).

## References & acknowledgements

This proposal is based on extended discussions in the `#immersive-web` community, and builds on proposals and suggestions by @ddorwin, @JohnPallett, @toji, and many others. This isn't intended to imply that it represents a consensus, it's supposed to be a starting point for further conversations.

## Full "barebones" example

The following self-contained "barebones" code is a lighly modified version of this sample:
https://immersive-web.github.io/webxr-samples/ar-barebones.html

```html
<!DOCTYPE html>
<html>
  <body>
    <div id="overlay">
      <header>
        <details open>
          <summary>Barebones WebXR DOM Overlay</summary>
          <p>
            This sample demonstrates extremely simple use of an
            "immersive-ar" session with no library dependencies, with an
            optional DOM overlay. It doesn't render anything exciting, just
            draws a rectangle with a slowly changing color to prove it's
            working.
            <a class="back" href="./index.html">Back</a>
          </p>
          <div id="session-info"></div>
          <div id="warning-zone"></div>
          <button id="xr-button" class="barebones-button" disabled>
            XR not found
          </button>
        </details>
      </header>
    </div>
    <main style="text-align: center;">
      <p>Body content</p>
    </main>
    <script type="module">
      // XR globals.
      let xrButton = document.getElementById("xr-button");
      let xrSession = null;
      let xrRefSpace = null;

      // WebGL scene globals.
      let gl = null;

      function checkSupportedState() {
        navigator.xr.isSessionSupported("immersive-ar").then(supported => {
          if (supported) {
            xrButton.innerHTML = "Enter AR";
          } else {
            xrButton.innerHTML = "AR not found";
          }

          xrButton.disabled = !supported;
        });
      }

      function initXR() {
        if (!window.isSecureContext) {
          let message = "WebXR unavailable due to insecure context";
          document.getElementById("warning-zone").innerText = message;
        }

        if (navigator.xr) {
          xrButton.addEventListener("click", onButtonClicked);
          navigator.xr.addEventListener("devicechange", checkSupportedState);
          checkSupportedState();
        }
      }

      function onButtonClicked() {
        if (!xrSession) {
          // Ask for an optional DOM Overlay, see https://immersive-web.github.io/dom-overlays/
          navigator.xr
            .requestSession("immersive-ar", {
              optionalFeatures: ["dom-overlay"],
              domOverlay: { root: document.getElementById("overlay") }
            })
            .then(onSessionStarted, onRequestSessionError);
        } else {
          xrSession.end();
        }
      }

      function onSessionStarted(session) {
        xrSession = session;
        xrButton.innerHTML = "Exit AR";

        // Show which type of DOM Overlay got enabled (if any)
        document.getElementById("session-info").innerHTML =
          "DOM Overlay type: " + session.domOverlayState.type;

        session.addEventListener("end", onSessionEnded);
        let canvas = document.createElement("canvas");
        gl = canvas.getContext("webgl", {
          xrCompatible: true
        });
        session.updateRenderState({ baseLayer: new XRWebGLLayer(session, gl) });
        session.requestReferenceSpace("local").then(refSpace => {
          xrRefSpace = refSpace;
          session.requestAnimationFrame(onXRFrame);
        });
      }

      function onRequestSessionError(ex) {
        alert("Failed to start immersive AR session.");
        console.error(ex.message);
      }

      function onEndSession(session) {
        session.end();
      }

      function onSessionEnded(event) {
        xrSession = null;
        xrButton.innerHTML = "Enter AR";
        document.getElementById("session-info").innerHTML = "";
        gl = null;
      }

      function onXRFrame(t, frame) {
        let session = frame.session;
        session.requestAnimationFrame(onXRFrame);
        let pose = frame.getViewerPose(xrRefSpace);

        if (pose) {
          gl.bindFramebuffer(
            gl.FRAMEBUFFER,
            session.renderState.baseLayer.framebuffer
          );

          // Update the clear color so that we can observe the color in the
          // headset changing over time. Use a scissor rectangle to keep the AR
          // scene visible.
          const width = session.renderState.baseLayer.framebufferWidth;
          const height = session.renderState.baseLayer.framebufferHeight;
          gl.enable(gl.SCISSOR_TEST);
          gl.scissor(width / 4, height / 4, width / 2, height / 2);
          let time = Date.now();
          gl.clearColor(
            Math.cos(time / 2000),
            Math.cos(time / 4000),
            Math.cos(time / 6000),
            0.5
          );
          gl.clear(gl.COLOR_BUFFER_BIT | gl.DEPTH_BUFFER_BIT);
        }
      }

      initXR();
    </script>
  </body>
</html>
```

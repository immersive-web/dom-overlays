# DOM overlay experiments


# Overview

This document collects information from experiments with implementing DOM overlay functionality in Chrome for Android targeting smartphone AR and VR headset mode.

# Experiment description

As an experiment, I modified Chrome for Android to compose a single interactive DOM quad with a transparent background on top of XR content. The goal was to learn more about the problem space, and to get a feel for potential benefits or problems with specific design choices. 


## Smartphone AR mode

In smartphone AR mode, the DOM content was effectively used as a fullscreen layer that's composed on top of the camera image and rendered scene content. Touch interactions with the DOM content worked as usual, with DOM user interface elements behaving essentially the same as in a non-AR fullscreened DOM element.


### AR: rendering and DOM integration



*   Text quality in general is pixel-identical to normal web browsing on the phone, it uses the same compositing and rendering logic. Drawing text on transparent elements can lead to contrast/legibility issues depending on the underlying content, but simple methods such as adding a _text-shadow_ style were quite effective. 
*   The experimental implementation effectively used three layers in a fixed stacking order. The camera view is on the bottom, the application WebGL content is alpha-blended on top of that, and the DOM content is a separate layer at the top, leaving the lower layers visible where the DOM layer has translucent color styles.
*   The browser's developer tools including Inspector worked for the DOM content, including screen captures (without the underlying camera view and GL output), and supporting interactive style modifications.


### AR: input and interactions



*   DOM UI interactions behaved just as they would for a regular DOM element using the HTML Fullscreen API. `<select>` inputs open an Android native selector, blurring the underlying content while active. Text input fields open the soft keyboard which works as expected. (Pose delivery may need to be paused for privacy while typing, this wasn't implemented for this test.)
*   Touch gestures such as inertial scrolling worked naturally. (This even included some unexpected gestures such as pull-to-reload that may need to be disabled in a production version.)
*   For this experiment, the DOM input events replaced WebXR input events while the DOM overlay is active. (This is a complex topic, see further discussions below in the "Design choices" section.)


### AR: interactions with HTML fullscreen API



*   The experiment was based on a modification of the existing HTML fullscreen API. This wouldn't necessarily be the case for a shipping implementation, but there will likely need to be at least some integrations between a smartphone _immersive-ar_ mode and the fullscreen API. For example, when the user exits _immersive-ar_ mode by pressing the Back button, it would be confusing if this leaves the user in a non-AR fullscreen mode where the user would need to press Back again to return to normal browsing mode. This would be a security risk if this could be abused to show a fake browser address bar. In this implementation, ending an immersive session also exits a separately requested fullscreen mode automatically. 


## VR mode on Daydream View

In VR mode (using a Daydream View headset and controller), the DOM content appeared as a quad floating in space rendered by Chrome's VR browsing mode, but modified to use a transparent background and remaining in the scene during the immersive-vr session. It supported interacting with DOM elements by using the controller as a pointing ray and clicking the primary button.


### VR: rendering and DOM integration



*   Text quality was good due to using a platform content quad layer that's separately composited, this avoids resampling into the lower-resolution WebXR application rendering buffer. However, due to the limited angular resolution while in VR, content would generally need to be rendered at larger text sizes than for normal smartphone browsing.
*   The UA-placed floating quad tended to intersect world content in immersion-breaking ways, and inconsistent Z depth due to lack of occlusion was uncomfortable. This happened frequently when using teleport movement. Other approaches such as a head-locked mode would behave differently, but it seems likely that users and developers will be frustrated with automated placement in at least some cases. Even small displacements can make a large difference here - text floating slightly in front of world content is fine, but floating slightly behind a wall looks bad.
*   If the application is responsible for drawing the targeting ray, accurate interactions require a clearly visible indicator where the ray intersects the DOM content quad. This wasn't implemented in my test. If the application is responsible for doing so, it would need per-frame information from the UA to do so, for example the current world space location of the content quad or a distance along the pointing ray. Alternatively, the UA could highlight the intersection point in the DOM layer, but this would need flexibility to ensure appropriate visual styling.
*   The application's drawn targeting ray needs to exactly match the UA's internal ray used for input events, any mismatch makes it difficult to do interactions. Applications don't have freedom to adjust the ray angle. This restriction seems reasonable, but several native VR applications allow input customizations such as adjusting the grip angle for virtual firearms, and this would conflict with UA-controlled input handling if the application renders the target ray differently as a result.


### VR: input and interactions 



*   UI elements such as `<select>` popups overall worked as expected due to reusing VR Browser's code for this, though they can be immersion-breaking. The current implementation rendered them as large floating versions of Android-native select dialogs, obscuring the scene content and temporarily removing focus from the scene while this interaction is in focus.
*   Interactions with DOM elements with a pointing ray can be fiddly. Small hand rotations while clicking get magnified into significant movement of the intersection point, making it difficult to accurately hit small elements, and clicks may be misinterpreted as drag motions. This could be worked around by tuning filtering parameters in the UA, but this needs to be done carefully to avoid inconsistencies with an application-rendered targeting ray.
*   Keyboard input into text fields wasn't supported in this experiment, implementing that would require showing a virtual keyboard. For privacy reasons, i.e. if the DOM element may contain cross-origin content, the UA may need to remove focus from the scene while input is active to ensure that the application can't read controller poses corresponding to keyboard input.


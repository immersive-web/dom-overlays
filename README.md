# DOM Overlays Specification

[![Build Status](https://travis-ci.org/immersive-web/dom-overlays.svg?branch=master)](https://travis-ci.org/immersive-web/dom-overlays)

The [DOM Overlays](https://immersive-web.github.io/dom-overlays/) is the 
repository of the [Immersive Web Working Group][webxrwg].

Originating proposal: [#50](https://github.com/immersive-web/proposals/issues/50)

## Taking Part

1. Read the [code of conduct][CoC]
2. See if your issue is being discussed in the [issues][8], or if your idea is being discussed in the [proposals repo][19].
3. We will be publishing the minutes from the bi-weekly calls.
4. You can also join the working group to participate in these discussions.


# Summary

A simple DOM overlay on top of the graphics (e.g., WebGL). The overlay would generally be mostly transparent, allowing the immersive graphics (or real world for AR), to show through. The overlay is useful for HUD, options, configuration, etc. It is not intended to provide in-world UI, such as a sign on the facade of a building.

## Motivation

Developers want to use DOM to create UI for their XR experiences. For VR, inline sessions are by definition within the DOM, but we have deferred this capability for immersive VR sessions. For AR, though, there is no inline mode, and 2D UI is especially important for popular use cases. We believe that supporting such UI is part of the minimum viable product for AR.

This was previously discussed in [WebXR issue #400](https://github.com/immersive-web/webxr/issues/400).

## Comparison to Quads/Layers

While there are a number of reasons to support a more general and capable mechanism for quads/layers (as previously discussed), that is a very complex topic that could take a long time to work out and may not be the best fit for some use cases and/or developers. This exploration does not preclude the other, and it may end up being desirable to support both.

# Example use cases

Use cases include:
*   Shopping: Select colors, etc. in 2D UI that change the rendered object. Also, Buy/Add to cart.
    *   A car configurator is a more complex version of this.
*   Education: Display information about things found in the real world or in response to interaction with virtual items.
    *   E.g., the [Chacmool AR demo](https://youtu.be/Zu6MXyfi-Ts?t=33)
*   Games: Provide HUD, score, options, etc.

In all of the cases above, there is a well-defined portion of the experience that is most easily generated using DOM and for which being separate from the immersive world is acceptable, if not desirable. User interaction with the DOM affects the immersive graphics and/or user interaction with the immersive graphics/world affects what is displayed in the DOM.

# Approach

Our initial idea is to explore something similar to the [Fullscreen API](https://developer.mozilla.org/en-US/docs/Web/API/Fullscreen_API), which causes a single DOM Element (of most any type) to be sized to a screen-dependent size and laid out independently of its parent tree. Where exactly the element would come from (i.e., would/could it be part of the page) will be part of the exploration. Another potential approach is one similar to [Picture-in-Picture V2](https://github.com/WICG/picture-in-picture/blob/v2/v2_explainer.md) where a separate `Document` is used.

For simplicity, we will start with a single “fullscreen” overlay, though the idea could potentially be extended to multiple overlay instances and/or allow the application and/or UA to affect the placement.

Our initial intent is that the API and overlay will not provide any guarantees about the relationship between pixels in the overlay and the position of virtual or real-world objects. Such use cases are probably better served by an API focused on world-relative placement. That said, if the exploration discovers an opportunity to address that use case in an overlay, we may explore it.

Input will be a key part of the exploration as the overlay will need to allow input to pass through the “transparent” portions of the overlay. There _may_ also be privacy and security issues related to input.

## A note on headsets

The mapping of this functionality to traditional flat displays is fairly clear - like many inline sessions and the immersive AR rendering, the DOM overlay would stretch across the entire screen. However, it is important that experiences created with such an API also work on other form factors, specifically headsets. Therefore, part of the exploration will include how a DOM overlay can be usefully rendered in headsets.

## Could this be used for VR modes?

In a word, maybe. This functionality is based on the fact that all AR sessions are “immersive,” even on smartphones. If this exploration is successful, it may be worth considering whether “immersive-vr” should also be supported on traditional displays and allow such an overlay to be specified. As part of this VR exploration, we may want to consider the usefulness vs. the complexity for VR implementations and the likely adoption by implementers.

<!-- Links -->
[CoC]: https://immersive-web.github.io/homepage/code-of-conduct.html
[webxrwg]: https://w3.org/immersive-web


# Security and Privacy Questionnaire

This document answers the [W3C Security and Privacy
Questionnaire](https://www.w3.org/TR/security-privacy-questionnaire/) for the
WebXR DOM Overlay Module specification.

**What information might this feature expose to Web sites or other parties,
and for what purposes is that exposure necessary?**

This feature does not directly expose any information to web sites or other
parties. It supports showing DOM content while a WebXR session is in progress
and supports interactions with this DOM content in addition to WebXR input
events. If the DOM content includes third-party content such as a cross-origin
iframe, the specification requires mitigations to ensure that user interactions with
this cross-origin content cannot be observed by the hosting site.

**Is this specification exposing the minimum amount of information necessary to
power the feature?**

Yes. The goal is to not expose any information beyond what the site would have
access to without using this module.

**How does this specification deal with personal information or
personally-identifiable information or information derived thereof?**

There are no direct PII exposed by this specification.

**How does this specification deal with sensitive information?**

As described in section [Event handling for cross-origin
content](https://immersive-web.github.io/dom-overlays/#cross-origin-content-events),
the user agent must not provide poses or gamepad input state for user
interactions with cross-origin content. The user agent can do so by preventing
interactions with cross-origin content, or by stopping input events while the
user is interacting with cross-origin content.

The logic for stopping input events follows the way that pointer/mouse events
are handled for normal 2D web pages that include cross-origin content, where the
hosting page stops getting `pointermove` and similar events while the pointer
position is over cross-origin content. The specification extends this to apply
to WebXR input events and gamepad data in addition to DOM events.

**Does this specification introduce new state for an origin that persists
across browsing sessions?**

No.

**What information from the underlying platform, e.g. configuration data, is
exposed by this specification to an origin?**

None.

**Does this specification allow an origin access to sensors on a user’s
device**

There is no additional sensor access. The specification adds restrictions to the
data provided from WebXR input devices, requiring that the user agent must block
updates while the user is interacting with cross-origin content.

**What data does this specification expose to an origin? Please also document
what data is identical to data exposed by other features, in the same or
different contexts.**

This specification isn't directly exposing any data to the origin.

**Does this specification enable new script execution/loading mechanisms?**

No.

**Does this specification allow an origin to access other devices?**

No.

**Does this specification allow an origin some measure of control over a user
agent’s native UI?**

No. This specification only applies to WebXR immersive sessions, and those
already use an immersive view that generally hides the browser's native UI, for
example using fullscreen mode for smartphone AR. The user agent already provides
mechanisms and instructions for exiting WebXR immersive sessions, and these
continue to apply when this module is in use.

**What temporary identifiers might this this specification create or expose to
the web?**

None.

**How does this specification distinguish between behavior in first-party and
third-party contexts?**

It is an extension to WebXR which is by default blocked for third-party contexts
and can be controlled via a Feature Policy flag. When blocked, third-party
contexts cannot use WebXR or any features from this specification.

If the first-party context includes third-party content, the specification adds
requirements to ensure that the first-party context does not get data about user
interactions with third-party content, as described elsewhere in this document.

**How does this specification work in the context of a user agent’s Private
Browsing or "incognito" mode?**

The specification does not mandate a different behaviour.

**Does this specification have a "Security Considerations" and "Privacy
Considerations" section?**

Yes.

**Does this specification allow downgrading default security characteristics?**

No.

**What should this questionnaire have asked?**

N/A

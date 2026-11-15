# Integration notes

## Scope

This is an extracted renderer with reference hosts, not a replacement launcher. Its Android
application ID is `io.github.tylerbuza.wintercabin`, separate from CleanYourTV. The renderer
retains its `com.gathin.winter` namespace so its two Java source files match the verified
in-launcher version without rewriting their packages.

The reference app has no Home intent filter, launcher configuration provider, module-card
metadata, network access or system-settings writer. Installing it alone would **not** put an
animated scene behind CleanYourTV's dock. No automatic installation or activation is supplied.

## Renderer lifecycle

Create `WinterCabinView` with the host's context and give it the available screen area.
It fills/crops to the view's aspect ratio from a 1280 x 720 design; a 16:9 screen shows the
whole composition. It is deliberately not focusable or clickable.

- Call `start()` when the host is resumed or the dream starts.
- Call `stop()` when it is paused, the dream stops, or the host is torn down.
- The renderer additionally gates callbacks on attachment and actual/window visibility.
- Explicit stop removes callbacks and drops the artwork reference rather than recycling a
  bitmap the render thread might still reference.
- A new eligible draw rebuilds the cached static artwork if needed. Repeated start/stop
  calls do not create extra timer loops.

The scene uses `SystemClock.elapsedRealtime()` so it needs no network clock, randomness per
frame or wall-clock/date permission. It never handles remote keys; normal host/Android
navigation remains responsible for leaving the scene.

## Home integration is a separate design decision

In the verified local implementation, the renderer sits behind Home content and the dream
runs in the existing launcher package. The launcher starts/stops the view with its lifecycle.
That implementation is **not included as a launcher patch**, because the personal fork has
unrelated features and the upstream integration route needs maintainer agreement first.

If upstream considers Home support, it should be opt-in/off by default with no behavior
change when disabled, no new permissions/network/dependencies, no Home-key or boot-path
changes, and no relaxation of the existing APK-size gate. The source here does not establish
that a proposed upstream integration would fit that gate; it would need to be built/measured.

If an independent add-on is preferred, a module-card contract implementation and actual-TV
verification of that separate package would still be needed before a gallery PR.

## Device evidence and known failure

The verified deployment is an in-launcher Home view and DreamService on Hisense SmartTV 4K FFM,
Android 11 / API 30, firmware 00.00.01.38. The current armchair drawing was checked on both
Home and screensaver. A prior integrated build passed a reboot test; the drawing-only change
was not separately reboot-tested.

An earlier standalone-package prototype failed to cold-start the dream after a reboot on
that firmware (system_server reported a null process-start result). An in-launcher dream
worked instead. This does not prove that every independent dream will fail, or that this
new reference package fixes it. The extracted Activity and DreamService are build-tested
only; device installation, cold start, reboot and remote-navigation testing remain open.

The public previews are scene-only, metadata-stripped images. The GIF samples two actual TV
frames; its playback timing is illustrative and is not a performance measurement. No raw
device logs, package lists, account/network identifiers, backups or viewing history are
needed to review this proposal.

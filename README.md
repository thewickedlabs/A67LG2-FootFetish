# A67LG2 Foot Fetish

Screenshot `FLAG_SECURE` windows without stripping the flag.

A KernelSU Next module for the **Foxxd A67L Gen 2**.

Apps mark sensitive windows with `FLAG_SECURE`; SurfaceFlinger then leaves those
layers out of normal captures, so the built-in screenshot hands back a black
frame. Authenticators and password managers, incognito/private browsing,
screenshot-blocking messengers, the Recents thumbnails of those apps — all of it
comes out black. This module captures the real content and saves it to your
gallery with the same chord you already use.

## How it's different

Most "disable secure screen" modules remove `FLAG_SECURE` itself — hooking the
WindowManager / SurfaceControl path so the flag never takes effect. That does
make the stock screenshot work, but it un-secures those windows *system-wide*:
every app's secure surfaces become ordinary ones, Recents starts caching them,
and the change is trivially detectable.

Foot Fetish leaves `FLAG_SECURE` intact. Apps still get a genuinely secure
window; nothing is globally un-secured. Instead it changes only the screenshot
path, in two parts:

- **Input.** A small root daemon `EVIOCGRAB`s the `gpio-keys` device and
  re-injects every event through a `uinput` clone *except* the Power +
  Volume-Down chord. The stock input stack never sees the chord, so the built-in
  screenshot never runs — which is exactly why there's **no shutter sound, no
  flash/animation and no corner preview**.
- **Capture.** On the chord the daemon signals a Zygisk module living in
  `system_server` — the one context allowed to — which calls the platform
  `ScreenCapture` with secure layers enabled, then a root service files the PNG
  into **Pictures/Screenshots**, indexes it in MediaStore, and pops a toast.

The net effect is the same picture in the same place as a normal screenshot,
only the secure layers are no longer blanked, and the secure flag itself is
never touched.

## Requirements

- Foxxd A67L Gen 2
- Rooted with KernelSU Next

The installer stops without changing anything if either is missing.

## Install

1. Download [`A67LG2-FootFetish.zip`](https://github.com/thewickedlabs/A67LG2-FootFetish/releases/latest/download/A67LG2-FootFetish.zip) (latest release).
2. KernelSU Next → Modules → Install from storage → select the zip.
3. Reboot.

It starts on its own at every boot. Tap the module's **Action** button in
KernelSU Next to turn the interceptor off or back on without rebooting (off =
the stock blacked-out screenshots return).

## Uninstall

Disable or remove the module and reboot. The grab is dropped and the key chord
goes straight back to the stock screenshot, shutter and animation and all.
Everything is systemless — no files under `/system` are touched — so there is
nothing to restore.

## Scope

It captures the secure layers SurfaceFlinger composites. Hardware-protected
buffers (Widevine L1 and similar) are dropped by the compositor itself and stay
black regardless; this device is L3. It's for imaging your own screen, not a DRM
workaround.

## License

[WTFPL](LICENSE).

---

Wicked Labs · https://www.cyberspace7.org

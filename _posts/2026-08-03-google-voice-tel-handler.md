---
title: "Make macOS tel: links open in Google Voice"
---

All I wanted was to be able to click on a phone number and have it do
something useful. One bald yak later, here we are. A phone number link
is just a URL. It often looks something like `tel:+18005550111`. It
would be nice to click on that and have it do something useful. Click
in Google Maps, click in your terminal, whatever.

On macOS, clicking that link does not necessarily mean your browser
gets to decide what happens. The operating system gets the first vote.
And my macOS system is sure eager to send it to FaceTime. Which I
don't use and is a dead-end on my system.

Here's how it works. LaunchServices looks for the application
registered as the handler for the `tel:` URL scheme, and it sends the
URL there. Meaning if FaceTime is the current default handler, then
clicking a phone number in Google Maps, in Chrome, can still end up in
FaceTime. No amount of poking through Chrome preferences will fix it.
You can see if you try the more general:

```bash
open 'tel:+18005550111'
```

Boom. FaceTime. From macOS.

I wanted all the `tel:` links that the system knows how to dispatch to
open my Google Voice account instead. From the browser, from a
terminal, from a document, from whatever you can think of.

That's the reason for the little helper app:

[google-voice-tel-handler](https://github.com/jasonhemann/google-voice-tel-handler)

## What the helper does

This is an AppleScript applet with an `on open location` handler. The
OS can send URL events to an applet, so the script receives the
original `tel:` URL, extracts the phone number, and opens the
corresponding Google Voice call URL in the default browser.

The script accepts the common shapes I've seen:

```text
tel:+18005550111
tel://+18005550111
tel:800-555-0111
```

It strips the `tel:` wrapper, keeps the digits, and opens a Google Voice URL
using the `/u/0` account slot:

```text
https://voice.google.com/u/0/calls?a=nc,%2B18005550111
```

Your default browser takes it from there.

## How it works

You can see all the source code. It's not much, and the AppleScript
itself is pretty readable.

There's a build script that compiles the applet with
`osacompile`, then patches the generated `Info.plist` so the bundle has a
stable identity and declares the URL scheme:

```text
CFBundleIdentifier = com.jasonhemann.google-voice-tel-handler
CFBundleURLTypes   = tel
```

The tools involved are already on macOS: `osacompile`, `PlistBuddy`,
and `codesign`.

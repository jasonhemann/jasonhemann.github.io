---
title: "Can Emacs Mac Port Have It All? Multi-ttwhy?"
---

I want to have just one warm Emacs process. From it, I want to open a
graphical frame when working in my OS GUI, open a text frame from a
terminal or over SSH, and have both be views onto the same buffers and
the same Lisp world. This does not seem like a big ask. Emacs has a
server; `emacsclient` has flags for exactly this:

```bash
emacs --daemon
emacsclient -c             # a new graphical frame
emacsclient -t             # a new frame on this terminal
```

Easy peasy. And, once the server is warm, lickety-split.

Except that this combination does not work in an unpatched Emacs Mac
Port. GNU Emacs includes its official backend. Emacs Mac Port is a
separate port with its own collection of excellent macOS integrations
and its own `mac` display backend. On a mac it should be the obvious
choice. However, in Mac Port as it stands, GUI-first startup redirects
`-t` to a graphical frame, while daemon-first startup makes `-c` fall
back to a terminal frame. 0/2.

The [current Mac Port source still says](https://github.com/jdtsmith/emacs-mac/blob/emacs-mac-30_1_exp/README-mac#L209-L212)
that it does not support multi-tty together with its GUI. TTY-only
multi-tty is supposed to work.

My current workaround was to stop using the Mac Port. I gave up the
Mac Port-specific features, but I do get the client/server model I was
trying to configure. But there is hope on the horizon.

## The failure is asymmetric

First, an Emacs vocabulary lesson. A graphical window containing Emacs
is a *frame*. An Emacs *window* is one of the *panes* inside it. A
terminal screen occupied by Emacs is also a frame, on a different
terminal. *Multi-tty* is Emacs's ability to attach one running Emacs
process to multiple terminals at the same time. Each terminal gets its
own Emacs frame, while all the frames share the same buffers. A
graphical frame and a text frame belonging to one Emacs process
are one particular instance of this.

One Emacs process is designed to use [graphical and text terminals
simultaneously](https://www.gnu.org/software/emacs/manual/html_node/elisp/Frames.html).
The documented jobs of [`emacsclient -c` and `emacsclient
-t`](https://www.gnu.org/software/emacs/manual/html_node/emacs/emacsclient-Options.html)
are to ask that process for a graphical or text frame, respectively.
This is the general Emacs model.

So there are two kinds of failures in current release Mac Port:

* Start the GUI first and call `server-start`: graphical client frames
  work, but `emacsclient -t` is instead redirected to a graphical frame.
* Start a frame-less `--daemon` first: the process cannot initialize
  the Mac GUI later, so `emacsclient -c` cannot make its first
  graphical frame. When `-c` cannot make a graphical frame,
  `emacsclient` deliberately falls back to a terminal frame, and so it
  looks like the client is just ignoring our request.

Both of these symptoms are described in the still-open
[issue #52](https://github.com/railwaycat/homebrew-emacsmacport/issues/52).

## A partial fix ships downstream

Railwaycat's Homebrew builds apply a downstream multi-tty patch in the
current formulas for
[Emacs Mac 29](https://github.com/railwaycat/homebrew-emacsmacport/blob/master/Formula/emacs-mac%4029.rb#L26-L33),
[30 experimental](https://github.com/railwaycat/homebrew-emacsmacport/blob/master/Formula/emacs-mac%4030exp.rb#L26-L33),
and [31 experimental](https://github.com/railwaycat/homebrew-emacsmacport/blob/master/Formula/emacs-mac%4031exp.rb#L48-L55).

That patch fixes the GUI-first half. Start the Mac Port as an ordinary
graphical application, run `server-start`, and the same process can
serve both GUI and TTY clients. Users of Railwaycat's working Homebrew
builds do not have to apply it by hand. As of September 1, 2026, however,
the 31 experimental formula fails to install because that patch no
longer applies cleanly after upstream branch changes; the failure is
tracked in [issue #420](https://github.com/railwaycat/homebrew-emacsmacport/issues/420).

It does not fix the daemon-first half. A true Mac Port daemon still
cannot initialize its first graphical display. And deleting the last
graphical frame while a TTY frame remains can leave the Mac application
in a bad state. The established
[`mac-pseudo-daemon`](https://github.com/DarwinAwardWinner/mac-pseudo-daemon)
workaround handles that lifecycle problem by starting graphically and
keeping a hidden GUI frame alive. It is a useful pseudo-daemon, but the
"pseudo" is doing real work in that name.

## Good news everyone---a full fix?

Here is a delightful bit of news. On August 30, 2026, someone opened
[Mac Port pull request #143](https://github.com/jdtsmith/emacs-mac/pull/143),
"mac: support GUI and TTY frames from daemons". It removes the remaining
mixed-frame blockers, makes a frame-less daemon recognize the Mac
display before its first GUI frame exists, and starts AppKit event
processing after that display is initialized. The author reports passing
the server and client tests and manually testing TTY and GUI clients in
both orders.

The patch is small and cleanly mergeable, but it has no human review
or project CI result yet. This is the first credible candidate I have
found for the whole Mac Port problem. This two-day-old code could be
just the piece we needed. Right now it is awaiting review.

The state of play as of September 1, 2026, is:

| Setup                                        | GUI and TTY frames together | Frame-less daemon can create its first GUI frame |
|----------------------------------------------|-----------------------------|--------------------------------------------------|
| Standard GNU Emacs build                     | Yes                         | Yes                                              |
| Unpatched Emacs Mac Port                     | No                          | No                                               |
| Railwaycat-patched Mac Port                  | Yes, if GUI-first           | No                                               |
| PR #143 applied to experimental Emacs Mac 30 | Author reports yes          | Author reports yes                               |

## So, multi-ttwhy?

The best way to solve a problem is sometimes to let it be the problem of
someone with more time or talent. Here, GNU Emacs already had the model,
Railwaycat carries the practical Mac Port patch, and now a new pull
request attempts the last daemon-to-GUI step.

The prospective full fix is only two days old, but our long international parenthetical editing nightmare may soon be over.

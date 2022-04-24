---
title: Resolving emacs load path shadows
---

I recently experienced some emacs problems suggestive of load path
issues. Executing `M-x list-load-path-shadows` gives a nice long list
of shadowed files from load path. I use `straight.el` as my emacs
package manager. It's very convenient for all the reasons that the
`straight.el` documentation lays out, but naive usage can lead you to
introducing load-path conflicts. Consider an example like the following:



```
(straight-use-package 'eri)
(straight-use-package 'agda2-mode)
```

I included both of these in my `.emacs` configuration, asking
`straight.el` to load, and install if necessary, both of these
packages at start-up. However! It turns out `agda2-mode` *already*
includes `eri`! But `straight` doesn't know that. So instead we need
to inform it of that fact.


```
(straight-use-package '(agda2-mode :includes eri))
```

It turns out that `agda2-mode` contains within it a couple of things,
so `straight.el` provides some extra syntax for passing a list of
options.


```
(straight-use-package '(agda2-mode :includes (eri annotation)))
```



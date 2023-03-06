## Introduction

This package provides a way to reuse the current dired buffer to visit
another directory (rather than creating a new buffer for the new directory).
Optionally, it allows the user to specify a name that all such buffers will
have, regardless of the directory they point to.

## Installation

Put this file on your Emacs-Lisp load path and add the following to your
~/.emacs startup file.

```
(require 'dired-single)
```

or you can load the package via autoload:

```
(autoload 'dired-single-buffer "dired-single" "" t)
(autoload 'dired-single-buffer-mouse "dired-single" "" t)
(autoload 'dired-single-magic-buffer "dired-single" "" t)
(autoload 'dired-single-toggle-buffer-name "dired-single" "" t)
```

or you can just add `(package-initialize)` to automatically load every
package installed by `package.el`.

To add a directory to your load-path, use something like the following:

```
(setq load-path (cons (expand-file-name "/some/load/path") load-path))
```

## Usage

`M-x dired-single-buffer`

Visits the selected directory in the current buffer, replacing the
current contents with the contents of the new directory.  This doesn't
prevent you from having more than one dired buffer.  The main difference
is that a given dired buffer will not spawn off a new buffer every time
a new directory is visited.

If the variable dired-single-use-magic-buffer is non-nil, and the current
buffer's name is the same as that specified by the variable
dired-single-magic-buffer-name, then the new directory's buffer will retain
that same name (i.e. not only will dired only use a single buffer, but
its name will not change every time a new directory is entered).

`M-x dired-single-buffer-mouse`

Essentially this is the same as `dired-single-buffer', except that the
action is initiated by a mouse-click instead of a keystroke.

`M-x dired-single-magic-buffer`

Switch to an existing buffer whose name is the value of
dired-single-magic-buffer-name. If no such buffer exists, launch dired in a
new buffer and rename that buffer to the value of
dired-single-magic-buffer-name.  If the current buffer is the magic buffer,
it will prompt for a new directory to visit.

`M-x dired-single-toggle-buffer-name`

Toggle between the 'magic' buffer name and the 'real' dired buffer
name.  Will also seek to uniquify the 'real' buffer name.

`M-x dired-single-up-directory`

Like `dired-up-directory`, but reuses the buffer.  It handles the case
where the dired buffer has multiple subdirs listed (with
`dired-maybe-insert-subdir`), by just moving the point to the subdir
without refreshing the buffer.

## Recommended Keybindings

To use the single-buffer feature most effectively, I recommend adding the
following code to your .emacs file.

The following code will work whether or not dired has been loaded already.

```
(defun my-dired-init ()
  "Bunch of stuff to run for dired, either immediately or when it's
   loaded."
  ;; <add other stuff here>
  (define-key dired-mode-map [remap dired-find-file]
    'dired-single-buffer)
  (define-key dired-mode-map [remap dired-mouse-find-file-other-window]
    'dired-single-buffer-mouse)
  (define-key dired-mode-map [remap dired-up-directory]
    'dired-single-up-directory))

;; if dired's already loaded, then the keymap will be bound
(if (boundp 'dired-mode-map)
    ;; we're good to go; just add our bindings
    (my-dired-init)
  ;; it's not loaded yet, so add our bindings to the load-hook
  (add-hook 'dired-load-hook 'my-dired-init))
```

To use the magic-buffer function, you first need to start dired in a buffer
whose name is the value of dired-single-magic-buffer-name.  Once the buffer
has this name, it will keep it unless explicitly renamed.  Use the function
dired-single-magic-buffer to start this initial dired magic buffer (or you can
simply rename an existing dired buffer to the magic name).  I bind this
function globally, so that I can always get back to my magic buffer from
anywhere.  Likewise, I bind another key to bring the magic buffer up in the
current default-directory, allowing me to move around fairly easily.  Here's
what I have in my .emacs file:

```
(global-set-key [(f5)] 'dired-single-magic-buffer)
(global-set-key [(control f5)] (function
        (lambda nil (interactive)
        (dired-single-magic-buffer default-directory))))
(global-set-key [(shift f5)] (function
        (lambda nil (interactive)
        (message "Current directory is: %s" default-directory))))
(global-set-key [(meta f5)] 'dired-single-toggle-buffer-name)
```

Of course, these are only suggestions.

## Customization

M-x `dired-single-customize` to customize all package options.

The following variables can be customized:

* `dired-single-use-magic-buffer`

Boolean used to determine if the dired-single functions should look
for and retain a specific buffer name.  The buffer name to look for
is specified with `dired-single-magic-buffer-name`.

* `dired-single-magic-buffer-name`

Name of buffer to use if `dired-single-use-magic-buffer' is true.  Once a
dired buffer has this name, it will always keep this name (unless it's
explicitly renamed by you).

## Credits

The single buffer code is loosely based on code posted to the NT-Emacs
mailing list by Steve Kemp & Rob Davenport.  The magic buffer name code
is all mine.

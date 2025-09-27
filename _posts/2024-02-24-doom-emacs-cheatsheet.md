---
title: "Doom Emacs Cheatsheet"
date: 2024-02-24 15:00
categories: [doom, emacs,cheatsheet]
tags: [doom,emacs,cheatsheet]
published: false
---

Doom Emacs enhances Emacs by configuring it with sane defaults and adding a modern UI, among other improvements. Here's a quick cheat sheet to get you started with Doom Emacs.

## General

SPC is your space key

| Command                | Description                        |
|------------------------|------------------------------------|
| `SPC .`                | Open file finder                   |
| `SPC x`                | Execute command                    |
| `SPC b b`              | Switch buffer                      |
| `SPC b d`              | Kill buffer                        |
| `SPC b R`              | Revert buffer without confirmation |
| `ctrl w v`             | Split window vertically            |
| `ctrl w s`             | Split window horizontally          |
| `SPC w d` or `SPC w c` | Close current window               |
| `SPC TAB`              | Switch to last buffer              |
| `SPC w w`              | Switch window                      |

## Code and Text Editing

| Command                  | Description                              |
|--------------------------|------------------------------------------|
| `SPC c c`                | Compile                                  |
| `SPC c l`                | Comment or uncomment lines               |
| `SPC /`                  | Search in project                        |
| `SPC s s`                | Search in current buffer                 |
| `SPC s r`                | Replace in current buffer                |
| `SPC g g`                | Magit status                             |
| `SPC g c c`              | Commit changes                           |
| `SPC g p p`              | Push changes                             |
| `SPC g o`                | Open GitHub/GitLab repository            |

## Navigation

| Command                  | Description                              |
|--------------------------|------------------------------------------|
| `SPC h d b`              | Describe bindings                        |
| `SPC h d f`              | Describe function                        |
| `SPC h d v`              | Describe variable                        |
| `SPC h d k`              | Describe key                             |
| `SPC j j` or `C-j`       | Jump down                                |
| `SPC j k` or `C-k`       | Jump up                                  |

## Org Mode

| Command                  | Description                              |
|--------------------------|------------------------------------------|
| `SPC m a`                | Org agenda                               |
| `SPC m c`                | Capture item                             |
| `SPC m l`                | Insert link                              |
| `SPC m t`                | Todo state cycle                         |
| `SPC m T`                | Set specific todo state                  |
| `SPC m h`                | Insert heading                           |
| `SPC m H`                | Insert subheading                        |
| `SPC m -`                | Insert list item                         |

## Emacs and Doom

| Command                  | Description                              |
|--------------------------|------------------------------------------|
| `SPC h r r`              | Reload Doom configuration                |
| `SPC q r`                | Restart Emacs                            |
| `SPC q q`                | Quit Emacs                               |


This cheat sheet covers some basic Doom Emacs functionality. Explore the Doom Emacs documentation and the vast ecosystem of Emacs packages to further enhance your productivity.

## Python Development Environment

### Inside the init.el, uncomment

```lisp
(python +lsp)
lsp
```

### Install the language servers, i use pipenv, but pip will work

```bash
pipenv install python-lsp-server

sudo pacman -S npm

npm install pyright pyright-langserver
```

### Inside the config.el

```lisp
;; REMEMBER TO CHANGE THE PATH TO YOUR NODE-MODULES .bin DIRECTORY
(after! tramp
  (setq tramp-remote-path (append tramp-remote-path '("/home/jonny/node_modules/.bin/"))))

(with-eval-after-load 'tramp
  (add-to-list 'tramp-remote-path 'tramp-own-remote-path))

;;(after! eglot
  ;; Ensure `pyright` is used for Python files
;;  (add-to-list 'eglot-server-programs '(python-mode . ("pyright-langserver" "--stdio"))))
(use-package! company
  :after eglot
  :config
  ;; This configures company to use company-capf, which is the completion-at-point-function backend
  (setq company-idle-delay 0.5) ; Adjust the delay according to preference
 ;; company-capf is a generic completion mechanism that works with eglot.
  (setq company-backends '(company-capf)))

;; Ensure `eglot` is installed and configured for Python
(use-package! eglot
  :hook (python-mode . eglot-ensure)
  :config
  (add-to-list 'eglot-server-programs '(python-mode . ("pyright-langserver" "--stdio")))
  (add-hook 'eglot-managed-mode-hook (lambda () (company-mode 1))))

(setq org-agenda-files (directory-files-recursively "~/org-agenda" "\\.org$"))
(require 'pyvenv)
(add-hook 'python-mode-hook #'pyvenv-mode)

(defun open-eshell-in-bottom-window ()
  "Split the window and open eshell in the bottom window with a specific size."
  (interactive)
  ;; Calculate the desired window height for the eshell window: e.g., 1/3 of the frame height.
  (let ((window-size (- (floor (* 0.33 (frame-height))) window-min-height)))
    ;; Split the window with the specified height and open eshell in the new window.
    (split-window-below (- window-size))
    (other-window 1)
    (eshell)))

(global-set-key (kbd "C-c e") 'open-eshell-in-bottom-window)
```

### Inside the packages.el

```lisp
(package! pyvenv)
(package! eglot)
```

## Inside alt-x
    - open-eshell-in-bottom-window C-c e
    - doom/reload
    - eshell
    - eglot
    - pyvenv-activate (although, my config should automatically load this if in a package directory with a virtual environment)

# Using Emacs for Haskell development

Emacs is a text editor with an amazing support for extensions. Many people have built add-ons for Emacs to deal with everything from e-mail to version control or agenda planning, but it excels in providing good support for programming. Emacs support for Haskell is actually amazingly good - it integrates highlighting, querying information, building with Cabal and even refactoring!

Note that the support for Haskell is not provided by a monolithic extension, but rather by a set of them with different features and also with different sets of requirements. From my point of view, `haskell-mode` and `ghc-mod` are a must, and work fairly well together.

## Index

* [Installing and setting up Emacs](#installing-and-setting-up-emacs)
  * [Keeping packages up-to-date](#keeping-packages-up-to-date)
* [Haskell preliminaries](#haskell-preliminaries)
  * [Dealing with Cabal hell](#dealing-with-cabal-hell)
* [`haskell-mode`](#haskell-mode)
  * [Indentation modes](#indentation-modes)
  * [Non interactive commands](#non-interactive-commands)
  * [Interactive commands](#interactive-commands)
  * [Debugging](#debugging)
  * [Further customization](#further-customization)
* [`ghc-mod`](#ghc-mod)
  * [Insertion commands](#insertion-commands)
  * [Working with holes](#working-with-holes)
  * [Completion with `company-ghc`](#completion-with-company-ghc)
  * [Refactoring with HaRe](#refactoring-with-hare)
* [`structured-haskell-mode`](#structured-haskell-mode)
* [Other niceties](#other-niceties)
* [Summary](#summary)
  * [List of key bindings](#list-of-key-bindings)
  * [Installing all packages](#installing-all-packages)

## Installing and setting up Emacs

The first thing you need to do is to get Emacs up and running on your computer. Note that you need at least version 23 (which is any case is quite old, the current one is 24.x). How to do this is easy in any operating system you use:

  * In Linux, you usually need to install the `emacs` package from your package management system. In Ubuntu or Debian, you can do this graphically using either the Software Center or Synaptic, or using the command line by issuing the command `sudo apt-get install emacs`.
  * In Mac OS X, download [Emacs for OS X](http://emacsformacosx.com/), and install like any other application.
  * In Windows you can use [NTEmacs](http://ntemacs.sourceforge.net/).

Given the huge range of extensibility that Emacs provides, instead of a graphical interface sometimes it becomes necessary to change options by writing in your personal configuration file. So, open Emacs, press `M-:`, then write `(find-file user-init-file)` and finally press Enter. The configuration file will appear in front of you.

Wait, wait! What does this `M-:` thing mean? This is how key combinations are represented in Emacs. In particular, `M` stands for the meta key, which means Alt in Linux or Windows systems, or the Option/Command key if running on Mac OS X (Mac OS X users can find more information about how to set up their preferred `M` key in the [Emacs documentation](http://ftp.gnu.org/old-gnu/Manuals/emacs/html_chapter/emacs_36.html#SEC548)). When followed by a dash `-`, keys must be pressed together. Thus, you should press the meta key corresponding to your system and at the same time those corresponding to `:`. On my computer, this means `Alt+Shift+;`. The Emacs culture involves using many of these key combinations to perform operations in a fast way. But fear not - you always have the menu and the toolbar to look through for commands, while you are learning the key combinations.

Note also that it is important that you write the command as-is, including all parentheses. This slightly strange syntax is due to Emacs using [Lisp](http://en.wikipedia.org/wiki/Emacs_Lisp) as its configuration language.

OK, let's move on with the configuration. In the file that has just been opened, paste the following content:
```lisp
(require 'package)
(add-to-list 'package-archives
  '("melpa-stable" . "http://stable.melpa.org/packages/") t)
(package-initialize)
```
What this configuration is doing is adding a new repository of packages. Repositories are places where the integrated Emacs package manager looks for new extensions. In this case, we add the [MELPA Stable](http://stable.melpa.org/) repository, which provides the packages mentioned throughout this article.

**Note:** there is also an [unstable MELPA repository](http://melpa.org), but we strongly advise to use the stable one.

To save the contents of the file, press `C-x C-s`. `C` is mapped to the Ctrl key in Windows, Linux, and Mac OS X; and the fact that two key combinations are given separated by space means that you need to press `C-x` and then `C-s`, in separate keystrokes.

With the changes saved, they'll be applied next time you open Emacs, and every time thereafter. But you don't need to restart Emacs right now in order to apply them in your current session! That's just a snippet of Emacs Lisp, after all, and you can run it like any other. To do so, press `M-x`, then write `eval-buffer` at the prompt, and finally press Enter. Now your Emacs knows about the MELPA package repository.

Wait, wait! What is this `eval-buffer` thing? It's an example of an Emacs _extended command_, and indeed most things that Emacs can do, via keystrokes or menu items or toolbar buttons or all of the above, can also be done in this way. It's basically like having a command line right there in your editor, and it's an extremely useful thing; there's no need to worry overmuch about it right now, but do keep it in mind, especially when you discover a command that you don't use often enough to find it worth binding to a keystroke, but like to have handy when you do need it. Onward!

Speaking of keystrokes, a piece of advice: some of the common shortcuts in graphical interfaces are different within Emacs. For example, cut, copy and paste are  `C-w`, `M-w` and `C-y` respectively. I suggest taking some time and learning these new commands, because they integrate very tightly with the Emacs workflow. However, if you are a newcomer and want to follow this tutorial with the key bindings found in other text editors, you can change the Emacs defaults to those by adding:
```lisp
(cua-mode 1)
```
to your Emacs initialization file. This helped me a lot in my transition to Emacs!

### Keeping packages up-to-date

One nice thing about using the integrated Emacs package manager is that you can easily upgrade your installed packages to the latest version. To do so, go to the full package view by issuing `M-x package-list-packages`. You will get a list of all packages that you may install from MELPA. The first thing to do is refreshing that list to the latest version, something you can do by either pressing `r` or going to the _Package_ menu and selecting _Refresh Package List_.

Now Emacs knows of all latest versions. The next step is marking all the upgradeable packages to be installed. You can do so via the _Package_ menu, in _Mark Upgradeable Packages_, or just pressing `U`. Finally, you need to execute the install plan either from the same menu, choosing the _Execute Actions_ item, or pressing `x`. Emacs will ask for confirmation, and will then download and install the new versions and remove the previous ones. Since some of these changes affect parts of Emacs which can't be modified within a running session, you will want to restart Emacs after the package update is complete, in order to avoid the random brokenness which may otherwise result.

## Haskell preliminaries

In order to use some of the Emacs packages presented here, you need to install Haskell packages on your computer. Many of them need as a prerequisite the `happy` tool. However, in the current state, Cabal is not able to install tools needed to build a package automatically, so you need to do this first. Thus, open a terminal and execute:
```
cabal update
cabal install happy
```
The first command ensures that you have update information of the libraries and tools available in Hackage, the Haskell community repository. The second command takes care of downloading and installing `happy` itself.

### Dealing with Cabal hell

In some cases, Cabal will refuse to install some of the packages described in this section because of incompatibilities between versions. The point where you arrive when this happens is usually called _Cabal hell_. Fortunately, it is a hell from which you can escape.

The main solution is simply wiping out your whole Cabal user cache of packages, and starting anew. This is done by running on a terminal:
```
rm -rf `find ~/.ghc -maxdepth 1 -type d`
rm -rf ~/.cabal/lib
rm -rf ~/.cabal/packages
rm -rf ~/.cabal/share
```
Note that this will force reinstalling any dependencies of other packages that you use. But this is OK, since Cabal takes care of doing this automatically.

In order to avoid going into hell again, my recommendation is to use _sandboxes_ in your own projects. A sandbox creates an isolated environment where a specific set of packages is available, instead of installing everything in a global database. To get more information about sandboxes, you can visit [the corresponding section of the Cabal documentation](http://www.haskell.org/cabal/users-guide/installing-packages.html#developing-with-sandboxes).

## `haskell-mode`

Now that the repository is set up, installing `haskell-mode` is very easy:
  * Press `M-x`, then write `package-refresh-contents`. In the bottom of your Emacs window you should see the text `M-x package-refresh-contents`. Then press Enter: this will update the list of available packages in the repositories.
  * Press `M-x`, then write `package-install` and press Enter.
  * The bottom of the window will show the message `Install package:`. Write `haskell-mode` and press Enter.
  * The Haskell mode will be downloaded and installed under your user directory.

A piece of advice: most of the times when you need to write something in Emacs, you can use Tab to autocomplete what you have written or show different possibilities. For example, if you press `M-x` and then write `package-` and press Tab, you will see all the commands related to package management popping up. If you then continue writing `r` and press Tab, the only option is `package-refresh-contents`, so it will be written for you ;)

Before continuing, note that `haskell-mode` has many more features and options that the ones we are going to talk about. You can learn more about them in [its wiki](https://github.com/haskell/haskell-mode/wiki).

### Indentation modes

In order to use `haskell-mode`, you need to select one of the three [indentation modes](https://github.com/haskell/haskell-mode/wiki/Indentation) that it provides. The indentation mode specifies how Enter and Tab will be treated when working with Haskell code. The most advanced one is called `haskell-indentation`. To enable it:
  * Open your personal configuration file. Remember, to do so press `M-:`, then write `(find-file user-init-file)` and finally press Enter.
  * Add a new line containing `(add-hook 'haskell-mode-hook 'turn-on-haskell-indentation)`.
  * Save the file with `C-x C-s`.
  * Apply the changes with `M-x eval-buffer RET`, or restart Emacs.

It is time to open a Haskell file! You can do it by issuing `C-x C-f` and then writing the path to either a new or an existing file. It is required that the extension is `.hs` or `.lhs` for Emacs to recognize that it is a Haskell file and enable the newly-installed `haskell-mode`.

If you write Haskell code now, it should be syntax highlighted. If you press Tab in the file, `haskell-indentation` will take care of the layout rule and indent your code as needed. But let's use some time to look at the bottom of the screen: after the file name, you should notice that `Haskell Ind` is written. This tells you about the major and minor mode of this file, an important concept in Emacs:
  * A major mode, in this case `Haskell`, defines the global way of interacting with the file. Each file has one major mode associated with it, usually related to the programming language in which the file is written.
  * In addition, any amount of minor modes can be enabled. Those define smaller interactions which complement the major one. In this case, we have `Ind`, related to indentation. In many cases, you will have a great deal of minor modes enabled per file.

There is also an external package, called _haskell-indentation 2n try_, or simply [`hi2`](https://github.com/errge/hi2), which provides some changes from `haskell-indentation` to make it easier to use. The package is available in MELPA, so you can get it easily from the repository. As a reminder, installing a package is done by pressing `M-x`, then `package-install`, and finally writing the name of the package, in this case `hi2`. To activate the indentation mode for Haskell files, you need to insert the following text in the configuration file:
```lisp
(add-hook 'haskell-mode-hook 'turn-on-hi2)
```
Note that adding this extra indentation mode is not strictly necessary: in many cases, the built-in `haskell-indentation` works perfectly.

For re-indenting parts of your code in an appropiate style, you might look at the [`hindent`](https://github.com/chrisdone/hindent) from Chris Done. You can get it from MELPA, using the package name `hindent`. You will also need to install the corresponding package using `cabal install hindent` (wait for the next section to get more information about installation of Haskell libraries). Finally, in your configuration file add the following text:
```lisp
(add-hook 'haskell-mode-hook #'hindent-mode)
```
Then, by calling `M-q`, you can reformat the current declaration.

### Non interactive commands

`haskell-mode` commands are divided into two groups, depending on whether they rely on an open interpreter session. To those who need such an interpreter, we will refer to as *interactive*. We have already seen that `haskell-mode` helps with indentation, but what else can it do in non interactive mode?

The first thing it can do is help with `import`s. If you issue the command `M-x haskell-navigate-imports`, the editor will be focused on each of the import blocks in your file. Note that given the usefulness of this feature, the [`haskell-mode` wiki](https://github.com/haskell/haskell-mode/wiki/Import-management) suggests binding the `F8` key combination to run this command, by adding to your personal configuration file the following line:
```lisp
(eval-after-load 'haskell-mode
          '(define-key haskell-mode-map [f8] 'haskell-navigate-imports))
```
Once you've applied your changes via `M-x eval-buffer RET`, pressing `F8` moves you to the import section. (`RET` is how Emacs keybindings represent the Enter or Return key.) But this is not enough, `haskell-mode` can also sort and align your import sections nicely. This is available in the key binding `C-c C-.`

In addition to going to `import` sections, `haskell-mode` can navigate to any other definition in the file. However, to get this working you need to install an extra program called `hasktags` and enable the feature in Emacs. As you may know, the installation of Haskell libraries and tools is centralized via a tool called Cabal which comes along with the Haskell platform or the GHC compiler. To install `hasktags`, open a terminal and execute the commands:
```
cabal update
cabal install hasktags
```
The first command ensures that you have update information of the libraries and tools available in Hackage, the Haskell community repository. The second command takes care of downloading and installing the `hasktags` program along with any dependencies.

Then open the personal configuration file and add the following lines:
```lisp
(let ((my-cabal-path (expand-file-name "~/.cabal/bin")))
  (setenv "PATH" (concat my-cabal-path path-separator (getenv "PATH")))
  (add-to-list 'exec-path my-cabal-path))
(custom-set-variables '(haskell-tags-on-save t))
```
The first three lines are needed to tell Emacs that it should look for the `hasktags` program in your Cabal binaries directory, which is not a location it already knows about. The next line is the one enabling `hasktags` itself. From now on, you can use the `M-.` key combination to jump to the definition of an element when you are over it.

Note: in order to use the `M-.` command, your file needs to be in a Cabal project, or `haskell-mode` will ask to create a temporary one.

The third feature that `haskell-mode` provides without interaction with an interpreter is formatting of code. This is done again by resorting to an external tool, which should be installed with:
```
cabal update
cabal install stylish-haskell
```
Note that Emacs needs to find the corresponding executable for this feature to work. In particular, if you haven't done it previously, you need to add the first two lines referred to in the `hasktags` section above (those dealing with paths). Any way, by using the command `M-x haskell-mode-stylish-buffer`, you can format your Haskell file in a nice way :)

### Interactive commands

As usual, enabling a new feature in the Haskell mode involves changing some configuration file. In particular, we need to enable the interactive features and also define the key bindings to be used. Opening the configuration file is becoming normal for you now, so do so and add:
```lisp
(custom-set-variables
  '(haskell-process-suggest-remove-import-lines t)
  '(haskell-process-auto-import-loaded-modules t)
  '(haskell-process-log t))
(eval-after-load 'haskell-mode '(progn
  (define-key haskell-mode-map (kbd "C-c C-l") 'haskell-process-load-or-reload)
  (define-key haskell-mode-map (kbd "C-c C-z") 'haskell-interactive-switch)
  (define-key haskell-mode-map (kbd "C-c C-n C-t") 'haskell-process-do-type)
  (define-key haskell-mode-map (kbd "C-c C-n C-i") 'haskell-process-do-info)
  (define-key haskell-mode-map (kbd "C-c C-n C-c") 'haskell-process-cabal-build)
  (define-key haskell-mode-map (kbd "C-c C-n c") 'haskell-process-cabal)
  (define-key haskell-mode-map (kbd "SPC") 'haskell-mode-contextual-space)))
(eval-after-load 'haskell-cabal '(progn
  (define-key haskell-cabal-mode-map (kbd "C-c C-z") 'haskell-interactive-switch)
  (define-key haskell-cabal-mode-map (kbd "C-c C-k") 'haskell-interactive-mode-clear)
  (define-key haskell-cabal-mode-map (kbd "C-c C-c") 'haskell-process-cabal-build)
  (define-key haskell-cabal-mode-map (kbd "C-c c") 'haskell-process-cabal)))
```
Note that the key bindings deviates from [the ones in the wiki](https://github.com/haskell/haskell-mode/wiki/Haskell-Interactive-Mode-Setup). The reason to do so is to avoid conflicts with key bindings in `ghc-mode` and HaRe.

A piece of advice: if you are using a modern version of Cabal (more than 1.18), you should use its integrated REPL capabilities instead of `ghci`. This will ensure that your projects stay sandboxed, instead of polluting the global database. To enable it, add
```lisp
(custom-set-variables '(haskell-process-type 'cabal-repl))
```
to your personal configuration file. However, if your personal configuration file already _has_ a `custom-set-variables` command, you'll need to instead add a blank space and then `'(haskell-process-type 'cabal-repl)` before its closing parenthesis.

As always, once you're done editing this file, save it and `M-x eval-buffer RET` to apply the changes to your running Emacs session.

In order to use the rest of the features in `haskell-mode`, you need to establish a connection with an interpreter, which can be then queried for information about your file. This is called *loading a file*, and it's done by running `C-c C-l` in the file. You will be asked about the root of the Cabal project, which should be auto-configured for you anyway, and then you will get a interpreter window with a `λ>` prompt.

You will get an interpreter, except in the case that your file has some errors that the Haskell mode can automatically fix for you! For example, if you include a strictness pattern in your file, but do not enable the corresponding GHC extension, you will receive a message like:
```
Add {-# LANGUAGE BangPatterns #-} to the top of the file? (y or n)
```
If you have errors that cannot be fixed automatically, the key combination `C-c C-z` will navigate around them.

Note that the interpreter is a normal Haskell one, so you can run any command you want in the `λ>` prompt. `haskell-mode` contains many small features which make it more appealing to look and traverse Haskell values. Most of them do not need any special setup, except for my favourite: presenting a variable. To enable it, install the `present` package:
```
cabal install present
```
Now, in the interpreter window you can ask to present any value, even infinite ones:
```
:present [1 ..]
```
Instead of an infinite computation, what you get is an interactive value, where you can click to evaluate one step further.

Note for newcomers to Emacs: apart from using the mouse and the menus, Emacs offers two powerful ways to move between buffers (editor instances). The easiest one is using `C-x o`, which cycles between all buffers which are currently shown on the screen. The other possibility is using `C-x b`, which asks for a buffer name (usually, the name of the file) and gives focus to it, even if it was not visible beforehand. As with any Emacs prompt, you can just press Enter without entering any text to select the default one, if a default is shown in parentheses as part of the prompt.

Once you have started the session, your Haskell file can be queried in even more powerful ways. If you select or stay over an expression, you can query its type using `C-c C-n C-t` and extra information via `C-c C-n C-i`. In order to remember those key combinations, remember that `C-c` is used for all the commands in this article, `C-n` stands for i*n*teractive, and then `C-t` or `C-i` come from the first letter of `type` or `info`.

`haskell-mode` also integrates with Cabal in many interesting ways. Issuing the command `C-c C-n C-c` will run `cabal build` in your project and show the output messages in the interpreter window. Or if you want to run any other command, use `C-c C-n c` and you will be asked for the command. So now you have a complete development environment which builds and highlights errors for you! Actually, the support for Cabal is even greater, because `haskell-mode` also integrates a mode for Cabal files.

In some cases, loading the interactive mode will be turn to be very cumbersome. For a simpler interface, which still shows the errors in your file, but does not allow interaction with the code in a REPL, you can use `haskell-mode` compilation mode. To add a keybinding for it, include the following in your configuration file:
```lisp
(eval-after-load 'haskell-mode
  '(define-key haskell-mode-map (kbd "C-c C-o") 'haskell-compile))
(eval-after-load 'haskell-cabal
  '(define-key haskell-cabal-mode-map (kbd "C-c C-o") 'haskell-compile))
```
While editing a Haskell or Cabal file, you can press `C-c C-o` to invoke the compiler and look at the warnings and errors. Note that if you don't want to use the interactive features in `haskell-mode`, my suggestion is to instead use `ghc-mod`, which performs this compilation in the background for you, instead of you having to explicitly invoke it.

### Debugging

`haskell-mode` integrates with the debugger found in the GHC interpreter from version 7 on. You can find the documentation about this feature in [its page in the official wiki](https://github.com/haskell/haskell-mode/wiki/Haskell-Interactive-Mode-Debugger).

### Further customization

Above we have configured many `haskell-mode` options by changing the personal configuration file by hand. This is a very powerful way to do so, but at the same time quite simple to put together from different snippets. However, I should also mention that Emacs also has a _graphical_ interface to change many of the customization options. To open it, press `M-x customize`.

At that point, you will be presented with a list of customization groups, each of which groups together options related to some aspect of the editor or some specific mode. In our case, you need to go to _Programming_, then _Languages_, and finally _Haskell_. Every option which you may configure through this interface comes with a small description. Feel free to look around and see if you can make `haskell-mode` fit your own workflow.

## `ghc-mod`

**Important note**: the current `ghc-mod` version in Hackage may not work correctly under GHC 7.10. [This GitHub issue](https://github.com/kazu-yamamoto/ghc-mod/issues/437) describes alternatives, including downloading the source code directly from the repo. In my opinion, the easiest path now is to remain with GHC 7.8 if possible.

In contrast with `haskell-mode`, [`ghc-mod`](http://www.mew.org/~kazu/proj/ghc-mod/en/) needs an extra executable to work. It is available on Hackage, so you can install it by directly writing on a terminal:
```
cabal install ghc-mod
```
The executable created must be on the search path of Emacs, as discussed before with other tools. Remember to include in your configuration file, if you haven't done it yet, the following lines when on a Linux or Mac OS X system:
```lisp
(let ((my-cabal-path (expand-file-name "~/.cabal/bin")))
  (setenv "PATH" (concat my-cabal-path ":" (getenv "PATH")))
  (add-to-list 'exec-path my-cabal-path))
```
On Windows machines, the code is slightly different, a `:` becomes a `;`:
```lisp
(let ((my-cabal-path (expand-file-name "~/.cabal/bin")))
  (setenv "PATH" (concat my-cabal-path ";" (getenv "PATH")))
  (add-to-list 'exec-path my-cabal-path))
(custom-set-variables '(haskell-tags-on-save t))
```
The next step is installing the package in Emacs. To remind you of the process (similar to installing `haskell-mode`):
  * Press `M-x package-install`
  * Write `ghc` as the name of the package to install
  * Et voilà!
If you are using `ghc-mod` from another source than Hackage (e.g., Stackage), it is possible that the `ghc` package from the package manager may not work for you. In that case, it is better to rely on the files that come with the exact `ghc-mod` distribution that you are installing. For that, you need to include the following line in your configuration file:
```lisp
(add-to-list 'load-path "~/.cabal/share/ghc-mod-x.y.z")
```
where the `x.y.z` part correspond to the version of `ghc-mod` in your system.

Finally, you need to configure `haskell-mode` to initialize `ghc-mod` each time you open a Haskell file. To do so, open your configuration file and insert:
```lisp
(autoload 'ghc-init "ghc" nil t)
(autoload 'ghc-debug "ghc" nil t)
(add-hook 'haskell-mode-hook (lambda () (ghc-init)))
```
After you `M-x eval-buffer RET`, each time you open a Haskell file, both `haskell-mode` and `ghc-mod` will be available for you.

The main difference between `haskell-mode` and `ghc-mod` is that the former needs you to load files explicitly, whereas `ghc-mod` runs always in the background, checking and giving suggestions directly in the file. On my daily routine, I use `ghc-mod` to highlight and query information about my file, and only move to `haskell-mode` interactive features for testing in the interpreter or working with Cabal.

In order to discover the features of `ghc-mod`, create a new empty file (for those newcomers to Emacs, this is achieved with `C-x C-f`). Now include the following content:
```haskell
module Example where

import Data.Ma

f = Just
```
And save the file. If `ghc-mod` is correctly loaded, you should see a red line under the line `import Data.Ma`, and a `!` sign at the left margin. This means that there is an error on this line. If you want to read the error, you have two options:
  * Move the mouse to the line, and the error will be shown in a tooltip.
  * Navigate to the error using `M-n` or `Esc n` (both for the next error) and `M-p` or `Esc p` (both for the previous one). Then press `M-?` and the error will be shown in a new buffer.
Correct the error by changing `Ma` to `Map` and save the file (remember, `C-x C-s`). The `ghc-mod` extension will check the file again (it does so only when saving a file) and now highlights the lines in a yellow color with a `?` symbol at the margin. This means that the lines have warnings associated with them. You can navigate through them as you did with errors. But note that warnings will only be shown if no errors are present in the file.

Apart from the GHC compiler, there is another tools that helps writing good Haskell code, namely [HLint](http://community.haskell.org/~ndm/hlint/). HLint provides suggestions to help you write more idiomatic and concise code. To use it, you need to install the corresponding executable:
```
cabal install hlint
```
After doing so, you can change whether `ghc-mod` checks the file with the GHC compiler or with HLint by pressing `C-c C-c`. HLint suggestions will be shown as warning, which you can navigate normally.

In a similar way to `haskell-mode`, you can query about the type or information of an expression by issuing `C-c C-t` or `C-c C-i` respectively. In this case, there is no need to run interactive mode, because `ghc-mod` takes care of looking for the information. One nicety is that by pressing `C-c C-t` several times, you can enlarge the expression which is queried automatically. That is, if you write `Just 3`, stay over the `3` and press `C-c C-t`, `Integer` will be shown. If you press `C-c C-t` again, the entire `Just 3` is selected and the type is shown as `Maybe Integer`.

If you need more information about an identifier, you can ask `ghc-mod` to show the corresponding documentation in the browser. Just focus over the expression you care to know more about and press `Esc C-d` (`M-C-d` should also work, but it is usually captured by window managers in Linux systems). Your default browser will be executed showing the on-line documentation of the module or identifier that you requested.

Sometimes you don't know the name of the function you need to use, but have some information like its type. In that case, you can use [Hoogle](http://www.haskell.org/hoogle/) to search in Hackage for functions. In order to do so locally, you need to install the corresponding helper executable and populate its database by running in a terminal:
```
cabal install hoogle
hoogle data
```
Now, at any moment you can press `C-c C-h` and query Hoogle for the information you want.

### Insertion commands

`ghc-mod` also helps you to write your code. At any point, you can press either `Esc Tab` or `M-C-i` (both key combinations are equivalent) in a not-finished name and a list of possible completions will be shown. In the new buffer, select a completion and press Enter or click on it to insert it. Of course, in the case where only one possibility is available, it will be automatically inserted.

In some cases, the problem is that you have used a name, but haven't included the corresponding module import. In many cases, the compiler even gives you a hint as to the line to include. `ghc-mod` integrates into this feedback loop, and by issuing `M-C-m` or `Esc C-m` you can automatically insert the import. Note that this augments the functionality in `haskell-mode` by not needing to load the file to make these small fixes.

In many cases, you can derive the shape of a function or type class instance from the signature or declaration. So, it would be nice to have commands which given that information generates some initial skeleton. Well, `ghc-mod` includes it! As an example, include in your file a new data type and the declaration of a new instance:
```haskell
data ExampleType = ExampleType Int Int

instance Eq ExampleType where
```
Now move to the `instance` declaration and press `C-u M-t`. Automagically, the declaration is expanded into:
```haskell
instance Eq ExampleType where
  x == y = _body
  x /= y = _body
```
Code generation also works with function signatures. For example, let's write the signature of a `fmap`-like function for `Maybe`s:
```haskell
maybeMap :: Maybe a -> (a -> b) -> Maybe b
```
Press `C-u M-t` once again and the skeleton of the function appears:
```haskell
maybeMap x f = _maybeMap_body
```
Note that this code generation assumes that you want to generate functions its arguments fully stated. That is, if you prefer to write `maybeMap x = ...` and return a functional value, you have to write the skeleton yourself.

This functionality does not end there. Apart from code generation, `ghc-mod` can also generate the neccessary pattern matching, by splitting a variable into its possible constructors. For example, stay over the `x` on `maybeMap` and press `M-t` (to complete *t*emplate). Automagically, the code becomes:
```haskell
maybeMap Nothing  f = _maybeMap_body
maybeMap (Just x) f = _maybeMap_body
```
You can split on every algebraic data type, be it from a library or defined in your own files. Note that in the case of a data type defined as a record, the match will be written using record syntax with all the fields explicitly shown.

Another task where `ghc-mod` adds some niceties to `haskell-mode` is indentation of blocks. By using `C-c <` and `C-c >` you can indent a region less or more, respectively, respecting Haskell layout rules. Note that below we discuss `structured-haskell-mode`, which provides a more powerful way to deal with Haskell code respecting the scoping and indentation rules.

### Completion with `company-ghc`

As stated above, `ghc-mod` provides a simple completion mechanism with `Esc Tab`: it uses an integrated feature of Emacs, called the minibuffer, which is not very featureful. Instead, you can add [`company-ghc`](https://github.com/iquiw/company-ghc) to your mix and get nicer graphical completion features.

![`company-ghc` in action](https://github.com/iquiw/company-ghc/blob/master/images/doc-buffer.png)

In order to get `company-ghc`, you first need to install it using the package manager. As a reminder, press `M-x package-install` and when asked, tell the system to install `company-ghc`.

The `company-ghc` name comes from the fact that it makes use of [`company-mode`](http://company-mode.github.io/), a Emacs minor mode which helps building completion features. Note that the package manager ensures that `company-mode` is also installed when installing `company-ghc`, so you don't need to worry about that.

There are two ways to enable `company-mode` for Emacs. The first one is enabling it for all types of files. Note that this will show completion in every other kind of file in your system, so it may not be what you wanted. To have completions via `company-mode` globally, add to your configuration file the following lines:
```lisp
(require 'company)
(add-hook 'after-init-hook 'global-company-mode)
```
Another possibility is using `company-mode` only in Haskell files. To do so, you should add instead:
```lisp
(require 'company)
(add-hook 'haskell-mode-hook 'company-mode)
```
In any case, you need to instruct `company-mode` to get completions from `ghc-mod` by adding at the end:
```lisp
(add-to-list 'company-backends 'company-ghc)
(custom-set-variables '(company-ghc-show-info t))
```

Now, the moment for magic has come! `M-x eval-buffer RET` and open any Haskell file. Write the first three letters of a function name, like `spl`, and wait a moment. A menu with possible completions should appear: you can move between possibilities using arrows, and select something to insert with Enter. This works not only with functions, but also with module names, and even language extensions.

If you took some time to install and download the data for Hoogle, a nice extra comes bundled with `company-ghc`. While navigating through the completion possibilities, you can press `F1` and get the documentation for the corresponding element. The information will disappear once you choose an option.

### Working with holes

If you are using `ghc-mod` on GHC 7.8 or greater, and ran the code generation and splitting, you may have noticed that some strange elements starting with an underscore appear. These are *holes*, the way in which to tell the compiler that you know that an expression is missing, but still want it to tell you about possible errors or warnings. The nice thing is that GHC spits out a lot of information about each hole in a file, and `ghc-mod` benefits from that.

Holes are highlighted in files with a purple line below them. You can navigate between holes as you did with errors and warnings, but you can also use specific key combinations for them. In this case, they are `C-c M-n` and `C-c M-p` (the same as for the errors and warnings, but with `C-c` before them). If you do so, a new buffer appears with the information obtained from GHC:
```
Found hole ‘_maybeMap_body’ with type: Data.Maybe.Maybe b
Where: ‘b’ is a rigid type variable bound by
           the type signature for maybeMap
Relevant bindings include
  f :: a -> b
  maybeMap :: Data.Maybe.Maybe a -> (a -> b) -> Data.Maybe.Maybe b
```
Note that the previous output is cropped from the one given by GHC, which contains much more information, along with position of the identifiers. You will get all that information while working with `ghc-mod`.

Of course, your aim should be to get rid of all holes. Type-oriented programming is a good-way to do so: realize part of the expression by looking at the type given in the hole message and add things to the hole until you are finished. To support this loop, `ghc-mod` includes a refine command, available at the key combination `C-c C-f`. When you press this over a hole, the system asks for an expression to refine with, and replaces the hole with the expression given followed by as many holes as necessary to fulfill the corresponding type. For example, if in the second equation of `maybeMap`:
```haskell
maybeMap (Just x) f = _maybeMap_body
```
you refine using `Just`, the code changes to:
```haskell
maybeMap (Just x) f = Just _maybeMap_body_1
```
because you still need a value for the function to return what you expect.

This is nice, but in some cases `ghc-mod` can do even more for you: it can write your whole expression! It does so by leveraging the power of [Djinn](http://hackage.haskell.org/package/djinn). For example, let's go back to the definition of `maybeMap` after splitting:
```haskell
maybeMap Nothing  f = _maybeMap_body
maybeMap (Just x) f = _maybeMap_body
```
If you press `C-c C-a` in each of the holes, several options for the code to be written there will be shown, including `Nothing` in the first case, and `Nothing` and `Just x` in the second case. You just need to select the code you want to include from a list, and it will be automatically completed. Note that this functionality becomes very handy when you need to work with expressions involving currying and tupling, because it takes care of obtaining a correctly-typed expression for you.

Note that in order to prevent non-termination, automatic completion will not look for recursive definitions. However, you can still take advantage of `ghc-mod` by refining those places where recursion is needed, and then using automatic completion for simpler holes.

### Refactoring with HaRe

We have talked about changing the code on a file automatically, but none of these changes could be considered a real refactoring step. However, there is a tool called [HaRe](https://github.com/alanz/HaRe), which is integrates with `ghc-mod` and provides those features.

Note: HaRe does not work at the moment of writing on GHC 7.8, and nor is its package yet uploaded to MELPA. This tutorial will be updated with the entire installation procedure when those problems are fixed. Until now, follow the instructions in the [official webpage](https://github.com/alanz/HaRe) up to the first block on Emacs integration.

HaRe must be loaded at the same time as `ghc-mod`. To do so, open your configuration file and change the line reading:
```lisp
(add-hook 'haskell-mode-hook (lambda () (ghc-init)))
```
to read:
```lisp
(add-hook 'haskell-mode-hook (lambda () (ghc-init) (hare-init)))
```
After doing so, in every Haskell file you will see a new menu called *Refactor*, containing the current refactorings. In each menu item you can find the corresponding key combinations: notice that all start with `C-c C-r`. Since the list of refactorings are growing every day, we refrain here from explaining all of them and focus on one specific case.

The most common case for refactoring is renaming. This is done by issuing the command `C-c C-r r` over the element to rename. The system asks for the new name, and then generates the refactoring script. Note that the refactoring is *not directly applied*. Instead, a new buffer shows the modifications that would be done in the files belonging to your project, and you can accept or decline them. This buffer works under the [`ediff` mode](https://www.gnu.org/software/emacs/manual/html_node/ediff/).

## `structured-haskell-mode`

Working with Haskell is usually quite aesthetically appealing because of the layout rule (remember that this rule is the one delimiting the blocks and scopes in Haskell code). However, maintaining the rule can become cumbersome after some time, especially if you are editing something in an inner part of your syntax, which makes the rest of the block become unaligned.

To handle Haskell code in a way that respects Haskell layout rule and conventions, we have [`structured-haskell-mode`](https://github.com/chrisdone/structured-haskell-mode), in short SHM.

Installing it is as simple as the rest of packages in this article: execute `M-x package-install` and select `shm`.  In addition, you need the `structured-haskell-mode` package from Hackage:
```
cabal install structured-haskell-mode
```
It is important now to tell `haskell-mode` that the indentation should be handled via SHM instead of itself. To do so, open your configuration file, and change the line resembling:
```lisp
(add-hook 'haskell-mode-hook 'turn-on-haskell-indentation)
```
to use SHM instead:
```lisp
(add-hook 'haskell-mode-hook 'structured-haskell-mode)
```

Note that now when opening a Haskell file you see a new minor mode, represented by `SHM` in the bottom of the buffer. If the file is in syntactically-correct Haskell, when staying over an expression you should see it highlighted with a different background. This is the *current node* in which SHM is working. If you want to select a larger portion of the code, you can enlarge the scope of your highlighting using `M-a`.

You will notice that when you write some code, such as the beginning of a `case` statement, the code changes to include a complete Haskell expression instead, with holes to fill inside. Try to do so, and you will get something like:
```haskell
case undefined of
  _ -> undefined
```
How you would fill now the structure is by moving to the `undefined` and `_` places and adding some code, such as:
```haskell
case n of
  0 -> 1
```
After doing so, instead of pressing Enter and then Tab, press `C-j` (which is known as *newline and indent*). It will take care of adding a new node in the correct place, which should be indented at the point of `0`. But if instead you had a call to a function which expects more arguments, the indentation would go into that point. The types of the expressions are thus taken into account when indenting.

Furthermore, if you know try to add something new before the `case` statement, such a call to `maybeMap` defined previously, you will notice that the entire `case` block moves! In that way, the nice layout you had is respected :)

Note than SHM contains some extra commands to manipulate Haskell programs in an easier way, apart from the basics `M-a`, `C-j` and autocompletion. You can read the entire list, along with graphical explanations, at its [website](https://github.com/chrisdone/structured-haskell-mode).

My piece of advice is that SHM can be difficult to master at first. If you feel like you need a more normal editor, just revert back the changes to your configuration file and use normal indentation from `haskell-mode`.

## Other niceties

There are tons of other packages available for Emacs. Without prejudice to any of the others, I would like to speak about three packages which I feel that integrate very well with Haskell programming.

The first one is [`rainbow-delimiters`](https://github.com/jlr/rainbow-delimiters). Its goal is very simple: show each level of parenthesis or braces in a different color. In that way, you can easily spot until from point some expression scopes. Furthermore, if the delimiters do not match, the extra ones are shown in a red, warning color. To get it just install `rainbow-delimiters` using the Emacs package manager, and add to your configuration file:
```lisp
(require 'rainbow-delimiters)
(add-hook 'haskell-mode-hook #'rainbow-delimiters-mode)
```

The second one is [Magit](http://magit.github.io/), which provides Git integration with Emacs. It is also available in MELPA, and it's only one `M-x package-install` away. One nice thing is that it shows the Git status of your file at the bottom of the buffer. Magit is a very big package, so the reader is suggested to look at its official webpage.

Finally, you may want to change the default color scheme in Emacs. From version 24 on, this is done very easily using its theme support. For example, I am quite a fan of the [Solarized themes](http://ethanschoonover.com/solarized). In MELPA you can find a lot of these and instructions on how to install them.

## Summary

An Emacs configuration file with all the features discussed in this article can be found [in this repository](https://github.com/serras/emacs-haskell-tutorial/blob/master/dot-emacs.el). Depending on your current configuration, you may need to change some of the values from the ones shown in that file.

### List of key bindings

Key binding                       | Description
----------------------------------|-----------------------------------------------------------
`F8`                              | jump to imports
`C-c C-.`                         | sort and align imports
`M-.`                             | jump to definition
`M-x haskell-mode-stylish-buffer` | format file
`C-c C-l`                         | load file in interpreter
`C-c C-z`                         | navigate errors in the file (`haskell-mode`, interactive)
`C-c C-n C-t`                     | show type of expression (`haskell-mode`, interactive)
`C-c C-n C-i`                     | show info of expression (interactive)
`C-c C-n C-c`                     | run `cabal build` (interactive)
`C-c C-n c`                       | run any other `cabal` command (interactive)
`M-x haskell-debug`               | start debugger
`M-n` or `Esc n`                  | go to next error or warning
`M-p` or `Esc p`                  | go to previous error or warning
`M-?` or `Esc ?`                  | show error or warning information
`C-c C-c`                         | change checking between GHC and HLint
`C-c C-t`                         | show type of expression (ghc-mod)
`C-c C-i`                         | show info of expression (ghc-mod)
`M-C-d` or `Esc C-d`              | show documentation of expression
`C-c C-h`                         | search using Hoogle
`M-C-i` or `Esc C-i`              | auto-completion
`M-C-m` or `Esc C-m`              | insert module import
`C-u M-t`                         | initial code generation
`M-t`                             | perform case split
`C-c <`                           | indent region shallower (`ghc-mod`)
`C-c >`                           | indent region deeper (`ghc-mod`)
wait                              | show completions (`company-ghc`)
`C-c M-n`                         | go to next hole
`C-c M-p`                         | go to previous hole
`C-c C-f`                         | refine hole
`C-c C-a`                         | automatically fill hole
`C-c C-r`                         | refactoring command prefix
`C-c C-r r`                       | rename refactoring
`M-a`                             | go to parent node (SHM)
`C-j`                             | newline and indent

### Installing all packages

This is the Haskell part of the modes using Cabal:
```
cabal update
cabal install happy hasktags stylish-haskell present ghc-mod hlint hoogle structured-haskell-mode hindent
```
The Emacs packages to be installed using `M-x package-install` are `haskell-mode`, `hindent` (if wanted), `ghc`, `company-ghc` and `shm`.

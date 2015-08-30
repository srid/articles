% Mouseless window switching
%
% January 10, 2010

*This article was originally posted as an answer at [superuser.com](http://superuser.com/a/97153/1443).*

Switching to most frequently used applications is best done by keyboard shortcuts. Here is how I do it on each of the three major platforms.

# GNU/Linux

I use GNOME as the desktop environment. Avoiding the use of mouse in areas where the same function can be carried out much faster using the keyboard is highly recommended for the added productivity it offers.

For example, if you are using an external monitor configured using [TwinView][2], it takes a while to move the mouse pointer from a window in your laptop display to a window in the external monitor. If your monitor resolution is high, then it takes even more time.

What follows is a list of functions which are usually done using the mouse, but have an equivalent keyboard-centric approach as documented here.

## Switching to a particular window

Let's say that you have about 10 windows open and want to switch to a particular window. [80/20 rule][3] applies here - most window switches you do are for a small subset of all possible windows. In my case, I more often switch to three applications: Emacs, Chrome and Terminal. It is thus more useful to bind predefined keys to these windows.

The following key combination, when pressed will activate the corresponding window.

    ctrl + alt + u: Chrome
    ctrl + alt + k: Terminal
    ctrl + alt + j: Emacs

These are the most convenient shortcuts for me, but you can assign different keys as you wish.

The only question that remains is how do we do this? If you are using Sawfish, for instance, this is a no brainer task. But for other underpowered window managers like Metacity (default in Ubuntu), there is a solution: **[wmctrl][4]**. On Ubuntu, you can use apt-get to install wmctrl. After installing, try running the following commands:

    $ wmctrl -a Chrome
    $ wmctrl -a Emacs
    $ wmctrl -a Terminal
    
The -a option activates the window whose title matches with the argument given. 

`wmctrl` may not always work, in which case, you may try xdotool:

    xdotool search --all --class --onlyvisible --limit 1 "Terminal" windowactivate

### Option #1 - Ubuntu way

Go to `System Settings > Keyboard > Shortcuts > Custom Shortcuts` and add custom shortcuts to run the appropriate `wmctrl` commands. Done!

### Option #2 - xbindkeys

Install **xbindkeys** using apt-get and start writing the config file ~/.xbindkeysrc. The following is my configuration:

    "wmctrl -a Firefox"
      m:0xc + c:30
      Control+Alt + u
    
    "wmctrl -a Terminal"
      m:0xc + c:44
      Control+Alt + j
    
    "wmctrl -a emacs"
      m:0xc + c:45
      Control+Alt + k
  
I usually use the xbindkeys -k command to come up with all those numeric codes you see above. For example, m:0xc corresponds to the Control+Alt key combination. You can also use xbindkeys-config, a graphical configuration utility, to create ~/.xbindkeysrc.

You may also consider adding xbindkeys to GNOME Session Preferences to ensure automatic startup on every boot.

# Microsoft Windows

The same can also be done on Microsoft Windows using a program called **[AutoHotkey][8]**.

Here's the AutoHotKey script I use on my Windows-based laptop:

    ; match window title anywhere
    SetTitleMatchMode 2
    
    ^!u::WinActivate Opera
    ^!j::WinActivate ActiveState Komodo
    ^!k::WinActivate sridharr@double
    ^!h::WinActivate Mozilla Thunderbird
	
# Mac OS X

**Update (Feb 2015)**: I now use [Apptivate](http://www.apptivateapp.com) on OS X.

On Mac, there is no Unixy way to assign global keyboard shortcuts (eg: xbindkeys) ... but there are several workarounds. Thanks to [this serverfault post][9], I found **[Quicksilver][10]** to be a good enough way to assign keyboard shortcuts to activate specific applications.

For detailed instructions on assigning global keyboard shortcuts, [follow this post][11]. As the settings will be saved to the file ~/Library/Application Support/Quicksilver/Triggers.plist, you can easily move it around or symlink it to your [Dropbox][12] directory.


# Emacs

**Update (Feb 2015)**: I now recommend [Helm instead of ido](https://github.com/emacs-helm/helm).

Emacs has the excellent [ido mode][6] that enables you to interactively fuzzy match buffer names when switching buffers. Normally, one presses C-x b in order to bring up the minibuffer and then types the buffer name manually with tab completion. With ido mode, typing 'ny', for example, will match the buffer main.py; and it does that interactively without you having to press Enter key. Use the following elisp code in your .emacs after adding ido.el to your path:

```elisp
;; Buffer switching
(require 'ido)
(ido-mode t)
(setq ido-enable-flex-matching t)
(global-set-key (kbd "M-i") 'ido-switch-buffer)
```

Now press Alt+i to switch buffers interactively.



  [1]: http://evernote.com/pub/srid/blog#v=t&n=10b49a23-cd12-4da5-98ae-11352c65b5bf&b=0
  [2]: http://www.nvidia.com/object/feature_twinview.html
  [3]: http://en.wikipedia.org/wiki/Pareto_principle
  [4]: http://tripie.sweb.cz/utils/wmctrl/
  [5]: https://wiki.mozilla.org/Labs/Ubiquity
  [6]: http://www.emacswiki.org/emacs/InteractivelyDoThings
  [8]: http://www.autohotkey.com/
  [9]: http://serverfault.com/questions/39156/xbindkeys-for-mac
  [10]: http://blacktree.com/?quicksilver
  [11]: http://hackaddict.blogspot.com/2007/07/setup-global-keyboard-shortcuts-to-open.html
  [12]: https://www.getdropbox.com/referrals/NTg3MDQ1OQ


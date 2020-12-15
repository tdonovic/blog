+++
author = "tdonovic"
date = 2020-12-15
title = "apple watch PAM"
+++
I noticed the ~weirdest~ coolest thing when using an older (non-touch id) mac with my Apple Watch. When I had to authenticate to, for example, allow `sshfs` to run, I got prompted to double tap the button on my Apple Watch, in much the same manner as if you are authenticating to use it for Apple Pay. This then unlocked the gear icon thing, and I could run `sshfs`. 

![watch prompt](watch_prompt.png#center)

This got my thinking, however, is there some way to make it when I run `sudo` that I just get a watch prompt instead? Could you do this with TouchID if your mac supported it? The answer?

Of course you can! With PAM!

## What is PAM?
PAM (Pluggable Authentication Modules) are a means of proving your identity to the system, other than putting in a boring password. PAM can be used, for example to unlock your computer with a YubiKey. In our case, we are going to exploit PAM to authenticate `sudo` instead.

## pam-watchid
Thanks to [biscutehh's pam-watchid](https://github.com/biscuitehh/pam-watchid) project, a fork from [Reflejo's pam-touchID](https://github.com/Reflejo/pam-touchID), all you need to do is clone the repo, `make install` and modify your `/etc/pam.d/sudo` file. Then you can use Watch to sudo! I presume this will also work with TouchID with the other project, although I was unable to test, since my Mac is too old.


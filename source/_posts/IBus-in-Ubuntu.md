---
title: IBus on Ubuntu
date: 2024-05-06 20:45:13
description: How to install IBus input method on Ubuntu?
categories:
- Linux
tags:
- Linux
- Ubuntu
- Input Method
---

# Install Chinese Input Method

If you choose English when you install ubuntu, there is not a Chinese input method installed, but you can install it with commands in the terminal.

## Update Your System

Before you install input method, you should update your software.

If you want to stay in 20.04(or what you are using), **DON'T** upgrade your system. Just update the software.

<img src="not_upgrade_1.png" style="max-width:50%">
<img src="not_upgrade_2.png" style="max-width:50%">

Update your software repository and upgrade your software  
Use `ctrl`+`alt`+`t` to start a terminal, and run the command to update
```sh
# Update the software repository
$ sudo apt update
# Update the software
$ sudo apt upgrade
# If failed, type `sudo apt upgrade --fix-missing` to try again.  
# Restart your computer.
$ reboot
```

Or just click `Install Now` when the window pop up.

<img src="update_1.png" style="max-width:70%">

If you don't see the pop-up window, you still have 2 ways to start the update.

<img src="update_2.png" style="max-width:70%"> or <img src="Software_update.png" style="max-width:70%">
## Install the Pinyin Input Method

Install ibus-pinyin
```sh
$ sudo apt install ibus-pinyin
```
You'd better restart your system or log out and log in again for the ibus to load its profile again.

## Setup IBus

So what is **IBus**?

[IBus](https://help.ubuntu.com/community/ibus)
---
The Intelligent Input Bus (IBus) is an input method framework for multilingual input in Unix-like operating systems. It's called "Bus" because it has a bus-like architecture.

```sh
$ ibus-setup
```
Click `Input Method`-`Add` and choose Pinyin (or Bopomofo if you like to).

<table>
    <tr>
        <td ><center><img src="Add_Chinese_Input_Method.png" style="max-width:100%">
        <td ><center><img src="Add_Chinese_Input_Method_2.png" style="max-width:100%">
        <td ><center><img src="Add_Chinese_Input_Method_3.png" style="max-width:106%">
        </tr>
</table>

## Add Input Source
Add input source in system settings by `Settings`-`Region & Language`-`Input Sources`, click the `+` and add Chinese input method.
<table>
    <tr>
        <td ><center><img src="settings.png" style="max-width:100%">
        <td ><center><img src="Region_and_Language.png" style="max-width:80%">
    </tr>
</table>
<img src="Region_and_Language_3.png" style="max-width:50%">

Click Super(Win)+Space to switch your input method to Chinese and you can happily type Chinese.


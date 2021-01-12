---
title: Get SteelSeries Artic 9's Bluetooth to work with Arch
author: Karl Webster
date: 2021-01-06
categories: [Arch Linux, Bluetooth, Headphones]
tags: [Arch Linux, Bluetooth, Headphones]
---

## Why

So I rencently purchased a new set of SteelSeries Artic 9's. I really liked my 7's and when they broke after 2 years of almost non stop use, I thought I would upgrade.

The 9's have bluetooth which would make using it with my Home PC and my Worklaptop a breeze.


## Arch

Of course as I am using Arch Linux its not quite as simple as put the headset into pairing mode and connect...

### Pre Req's
I had to install the following packages first:

```bash
pacman -S pulseaudio-bluetooth pulseaudio-alsa bluez bluez-utils pavucontrol blueman
```

Then I had to start the service and ensure it would persist on reboot:
```bash
systemctl enable bluetooth --now
```

### Pairing
Run `bluetoothctl` and copy the steps below (You can TAB auto complete):
```bash
$ bluetoothctl
power on

$ bluetoothctl
scan on

$ bluetoothctl
pair MAC-For-Headset

$ bluetoothctl
connect MAC-For-Headset

$ bluetoothctl
exit
```

## It sounds awful?

That is because in blueman you want to change the settings, you will have 2:

High fidelity - Listening to music

Headset - This will activate the heatset mic
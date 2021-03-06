---
title: PulseAudio
permalink: /PulseAudio/
---

*Note:* This page is work in progress. Until finished it is only of limited usefulness.

Network
=======

Server configuration.

` server $ cat ~/.pulse/default.pa`
` load-module module-augment-properties`
` load-module module-null-sink`
` load-module module-always-sink`
` load-module module-udev-detect`

` load-module module-device-restore`
` load-module module-stream-restore`
` load-module module-card-restore`
` load-module module-default-device-restore`

` load-module module-native-protocol-unix`
` load-module module-native-protocol-tcp`

` server $ pulseaudio`

Client configuration.

` client $ cat ~/.pulse/default.pa`
` load-module module-augment-properties`
` load-module module-null-sink`
` load-module module-always-sink`
` load-module module-udev-detect`

` load-module module-device-restore`
` load-module module-stream-restore`
` load-module module-card-restore`
` load-module module-default-device-restore`

` load-module module-native-protocol-unix`
` load-module module-native-protocol-tcp`

` client $ scp server:.pulse-cookie .pulse-cookie`
` client $ pulseaudio`

Applications
============

MPlayer
-------

Enable Pulse Audio support in MPlayer.

` $ cat <<EOF`
` let`
`   pkgs = (import `<nixpkgs>`) {};`
` in`
`   pkgs.MPlayer.override { pulseSupport = true; }`
` EOF > ~/.nix-defexpr/mplayer.nix`

Install MPlayer with Pulse Audio support.

` $ nix-env -iA mplayer`

Make Pulse Audio the default output device.

` $ cat <`<EOF
  ao=pulse
  >`> ~/.mplayer/config`
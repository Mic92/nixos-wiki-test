---
title: Audio HOWTO
permalink: /Audio_HOWTO/
---

Getting ALSA to work
--------------------

-   on a console fire up alsamixer

` alsamixer`

-   you see plenty of vertical bars?
    -   you should be okay

<!-- -->

-   you see very few vertical bars and the sound card (top-left) is something like "PC Speaker"?
    -   hit the 'S' key, you should be able to switch to the "real" audio card (if not your audio card is likely to not being supported yet).
    -   when the real audio card is selected you should be viewing the "plenty vertical bars" thing.
        -   first thing to do is to disable pc speaker (kernel module "snd-pcsp", see below.

### Make your audio card the default ALSA card

Sometimes the pc-speaker is the default audio card for ALSA. You can make your real sound card default instead. For example, if your sound card is "hda-intel" then add

      boot.extraModprobeConfig = ''
        options snd slots=snd-hda-intel
      '';

to your /etc/nixos/configuration.nix.

Alternatively you can ...

### Disable PC Speaker "audio card"

edit /etc/nixos/configuration.nix and add "snd_pcsp" to boot.blacklistedKernelModules option:

` boot.blacklistedKernelModules = [ "snd_pcsp" ];`

Now reboot and retry from the beginning (i.e. check that your real card is shown by alsamixer without using the 'S' key).

### Other hardware specific problems

If you're mic is always at a very low volume or you experience other oddness and you have already checked the settings in alsamixer to make sure nothing is muted, and also any buttons on your computer (I have twice overlooked the mute button on laptops!) you may need to specify a particular card model when loading snd-hda-intel:

      boot.extraModprobeConfig = ''
        options snd-hda-intel model=YOUR_MODEL
      '';

You should be able to look up the available options for model in [1](http://www.kernel.org/doc/Documentation/sound/alsa/HD-Audio-Models.txt:HD-Audio-Models.txt). You can try them out interactively as follows:

1.  Close any applications using the sound card
    1.  See if any applications are using the sound card

        $ lsof /dev/snd/\*

        COMMAND PID USER FD TYPE DEVICE SIZE/OFF NODE NAME

        </pre>

        pulseaudi 14080 goibhniu 30u CHR 116,7 0t0 5169 /dev/snd/controlC0

        pulseaudi 14080 goibhniu 37u CHR 116,7 0t0 5169 /dev/snd/controlC0

    2.  Kill them

        for any process apart from pulseaudio you could just do:

        $ kill -9 14080

        but in the case of pulseaudio you have to prevent it from respawning itself automatically

        $ mkdir -P ~/.pulse && echo "autospawn=no" &gt;&gt; ~/.pulse/client.conf

        you can then stop pulseaudio with:

        pulseaudio -k \# or kill it by process id

2.  Unload the snd-hda-intel module

    rmmod snd-hda-intel

3.  \[<http://www.kernel.org/doc/Documentation/sound/alsa/HD-Audio-Models.txt:HD-Audio-Models.txt>: Look up the model options for your card\]
4.  Try each one

    modprobe snd-hda-intel model=3stack-6ch

5.  Test if this has fixed your problem (tip: aplay and arecord are alsa based command line tools you can use to quickly check)
6.  Repeat until you have exhausted all the options or have fixed your problem
7.  TIDY UP!

    Don't forget to re-enable pulse autospawning by changing "autospawn=no" to "autospawn=yes" in ~/.pulse/client.conf

### Fixing sound in KDE

(alsamixer shows my audio card by default, but i get no sound in KDE)

-   be sure to explicitly install pkgs.kde44.phonon in /etc/nixos/configuration.nix (environment.kdePackages option) or via nix-env -i

After this step you should be seeing xine and gstreamer backends in the systemsettings / Computer Administration / Multimedia 's Backends tab.

-   If you still see Pulseaudio as the only option in the main tab of systemsettings / Computer Administration / Multimedia, do the following:

` $ rm -rf ~/.pulse/*`
` $ killall5 -9 pulseaudio`

-   Log out of kde and re-login

Now you should be able to select your audio card, default hw0,0 etc as your audio device in systemsettings / Computer Administration / Multimedia

Test the audio and enjoy the relaxing KDE's arpeggio.

### Making PulseAudio work for most of apps *OBSOLETE*

The instructions described below can now be achieved with a configuration option:

    hardware.pulseaudio.enable = true

***The rest of this section will be removed shortly***

This routes default ALSA sound streams via PulseAudio

-   Install alsa-plugins(pkgs.alsaPlugins)
-   Put this into ~/.asoundrc:

<!-- -->

     pcm.pulse {
       type pulse
       }

       ctl.pulse {
       type pulse
       }

       pcm.!default {
       type pulse
       }

       ctl.!default {
       type pulse
       }

You can check if everything works by using pavucontrol to see the audio streams and make sure PulseAudio detects your audio hardware.

You may need to add your users to the audio group:

    $usermod -a -G audio myusername

If a user is not a member the audio group only a dummy device will appear in pavucontrol.

### Using JACK with pulseaudio

[Jack Audio Connection Kit](http://jackaudio.org:The) is used by most of the serious audio applications on Linux. It provides real-time, low latency connections for both audio and MIDI data between applications that implement its API. NixOS uses the dbus version of JACK2 (jackdbus). This can be used together with pulseaudio with a little configuration. The result is that you don't have to manually hunt down applications which are using the sound device and kill them before starting JACK. You can also continue to use non-JACK aware applications (e.g. flash) at the same time as using JACK applications (e.g. Ardour).

1.  Enable JACK support

    In your configuration file:

        environment.systemPackages = [ (pkgs.pulseaudio.override {jackaudioSupport = true;}) ];

2.  Ensure that the JACK enabled pulseaudio is being used

    ~/.pulse/client.conf

        daemon-binary=/var/run/current-system/sw/bin/pulseaudio

3.  Configure QjackCtl
    1.  Enable jackdbus

        Setup -&gt; Settings -&gt; Server Path: jackdbus

        [<File:qjackctl-settings.png>](/File:qjackctl-settings.png "wikilink")

        Setup -&gt; Misc -&gt; Enable D-Bus interface: check

        [<File:qjackctl-misc.png>](/File:qjackctl-misc.png "wikilink")

    2.  Load the jack modules for pulseaudio after starting jackdbus

        Setup -&gt; Settings -&gt; Options -&gt; Execute script after Startup: check

            pactl load-module module-jack-sink channels=2; pactl load-module module-jack-source channels=2; pacmd set-default-sink jack_out

        Setup -&gt; Settings -&gt; Options -&gt; Execute script on Shutdown: check

            pactl unload-module `pactl list|grep -A 3 jack-source|tail -1|awk '{ print $NF }'`;pactl unload-module `pactl list|grep -A 3 jack-sink|tail -1|awk '{ print $NF }'`

        [<File:qjackctl-options.png>](/File:qjackctl-options.png "wikilink")

You should now be able to start JACK with QjackCtl, you will notice a new playback and capture device in your sound mixer along with your normal devices.

[<File:kmix-pulseaudio-with-jack.png>](/File:kmix-pulseaudio-with-jack.png "wikilink")
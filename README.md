# Mic noise reduction

A webpage to help document a good mic noise reduction setup.

## Linux

### Pipewire setup

Pipewire + [Carla](https://kx.studio/Applications:Carla) will be used.

Echo cancellation to prevent echo when using speakers and mic
Latency must be a multiple of 480. Increasing latency will prevent Xruns in Carla.
Autoconnect to your mic. (Use target.object)

```
~/.config/pipewire/pipewire.conf.d $ cat 20-echo-cancel.conf 
 context.modules = [
  {   name = libpipewire-module-echo-cancel
      args = {
          # library.name  = aec/libspa-aec-webrtc
          node.latency = 1440/48000
          source.props = {
             node.exclusive = true
             node.name = "Echo Cancellation Source"
             target.object = "alsa_input.pci-0000_00_1b.0.analog-stereo"
          }
          sink.props = {
             node.name = "Echo Cancellation Sink"

          }
      }
  }
]

```
Some conferencing applications do not support jack input, thus we need a dummy duplex device to connect carla and those applications
```
~/.config/pipewire/pipewire.conf.d $ cat 30-virtualmic.conf 
context.modules = [
    {   name = libpipewire-module-loopback
        args = {
            audio.position = [ FL FR ]
            capture.props = {
                media.class = Audio/Sink
                node.name = my_sink
                node.description = "my-sink"
                audio.channels = 2
                audio.position = [ FL FR ]
                node.autoconnect = false
            }
            playback.props = {
                media.class = Audio/Source
                node.name = my_source
                node.description = "my-source"
                audio.channels = 2
                audio.position = [ FL FR ]
                node.autoconnect = false
            }
        }
    }
]
```
Restart pipewire.

### Carla setup

#### Carla startup
Start carla like this:

```
pw-jack -p 1440 carla
```
If Carla's buffer size is not 1440, restart  carla until it is.

#### Engine settings
Make engine settings as below and restart Carla

![image](https://user-images.githubusercontent.com/5956557/205489823-04f3e911-c174-4560-9eb7-bb8b52536c48.png)


#### Testing

Using qpwgraph, you can connect Carla to echo-cancel source and Sink. Thus, you should hear your voice when you test.
I suggest to test your voice with headphones to prevent echo. Test when you say a whole phrase/sentence, not only individual words.
Make sure that the mic is not peaking. All plugins in Carla must not touch the red areas in the volume bar of each plugin.
If the mic is peaking reduce the sensitivity/capture volume of it, through Alsa or Pulseaudio mixer.

![image](https://user-images.githubusercontent.com/5956557/205509754-3fbf085e-e321-4030-9095-3c6ed784309a.png)
Use masterme graph to adjust the gate and check how loud your voice is made.
![image](https://user-images.githubusercontent.com/5956557/205509911-887b5e1d-a513-4a67-accd-15c5597ea1fe.png)

After you are happy with the settings of all plugins, store the carla plugin chain in a Carla project file. (For example voice.carxp)

### Plugins

1. Equalizer (lv2 plugin) [https://lsp-plug.in/?page=manuals&section=graph_equalizer_x16_stereo](https://lsp-plug.in/?page=manuals&section=graph_equalizer_x16_stereo)
2. Master me (lv2 plugin) [https://github.com/trummerschlunk/master_me](https://github.com/trummerschlunk/master_me) It acts like gate + compressor + limiter
3. Calf Desser (lv2 plugin) [https://calf-studio-gear.org/doc/Deesser.html](https://calf-studio-gear.org/doc/Deesser.html)
4. Noise reduction plugin (vst2 plugin) [https://github.com/werman/noise-suppression-for-voice](https://github.com/werman/noise-suppression-for-voice)


#### Equalizer

You can use FFT analysis to see how your voice is heard. Remove very low frequencies and try to make your voice sound less muffled. Maybe decrease central frequencies and increase high ones.

![image](https://user-images.githubusercontent.com/5956557/205489954-b47498da-d43f-4e87-ba9a-4be5812d4c25.png)

#### Master me

From easy view select 'speech general' preset. Then switch to expert and activate gate.
You need to select the threshold of the gate based on what is heard when no or low sound is made.
Use 3 ms as attack and 300ms hold and 50ms as release. You can play with hodl value if you hear that your voice is chopped.
This plugin will insure that your voice is heard loud and clear.

![image](https://user-images.githubusercontent.com/5956557/205490368-68bfceaa-635f-4dba-91b5-e0c7eaefb6e3.png)

#### Desser

Due to compression you might emit 'sh' or 'ch' sounds that make your voice very unpleasant to listen.
Desser can fix that. With the help of equilizer and fft analysis test phrases that have many 'sh'/'ch' sounds for the language that you speak.
Use also the listen button to hear what it filters out. (It shouldn't destroy vowels)

![image](https://user-images.githubusercontent.com/5956557/205509494-e1a5090a-3769-4ffc-b4ea-5b9c92615f0b.png)

#### Noise suppresion

This plugin should remove unwanted sounds, like keyboard keys, cars, fans etc.
Use the below settings.

![image](https://user-images.githubusercontent.com/5956557/205510222-74e684c9-1271-449c-a385-ffd9665847ce.png)

#### Autromated Startup
After saving the project, you can automate startup like this:

```
#!/bin/bash

pw-jack -p 1440 carla ~/voice.carxp &
sleep 5
pw-link -d alsa_input.pci-0000_00_1b.0.analog-stereo:capture_FL Carla:audio-in1
pw-link -d alsa_input.pci-0000_00_1b.0.analog-stereo:capture_FR Carla:audio-in2
pw-link -d Carla:audio-out2 alsa_output.pci-0000_00_1b.0.analog-surround-51:playback_FR
pw-link -d Carla:audio-out1 alsa_output.pci-0000_00_1b.0.analog-surround-51:playback_FL
pw-link Carla:audio-out2 "my_sink:playback_FR"
pw-link Carla:audio-out1 "my_sink:playback_FL"
pw-link "Echo Cancellation Source:capture_FL" Carla:audio-in1
pw-link "Echo Cancellation Source:capture_FR" Carla:audio-in2

```

## Windows

Follow this guide to use equalizer APO:

(https://medium.com/@bssankaran/free-and-open-source-software-noise-cancelling-for-working-from-home-edb1b4e9764e)[https://medium.com/@bssankaran/free-and-open-source-software-noise-cancelling-for-working-from-home-edb1b4e9764e]

Master Me plugin is also available from Windows.
Any suggestion for free eq plugin and desser?

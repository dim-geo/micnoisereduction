# Mic noise reduction

A webpage to help document a good mic noise reduction setup.

## Linux

### Pipewire setup

Pipewire + carla (https://kx.studio/Applications:Carla) will be used.

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
### Carla setup
Make engine settings as below

![image](https://user-images.githubusercontent.com/5956557/205489823-04f3e911-c174-4560-9eb7-bb8b52536c48.png)

### Plugins

1. Equalizer (lv2 plugin) https://lsp-plug.in/?page=manuals&section=graph_equalizer_x16_stereo 
2. Master me (lv2 plugin) https://github.com/trummerschlunk/master_me It acts like gate+ compressor + limiter
3. Calf Desser (lv2 plugin) https://calf-studio-gear.org/doc/Deesser.html
4. Noise reduction plugin (vst2 plugin) https://github.com/werman/noise-suppression-for-voice


#### Equalizer

You can use FFT analysis to see your voice is heard. Remove very low frequencies and try to make your voice sound less muffled. Maybe decrease central frequencies and increase high ones.

![image](https://user-images.githubusercontent.com/5956557/205489954-b47498da-d43f-4e87-ba9a-4be5812d4c25.png)

#### Master me

From easy view select 'speech general' preset. Then switch to expert and activate gate.
You need to select the threshold of the gate based on what is heard when no or low sound is made.
Use 3 ms as attack and 300ms hold and 50ms as release. You can play with hodl value if you hear that your voice is chopped.

![image](https://user-images.githubusercontent.com/5956557/205490368-68bfceaa-635f-4dba-91b5-e0c7eaefb6e3.png)

## Windows

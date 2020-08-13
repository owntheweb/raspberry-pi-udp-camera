# Raspberry Pi UDP Camera

[![Watch Twitch](https://img.shields.io/badge/watch-Twitch-blueviolet.svg)](https://twitch.tv/hotsparklab) [![Made with](https://img.shields.io/badge/made%20with-coffee-orange)](https://streamelements.com/hotsparklab/tip) [![Fight COVID](https://img.shields.io/badge/fight-covid-red)](https://foldforcovid.io/)

Create a **low-latency** (*fast*) video camera with a [Raspberry Pi](https://www.raspberrypi.org/) and [camera module](https://www.raspberrypi.org/products/camera-module-v2/). Stream the video in realtime to a Mac, PC or other Pi for use in media projects in [Open Broadcaster Software (OBS)](https://obsproject.com/) and other media tools. Optionally scale up by creating a fleet of cameras that broadcast video to one or more machines.

Raspberry Pi UDP Camera is not a webcam, however it may be used similarly via [obs-gstreamer](https://github.com/fzwoch/obs-gstreamer) (for OBS), [GStreamer](https://gstreamer.freedesktop.org/) projects in general and other software (todo for other software, untested, solutions welcome).

This project was created with a need to use multiple cameras in a [live Twitch stream](https://twitch.tv/hotsparklab). During a pandemic, webcams have been considerably more expensive, while Raspberry Pi camera modules offer arguably higher quality video than some webcams. With Pis and camera modules handy, let's share.

## Setup balenaCloud Application

Using balena.io allows for quick deployment and monitoring of Docker-driven apps to devices such as Raspberry Pi (among others). It also allows for mass deployment, monitoring and updates, which had a use case when this project was created (multiple low-latency cameras for the [HotSparkLab](https://twitch.tv/hotsparklab) Twitch stream).

### Quick Setup

Use the following link to quickly setup the app in balena.io. See *Slow Setup* below for full details of what's included and various options.

[![Deploy with Balena](https://www.balena.io/deploy.png)](https://dashboard.balena-cloud.com/deploy)

### Slow Setup

<details>
  <summary>Expand</summary>

  Here's a full run-through from scratch if not using the deploy button above, including descriptions of available options and where they live.

  ## Configure the Pi Camera Fleet
  
  Setup a [belana.io account](https://dashboard.balena-cloud.com/signup).

  On the [Applications dashboard](https://dashboard.balena-cloud.com/apps), choose **Create application**.

  The camera fleet of one or more Pis will be configured with defaults that will apply to all potential camera devices. Select **Fleet configuration** on the left navigation.

  Under **CUSTOM CONFIGURATION VARIABLES**, add the following variables and values. Memory will adapt to what's available on the Pi:

  | Variable Name | Value |
  | --- | --- |
  | BALENA_HOST_CONFIG_start_x | 1 |
  | BALENA_HOST_CONFIG_gpu_mem_256 | 192 |
  | BALENA_HOST_CONFIG_gpu_mem_512 | 256 |
  | BALENA_HOST_CONFIG_gpu_mem_1024 | 448 |

  Select **Environment Variables** in the left menu. Add the following required environment variables and values that tell the camera which computer to broadcast to.

  | Environment Variable | Value | Description |
  | --- | --- | --- |
  | STREAM_DESTINATION_IP | Example: 192.168.0.10 | This is the local IP address of the computer that will receive the UDP video stream. |
  | STREAM_DESTINATION_PORT | Example: 5001 | This is the the port of the receiving computer to send to. Note that this will need to be unique per camera if the computer is receiving multiple video streams. As this is in the fleet environment variables as the default, it can be overridden per device as needed. |

  Additional optional environment variables may be set as needed to customize camera/raspivid settings.

  | Environment Variable | Default Value | Description |
  | --- | --- | --- |
  | WIDTH | 1280 | Available camera image width as [supported by raspistill](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) (see 'Version 1.x' and 'Version 2.x' depending on Pi camera version used). Choose an option that pairs with IMAGE_HEIGHT. |
  | HEIGHT | 720 | Available camera image height as [supported by raspistill](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) (see 'Version 1.x' and 'Version 2.x' depending on Pi camera version used). |
  | ROTATION | 0 | Supported image rotation values: `0`, `90`, `180` or `270` |
  | FLIP_HORIZONTAL |  | Flip the image horizontally. If enabling, use a value of `-hf`. |
  | FLIP_VERTICAL |  | Flip the image vertically. If enabling, use a value of `-vf`. |
  | FRAMES_PER_SECOND | 30 | A framerate of `2` to `30` frames per second is supported. |
  | INTRA_REFRESH_PERIOD | 60 | This option specifies the number of frames between each I-frame. Larger numbers will reduce the size of the resulting video. Smaller numbers make the stream less error-prone. |
  | BITRATE | 2500000 | Bits per second. 2500000 would be 2.5Mbits/s. Setting to high will result in stuttering video (untested in this project so far, 2500000 was smooth with Ethernet connection). |
  | METERING_MODE | average | The following metering modes are supported: `average`, `spot`, `backlit` or `matrix`. See -mm details [here](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) for full details. |
  | EXPOSURE_MODE | fixedfps | The following exposure modes are supported: `auto`, `night`, `nightpreview`, `backlight`, `spotlight`, `sports`, `snow`, `beach`, `verylong`, `fixedfps`, `antishake`, `fireworks`. See -ex details [here](https://www.raspberrypi.org/documentation/raspbian/applications/camera.md) for full details. |

  ## Uplaod Application via balena.io CLI

  lone the Raspberry Pi UDP Camera repository.

  ```
  git clone https://github.com/hotsparklab/raspberry-pi-udp-camera.git
  ```

  Follow [these instructions](https://www.balena.io/docs/reference/balena-cli/) to install the balena.io CLI, used to upload the project locally to the balena.io application.

  Login via the CLI (web authentication option when asked is recommended)

  ```
  balena login
  ```

  List available projects.

  ```
  balena apps
  ```

  Take note of the app name that was created for Raspberry Pi UDP Camera and push to the app.

  ```
  balena push your-app-name-here
  ```

  🦄 A successful deploy will end with an ascii art unicorn named Charlie.

</details>

## Install Raspberry Pi UDP Camera to the Pi

<details>
  <summary>Expand</summary>

  In the balena.io dashboard, choose **Devices** in the left navigation.

  Click the **Add Device** button at the top.

  In the modal window, set an Application Name and choose the device type, `Raspberry Pi 3` for example (several Pis supported, tested on Pi 3 and Zero Wireless), then choose **Create new application**. Note: Wireless setup can also be configured here. Wireless connections could result in added latency or dropped frames when viewing the camera depending on the connection.

  After downloading the application for the device, it can be burned to a MicroSD card using [balenaEtcher](https://www.balena.io/etcher/). Ensure the camera is installed properly to the Pi. Insert the card. Plug-in via Ethernet on same switch or use wireless if configured as such. Power the device and wait for a couple minutes. Connection status can be tracked on the Balena.io dashboard.

  ## How Stream Video In Open Broadcaster Software via GStreamer (OBS)

  GStreamer is a robust, open source multimedia framework, in this use case great for the purpose of receiving and displaying video broadcasted from the Pi. The [obs-gstreamer](https://obsproject.com/forum/resources/obs-gstreamer.696/) plugin bridges the gap between GStreamer and OBS, allowing configuration and display within OBS for a new "GStreamer source" (kinda like VLC Player, more robust and without a UI).
</details>

## Setup Open Broadcasting Software (OBS) To Use Camera(s)

<details>
  <summary>Expand</summary>

  ### Install GStreamer (Windows 10)

  For the [obs-gstreamer](https://github.com/fzwoch/obs-gstreamer) to work, GStreamer runtime and the GStreamer development SDK will need to be installed to the PC. Note: There's an experimental Mac and Linux obs-gstreamer plugin, untested here. Post an issue or PR to inform us how it works if you use it.

  Download the **MinGW 64-bit** (or 32-bit if needed) runtime and development installers from [GStreamer's website](https://gstreamer.freedesktop.org/download/) and install.

  ### Add GStreamer to PATH Environment Variable (Windows 10)

  An environment variable will need to be added to Windows PATH so that obs-gstreamer knows where to look for GStreamer. Otherwise, it won't show as an option in OBS.

  Open Control Panel and search for **environment variables** on the top-right.

  Choose **Edit environment variables for your account**.

  Add a new environment variable with **variable name** of `GSTREAMER_1_0_ROOT_X86_64` and **variable value** of `C:\gstreamer\1.0\x86_64` (or wherever you installed GStreamer if different).

  Hit the **OK** button, then the parent window's **OK** button to finish up.

  ## Install obs-gstreamer Plugin to OBS

  obs-gstreamer will enable a new configurable "GStreamer source" in OBS.
  
  Visit the [obs-gstreamer](https://obsproject.com/forum/resources/obs-gstreamer.696/) page and choose **Go to download**.

  Download and extract the latest **obs-gstreamer.zip**.

  copy **obs-gstreamer.dll** into **C:\Program Files\OBS\plugins** (64-bit) or **C:\Program Files (x86)\OBS\plugins** (32-bit), altered if OBS was installed elsewhere.

  ## Add Streaming Video From Raspberry Pi To OBS

  Now for the fun part. Get live, low-latency video from the Pi to show in OBS as a video source.

  ### Find Streaming Computer IP Address

  The Pi with camera will send video to the streaming computer via UDP. To do that, the Pi will need to know the streaming computer's local IP address.

  In Windows, type `cmd` in the Windows Start search bar and hit enter to launch Windows terminal.

  Type, `ipconfig` and press Enter.

  The IP address will be towards the top and looking similar to, `192.168.0.X (just not the .255 one listed). Note that address.

  ### Point the Pi Fleet To the Streaming PC

  Back to the balena.io for a moment, visit the application's Environment Variables page. Update **STREAM_DESTINATION_IP**  value to the noted IP address above.

  If multiple Pis are going to be running the application for the same streaming computer, make sure each device specifies a unique **STREAM_DESTINATION_PORT**, e.g. `5001`, `5002`, etc.

  ### Add a GStreamer Source in OBS

  Launch OBS.

  With a scene active, under **Sources**, choose **+**.

  Select **GStreamer Source**.

  In the pipeline field enter the following, replacing YOUR_PI_PORT with the port specified in belana.io for that device (or fleet port is fine if there's only one device).

  ```
  udpsrc port=YOUR_PI_PORT ! h264parse ! avdec_h264 ! video.
  ```

  If/when the Pi is booted and running, the camera source will be displayed in the scene and can be scaled and filtered to taste. Note: Pi status can be monitored in the balena.io dashboard with latest logs shown.

  🚨🎉🚨🎉

</details>

## How To Stream Video To VLC Player

There hasn't been luck successfully playing this formatted UPD video stream in VLC Player on Mac or PC yet (or any **low-latency** video stream from a Pi to VLC Player). Prove me wrong. Post an issue or PR with the solution. Let's add instructions for it as it would be very accessible to OBS users with VLC Player  already installed (if simple and possible).

## How To Stream From A Raspberry Pi As A Webcam

While untested here, UDP video streams have been known to be used as webcams for a receiving computer through third-party applications. A [Google search](https://www.google.com/search?q=use+udp+camera+as+webcam+source&oq=use+udp) brought up several results to explore. Which solution works for you? Post an issue or a PR and share!

## Special Thanks

Many more holes would be in the wall with far fewer brain cells active today if it wasn't for [fzwoch](https://github.com/fzwoch)'s [obs-streamer](https://obsproject.com/forum/resources/obs-gstreamer.696/) project. Also thanks to great comments from [Dregu](https://github.com/Dregu) and [crossan007](https://github.com/crossan007) in [this issue](https://github.com/fzwoch/obs-gstreamer/issues/3) and the [balena.io](https://balena.io) team for their feedback on live streaming topics. Thank you!

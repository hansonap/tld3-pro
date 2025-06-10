# tld3-pro
An overhaul of a Tenlog TL-D3 Pro stock printer to one that adds a new screen and Raspberry Pi to run Klipper.

## Random Resources
### [jonpackard/3d-printing-profiles](https://github.com/jonpackard/3d-printing-profiles)
This repository contains a useful PrusaSlicer profile for the stock printer and was used as a starting point
during developement. The README.md also contains the stock printer settings as reported via the G-Codes M115 
and M503. **The tool change G-Code macro ended up causing prints to fail to start when starting with the 2nd 
tool head as when it docked the first tool head it would try and retract the fillament on a cooled head which
Klipper will throw an error (Marlin flavor does not throw an error and just skips it). Currently the tool head
macro has been disabled as I need to understand how to retract conditionally**.
> These Klipper G-Code conditional resources may help:
> * https://jinja.palletsprojects.com/en/stable/templates/
> * https://klipper.discourse.group/t/macro-creation-tutorial/30/4
> * https://github.com/Klipper3d/klipper/blob/master/config/sample-multi-extruder.cfg

### [kiler129\klipper](https://github.com/kiler129/klipper/blob/73bdcc74fd325413cf7a9beca0b636748c1abf6e/config/printer-tenlog-d3-pro.cfg)
This config got the printer up and running however either the x-axis or y-axis was reversed and I believe there
was another issue.
> TODO: Compare this config to the current one and docuement the differences.

### Pizo Buzzer
Klipper does not support the use a buzzer / speaker for notifications but 
[this](https://github.com/Klipper3d/klipper/pull/1554/commits/42fe774f90b04a824edff7ceabea13ff3c2d41f9) 
pull request shows how to add it as an output pin and how to define a gcode macro to use it. Additionally,
[this](https://github.com/ac316scu/songs.cfg/blob/main/songs.cfg) config shows how to play songs with it. I
currently don't have a use case for it, but I envision a tone being played each time the printer needs human
interaction, if a failed print was detected, when the print had finished, or after the bed had cooled down
enough to release parts.

### Misc.
* [jonpackard/klipper_config_tld3pro](https://github.com/jonpackard/klipper_config_tld3pro) - This is a TL-D3
  Pro Klipper conversion, but I have not yet studdied it.

> TODO: add [this](https://assets.website-files.com/5fabc8dadc66fc8a47db709b/63f8ed1b73441360f8c195fb_Raspberry%20pi%20pinout%20header%20image.webp)
> image somewhere

## Build Notes
### Touchscreen
The physical orientation of the screen required to fit the exiting chasis resulted in an upside display image 
and thus had to be changed in software. There were two steps required for this, screen rotataion, and touch 
rotation. These steps seem to be Pi version and screen dependent but worked for the chosen hardware is 
documented below.

Per the Klipper Screen [Screen Rotation docs](https://klipperscreen.readthedocs.io/en/latest/Troubleshooting/Rotation/#universal-xorg-configuration):
* I don't remember trying the *Universal xorg configuration* method and the file
  /usr/share/X11/xorg.conf.d/90-monitor.conf does not exist.
* The *Raspberry Pi using kernel cmdline* method worked and the text " video=DSI-1:800x480@60,rotate=180" was
  added to the end of the existing line.
* The manufacture of the sceen provided instructions but these did not work.

Per the Klipper Screen 
[Touchscreen issues](https://klipperscreen.readthedocs.io/en/latest/Troubleshooting/Touch_issues/) page:
* The instruction under "Touch rotation and matrix" did allow me to identify / test the correct rotation but
  the instructions under "Save touch calibration" did not work.
* The "Alternative Example" method worked.

### Klicky Probe
I bought a BTT Klicky Probe but have not made a plan but here are some resources that my prove useful:
* [Klicky Prode docs](https://jlas1.github.io/Klicky-Probe/)
* [Klicky GitHub](https://github.com/jlas1/Klicky-Probe)
* [Klicky PCB](https://github.com/tanaes/whopping_Voron_mods/tree/main/pcb_klicky) - I Believe this is what I
  have.
* [Klicky-00: Zero X,Y offset probe](https://github.com/DW-Tas/Klicky-00?tab=readme-ov-file#klicky-00-zero-xy-offset-probe-rc3)
  - A more advanced version that could be looked at in the future.

### Arducam
Contrary to what the 
[camera docs](https://docs.arducam.com/Raspberry-Pi-Camera/Native-camera/12MP-IMX708/#arducam-autofocus-imx708-camera-module) 
say, do not change `camera_auto_detect=1` to `camera_auto_detect=0` but do add the line `dtoverlay=imx708`.
I ended up with the below section directly above the MailsailOS header
```
[all]
# [AH] add arducam
dtoverlay=imx708
```
After that you can follow the Mainsail 
[How to setup a Raspicam](https://crowsnest.mainsail.xyz/faq/how-to-setup-a-raspicam#step-2-set-your-device-path-1)
instructions.

> * It is possible auto focus needs to be enabled
> * [obico](https://www.obico.io/blog/klipper-camera/#raspberry-pi-cameras) uses AI to detect failed prints.

### 24V Relay
As described on on this [page](https://www.reddit.com/r/klippers/comments/z79kx6/relay_control_mainsail_dashboard/)
There are two approaches for controlling the relay, you can use the Klipper config, but this will only work
if all MCUs are connected, or through the moonraker config. The below section was added to our moonraker.conf
```
# 24V realy A
[power 24V Power]
type: gpio
pin: gpio19
initial_state: off
off_when_shutdown: True
locked_while_printing: True
on_when_job_queued: True
# bound_service: klipper # We don't want this as we want the MCU to be powered by the physical power switch
# and to always connect via klipper when powered - the reason we have the relay is so that we can kill the
# 24V power for flashing the klipper firmware on to the MCU as instructed by the docs.
```

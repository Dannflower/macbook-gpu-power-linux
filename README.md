# macbook-gpu-power-linux

If you've found this, you're likely several hours down the rabbit hole of old hardware and obscure fixes. The Linux experience.

# Warning!

This is a hack, it is not supported, and you proceed at your own risk. Skipping steps, or unforseen errors will quickly lead to frozen screens when GDM starts or even unbootable states.

## Context & Goal

Some older MacBook Pros from around 2012 have hybrid graphics which include a discrete AMD or Nvidia GPU and integrated Intel graphics. Unfortunately, the discrete GPUs are so old at this point they don't really help with much and just drain your battery and make your laptop extremely hot.

The goal of this repo is to provide a reasonably easy way to switch to using the integrated Intel graphics and simultaneously switch off power to the discrete GPU.

## Tested Hardware & OS

Macbook Pro 9,1 (Mid 2012, non-retina, 15") - Manjaro with Gnome Desktop 41.1

# Pre-requisites

## Driver
Install the open-source video-linux drivers. Specifically, this has only been tested with the Nouveau driver.

## Gpu-switch
Install gpu-switch from AUR: https://aur.archlinux.org/packages/gpu-switch

This is a handy tool for older MacBooks that allows for switching between integrated graphics or discrete between boots. This should be available for other distros as well.

For more info see the repo: https://github.com/0xbb/gpu-switch

# Setup

Clone this repo to your local machine.

Copy the gpu-power service config to systemd.

```
sudo cp gpu-power.service /etc/systemd/system/
```

Make the gpu-power script executable.
```
sudo chmod +x gpu-power
```

Copy the gpu-power script to /usr/bin.

```
sudo cp gpu-power /usr/bin
```

Enable the gpu-power service to start on reboot.
```
sudo systemctl enable gpu-power
```

Set the MacBook to use integrated Intel graphics on next boot.
```
sudo gpu-switch -i
```

Reboot your computer for the changes to take affect.

This change adds a non-trivial amount of startup time. The gpu-power script will first sleep for 5 seconds to allow the GPU driver to load. It will then power down the discrete GPU which takes a moment, and finally it will restart GDM which also takes a moment. The screen will flicker a couple of times and then the login should appear.

## Verify

To verify what GPU is in use and what is powered on, check with vgaswitcheroo.
```
sudo cat /sys/kernel/debug/vgaswitcheroo/switch
```

The output should look something like this:
```
1:IGD:+:Pwr:0000:00:02.0
2:DIS: :Off:0000:01:00.0
```

IGD should have the + and PWR while DIS should not and be marked as Off.

Enjoy the cooler laptop and longer battery life!

# Troubleshooting

## Frozen Login Screen

If something goes wrong or an error occurs, GDM will likely freeze as soon as it boots and you will be greeted by a gray or black screen and maybe a mouse cursor.

I find this most often happens if the discrete GPU is not powered off but gpu-switch has been used to switch to the integrated GPU.

Generally this can be fixed by logging in with a tty session instead of GDM and undoing the gpu-switch change.

When the black/grey GDM screen comes up, switch to tty.
```
ctrl + alt + F2
```

Login and switch back to discrete graphics.
```
sudo gpu-switch -d
```

Reboot to apply the change.

This should get the system back into a state where GDM loads up normally and you can troubleshoot from there.

Note: You can also check the vgaswitcheroo/switch status while in the tty session as a helpful troubleshooting point to see what's powered/in-use.
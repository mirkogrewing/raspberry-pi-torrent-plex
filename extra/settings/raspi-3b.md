# Settings for Raspberry Pi 3 - Model B

When I started using my device even for just downloading torrents I noticed that it was crashing quite frequently. Plex added even more workload since it stresses both the CPU and the GPU, especially if you want to use some features like thumbnails generation.

After a little bit of testing I was satisfied with the following settings in `/boot/config.txt`:

```
gpu_mem=320

force_turbo=1
boot_delay=1
over_voltage=6
```

Please note that `gpu_mem` is already present in the file, so you only have to change the value. The other parameters should not be there if you never edited the file.

After rebooting, you can check how memory is split via the command:

```
$ vcgencmd get_mem arm && vcgencmd get_mem gpu
```

in my case, since the Raspberry Pi 3B has 1 GB (1024 MB) memory, I got:

```
arm=704M
gpu=320M
```

Also, you can confirm that Performance mode is enabled by default with the command:

```
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
```

In my case, it's 1.2 GHz, and the command returned `1200000`, correctly.

### Extra: Monitor clock, temperature and voltage

For your test and if you are concerned about the health of your device, you can use the following script:

```
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
/opt/vc/bin/vcgencmd measure_temp
/opt/vc/bin/vcgencmd measure_volts core
```

that you can save as `monitor_rasbpi.sh` and launch with:

```
watch ./monitor_rasbpi.sh
```

to see the output updated every 2 seconds.

### Extra: Overclock

You can go further and overclock the CPU of your Raspberry Pi.

:exclamation: WARNING :exclamation:
Overclocking your CPU can result in serious damages to your the same and might void the warranty. Please do it at your own risk. In my case, even without overclock, the Raspberry was constantly working on a temperature around 70˚C, reaching occasionally 81-82˚C. Ofter installing a copper heatsink for the RAM and a heatsink with fans for the board, the temperature - when idle - never went above 44˚C - ~~even after overclocking~~ while it stayed between 58 and 62˚C while streaming a movie.

In my case (Raspberry Pi 3b) I ~~could~~ tried to overclock from 1.2 to 1.35 GHz by adding the following to `/boot/config.txt`

```
arm_freq=1350
core_freq=500
disable_splash=1
```

~~and rebooting~~, but after reboot I noticed that every video I was playing was crashing with the following error:
> The transcoder process crashed

Reverting to the normal settings solved the problem.

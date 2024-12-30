# pwm-gpio-fan overlay

Raspberry Pi device-tree overlay for a GPIO connected PWM cooling fan controlled by the software-based GPIO PWM kernel module

## Description

There are several PWM controlled cooling fans avaliable for the Raspberry Pis prior to the Pi 5, that are connected via the Pi's GPIO header.
Examples are the Argon mini-fan, HighPi Pro Fan or Waveshare FAN-4020-PWM-5V.

What these fans have in common is, that they are using the +5V pin and GND pin from the Pi's GPIO header to supply power to the fan and an additional GPIO pin (mostly GPIO 18) to control the fan state to be either on or off.

### Available legacy solutions

#### 1. The gpio-fan overlay:
The Raspberry Pi OS supports a simple kernel module controlled by the gpio-fan overlay to just switch the fan on or off at a specific temperature. This solution is available back from the legacy Buster OS version.
  - **Pro:** Built-in solution, even configurable from the raspi-config tool in Raspberry Pi OS Buster and Bullseye.
  - **Con:** Unfortunately these small fans tend to be quiet noisy when just switched on at full speed.

#### 2. 3rd party overlays to control the fan by the Pis built-in hardware PWMs:
There are several implementations available on the internet to do this.
  - **Pro:** The fan remains much quieter, often not even noticable.
  - **Con:** These solutions require one of the two Pi's hardware PWMs (PWM0 or PWM1). But, both hardware PWMs are required by the Pi for analogue audio output, so analogue audio is heavily disturbed if you use a hardware PWM for fan control (if you like to try it, please use very low volume levels, its an ugly noise on the speakers or head phones).

#### 3. Userland scripts:
There are several implementations available on the internet to do this. Mostly python scripts, sometimes combined with C code.
  - **Pro:** The fan remains much quieter, often not even noticable, and no hardware PWM is occupied, so no conflict with the Pi's analogue audio output.
  - **Con:** The scripts can be unreliable to control the fan at very high CPU loads. The scripts itself sometimes consume noticable CPU power to do their job. And finally, the python GPIO interfaces have changed substantially over time. There are still many scripts available, that depend on WiringPi and WiringPi is no longer part of the OS repositories since Bullseye in 2021.

### New solution by using software PWM

Fortunately a new kernel based software PWM solution is available since November 2024. A software-based PWM kernel module is available since then, back-ported from the Linux kernel 6.11 to the Raspberry Pi OS Bookworm kernel 6.6.62.

This made me write the **pwm-gpio-fan** overlay for my own use and publish it here for the community.

- **Pro:** The fan remains much quieter, often not even noticable, and no hardware PWM is occupied, so no conflict with the Pi's analogue audio output. Reliable on even high CPU loads, as it's part of the kernel. Doesn't consume noticable CPU power even on a Pi 3.

- **Con:** Available on Raspberry Pi OS Bookworm with kernel 6.6.62 or above only. Sorry no support for earlier OS or kernel versions.

### How to install

1. Download the file pwm-gpio-fan.dtbo from the repository
2. Copy pwm-gpio-fan.dtbo to /boot/firmware/overlays/ on your Raspberry Pi
3. Add one line to the end of /boot/firmware/config.txt: "dtoverlay=pwm-gpio-fan"
4. Reboot

### Customization

There are several parameters you can specifity in the /boot/firmware/config.txt command line to customize the behaviour of the overlay:

- "fan_gpio"	BCM number of the pin driving the fan, default 18 (GPIO 18)
- "fan_temp0"	CPU temperature at which fan is started with low speed in millicelsius, default 55000 (55 °C)
- "fan_temp1"	CPU temperature at which fan is switched to medium speed in millicelsius, default 60000  (60 °C)
- "fan_temp2"	CPU temperature at which fan is switched to high speed in millicelsius, default 67500  (67.5 °C)
- "fan_temp3"	CPU temperature at which fan is switched to max speed in millicelsius, default 75000  (75 °C)
- "fan_temp0_hyst"	Temperature hysteris at which fan is stopped in millicelsius, default 5000 (resulting in 50 °C)
- "fan_temp1_hyst"	Temperature hysteris at which fan is switched back to low speed in millicelsius, default 5000 (resulting in 55 °C)
- "fan_temp2_hyst"	Temperature hysteris at which fan is switched back to medium speed in millicelsius, default 5000 (resulting in 62.5 °C)
- "fan_temp3_hyst"	Temperature hysteris at which fan is switched back to high speed in millicelsius, default 5000 (resulting in 70 °C)
- "fan_temp0_speed"	Fan speed for low cooling state in range 0 to 255, default 114 (45% PWM duty cycle)
- "fan_temp1_speed"	Fan speed for medium cooling state in range 0 to 255, default 152 (60% PWM duty cycle)
- "fan_temp2_speed"	Fan speed for high cooling state in range 0 to 255, default 204 (80% PWM duty cycle)
- "fan_temp3_speed"	Fan speed for max cooling state in range 0 to 255, default 255 (100% PWM duty cycle)

> [!NOTE]
> Lowest temperature default is set to 55 °C, to keep the fan off when the Pi is idle. PWM duty cycles start from a rather high 45% to ensure the fan is started reliable. Many small fans have difficulties to start reliable at low duty cycle values.

### How to compile the overlay from the provided source

The source code of the overlay is provided in the repository as **pwm-gpio-fan-overlay.dts** to allow you to make any modifications you like.

1. Download the file pwm-gpio-fan-overlay.dts from the repository
2. Modify it to your needs
3. Compile with the command "sudo dtc -I dts -O dtb -o /boot/firmware/overlays/pwm-gpiofan.dtbo pwm-gpiofan-overlay.dts"
4. Add one line to the end of /boot/firmware/config.txt: "dtoverlay=pwm-gpio-fan"
5. Reboot

### WARRANTY
> [!IMPORTANT]
> The device-tree overlay provided here is free software. It comes with ABSOLUTELY NO WARRANTY, to the extent permitted by applicable law.

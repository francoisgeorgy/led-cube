LED Cube Operating System installation
======================================

The operating system I installed is **Raspberry Pi OS Lite (32-bit)** (Debian Bookworm). All instructions contained 
in this document are for this version of the operating system. There may be differences if you choose another OS distribution.

An operating system distribution without any desktop environment allows better "real-time" performance with the tradeoff 
that installation and configuration is a little more tedious (especially for wifi).

## Install the OS : 

1. Install [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Choose :
   1. Raspberry Pi OS (Other)
      1. Raspberry Pi OS Lite (32-bit)
3. Open settings
   1. enable SSH with :
      1. hostname : _choose a hostname_
      2. user: _choose a username_
      3. wifi : _the SSID of your wifi network_
4. Flass the sd-card, install it in the RPi and start the RPi

## Log in the RPi

Find the ip address of the RPi :

    sudo nmap -sn 192.168.1.0/24 | grep -iB2 "D8:3A:DD"

If the Raspberry Pi is found, the result should be something like : 

    Nmap scan report for pi-e1-12 (192.168.1.100)
    Host is up (0.12s latency).
    MAC Address: D8:3A:DD:39:E1:12 (Unknown)

Log in using the IP address :

(replace `user` by the username you chose previously)

    ssh user@192.168.0.100

## OS configuration 

First, make sure the OS is up-to-date : 

    sudo apt update && sudo apt upgrade -y

### Disable sound system : 

We need to disable the Raspberry Pi sound system because the LED matrix driver uses the same hardware
and can't work if the sound is enabled. 

Following instructions given in https://github.com/hzeller/rpi-rgb-led-matrix/blob/master/README.md#bad-interaction-with-sound, do : 

    sudo sed -i 's/audio=on/audio=off/' /boot/firmware/config.txt

This is not sufficient and we also need to blacklist the sound module : 

    echo "blacklist snd_bcm2835" > blacklist-snd_bcm2835.conf
    sudo mv blacklist-snd_bcm2835.conf /etc/modprobe.d/
    sudo chown root:root /etc/modprobe.d/blacklist-snd_bcm2835.conf

### CPU usage : 

The rpi-rgb-led-matrix project also recommend to reserve one CPU core for the display refresh : 

    echo " isolcpus=3" | sudo tee -a /boot/firmware/cmdline.txt

### I2C :

We need to enable I2C (used by the accelerometer) :

    sudo raspi-config nonint do_i2c 0

### Remove unused applications : 

Remove unwanted tools :

    sudo apt-get remove triggerhappy pigpio -y

### Install dev tools :   

Install the packages we will need later on :

    sudo apt install \
         git build-essential cmake \
         python3-dev python3-pillow \
         python3 python3-pip \
         libxcursor-dev libxinerama-dev libxi-dev libx11-dev \
         libglu1-mesa-dev libxrandr-dev libxxf86vm-dev \
         i2c-tools -y

check that python version >= 3.10 :

    python -V

you should get an answer like : 

    Python 3.11.2

### Reboot

Finally, reboot to make sure all the previous changes are loaded :

    sudo reboot

## Cube software installation

### Create a home dir for the cube

    sudo mkdir /home/cube
    sudo chown $USER:$USER /home/cube
   
### Install the RGB library : 

    cd /home/cube
    git clone https://github.com/hzeller/rpi-rgb-led-matrix.git
    cd rpi-rgb-led-matrix/
    make
    make build-python PYTHON=$(command -v python3)
    sudo make install-python PYTHON=$(command -v python3)

try a demo : 

    cd examples-api-use/
    sudo ./demo --led-rows=64 --led-cols=64 --led-slowdown-gpio=5 -D 0

if you have the adafruit hat : 

    sudo ./demo --led-rows=64 --led-cols=64 --led-gpio-mapping=adafruit-hat --led-slowdown-gpio=5 -D 0

with the 6 panels connected : 

    sudo ./demo --led-rows=64 --led-cols=64 --led-chain 3 --led-parallel 2 --led-slowdown-gpio=5 -D 4

test the python samples : 

    cd /home/cube/rpi-rgb-led-matrix/bindings/python/samples

with one panel : 

    sudo python pulsing-colors.py --led-rows=64 --led-cols=64 --led-slowdown-gpio=5

with all the panels : 

    sudo python pulsing-colors.py --led-rows=64 --led-cols=64 --led-chain 3 --led-parallel 2 --led-slowdown-gpio=5

### LIS3DH accelerometer : 

Connect the LIS3DH and do : 
       
    i2cdetect -y 1

the command must return : 

         0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
    00:                         -- -- -- -- -- -- -- --
    10: -- -- -- -- -- -- -- -- 18 -- -- -- -- -- -- --
    20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
    70: -- -- -- -- -- -- -- --

This shows that the LIS3DH has the address 0x18. 

## Next steps 

You can now install the Python software by following [python-installation.md](python-installation.md).

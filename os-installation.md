LED Cube Operating System installation
======================================

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

### Bookworm Pi Imager bug : 

- https://forums.raspberrypi.com/viewtopic.php?t=358046#p2165884
- https://github.com/raspberrypi/rpi-imager/issues/749

edit /boot/firstrun.sh

after 

      if [ -f /usr/lib/raspberrypi-sys-mods/imager_custom ]; then

add

      sed -i 's/$(uuid -v4)/12345678-1234-1234-1234-123456789abc/g'  /usr/lib/raspberrypi-sys-mods/imager_custom


## Log in the RPi

1. Find the ip address of the RPi :
   1. `sudo nmap -sn 192.168.1.0/24 | grep -B2 "E1:12"`


         Nmap scan report for pi-e1-12 (192.168.1.100)
         Host is up (0.12s latency).
         MAC Address: D8:3A:DD:39:E1:12 (Unknown)  
1. Log in :
   1. ssh user@192.168.0.100

## Adjust config : 

    sudo sed -i 's/audio=on/audio=off/' /boot/config.txt
    echo " isolcpus=3" | sudo tee -a /boot/cmdline.txt

audio=off is not sufficient, we need to blacklist the bmc2835 module  :
   
do : 

    cat <<EOF | sudo tee /etc/modprobe.d/blacklist-rgb-matrix.conf

enter : 

    blacklist snd_bcm2835
    EOF
   
source : https://github.com/hzeller/rpi-rgb-led-matrix/blob/master/README.md#bad-interaction-with-sound

update os : 

    sudo apt update && sudo apt upgrade -y

remove unwanted tools and apps :

    sudo apt-get remove bluez bluez-firmware pi-bluetooth triggerhappy pigpio -y

install packages we will need later on :

    sudo apt install \
         git build-essential cmake \
         python3-dev python3-pillow \
         python3 python3-pip \
         libxcursor-dev libxinerama-dev libxi-dev libx11-dev \
         libglu1-mesa-dev libxrandr-dev libxxf86vm-dev -y

    sudo apt install  -y

check that python version >= 3.10 :

    python -V

    Python 3.11.2

## create a home dir for the cube

    sudo mkdir /home/cube
    sudo chown francois:francois /home/cube
    cd /home/cube
   
## install rgb lib : 

    cd /home/cube
    git clone https://github.com/hzeller/rpi-rgb-led-matrix.git
    cd rpi-rgb-led-matrix/
    make
    make build-python PYTHON=$(command -v python3)
    sudo make install-python PYTHON=$(command -v python3)

try a demo : 

    cd examples-api-use/
    sudo ./demo --led-rows=64 --led-cols=64 --led-slowdown-gpio=5 -D 0

with all the panels : 

    sudo ./demo --led-rows=64 --led-cols=64 --led-chain 3 --led-parallel 2 --led-slowdown-gpio=5 -D 4

if you have the adafruit hat : 

    sudo ./demo --led-rows=64 --led-cols=64 --led-gpio-mapping=adafruit-hat --led-slowdown-gpio=5 -D 0

test the python samples : 

    cd /home/cube/rpi-rgb-led-matrix/bindings/python/samples
    sudo python pulsing-colors.py --led-rows=64 --led-cols=64 --led-chain 3 --led-parallel 2 --led-slowdown-gpio=5

## Python lib and sample apps : 

We will use a python venv : 

    #sudo apt install python3-full

    mkdir /home/cube/python
    cd /home/cube/python

    # it's important to link the system packages, because the rpi-rgb lib is installed system wide.
    python3 -m venv --system-site-packages .venv

    . .venv/bin/activate

we need to have root privileges to get good performances, so we need to use sudo.

Here is how to use a python venv with sudo  :

    sudo -E env PATH=$PATH ...

example : 

    (.venv) $ sudo -E env PATH=$PATH python -c 'import sys; print(sys.path)'
    (.venv) $ sudo -E env PATH=$PATH pip -VV

Note: sudo is only needed when running a python script which uses the rip-rgb lib. It is not needed otherwise.

## rgbmatrix library

We will use the python bindings created while installing the rpi-rgb library before.

DO NOT install rgbmatrix in the venv otherwise it will be impossible to update it, for exemple, to update the pixel mapper.

## our own library

install our ledcube python stack :

    . .venv/bin/activate

    # this MUST be done in a venv
    pip install --editable .

test : 

    # one panel : 
    sudo -E env PATH=$PATH python src/samples/rgb.py --led-slowdown-gpio 5

    # all panels : 
    sudo -E env PATH=$PATH python src/samples/rgb3d.py --led-slowdown-gpio 5 --led-rows=64 --led-cols=64 --led-chain 3 --led-parallel 2


----

## IMU LIS3DH

### Prerequisite : 

Make sure i2c is enabled. 

    sudo raspi-config nonint get_i2c

must return 0. If not, then do : 

    sudo raspi-config nonint do_i2c 0

### Libraries : 

See also : https://github.com/adafruit/Raspberry-Pi-Installer-Scripts/blob/main/raspi-blinka.py

    sudo apt update
    sudo apt install -y i2c-tools

    pip3 install RPi.GPIO adafruit-blinka

Note : `import board` must now be possible in Python.

In cas of SSL error, try : 

    $ pip3 install --trusted-host pypi.org --trusted-host pypi.python.org --trusted-host files.pythonhosted.org adafruit-blinka

### Detect LIS3DH
    
    
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

### Test LIS3DH

Test with the script `src/samples/tests/test-lis3dh.py`.


Digital Fab Lab

#Disable Screen Blanking (Screensaver)
Disable text terminals from blanking
setterm - terminal blanking
```
nano ~/.bashrc
```
add 
```
setterm -blank 0 -powerdown 0
```

kbd configuration - terminal blanking
```
sudo nano /etc/kbd/config
```
```
BLANK_TIME=0
BLANK_DPMS=off
POWERDOWN_TIME=0
```

```
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
or 
(seems to work for pixel)
sudo nano ~/.config/lxsession/LXDE-pi/autostart
```

```
#@xscreensaver -no-splash
@xset s off
@xset -dpms
@xset s noblank
```
###Turn off cursor
```
sudo apt-get install unclutter

```
```
sudo nano ~/.config/lxsession/LXDE-pi/autostart
```
add
```
@unclutter -idle 0.01 -root
```

#TouchScreen turn off blanking

Use nano
```
sudo nano /etc/lightdm/lightdm.conf
```
Look for the line 
```
#xserver-command=X
``` 
Change it to 
```
xserver-command=X -s 0 -dpms
```
It should be at line 87 if things don't change.
Save and reboot.


#Set Static IP Address
```
sudo nano /etc/dhcpcd.conf
```
Add the following at the bottom

```
interface eth0

static ip_address=192.168.0.10/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1

interface wlan0

static ip_address=192.168.0.10/24
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
```


#Setup Pi

###change password
```
passwd
```

###network address update (pi name modification using bonjour & avahi)
Change name from raspberrypi to a name that is more meaningful in below:
```
sudo nano /etc/hosts
sudo nano /etc/hostname
```
change 'raspberrypi' to desired device name (ex: jeremy)

```
sudo apt-get install avahi-daemon
sudo apt-get install avahi-utils
ctrl + c to cancel process
avahi-resolve --address <ip.address>
sudo reboot
```

From remote on same network
```
ssh pi@new_hostname
```

###Install npm and latest node
```
sudo apt-get install git && git clone https://github.com/audstanley/NodeJs-Raspberry-Pi-Arm7 && cd NodeJs-Raspberry-Pi-Arm7 && chmod +x Install-Node.sh && sudo ./Install-Node.sh;
```

Check installation:
```
node -v
```  



###Disable rainbow boot screen
```
sudo nano /boot/config.txt
```

add the following line:
```
disable_splash=1
```

###Rotate Pi Screen through HDMI
```
display_rotate=1
```
###Flip Raspberry Pi TouchScreen
```
sudo nano /boot/config.txt
```
add the following line:
```
lcd_rotate=2
```


###remove raspberry pi logos at top( one for each core)
```
sudo su
cd /boot
nano cmdline.txt
```

add to beginning of line
```
logo.nologo
```
turn off jessie pixel splash by deleting word "splash" from cmdline.txt

###Start Tmux fullscreen


####install requirements  
```
sudo apt-get install -y tmux wmctrl htop iftop vim
sudo pip install tmuxp	
```
    
####scripts
**create shell script to launch tmux**
```
!#/bin/bash
tmuxp load start.yaml
```
#### install vim
```
sudo apt-get install vim
curl http://j.mp/spf13-vim3 -L -o - | sh
```

#### update autostart
    sudo vim /home/pi/.config/lxsession/LXDE-pi/autostart
    @lxterminal --command "/home/pi/dev/start.sh"

#### tmuxp
***save***  
tmuxp freeze  

***load***  
tmuxp load <yoursavedconfig>.yaml


#### start.yaml
```
vim /home/pi/.tmuxp/start.yaml
```
**add**
```
session_name: nodeapp
windows:
- focus: 'true'
  layout: 8fb4,159x67,0,0{66x67,0,0,1,42x67,67,0,2,49x67,110,0[49x33,110,0,3,49x33,110,34,4]}
  options:
    automatic-rename: 'off'
  panes:
  - shell_command:
      - "wmctrl -r :ACTIVE: -b add,fullscreen"
      - "ls -lash"
  - shell_command:
      - "sudo node /home/pi/app/meep-robot/meep.js"
  - shell_command:
      - "htop"
  - shell_command:
      - "sudo iftop"
  start_directory: /home/pi/app
  window_name: start
```



####Mac Keyboard fix
	sudo dpkg-reconfigure keyboard-configuration

- **select**  
MacBook/MacBook Pro (Intl)

- **select**  
English (US) - English (Macintosh)

- Reboot.

####Fix Time
	sudo date -s “Apr 9 11:37”


####Fun
**Look busy**
```  
hexdump -C < /dev/urandom | grep "ca fe"
sudo apt-get install pv
hexdump -C < /dev/urandom | grep "ca fe"
```


####wmcntrl - for fullscreen terminal
**install**
```
sudo apt-get wmctrl
```

**fullscreen**
```
wmctrl -r :ACTIVE: -b add,fullscreen
```
**remove fullscreen**
```
wmctrl -r :ACTIVE: -b remove,fullscreen
```

#### LXterminal hide menu and scrollbars
To hide menu and scrollbar go into terminal preferences



####Tmux cheatsheet
```
sudo apt-get install -y tmux
```
```
Ctrl -b
```

Create new tmux and name it start:
Tmux new -s start
Close session
Ctrl-b type d twice
Reattach
Tmux attach -t start

%  vertical split
"  horizontal split
o  swap panes
q  show pane numbers
x  kill pane
sudo apt-get install tmux. Also check out the manpage with man tmux.
You can start it by typing tmux on one of your consoles (see XTL's answer).
Here are the most important commands (C-b d means: press control and B at the same time, then press D):
C-b d detach session
tmux attach on the shell to re-attach a running session
C-b " split the current frame horizontally (new shell is started)
C-b % split the current frame vertically (new shell is started)
C-b arrow (up, down, left, right) navigate between windows in current frame
C-b c new frame (new shell is started)
C-b n next frame
C-b l last frame
C-b b send C-b to the running application

sudo tmux tmux new-session -s diego
CTRL + b -> %
CTRL + b -> o

sudo tmux attach -t diego
CTRL + b -> d


Iftop on mac

up vote2down vote
You can remedy things in your current shell by doing:
mkdir -p /usr/local/sbin export PATH=${PATH}:/usr/local/sbin brew link iftop
That'll get you past the warnings and let Homebrew install the iftop package. If the iftop package is installing things in to /usr/local/sbin that you're looking to run, you'll need to ensure this is on your $PATH when you open a shell. To do this, edit ~/.bash_profile and add the line:
export PATH=${PATH}:/usr/local/sbin
To the end of the file to prepend /usr/local/sbin to each new shell you open.

#Install Powerline-status:
(Allows for ip address in bottom bar of tmux)
```
sudo pip install powerline-status
```
show .tmux.conf
```
cd ~/
tmux show -g | cat > ~/.tmux.conf
```
configure
```
https://github.com/passcod/tmux-powerline/blob/master/README.md
```

http://askubuntu.com/questions/283908/how-can-i-install-and-use-powerline-plugin

#Pi Jessie Pixel autostart
check for terminal if not initial screen run command
```
vim /home/pi/.profile
```
```
if [[ ! $TERM =~ screen ]] ; then
	tmuxp load ~/.tmuxp/start.yaml
fi
```

#Update Python
```
sudo apt-get install python-dev
curl -O https://bootstrap.pypa.io/get-pip.py
sudo python get-pip.py
sudo pip install virtualenv
```

#NeoPixel over PWM

##Compile & Install rpi_ws281x Library
These steps will show you how to compile the rpi_ws281x library and install a Python wrapper around it.

To start, connect to a terminal on the Raspberry Pi and execute the following commands to install some dependencies:

```
sudo apt-get update
sudo apt-get install build-essential python-dev git scons swig
```

Now run these commands to download the library source and compile it:
```
git clone https://github.com/jgarff/rpi_ws281x.git
cd rpi_ws281x
scons
``` 

After running the scons command above you should see the library successfully compiled.  Next you can install the Python library by executing:
```
cd python
sudo python setup.py install
```
After running those commands the Python wrapper around the rpi_ws281x library should be generated and installed.

###On Raspberry Pi3 you may need to blacklist the audio driver

#TMUX configuration for powerline
Requires you to update .tmux.conf file with below script

```
source ~/.local/lib/python2.7/site-packages/powerline/bindings/tmux/powerline.conf
set-option -g default-terminal "screen-256color"

set -g @plugin 'seebi/tmux-colors-solarized'

# show host name and IP address on left side of status bar
set -g status-left-length 70
set -g status-left "#[fg=green]: #h : #[fg=brightblue]#(curl icanhazip.com) #[fg=yellow]#(ifcon#     #set -g status-left "#[fg=green]: #h : #[fg=brightblue]#(curl icanhazip.com) #[fg=yellow]#(#

#### COLOUR (Solarized dark)
# default statusbar colors
set-option -g status-bg black #base02
set-option -g status-fg yellow #yellow
set-option -g status-attr default
# default window title colors
set-window-option -g window-status-fg brightblue #base0
set-window-option -g window-status-bg default
#set-window-option -g window-status-attr dim
# active window title colors
set-window-option -g window-status-current-fg brightred #orange
set-window-option -g window-status-current-bg default
#set-window-option -g window-status-current-attr bright
# pane border
set-option -g pane-border-fg black #base02
set-option -g pane-active-border-fg brightgreen #base01

# message text
set-option -g message-bg black #base02
set-option -g message-fg brightred #orange

# pane number display
set-option -g display-panes-active-colour blue #blue
set-option -g display-panes-colour brightred #orange
# clock
set-window-option -g clock-mode-colour green #green
# bell
set-window-option -g window-status-bell-style fg=black,bg=red #base02, red
```

Fix login loop:
```
sudo mv ~/.Xauthority ~/.Xauthority.backup
sudo service lightdm restart
```


#Install ngrok on Raspberry PI
```
sudo wget https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-arm.zip
$unzip ngrok_2.0.19_linux_arm.zip

./ngrok authtoken <put token from dashboard here>

./ngrok tcp 22
```

#enable serial connection on pi3
Use:
```minicom -b 9600 -o -D /dev/serial0```

Follow config here http://spellfoundry.com/2016/05/29/configuring-gpio-serial-port-raspbian-jessie-including-pi-3/

#python deamon
```
sudo easy_install supervisor
```
or
```
sudo pip install supervisor
```
Generate config file
```
sudo su
echo_supervisord_conf > /etc/supervisord.conf
```

#change time clock
Can raspi-config to change region but when that doesn't work 
```
sudo date -s "14 DEC 2016 13:26:00"
```

or 

```
sudo apt-get install ntpdate
sudo ntpdate -u ntp.ubuntu.com
```

#copy files via ssh
```scp -p port number pi@ipaddress:file localpath```

#7 inch 800*480 and 1024*600 Capacitive Touch Screen HDMI

###800*400 
```
sudo nano /boot/config.txt
```
```
hdmi_cvt=800 480 60
```
###1024*600
```
sudo nano /boot/config.txt
```
```
max_usb_current=1
hdmi_group=2
hdmi_mode=87
hdmi_cvt 1024 600 60 6 0 0 0
```


###copy sdcard to harddrive
```
sudo dd bs=4M if=/dev/sdb | gzip > /home/your_username/image`date +%d%m%y`.gz
```

###burn image to sdcard
```
sudo gzip -dc /home/your_username/image.gz | sudo dd bs=4M of=/dev/sdb
```

###install netatalk enables Zeroconf (aka Bonjour) networking, so the system appears on the network as “mypi.local” (or whatever hostname you configured) instead of a numeric IP address:

sudo apt-get -y install netatalk

As a bonus for Mac users, this also enables AppleTalk sharing, which can make it easier to transfer files to and from the system if needed.

With netatalk installed, you can easily access the Raspberry Pi remotely using an ssh client from another system on the network. For example, using the Terminal application in Mac OS X, one would type:

ssh pi@mypi.local

You should get a password prompt. Once logged in, you can perform all administration duties remotely (including the steps that follow), and the monitor and keyboard are no longer needed on the Raspberry Pi.

###Twitch Stream
```
sudo apt-get install libmp3lame-dev; sudo apt-get install autoconf; sudo apt-get install libtool; sudo apt-get install checkinstall; sudo apt-get install libssl-dev

sudo apt-get install libx264-142 libx264-dev

mkdir /home/pi/src
cd /home/pi/src
git clone git://git.videolan.org/x264
cd x264
./configure --host=arm-unknown-linux-gnueabi --enable-static --disable-opencl
make
sudo make install

cd
cd /home/pi/src
sudo git clone git://source.ffmpeg.org/ffmpeg.git
cd ffmpeg
sudo ./configure --enable-gpl --enable-nonfree --enable-libx264 --enable-libmp3lame
sudo make -j$(nproc) && sudo make install

raspivid -o - -t 0 -vf -hf -fps 30 -b 6000000 | ffmpeg -re -ar 44100 -ac 2 -acodec pcm_s16le -f s16le -ac 2 -i /dev/zero -f h264 -i - -vcodec copy -acodec aac -ab 128k -g 50 -strict experimental -f flv rtmp://a.rtmp.youtube.com/live2/YOURCODEHERE
```

```rtmp://<twitch-ingest-server>/app/<stream-key>```


##turn off internal wifi and bluetooth on pi3
update firmware first
```sudo nano /boot/config.txt```
add:
```
# turn wifi and bluetooth off
dtoverlay=pi3-disable-wifi
dtoverlay=pi3-disable-bt
```

#test for camera from command
```vcgencmd get_camera```

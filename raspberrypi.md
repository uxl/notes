
#Disable Screen Blanking (Screensaver)
Disable text terminals from blanking
change two settings by 
```
sudo nano /etc/kbd/config
```
```
BLANK_TIME=0
POWERDOWN_TIME=0
```

```
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```

```
#@xscreensaver -no-splash
@xset s off
@xset -dpms
@xset s noblank
```

#Turn off cursor
```
sudo apt-get install unclutter

```
```
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```
```
@unclutter -idle 0.01 -root
```

```
#@xscreensaver -no-splash


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

```
from CPU ssh pi@new_hostname
```

###Install npm and latest node
```
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
```
```
sudo apt install -y nodejs
```

Check installation:
```node -v```  



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

###Start Tmux fullscreen


####install requirements  
    sudo apt-get install -y tmux
    sudo pip install tmuxp
    sudo apt-get install wmctrl
    sudo apt-get install -y htop
    sudo apt-get install -y iftop

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
    @lxterminal --command "/home/pi/app/startup_terminal.sh

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

#Setup Pi

###change password
```
passwd
```

###bonjour name
Change name from raspberrypi to a name that is more meaningful
```
sudo nano /etc/hostname
sudo nano /etc/hosts
```

###Install npm and latest node
```
wget https://nodejs.org/dist/v4.3.1/node-v4.3.1-linux-armv6l.tar.xz tar xf node-v4.3.1-linux-armv6l.tar.xz cd node-v4.3.1-linux-armv6l/
```
```
sudo cp -R * /usr/local
```
Check installation:
```node -v```  

###Flip Raspberry Pi TouchScreen
```lcd_rotate=2```

###Disable rainbow boot screen
```sudo nano /home/pi/config.txt```

add the following line:
```disable_splash=1```


###Start Tmux fullscreen


####install requirements  
    sudo apt-get install -y tmux
    sudo pip install tmuxp
    sudo apt-get wmctrl
    sudo apt-get install -y htop
    sudo apt-get install -y iftop

####scripts
    **create shell script to launch tmux**
    ```
    !#/bin/bash
    tmuxp load nodeapp.yaml
    ```

#### update autostart
    sudo vim /home/pi/.config/lxsession/LXDE-pi/autostart
    @lxterminal --command "/home/pi/app/startup_terminal.sh

#### tmuxp
***save***  
tmuxp freeze  

***load***  
tmuxp load <yoursavedconfig>.yaml


#### nodeapp.yaml
```
vim /home/pi/.tmuxp/nodeapp.yaml
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
  window_name: python
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
sudo apt-get install -y tmux
Ctrl -b
Create new tmux and name it brian:
Tmux new -s brian
Close session
Ctrl-b type d twice
Reattach
Tmux attach -t brian

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

#Powerline-status install guide
Allows for ip address in bottom bar of tmux
http://askubuntu.com/questions/283908/how-can-i-install-and-use-powerline-plugin

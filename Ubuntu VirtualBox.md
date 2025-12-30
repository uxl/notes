####Add Shared volume in settings

####Add more CPU's if you get errors on startup

####Need Virtualbox extension pack installed first
VirtualBox > Devices > Insert Guest Additions CD image

####After adding shared directory:
sudo adduser <username> vboxsf

####To get a normally plug and play device to work:

1. Shut down the guest machine.
2. Highlight the guest machine in the VirtualBox panel.
3. Plug your usb device.
4. Click on blue USB category.
5. Click the green plus icon to add your device.
6. Your usb device should now be visible, click to add that.
7. Done.

###copy sdcard to harddrive
```
sudo dd bs=4M if=/dev/sdb | gzip > /home/your_username/image`date +%d%m%y`.gz
```

###burn image to sdcard
```
sudo gzip -dc /home/your_username/image.gz | sudo dd bs=4M of=/dev/sdb
```

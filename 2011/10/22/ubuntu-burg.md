# [ubuntu]Burg

今天再一次升级11.10,其实还是11.04好用..反映迅速,或许是我的CS硬件已经不适应潮流了吧 - -。 升级之后,需要重新安装BURG.转一篇资料. To Install Burg in Ubuntu 11.04 Natty Narwhal and Ubuntu 11.10 11.10 "Oneiric Ocelot" , Open termianl and enter the following commands :

```bash
sudo add-apt-repository ppa:n-muench/burg
sudo apt-get update
sudo apt-get install burg burg-themes
```

Now to make BURG integrate the HardDisk enter the following command in terminal:

```bash
sudo burg-install "(hd0)"
```

Remember to substitute ‘hd0′ with the drive on which your MBR is installed. Now update burg:

```bash
sudo update-burg
```

Finally, run the following to test and set your theme:

```bash
sudo burg-emu
```

Press F2 To change theme . Press F3 to change resolution If you want to use another easy way to configue burg, you need to install Burg- manager, for this check our previuos post. Here are the hot-keys defined in the boot menu: t - Open theme selection menu f - Toggle between folding mode n - Jump to the next item with the same class w - Jump to the next Windows itemu - Jump to the next Ubuntu item e - Edit the command of current boot item c - Open a terminal window 2 - Open two terminal windows h - Display help dialog (only available in sora theme) i - Display about dialog (only available in sora theme) q - Return to old grub menu F5/ctrl-x - Finish edit F6 - Switch window in dual terminal mode F7 - List the folded boot items F8 - Toggle between graphic and text mode F9 - shutdown F10 - reboot ESC - quit from the current popup menu or dialog.


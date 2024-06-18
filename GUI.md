# Explanation
This File contains information about how exactly you can get a gui for arch linux

## Gnome Desktop environment
Open the Arch Linux Terminal & execute the following command.
**Be sure to only press enter when it asks you something.**
This ensures, that the default setting is choosed.

```bash
sudo pacman -Syu
```

![image](https://github.com/H14d3n/windows-account-creator/assets/146072924/78ec8ff5-2913-448b-940b-014cad1fbaff)

Next we'll have to Install the Xorg Package before moving to the GNOME Installation. 
So do the following:

```bash
sudo pacman -S xorg
```

![image](https://github.com/H14d3n/windows-account-creator/assets/146072924/72134193-17eb-4bd7-84e5-c13c4ba28d55)

Now it's time to Install GNOME on Arch Linux. 
Proceed like this:

```bash
sudo pacman -S gnome
```

![image](https://github.com/H14d3n/windows-account-creator/assets/146072924/0c8f9036-062a-4be9-b2e7-1567cebcf81d)

After that, the GNOME Desktop environment is installed. Now we have to Enable it.

```bash
sudo systemctl enable gdm.service
```
![image](https://github.com/H14d3n/windows-account-creator/assets/146072924/329b636e-16f2-47da-bcf3-cd414aeda093)

Be sure to start the GNOME Service after enabling.

```bash
sudo systemctl start gdm.service
```

![image](https://github.com/H14d3n/windows-account-creator/assets/146072924/310ce83c-ec35-478c-bfab-a89aa61c58a8)

reboot and you're good to go.





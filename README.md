# Spin up secondary WSL Ubuntu Distrbution

At home, I've decided to do all of my programming on Linux; however, I still need to use Windows for some tasks. Windows allows Linux distributions to run within Windows using WSL. The distribution I've chosen is Ubuntu 22.04 (Jammy). The added caveat is, I will be running multiple instances of Ubuntu for different programming tasks. I want to limit the amount of damage I do if I should corrupt my instance. I also use VSCode as my main IDE. The extensions can start to get difficult to manage the more you include, especially if you aren't using them a lot. 

The following is how I setup multiple Ubuntu distributions on my Windows machine for these various programming tasks like: C#, Go, TypeScript, and Terraform.

## Ubuntu Cloud Images

In order to spin up new distributions of the same type, we can use the `wsl` command to install a new distribution. This will allow us to have multiple instances of the same distribution running on the same machine. However, we need to download the image first.

Ubuntu provides cloud images for WSL. These images are available at the following link: [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/releases). Windows comuters will use the amd64 architecure and use the `root` image.

`ubuntu-22.04-server-cloudimg-<architecture>-root.tar.xz`

```powershell
Invoke-WebRequest -Uri "https://cloud-images.ubuntu.com/releases/jammy/release/ubuntu-22.04-server-cloudimg-amd64-root.tar.xz" -OutFile "C:\temp\ubuntu-distribution.tar.xz"
```

## Install Ubuntu Distribution

The image is used to create a virtual hard drive that WSL uses to run the distribution. However, the image, once used for a distribution, can't be used for another distribution. Create a permenant copy of the image before installing the distribution. Usually I create a directory just off the main user directory to store the images.

```powershell
New-Item -ItemType Directory -Force -Path "C:\Users\<local user>\wsl\distros\<distribution name>"
Copy-Item "./ubuntu-distribution.tar.xz" "C:\Users\<local user>\wsl\distros\<distribution name>"
```

Once the image has been downloaded, we can install the distribution using the `wsl` command. The `--import` flag is used to specify the name of the distribution and the location of the image. The `--version` flag is used to specify the version of the distribution.

```powershell
wsl --import <distribution name> "C:\Users\<local user>\wsl\distros\<distribution name>" .\ubuntu-distribution.tar.xz
```
## Setup new Distribution

Once the new distribution is imported into WSL, we can start the distribution and set up the user account. The `wsl` command is used to start the distribution. The `--distribution` flag is used to specify the distribution to start. 

```powershell
wsl --distribution <distribution name>
```

The distribution will start already logged into the `root` account. From here, it isn't recommended that the `root` account be used for day to day operation of the distro. So we'll create a new user account and set up the distribution to use that account. The new user account will have access to the sudo command so that it can be used to perform administrative tasks.

```bash
root@DESKTOP-NONYA81:/mnt/c/Users/***/Downloads# adduser <username>
Adding user `<username>' ...
Adding new group `<username>' (1000) ...
Adding new user `<username>' (1000) with group `<username>' ...
Creating home directory `/home/<username>' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for shawnmikelson
Enter the new value, or press ENTER for the default
        Full Name []: <username>
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
root@DESKTOP-NONYA81:/mnt/c/Users/***/Downloads# usermod -aG sudo <username>
root@DESKTOP-NONYA81:/mnt/c/Users/***/Downloads# visudo
```

The `visudo` command is used to edit the sudoers file. This file is used to specify which users have access to the sudo command. The file is opened in the default text editor however, the command will run a verification on the file when it is closed. Add the following line at the bottom of the file to give the new user access to the sudo command without requiring a password being entered. Once this has been added, hit `Ctrl + S` to save the file and `Ctrl + X` to exit the editor.

```bash
<username> ALL=(ALL) NOPASSWD:ALL
```
## Set New account as Default User

Before exiting the distribution, modify the `/etc/wsl.conf` file to set the default user to the new user account. This will allow the new user account to be used to start the distribution.

```text
[user]
default=<username>
```

## Restart the distribution

Once the new user account has been set up, exit the distribution and restart it. The new user account should be used to start the distribution.

```powershell
wsl --terminate <distribution name>
wsl --distribution <distribution name>
```

You can now use the new distribution to perform tasks. 
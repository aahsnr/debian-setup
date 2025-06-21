# Explanation 1:

Great, I’ll dive into the code from the GitHub repository at [https://github.com/sudorook/debian](https://github.com/sudorook/debian) and provide a complete, beginner-friendly explanation of how everything works. This will include:

* A breakdown of all files and folders
* An explanation of each script or configuration file
* How the code fits together to achieve its goals
* How to run the project, modify it, and build on it
* Educational context for the Linux, programming, and scripting concepts involved

I’ll get started and let you know when the detailed guide is ready.


# Overview of the Project

The **sudorook/debian** repository is a collection of shell scripts that automate installing Debian Linux and setting it up with your preferred packages and configurations. In plain terms, instead of doing a manual Debian install and then painstakingly installing desktop environments, software, themes, etc., these scripts do it for you. As the README explains: *“This is a set of scripts installing Debian and running post-installation tasks, e.g. installing a desktop environment, packages, and config files.”*. The scripts aim to set up popular desktop environments like Cinnamon, GNOME, or KDE, along with many common utilities and settings, so you end up with a ready-to-use system without doing each step by hand.

Before using the scripts, you’ll need a few basic tools installed:

* **curl** (for downloading content from the internet),
* **fzf** (a fuzzy-search tool that makes menu selections easier),
* **git** (for downloading the repository itself), and
* **sudo** (so the scripts can run commands as the system administrator).

Make sure these are installed on your Debian system (they’re usually available via `apt-get`). In a terminal you might run:

```
sudo apt update
sudo apt install curl fzf git sudo
```

Once you have the repository on your machine (for example by running `git clone https://github.com/sudorook/debian.git` and then `cd debian`), you can explore the code. The main work is done by two scripts at the top level: **`install`** and **`postinstall`**. The `install` script handles partitioning the disk and installing a minimal Debian system, and the `postinstall` script (run *after* the first script) installs all the extra packages, desktop environments, and configurations.

In addition to these scripts, the repository has several folders:

* **`packages/`** – Text files listing groups of Debian package names (e.g. base system, desktop apps, utilities).
* **`dotfiles/`** – Personal configuration files (like `.bashrc`, `.vimrc`, etc.) that get placed in your home directory.
* **`configs/`, `dconf/`, `hooks/`** – Various configuration files and system hooks (for example, saved GNOME settings or systemd-boot hooks).
* **`functions/`, `utils/`** – Helper shell scripts and functions used by the main scripts (to avoid code duplication).
* **`sources/`** – Custom APT source list entries, if needed.
* **`daemons/`** – Any custom service files (not used in most setups).
* **`tests/`** – Automated tests for developers (you can ignore these as an end user).
* **`LICENSES`** – The project’s license (GPL-3.0).

Each part connects together: the `install` and `postinstall` scripts call functions from `functions/`, use package lists from `packages/`, apply configurations from `configs/` and `dconf/`, and finally link in your `dotfiles/`. Overall, reading through the code will show that it’s primarily Bash shell scripting with commands like `sgdisk` (for partitioning), `debootstrap` (for a minimal Debian install), `apt-get` (for installing packages), and various file operations (`mkdir`, `cp`, `ln`, etc.) to set things up.

# How to Run the Scripts

## 1. Get the Code and Prepare

First, you need to have the scripts on your machine. If you haven’t already, open a terminal and run:

```
git clone https://github.com/sudorook/debian.git
cd debian
```

This downloads the entire repository. You should see files like `install`, `postinstall`, and folders like `packages/` and `dotfiles/`. Make sure these scripts are executable:

```bash
ls -l install postinstall
```

You should see an “x” in the permissions (like `-rwxr-xr-x`). If not, run `chmod +x install postinstall` to make them executable. Also ensure you have backed up any important data, because the `install` script will repartition a disk!

## 2. Running the Install Script

The `install` script must be run as root (administrator) because it will partition disks and install the OS. From within the repository directory, run:

```
sudo ./install
```

(You’ll be prompted for your password.) This starts the Debian installation process on a *chosen block device* (like a hard drive or SSD). The script will first show options on how to partition your disk:

* It offers **Partitioning schemes** (e.g. LVM on LUKS, plain LVM, or plain primary partitions).
* It offers **Bootloader options** (e.g. GRUB or systemd-boot).
* It will ask for things like hostname, username, passwords, locale, and timezone.

You’ll navigate these prompts in the terminal. For example, if you choose “LVM on LUKS”, the script will create a large encrypted partition (using LUKS) and inside it make logical volumes for swap, root, and home. If you pick “Plain”, it will just make normal primary partitions. **GPT partition table** is used, which means partitions are labeled in MiB (e.g. “500M” is \~500 mebibytes).

For the bootloader, GRUB is the default. If you have UEFI firmware, it will make an EFI partition and install GRUB there. The script can even encrypt `/boot` under GRUB so you only enter your decryption password once. (If you choose systemd-boot, it will copy kernels to the EFI partition and set up hooks so updates still work.)

Once you’ve answered all prompts, the script proceeds automatically:

* It **partition the disk** using the `sgdisk` tool (like `sgdisk -Z` and `sgdisk -n` commands) to set up the partitions you selected.
* It then **formats and mounts** these partitions. For example, it might do `mkfs.ext4 /dev/sdx3` for the root partition, `mkswap` for swap, and `mount` them under `/mnt`. If LUKS encryption was chosen, it will run `cryptsetup` to open the encrypted container first.
* It uses **`debootstrap`** to install a minimal Debian system into `/mnt`. Debootstrap is a Debian tool that downloads and unpacks the core base-system packages into a directory. In effect, it builds the initial system (including the Linux kernel and basic utilities). The README notes that it installs all the “base” and “base-devel” packages via debootstrap.
* Next, it **chroots** into `/mnt` (using `chroot /mnt /bin/bash`) to configure the new system. In chroot, it sets the hostname, creates the user you specified, locks the root account (for security), and does any final configuration. (Locking root means you won’t log in directly as root; instead you use `sudo` with your new user.)
* Finally, it **unmounts everything** and exits.

At this point, you have a freshly installed Debian base system on the target disk. You can reboot into it (if you were running from a USB or live CD when doing this), or proceed to the next step.

> **Tip:** If you’re unsure, try this in a virtual machine (VirtualBox, QEMU, etc.) first. That way you can experiment without risking real data.

## 3. Running the Post-Install Script

After the base system is installed and you have rebooted into it (as your normal user, not root), you run the second script:

```
./postinstall
```

This script installs everything else – desktops, applications, and personal settings. When you start it, it will first check that required tools (like `curl`, `fzf`, network connection) are present. Then it will show you a text-menu with numbered sections. The main sections are:

* **Autopilot**: One-click mode that just does everything without asking questions.
* **Base**: Tasks like installing core packages, enabling extra repositories (contrib, non-free), updating the system, and removing unwanted defaults.
* **Miscellaneous**: Utilities, power management (TLP), SELinux support, alternative shells, etc..
* **Desktop Environment**: Here you choose a full desktop. Options include GNOME, Cinnamon, or KDE.
* **Network Tools**: Installs network and SSH tools (NetworkManager, OpenSSH), firewall (UFW), Tor, etc..
* **Applications**: A huge menu of various software categories – media tools, programming tools, games, virtualization (KVM/VirtualBox), Wine, etc..
* **Themes & Fonts**: Downloads and installs UI themes (Arc, Adapta, Materia, Papirus icons, etc.).
* **Personalization**: Sets your default fonts, icon themes, shell (bash or zsh), terminal profiles, dark mode, and other preferences.

Each menu item then drills down into more options. For example, under **Applications** you might see an option to install all development tools, or just “Vim”, or “Neovim”, or “VirtualBox host”, etc. Most of these choices correspond to package lists found in the `packages/` folder. For instance, choosing “GTK applications” would cause the script to install every package listed in `packages/apps.list`.

The scripts use `apt-get` (or `apt`) under the hood to install everything. They typically run commands like `sudo apt-get install -y $(cat packages/base.list)` to install a whole list of software. (The `-y` means “yes to all prompts” so it won’t stop to ask you about each package.)

If you pick the **All** option in any menu (like “2) All” in Desktop Environment or Applications), it just runs through all sub-items in order. The **Autopilot** mode is even more aggressive: it takes every “all” option without waiting, giving you a complete setup by the end.

At the end of post-install, the script also sets up your **dotfiles**. The `dotfiles/` directory contains hidden configuration files (like `~/.bashrc`, `~/.zshrc`, `~/.vimrc`, etc.) that define your shell environment and editor settings. The script typically creates *symbolic links* so that your home directory points to these files (e.g. `ln -s /path/to/debian/dotfiles/.bashrc ~/.bashrc`). This way, you get a pre-configured shell setup (often using [Oh My Zsh](https://ohmyz.sh/) or powerline prompts).

Finally, the script will often re-run `sudo ldconfig` or other commands to finalize things, and then prompt you to reboot into your new, fully configured Debian system.

# How the Code Works (File by File)

Below is a high-level tour of the main parts of the code and the key Linux concepts involved. You don’t need to be a programmer to follow this; think of each part as a recipe of commands or config files.

## The `install` Script

This is a Bash shell script (you’ll see `#!/bin/bash` at the top). It does roughly this:

1. **Device Selection**: It uses `lsblk` or similar to list available disks and asks you which one to install on. This is the “prompted block device” in the README.
2. **Partitioning**: Depending on your choices (LUKS, LVM, etc.), it runs `sgdisk` to create partitions. For example, it might run something like `sgdisk -a1 -n1:34:2047 -t1:ef02 /dev/sdx` to make a BIOS boot partition, then `-n2:0:+512M -t2:ef00` for EFI, and so on. (These commands come from the `sgdisk` manual; you don’t need to memorize them, but they mean “create partition #2 starting at sector 0 with size +512M and type EF00 (EFI System)”, etc.)
3. **Formatting**: It formats the new partitions. For example, `mkfs.fat -F32` for the EFI, `mkfs.ext4` for root/home, and `mkswap` for swap. If LVM was chosen, it would use `pvcreate`, `vgcreate`, and `lvcreate` to make logical volumes, then format them. If LUKS was chosen, it would run `cryptsetup luksFormat` and `cryptsetup open`.
4. **Mounting**: It mounts the root partition on `/mnt` (or a temporary directory), e.g. `mount /dev/mapper/vg0-root /mnt`. Swap is activated with `swapon`. If you made a separate home or boot, those are mounted under `/mnt/home` or `/mnt/boot` accordingly.
5. **`debootstrap`**: This is a special Debian tool (borrowed from the Arch Linux “pacstrap” concept). The script runs something like `debootstrap --include=... bullseye /mnt http://deb.debian.org/debian` (using your chosen release and mirror). This downloads core Debian packages and unpacks them into `/mnt`. It builds the very base system.
6. **Chroot Setup**: After debootstrap, the script “jumps into” the new system with `mount --bind /dev /mnt/dev`, `mount -t proc /proc /mnt/proc`, etc., then `chroot /mnt /bin/bash`. Now your shell is running “inside” the new system. Inside the chroot, it configures `/etc/hostname`, `/etc/fstab` (by generating it with something like `genfstab`), sets root’s password to locked (`passwd -l root`), and creates your user (`useradd -m -G sudo $username` plus `passwd $username`). The user we create is given sudo rights, so you can administer the system from that account later.
7. **Bootloader**: Still in the chroot, the script installs GRUB (for BIOS) or GRUB EFI (for UEFI). This involves running `grub-install` and `update-grub` so that the system will boot. If you picked systemd-boot, the script would copy kernels to EFI and set up `/boot/loader/entries`.
8. **Cleanup**: It unmounts `/mnt/dev`, `/mnt/proc`, and `/mnt` itself, finishing the installation.

In code terms, all of the above is done by a sequence of Bash commands. For example, you might see in the script:

```bash
sgdisk -Z $dev          # wipe existing partitions
sgdisk -n 1:1MiB:+1MiB -t 1:ef02 $dev   # BIOS partition if needed
sgdisk -n 2:0:+512MiB -t 2:ef00 $dev    # EFI partition
...and so on...
mkfs.ext4 ${rootpart}
mount ${rootpart} /mnt
debootstrap --include=...
mount --bind /dev /mnt/dev
chroot /mnt bash -c "echo '$HOSTNAME' > /etc/hostname; passwd -l root; useradd -m -G sudo $USER"
...
grub-install ... && update-grub
```

Each of these lines is a familiar Linux command. The script may define some functions (from `functions/`) to make this more readable, but essentially it’s automating exactly what you’d do manually in a Debian install guide.

## The `postinstall` Script

This script is also Bash. It defines a big menu of actions and waits for you to type a number. Under the hood it might use `read` or even a dialog program (but more likely just simple text prompts).

When you pick an option, it often calls a function or a series of commands. For instance, “Install base packages” will loop through the file `packages/base.list` (which has names of important packages) and run `sudo apt-get install -y <that-package>`. The **packages/** folder holds many such list files (e.g. `apps.list`, `utils.list`, `3d-accel.list`, etc.). For example, the README references “base.list”, “utils.list”, and others when describing the menu.

Some important actions performed by postinstall include:

* **Updating the system**: It might run `sudo apt-get update && sudo apt-get upgrade -y` to make sure you have the latest packages.
* **Enabling repositories**: It can edit `/etc/apt/sources.list` to add “contrib”, “non-free”, or “non-free-firmware” components (these contain software like graphics drivers or firmware that aren’t in the main free Debian area).
* **Installing command-line utilities**: For example, under “Miscellaneous” the script can install the packages in `packages/utils.list` (likely things like `vim`, `tmux`, `htop`, etc.).
* **Desktop environments**: If you pick GNOME, it does `sudo apt-get install -y gnome-core gdm3` (for example). If Cinnamon: `apt-get install cinnamon lightdm xserver-xorg-video-intel` (just as a guess). These sets are hard-coded or in list files. The README mentions options 3/4/5: “Install GNOME desktop (with GDM)”, “Cinnamon (with LightDM)”, “KDE (with SDDM)”.
* **Copying dotfiles and dconf**: The script will copy the contents of `dconf/` into your user’s settings using `dconf load`. It will link the `dotfiles/` to your home. This applies custom keyboard shortcuts, terminal themes, etc.
* **Enabling services**: For example, if you chose to install Tor, it might do `sudo systemctl enable tor`.
* **System tweaks**: Some choices do things like disabling the system bell, adding “sudo insults” (a fun apt-get package that makes sudo scold you), and other small tweaks.

Everything in `postinstall` is essentially a series of shell commands that you could type by hand. The advantage of the script is it organizes them into menus, and uses non-interactive `apt-get` so it won’t stop to ask you anything.

## Other Files and Concepts

* **`packages/` folder**: Contains plain text `.list` files. Each line is a Debian package name. The scripts read these (with something like `xargs sudo apt install -y < packages/<something>.list`). For example, `packages/android.list` might have tools to manage Android devices (adb, fastboot, etc.), `packages/firewall.list` might list ufw, etc.

* **`functions/` folder**: These are Bash snippets with reusable code. For instance, there might be a `functions/install.sh` defining a function to install a list of packages, or `functions/luks.sh` for encrypting partitions. The main scripts can “source” these files (using `source functions/install.sh`) so they can call the functions. This keeps the code organized. For a beginner, think of this like breaking a large recipe into smaller sub-recipes (e.g. one function “make\_swap()”, another “make\_root\_part()”, and so on).

* **`utils/` folder**: Likely contains helper scripts. One example from searching is a `genfstab` script (which on Arch Linux generates an `/etc/fstab` file listing all partitions by UUID). Indeed, during install the script probably runs something like `utils/genfstab -U /mnt >> /mnt/etc/fstab` to auto-generate fstab entries (this was hinted at in the code comments). It might also have scripts to install firmware, or to configure the bootloader. These are little tools written in shell or other languages that the main scripts call when needed.

* **Dotfiles and Symbolic Links**: The term *dotfile* means a file starting with a dot, e.g. `.bashrc`, which stores settings. In `dotfiles/` you’ll find many such files. Instead of copying them, the script will likely create *symbolic links* (using `ln -s`). A symbolic link is like a shortcut – it means “\~/.bashrc points to debian/dotfiles/.bashrc”. That way, editing the dotfile in the repository immediately reflects in your home. It also means your home directory stays clean and organized. Explaining *symbolic link*: it’s like having a pointer or alias. The command `ln -s /path/to/source /path/to/link` creates one. (By contrast, a *hard link* is a lower-level connection; scripts here almost certainly use symbolic links.)

* **File Permissions**: Many scripts change file modes with `chmod`. For example, `chmod +x` makes a script executable (if you double-clicked it, or ran it with `./script`, the “+x” bit must be set). You’ll often see `chmod 755` or similar. Regular files like config text usually have `644` (read/write for owner, read-only for group/others). When user accounts are created, their home directories get permissions like `700` or `755` so only they (and root) can access them.

* **`.gitignore`**: This file lists patterns of files that Git should ignore (not track). For example, build artifacts or personal secrets. You don’t need to worry about it for running the scripts; it just keeps the Git repository clean.

* **Shell Scripting Basics**: These scripts are just text files with commands. Each line is a command (or part of an `if/then`, `for` loop, or function definition). Comments start with `#`. Variables are used (like `$HOSTNAME` or `$USER`). Control flow (`if`, `while`, `case`) is used to branch on your menu choice. For instance, something like:

  ```bash
  echo "Choose desktop: "
  echo "3) GNOME"
  read choice
  case $choice in
    3) sudo apt-get install -y gnome-core gdm3 ;;
    4) sudo apt-get install -y cinnamon lightdm ;;
    5) sudo apt-get install -y kde-standard sddm ;;
  esac
  ```

  This is a simplified example of how the code presents a menu and acts on your input. Even if you’ve never programmed, you can recognize that “case” is checking which number you entered and then running an `apt-get install` accordingly.

* **Chroot**: As mentioned, `install` uses `chroot` to switch into the new system. If you’re not familiar: *chroot* stands for “change root” and it means “pretend the directory /mnt is really /”. All commands after that see `/mnt` as the top of the file system. It’s like stepping inside the new system as if you had booted into it, so you can run `passwd`, `apt-get`, etc., inside the new environment. After chroot work is done, the script “exits” and you go back to the original environment.

* **Encryption and LVM**: The script can set up LUKS encryption (this uses `cryptsetup luksFormat` to encrypt a partition with a passphrase, and `cryptsetup open` to unlock it). It can also use LVM (Logical Volume Manager) which is like a layer that lets you combine partitions or resize them flexibly. If you chose “LVM on LUKS”, it actually layers LVM *inside* an encrypted container. (So your root and home become logical volumes inside the encrypted LUKS volume.) All these details are automated by the script.

* **GRUB and bootloader**: GRUB is the program that starts the Linux kernel at boot time. The script installs it with something like `grub-install --target=x86_64-efi /dev/sdx`. It then updates GRUB config so it knows about the new kernel (`update-grub`). If you selected “encrypt /boot”, GRUB itself is in a tiny unencrypted partition, but the Linux kernel in /boot is encrypted. The script handles this by generating an initramfs that knows how to unlock LUKS at boot.

* **APT Sources**: The `sources/` folder may contain alternate `/etc/apt/sources.list` entries (for example, switching from DVD to online repos, or adding backports). The postinstall script likely copies these into place or appends them, using commands like `sed` or `tee`.

* **Making Changes and Building On It**: Because everything is script files, you can edit them in a text editor. For example, if you want to add a new package to the “Music” list, open `packages/audio.list` and add the package name on a new line. If you want to change a theme, you could edit `postinstall` to use a different Git URL. The license is GPL-3.0 (seen in the source headers), so you’re free to modify it as long as you also share changes. Always test changes on a spare system or VM first.

# Beginner Tips and Concepts

* **Running Scripts Safely:** Always double-check which disk you’re installing on. The script will ask (for example) `/dev/sda` or `/dev/sdb`. Choosing the wrong one could wipe data. If you’re not sure, you can even unplug other drives so only the target is visible.

* **Interactive Menus with `fzf`:** The requirement of `fzf` means that some parts of the script let you fuzzy-search for items. For example, you might get a list of keyboard layouts or font names and type a few letters to filter. It makes the interface faster than scrolling through a long numbered list.

* **What is a “dotfile”?** It’s a configuration file whose name begins with a dot (e.g. `.bashrc`). By convention, Unix-like systems hide these files in your home directory. These configure shells, editors, and other tools. This project includes example dotfiles in its `dotfiles/` directory, which will be linked into your home so you get those configurations.

* **Permissions and `sudo`:** In Linux, only the root user can modify system-wide things (like partitions, `/etc/apt`, `/usr/bin`). `sudo` is a way for a normal user to run a single command as root (it stands for *superuser do*). The installation script uses `sudo` or runs as root for any privileged operation. That’s why you use `sudo ./install`. After the base install, the new user is given sudo rights (so you don’t have to enable root login).

* **Analogies:**

  * *Partitioning a disk* is like dividing a hard drive (hard disk) into separate rooms or drawers – one for system, one for swap, one for home.
  * *Encryption (LUKS)* is like putting a lock on a drawer – you need a password to open it.
  * *LVM* is like having adjustable shelves – you can resize your “rooms” or add more space easily later.
  * *Filesystem (ext4)* on a partition is like furnishing a room – it lets files be stored inside it.
  * *Chroot* is like stepping through a portal into a different house – once you’re “in” it, you think that house’s root (/) is the real root.
  * *Symbolic link* is like a shortcut or alias on your desktop that points to another file: the link looks like a file but really just refers to the original.

* **Experimentation:** Feel free to customize the scripts. For example, if you don’t want KDE, you could remove that menu section or change which packages it installs. If you want a lighter system, you could skip some desktop themes. The code is plain text, so you can see exactly what it will do before you run it.

* **Learning More:** If you see a command you don’t understand (say `sgdisk` or `debootstrap`), you can look them up. Running `man sgdisk` in a terminal will show its manual. Likewise, search the Debian wiki or forums. The scripts here use many standard Linux tools, so understanding them one by one will teach you how Debian systems work under the hood.

# Summary

In short, the `sudorook/debian` project automates a complete Debian install plus user setup. You run `sudo ./install` to do the first-time disk partitioning and base system install, and then as your user run `./postinstall` to add software and personalization. Underneath, it’s just Bash scripts using familiar Linux commands: partitioning (`sgdisk`, `mkfs`), package installation (`debootstrap`, `apt-get`), and file configuration (copying dotfiles, editing config files).

By studying each part of the repository—scripts, package lists, functions, and config files—you can learn both shell scripting and the structure of a Debian system. For example, look inside `packages/` text files to see how software is grouped, or open a script in `functions/` to see a re-used routine. Modify a list or a function, rerun the script, and observe the changes. Over time, you’ll build the skills to tweak this setup or even write your own installer scripts.

**Sources:** The information above is based on the repository’s README and file contents, as well as general Debian documentation on tools like `debootstrap` and `sgdisk`. The README provides the high-level overview and menu descriptions, which guided this explanation.


# Explanation 2: [Enhanced]


Got it! I’ll provide a much deeper, beginner-focused expansion on the sudorook/debian project. This will include:

* Full details of how the install script partitions, encrypts, and formats disks.
* In-depth explanation of how the postinstall script installs packages and personalizes the system.
* Beginner introductions to Bash scripting, symbolic links, Linux filesystems, bootloaders, and encryption.
* File-by-file walkthroughs of the `dotfiles/`, `functions/`, `packages/`, and other key directories.
* Practical tips for safely experimenting, modifying, and extending the project.

I’ll work on compiling this expanded tutorial and let you know when it’s ready.


# Overview of sudorook’s Debian Installer Scripts

The **sudorook/debian** project is a collection of shell scripts that automate installing Debian from scratch and setting up a fully configured desktop environment. It consists of two main scripts: an `install` script that partitions the disk, formats it (optionally encrypting and using LVM), and bootstraps a minimal Debian system; and a `postinstall` script that runs inside the new system to install additional packages and apply user configurations (dotfiles, themes, etc.).

This tutorial walks through how these scripts work, what the user will see, and explains all the related concepts step-by-step for a beginner.  We cover:

1. **Disk Partitioning and Installation** – how the `install` script creates and formats partitions (GPT, LUKS encryption, LVM), runs Debian’s debootstrap to install a base system, and uses `chroot` to configure it.
2. **Post-Installation Script** – how the `postinstall` script installs software by reading package-list files, and how it applies configuration (dotfiles, themes, dconf settings).
3. **Key Concepts Explained** – basics of shell scripting (variables, loops, conditionals), Linux filesystems and mount points (`/`, `/boot`, `/home`, swap, EFI), disk encryption with LUKS and how `initramfs` prompts for passphrase, symbolic links and dotfiles, and tools like `sudo`, `systemd`, `apt`, `grub`, and `systemd-boot`.
4. **Repository Walkthrough** – a guided look at the project’s directories (`dotfiles/`, `functions/`, `packages/`, `configs/`, `dconf/`, `hooks/`, `utils/`), explaining how they are organized and used by the scripts.
5. **Modifying and Extending** – how to safely edit package lists, test changes (e.g. in a VM), and add new menu options or features to the scripts.

Everything is explained in plain English, with examples and analogies.  Citations in brackets link to documentation or code to show where each idea comes from.

## 1. Disk Partitioning and Installation

The `install` script is the first step you run on a new machine (for example, booting from a live USB).  It automates creating partitions and filesystems and then installing Debian on them. Here’s how it works:

### Choosing the Partitioning Scheme

When you run the script as **root**, it will first ask which disk to use (e.g. `/dev/sda`).  After selecting a disk, it presents a menu of **partitioning schemes**:

* **LVM on LUKS** – Create one big encrypted partition (LUKS) and inside it create an LVM volume group with logical volumes (for root, home, swap, etc.).  This is *full disk encryption*: everything (except an EFI boot partition) lives inside an encrypted container unlocked at boot.
* **LVM (unencrypted)** – Do *not* encrypt; just use LVM directly on the disk.  This still gives flexible resizing but without encryption.
* **Plain (no LVM or LUKS)** – Use regular primary partitions (e.g. ext4) without LVM or encryption.
* **Cancel/Back** – Exit or go back.

These correspond to the menu options **2) LVM on LUKS**, **3) LVM**, **4) Plain**.  (If you have older BIOS mode or decide not to use a bootloader, the script will still create a simple GPT setup as needed.)

For example, **LVM on LUKS** means the script will create one large **LUKS-encrypted** partition, then inside that set up LVM logical volumes. In contrast, **LVM** means it makes an LVM volume group on unencrypted space.  The README notes: *“Using GRUB will also encrypt `/boot` and write the decryption key into the initrd so that you’ll only be asked for your passphrase once”*.  That refers to the **LVM on LUKS** case with GRUB, where even the `/boot` partition gets encrypted (with a key stored in the initramfs image).

### GPT Partitions Basics

The script uses **GPT (GUID Partition Table)** to label partitions. GPT is the modern standard for disks (required for UEFI boot and supporting large disks).  Unlike the old MBR scheme (which only allowed 4 primary partitions), GPT supports many partitions and uses 64-bit addressing.  In simple terms, GPT stores partition entries by unique IDs and is compatible with most modern Linux setups.

A typical partition layout the script might create is (depending on choices):

* **BIOS (1 MiB)** – A tiny “BIOS boot” partition, only needed if using GRUB on a BIOS (non-UEFI) system. It’s blank and very small (around 1 MiB).
* **EFI (512 MiB)** – On UEFI systems, an EFI System Partition is created (formatted FAT32) where bootloaders live.  The script labels this partition type “EFI”. It is usually mounted at `/boot/efi` in Linux.
* **Shared partition (optional)** – If you selected `Plain` or another option, the script can create a separate “share” partition (often NTFS or FAT) for sharing files with other OSes like Windows.
* **Linux system (rest of disk)** – The remaining space holds your Debian system.  Depending on your choice:

  * In **LVM on LUKS** mode, this space becomes one big LUKS encrypted container, inside which the script will make LVM volumes (for swap, root, /home, etc.).
  * In **LVM** mode, it will be an unencrypted LVM volume group.
  * In **Plain** mode, it will be a normal Linux partition.

The README gives an example layout (GPT, ext4 by default):

> *Example partition scheme (BIOS, GPT, ext4):*
>
> 1. **BIOS compatibility** partition – 1 MiB (empty if GRUB not used).
> 2. **EFI System** partition – 512 MiB (for UEFI boot files).
> 3. **Shared** partition (optional) – formatted (NTFS/VFAT) for sharing with other OSes.
> 4. **Debian** (rest of disk) – use for swap, root, home (inside LVM if selected).

We see from \[6] that partition “3” is an optional share, and “4” is for all of Debian’s volumes.  Inside “4”, if LVM or LUKS are used, the script will carve out logical volumes or encrypted volumes accordingly.

### Creating and Formatting Partitions

Under the hood, the script uses tools like `sgdisk` (for GPT) to create partitions.  For example, it might run commands like:

```sh
# (Example only) Create partitions with sgdisk:
sgdisk --new=1:0:+1M   --typecode=1:ef02 /dev/sda   # BIOS boot (empty)
sgdisk --new=2:0:+512M --typecode=2:ef00 /dev/sda  # EFI (type ef00 = ESP)
sgdisk --new=3:0:+100G --typecode=3:0700 /dev/sda  # Shared (NTFS/VFAT)
sgdisk --new=4:0:0     --typecode=4:8300 /dev/sda  # Debian (Linux)
```

Then it formats each partition with `mkfs` or `cryptsetup` plus `mkfs` if encrypted.  For instance:

* The EFI partition is formatted FAT32 (`mkfs.fat -F32 /dev/sda2`).
* The Linux root partition (or LVM physical volume) is formatted ext4 (`mkfs.ext4`).
* If LUKS is chosen, it runs `cryptsetup luksFormat /dev/sda4` and `cryptsetup open /dev/sda4 cryptroot`, then creates an LVM PV inside `/dev/mapper/cryptroot` (with `pvcreate`).

In **LVM mode**, it might do:

```sh
pvcreate /dev/sda4
vgcreate vg0 /dev/sda4
lvcreate -L 20G -n root vg0    # 20GiB root volume
lvcreate -L 8G  -n swap vg0    # 8GiB swap volume
lvcreate -l 100%FREE -n home vg0  # rest as /home
mkfs.ext4 /dev/vg0/root
mkswap   /dev/vg0/swap
mkfs.ext4 /dev/vg0/home
```

All of these steps are automated by the script using common Linux disk utilities. After partitioning and formatting, the script **mounts** the new filesystems (e.g. it might mount root at `/mnt/debian`, EFI at `/mnt/debian/boot/efi`, etc.).

### Installing Debian with debootstrap and chroot

Once the partitions are ready and mounted, the script installs Debian’s base system using **debootstrap**.  *debootstrap* is a tool that downloads Debian packages and unpacks them into a directory as a minimal system. It does not require the Debian installer or an ISO; it just needs network access to Debian mirrors.

A typical debootstrap sequence is:

```sh
debootstrap --arch=amd64 bullseye /mnt/debian http://deb.debian.org/debian/
```

This installs a very basic Debian (packages `base` and `base-devel`) into `/mnt/debian`.  The README notes it will “download and install all the `base` and `base-devel` packages via `debootstrap`, set up the specified user account, lock the root account, and unmount everything”.  (Locking root means you won’t log in as root, but use sudo instead.)

After debootstrap finishes, the script copies some host files (like `/etc/resolv.conf`) into the new system and then uses `chroot` to enter the new system.  The `chroot` command stands for **change root**: it makes a directory appear to be the top `/` for any process.  In other words, when you run:

```sh
mount -t proc /proc /mnt/debian/proc
mount -t sysfs /sys /mnt/debian/sys
chroot /mnt/debian /bin/bash
```

you get a shell that thinks `/mnt/debian` is `/`. This lets the script configure the new system as if it were running normally (install more packages, configure network, set timezone, etc.).  The *GeeksforGeeks* tutorial explains `chroot` as creating a “jailed” environment where processes are limited to the new root directory.  That is exactly what the installer does for final setup: it enters the new Debian and runs commands to finish setting it up.

Throughout the `install` process, the script prompts the user for a few things (all interactively):

* **Disk to install on** (e.g. `/dev/sda`).
* **Partition scheme** (as above: LUKS/LVM/plain).
* **Bootloader choice**: GRUB (default) or systemd-boot. (If UEFI is used, you can pick systemd-boot, a simple UEFI boot manager; if BIOS or advanced, GRUB is used.)
* **Hostname, username, password** for the new system.
* **LUKS passphrase** (if encryption chosen).
* **Locale and timezone**.

These answers control how the partitions are set up and how Debian is configured.  For example, choosing **GRUB** will install the GRUB bootloader and may encrypt `/boot`; choosing **systemd-boot** means the system will use the Linux kernel’s EFI stub and bootloader on the EFI partition.

After `chroot` configuration, the script unmounts everything and (usually) reboots into the new Debian.  At that point, the base system is installed and you can run the **post-install** script (either automatically on first boot or manually).

## 2. The Post-Installation Script

Once the base Debian system is in place (root `/` mounted, Internet connected), the `postinstall` script can run to add software and tweak settings.  It is typically run from the newly installed system (for example, as your normal user with `sudo postinstall`). This script presents a **menu of options** grouped into sections, and performs installations and configurations accordingly.

### Installing Packages from Lists

The `postinstall` script uses plain text “package list” files (in the `packages/` directory) to know what to install for each category.  For example, `packages/base.list` might contain `sudo`, `vim`, etc.  When you select a menu item, the script does something like:

```sh
xargs -a packages/base.list sudo apt-get install -y
```

It simply runs `apt install` on each list.  Key package lists include:

* **base.list** – essential system packages (sudo, apt-utilities, etc).
* **purge.list** – default packages to remove.
* **firmware.list** – non-free firmware (WiFi drivers).
* **utils.list** – command-line utilities (grep, htop, etc).
* **apps.list**, **apps-kde.list** – general applications (GTK and KDE apps).
* **3d-accel.list**, **android.list**, **extra.list** etc. – specialized packages.
* **desktop environment** lists – e.g. `gnome.list`, `kde.list` (for GNOME, KDE).

For example, the **“Base”** section of the menu lets you do things like:

* Install the base packages (`base.list`),
* Remove unwanted default packages (`purge.list`),
* Install firmware for wireless,
* Update the system,
* Enable `contrib`/`non-free` repos,
* Upgrade Debian release,
* Enable fun things like “sudo insults” (which gives humorous messages on sudo failures),
* Disable the PC speaker beep modules.

The menu label “Base packages” corresponds to installing everything in `base.list`.  The text says “3. Installs base.list.” and “4. Purge packages in purge.list…”, which means option 3 reads and installs from `base.list`, option 4 does the same for `purge.list`.

### Major Menu Sections

After the Base section, the menu has these main sections, each with its own submenu items:

* **Miscellaneous** – things like general utilities, laptop tools, SELinux, shell (zsh), etc.  For example, “3) Linux utilities” installs programs from `utils.list`, and “7) zsh” installs the Z shell and related fonts/themes.
* **Desktop Environment** – options for GNOME, Cinnamon, KDE desktops.  Selecting “GNOME” installs packages for a GNOME desktop (and uses GDM), etc.
* **Network Tools** – install networking software.  E.g. “Networking” installs NetworkManager and OpenSSH, “Firewall” installs UFW, “Install tor” adds Tor, etc.
* **Applications** – many categories for applications (3D graphics, Android tools, general apps, codecs, containers, development, virtualization, etc.).  Each item says which list or packages it installs (e.g. item 3 “3D acceleration” installs `3d-accel.list`).
* **Themes** – desktop theme and icon theme installers.  It can download and compile GTK/KDE themes like Arc, Adapta, etc, and install fonts/icons like Papirus.
* **Personalization** – tweak settings (fonts, shell, login, dark mode, dconf import, etc.).  For instance, options like “Import Cinnamon dconf” or “Select default login shell” or “Enable autologin” are here. These use pre-made config files.

In each section, you can usually choose “All” to do everything in that section.  For example, **“4) Desktop environment”** submenu has “2) All” which would install all supported DEs (GNOME, Cinnamon, KDE) at once.

### Dotfiles and dconf Settings

After installing packages, the script applies user configuration.  The repository contains a `dotfiles/` directory with files like `.bashrc`, `.vimrc`, etc.  The postinstall script will **create symbolic links** (symlinks) in your home directory pointing to these dotfiles so that your shell and applications use them.  A **symbolic link** is like a shortcut or pointer: the system treats it as if the linked file were in your home.  In Linux, a symlink is *“a file whose purpose is to point to a file or directory (the ‘target’) by specifying a path thereto”*.  The script uses the `ln -s` command to make these links.

It also uses the `dconf` command to import desktop configuration settings.  *dconf* is GNOME’s low-level settings database. Think of it as a place where GNOME apps store settings (theme choices, panel layouts, terminal colors, etc.).  The script has pre-made dconf dump files in `dconf/` and uses `dconf load` to apply them.  (This is like restoring a backup of your settings.) The LinuxConfig tutorial notes: *“Dconf is the low-level configuration system used by the GNOME desktop environment. It is basically a database, where the various configurations are stored as keys together with their values.”*.

So in “Personalization” you’ll see options like “Import GNOME dconf” or “Import KDE settings” – choosing those runs something like `dconf load /org/gnome/ < ~/dconf/gnome-settings.dconf` to restore preset settings.

### System Services and Bootloader

The `postinstall` script also enables or restarts services as needed. For instance, installing a desktop might enable `gdm` or `sddm`.  The menu items themselves don’t show service details, but the scripts behind them will often run `systemctl enable <service>` or similar.

If you changed bootloader type in install, the script also handles that.  For example, if **systemd-boot** was chosen, the script runs `bootctl install` inside the new system so that systemd-boot is installed to the EFI partition.  (Systemd-boot is a simple UEFI boot manager included with systemd.) If GRUB was chosen, it runs `grub-install` and updates `/boot/grub/grub.cfg`.

Throughout all these steps, the script provides menu prompts like “Back”, “All”, etc., and descriptions of what each option does.  For example, \[45†L461-L469] shows the Network menu with item “3. Install NetworkManager and OpenSSH” and “4. Install Avahi and Samba”, each with a brief description of what packages they install.

> **Example Menu Entry (from README)**:
> “3. Install Network Manager and OpenSSH. Sets NetworkManager to use random MAC addresses for network interfaces.”.

That means option 3 in the Network menu will `apt install network-manager openvpn openssh-server` etc., and configure NM to use random MACs (for privacy).

Each menu section in the README is documented with the actions (see \[45] and \[47]).  You can refer to those descriptions when running the script to know what each choice will do.

## 3. Key Concepts Explained

This project touches on many fundamental Linux concepts. Below is a brief primer on each, in beginner-friendly terms:

### Shell Scripting Basics

The installer and post-install scripts are **Bash shell scripts**.  That means they are text files of commands for the Linux shell (bash). Key points:

* **Variables:** like `USERNAME="alice"`. They store values (strings, numbers). You can use `$USERNAME` later in the script to refer to that value.
* **Functions:** reusable pieces of code. For example:

  ```bash
  greet() {
      echo "Hello, $1!"
  }
  greet "world"  # prints: Hello, world!
  ```

  Here `greet` is a function taking an argument.
* **Conditionals:** tests and branches. E.g. `if [ "$CHOICE" = "1" ]; then ...; else ...; fi`. This lets the script do different actions based on choices.
* **Loops:** repeating code. E.g. `for pkg in $(cat packages.list); do apt install $pkg; done` will install each package from a list one by one.
* **Reading input:** The script uses commands like `read choice` to get user input, or the `select` command for menus.

Because you’re new, you don’t need deep expertise. But know that bash scripts can automate almost any sequence of shell commands, with logic. The sudorook scripts use these basics to drive menus and act on your answers.

### Linux Filesystems and Mount Points

Linux organizes files in a single tree starting at root `/`.  Here are common directories and mounts:

* **/** – the “root” directory, top of the filesystem. Everything else is under `/`. Only the **root user** can write directly under `/`. Think of `/` as the trunk of a tree, with all other dirs as branches.
* **/boot** – holds kernel images and bootloader files. On non-EFI (BIOS) installs, `/boot` contains GRUB and kernels. On EFI, we usually use an EFI System Partition (often mounted at `/boot/efi`) instead, but `/boot` may still hold kernels.
* **/home** – where user home directories live. E.g. your user “alice” has `/home/alice`. This is where personal documents and dotfiles go.
* **Swap** – not a directory, but a special “swap partition” used as extra memory. It’s like an overflow space on disk when RAM fills up. The installer creates a swap volume during setup (or you can configure one later).
* **EFI System Partition (ESP)** – a special FAT32 partition used on UEFI systems to store bootloaders. It’s not a Linux filesystem per se, but the OS mounts it (usually at `/boot/efi`) so that the firmware can load the Linux kernel from it.

You may also see **/etc** (config files), **/var** (variable data), **/tmp** (temporary files) – those are part of the standard **Filesystem Hierarchy Standard (FHS)**.  The key takeaway is that the installer sets up at least `/` (root), and often `/home` on a separate volume (if using LVM or encryption), and `/boot` or ESP for boot files.

### Disk Encryption with LUKS

**LUKS** stands for *Linux Unified Key Setup*. It’s the standard way to encrypt a disk or partition on Linux.  When you use LUKS (as in “LVM on LUKS” mode), the script will ask you for a passphrase. This passphrase unlocks a key that encrypts the disk.

All data on that partition is scrambled without the key.  When you boot the system, *initramfs* (the temporary kernel environment) will prompt for your LUKS passphrase, unlock the partition, and then mount your real `/` from inside it.

Wikipedia explains that *“LUKS is used to encrypt a block device. There is an unencrypted header at the beginning of an encrypted volume which allows up to 8 (LUKS1) or 32 (LUKS2) encryption keys”*. The main user benefit is that you can have multiple passphrases (keys) for the same disk. If you lose the header, you lose everything, so backup your header if needed.

In simpler terms, imagine your hard disk contents are like a locked treasure chest. LUKS locks the chest with a key (your passphrase). You open the chest at boot, then Linux can read/write inside. Without unlocking, all contents look like gibberish.

The script’s LVM-on-LUKS mode sets up **LVM inside a LUKS container**. It creates one big encrypted container, and then creates LVM volumes inside it (for root, home, swap, etc.). This way, one passphrase unlocks all volumes. The Wikipedia article notes:

> *“When LVM is used on an unlocked LUKS container, all underlying partitions (which are LVM logical volumes) can be encrypted with a single key. This is akin to splitting a LUKS container into multiple partitions. The LVM structure is not visible until the disk is decrypted.”*.

So the encryption is transparent to the rest of the system (once unlocked).

On the other hand, in plain LVM mode (no encryption), the volumes are not encrypted at all.

### The Role of initramfs

During boot of an encrypted system, the Linux kernel uses **initramfs** (initial RAM filesystem). Initramfs is a tiny temporary root file system (packed into a kernel image or separate file) that runs before the real `/` is mounted. It contains tools like the kernel modules for LUKS and LVM, and scripts that ask you for the passphrase. Once you enter the passphrase, initramfs unlocks the root partition and hands control to the real system.

The README hints at this: using GRUB with LUKS “writes the decryption key into the initrd so you’re only asked once”.  (Initrd is an older term for initramfs.) In plain words, the system boots into a tiny environment where it asks for your LUKS password, then continues booting.

You don’t need to know all details, just that encryption requires a password at boot time, and initramfs handles that.

### Symbolic Links and Dotfiles

A **symbolic link (symlink)** is a special type of file that points to another file or directory. It’s like a shortcut. For example, if the script creates a symlink from `/home/alice/.bashrc` to `/etc/skel/.bashrc`, then whenever the shell tries to read `~/.bashrc` it’ll actually read the target file.

Formally, *“a symbolic link … is a file whose purpose is to point to a file or directory (called the ‘target’) by specifying a path thereto.”*.  The `ln -s` command makes symlinks.

The installer’s postscript uses symlinks to set up **dotfiles**: configuration files (like `.bashrc`, `.vimrc`, `.profile`, etc.) that live in the user’s home. The repo’s `dotfiles/` contains standard configs, and the script will link them into your home.  That way, your user account automatically uses those settings (aliases, editor prefs, etc.) from day one.

### sudo, systemd services, apt, and Bootloaders

* **sudo** – a tool that lets normal users run commands as root.  The scripts often use `sudo` (or are run as root). Enabling the `sudo` package is one of the base tasks.  Basically, you become a sudoer so you can do admin tasks without logging in as root.
* **systemd and services** – systemd is the init system in Debian, managing services.  When packages are installed, their systemd services may be enabled by default. The installer might run things like `systemctl enable NetworkManager` if not already enabled. Most configuration is automatic; you just should know that “enabling a service” means it will start on boot.
* **apt (Debian package manager)** – the script uses `apt-get install` or `apt-get purge` to add/remove software. Each menu option typically translates to `apt install ...` for the given list of packages. For example, selecting *“Install virtualbox (host)”* will run something like `apt install virtualbox virtualbox-dkms`.
* **GRUB vs systemd-boot** – these are two different bootloaders. **GRUB** is a powerful, legacy bootloader that supports BIOS and UEFI. **systemd-boot** (formerly gummiboot) is a simpler UEFI-only boot manager included with systemd.  If you chose GRUB in the installer, it will install GRUB to your disk; if you chose systemd-boot (option 2 in the first menu), it will set up systemd-boot on the EFI partition.  The script’s menu prints will say “Boot loader: \[GRUB / systemd-boot]” so you know which one is in use. Systemd-boot has straightforward text configs and boots the kernel directly (kernel must have an EFI stub).  The ArchWiki notes it is “easy-to-configure” for UEFI systems.

## 4. Walkthrough of the Repository Files

Let’s look at how the project’s files are organized. You can inspect these directories in the GitHub repo or your local clone:

* **dotfiles/** – This folder contains user config files (dotfiles) such as `.bashrc`, `.vimrc`, `.xinitrc`, etc. After package installation, the scripts will create symlinks from your home directory to these files. For example, you might see `dotfiles/.bashrc`. The script does something like `ln -s ~/debian/dotfiles/.bashrc /home/$USER/.bashrc`.

* **functions/** – A library of shell functions. These are reusable snippets (like text-coloring functions, menu handling, or formatted echo). For example, there may be a `functions/mount.sh` to mount partitions. These functions are sourced by the main scripts (`install` and `postinstall`) so that the code is modular. You generally don’t need to edit them unless adding new behaviors.

* **packages/** – This directory holds text files listing package names for different categories. For instance:

  * `base.list` – base system packages.
  * `utils.list` – command-line tools.
  * `gnome.list`, `kde.list` – desktop environment packages.
  * `apps.list`, `apps-kde.list` – additional applications (GTK/KDE).
  * And many more (3d-accel.list, android.list, etc.) as seen in the menu .

  The post-install script reads these files when installing software. You can **edit these `.list` files** to add or remove software. For example, to add Firefox to all installs, you could add `firefox-esr` to `apps.list`.

* **configs/** – Contains configuration file templates that may be copied into the new system. For example, a custom `grub` config or `systemd` service overrides. The script can apply them using `cp`. If you customize or add configuration, you can put files here and update the script to use them.

* **dconf/** – Holds the exported settings for GNOME, KDE, etc. These are usually created by dumping `dconf` (e.g. `dconf dump /org/gnome/ > cinnamon-settings.dconf`). The postinstall uses these with `dconf load` to restore your chosen settings. You can modify these dumps if you want different default themes or terminal profiles.

* **hooks/** – This might contain post-install scripts or hooks (for example, apt hooks) that run at certain points (like after package update). The README hints at “hooks” in the context of *“general utility scripts”*, but you can check if any exist. In Arch installation scripts (the repo is also mirrored on Arch scripts), hooks often copy files into `/etc` after mkinitcpio, etc. Here, they might include things like `xinit.sh` or autologin setup.

* **utils/** – Miscellaneous helper scripts or tools. For example, there might be a `git-clone` or `download` script. These are used internally by the main scripts for tasks like downloading and compiling themes (if not in standard repos), or cloning repos (e.g. for Kitty terminal). You usually don’t run these directly; they support the other scripts.

The `install` script is the main entrypoint for partitioning and base install. It sources functions from `functions/` and uses the package lists in `packages/`. The `postinstall` script does the rest: reading package lists, linking dotfiles, loading dconf, etc.

**Symbolic Links for Dotfiles:** You’ll see lines in the scripts or functions where it does something like: `ln -s ~/debian/dotfiles/.$file /home/$USER/.$file`.  This creates a symlink named `.$file` in the user’s home that points back to the `dotfiles/` version.  Because the target (in `dotfiles/`) is read-only code from the repo, using symlinks is safer than copying – it keeps the source of truth in one place.

## 5. How to Modify and Extend

Since the user wants to safely customize, here are tips:

* **Editing Package Lists:** Each category in the menu corresponds to a `.list` file in `packages/`. To add software, open the appropriate file (e.g. `apps.list`) and add package names (one per line). For example, to install `audacity` every time, add `audacity` to `apps.list`. After modifying, test the change by running the script's relevant menu option (or “All”) and watching that `apt` picks up the new package.

* **Testing in a VM:** Before trying changes on real hardware, use a virtual machine (VirtualBox, QEMU, etc.) with Debian live CD or any Linux live CD. Boot it, clone this repo into the VM, and run `sudo ./install`. That way you can iterate fast without risking your main system. For example, you can experiment with partition settings, or adding bad packages, and simply delete and recreate the VM if something breaks.

* **Creating New Menu Entries:** If you want a new custom action (say, install a set of fonts), you can modify `postinstall`: find the section matching where it should go (like “Themes” or “Miscellaneous”), then copy an existing menu case and adjust. For instance, add a new `*)` case in the `case` statement that runs `apt install <package>` or other commands. Also add a description line so your new menu item shows up. Remember to add any new packages to a `.list` file or directly apt-get them.

* **Scripts and Safety:** Always run scripts as root (`sudo` or booted as root). Be very careful with partitioning commands (`sgdisk`, `mkfs`, `cryptsetup`) – they will erase data! Ensure you target the correct device. That’s why testing in a VM or on a blank disk is crucial first. Use echo statements or dry-runs (`--no-act` flags if available) to verify before letting them run. Back up any important data on a disk before using the installer.

* **Re-running Post-install:** If you make mistakes, you can re-run portions of `postinstall` without reinstalling the OS. For example, to reinstall packages from `apps.list`, just run that menu option again. Or to fix dotfiles, you can manually `ln -sf` them. The scripts are idempotent enough that re-running isn’t too harmful (often it will just tell you packages are already installed).

* **Adding Utility Scripts:** If you write new helper scripts (in `functions/` or `utils/`), you can call them from the main scripts. For example, add `source ./functions/myfuncs.sh` at the top of `postinstall`, then `my_custom_action` in a menu handler.

* **Documentation and Help:** The README itself (and this tutorial) explain what each part does. You can also read comments inside the scripts (open them in a text editor) – the author often commented what each section does. There’s also help built into the scripts: usually pressing `h` or `?` on a menu might list options.

In summary: start by reading the **README** and code, test small changes in a VM, and proceed step-by-step. Always use source control (git) or make copies of config files before editing, so you can revert if something goes wrong. With patience, you can tailor the installer to add any software or customization you like.

### Example Command Summary

Here are a few example commands and concepts covered:

* **Partitioning example (bash):**

  ```bash
  # Make GPT partition table on /dev/sda
  sgdisk --clear --mbrtogpt /dev/sda
  # Create 512MB EFI partition
  sgdisk --new=1:0:+512M --typecode=1:ef00 /dev/sda
  # Create 1MiB BIOS partition (if needed)
  sgdisk --new=2:0:+1M   --typecode=2:ef02 /dev/sda
  # The rest will be used for Linux system (LUKS/LVM swap/root)
  sgdisk --new=3:0:0     --typecode=3:8300 /dev/sda
  ```
* **Formatting:**

  ```bash
  mkfs.fat -F32 /dev/sda1      # EFI
  cryptsetup luksFormat /dev/sda3    # ask passphrase (only if encrypting)
  cryptsetup open /dev/sda3 cryptroot
  pvcreate /dev/mapper/cryptroot
  vgcreate vg0 /dev/mapper/cryptroot
  lvcreate -L 16G -n swap vg0   # swap
  lvcreate -l 100%FREE -n root vg0  # root (rest)
  mkfs.ext4 /dev/vg0/root
  mkswap /dev/vg0/swap
  ```
* **Debootstrap:**

  ```bash
  mount /dev/vg0/root /mnt/debian
  mkdir /mnt/debian/boot/efi
  mount /dev/sda1 /mnt/debian/boot/efi
  debootstrap --arch amd64 bullseye /mnt/debian http://deb.debian.org/debian/
  ```
* **Chroot:**

  ```bash
  for f in /dev /dev/pts /proc /sys; do mount --bind $f /mnt/debian$f; done
  chroot /mnt/debian /bin/bash
  # now in new system: set timezone, locales, root password, etc.
  ```
* **Postinstall apt:**

  ```bash
  xargs -a packages/utils.list sudo apt-get install -y
  xargs -a packages/apps.list  sudo apt-get install -y
  ```
* **Linking a dotfile:**

  ```bash
  ln -s /usr/local/share/debian/dotfiles/.bashrc /home/alice/.bashrc
  ```

Remember that the actual script handles details like checking for errors and cleaning up; these examples are for illustration.

With this guide and careful experimentation, you should be able to understand and even tweak every part of the sudorook/debian installer. Always read prompts carefully, and back up data. Enjoy customizing your Debian install!

**Sources:** The explanations above combine the project’s README and scripts with standard Linux documentation on GPT partitions, LUKS encryption, debootstrap, `chroot`, the Linux filesystem hierarchy, symbolic links, dconf, and systemd-boot. All commands shown are examples consistent with how these tools work.

## Setting Up Debian

#### Hyprland DE Specific
- [ ] The following hyprland plugins need to installed after logging into hyprland:
   - [ ] hyprexpo from hyprland-plugins
   - [ ] hyprspace
   - [ ] hyprnome
   - [ ] hy3
- [ ] Add prettify symbols hook to org-mode in doom emacs
- [ ] Make nwg-hello work with Hyprland
- [ ] Configuration files for hypridle and hyprlock need to updated from rxhyn nix files or hyprdots
- [ ] Trackpad needs to be disabled while typing in hyprland
- [ ] Install fonts in /usr/share/fonts/nerd instead of /usr/share/local/fonts/nerd
- [ ] Apply themes to flatpak apps individually. If theming does not work, try adding install the same app from an arch linux distrobox
- [ ] Asusctltray needs to be fix
- [ ] Eventually, all configurations are to be ported from their respective nixos configurations 
   - [ ] Hyprland configuration has already been ported 
- [ ] To port hm configurations, use the following steps:
   - [ ] install hm module one by one
   - [ ] when one module is setup, copy its contents to your dotfiles
   - [ ] then remove that module by commenting over it.
- [ ] Do a btrfs snapper snapshot to test the following:
   - [ ] the snapshot can be booted into
   - [ ] use arch wiki to make the snapshots read/write
- [ ] Setup secureboot using debian wiki
- [ ] Add gpg-key support to git by following the git hm module
- [ ] Add git-lfs support to git as well.
- [ ] Configure zellij
- [ ] Setup ags HyprPanel.
- [ ] Add nwg-menu to waybar
- [ ] Consider adding qt5ct,qt6ct configs from hyprdots
- [ ] Also consider adding hypridle from hyprdots
- [ ] Move hyprterm from alacritty to foot
- [ ] Add checkupdates feature to waybar
- [ ] Setup custom nwg-hello config
- [ ] Use https://gitlab.com/marcaux/g14-2021-s3-dsdt to fix suspend-to-RAM in asus G14
- [ ] Setup fastfetch from garuda-hyprdots
- [ ] Write tmux config 
- [ ] Consider porting hypridle, hyprlock and waybar config from raexera/yuki
- [ ] Use catppuccin gtk github instructions to change flatpak app theming
- [ ] Add catppuccin gtk theme as gtk3,gtk4 theme; papirus-dark and catppuccin folders as icons and bibata cursors as cursor theme
- [ ] Try adding flatpak themes again now that flatpaks are installed systemd-wide
- [ ] Fix scaling gtk4 apps in hyprland using the following command:
- [ ] 
- [ ] p


#### Doom Emacs
- [ ] Finish doom emacs config for latex and python & jupyter

#### Steps for optimizing pkgs
Optimize debian by setting up debian pkgs and installing them in the following workflow
glibc --> binutils --> gcc --> go --> python --> glibc --> binutils --> gcc --> llvm --> clang 

##### Rebuild and Optimize these packages 
1. firefox
2. deluge


#### Build custom kernel for debian
- [ ] First download the vanilla kernel
- [ ] Apply debian patches
- [ ] Finally apply cachyos-base-all patch
- [ ] Then build the custom kernel as debian pkgs and install them accordingly

#### Setup debian to build packages
- [ ] Setup a personal repository
- [ ] Install the required tools
  

#### Build these as debian packages for the latest release
- [ ] emacs-lsp-booster
- [ ] texlab
- [ ] nwg-drawer
- [ ] nwg-menu 
- [ ] nwg-look
- [ ] nwg-hello
- [ ] rofi-wayland9
- [ ] grub-btrfs
- [ ] neovim
- [ ] apparmor
- [ ] lxc & lxd
- [ ] flatpak apps
- [ ] cargo apps
- [ ] bun apps
- [ ] sagemath
- [ ] nvidia-drivers
- [ ] obsidian
- [ ] hyprland and related packages 


#### Different debian tasks
- [ ] Setup snapper and make sure snapshots are read/write capable 
- [ ] Create an initial snapshot that I can recover from. Do this every week @weekly
- [ ] Create a manual snapshot and tpm2 support to gentoo
- [ ] Create a manual snapshot and all hardening stuff from gentoo handbook
- [ ] Install both distrobox and distrobox gui
- [ ] Create 3 distrobox containers
   - [ ] gentoo
   - [ ] arch
   - [ ] clear linux
- [ ] Setup distrobox with nvidia-container-toolkit
- [ ] Figure out how doom emacs might access this distrobox, possibly direnv 
- [ ] Install and Setup atuin, chezmoi at a later date
- [ ] Setup nix, home-manager and flake in debian
- [ ] Setup debian iso using https://github.com/rhino-linux/os
- [ ] After all these steps, create a btrfs snapshot and move from update-initramfs to dracut. Make sure to copy dracut.conf to /etc
- [ ] Need to add a transition mechanism from centos to debian like ubuntu
- [ ] Add kdump-tools
#### Hardening Debian
- [ ] Add kicksecure repo to debian
- [ ] Add kali linux repo to debian and install specific packages in debian-pinning
- [ ] Use parrotos conversion script on debian and install it's kernel images
- [ ] For each of the above steps 1st create custom snapshot before proceeding 
- [ ] Use parrotos/kali's network policy and disable network specific systemd services. Setup systemd hooks like parrotos and kali
- [ ] Harden and secure docker
- [ ] Create a manual snapshot and add secureboot support to debian
- [ ] Harden systemd services using shh 
- [ ] Add extra apparmor profiles from https://github.com/roddhjav/apparmor.d
- [ ] Fullfill Common Criteria Certification requirements
- [ ] Fullfill CIS requirements
- [ ] Setup fips in debian
- [ ] Hardening profiles are taken from parrotos_files and whonix
- [ ] Harden ssh 
- [ ] Consider if rudder free version can achieve what I want at free of cost by programming for it
- [ ] Setup puppet and ansible as an alternative to rudder
- [ ] Setup aide for file integrity checks

#### Neovim
- [ ] Setup neovim using kickstart.nvim
- [ ] Fix the issue where pasting in visual mode copies the previous text
- [ ] Add neotree plugin to astrovim

### Ubuntu
- [ ] Port features from ubuntu to debian
Look at the release page for both ubuntu 24.04 and 24.10\
- https://ubuntu.com/blog/ubuntu-desktop-24-04-noble-numbat-deep-dive
- https://ubuntu.com/blog/whats-new-in-security-for-ubuntu-24-04-lts
- https://ubuntu.com/blog/canonical-releases-ubuntu-24-10-oracular-oriole
- [ ] Add multiple gcc versions
- [ ] Add multiple python versions
- [ ] Add multiple rustc versions
- [ ] Add kcoredumps support
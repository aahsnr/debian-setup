Here are the updated instructions for creating a custom Debian Sid ISO, incorporating your latest changes. This version configures specific partition sizes and sets up disk encryption that requires you to enter a password during the installation process.

### **Key Changes in This Workflow**

1.  **Interactive Encryption Passphrase**: The installation process will now **pause and prompt you to enter an encryption password**. This makes the installation semi-automated but is significantly more secure as the password is never stored on the ISO.
2.  **/boot Partition Size**: The separate, unencrypted `/boot` partition will be set to **512 MiB**.
3.  **/boot/efi Partition Size**: The EFI System Partition (ESP) will be set to **512 MiB**.
4.  **Swap Volume Size**: The swap logical volume, which resides inside the encrypted container, will be set to a fixed size of **16 GiB**.

---

### **Step-by-Step Guide with Interactive Encryption**

#### **1. Setting Up the Build Environment**

This step remains unchanged. You need a Debian-based system to build the ISO.

```bash
sudo apt update
sudo apt install live-build live-manual live-config whois
```

Create your project directory:
```bash
mkdir my-debian-live
cd my-debian-live
```

#### **2. Initial `live-build` Configuration**

This step is also unchanged. It creates the base configuration for a Debian Sid ISO.

```bash
lb config \
    --distribution unstable \
    --architectures amd64 \
    --archive-areas "main contrib non-free non-free-firmware" \
    --debootstrap-options "--variant=minbase" \
    --bootappend-live "boot=live components"
```

#### **3. Creating the `preseed.cfg` for Interactive Encryption**

This preseed file is updated to reflect the new partition sizes and to remove all automated passphrase handling, which will force the installer to ask for the password.

**File: `config/includes.installer/preseed.cfg`**
```cfg
# --- Partitioning: Encrypted Btrfs on LVM (Interactive Passphrase) ---
# This recipe defines the partition layout. Since no passphrase or keyfile is
# provided, the installer will stop and ask the user to enter one.

d-i partman-auto/disk string /dev/sda
d-i partman-auto/method string crypto
d-i partman-auto/expert_recipe string                         \
      crypto-lvm ::                                           \
              # 512MB EFI System Partition
              512 512 512 fat32                               \
                      $primary{ } $bootable{ }                \
                      method{ efi } format{ }                 \
                      mountpoint{ /boot/efi }                 \
              .                                               \
              # 512MB /boot partition
              512 512 512 ext4                                \
                      $primary{ }                             \
                      method{ format } format{ }              \
                      mountpoint{ /boot }                     \
              .                                               \
              # The rest of the disk for the encrypted container
              12000 1000 -1 linux-crypto                      \
                      $primary{ }                             \
                      method{ crypto }                        \
              .

# --- LVM and Btrfs Configuration (Inside Crypto) ---
# This part now targets the encrypted device.
d-i partman-auto-lvm/new_vg_name string main-vg
d-i partman-auto-lvm/guided_size string max

d-i partman-auto/choose_recipe select lvm
d-i partman-auto-lvm/select_crypto select Guided

d-i partman-auto/expert_recipe string \
      lvm-on-crypto :: \
        # 16GiB (16384MiB) swap volume
        16384 1000 16384 linux-swap \
          $lvmok{ } lv_name{ swap } method{ swap } format{ } \
        . \
        # Btrfs root volume using the remaining space
        10000 500 -1 btrfs \
          $lvmok{ } lv_name{ root } method{ format } format{ } \
          use_filesystem{ } filesystem{ btrfs } mountpoint{ / } \
        .

# --- Btrfs Subvolume Configuration ---
d-i partman-btrfs/subvolumes string \
    / @ \
    /home @home \
    /root @root \
    /srv @srv \
    /var @var \
    /opt @opt

# --- Confirmation ---
d-i partman-partitioning/confirm_write_new_label boolean true
d-i partman/choose_partition select finish
d-i partman/confirm boolean true
d-i partman/confirm_nooverwrite boolean true

# --- User, Locale, and Post-Install Script (Unchanged) ---
# NOTE: Passwords for users are still pre-seeded for automation after the disk encryption step.
d-i passwd/root-password-crypted password YOUR_ENCRYPTED_ROOT_PASSWORD
d-i passwd/user-fullname string Your Full Name
d-i passwd/username string youruser
d-i passwd/user-password-crypted password YOUR_ENCRYPTED_USER_PASSWORD
d-i passwd/user-default-groups string sudo
d-i clock-setup/utc boolean true
d-i time/zone string Etc/UTC
d-i debian-installer/locale string en_US.UTF-8
d-i preseed/late_command string \
    in-target /bin/bash -c 'cp /cdrom/install/my_script.sh /home/youruser/my_script.sh && chown youruser:youruser /home/youruser/my_script.sh && chmod +x /home/youruser/my_script.sh'; \
    in-target /usr/bin/sudo -u youruser /home/youruser/my_script.sh
```

**Important Notes on the `preseed.cfg`:**

* **No Passphrase/Keyfile**: All lines related to `partman-crypto/passphrase` and `partman-crypto/keyfile` have been removed. This is what makes the installation interactive.
* **Partition Sizes**: The numbers in the `expert_recipe` are `<min> <priority> <max>` in MiB.
    * `/boot` and `/boot/efi` are fixed at `512 512 512`.
    * Swap is fixed at `16384 1000 16384` (16 GiB).
    * The `-1` in the `linux-crypto` and `btrfs` lines means they should take up all available remaining space.
* **User Passwords**: Remember to generate encrypted hashes for the `root` and `youruser` accounts using `mkpasswd -m sha-512 "your_password"` and place them in the file.

#### **4. Updating the Package List**

This step is unchanged but remains essential. The `cryptsetup` package is required for the installed system to handle the encrypted volume.

**File: `config/package-lists/my.list.chroot`**
```
# Core Utilities
cryptsetup
btrfs-progs
lvm2
sudo

# Your other desired packages (e.g., Desktop Environment)
task-kde-desktop
konsole
git
neovim
```

#### **5. Custom Post-Installation User Script**

This section is **unchanged**. You still place your personal setup script in `config/includes.installer/`. The `late_command` in the preseed file will copy it to the home user's directory and execute it with sudo privileges after the main installation is complete.

**File: `config/includes.installer/my_script.sh`**
```bash
#!/bin/bash
# This is your personal script for post-install configuration.
# It runs as your user with sudo rights.

echo "Running custom user setup script..."

# Example: Configure git
git config --global user.name "Your Full Name"
git config --global user.email "youruser@example.com"

# Example: Create a development directory
mkdir -p $HOME/Development/projects

echo "Post-install script finished at $(date)" >> $HOME/post-install-log.txt
```

#### **6. Adding Custom User Dotfiles**

This step is **unchanged**. It remains the best way to pre-configure the shell environment for your user and for root.

* Place user dotfiles in `config/includes.chroot/etc/skel/`
* Place root's dotfiles in `config/includes.chroot/root/`

#### **7. Building the ISO**

This final step is unchanged. From your project's root directory (`my-debian-live`), run the build command:

```bash
sudo lb build
```

The resulting `live-image-amd64.hybrid.iso` is now ready. When you boot from it, the installation will proceed automatically until it reaches the disk setup stage, where it will pause and prompt you to enter and confirm a password for disk encryption. After you provide the password, the rest of the installation will continue automatically.

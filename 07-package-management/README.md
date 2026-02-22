# 📦 Chapter 07: Package Management

<p align="center">
  <img src="https://img.shields.io/badge/Level-Beginner-green?style=for-the-badge" alt="Beginner">
  <img src="https://img.shields.io/badge/Chapter-07%20of%2034-blue?style=for-the-badge" alt="Chapter 07">
</p>

---

## 📑 Table of Contents

- [What is a Package Manager?](#what-is-a-package-manager)
- [APT (Debian/Ubuntu)](#apt-debianubuntu)
- [DNF (Fedora/RHEL)](#dnf-fedorarhel)
- [Pacman (Arch Linux)](#pacman-arch-linux)
- [Zypper (openSUSE)](#zypper-opensuse)
- [Universal Package Managers](#universal-package-managers)
- [Compiling from Source](#compiling-from-source)
- [Package Manager Comparison](#package-manager-comparison)
- [Practice Exercises](#-practice-exercises)

---

## What is a Package Manager?

A **package manager** automates installing, updating, configuring, and removing software.

> 🏠 **Analogy**: A package manager is like an app store for your terminal. It handles downloads, dependencies, updates, and cleanup — so you don't have to.

### Key Concepts

| Term | Description |
|------|-------------|
| **Package** | A bundle of software files + metadata + install scripts |
| **Repository (repo)** | A server hosting packages |
| **Dependency** | Other software a package needs to work |
| **Cache** | Locally stored package files |

---

## APT (Debian/Ubuntu)

**APT** (Advanced Package Tool) is used by Debian, Ubuntu, Linux Mint, Pop!_OS, and derivatives.

### Updating & Upgrading

```bash
sudo apt update                    # Refresh package lists from repos
sudo apt upgrade                   # Upgrade all installed packages
sudo apt full-upgrade              # Upgrade + remove obsolete packages
sudo apt dist-upgrade              # Major upgrades (kernel, etc.)
```

> 💡 **Always run `apt update` before `apt install`** to ensure you get the latest package versions.

### Installing Packages

```bash
sudo apt install vim               # Install a single package
sudo apt install vim git curl      # Install multiple packages
sudo apt install -y nginx          # Auto-confirm with -y
sudo apt install ./package.deb     # Install a local .deb file
```

### Removing Packages

```bash
sudo apt remove vim                # Remove package (keep config files)
sudo apt purge vim                 # Remove package + config files
sudo apt autoremove                # Remove unused dependencies
sudo apt clean                     # Clear downloaded package cache
sudo apt autoclean                 # Remove old package cache only
```

### Searching & Information

```bash
apt search "web server"            # Search for packages
apt show nginx                     # Detailed package info
apt list --installed               # List all installed packages
apt list --upgradable              # List packages with updates
dpkg -l                            # List installed packages (low-level)
dpkg -L nginx                      # List files installed by a package
dpkg -S /usr/bin/vim               # Which package owns this file?
apt-cache depends nginx            # Show package dependencies
apt-cache rdepends nginx           # Reverse deps (who depends on nginx)
```

### Managing Repositories

```bash
# List current repos
cat /etc/apt/sources.list
ls /etc/apt/sources.list.d/

# Add a PPA (Personal Package Archive) — Ubuntu only
sudo add-apt-repository ppa:user/repo
sudo apt update

# Remove a PPA
sudo add-apt-repository --remove ppa:user/repo

# Add a third-party repo manually
echo "deb [signed-by=/usr/share/keyrings/example.gpg] https://repo.example.com stable main" | \
  sudo tee /etc/apt/sources.list.d/example.list
```

### Holding Packages (Prevent Updates)

```bash
sudo apt-mark hold nginx           # Don't update this package
sudo apt-mark unhold nginx         # Allow updates again
apt-mark showhold                  # List held packages
```

---

## DNF (Fedora/RHEL)

**DNF** (Dandified YUM) is used by Fedora, RHEL 9+, CentOS Stream, AlmaLinux, Rocky Linux.

```bash
# Update
sudo dnf check-update              # Check for available updates
sudo dnf upgrade                   # Upgrade all packages
sudo dnf upgrade --refresh         # Refresh cache and upgrade

# Install
sudo dnf install vim               # Install a package
sudo dnf install ./package.rpm     # Install a local RPM
sudo dnf install @"Development Tools"  # Install a group of packages
sudo dnf groupinstall "Development Tools"  # Same as above

# Remove
sudo dnf remove vim                # Remove a package
sudo dnf autoremove                # Remove unused dependencies
sudo dnf clean all                 # Clear all caches

# Search & Info
dnf search "web server"            # Search packages
dnf info nginx                     # Package details
dnf list installed                 # List installed packages
dnf list available                 # List all available packages
dnf provides /usr/bin/gcc          # Which package provides this file?
dnf repolist                       # List enabled repos
dnf history                        # Transaction history
dnf history undo 15                # Undo transaction #15

# Repos
sudo dnf config-manager --add-repo https://example.com/repo.repo
sudo dnf config-manager --set-enabled repo-name
```

---

## Pacman (Arch Linux)

**Pacman** is used by Arch Linux, Manjaro, EndeavourOS, and Arch-based distros.

```bash
# Sync database & upgrade
sudo pacman -Sy                    # Sync package database
sudo pacman -Syu                   # Sync + full system upgrade (ALWAYS do this)

# Install
sudo pacman -S vim                 # Install a package
sudo pacman -S --needed vim git    # Install only if not already installed
sudo pacman -U /path/package.pkg.tar.zst  # Install local package

# Remove
sudo pacman -R vim                 # Remove package
sudo pacman -Rs vim                # Remove + unused dependencies
sudo pacman -Rns vim               # Remove + deps + config files (cleanest)

# Search & Info
pacman -Ss "web server"            # Search repos
pacman -Qs vim                     # Search installed packages
pacman -Si nginx                   # Info from repo
pacman -Qi vim                     # Info about installed package
pacman -Ql vim                     # List files installed by package
pacman -Qo /usr/bin/vim            # Which package owns this file?

# Clean cache
sudo pacman -Sc                    # Remove old cached packages
sudo pacman -Scc                   # Remove ALL cached packages
```

### AUR (Arch User Repository)

The AUR is a community-driven repo with thousands of additional packages.

```bash
# Install an AUR helper (yay)
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay && makepkg -si

# Use yay like pacman
yay -S google-chrome               # Install from AUR
yay -Syu                           # System upgrade + AUR
yay -Ss discord                    # Search AUR + repos
```

---

## Zypper (openSUSE)

```bash
sudo zypper refresh                # Refresh repo data
sudo zypper update                 # Update all packages
sudo zypper install vim            # Install
sudo zypper remove vim             # Remove
zypper search vim                  # Search
zypper info vim                    # Package info
sudo zypper dist-upgrade           # Distribution upgrade
```

---

## Universal Package Managers

These work across all distros:

### Snap (Canonical)

```bash
sudo apt install snapd             # Install snap daemon
sudo snap install code --classic   # Install VS Code
snap list                          # List installed snaps
sudo snap remove code              # Remove
sudo snap refresh                  # Update all snaps
snap find "media player"           # Search snap store
```

### Flatpak

```bash
sudo apt install flatpak           # Install flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

flatpak install flathub org.gimp.GIMP  # Install from Flathub
flatpak list                       # List installed
flatpak update                     # Update all
flatpak uninstall org.gimp.GIMP    # Remove
flatpak search gimp                # Search
```

### AppImage

```bash
# No installation needed — just download and run
chmod +x MyApp.AppImage
./MyApp.AppImage

# Optional: move to /usr/local/bin for system-wide access
sudo mv MyApp.AppImage /usr/local/bin/myapp
```

### Comparison

| Feature | Snap | Flatpak | AppImage |
|---------|------|---------|----------|
| **Sandbox** | ✅ Yes | ✅ Yes | ❌ No |
| **Auto-update** | ✅ Yes | ✅ Yes | ❌ Manual |
| **Store** | [snapcraft.io](https://snapcraft.io) | [Flathub](https://flathub.org) | [AppImageHub](https://appimage.github.io) |
| **Bundle size** | Larger | Smaller (shared runtimes) | Self-contained |
| **Distro support** | Ubuntu-centric | All distros | All distros |

---

## Compiling from Source

When a package isn't in any repository, you can compile it from source:

```bash
# 1. Install build dependencies
sudo apt install build-essential cmake git    # Debian/Ubuntu
sudo dnf groupinstall "Development Tools"     # Fedora

# 2. Download source code
git clone https://github.com/example/project.git
cd project

# 3. Configure
./configure --prefix=/usr/local
# or
mkdir build && cd build
cmake ..

# 4. Build
make -j$(nproc)                    # Compile using all CPU cores

# 5. Install
sudo make install

# 6. (Optional) Uninstall
sudo make uninstall
```

> 💡 **Tip**: Use `checkinstall` instead of `make install` to create a proper `.deb` package that your package manager can track:
> ```bash
> sudo apt install checkinstall
> sudo checkinstall     # Creates a .deb and installs it
> ```

---

## Package Manager Comparison

| Action | APT (Debian) | DNF (Fedora) | Pacman (Arch) |
|--------|-------------|-------------|---------------|
| Update repos | `apt update` | `dnf check-update` | `pacman -Sy` |
| Upgrade all | `apt upgrade` | `dnf upgrade` | `pacman -Syu` |
| Install | `apt install pkg` | `dnf install pkg` | `pacman -S pkg` |
| Remove | `apt remove pkg` | `dnf remove pkg` | `pacman -R pkg` |
| Search | `apt search pkg` | `dnf search pkg` | `pacman -Ss pkg` |
| Info | `apt show pkg` | `dnf info pkg` | `pacman -Si pkg` |
| List installed | `apt list --installed` | `dnf list installed` | `pacman -Q` |
| Clean cache | `apt clean` | `dnf clean all` | `pacman -Sc` |
| File owner | `dpkg -S /path` | `dnf provides /path` | `pacman -Qo /path` |

---

## 🏋️ Practice Exercises

1. **Update** your system using the appropriate package manager
2. **Install** `htop`, `tree`, and `neofetch`
3. **Search** for a package containing "text editor"
4. **Info**: Check the details of the `curl` package
5. **Find** which package owns `/usr/bin/python3`
6. **Remove** `neofetch` and its unused dependencies
7. **Install** an AppImage: Download one from [AppImageHub](https://appimage.github.io)
8. **Compile from source**: Try building a small tool like `bat` or `fd`

---

<p align="center">
  <a href="../06-users-groups-permissions/README.md">← Previous: Users, Groups & Permissions</a> · <a href="../README.md">🏠 Home</a> · <a href="../08-text-editors/README.md">Next: Text Editors →</a>
</p>

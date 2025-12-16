# RoboRacer NUC Setup Guide (Ubuntu)

Minimal step-by-step guide to set up an Intel NUC for **RoboRacer** on **Ubuntu**.

---

## 1. Ubuntu Installation

- Install **Ubuntu 24.04** via bootable USB
- During setup:
  - **Hostname:** `racecar_version`
  - **Username:** `larace`

Reboot after installation.

---

## 2. Network Setup

### 2.1 Wi-Fi Connections

Configure **two Wi-Fi profiles**:

1. **Race Router (ROS Network)**
   - Enable **automatic connection**
   - Use **only** for autonomous driving / ROS network

2. **Internet Wi-Fi**
   - Disable automatic connection
   - Connect **manually only** when updates or downloads are needed

---

### 2.2 Hokuyo LiDAR Ethernet (UST-20LX)

1. Connect the Hokuyo LiDAR via Ethernet and power it on.
2. Open **Network Settings → Wired Connection → Gear Icon**
3. Go to **IPv4**
4. Set **Method** to `Manual`
5. Configure:
   - Address: `192.168.0.10`
   - Netmask: `255.255.255.0`
   - Gateway: `192.168.0.1`
6. Rename the connection to: ```Hokuyo UST-20LX```
7. Test connection: ```ping 192.168.0.10```

---

## 3. System Update & Base Packages

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

Install required packages:

```bash
sudo apt install -y \
  nano vim python3-pip git firefox \
  openssh-client openssh-server \
  curl htop bluetooth \
  jstest-gtk evtest wget rsync \
  joystick net-tools apt-utils \
  x11-apps
```

Cleanup:

```bash
sudo apt autoremove -y
sudo apt update
```

---

## 4. Udev Rules

### 4.1 VESC (Electronic Speed Controller)

```bash
sudo -i
nano /etc/udev/rules.d/99-vesc.rules
```

Add:

```
KERNEL=="ttyACM[0-9]*", ACTION=="add", ATTRS{idVendor}=="0483", ATTRS{idProduct}=="5740", MODE="0666", GROUP="dialout", SYMLINK+="sensors/vesc"
```

---

### 4.2 Controller (DualShock / Logitech F710)

```bash
sudo -i
nano /etc/udev/rules.d/99-ds4.rules
```

Add:

```
KERNEL=="js[0-9]*", ACTION=="add", ATTRS{idVendor}=="1d6b", ATTRS{idProduct}=="09cc", SYMLINK+="input/joypad-f710"
```

---

### 4.3 Reload Rules

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```

**Tip – inspect device attributes**

```bash
sudo udevadm info --name=<KERNEL> --attribute-walk
```

---

## 5. Docker Installation

### 5.1 Add Docker Repository

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

### 5.2 Install Docker

```bash
sudo apt-get install -y \
  docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```

### 5.3 Test Docker

```bash
sudo docker run hello-world
```

---

### 5.4 Docker User Permissions

```bash
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
```

---

## 6. Race Stack Installation

Reboot for a clean state:

```bash
sudo reboot
```

Clone repository:

```bash
git clone https://gitlab.igg.uni-bonn.de/f1_tenth/race_stack
cd race_stack
```

Install stack:

```bash
chmod +x src/core/system/install_stack.sh
bash src/core/system/install_stack.sh
source ~/.bashrc
```


# Project Zomboid Dedicated Server on Oracle Cloud (ARM64)

This guide details the installation and configuration of a **Project Zomboid Dedicated Server** on an **Oracle Ampere ARM64 Linux Server** (VM.Standard.A1.Flex). The setup includes **FEX-Emu** to allow running x86 applications on the ARM64 architecture.

## **Server Specifications**
- **Machine:** Oracle Ampere VM.Standard.A1.Flex
- **Shape:** 4 cores, 24GB RAM, standard boot volume
- **OS:** Ubuntu 22.04

## **Step 1: Initial Setup and Firewall Configuration**

1. **Connect via SSH** using the `.ppk` file generated with PuTTYgen.
2. Run the following commands to configure firewall rules and update the system:

```bash
sudo su
iptables -I INPUT -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 16261 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 16261 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 16262 -j ACCEPT
sudo iptables -A INPUT -p udp --dport 16262 -j ACCEPT
sudo iptables-save > /etc/iptables/rules.v4
sudo systemctl restart iptables
sudo ufw disable
apt update && apt upgrade
reboot
```

## **Step 2: Install Dependencies**

```bash
sudo su
apt-get install git cmake ninja-build pkg-config ccache clang llvm lld binfmt-support libsdl2-dev libepoxy-dev libssl-dev python-setuptools g++-x86-64-linux-gnu nasm python3-clang libstdc++-10-dev-i386-cross libstdc++-10-dev-amd64-cross libstdc++-10-dev-arm64-cross squashfs-tools squashfuse libc-bin expect curl sudo fuse wget
```

## **Step 3: Install FEX-Emu**

```bash
useradd -m -s /bin/bash fex
usermod -aG sudo fex
echo "fex ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers.d/fex
exit
```

```bash
sudo su - fex
git clone --recurse-submodules https://github.com/FEX-Emu/FEX.git
cd FEX && mkdir Build && cd Build
```

```bash
sudo apt install qtbase5-dev qtdeclarative5-dev qttools5-dev-tools libqt5svg5-dev
cd ~/FEX/Build
CC=clang CXX=clang++ cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release -DUSE_LINKER=lld -DENABLE_LTO=True -DBUILD_TESTS=False -DENABLE_ASSERTIONS=False -G Ninja ..
ninja
exit
```

```bash
sudo su
cd /home/fex/FEX/Build
ninja install
ninja binfmt_misc
```

## **Step 4: Configure Steam User and Install SteamCMD**

```bash
useradd -m -s /bin/bash steam
echo 'root:steamcmd' | chpasswd
exit
```

```bash
sudo usermod -aG steam ubuntu
sudo apt install acl
sudo setfacl -b /home/steam
```

```bash
sudo su - steam
mkdir -p ~/.fex-emu/RootFS && cd ~/.fex-emu/RootFS
wget -O Ubuntu_22_04.tar.gz https://www.dropbox.com/scl/fi/16mhn3jrwvzapdw50gt20/Ubuntu_22_04.tar.gz?rlkey=4m256iahwtcijkpzcv8abn7nf
tar xzf Ubuntu_22_04.tar.gz && rm Ubuntu_22_04.tar.gz
cd ~/.fex-emu
echo '{"Config":{"RootFS":"Ubuntu_22_04"}}' > ./Config.json
```

```bash
mkdir ~/Steam && cd ~/Steam
curl -sqL "https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz" | tar zxvf -
```

```bash
FEXBash ./steamcmd.sh
force_install_dir /home/steam/pz
login anonymous
app_update 380870 validate
quit
```

## **Step 5: Start the Project Zomboid Server**

```bash
cd ~/pz
FEXBash "./start-server.sh -servername Panitas.V3"
```

## **Step 6: Importing an Existing Server to Oracle Cloud**

If you have an existing server and want to migrate it to Oracle Cloud, follow these steps:

1. **Inside the VPS console**, start a new server with the desired name:
   ```bash
   ./start-server.sh -servername NameOfYourServer.ini
   ```
   - This creates the necessary files and directories.
   - Join the server once to verify functionality.

2. **Close the server and edit the `.ini` file** to add mod IDs (manually verified beforehand).
3. **Restart the server**, join with a player to ensure it works, then shut it down again.
4. **On your local server, compress the following directories:**
   ```
   C:\Users\UserName\Zomboid\db
   C:\Users\UserName\Zomboid\Saves\Multiplayer\ServerName
   C:\Users\UserName\Zomboid\Server
   ```
5. **Upload and extract these files to the VPS** in their corresponding locations, replacing existing files.
6. **Edit the `.ini` file on the VPS** to match the settings of your local server.
7. **Start the server again** and verify that everything is working properly.

## **Final Notes**
With these steps, your **Project Zomboid Dedicated Server** should be running smoothly on Oracle Cloud with FEX-Emu. If you have any issues or questions, feel free to reach out. **Happy gaming!** 🚀


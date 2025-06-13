# JetConfig: NVIDIA Jetson Orin Nano SSD + Docker Configuration Tool

JetConfig is a command-line interface (CLI) tool designed to streamline the configuration of an SSD on your NVIDIA Jetson Orin Nano Developer Kit. It also optimizes Docker to use the SSD and provides memory optimization options for a lean system.

All steps are according to the official [Jetson Setup Guide](https://www.jetson-ai-lab.com/initial_setup_jon.html).

## Features:
- Formats and mounts an SSD
- Configures `fstab` automatic SSD mounting
- Installes `nvidia-container` (if not already installed) and configures Docker to use the NVIDIA runtime
- Migrates Docker's data directory (`/var/lib/docker`) to the SSD
- **(Optional)** Enables memory optimization by disabling GUI, specific services, and configuring a swap file on the SSD

## Prerequisites:
- An **NVIDIA Jetson Orin Nano (Super) Developer Kit**
- An unformatted or empty **NVMe SSD**
- **JetPack 6.2** is running on the device (See [Initial Setup](https://www.jetson-ai-lab.com/initial_setup_jon.html))
- SSD is *physically installed* to the carrier board

## Installation & Usage:
To use JetConfig, simply clone this repository and run the script

1. **Clone the Repository:**
    Open a terminal on your Jetson and run:
    ```bash
    git clone https://github.com/JoshuaWellbrock/jetconfig.git
    cd jetconfig
    ```

2. **Make the script executable (if not already):**
    The script should already be executable, but in case it isn't just run:
    ```bash
    chmod +x jetconfig
    ```

3. **Run the script**
    To run the script you must specify the SSD device. You can find available devices using `lsblk`.
    The Output should look something like this:
    ```bash
    NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
    loop0          7:0    0    16M  1 loop
    mmcblk1      179:0    0  59.5G  0 disk
    ├─mmcblk1p1  179:1    0    58G  0 part /
    zram0        251:0    0   1.8G  0 disk [SWAP]
    zram1        251:1    0   1.8G  0 disk [SWAP]
    zram2        251:2    0   1.8G  0 disk [SWAP]
    zram3        251:3    0   1.8G  0 disk [SWAP]
    nvme0n1      259:0    0 238.5G  0 disk
    ```

    If you have found your SSD device you can then simply run:
    ```bash
    ./jetconfig -d /dev/nvme0n1
    ```

    :warning: **The `-d` argument points to the entire disk and running this tool will FORMAT and ERASE ALL DATA on that device.**
    You have to confirm this operation after starting the script.

    ***Available Options:***
    - `-d, --device <device>`: **(Required)** Specify the SSD device (e.g., `/dev/nvme0n1`).
    Run `lsblk`to find available devices.
    - `-mp, --mounting-point <path>`: Specify the mounting point (default: `/ssd`).
    - `-m, --memory-optimization`: Enable memory optimization (this will disable GUI, disbale misc services and mount swap, disable ZRAM).
    - `-h, --help`: Show this help message.

## Post-Installation Notes:
- **Docker Group:** After running the script, you may need to log out and log back in for Docker commands to work without `sudo` (due to being added to the `docker` group).
- **GUI:** If you enabled memory optimization, the GUI will be disabled on next reboot. To re-enable the GUI (if needed), run: `sudo systemctl set-default graphical.target`
- `/var/lib/docker.old`: The old Docker data directory at `/var/lib/docker.old`can be safely removed to free up space after you've verified Docker is working correctly with the SSD. To remove the old data directory, simply run: `sudo rm -rf /var/lib/docker.old`

#### Verify that Docker is working with the SSD
To verify that Docker is correctly working with the SSD you can open up two Terminal windows.
1. In the first terminal run `watch -n1 df` to monitor the disk usage.
2. In the second terminal run `docker pull hello-world`
3. Back in the first terminal you should be able to see that the usage on `/ssd` is increasing
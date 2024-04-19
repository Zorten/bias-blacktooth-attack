# Blacktooth Attack Replication

Repository forked from: [BIAS](https://github.com/francozappa/bias)

## Collaborators
[Zergio Ruvalcaba](https://github.com/Zorten) 

[Shruti Jawale]()

[Mariam Golwalla](https://github.com/mgolwalla)

## Videos

[Presentation](https://www.youtube.com/watch?v=H4dhlwKoYmk&ab_channel=MariamGolwalla)

[Live Demo](https://www.youtube.com/watch?v=twfHxUX7Sak&ab_channel=MariamGolwalla)

## Introduction

In this project we attempt to replicate the Blacktooth [1](https://github.com/Zorten/bias-blacktooth-attack/edit/master/README.md#resources) attack. We use a CYW920735Q60EVB-01 Evaluation Board connected to a laptop running a Fedora VM with a modified Linux 4.14.111. We attack a pair of Microsoft Modern Wireless Headset, the information of which can be found in the [Impersonation File](https://github.com/Zorten/bias-blacktooth-attack/blob/master/bias/IF_MSHEADSET.json) in this repository. We make use of the BIAS [2](https://github.com/Zorten/bias-blacktooth-attack/edit/master/README.md#resources) as well as the InternalBlue[3](https://github.com/Zorten/bias-blacktooth-attack/edit/master/README.md#resources) toolsets. The attack was successful in impersonating the victim device and establishing a connection, however the connection is not maintained and we were not able to elevate the Bluetooth profile.  

## Files Accessed/Changed

- ### `bias-CS254/linux-4.14.111/`
  - All of the files within this folder were copied to their respective locations in the Linux 4.14.111 kernel data folders before compiling and installing it.

- ### `bias-CS254/bias/`
  - #### `Makefile`
    - Edited to only call `python3` since the attack script generated is now in Python 3, instead of the original BIAS which creates a Python 2 script.
  
  - #### `bias-template.py`
    - Serves as the basis for building the `bias.py` attack script. 
    - Edited to use Python 3 instead of Python 2.
  
  - #### `generate.py`
    - Modified to call the impersonation file (`IF`) for the device we are impersonating.
    - Edited so that the `bias.py` script it generates is in Python 3.
  
  - #### `IF_MSHEADSET.json`
    - Created with all the information collected for the specific device we were impersonating.
  
  - #### `AF.json`
    - Name changed to match the evaluation board we are using.
    - Modified to match the HCI device number we are using.

## Commands and Attack Flow

- **Initial Setup and Configuration**
  - Connect the development board to the host machine via USB. The host machine we used was running a Fedora VM with a modified Linux 4.14.111 kernel installed. Ensure VM settings are configured to detect the USB connection.
  - Set the device baud rate (115200 for CYW920735Q60EVB-01) and attach it (assuming it is connected to ttyUSB0 interface):
    - `stty -F /dev/ttyUSB0 115200`
    - `btattach -B /dev/ttyUSB0`
  - Set up the device as an HCI device. This command stays running, so a new terminal is needed if run on the foreground:
    - `hciconfig hci0 up`
  - Confirm the device was set up properly, this command lists all HCI devices:
    - `hcitool dev`
  - Enable development board diagnostic mode. File found at 'bias' folder of the repository:
    - `sudo python2 enable_diag.py`
  - Scanning for nearby devices, provides BT MAC address and BT Name for devices found:
    - `hcitool scan`
  - Enter BlueZ command interface:
    - `bluetoothctl`

- **Commands within Bluetoothctl**
  - Power on the device and set agents:
    - `power on`
    - `agent on`
    - `default-agent`
  - Commands to scan/stop scanning for BT devices within bluetoothctl:
    - `scan on`
    - `scan off`
  - Commands to send pair/connect requests to a specific MAC address (used for testing board functionality):
    - `pair <Bluetooth_Device_Address>`
    - `connect <Bluetooth_Device_Address>`

- **Gathering Device Information**
  - Collect Bluetooth device information for the Impersonation File, such as LMP version/subversion, Company ID, and Feature Pages:
    - `hcitool info <Bluetooth_Device_Address>`

- **Using Internalblue for Development**
  - Follow the instructions on the internalblue repository to perform a full development install. Launch internalblue tool command interface as follows:
    - `cd internalblue`               # Run every time
    - `virtualenv -p python3 venv`    # Run only the first time after installing
    - `source venv/bin/activate`      # Run every time to launch virtual environment
    - `internalblue`                  # Run to launch command interface
  - Begin Wireshark monitoring from within internalblue:
    - `wireshark start`

- **Script Generation and Execution**
  - Run within the virtual environment. The following creates the attack script file ('bias.py'). This should be done inside the 'bias' folder of the repository. Makes use of 'generate.py', 'bias-template.py', 'AF.json', and 'IF_MSHEADSET.json' to create the script:
    - `make generate`
  - Run within the virtual environment. Runs attack script to overwrite development board firmware:
    - `make bias`

- **Making the Board Discoverable and Connectable**
  - Run the following inside the bluetoothctl command interface. Makes the board discoverable by other devices and accept connections:
    - `pairable on`
    - `discoverable on`

- **Connection Attempt**
  - Once the attack script has been executed, run the following within the internalblue interface to send a connection request to the victim device. It might take a couple of attempts for the connection to happen.
  - Once successful, it terminates almost immediately, which we suspect happens because the encryption key is not negotiated correctly, so no data transfer is possible.
  - Silent connection without authorization is observable from the victim device as the BT device name of the impersonated device gets overwritten by the name of the board, ‘localhost.localdomain’:
    - `connect <Bluetooth_Device_Address>`


## Resources 

1. Ai, Mingrui, et al. "[Blacktooth: breaking through the defense of bluetooth in silence.](https://dl.acm.org/doi/10.1145/3548606.3560668)" Proceedings of the 2022 ACM SIGSAC Conference on Computer and Communications Security. 2022.
2. [BIAS GitHub Repo](https://github.com/francozappa/bias/tree/master)
3. [InternalBlue GitHub Repo](https://github.com/seemoo-lab/internalblue/tree/master)
4. [Evaluation Board Documentation](https://docs.rs-online.com/0b8d/0900766b816b6e24.pdf)
5. [Cypress Bluetooth Chip Documentation](https://www.mouser.com/datasheet/2/196/CYPR_S_A0005170192_1-3004064.pdf)


## Individual Contributions

### Zergio

For this project, most of my time and effort was spent researching how to use the CYW920735Q60EVB-01 Evaluation Board, as well as the InternalBlue and BIAS toolkits to implement our own attack. The Blacktooth paper describes the attack flow but it is not very specific about the process for information collection for the impersonation, or how they use the tools mentioned. Furthermore, we couldn’t use the same evaluation board as the one in the paper or in the original BIAS toolkit, since it is not available anymore, so there were multiple hurdles in replicating the attack with our new board. I was also the one to set up the GitHub repository for our project, and the one who edited the code to match our needs so that we could implement the attack on our board. Aside from this, I collaborated with my teammates on everything else. Specifically, we all worked on connecting and setting up the evaluation board, testing the functionality of the board, collecting/reverse-engineering the data for the Impersonation File needed for the attack, and then running the attack. 

### Shruti

I mostly worked on the VM part of the project. I created the VMs using KVM on my Linux laptop. This portion of the project took a lot longer than I expected due to issues with downgrading the version of the kernel on the VM and then the size of the disk. I created a Fedora 39 VM with 25G disk space for the project with no issue and then tried to downgrade the kernel version from 6.6.8 to 4.14.111. I was able to install the preferred kernel but when I tried to build it, it errored out with a message saying there was a duplicate declaration of some function. I tried debugging it but nothing worked. Then I had to find an older version of Fedora that had an earlier version of the kernel than the one we wanted. Fedora 27 fit that description with a kernel version of 4.13.100, which I could then ‘upgrade’ to 4.14.111. I created a Fedora 27 VM with 25G disk space. I was able to install kernel version 4.14.111 and build it but when I opened the VM again after restarting it, it said there was no more space on the device. I tried to manually increase the size of the disk, which involves running a command on my local machine after shutting down the VM and then a corresponding command from inside the VM. Unfortunately, before shutting down the VM, I created a screenshot of the VM to save the progress so far. When I tried to turn the VM back on, it was loading for a long time and it never finished loading. I looked up the issue online and I found out that you are not supposed to change the size of a VM that already has saved screenshots, so I clearly made an uninformed decision when I took a screenshot of my VM. I had to delete this VM and recreate it anyway since I messed up so I decided to increase the size of the disk of the Fedora 27 VM from 25G to 30G. This was shown to be a good choice since I did not have any more issues with changing the kernel version of the VM.

Also it should be noted that each time I created a VM, I needed to modify some of the files in the 4.14.111 kernel directory to match the files in the Bias repo so that later on we would be able to run the BIAS impersonation and attack files. Additionally, I also had to download various software packages specified in the internalblue and BIAS repo README files to get the attack to work. Some examples of packages I had to repeatedly download are adb, gcc, binutils-arm-linux-gnueabi, the actual internalblue software, and wireshark. For the internalblue software in the final VM, I had to download it multiple different ways to try to get it to run properly for the attack, and in the end I ran the Development Install command lines to properly download it. Once the VM was set up with the correct kernel version and the correct packages were installed, we all met up numerous times to work on the evaluation board, to get all the information necessary for the attack to work and then actually implement/run the attack, all of which was done on the VM.

### Mariam

Because the VM’s kernel version was a major bottleneck that was preventing us from making any progress I also attempted to downgrade my Ubuntu VM’s kernel version to 4.14.111. I also ran into many problems where there were many dependencies that were deprecated so they each had to manually be downloaded to clear the error and ultimately I ran into the same storage issue, that Shruti was able to clear. Alongside that I did the research on the parts, we simultaneously were not trying to spend too much money but then we also had to make sure we were getting parts that we could run this attack. We were debating older devices as we were convinced newer devices would have patched the vulnerabilities they were targeting. In my research it seemed like ordinary devices that we had on hand would work. I worked on being able to interface with the raspberry pi from my laptop as it would be really helpful in the pairing and connecting process if we had a GUI. This was surprisingly difficult when tied with the fact that school wifi blocks SSH, but we got around it by setting up a hotspot for it. 

Then we all worked together when it came to sniffing the headphones’ information and the attack. The next bottleneck I worked on was figuring out the hci commands and where to look on wireshark to get all the information needed to populate the impersonation file, so we all spent a lot of time on that. With that done we met up to run the attack and troubleshoot and test it. The microsoft headset switching to localhost.localdomain was not the expected behavior as if it had all the correct parameter addresses changed, we shouldn’t have seen any changes on the user (victim) side. So we all spent some time running and rerunning and trying different devices, beyond the Raspberry Pi like Shruti’s phone and my phone to understand what could be happening and what vulnerabilities we exploited ( connecting without pairing and connecting without authenticating) and what vulnerabilities didn’t work (being able to send data). 



======================================================================================
## Original README

Repository about [Bluetooth Impersonation AttackS (BIAS)](https://francozappa.github.io/about-bias/).

* [Instruction to perform the BIAS attacks](https://github.com/francozappa/bias/tree/master/bias)
* [Code to patch linux-4.14.111 to enable H4 parsing](https://github.com/francozappa/bias/tree/master/linux-4.14.111)
    * Make sure to install the relevant kernel modules to interface with the
        devboard. For example, USB serial drivers and device drivers for the
        Bluetooth subsystem.
* [Code to validate the legacy authentication procedure](https://github.com/francozappa/bias/tree/master/la)
* [Code to validate the secure authentication procedure](https://github.com/francozappa/bias/tree/master/sa)

Related work:

* [BIAS: Bluetooth Impersonatoin AttackS](https://francozappa.github.io/publication/bias/) [S&P20]
* [The KNOB is Broken: Exploiting Low Entropy in the Encryption Key Negotiation of Bluetooth BR/EDR](https://francozappa.github.io/publication/knob/) [SEC19]


I'd like to thank Nils Glörfeld for his contributions to reverse-engineer
and patch the CYW920819 development board's firmware, and to patch the Linux kernel.

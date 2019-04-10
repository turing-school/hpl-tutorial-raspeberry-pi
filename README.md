<center><h2 style="text-decoration: none">Setting up a Raspberry PI3 cluster with mpich</h2></center>

<center><i><b>Lukas Baranovas & Vykintas Valužis</b></i></center>

---

### Index

1. [Skill requirements](#skills)

2. [Hardware checklist](#hardware)

3. [Setting up raspbian](#setup) (5-10 min.)

4. [Installing mpich](#mpich) (2-5 min.)

5. [Installing BLAS](#blas) (2-5 min.)

6. [Downloading HPL](#hpl) (5-8 min.)

7. [Connecting the hardware](#connect) (5-10 min.)

8. [Configuring network interfaces](#network) (15-25 min.)

9. [Testing link between the raspberries](#network_test) (5-10 min.)

10. [Setting up SSH keys](#ssh_keys) (5-10 min.)

11. [Testing passwordless SSH](#ssh_test) (2-5 min.)

12. [Testing the mpich connection](#mpich_test) (2-5 min.)

13. [Configuring the HPL benchmark](#hpl_config) (5-15 min.)

14. [Tuning the HPL benchmark](#tuning) (5-20 min.)

15. [Running HPL on one pi](#run_one) (10-30 min.)

16. [Running HPL on both pis](#run_two) (10-30 min.)

#### <i>At most, the process will take 3h30min start to finish</i>

---

### <a name="skills"></a>1. Skill requirements

#### You should know these topics:

- cp, rm, mv, dd; for file manipulation and image writing

- nano, vim or others; for text editing

- ls, cd, pwd, lsblk; for navigating within the filesystem

- cat, less, grep; for reading and analysing files use linux pipes to redirect output to different programs (|)

- Use redirection symbols to append output to a file (<<;>>;)

- Compiling programs with the make command

- Have a basic understanding of network interfaces

### <a name="hardware"></a>2. Hardware checklist

- 2xRaspberry pi3 microcomputers

- 1xComputer with a unix os installed; has to be able to read SD cards

- 2xPower supplies, capable of delivering 5V2A or more current

- 1xEthernet network switch (at least 2 ports)

- 2xMicroSD cards with at least 4GB of space

- 1xHDMI cable with access to a screen

- 2xEthernet cables

- 1xMicroSD to SD adapter

- 1xUSB keyboard with cable

- An internet connection (preferably wireless)

### <a name="setup"></a>3. Setting up raspbian

Download Raspbian Strech Lite image [here](https://www.raspberrypi.org/downloads/raspbian/).

#### For MacOS users

1. Extract your downloaded image in Finder

2. Download any image flashing program, for example [Etcher](https://www.balena.io/etcher/)

3. Flash the image to SD card.

4. Go to 10. step for Linux users.

#### For Linux users:

1. Navigate to the downloaded file and unzip it

    ```

    cd Downloads

    unzip rasp.zip

    ```

2. Locate your SD card with lsblk and make sure it isn't mounted

    ```

    [user@ArchLaptop ~]$ lsblk

    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT

    sda           8:0    0 238.5G  0 disk 

    ├─sda1        8:1    0   100M  0 part /efi

    └─sda2        8:2    0 238.4G  0 part /

    mmcblk0     179:0    0  14.5G  0 disk  

    ```

    - If it is mounted, unmount it with the following command

    `sudo umount /dev/mmcblk0*`

3. Burn the image to the SD card

    

    **Make sure that the path provided after "of=" is for the SD card!**

    

    `sudo dd bs=4M if=image.iso of=/dev/mmcblk0 conf=fsync status=progress && sync`

4. After successfully burning the iso, type lsblk again and find the smaller partition of the two

    ```

    [user@ArchLaptop ~]$ lsblk

    NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT

    sda           8:0    0 238.5G  0 disk

    ├─sda1        8:1    0   100M  0 part /efi

    └─sda2        8:2    0 238.4G  0 part /

    mmcblk0     179:0    0  14.5G  0 disk

    ├─mmcblk0p1 179:1    0  43.9M  0 part <- This one is smaller

    └─mmcblk0p2 179:2    0  14.4G  0 part

    ```

5. Mount the smaller partition and navigate to it

    

    ```

    sudo mount /dev/mmcblk0p1 /mnt

    cd /mnt

    ```

6. Now, create an empty file - .ssh

    

    `sudo touch .ssh`

7. And open a new file for editing - wpa_supplicant.conf

    

    `sudo nano wpa_supplicant.conf`

8. Copy and paste the following configuration, making sure to change the ssid for your network name

    ```

    ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev

    update_config=1

    network={

	    ssid="{wifi}"	    psk="{password}"

    }

    ```

9. Finally, exit out of the /mnt directory and unmount the SD card

    ```

    cd ~

    sudo umount /dev/mmcblk0*

    ```

10. Make sure your wifi network has a decent signal, insert the SD card into the pi and power it on

11. Find your raspberry pi's address with an IP scanner app on your phone or other means f.e. nmap

### <a name="mpich"></a>4. Installing mpich

1. Connect to your raspberry pi through SSH **(both machines must be on the same network)**

    `ssh pi@{ip_address}`

2. You will be asked if you want to trust this machine, type `yes` and press enter

3. Next, you will be prompted for a password. Type **raspberry** and press enter

4. When connected, check that you have a link to the internet with a ping command

    `ping 8.8.8.8 -c4`

    - If you have a link, you should see a similar output

    ```

    [user@ArchLaptop ~]$ ping 8.8.8.8 -c4

    PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

    64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=15.3 ms

    64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=14.8 ms

    64 bytes from 8.8.8.8: icmp_seq=3 ttl=119 time=22.5 ms

    64 bytes from 8.8.8.8: icmp_seq=4 ttl=119 time=14.8 ms

    --- 8.8.8.8 ping statistics ---

    4 packets transmitted, 4 received, 0% packet loss, time 8ms

    rtt min/avg/max/mdev = 14.770/16.839/22.467/3.258 ms

    ```

5. Update the package manager database and upgrade the system packages

    

    `sudo apt-get update -y && sudo apt-get upgrade -y`

6. Download and install mpich through the package manager

    

    `sudo apt-get install mpich -y`

### <a name="blas"></a>5. Installing BLAS

BLAS stands for Basic Linear Algebra Subprograms, and is needed for proper functioning of the

HPL software.

1. Download and install these two packages

    ```

    sudo apt-get install libatlas-dev

    sudo apt-get install libatlas-base-dev

    ```

### <a name="hpl"></a>6. Downloading HPL

HPL stands for Highly Parallel Linpack, which is the actual benchmarking software we will use to run

the benchmarks

1. Get and extract it with the following commands

    ```

    curl -LO http://www.netlib.org/benchmark/hpl/hpl-2.3.tar.gz

    tar -xf hpl-2.3-tar.gz

    ```

    I suggest you check that this link is up to date, as a new version could have been released since the time of writing.

**IMPORTANT - repeat steps 3-6 on the second pi**

### <a name="connect"></a>7. Connecting the hardware

This is where the network switch comes into play, so get your network cables, switch, and a power

source.

1. Connect the network switch to power

2. Connect both ethernet cables to a port on the switch

3. Connect the HDMI cable to one of the pis and to the screen

4. Connect a USB keyboard to the pi with the HDMI cable

5. Plug a power adapter into the socket, and into the pi attached to the screen

If you see a colourful image and/or text scrolling by, it's working!

6. You will be prompted for a username and password. Enter **pi** as the username and press enter

7. Enter **raspberry** for the password and press enter

8. You should now see a line of text similar to this with a blinking cursor

    

    `pi@raspberry~$`

### <a name="network"></a>8. Configuring the network interfaces

We have now connected both pis to the network switch, but in order for them to communicate with

each other, they must be on the same network segment (subnet). We can achieve this by modifying the

file `/etc/network/interfaces` to suit our needs

1. Open the interfaces file for editing and paste the following configuration

    

    `sudo nano /etc/network/interfaces`

    ```

    # interfaces(5) file used by ifup(8) and ifdown(8)

    # Please note that this file is written to be used with dhcpcd

    # For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

    # Include files from /etc/network/interfaces.d:

    #source-directory /etc/network/interfaces.d

    auto lo

    auto lo inet loopback

    auto eth0

    iface eth0 inet static

	    address 10.0.0.40

	    netmask 255.255.255.0

	    network 10.0.0.0

	    broadcast 10.0.0.255

	    gateway 10.0.0.1

    ```

2. Disable the wifi service, reboot, and connect the other pi to the screen and keyboard

    ```

    sudo systemctl disable wpa_supplicant

    reboot

    ```

3. Login into the pi with username **pi** and password **raspberry**

4. Open the interfaces file for editing and paste this configuration

    `sudo nano /etc/network/interfaces`

    ```

    # interfaces(5) file used by ifup(8) and ifdown(8)

    # Please note that this file is written to be used with dhcpcd

    # For static IP, consult /etc/dhcpcd.conf and 'man dhcpcd.conf'

    # Include files from /etc/network/interfaces.d:

    #source-directory /etc/network/interfaces.d

    auto lo

    auto lo inet loopback

    auto eth0

    iface eth0 inet static

	    address 10.0.0.41

	    netmask 255.255.255.0

	    network 10.0.0.0

	    broadcast 10.0.0.255

	    gateway 10.0.0.1

    ```

5. Enable the SSH server and disable the wifi service with these commands

    ```

    sudo systemctl enable ssh

    sudo systemctl disable wpa_supplicant

    ```

6. Reboot this pi and switch the screen and keyboard over to the first one

7. Login, and then disable the ssh server as it was enabled in the setup section

    `sudo systemctl disable ssh`

### <a name="network_test"></a>9.Testing link between the raspberries

Now that we have installed all of the necessary software on both pis, switched off the wifi service,

and connected both of them to the network switch, it's time to **double check your connections and

configurations** and test the link between the two

1. Use the ping command with the ip address of the second pi (10.0.0.41)

    `ping 10.0.0.41 -c4`

    - The pi should respond fairly quickly (<=1s)

### <a name="ssh_keys"></a>10.Setting up SSH keys

Great! Our pis see each other on the network and can send information back and forth, which means we

should move on to getting the SSH on both of them set up.

1. Test the SSH connection from the pi we control to the second pi

    `ssh pi@10.0.0.41`

    Again, we should be asked if we trust this device. Type `yes` and press enter, and authenticate

    with the password **raspberry** when prompted

    

    If we connected successfully, we can start getting rid of that pesky password prompt

2. Disconnect from the second pi and generate your SSH keys

    ```

    exit

    ssh-keygen

    ```

    When prompted for a passphrase or other information, leave the fields blank and press enter

3. Copy your public SSH key to the second pi

    

    `scp /home/pi/.ssh/id_rsa.pub pi@10.0.0.41:/home/pi/.ssh/public_1`

4. Now SSH back to the second pi, and generate its keys

    ```

    ssh pi@10.0.0.41

    ssh-keygen

    ```

    Again, leave the fields blank when prompted and press enter

5. Next, create a file with the contents of the key we copied earlier, delete the original and disconnect

    ```

    cat /home/pi/.ssh/public_1 >> /home/pi/.ssh/authorized_keys

    rm /home/pi/.ssh/public_1

    exit

    ```

6. Copy the second pi's public SSH key over to the local directory and create a file in the same manner

    ```

    scp pi@10.0.0.41:/home/pi/.ssh/id_rsa.pub /home/pi/.ssh/public_2

    cat /home/pi/.ssh/public_2 >> /home/pi/.ssh/authorized_keys

    rm /home/pi/.ssh/public_2

    ```

### <a name="ssh_test"></a>11.Testing passwordless SSH

If everything went according to plan, you should now be able to connect without a password! Test this by simply connecting to the second pi again

    

    `ssh pi@10.0.0.41`

### <a name="mpich_test"></a>12.Testing the mpich connection

To test the mpich connection, we should create something called a hostfile, containing the IP

addresses of all nodes (2 in our case), so that we wouldn't have to type them out every single

time we want to run a program

1. Create a new file for editing and write all of the IPs seperated by a newline

    `nano /home/pi/hostfile`

    ```

    10.0.0.40

    10.0.0.41

    ```

2. Save it with Ctrl+x, then pressing the `y` key

3. To test the mpich connection run the following command

    `mpiexec -f /home/pi/hostfile -n 2 hostname`

    The test will be successful if you get a couple of lines printed back with no errors

### <a name="hpl_config"></a> 13. Configuring the HPL benchmark

We have properly set up mpich and tested the connection between each node, so we can now configure

and compile the benchmarking software for our pis

1. Navigate to setup directory

    

    `cd /home/pi/hpl-2.3/setup/`

2. Here, we can see some Makefile templates for various architectures. Unfortunately, there isn't one for our pis. We can remedy this by modifying the Make.Linux_ATHLON_CBLAS file

3. Firstly, make its copy at the start of the HPL directory

    ```

    cd ..

    cp setup/Make.Linux_ATHLON_CBLAS ./Make.rpi

    ```

    rpi is just an arbitrary name we thought up, you can choose whatever suffix you like

4. Open the copied Make.rpi in your preferred text editor

    `vim Make.rpi`

5. Now you'll need to find the corresponding lines and make these changes to the file

    ```

    ARCH = rpi

    TOPdir = $(HOME)/hpl-2.3

    MPdir = /usr/local/mpich

    Mpinc = -I/usr/include/mpich/

    MPlib = $(MPdir)/libmpich.a

    LAdir = /usr/lib/atlas-base/

    LAlib = $(LAdir)/libcblas.a $(LAdir)/libatlas.a

    CCFLAGS = $(HPL_DEFS) -pthread -fomit-frame-pointer -O3 -funroll-loops -W -Wall

    ```

6. You are now all set to compile HPL. Execute

    `make arch=rpi`

7. After `make` has successfully concluded, navigate into the bin/rpi directory

    `cd bin/rpi`

8. You should find two files there: `HPL.dat` and `xhpl`

    `ls`

    

    The `xhpl` file should show up green. If it doesn't, run `chmod +x xhpl`.

### <a name="tuning"></a> 14. Tuning the HPL benchmark

Before running the benchmark, you'll need to adjust some benchmarking parameters to achieve peak performance and to prevent your pis from exploding ;)

1. Open HPL.dat

    

    `vim HPL.dat`

2. We recommend these settings, although we do provide a preconfigured file later on if you're lazy

    - Set all "# of {N, NBs, Process Grids, Ps & Qs}" to 1 unless you want to run a few tests in sequence. In the steps below it's assumed the values are set to 1.

    - "Ns" line should be around 80%-90% of your total RAM for the most accurate results, but for testing it could be lower to increase the test speed (while sacrificing accuracy).

    Formula for calculating 90% of total RAM: ![Equation should be here, but it isn't :(](https://i.imgur.com/aKBTAAI.png)

    - You can leave "NBs" set to default or try to adjust it for better performance. Recommended interval is 96-256.

    - Set "Ps" and "Qs" so that P * Q would be equal to total cores of your cluster (Rpi has 4 cores each). P should be lower than Q. For two nodes they could be 2 and 4

    - Other parameters are related to the algorithm and can be left default.

Here's our preconfigured file for the lazy ones (2 node setup): [HPL.dat](https://drive.google.com/file/d/1LmQkaTejnot7_pNXbDcBR_arTTF9a_AX/view?usp=sharing)

### <a name="run_one"></a>15. Running HPL on one node

Now you can run your HPL on one node first just to test if everything is working. One fried pi is better than two.

1. Make sure HPL.dat is configured for one node

  - For testing, you can set *Ns to 1000*

  - P * Q should be equal to the total number of cores (4 for each node)

2. Let's run the benchmark!

    `mpirun -n 4 ./xhpl`

3. If your pi is still alive, you will get something like this

    ```

    ================================================================================

    T/V                N    NB     P     Q               Time                 Gflops

    --------------------------------------------------------------------------------

    WR11C2R4       21400   128     3    11              537.10              1.210e-01

    HPL_pdgesv() start time Mon Jun 23 17:29:42 2014

    HPL_pdgesv() end time   Mon Jun 23 17:55:19 2014

    --------------------------------------------------------------------------------

    ||Ax-b||_oo/(eps*(||A||_oo*||x||_oo+||b||_oo)*N)=        0.0020152 ...... PASSED

    ================================================================================

    ```

### <a name ="run_two"></a>16. Runing HPL on multiple nodes

Finally, you can run the benchmark on any number of not yet fried pis (provided that you install all the necessary

software). Here it's assumed you have two of them.

**IMPORTANT - when running the test on multiple nodes, the HPL.dat file must be identical (you must

update it with every change you make), and here's a nifty command to update it**

`ssh -t pi@10.0.0.41 "rm -rf hpl-2.3" && scp -r /home/pi/hpl-2.3 pi@10.0.0.41:/home/pi/`

1. Again, adjust HPL.dat for more memory and cores

  - "Ns" can be left at 1000 for faster results, but if you want them to be more accurate, you should calculate it according to the formula above

  - Ps multiplied by Qs should yield the total number of cores in the cluster

2. Run the benchmark

    `mpiexec -f /home/pi/hostfile -n 8 ./xhpl`

3. If your "N" is set to a high number, it will take some time. Also your pis' CPUs will become pretty [hot](https://cdn.vox-cdn.com/thumbor/2q97YCXcLOlkoR2jKKEMQ-wkG9k=/0x0:900x500/1200x800/filters:focal(378x178:522x322)/cdn.vox-cdn.com/uploads/chorus_image/image/49493993/this-is-fine.0.jpg).

4. Again, if your pis are alive, you will get results in a similar format as provided in the earlier section.

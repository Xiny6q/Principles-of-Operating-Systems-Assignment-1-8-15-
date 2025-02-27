Download link :https://programming.engineering/product/principles-of-operating-systems-assignment-1-8-15/


# Principles-of-Operating-Systems-Assignment-1-8-15-
Principles of Operating Systems Assignment 1 (8/15)
Document Outline:

Page 2: Introduction

Pages 3 – 4: Setting up a Virtual Machine (Question 1)

Pages 5 – 6: Familiarizing Yourself with the Kernel and System Calls (Questions 2 – 3)

Pages 7 – 10: Adding a System Call to the Linux Kernel (Questions 4 – 6)

Pages 11- 12: Compiling and Installing the New Kernel (Questions 7 – 8)

Page 13: Test the System Call (Question 9)












Introduction


In this assignment, you will study the system-call interface provided by the Linux OS and learn how user programs communicate with the OS kernel via this interface. Your task is to incorporate a new system call into the kernel, thereby expanding the functionality of the OS.


A user-mode procedure call is performed by passing arguments to the called procedure either on the stack or through registers, saving the current state and the value of the program counter, and jumping to the beginning of the code corresponding to the called procedure. The process continues to have the same privileges as before.


System calls appear as procedure calls to user programs but result in a change in execution context privileges. In Linux, a system call is accomplished by storing the system-call number into the EAX register, storing arguments to the system call in other hardware registers, and executing a trap instruction. After the trap is executed, the system-call number is used to index into a table of code pointers to obtain the starting address for the handler code implementing the system call. The process then jumps to this address, and the privileges of the process are switched from user to kernel mode. With the expanded privileges, the process can now execute kernel code, which may include privileged instructions that cannot be executed in user mode. The kernel code can then carry out requested services, such as interacting with I/O devices, and can perform process management and other activities that cannot be performed in user mode.


Note: I will use nano as the text editor, but you can use any editor (e.g., vim or emacs) you like. I also named my system call “helloworld”, but you can name your system call whatever you like.


Reminder: Please be aware of the number of virtual machines you create and delete any VMs you are no longer using. Each time you create a VM it is allocated resources (e.g., RAM and disk), and the server has a limited amount of resources.















I. Setting up a Virtual Machine

a. Log into Xen Orchestra using your UNO credentials.

https://cyber-range.cs.uno.edu/


b. After you log in, I will be able to add you to the class and you can set up your virtual machine. You may have to wait a day or two to be added so make sure you do this right away! You will not be able to continue until you are added to the class, and I cannot do that until you log in for the first time.


c. On the left-hand side, select “New” -> “VM”.


d. Choose: Create a new VM on CSCI4401


e. Choose:

Template: “4401_ubuntu_server”

Name: “[last name]-[4401/5401]”

You can leave all of the other default settings (e.g., 1 CPU, 4 GB of RAM, 25 GB of disk storage).




f. Select “Create” at the bottom. It may take your VM several minutes to start up.


g. Go to the “Console” tab and login (username: student, password: student). You can change your password with the command $passwd.





Question 1. Run $echo “Hello from [your name]”. Submit a screenshot of your running VM. Below is an example:





II. Familiarizing Yourself with the Kernel and System Calls


a. Look up your current kernel version with the following command. We will download and use a different version later so you can confirm that you successfully loaded the new kernel.


$uname -r


b. Run the following command. You should see a config file, System.map, and a vmlinuz file that correspond to your kernel version in the previous question. Later, when we install the new kernel, the (new) config file, System.map, and vmlinuz files will also be in this directory.


$ls /boot/



Question 2. What is your current kernel version? Hint: it should be in the format 5.x.x.



c. Download and install the essential packages to compile kernels:


$sudo apt-get update


$sudo apt install git fakeroot build-essential

libncurses-dev libssl-dev libelf-dev bison flex xz-utils -y


Note: If you see a prompt asking what services should be restarted, hit tab and enter or “OK”.


d. Locate your system call files by issuing the following command (note the hyphen at the end):


$printf SYS_read | gcc -include sys/syscall.h -E –


e. The system call numbers for the recent version of the Linux kernel are listed in unistd_64.h. Go to the relevant directory (look at your output from Step d for the file unistd_64.h and use $cd to change the directory). Issue the following command to look at the system call numbers.


$less unistd_64.h


(screenshot on next page)



For example, __NR_close corresponds to the system call close(), which is invoked for closing a file descriptor, and is defined as value 3.




Question 3. Submit a screenshot (similar to the example above) with at least ten of your system calls.
















­­

III. Adding a System Call to the Linux Kernel


a. https://cdn.kernel.org/pub/linux/kernel/v5.x/ contains different kernel versions you can download. Select a version to download that is different from your current version. For example, I chose version 5.16.13.


$wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.16.13.tar.xz


b. Unpack the tarball you just downloaded


$tar -xvf linux-5.16.13.tar.xz


c. Go to the directory for your source code ($cd linux-5.16.13/). Create a home directory for your system call (e.g., I named mine “helloworld”).


$mkdir helloworld


d. Move to that directory ($cd helloworld/). Create a C file that matches your system call name:


$nano helloworld.c


e. Write the following code in it, but replace my name with your name:


Note: SYSCALL_DEFINE0 – that’s a zero at the end.



Question 4. Submit a screenshot of your system call code.

f. In the same directory, create a Makefile for your system call with the following command:


$nano Makefile


g. Type the following in it:




h. Next, you will add the home directory of your system call to the main Makefile of the kernel. To do this, go back up to the directory for your kernel source code ($ cd ..). Search for “core-y” in the Makefile. You will want the second result (at line 1095 in my screenshot) that contains a series of directories (kernel/, certs/, mm/, etc.):


$grep -n ‘core-y’ Makefile





i. Edit the Makefile by adding the directory of your system call. Note: directories end with a forward slash.



$nano Makefile

Or you can jump to the line number

$nano +[your line#] Makefile






Question 5. Submit a screenshot of your modified Makefile.



j. Open the header file:


$nano include/linux/syscalls.h


k. Go to the bottom of the file and write the following code just above #endif.


asmlinkage long sys_[name of your system call](void);







l. Next, you will add your system call to the kernel’s system call table in the source code you downloaded (arch/x86/entry/syscalls/syscall_64.tbl). Open the table with the following command:


$nano syscall_64.tbl


m. Navigate to the bottom. My last system call has a number of 547 so I will use 548 (you should also use N+1).






Question 6. Submit a screenshot of your syscall_64.tbl that includes your system call.













IV. Compiling and Installing the New Kernel


a. Copy the existing config file from the boot directory. From the directory of the source code:


$cp /boot/config-$(uname -r) .config


b. Set the default configuration.


$make defconfig


c. Compile the kernel’s source code. This could take 30 – 60 minutes.


$sudo make


d. Install the kernel modules and install the kernel.


$sudo make modules_install

$sudo make install


e. The new kernel config file, System.map, and vmlinuz file should be in the /boot/ directory.


$ls /boot/








Question 7. Submit a screenshot of your boot directory containing those files.



f. Run the following command to enable the new kernel for a boot.


$sudo update-initramfs -c -k 5.16.13


g. Update the bootloader GRUB.


$sudo update-grub


h. Reboot your system


$shutdown -r now


i. When you restart your system (while the console is still black), hit the button in the upper left just above the console that looks like a keyboard and says “Send Ctrl-Alt-Delete” when you hover over it. You should get a screen like the following:




j. Select “Advanced options for Ubuntu” and then select your new kernel version.


k. One your VM is finished starting up, verify your (new) kernel version.

$uname -r



Question 8. Submit a screenshot that verifies you are using your new kernel version.

V. Testing the System Call

a. Write a script that calls your system call. Below is an example:

$nano test.c




b. Compile and run your script:

$gcc test.c -o test.o && ./test.o


c. If successful, it prints “System call sys_hello returned 0”. Check the kernel log buffer to see if your message is there. This will print many lines, but your message should be near the bottom.

$dmesg



Question 9. Submit a screenshot of your output from $dmesg containing the message from your system call.

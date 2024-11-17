1. Installation guide/wiki
	1. https://wiki.archlinux.org/title/Installation_guide
	
2. Acquire image
	1. Download
		1. Pick a release from the downloads page: https://archlinux.org/download/
		2. I did one from MIT (codi chose it in class so I decided to play it safe and follow suit) https://mirrors.mit.edu/archlinux/iso/2024.10.01/archlinux-2024.10.01-x86_64.iso
	2. Verify
		1. Cd'd to my downloads folder in windows terminal and ran
			1. CertUtil -hashfile archlinux-2024.10.01-x86_64.iso SHA256
			2. Compared it with what was on the wiki and there were no problems
	
3. Setting up VM in VMware
	1. Click create a New Virtual Machine
	2. Select iso from downloads folder (Ignore warning about no operating system detected and hit next)
	3. Guest operating system is Linux and set the version to "Other Linux 6.x kernel 64-bit"
	4. Named it Arch Linux Demo
	5. Open machine settings, hit the options tab and go to "Advanced", in the mid to lower section there should be a button to switch from BIOS to UEFI, do that; if you don't you'll restart every time you shutdown your machine.
	
4. Pre-Installation
	1. Open the VM, select the Arch linux medium when booting and wait for it to install and a command line to appear.
	2. The boot mode should be verified so run, if it returns 64 then that confirms UEFI boot mode
		1. Cmd: cat /sys/firmware/efi/fw_platform_size\

	3. Connect to the internet
		1. Didnt need to mess with it, just pinged archlinux.org
	
	4. Update the system clock
		1. Cmd: timedatectl

	5. Partition the disks,
		1. Ran fdisk -l, to see what the system currently has partitioned
		2. Running fdisk /dev/sda, allowed me to partition free space into usable segments of storage
			1. Setting up EFI partition,
					1. This is where I ran into a lot of initial trouble, the arch linux provides no clarification, really just a blank page so i didnt know what it was expecting. Spent quite a bit of time on google and using LLM's trying to find what I need.
				1. type "n" and hit enter
				2. n opens the menu for selecting partition types (primary and extended) type p, or hit enter(primary is default) to select primary and continue.
				3. Type "1" to make it the first partition
				4. Hit enter for the default in first sector (2048)
				5. For the last sector state, "+1G", for 1 GiB of storage
				6. With this the first partition has been created but isnt done, type "t" to change the type.
				7. Hitting t will display the selected partition and allow running 'L' to see all available types. "ef" states for EFI so that is what should be run in the terminal next.
			3. Setting up root partition
				1. type "n" again
				2. partition type will be p
				3. partition number is 2
				4. first sector is default
				5. second sector is +8G (arch recommends 23GB but I didnt allocate the storage for that and id have to restart so we'll see)
			4. Setting up swap
				1. (I didnt think i needed to do it so i didnt >:D)
			5. To save your partition work type w to write and exit the editor
		3. Format partitions
			1. root formatting
				1. Cmd: mkfs.ext4 /dev/sda2
			2. EFI Formatting
				1. Cmd: mkfs.fat -F 32 /dev/sda1

		4. Mounting file system
				- For root
					- Cmd: mount /dev/sda2 /mnt
				- For EFI
					- Cmd: mount --mkdir /dev/sda1 /mnt/boot
	
5. Installation
	1. Select mirrors
		1. According to wiki it seems like it's automatically done
	2. Installing packages
		1. Cmd: pacstrap -K /mnt base linux linux-firmware
		2. Cmd: pacman -Sy man-db
		3. Cmd: pacman -Sy open-vm-tools
		4. Cmd: pacman -Sy networkmanager
			1. systemctl enable NetworkManager.service
			2. systemctl start NetworkManager.service

	3. Configuring the system
		1. Fstab (used to define disk partitioning)
			- make an fstab file
				- Cmd: genfstab -U /mnt >> /mnt/etc/fstab
		2. Change root for new system- 
				1. A few steps later I started wondering why this command was necessary and what kept me from doing all of the below within the normal root, I still dont know much but i did learn it's kinda like an environment used for system maintenance 
			1. Cmd: arch-chroot /mnt
		4. Set Time zone
			1. Cmd: ln -sf /usr/share/zoneinfo/US/Central /etc/localtime
			2. run hwclock to ensure it was sent correctly
		5. Localization
			2. Setting language
				1. Cmd: pacman -Sy nano to install nano so i could edit the files
				2. nano /etc/locale.gen
					1. found en_US.UTF-8 UTF-8 and uncommented
				3. ran Cmd: locale-gen
				4. cd to /etc and touch locale.conf
				5. then nano into it and write LANG=_en_US.UTF-8
		6. Network Config
			1. nano /etc/hostname
				1. (name the pc) I said wrote Teletran
		7. Initramfs
			1. I read "usually not required" and decided to move on, do not feel like over achieving today sorry
		8. Root password
			1. Cmd: passwd
				1. What is my password? Well arent you =curious= ?
		9. Installing boot loader
			1. Looked at compatible loaders and chose grub
				1. Downloading grub and efi boot manager
					1. Cmd: pacman -S grub efibootmgr
				2. Cmd: grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
				3. Cmd: grub-mkconfig -o /boot/grub/grub.cfg 
						1. I overlooked this command on the grub page for quite a while and it gave me a headache for an hour or two since without it whenever I booted to the Arch instead of the Arch iso a strange terminal was given, as in there was no familiarity to it, no login prompt or machine specifications. 
	At this point shut down the VM, go into vm management and remove the ISO/disable it as an option to boot
	
1. This is where the installation ends, for class I must install a desktop environment, a shell, ssh, color coding for terminal, add aliases, create a couple users that have sudo permission and configure the vm to boot into the GUI.
	1. Install Desktop environment (Assignment recommendation was LXQT)
		1. LXQT ArchLinux installation wiki: https://wiki.archlinux.org/title/LXQt, 
				1. At first the wiki annoyed me because it just says "Install lxqt and xorg server" but never provided a means how, turns out it's quite literal, I was also troubled by the fact by the lack of time it took to install the Desktop environment and spent another chunk of time doubting that it was the complete installation. (It wasn't im glad i spent time reading otherwise i would have forgotten the necessity for drivers)
				
			1. First install Xorg
				1. Cmd: pacman -S xorg-server
			2. Install drivers needed in GUI
				1. Cmd: pacman -S xf86-input-vmmouse (mouse driver)
				2. Cmd: pacman -S xf86-video-vmware (video driver)
			3. LXQT
				1. Cmd: pacman -S lxqt sddm
					1. sddm is display manager
						1. systemctl enable sddm
						2. systemctl start sddm
					2. lxqt is the Desktop environment
						1. selected all the defaults when prompted in installation
					3. Install icons
						1. Cmd: pacman -S oxygen-icons
					4. Other managers
						1. I realized I was missing these once I completed all the steps below, symptoms were: missing icons and clicking on the computer and network placeholders resulted in operation not supported, luckily wiki forums pointed out the packages I was missing.
							1. pacman -S gvfs, this fixed the issue with operation not supported
							2. Unfortunately it took me an hour and a half to fix the missing icon issue, as it turns out i installed oxygen-icons when the system was looking for breeze-icons by default, I found it by using nano on the lxqt.conf file.
								1. so as a result pacman -R oxygen-icons
								2. pacman -S breeze-icons
							
	2. Adding ssh
		1. Cmd: pacman -S openssh
		2. Cmd: systemctl start openssh
	3. Adding a shell and customizing it
		1. Installation of the fish shell: surprisingly the install was easy, archlinux provided a good wiki about fish, and when i opened it there was already a lot of color which was nice.
			1. Cmd: pacman -S fish
		2. Changing the Color coding:
			1. Fish comes colorcoded for different command types but if i wanted to change it
				1. Open the shell- Cmd: fish
				2. Cmd: fish_config
					1. it required downloading python, so pacman -S python
				3. fish_config provided a link to web interface, so continuing requires rebooting in DE and having a browser, pacman -S firefox
		3. Adding aliases
			1. fish
				1. Cmd: alias ...="cd ../.." -s
				2. Cmd: alias c=clear -s
				3. Cmd: alias cd..="cd .." -s
				4. Cmd: alias h=history -s
				5. Cmd: alias j="jobs -l" -s
			2. After what I thought was the completion of the desktop environment I did some tests on the Desktop to ensure I could access everything I installed and found that none of the fish alises worked. This is where i started investigating the relationship between chroot and the rest of the system. 
				1. I discovered the issue is that when saving the aliases above they are saved as functions in a folder that is specific to each user (this is because alias is a function that is used to create functions), however I want to make it so that the 5 aliases are universal.
				2. Cmd: sudo nano /etc/fish/config.fish
					1. Adding all of the alias commands above (without the -s) to the config file fixed the problem. Another option would be creating a shared alias file that fish referred to but I chose the quickest fix.
		
	4. Adding users
		1. pacman -S sudo vi (needed to edit the /etc/sudoers file)
		2. Create myself, justin, and codi
			1. Cmd: useradd myself -m
				1. Cmd: passwd myself
				2. Password hint: Well aren't you curious?
				
			2. Create justin- Cmd: useradd justin -m
				1. Set password- Cmd: passwd justin
					1. Password: justin might know
				2. Change password after login- Cmd: passwd -e justin
				
			3. Create codi- Cmd: useradd codi -m
				1. Set password- Cmd: passwd codi
					1. Password: Codi knows 
				2. Change password after login- Cmd: passwd -e codi
			
		1. Give new users sudo permissions
			1. sudo visudo
				1. find the #%sudo ALL=(ALL:ALL) ALL and uncomment it
				
			2. Commands for sudo
				1. tried Cmd: sudo usermod =aG sudo myself, to add myself to the sudo group but got the response that the sudo group did not exist.
				2. Create sudo group
					1. sudo groupadd sudo
				3. Did  (Cmd: sudo usermod =aG sudo myself) again and it worked
				4. Cmd: sudo usermod =aG sudo justin
				5. Cmd: sudo usermod =aG sudo codi
				6. ran Cmd: getent group sudo to confirm everyone was in the group
	5. Setting up boot
		1. Cmd: sudo systemctl set-default graphical.target
		2. Cmd: sudo reboot


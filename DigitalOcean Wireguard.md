
Ocean gate set up:

Terminal:
1. Connect to droplet you just made:
	1. Cmd: ssh root@165.232.143.161
	2. Say yes to continue connecting and input the password you made during setup
2. Docker install: https://thematrix.dev/install-docker-and-docker-compose-on-ubuntu-20-04/
	1. Install necessary tools:
		1. Cmd: sudo apt install apt-transport-https ca-certificates curl software-properties-common -y
	2. Add Docker Key
		1. Cmd: curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
	3. Add docker repo
		1. 32bit / 64bit OS
			1. Cmd: sudo add-apt-repository \"deb [arch=amd64] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) \stable"
		2. Switch to repo
			1. Cmd: apt-cache policy docker-ce
	4. Install Docker
		1. Cmd: sudo apt install docker-ce -y
		2. Run without root:
			1. Cmd: sudo usermod -aG docker ${USER}
	5. Install Docker-Compose
		1. Cmd: sudo curl -L "https://github.com/docker/compose/releases/download/1.27.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
		2. Permissions
			1. Cmd: sudo chmod +x /usr/local/bin/docker-compose
	6. Docker check
		1. sudo docker run hello-world
		2. docker compose version
3. WireGuard setup:
	1. Make directories for wireguard config and a yml file for compose
		1. Cmd:
		   mkdir -p ~/wireguard/
		   mkdir -p ~/wireguard/config/
		   nano ~/wireguard/docker-compose.yml
		2. Paste config below into the nano:
			1. 
				version: '3.8'
				services:
				  wireguard:
				    container_name: wireguard
				    image: linuxserver/wireguard
				    environment:
				      - PUID=1000
				      - PGID=1000
				      - TZ=Asia/Hong_Kong
				      - SERVERURL=1.2.3.4
				      - SERVERPORT=51820
				      - PEERS=pc1,pc2,phone1
				      - PEERDNS=auto
				      - INTERNAL_SUBNET=10.0.0.0
				    ports:
				      - 51820:51820/udp
				    volumes:
				      - type: bind
				        source: ./config/
				        target: /config/
				      - type: bind
				        source: /lib/modules
				        target: /lib/modules
				    restart: always
				    cap_add:
				      - NET_ADMIN
				      - SYS_MODULE
				    sysctls:
				      - net.ipv4.conf.all.src_valid_mark=1
			2. TZ needs to be changed for your time zone, I am cst so i am in america/chicago which is CST6CDT
			3. SERVERURL is the address of your droplet server, the one you used to in the ssh root@ command
			4. Finally Peers needed to be changed, it is responsible for generating the user-config-files, I did not want to deviate from the guide so I did PEERS=3(it does the same as what is currently listed, just different names and definition), to allow for 3 devices to connect.
	2.  Starting Wireguard
			1. Cmd: cd ~/wireguard/
			2. Cmd: docker-compose up -d
4. Connecting my phone
	1. Cmd: docker-compose logs -f wireguard
	2. The command runs the wireguard application and displays a lot of information on the terminal, including QR codes that correspond to the PEERS you had made in step 3.2.4
	3. The next step is to download Wireguard VPN on your phone, hit the plus sign and create a tunnel from QR code, scan one of the codes you displayed, I did PEER Phone 1, and your basically done, all thats left is to hit the on switch on your phone's app.
5. Connecting my laptop
	1. Unfortunately connecting your laptop to wireguard is not as intuitive
	2. Install the Wireguard app for your laptop: https://www.wireguard.com/install/
	3. Launch the program and select the add tunnel  dropdown on the left hand side of the screen, then click add empty tunnel.
	4. Upon creation a dialog box will pop up, with the text 
		1. [Interface]
		2. PrivateKey= (whatever your private key is)
	5. Next go back to terminal and find your peer.conf files
		1. cd ~/wireguard/
		2. cd config
		3. cd peer_pc1 (that was one of the peer names set in step 3.2.1, if you named it differently then your command will be cd (yourpeername))
		4. cat peer_pc1.conf (cat your peer file)
	6. After using cat on your peer.conf file copy all of the contents over to the dialog box fireguard produced when adding an empty terminal and ensure that it replaces what came with the dialog box (step 4.1.4 Part 1-2).
	7. Give the tunnel a name and click save, afterwards hit the activate button and your vpn should work.

6. Bonus:
	1. Requirements
		1. Want to use a port that maybe won’t be blocked on public WiFi (like 80)… you can change it in docker-compose.yml file BUT you have to set up port forwarding on your Ubuntu server to send port 80 comms to port 51820 in the Docker container. How? Not gonna tell you… that is why it is a BONUS.  
	2. I tried it and failed. I used so many different commands that I think I lost track and forgot to write them down, essentially I edited docker-compose.yml to 80:51820/udp in the ports area, then tried routing and redirecting in Ip table for port 80 to port 51820. I think I leaned too heavily into the internet and forums because I probably wrote a command or saved the tables incorrectly and now wireguard can activate the tunnels and establish a vpn connection but a timeout error is produced when trying to display the QR codes I did in a previous step or visit a website while using the vpn.
		1. This project was still a lot of fun though learned a lot of lessons, such as how I need to remember to document more.



My docker-compose.yml
GNU nano 8.1                       docker-compose.yml                               version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=CST6CDT
      - SERVERURL=165.232.143.161
      - SERVERPORT=51820
      - PEERS=pc1,pc2,phone1
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 51820:51820/udp
    volumes:
      - type: bind
        source: ./config/
        target: /config/
      - type: bind
        source: /lib/modules
        target: /lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
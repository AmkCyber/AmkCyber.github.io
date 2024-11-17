1. Docker installation
	1. Set up Docker's apt repository
		1. sudo apt-get update
		2. sudo apt-get install ca-certificates curl
		3. sudo install -m 0755 -d /etc/apt/keyrings
		4. sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
		5. sudo chmod a+r /etc/apt/keyrings/docker.asc
		6. echo \
			"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \_(the _ is supposed to be a space
			$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
			sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
		8. sudo apt-get update
		
	1. Install the Docker packages
		1. sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
		
	2. Verify successful installation
		1. sudo docker run hello-world
			
2. Post installation steps
	1. Manage Docker as a non-root user
		1. sudo groupadd docker
		2. sudo usermod -aG docker $USER
		3. Log out and log back in so group membership re-evaluated
		4. docker run hello-world
	2. Configure Docker to start on boot with systemd
		1. sudo systemctl enable docker.service
		2. sudo systemctl enable containerd.service

3. Install docker compose check (should have come installed)
	1. sudo apt update  
	2. sudo apt install docker-compose-plugin  
	3. docker compose version

4. Choosing app to install on docker
	1. I chose greenbone
		1. sudo docker run -d -p 443:443 --name mikesplain/openvas
			1. after the installation completes, open system monitor and wait till cpus are no longer being utilized at 100% (NVTs are still being downloaded)
		2. Open firefox go to https://localhost and accept security risk
			1. login and password are both admin
		3. At this point the greenbone install is done, everything else is up to personal preference and tinkering. (Essentially all of the below is just me tinkering with configurations and seeing what Greenbone has to offer)
			1. I thought thered be more to write here but I looked at the all the dasboards for scanning, assets, config, and sec info to get my feel for the app.
			2. After some googling I found out how to do a scan.
				1. First hover over the configuration tab and select target. 
					1. On the target page hit the blue box with the white star in the upper left. 
					2. Create a name for the target and provide the host IP; save it and then hover over the Scans tab. 
				2. Select tasks to pull up the tasks dashboard. 
					1. Click on the blue box with the white star again in the upper left and to create a scanning task. 
					2. Name the task and select your desired target, there are several other configuration options but if you just want to use defaults dont change anything else and hit save.
					3. Finally look at the table beneath the pi charts, and you should see the task you just made hit the green box with the white triangle to start the scan.
					4. The scan will take a while so refresh your page periodically to see the status/progress.
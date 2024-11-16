---
title: DockerInstall
filename: Docker lab 1.md
---

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
			
1. Post installation steps
	1. Manage Docker as a non-root user
		1. sudo groupadd docker
		2. sudo usermod -aG docker $USER
		3. Log out and log back in so group membership re-evaluated
		4. docker run hello-world
	2. Configure Docker to start on boot with systemd
		1. sudo systemctl enable docker.service
		2. sudo systemctl enable containerd.service

1. Install docker compose check (should have come installed)
	1. sudo apt update  
	2. sudo apt install docker-compose-plugin  
	3. docker compose version

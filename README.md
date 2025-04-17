This repository is to store notes on spinning up a mini aws malcolm-caldera lab.

### Initial Setup
- Setup a VPC
- Setup a Subnet in the VPC
- Setup a Security Group for the VPC
	- For testing purposes, setup an inbound rule for all IPv4 traffic from your local host machines Public IP. This was you can interact with the web interface of Malcolm and Caldera. For configuration of the Malcolm and Caldera boxes and access to additional boxes you will be using SSH. 

### Malcolm box
- Launch an EC2 instance - Ubuntu, t3.2xlarge, 100 GB
	- Be sure to select the VPC, Subnet, and Security Group you created earlier.
- Go to the connect page of this Ubuntu EC2 box and select SSH client.
	- In a termainl window, copy paste the example ssh command on the page to connect to you EC2 instance. 
- Helpful resource - https://malcolm.fyi/docs/quickstart.html#QuickStart
- `git clone https://github.com/cisagov/malcolm`
- `cd malcolm/scripts`
- Use the `ip a` command to find the network interface you want to capture live traffic on. 
- `sudo ./install.py`
- Be sure to add your username on the ubuntu box for the 'Add non-root user to the docker' group question. 
	- For an Ubuntu box in AWS the username is likely 'ubuntu'.
	- After the install, use the command `getent group docker` to confirm your username is in the docker group. 
- Reboot the system
- `cd malcolm/scripts`
- `./auth_setup`
- `docker comppose --profile malcolm pull`
- To start Malcolm be sure you are in the malcolm -> scripts directory and run the command `./start`. 
- Once Malcolm has finsihed starting, you should be able to go to the public IP of the Malcolm box from your host computer's browser to view the Malcolm web interface. 
	- Be sure NOT to use httpS when going to the Malcolm boxes' public IP.

### Caldera box
- Launch an EC2 instance - Ubuntu, t3.medium, 20 GB
	- Be sure to select the VPC, Subnet, and Security Group you created earlier.
- Go to the connect page of this Ubuntu EC2 box and select SSH client.
	- In a termainl window, copy paste the example ssh command on the page to connect to you EC2 instance. 
- `sudo apt install python3-venv` - Install Python's virtual environment package
- `python3 -m venv .venv` - Create a Python virtual environment
- `source .venv/bin/activate` - Activate the virtual envrionment
- `git clone https://github.com/mitre/caldera.git` - Clone the Caldera github repo
- `sudo apt-get update` - Update your Ubuntu system
- `sudo apt install nodejs npm` - Install nodejs and npm packages (these will be installed in the virtual environment)
- `cd caldera> `- Change to the newly created caldera directory from the git clone command
- `pip3 install -r requirements.txt` - Install the python package requirements for caldera
- `python3 server.py --insecure --build` - Start the caldera server 
- The Caldera server should be running now. To access the web interface, go to the public IP address for the Ubunutu box Caldera is installed in. Be sure to include port 8888 in the URL and use http, NOT httpS. 
	- x.x.x.x:8888
	- Username: red, Password: admin (defulat Caldera creds)

### Setup Traffic mirror
- Helpful resource - https://docs.aws.amazon.com/vpc/latest/mirroring/what-is-traffic-mirroring.html
- Tutorial on setting up Traffic Mirrors in AWS - https://medium.com/@yojohndunn/aws-traffic-mirroring-4d1fa60d9d6f
- From the VPC page in AWS, on the left toolbar, go to where it says Traffic Mirroring
- **Select Mirror targets**
	- Create a new traffic mirror target
	- Set the Target type to 'Network interface'
	- Set the Target to the network interface of your Ubuntu box you set Malcolm up on. 
		- Network interface IDs can be found under the 'Networking' tab of the EC2 instance page.
		- Network interface IDs should start with 'eni-'
- **Create a Mirror filter**
	- Create a new mirror filter
	- For this example, we are only going to set Outbound rules because we are going to create a Mirror Session for each box in the subnet (except the Malcolm box which will be our Mirror Target)
	- Under 'Outbound rules - optional' click 'Add Rule'
	- The first outbound rule will be to reject all TCP port 22 traffic with a source and destination set to 0.0.0.0/0
		- This will disregard the traffic from our host machine to the individual AWS EC2 instances. 
		- Obviously, this will reject additional SSH connections (including any internal to the subnet), oh well for now. 
		- Pick an arbitrary rule number. I suggest '100'. 
	- The second outbound rule will be to accept all the other traffic minus what is being rejected in our first rule.
		- Pick an arbitrary rule number that is greater than the number in the first rule. I suggest '200'. 
		- Select 'accept', 'All protocols', and set the source and desctination to 0.0.0.0/0
- **Create a Mirror Session**
	- You will need to create 
	- Create a new traffic mirror Session
	- For the Mirror Source you will pick the network interface ('eni-') for the box whose traffic you want to send to Malcolm. If you have a box for Caldera spun up you can select the network interface for that. 
	- For the Mirror Target you will select the mirror target you created prevously. This should point to the network interface of the Ubuntu box you spun Malcolm up on. 
	- Pick '1' for the 'Session number'
	- Select the filter you created previouisly for the 'Filter' field. 
- Once you have have created a Mirror Target, Filter and Session, all the traffic defined in the filter that is passing in and out of what your selected as Source should be sent to Malcolm encapsulated in a VXLAN packet (port 4789)




 



# Techpet DevOps challenge part 2

## Jenkins-packer-deployment

### Deploying Jenkins to an EC2 Instance
When setting up an EC2 instance, you can add the script below to your user-data. This is to install Jenkins on the EC2 instance:

```bash
#!/bin/bash -x
echo "This script is written to work with Ubuntu image"
sleep 3
echo

echo "Updating and Upgrading image packages"
sudo apt update && sudo apt upgrade -y

sleep 5

echo "add the repository key to the system"
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sleep 5

echo "add the repository to the system"
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sleep 5

echo "update the repository"
sudo apt update
sleep 5

echo "Install Java"
sudo apt install openjdk-11-jre-headless -y
sleep 5

echo "install jenkins"
sudo apt install jenkins -y
sleep 5

echo "start jenkins"
sudo systemctl start jenkins
sleep 5

echo "enable jenkins"
sudo systemctl status jenkins
sleep 5

echo "Restart jenkins"
sudo systemctl restart jenkins
sleep 5

echo "done running"
```

- View your login password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
- Navigate to port 8080 of your EC2 instance's IP address. You should see a web page similar to the one below:

<img width="883" alt="image" src="https://user-images.githubusercontent.com/49791498/170310418-9c7c1cd4-0648-4817-bc93-3f094079a442.png">

**Note**: 
- On your security group, enable the following traffic:
-- TCP on port 8080
-- HTTPS 
-- SSH
- Restart your Jenkins server before navigating to the browser to access Jenkins.

### Bake Jenkins into an AMI
- Install Packer

```bash
$ brew tap hashicorp/tap
$ brew install hashicorp/tap/packer
```

- Run the following commands:
```bash
$ export AWS_ACCESS_KEY_ID="VALUE"           
$ export AWS_SECRET_ACCESS_KEY="VALUE"
$ packer build jenkins.json
Build 'amazon-ebs' finished after 5 minutes 32 seconds.

==> Wait completed after 5 minutes 32 seconds

==> Builds finished. The artifacts of successful builds are:
--> amazon-ebs: AMIs were created:
us-east-1: ami-00d85b32acb77178d
```

- After this, use the baked AMI to spin up an EC2 instance.
<img width="760" alt="image" src="https://user-images.githubusercontent.com/49791498/170307969-898c9bb1-760d-44bb-b786-aa37149b64de.png">

- SSH into your EC2 instance and navigate to your IP's port 8080. 
<img width="883" alt="image" src="https://user-images.githubusercontent.com/49791498/170310418-9c7c1cd4-0648-4817-bc93-3f094079a442.png">

**Note**: 
- Do not type in any script in the user-data section. You already have Jenkins installed on your AMI.
- When SSHing into your instance, log in as the value of ```"ssh_username"```.
```bash
ssh -i "key.pem" <value-of-ssh_username>@.......
```

### Troubleshooting guide
- [Jenkins not starting](https://stackoverflow.com/questions/8072700/how-to-restart-jenkins-manually)

### Resources
- [Install Jenkins](https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/).

- [Pipeline as Code](https://github.com/mlabouardy/pipeline-as-code-with-jenkins/blob/master/chapter4/standalone/template.json)

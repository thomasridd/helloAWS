# HelloAWS
In this tutorial we take the simple REST Api server  created in [davidcarboni/hellojava](https://github.com/davidcarboni/hellojava) and deploy it on an Amazon EC2 web server
> Requirements
> - Github account
> - AWS account

##1. Setup an AWS server

Simple stuff if you are an old hand but I got lost on this bit so lets go slow.

 Sign in to AWS and go to the [main console screen](https://eu-west-1.console.aws.amazon.com/console/home?region=eu-west-1#)

#####Virtual Private Cloud setup (VPC)

 1. Click **Create New VPC**
 2. Give it a name and the default CIDR block (10.0.0.0/16)
 3. Select your VPC and pick Actions->Edit DNS Hostnames
 4. Select Yes and Save
 5. Click Internet Gateways
 6. Click **Create Internet Gateway**
 7. Give it a name and Save
 8. Select and click Attach to VPC
 9. Select your VPC
 5. Click Subnets
 6. Click **Create New Subnet**
 7. Give it a name and the default CIDR block (10.0.0.0/24)
 8. Select your Subnet and pick Actions->Modify Auto-Assign public IP
 9. Check Yes and Save
 10. Select the **Route Table** tab
 11. Click the link to the current route table
 12. Add a link 0.0.0.0/0 to your Internet Gateway

So to catch up we've created a new virtual network, added a subnet that our server is going to sit on, and hooked it up to the internet.

##### EC2 Server setup

 2. Click EC2 to go to your virtual servers
 3. Click Launch Instance
 4. Click Select next to the Ubuntu Server setup
 5. Pick t2.micro as the Instance Type  
 12. Click Security
 13. Click Add rule - Fill in protocol HTTP, port 80, source 0.0.0.0
 6. Click Launch
 7. Select new keypair, think of a sensible name, and download

####3. Connect to AWS
[Connect to your instance](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-connect-to-instance-linux.html) is the next request from Chris. Again it is really simple stuff for people used to AWS. I had problems. 

This is the terminal command


    $  ssh -i /path/my-key-pair.pem ubuntu@public_dns_name

/path/my-key-pair.pem is your full local path to the keypair downloaded above

public_dns_name can be a problem

1. Go to your EC2 server list 
2. Select your server
3. The description tab at the bottom should list public_dns

If public dns is blank something dodge has happened with the VPC setup. Try [the troubleshooting page from AWS](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/TroubleshootingInstancesConnecting.html#TroubleshootingInstancesConnectionTimeout) or [this answer from Stack Overflow](http://stackoverflow.com/questions/20941704/ec2-instance-has-no-public-dns) or perhaps just begin again

##2. Install git, Java, and Maven
Sign into your AWS Ubuntu server and run the following commands to install the required software
### git

    $  sudo apt-get update
    $  sudo apt-get install git

### Java 8

    $  sudo add-apt-repository ppa:webupd8team/java
    $  sudo apt-get install oracle-java8-installer

Edit the file /etc/environment

    $  sudo nano /etc/environment

You need to add the line

    JAVA_HOME="/usr/lib/jvm/java-8-oracle"
 And load the environment file

    source /etc/environment
 
### Maven
    $  sudo apt-get install maven

##3. Get hellojava up and running
Now get our simple REST Api up and running on localhost 8080

    $  git clone https://github.com/davidcarboni/hellojava.git
    $  sudo apt-get install maven

##4. Setup nginx
At present we have our fresh REST Api running on port 8080. We are lastly going to setup an nginx webserver to link up port 8080 with the outside world

    $  sudo apt-get install nginx

Now we need to fix up the config

    $  sudo nano /etc/nginx/nginx.conf

And add this bit to the server section

    server {
        listen       80;
        server_name  _;
    
        location / {
                proxy_pass http://127.0.0.1:8080;
        }
    }
 
 Lastly run the start command
    
    $  sudo service nginx start

##Finally
Should be working now. Visit [your public DNS]/rest to see the result

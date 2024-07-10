<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Hosting WordPress on an EC2 Instance</title>
<style>
  body {
    font-family: Arial, sans-serif;
  }
  .content {
    max-width: 800px;
    margin: auto;
  }
  .image-container {
    position: relative;
    display: inline-block;
    margin: 20px 0;
  }
  .image-container img {
    width: 100%;
    height: auto;
    max-width: 600px;
    cursor: pointer;
    transition: 0.3s;
  }
  .image-container img:hover {
    opacity: 0.7;
  }
  .modal {
    display: none;
    position: fixed;
    z-index: 1000;
    padding-top: 100px;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgb(0, 0, 0);
    background-color: rgba(0, 0, 0, 0.9);
  }
  .modal-content {
    margin: auto;
    display: block;
    width: 80%;
    max-width: 700px;
  }
  .modal-content, .modal-caption {
    animation-name: zoom;
    animation-duration: 0.6s;
  }
  @keyframes zoom {
    from { transform: scale(0); }
    to { transform: scale(1); }
  }
  .modal-close {
    position: absolute;
    top: 15px;
    right: 35px;
    color: #fff;
    font-size: 40px;
    font-weight: bold;
    transition: 0.3s;
  }
  .modal-close:hover,
  .modal-close:focus {
    color: #bbb;
    text-decoration: none;
    cursor: pointer;
  }
</style>
</head>
<body>
<div class="content">
  # Hosting WordPress on an EC2 Instance 🚀

  ## Table of Contents 📚
  - [Introduction](#introduction)
      - [Prerequisites](#prerequisites)
      - [What is EC2?](#what-is-ec2)
      - [Step 1: Create and Configure the VPC](#step-1-create-and-configure-the-vpc)
      - [Step 2: Create a Subnet](#step-2-create-a-subnet)
      - [Step 3: Create an Internet Gateway](#step-3-create-an-internet-gateway)
      - [Step 4: Create a Route Table](#step-4-create-a-route-table)
      - [Step 5: Create EC2 Instance with Public IP](#step-5-create-ec2-instance-with-public-ip)
      - [Step 6: Configuring Security Groups during EC2 Setup](#step-6-configuring-security-groups-during-ec2-setup)
      - [Step 7: Connect to the EC2 Instance](#step-7-connect-to-the-ec2-instance)
      - [Step 8: Install Nginx](#step-8-install-nginx)
      - [Step 9: Install PHP MySQL](#step-9-install-php-mysql)
      - [Step 10: Install MariaDB 10.5 on Amazon Linux using the Default Repo](#step-10-install-mariadb-105-on-amazon-linux-using-the-default-repo)
      - [Step 11: Log in to MariaDB as the Root User and Create a Database and User for WordPress](#step-11-log-in-to-mariadb-as-the-root-user-and-create-a-database-and-user-for-wordpress)
      - [Step 12: Nginx Configuration for WordPress and Setting the EC2 Public IP Address](#step-12-nginx-configuration-for-wordpress-and-setting-the-ec2-public-ip-address)
      - [Step 13: Completed the WordPress Installation](#step-13-completed-the-wordpress-installation)
      - [Clean up](#clean-up)

  ## Introduction 🌟
  Hosting WordPress on AWS EC2 allows you to leverage the scalability, reliability, and security of Amazon Web Services. This guide will go through the key AWS concepts such as EC2, VPC, subnet, route table, and security group, and how they all interlink to create a secure and scalable environment for your application.

  ## Prerequisites ✅

  - AWS Account
  - Basic understanding of networking concepts
  - Familiarity with the AWS Management Console

  ## What is EC2? 🤔
  Amazon Elastic Compute Cloud (EC2) is a web service provided by Amazon Web Services (AWS) that allows users to rent virtual servers (referred to as "instances") on which they can run their applications. EC2 instances are essentially virtual machines that can be launched in the cloud and scaled up or down as needed.

  <div class="image-container">
    <img src="images/ec2logo.png" alt="EC2">
  </div>

  **Why do we need an EC2 instance?**

  One of the main reasons we need EC2 instances is because they offer **scalability and flexibility**. We can launch EC2 instances **on demand and scale up or down** as needed to match our workload requirements. This means we can easily provision resources when we need them and only pay for what we use.

  ## Step 1: Create and Configure the VPC 🌐

  <div class="image-container">
    <img src="images/vpc.PNG" alt="VPC">
  </div>

  *Figure 1: VPC - Created a VPC with a CIDR block of 10.0.0.0/16. CIDR blocks define a range of IP addresses. The VPC is your private section of the internet where you can run your servers securely.*

  > [!NOTE]
  > What does it do?
  > This creates a virtual network dedicated to my AWS account with an address range of 10.0.0.0/16

  ## Step 2: Create a Subnet 🌍

  <div class="image-container">
    <img src="images/subnet.PNG" alt="Subnet">
  </div>

  *Figure 2: Subnet - Created a subnet with a CIDR block of 10.0.0.0/24 within the VPC. This subnet can support up to 256 IP addresses, including network, broadcast, and host addresses.*

  > [!NOTE]
  > This creates a smaller network within my VPC (10.0.0.0/24), allowing up to 256 IP addresses.

  ## Step 3: Create an Internet Gateway 🌐

  <div class="image-container">
    <img src="images/itg.PNG" alt="Internet Gateway">
  </div>

  *Figure 3: IGW - Created an Internet Gateway and attached it to the VPC. An Internet Gateway allows your servers to access the internet and lets people from the internet access your servers.*

  > [!NOTE]
  > What does it do?
  > This creates and attaches an Internet Gateway to my VPC, enabling internet connectivity essential for web servers and other resources that need to communicate with the outside world.

  ## Step 4: Create a Route Table 📑

  <div class="image-container">
    <img src="images/rtb.png" alt="Route Table">
  </div>
  <div class="image-container">
    <img src="images/rtb-1.PNG" alt="Route Table">
  </div>

  *Figure 4: RTB - Created a route table for the VPC, then created a route to direct internet traffic to the Internet Gateway and associated the route table with the subnet. The route table determines how traffic is directed within the VPC.*

  > [!NOTE]
  > What does it do?
  > Associating the route table with the subnet ensures that traffic from the subnet is routed through the Internet Gateway, allowing instances in the subnet to access the internet.

  ## Step 5: Create EC2 Instance with Public IP 💻

  <div class="image-container">
    <img src="images/ec2.png" alt="EC2 Instance">
  </div>

  *Figure 5: EC2 Instance - Created an EC2 Instance with a Public IP to host WordPress and be accessible from anywhere. This allows remote management using SSH. The EC2 instance is the backbone of the web server setup.*

  > [!NOTE]
  > Acts as your virtual server, providing the necessary compute power, storage, and network connectivity to host and run your resources, such as the web application WordPress.

  ## Step 6: Configuring Security Groups during EC2 Setup 🔐

  <div class="image-container">
    <img src="images/sg.png" alt="Security Group">
  </div>

  *Figure 6: During the EC2 setup, created a security group, enabled auto-assign public IP, and configured inbound rules for port 22 for SSH access. Security groups act like virtual firewalls for your instance.*

  > [!NOTE]
  > What does it do?
  > Controls the incoming and outgoing traffic to and from your instance.

  ## Step 7: Connect to the EC2 Instance 🔌

  <div class="image-container">
    <img src="images/ssh.png" alt="SSH">
  </div>

  *Figure 7: Ran the chmod command to ensure my key is not publicly viewable, then ran the ssh command to securely access my server. The chmod 400 command sets the file permissions of the private key file (coderco-prod.pem) so that only the owner can read it.*

  > [!NOTE]
  > What does it do?
  > Uses SSH to open a remote connection to the EC2 instance.

  ## Step 8: Install Nginx 🌐

  <div class="image-container">
    <img src="images/nginx.png" alt="Nginx">
  </div>
  <div class="image-container">
    <img src="images/nginxservice.png" alt="Nginx Service">
  </div>
  <div class="image-container">
    <img src="images/nginxlistening.png" alt="Nginx Listening">
  </div>
  <div class="image-container">
    <img src="images/installednginx.png" alt="Installed Nginx">
  </div>

  *Figure 8: Installed Nginx web server, checked if it was installed, enabled the service, and ensured it was running and listening on port 80. When running curl localhost, I confirmed the Nginx web server was successfully installed. Nginx ensures efficient, secure, and scalable delivery of web applications (WordPress).*

  > [!NOTE]
  > - **Web Server:** Handles HTTP requests, delivering web content to visitors’ browsers.
  > - **Reverse Proxy:** Acts as an intermediary, improving load distribution and reducing server load.
  > - **Load Balancer:** Distributes incoming traffic across multiple servers, preventing any single server from becoming a bottleneck.

  ## Step 9: Install PHP MySQL 🛠️

  <div class="image-container">
    <img src="images/phpinstall.png" alt="PHP Install">
  </div>
  <div class="image-container">
    <img src="images/confirmphp.png" alt="Confirm PHP">
  </div>

  *Figure 9: Installed PHP and set up the PHP environment to handle PHP scripts and connect to the MariaDB database. Verified that it is installed as mysqli/mysqlnd listed in the output. Installing PHP MySQL is essential for connecting and communicating with MariaDB databases.*

  > [!NOTE]
  > PHP MySQL is a PHP extension that provides the functionality for PHP to communicate with MariaDB and MySQL databases.

  ## Step 10: Install MariaDB 10.5 on Amazon Linux using the Default Repo 🗄️

  <div class="image-container">
    <img src="images/searchamazonrepo.png" alt="Search Amazon Repo">
  </div>
  <div class="image-container">
    <img src="images/installmariadb.JPG" alt="Install MariaDB">
  </div>
  <div class="image-container">
    <img src="images/statusmariadb.png" alt="Status MariaDB">
  </div>

  *Figure 10: Installed MariaDB 10.5 (including server and client), started, enabled, and checked the status to ensure it was running. MariaDB is essential for storing, managing, and retrieving data, as WordPress relies on a database to store its content, user data, settings, etc.*

  > [!NOTE]
  > What does it do?
  > - **MariaDB:** Handles data storage and management.
  > - **PHP MySQL:** Enables PHP applications such as WordPress to communicate with MariaDB, sending queries and receiving data.

  ## Step 11: Log in to MariaDB as the Root User and Create a Database and User for WordPress 🗃️

  <div class="image-container">
    <img src="images/MariaDBWordpress.png" alt="MariaDB for WordPress">
  </div>
  <div class="image-container">
    <img src="images/downloadwordpress.png" alt="Download WordPress">
  </div>
  <div class="image-container">
    <img src="images/wordpressconfig.png" alt="WordPress Config">
  </div>
  <div class="image-container">
    <img src="images/databasesettings.png" alt="Database Settings">
  </div>
  <div class="image-container">
    <img src="images/wppermissions.png" alt="WordPress Permissions">
  </div>

  *Figure 11: Logged in to MariaDB as the root user, created a database and user for WordPress, navigated to the web root directory, downloaded the latest version of WordPress, extracted the downloaded tar.gz file, and moved the extracted WordPress files to the HTML directory. Configured WordPress by setting database settings in wp-config.php and changed the ownership of all files and directories to the user 'nginx' and group 'nginx'.*

  > [!NOTE]
  > What does it do?
  > **Database:** Organizes data, ensuring WordPress can manage its data separately from other applications.
  > **User:** Restricts what the WordPress user can do within the database, minimizing security risks.
  > **Database Settings:** WordPress uses wp-config.php to connect to the MariaDB database.
  > **Changing Ownership:** Ensures that the Nginx user owns the WordPress files, allowing the web server to manage them correctly.
  > **Set Permissions:** Ensures directories are executable and readable (755), and files are readable (644), a secure configuration.

  ## Step 12: Nginx Configuration for WordPress and Setting the EC2 Public IP Address

  <div class="image-container">
    <img src="images/nanowordpress.png" alt="Nginx Configuration for WordPress">
  </div>

  *Figure 12: Configured Nginx to serve my WordPress site using my EC2 instance's public IP address by editing /etc/nginx/conf.d/wordpress.conf. This configuration ensures Nginx listens for requests coming to my EC2 instance's public IP address.*

  > [!NOTE]
  > /etc/nginx/conf.d/wordpress.conf: Specifies the configuration settings for serving your WordPress site.

  ## Step 13: Completed the WordPress Installation

  <div class="image-container">
    <img src="images/wpsetup.png" alt="WordPress Setup">
  </div>
  <div class="image-container">
    <img src="images/wpsuccess.png" alt="WordPress Success">
  </div>
  <div class="image-container">
    <img src="images/wplogin.png" alt="WordPress Login">
  </div>
  <div class="image-container">
    <img src="images/wpwelcome.png" alt="WordPress Welcome">
  </div>

  *Figure 13: Opened my web browser, navigated to http://35.176.248.55, and followed the on-screen instructions to complete the WordPress installation by entering details such as the site title, admin username, password, and email address. Successfully installed MariaDB, Nginx, PHP, and configured WordPress on my EC2 instance.*

  ## Clean-up

  > [!CAUTION]
  > To avoid incurring unnecessary charges, terminate the EC2 instance and delete the VPC, ensuring that all associated resources (subnets, internet gateway, route tables, security groups) are also removed.
</div>

<!-- The Modal -->
<div id="myModal" class="modal">
  <span class="modal-close">&times;</span>
  <img class="modal-content" id="img01">
  <div id="caption"></div>
</div>

<script>
  var modal = document.getElementById('myModal');
  var images = document.querySelectorAll('.image-container img');
  var modalImg = document.getElementById('img01');
  var captionText = document.getElementById('caption');
  images.forEach(img => {
    img.onclick = function() {
      modal.style.display = 'block';
      modalImg.src = this.src;
      captionText.innerHTML = this.alt;
    }
  });
  var span = document.getElementsByClassName('modal-close')[0];
  span.onclick = function() { 
    modal.style.display = 'none';
  }
</script>
</body>
</html>

---
title: "Lab Guide"
weight: 60
---

### Prerequisites
* You will need to have an AWS account with sufficient priviliges to create resources in Amazon Lightsail to complete this lab.

* You should be familiar with using a command line.

* The ability to edit files with a Linux text editor is helpful, but not mandatory

### Create Lightsail Instance
In this first step we're going to create a new instance that you can run you're application on. An instance is just another name for a cloud-based virtual server. In this case you're going to deploy the instance using an existing configuration (known as a blueprint) that is based on the Ubuntu operating system, and includes Node.js already installed. 

* Navigate to the <a href="https://dashboard.eventengine.run/dashboard" target="_blank">Event Engine Dashboard</a> and input the hash found on the slip of paper you were handed when you arrived

* Sign into the <a href="https://console.aws.amazon.com/console/home." target="_blank">AWS console</a>

* In the ***Find Services*** box type ***Lightsail***

    ![](./images/find_services.png?classes=border)

* Click on ***Lightsail*** to be taken to the Amazon Lightsail home page

* If prompted choose a language and click ***Save***

* Click ***Let's Get Started*** to bring up the Lightsail instance creation page

* Ensure that the Instance Location is ***Oregon, Zone A (us-west-2a)***

    ![](./images/instance_location.png?classes=border)

    If the instance location is not correct, click ***Change AWS Region and Availability Zone*** and choose ***Oregon (us-west-2)***

* Scroll down and under ***Select a blueprint*** choose ***Node.js*** 

    ![](./images/node_js.png?classes=border)

* Scroll all the way to the bottom of the screen and click on ***Create Instance**

### Create IAM user***
The application needs a set of credentials in order to access the Rekognition service. In this section you will use the AWS Identity and Access Management service to create a set of credentials that the API server can use to communicate with Rekognition. It's always a best practice to ensure that any security credentials provide access only to the services that are actually being used, in this case the API server needs to to have full access to Rekognition, but only read access to S3. 

* Navigate back to the main AWS console. 

* In the ***Find Services*** box enter IAM

    ![](./images/iam.png?classes=border)

* Click on ***IAM***

* From the left hand menu click ***Users***

    ![](./images/users.png?classes=border)

* Click ***Add User** 

    ![](./images/add_user.png?classes=border)

* Enter *lightsail-workshop* for the ***User name*** and select ***Programmtic access** (this ensures that this user can only access allowed AWS services via the command line or API)

    ![](./images/user_details.png?classes=border)

* Click ***Next: Permissions*** at the bottom right of the screen

* Click ***Attach existing permissions directly***

    ![](./images/existing_policies.png?classes=border)

* In the ***Filter policies*** search box type *S3* and select *AmazonS3ReadOnlyAccess* from the list

    ![](./images/s3_search.png?classes=border)

* Move back to the search box and type *Rekognition* and select *AmazonRekognitionFullAccess*

    ![](./images/rekognition_search.png?classes=border)

* Click ***Next: Tags**

* Click ***Next: Review***

* Click ***Create user***

* Under ***Secret Access Key*** click ***Show***

    ![](./images/show_key.png?classes=border)

* You will need both the Access Key ID and the Secret Access Key later in this workshop. ***You should copy and paste both values into a text editor*** (be sure to make a note of which value is which) because if you accidentally close this screen you won't be able access them and will need to repeat the previous steps to create a new user. 

### Install and Configure MySQL
The Node.js blueprint doesn't include MySQL (<a href="https://www.mysql.com" target="_blank">learn more about MySQL</a>), so you will need to install it. After you install the MySQL server you'll configure it to start when the instance boots up, as well as creating the database and table that the application will use. 

To access your instance you're going to start a remote session using Lightsail's web-based SSH client <a href="https://lightsail.aws.amazon.com/ls/docs/en_us/articles/understanding-ssh-in-amazon-lightsail" target="_blank">(learn more about SSH)</a>. 

{{% notice tip %}}
For the rest of this lab when you're asked to copy and paste a command, the command should always be pasted into the SSH web-based client. 
{{% /notice %}}

* To start the SSH web-based client click the terminal icon on the card for your Lightsail instance. 

    ![](./images/ssh.png?classes=border)

{{% notice tip %}}
Your SSH web-based client will open in a new browser window
{{% /notice %}}

{{% notice tip %}}
In each of the steps below you will copy the command as it is written from this guide into the SSH web-based client
{{% /notice %}}

* Ensure that the Ubuntu operating system is up to date by running the update and upgrade commands using ```apt-get``` (<a href="https://help.ubuntu.com/community/AptGet/Howto" target="_blank">learn more about apt-get</a>).

        sudo apt-get update -y && sudo apt-get upgrade -y

* The next two commands tell `apt-get` to set the password on the database to *scores*. 

        echo "mysql-server mysql-server/root_password password scores" | sudo debconf-set-selections

* Copy and paste the second command to set the default password. 

        echo "mysql-server mysql-server/root_password_again password scores" | sudo debconf-set-selections

* Install MySql

        sudo apt-get install mysql-server -y

* Start MySQL to enable it to start whenever your instance starts up

        sudo systemctl enable mysql

* This command creates a new database (scores) and a table in that database (Scores). The table holds the players name and their score. 

        echo "CREATE DATABASE scores; USE scores; CREATE TABLE Scores (Name varchar(255), Score int);" | mysql -h localhost -u root -pscores

{{% notice tip %}}
You will get a warning that using the password on the command line can be inecure. Since this is a workshop it's ok to do here, but in production you want to be very cautious about using passwords on the command line. 
{{% /notice %}}

Your databse is now ready for your application

### Download and unpack the application code, intall dependencies
The code for the application is stored in a git bundle file which has been uploaded to an S3 bucket. In this section you're going download the code and upack it using git. They you'll use NPM to install all the necessary dependencies. 

* Use `curl` to download the code bundle.

        cd /home/bitnami && curl https://s3-us-west-2.amazonaws.com/lightsail-demo-public/example-apps/techtogether.bundle -O  

* Use `git` to unpack the bundle and copy the application code into place (<a href="https://git-scm.com" target="_blank">learn more about git</a>)
    
        git clone -b master techtogether.bundle

* Change into the newly created application directory
    
        cd /home/bitnami/techtogether 

* Use the Node Package Manager (`npm`) to install the application dependencies (<a href="https://www.npmjs.com" target="_blank">learn more about NPM</a>)
    
        npm install


### Edit the AWS configuration file 
In this section you will update the `aws-config.json` file to contain the credentials you created earlier in the workshop. You'll update the file by creating two environment variables, one containing the Access Key ID and the other the Secret Access Key. Then you'll use the `sed` command to do a search and replace in the file, updating it with your credentials. 

{{% notice tip %}}
You will need the credentials you created earlier for this next section. Be sure to add in your credentials when you copy and paste the commands below. 
{{% /notice %}}
    
* In the web-based SSH client enter the following using your Access Key value

        AWS_ACCESS_KEY_ID=your Access Key ID

    For example: `AWS_ACCESS_KEY_ID=ASIAWF5EXAMPLEUK3DX3`

* Enter the following command to ensure the value was set properly. The output should be your Access Key ID value (e.g. `ASIAWF5EXAMPLEUK3DX3`)

        echo $AWS_ACCESS_KEY_ID

* In the web-based SSH client enter the following using your Secret Access Key value

        AWS_SECRET_ACCESS_KEY=your Secret Access Key

    For example: `AWS_SECRET_ACCESS_KEY=iEQB4PbqTZTEXMPLEqjxvx0JbCEib4ml56y4`

* Enter the following command to ensure the value was set properly. The value should be your Secret Access Key (e.g. `iEQB4PbqTZTEXMPLEqjxvx0JbCEib4ml56y4`)

        echo $AWS_SECRET_ACCESS_KEY

* `sed` is a utility that can be used to manipulate string values. In this case the command below looks in the `aws-config.json` file for the strings `SECRET_ACCESS_KEY` & `ACCESS_KEY` and replaces it with the values of the secret access key and access key id environment variables you set in the previous steps

        sed -i \
        "s/SECRET_ACCESS_KEY/$(echo $AWS_SECRET_ACCESS_KEY)/; \
        s/ACCESS_KEY/$(echo $AWS_ACCESS_KEY_ID)/;" \
        aws-config.json 

* `cat` the configuration file to ensure that the values were set properly. 

        cat aws-config.json

    The output should be similar to this (do NOT copy and paste the value below)

        { "accessKeyId": "ASIAWF5EXAMPLEUK3DX3",
          "secretAccessKey": "iEQB4PbqTZTEXMPLEqjxvx0JbCEib4ml56y4",
          "region": "us-west-2" }



### Stop the built-in apache server
The Node.js blueprint has a built-in Apache web server. Unfortunately that server runs on port 80, which is the same port you want to run your web front end one. In this section you'll stop the web server, and the rename the configuration file so that it won't start back up if the server restarts

* Stop the Apache web server

        sudo /opt/bitnami/ctlscript.sh stop apache

* Rename the configuration file so that Apache doesn't automatically restart

        sudo mv /opt/bitnami/apache2/scripts/ctl.sh /opt/bitnami/apache2/scripts/ctl.sh.disabled

### Run and test the application
The next step is to actually start the application using `npm`

* Use `npm` to start the application 

        sudo npm run dev

When you see the following output, you can test your application

    [1] You can now view lightsail-rekognition-memory in the browser.
    [1] 
    [1]   Local:            http://localhost:80/
    [1]   On Your Network:  http://172.26.15.214:80/


To view the running application point your web browser at the IP address of your Lightsail instance. You can find the IP address on the card for your instance. 

* Move back to the Lightsail home page

* Copy the IP address of your Lightsail instance

    ![](./images/lightsail_ip.png?classes=border)

* Open a new browser window or tab and paste the Lightsail IP address into the address bar. The matching game should appear. 

### Extra Credit: Creating a DNS entry for your application ###
When you access web sites you usually enter a domain name such as https://www.amazon.com. Your web browser uses the Domain Name System (DNS) to convert the domain name into an IP address.

<a href = "https://robzhu.github.io/domain2LS/" target= "_blank">This tutorial</a> will show you how to map your instance's IP address to your domain.com domain name. 


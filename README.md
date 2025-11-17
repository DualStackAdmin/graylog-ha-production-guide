🚀 Graylog 3-Node HA Guide (Prod Environment, 32GB RAM)

This guide covers the process of setting up a Graylog cluster in High Availability (HA) mode on 3 servers. It assumes each server has 32GB of RAM and all three components (MongoDB, Data Node, Server) are running on the same machine.

This guide is:

🔢 Step-by-Step: Broken down into 5 logical stages.

🔒 Anonymized: All IPs/hostnames are placeholders (e.g., <NODE1_IP>).

🚀 Prod-Optimized: Includes memory configurations for a 32GB RAM co-located setup.

ℹ️ Server Information

The following server names and IP addresses will be used throughout the installation:

Server

Hostname

IP Address

Node 1

<NODE1_HOSTNAME>

<NODE1_IP>

Node 2

<NODE2_HOSTNAME>

<NODE2_IP>

Node 3

<NODE3_HOSTNAME>

<NODE3_IP>

⚙️ STAGE 1: BASIC SYSTEM PREPARATION

Status: Must be executed on all three servers (<NODE1_HOSTNAME>, <NODE2_HOSTNAME>, <NODE3_HOSTNAME>).

A. Update System and Install Required Packages

sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y gnupg curl net-tools


B. Set Timezone to UTC

sudo timedatectl set-timezone UTC


C. Edit the /etc/hosts File

On each server, open the /etc/hosts file and add the IP and name of all three servers:

sudo nano /etc/hosts


Add the following to the file:

<NODE1_IP> <NODE1_HOSTNAME>
<NODE2_IP> <NODE2_HOSTNAME>
<NODE3_IP> <NODE3_HOSTNAME>


D. Mounting the Dedicated Disk for Graylog Data (/dev/sdb)

This step assumes you have an additional disk at /dev/sdb.

Format the Disk and Create Mount Point:

sudo mkfs.ext4 /dev/sdb
sudo mkdir -p /var/lib/graylog-datanode


Add the disk to fstab for automatic mounting on reboot:

DISK_UUID=$(sudo blkid -s UUID -o value /dev/sdb)
echo "UUID=$DISK_UUID /var/lib/graylog-datanode ext4 defaults 0 2" | sudo tee -a /etc/fstab


Mount the Disk and Verify:

sudo mount -a
df -h /var/lib/graylog-datanode


E. Adjust Kernel Parameters

echo 'vm.max_map_count=262144' | sudo tee -a /etc/sysctl.d/99-graylog-datanode.conf
sudo sysctl --system


🗃️ STAGE 2: MONGODB CLUSTER SETUP (REPLICA SET)

A. Install MongoDB (On All Three Servers)

💡 Important Note: This guide uses commands for MongoDB 8.0. Always refer to the official Graylog documentation (and their MongoDB guides) for the latest versions and official installation instructions:
https://go2docs.graylog.org/current/downloading_and_installing_graylog/ubuntu_installation_multi_node.htm

Download and Add the GPG Key:

curl -fsSL [https://www.mongodb.org/static/pgp/server-8.0.asc](https://www.mongodb.org/static/pgp/server-8.0.asc) | sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor


Add the Repository List:

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] [https://repo.mongodb.org/apt/ubuntu](https://repo.mongodb.org/apt/ubuntu) jammy/mongodb-org/8.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list


Update Package List:

sudo apt-get update


Install and Hold Version:

sudo apt-get install -y mongodb-org
sudo apt-mark hold mongodb-org


B. MongoDB Configuration (On All Three Servers)

Edit the /etc/mongod.conf file:

sudo nano /etc/mongod.conf


Change/ensure the file content is as follows. The bindIp, storage (8GB RAM), and replication sections are crucial.

# /etc/mongod.conf file content

# Store data in /var/lib/mongodb
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true
  # PROD: Limit MongoDB to 8GB for 32GB RAM
  wiredTiger:
    engineConfig:
      cacheSizeGB: 8

# Log messages to /var/log/mongodb/mongod.log
systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log

# Listen on all interfaces
net:
  port: 27017
  bindIp: 0.0.0.0 # Use this instead of 127.0.0.1

# Enable replication
replication:
  replSetName: rs0

processManagement:
  timeZoneInfo: /usr/share/zoneinfo


Start the Service:

sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod.service


C. Initializing the Replica Set (On <NODE1_HOSTNAME> ONLY!)

Access the MongoDB Shell:

sudo mongosh


Initiate the replica set:

rs.initiate(
  {
    _id: "rs0",
    members: [
      { _id: 0, host: "<NODE1_HOSTNAME>:27017" },
      { _id: 1, host: "<NODE2_HOSTNAME>:27017" },
      { _id: 2, host: "<NODE3_HOSTNAME>:27017" }
    ]
  }
)


Check the status. After a few seconds, the <NODE1_HOSTNAME> server should become PRIMARY:

rs.status()


Exit the console: exit

💾 STAGE 3: GRAYLOG DATA NODE INSTALLATION

A. Installation (On All Three Servers)

💡 Important Note: This guide shows the Graylog 6.3 repository. Always check the official installation documents for the latest supported repository link:
https://go2docs.graylog.org/current/downloading_and_installing_graylog/ubuntu_installation_multi_node.htm

Add the Graylog Repository:

wget [https://packages.graylog2.org/repo/packages/graylog-6.3-repository_latest.deb](https://packages.graylog2.org/repo/packages/graylog-6.3-repository_latest.deb)
sudo dpkg -i graylog-6.3-repository_latest.deb
sudo apt-get update


Install the Data Node package:

sudo apt-get install -y graylog-datanode


Set Data Directory Permissions:

sudo chown -R graylog-datanode:graylog-datanode /var/lib/graylog-datanode


B. Prepare the Password Secret (On <NODE1_HOSTNAME> ONLY!)

Create the password_secret:

openssl rand -hex 32


Copy the result (e.g., 6588f6c4...). This is needed for Stage 3C and 4D.

C. Configuration (On All Three Servers)

Edit the /etc/graylog/datanode/datanode.conf file:

sudo nano /etc/graylog/datanode/datanode.conf


Add/modify the following parameters:

# Place the secret you created in Stage 3B here!
password_secret = YOUR_PASSWORD_SECRET_VALUE

# MongoDB HA connection
mongodb_uri = mongodb://<NODE1_HOSTNAME>:27017,<NODE2_HOSTNAME>:27017,<NODE3_HOSTNAME>:27017/graylog?replicaSet=rs0

# PROD: Memory allocation for 32GB RAM (12 GB)
opensearch_heap = 12g

# Path to your disk
opensearch_data_location = /var/lib/graylog-datanode/opensearch/data


D. Start the Data Node Service (On All Three Servers)

sudo systemctl daemon-reload
sudo systemctl enable graylog-datanode.service
sudo systemctl start graylog-datanode.service


🖥️ STAGE 4: GRAYLOG SERVER INSTALLATION

A. Installation (On All Three Servers)

sudo apt-get install -y graylog-server


B. Graylog Server Memory (Heap) Configuration (On All Three Servers)

For a production environment, we must also allocate stable memory for the Graylog Server itself.

Create/edit the /etc/default/graylog-server file:

sudo nano /etc/default/graylog-server


Add the following line to the file (PROD: 8GB heap for 32GB RAM):

# PROD: 8GB heap for 32GB RAM and G1GC optimization
GRAYLOG_SERVER_JAVA_OPTS="-Xms8g -Xmx8g -XX:+UseG1GC -XX:MaxGCPauseMillis=200 -Djava.net.preferIPv4Stack=true"


C. Create the Root Password (On <NODE1_HOSTNAME> ONLY!)

Choose a password for the admin web user (e.g., Hallex28) and create its SHA-256 hash:

echo -n "Hallex28" | sha256sum | cut -d" " -f1


Copy the result (e.g., 8d1770466...). This is needed for Stage 4D.

D. server.conf Configuration (Different Leadership on Each Server!)

Edit the /etc/graylog/server/server.conf file:

sudo nano /etc/graylog/server/server.conf


Parameters that MUST BE THE SAME on all three servers:

# The secret you created in Stage 3B
password_secret = YOUR_PASSWORD_SECRET_VALUE

# The admin password hash you created in Stage 4C
root_password_sha2 = YOUR_ROOT_PASSWORD_SHA2_HASH

# Web interface listen address
http_bind_address = 0.0.0.0:9000

# External web interface address (Always points to the leader)
http_external_uri = http://<NODE1_IP>:9000/

# MongoDB HA connection
mongodb_uri = mongodb://<NODE1_HOSTNAME>:27017,<NODE2_HOSTNAME>:27017,<NODE3_HOSTNAME>:27017/graylog?replicaSet=rs0

message_journal_max_size = 5gb
processor_wait_strategy = blocking


LEADERSHIP parameters (DIFFERENT on each server):

On <NODE1_HOSTNAME> ONLY:

is_leader = true


On <NODE2_HOSTNAME> and <NODE3_HOSTNAME>:

is_leader = false


E. Start the Server Service (On All Three Servers)

sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service


✅ STAGE 5: FINAL SETUP AND TEST

A. Accessing the Web Interface

Open the Leader Node's address in your browser: http://<NODE1_IP>:9000/

Initial Setup (On First Login):

You may see an "Initial Setup" page on your first login. This is related to the Data Node's internal security and is normal.

Click the Provision certificates for your data nodes button/link on the page.

Wait for the process to complete and then click Configuration finished. The servers may restart.

Login Page:

Username: admin

Password: The password you chose in Stage 4C (e.g., Hallex28)

B. Create an Input for Log Ingestion (Web UI)

In the Web UI, go to System > Inputs.

Select GELF UDP from the Select input list and click Launch new input.

Configure as follows:

Node: global (To run on all servers)

Title: Test GELF Input

Bind address: 0.0.0.0

Port: 5000

Click Save.

C. Sending a Test Log (From <NODE1_HOSTNAME>)

Install the ncat tool:

sudo apt install ncat -y


Send a test log to the cluster (to the leader's IP):

echo -n '{"version": "1.1","host":"test-client","short_message":"Graylog Cluster Test Log Successful!","level":1}' | ncat -u -w 1 <NODE1_IP> 5000


D. Verifying the Result (Web UI)

Go to the Search page in the Graylog Web UI. You should see your message: "Graylog Cluster Test Log Successful!"

🎉 Congratulations!

You have successfully installed a 3-Node HA Graylog Cluster.

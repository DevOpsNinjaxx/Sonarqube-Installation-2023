```bash
# Install OpenJDK 17 or JRE 17
sudo apt-get install openjdk-17-jdk -y

# or

sudo apt-get install openjdk-17-jre -y

# Install and Configure PostgreSQL
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" >> /etc/apt/sources.list.d/pgdg.list'
wget -q https://www.postgresql.org/media/keys/ACCC4CF8.asc -O - | sudo apt-key add -
sudo apt install postgresql postgresql-contrib -y
sudo systemctl enable postgresql
sudo systemctl start postgresql
sudo passwd postgres
su - postgres
createuser sonar

# Login to Localhost PostgreSQL
psql
ALTER USER sonar WITH ENCRYPTED password 'my_strong_password';
CREATE DATABASE sonar OWNER sonar;
GRANT ALL PRIVILEGES ON DATABASE sonar to sonar;

# Login to RDS PostgreSQL
sudo psql -h <rds-endpoint> -U postgres
CREATE DATABASE sonar;
CREATE USER sonar WITH PASSWORD 'password';
GRANT ALL PRIVILEGES ON DATABASE sonar TO sonar;
GRANT rds_superuser TO sonar;
ALTER USER sonar CREATEDB CREATEROLE;
ALTER USER sonar WITH ENCRYPTED password 'password';
ALTER DATABASE sonar OWNER TO sonar;

# Check Database Settings
\l

# Check Database Users
\du

# Download and Install SonarQube
sudo apt-get install zip -y
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-10.2.1.78527.zip
sudo unzip sonarqube-10.2.1.78527.zip
sudo mv sonarqube-10.2.1.78527 /opt/sonarqube

# Add SonarQube Group and User
sudo groupadd sonar
sudo useradd -d /opt/sonarqube -g sonar sonar
sudo chown sonar:sonar /opt/sonarqube -R

# Configure SonarQube
sudo vim /opt/sonarqube/conf/sonar.properties

# Uncomment these 2 lines
sonar.jdbc.username=sonar
sonar.jdbc.password=my_strong_password

# Add this line to use local PostgreSQL
sonar.jdbc.url=jdbc:postgresql://localhost:5432/sonar

# To use RDS PostgreSQL, uncomment this line under PostgreSQL
# sonar.jdbc.url=jdbc:postgresql://<rds-endpoint>/sonar

# Setup Systemd Service
sudo vim /etc/systemd/system/sonar.service

# Paste the following
# --------------------------------------------->
# [Unit]
# Description=SonarQube service
# After=syslog.target network.target
# [Service]
# Type=forking
# ExecStart=/opt/sonarqube/bin/linux-x86-64/sonar.sh start
# ExecStop=/opt/sonarqube/bin/linux-x86-64/sonar.sh stop
# User=sonar
# Group=sonar
# Restart=always
# LimitNOFILE=131072
# LimitNPROC=8192
# [Install]
# WantedBy=multi-user.target
# <----------------------------------------------

sudo systemctl enable sonar
sudo systemctl start sonar
sudo systemctl status sonar

# Modify Kernel System Limits
sudo vim /etc/sysctl.conf

# Paste the following lines at the bottom
# ------------------------------------------->
# vm.max_map_count=524288
# fs.file-max=131072
# ulimit -n 131072
# ulimit -u 8192
# <-------------------------------------------

sudo reboot

# Install Sonar Scanner Locally
sudo apt-get update
sudo apt-get install openjdk-17-jdk -y

# or

sudo apt-get install openjdk-17-jre -y

# Download Sonar Scanner
wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip
sudo mv sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner
echo 'export PATH=$PATH:/opt/sonar-scanner/bin' | sudo tee -a /etc/profile
source /etc/profile

# Configure Sonar Scanner Properties File
sudo vim /opt/sonar-scanner/conf/sonar-scanner.properties

# ----------------------------------------->
# sonar.host.url=http://<your-sonar-server-ip>:9000
# sonar.token=<your-token>
# <-----------------------------------------

# To generate a token, follow these steps:
# 1. Click on your user icon in the top right corner.
# 2. Select "My Account" from the dropdown menu.
# 3. In the left sidebar, click on "Security."
# 4. Under the "User Token" section, click on the "Generate" button.
# 5. Provide a name for your token (e.g., "SonarScanner").
# 6. Click the "Generate" button to create the token.
# 7. Copy the Token: Once the token is generated, you will be presented with a long alphanumeric token. Copy this token to your clipboard.
# 8. Use the Token in SonarScanner Properties: In your sonar-scanner.properties file, you can set the sonar.login property to the copied token.

# Create a Sonar-Project.properties in the project root directory
vim sonar-project.properties

# Add a basic properties file
# ---------------------------->
# sonar-project.properties
# must be unique in a given SonarQube instance
# sonar.projectKey=my:project
# --- optional properties ---
# defaults to project key
# sonar.projectName=My project
# defaults to 'not provided'
# sonar.projectVersion=1.0
# Path is relative to the sonar-project.properties file. Defaults to .
# sonar.sources=.
# Encoding of the source code. Default is default system encoding
# sonar.sourceEncoding=UTF-8
# <------------------------------

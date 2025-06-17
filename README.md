########### sudo apt update && sudo apt upgrade -y
sudo apt install openjdk-17-jdk -y
sudo apt install maven -y
sudo apt install mysql-server -y

# 3-tier-app

#### Clone the repo git clone https://github.com/arjunkoppineni/3-tier-app.git
#### cd 3-tier-app
#### configure application.properties. keep your RDS Endpoint and databse name.
#### cd app-tier/pom.xml
####  mvn clean package
#### java -jar target/*.jar

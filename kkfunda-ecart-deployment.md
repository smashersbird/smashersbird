Spring Boot Shopping Cart Deployment using Docker
Step 1: Launch an Ubuntu EC2 Instance

Verify your server.

cat /etc/os-release

or

lsb_release -a
Step 2: Update Ubuntu
sudo apt update
sudo apt upgrade -y
Step 3: Install Git
sudo apt install git -y

Verify

git --version
Step 4: Install Maven
sudo apt install maven -y

Verify

mvn -v

If Java is not installed, install JDK.

Step 5: Install Java JDK

You finally used Java 17.

sudo apt install openjdk-17-jdk -y

Verify

java -version
javac -version
jar --version

All three commands should work.

Step 6: Set Java 17 as Default

Check available Java versions

sudo update-alternatives --config java

Choose Java 17.

Similarly

sudo update-alternatives --config javac

Choose Java 17.

Verify

java -version
javac -version

Both should display Java 17.

Step 7: Install Docker
sudo apt install docker.io -y

Start Docker

sudo systemctl start docker

Enable Docker

sudo systemctl enable docker

Verify

docker --version
Step 8: Give Docker Permission

Add current user

sudo usermod -aG docker ubuntu

Refresh group

newgrp docker

Verify

groups

Output should contain

docker
Step 9: Clone GitHub Repository
git clone <github-url>

Example

git clone https://github.com/xxxxx/shopping-cart.git

Move into project

cd shopping-cart
Step 10: Check Project Structure
ls

Expected

docker
pom.xml
src
README.md
Step 11: Build Maven Project
mvn clean package

This performs

Downloads dependencies
Compiles Java source code
Runs tests
Creates executable JAR
If Maven Build Fails

We faced this issue.

Invalid CEN header

Reason:

Corrupted Maven dependency.

Fix

Delete corrupted dependency

rm -rf ~/.m2/repository/org/aspectj/aspectjweaver

Download again

mvn dependency:get -Dartifact=org.aspectj:aspectjweaver:1.8.10

Run build again

mvn clean package
Step 12: Verify JAR
ls target

Expected

shopping-cart-0.0.1-SNAPSHOT.jar
Step 13: Verify Dockerfile

Your Dockerfile

FROM openjdk:8u151-jdk-alpine3.7

EXPOSE 8070

ENV APP_HOME /usr/src/app

COPY target/shopping-cart-0.0.1-SNAPSHOT.jar $APP_HOME/app.jar

WORKDIR $APP_HOME

ENTRYPOINT exec java -jar app.jar

Notice

shopping-cart-0.0.1-SNAPSHOT.jar

is copied as

app.jar

inside the image.

Step 14: Build Docker Image

Run from project root

docker build -t tmalli834/image:v1 -f docker/Dockerfile .

Verify

docker images

Example

REPOSITORY          TAG

tmalli834/image     v1
openjdk             8u151-jdk-alpine3.7
Step 15: Run Container
docker run -d --name con1 -p 8070:8070 tmalli834/image:v1
Step 16: Verify Container
docker ps

Example

CONTAINER ID

con1

Up
Step 17: Check Logs
docker logs con1

Expected

Tomcat started on port(s): 8070
Started ShoppingCartApplication
Step 18: Verify JAR inside Container
docker exec -it con1 ls -l /usr/src/app

Output

app.jar

Notice

Docker renamed

shopping-cart-0.0.1-SNAPSHOT.jar

to

app.jar
Step 19: Access Application

Browser

http://<EC2-Public-IP>:8070/home

Example

http://54.xx.xx.xx:8070/home
Step 20: Login

Admin

Username : admin

Password : admin

User

Username : user

Password : password
Step 21: Push Image to Docker Hub (Optional)

Login

docker login

Push

docker push tmalli834/image:v1
Step 22: Stop Container
docker stop con1
Step 23: Remove Container
docker rm con1
Step 24: Remove Image
docker rmi tmalli834/image:v1
Summary of the Commands You Used
sudo apt update
sudo apt install git -y
sudo apt install maven -y
sudo apt install openjdk-17-jdk -y
sudo apt install docker.io -y

git clone <repo-url>

cd shopping-cart

mvn clean package

docker build -t tmalli834/image:v1 -f docker/Dockerfile .

docker run -d --name con1 -p 8070:8070 tmalli834/image:v1

docker ps

docker logs con1

docker exec -it con1 ls -l /usr/src/app

http://<EC2-Public-IP>:8070/home

This is the same workflow followed in many real-world CI/CD pipelines (Jenkins → Maven → Docker → Docker Hub → Deployment), making it a good reference for interviews and future projects.

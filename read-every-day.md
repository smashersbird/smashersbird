Absolutely. Here's the complete end-to-end workflow we followed to run the Spring Boot Shopping Cart project in Docker. This is also the same workflow you'll use in many real-world DevOps projects.

Spring Boot Project Deployment using Docker (Step-by-Step)
Step 1: Launch an Ubuntu EC2 Instance

We created an Ubuntu server on AWS.

Verify the OS:

lsb_release -a

Output:

Ubuntu 26.04 LTS
Step 2: Install Git
sudo apt update
sudo apt install git -y

Verify:

git --version

Purpose:

To clone the project from GitHub.
Step 3: Install Maven
sudo apt install maven -y

Verify:

mvn -v

Initially, the output showed:

Java Runtime (JRE)

But there was no Java compiler (javac), which caused build issues.

Step 4: Install Java JDK

We discovered:

javac -version

Output:

Command 'javac' not found

So we installed the JDK:

sudo apt install openjdk-17-jdk -y

Verify:

java -version
javac -version
jar --version

Expected:

java 17
javac 17
jar 17

Why?

java → Runs applications.
javac → Compiles Java source code.
jar → Packages compiled classes into a JAR file.
Step 5: Set Java 17 as Default

Initially Maven was using Java 25.

Check:

mvn -v

Switch to Java 17:

sudo update-alternatives --config java
sudo update-alternatives --config javac

Verify:

mvn -v

Now Maven should show Java 17.

Step 6: Clone the GitHub Project
git clone <github-repository-url>

Go into the project:

cd kkfunda-ekart

Verify:

ls

Output:

docker
pom.xml
src
README.md
Step 7: Build the Project

Run:

mvn clean package

Purpose:

Downloads dependencies.
Compiles the Java code.
Runs tests.
Creates the executable JAR.
Step 8: Fix the Build Error

We encountered:

Invalid CEN header
aspectjweaver-1.8.10.jar

Cause:

A corrupted Maven dependency in the local repository.

Fix:

rm -rf ~/.m2/repository/org/aspectj/aspectjweaver

Download it again:

mvn dependency:get -Dartifact=org.aspectj:aspectjweaver:1.8.10

Then rebuild:

mvn clean package

Now the build succeeded.

Step 9: Verify the JAR File

Check:

ls target

Expected:

shopping-cart-0.0.1-SNAPSHOT.jar

This JAR contains the compiled Spring Boot application.

Step 10: Understand the Dockerfile

Dockerfile:

FROM openjdk:8u151-jdk-alpine3.7

EXPOSE 8070

ENV APP_HOME=/usr/src/app

COPY target/shopping-cart-0.0.1-SNAPSHOT.jar $APP_HOME/app.jar

WORKDIR $APP_HOME

ENTRYPOINT exec java -jar app.jar

Explanation:

FROM → Base image with Java installed.
EXPOSE → Documents port 8070.
ENV → Defines environment variables.
COPY → Copies the JAR into the image.
WORKDIR → Sets the working directory.
ENTRYPOINT → Starts the application when the container runs.
Step 11: Build the Docker Image

Initially we ran:

docker build -t tmalli834/image:v1 .

Error:

Dockerfile not found

Reason:

The Dockerfile was inside the docker directory.

Correct command:

docker build -t tmalli834/image:v1 -f docker/Dockerfile .
Step 12: Fix Docker Permission Error

We got:

permission denied while trying to connect to docker.sock

Reason:

The ubuntu user wasn't using the docker group in the current session.

Fix:

sudo usermod -aG docker ubuntu
newgrp docker

Verify:

groups

You should see:

docker
Step 13: Verify the Image
docker images

Output:

openjdk:8u151-jdk-alpine3.7
tmalli834/image:v1

Why two images?

openjdk → Base image.
tmalli834/image:v1 → Your application image built on top of the base image.
Step 14: Run the Container

Initially we ran:

docker run --name con1 tmalli834/image:v1

The application started, but it wasn't accessible from outside.

Step 15: Check Running Containers
docker ps

Output:

8070/tcp

This means the port wasn't published to the host.

Step 16: Stop and Remove the Container
docker stop con1
docker rm con1
Step 17: Run with Port Mapping
docker run -d \
  --name con1 \
  -p 8070:8070 \
  tmalli834/image:v1

Now docker ps should show:

0.0.0.0:8070->8070/tcp

This maps:

Host port: 8070
Container port: 8070
Step 18: Check Logs
docker logs con1

Expected:

Tomcat started on port(s): 8070
Started ShoppingCartApplication

This confirms the application started successfully.

Step 19: Open the Application

In your browser:

http://<EC2-PUBLIC-IP>:8070/home

For example:

http://13.232.205.139:8070/home

Make sure your AWS Security Group allows inbound traffic on port 8070.

Step 20: Log In

Use the credentials from the project's README:

Admin:

Username: admin
Password: admin

User:

Username: user
Password: password
Overall Flow
Launch EC2
      │
      ▼
Install Git
      │
      ▼
Install Maven
      │
      ▼
Install JDK
      │
      ▼
Clone GitHub Project
      │
      ▼
mvn clean package
      │
      ▼
Creates shopping-cart.jar
      │
      ▼
docker build
      │
      ▼
Creates Docker Image
      │
      ▼
docker run -p 8070:8070
      │
      ▼
Container Starts
      │
      ▼
Tomcat Starts
      │
      ▼
Application Accessible in Browser

This is a complete CI/CD-style deployment flow and is very similar to what you'll do in real DevOps environments, except that in production these steps are typically automated by tools like Jenkins, Docker, and deployment pipelines.

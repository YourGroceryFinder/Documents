# Docker Research

## What is docker?
Docker is a tool that helps developers package their applications and all their required parts into bundles called containers. 
These containers make it easy to move applications between different computers or cloud platforms. With Docker, developers can ensure that their applications run the same way on any system, saving time and allot of struggle when sharing and managing software. 
It's like having a portable box that contains everything your application needs to work, making it simple to ship and run software in a consistent and reliable manner.

## Dockerfile
The Dockerfile is a text file containing the instructions to build a Docker image.  It contains things like: the base image, the files to copy and where to place them, what directory to use and what command to run. The file below is my current docker file:
```dockerfile
# Use the OpenJDK 17 image as the base image
FROM openjdk:17
# Copy the .jar file from the target and place it paste it into /app/
COPY target/back-end-0.0.1-SNAPSHOT.jar /app/
# Set the working directory for the image to /app
WORKDIR /app
# Specify the command to run when a container is created from the Docker image.
CMD ["java", "-jar", "back-end-0.0.1-SNAPSHOT.jar"]
```
The image that is going to be created needs a base. There are many different bases to use but since I use Java 17 in my project I decided using openjdk:17 was the best option. The jar file that I want to copy to the /app/ repository is the jar file in the target directory of the build context that maven creates. Next I want to change the working directory to /app. This means that the commands executed after this will be executed in this directory. Finally we want to define what command will run when you run a container. “Java” is responsible for executing the java application, the “-jar” is an option used to specify that the following argument is an executable JAR file and “back-end-0.0.1-SNAPSHOT.jar” is the name of said jar file.

## CD Workflow
To automatically build a new image every time you push to the master branch you need to create a work flow. The workflow below is the one I use in my CI/CD Pipeline.
```yaml
name: create image
on:
  push:
    branches:
      - "main"
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: 17
          distribution: 'adopt'
      - name: Install Maven
        run: sudo apt-get install maven -y
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Build JAR
        run: mvn clean install
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/ygc:backend
```
In this workflow there are a couple of steps. You start with naming it and configuring when you want the workflow to run. I have named it “Create image” and it runs when something is pushed to the main branch. This happens here:
```yaml
name: create image
on:
  push:
    branches:
      - "main"
```
After doing that we need to define what type of machine or environment the job will run on. I have chosen to use ubuntu-latest, it is defined in this part:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```
I chose ubuntu because when I tested a different options like windows it was clear that ubuntu was allot faster and I chose latest so that it keeps picking the latest version every time automatically.

Next are the steps the workflow goes through. The first step is the checkout. What this does is it checks for the latest version of your repository (including all branches, commits and files) and makes it usable (Selects it) for the next steps. This is done here:
```yaml
      - name: Checkout
        uses: actions/checkout@v3
```
After the checkout has completed you want to set up your environment that provides the necessary tools and libraries to run the application you have made. Since I made a Java application I will be using JDK 17:
```yaml
      - name: Set up JDK 17
        uses: actions/setup-java@v2
        with:
          java-version: 17
          distribution: 'adopt'

```
The next step is to install Apache maven. By doing this you make sure that maven is installed on the environment where the workflow is executed. It is needed since it’s a build tool for java. You can install it by doing this:
```yaml
      - name: Install Maven
        run: sudo apt-get install maven -y
```
Now comes the authentication for Docker Hub. To log into Docker Hub in the workflow you need 2 things. The Docker Hub username and the Docker Hub Access Token. The username is your normal username you use in docker. The token can be found in Account settings -> Security -> Access Tokens.  You can put these in the workflow directly but since it is crucial information for accessing the repository it is smarter to save them as secrets in GitHub. To Create these secrets you need to go to Your repostitory in github -> Settings -> Secrets and Variables -> Actions. Here you can create the repository secrets. These secrets can then be used like this: 
```yaml
      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
```
 And these are the secrets in github:
 ![](https://cdn.discordapp.com/attachments/1069302114620276918/1117952334140551228/Afbeelding2.png)
 
 The file that you want to use to build your docker image is the .jar file in your project. JAR stands for Java ARchive. It’s a file format based on the popular ZIP file format and is used for collecting many files into one. In maven you can run: 
 ```yaml
       - name: Build JAR
        run: mvn clean install
 ```
 This command builds the projects, runs the tests and packages the application into a new .jar file. We do this in our workflow so we don’t have to keep doing it manually every time before we push.
The final step is Building the Docker image and pushing it to a Docker registry. You do that by first defining what action to use. Then selecting the context and the file, indicate wether you want the Docker image to be pusched to a Docker Registry and the specify the tags. This is done by doing this:
```yaml
      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/ygc:backend
```
The action we defined is the docker/build-push-action. It is used for building and pushing the docker image. Then we select the context. The context is the path to the directory containing the files needed for building the Docker file. So I put in . meaning the current directory, which indicates that all of the content in the repository is used as the build context. Next we need to select the Dockerfile. Since the dockerfile is in the current directory we can select it by using “./Dockerfile”. Since we want to push our image to the registry we set push to “true”. Then finally come the tags. This field specifies the tags to assign to the docker image. It’s a way to label and identify the image. Since I have multiple projects I decided to name this one like this, 494690/ygc:backend. The part after the : mean the version of the current image.

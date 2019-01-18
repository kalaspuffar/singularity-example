# singularity-example
Simple example showing of the features of singularity

## Interactive image creation.

First of I want to show of a way to create an image interactivilly. This could be a good way to start to get all your dependencies in order before you write your container file.

To start this process you create an image inside of an a directory on your machine. Then you start a session as writable against that directory in order to work with the data.
```
sudo singularity build --sandbox ubuntu/ library://ubuntu
sudo singularity exec --writable ubuntu /bin/bash
```

### Inside of the container

First we need to upgrade the distribution and then install the dependencies required for our maven build.
```
apt upgrade
apt install maven openjdk-11-jdk
```

Next up we move over to the ```/opt``` directory. And it's important not to put your files inside of a home directory or in root. These files will be mounted from the host system. So if you put something inside of your ```/root``` directory in the image it will be created in the ```/root``` of the host system.

Below we go to ```/opt``` and create a new quickstart archetype.
```
cd /opt
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DarchetypeVersion=1.4 \
    -DgroupId=org.ea \
    -DartifactId=basic \
    -Dversion=0.0.1-SNAPSHOT \
    -DpackageName=org.ea
```

We enter the newly created hello world application and package it with maven.
```
cd /opt/basic
mvn package
```

Next up you need to press ```CTRL-D``` on your keyboard to exit the guest system out to your host system.

Then we will bake the directory into a sif file which is an singularity image file.
```
sudo singularity build work.sif ubuntu
```

Next up we will run our new image with a command that will run the hello world application inside of our image.
```
singularity exec work.sif java -cp /opt/basic/target/basic-0.0.1-SNAPSHOT.jar org.ea.App
```

## Building container from definition.

In this example we will bake a already prepared definition file for singularity into an image.
```
sudo singularity build singularity-java.sif singularity-java.def
```

Next up we will run the image in order to run the hello world application baked inside of the image.
```
singularity run singularity-java.sif
```


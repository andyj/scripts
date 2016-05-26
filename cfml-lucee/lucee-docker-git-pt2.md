Getting Lucee, Docker and your local git repos working together.

This is part 2, if you're just getting in to Docker please read pt1 first. This post will follow on from that setup to give you a working (for local dev) container.

To recap, our server is working with code loaded in to the container during the **docker build** but does this mean that when we make a change we have to re-build, re-run and reload the browser to see our changes.

###### Setting up your local environment

Luckily the answer is no, what I've been doing is mounting a local folder (a git repos) to the Docker container. This means that I can make changes, run pull/push/branch etc all on the HOST machine and just refresh the browser to see your changes. All we need to do is add the **-v** option pointing to our repos

``
$ docker run -p 80:80 -v ~/Projects/exmaple.com/www/:/var/www lucee-server
``

You can now make changes to your project in **~/Projects/exmaple.com/www/** 

This all works fine until you call need a datasource or regional settings. When you call **run** you effectively have a new box waiting for you. To get around this you will want to load your own Lucee config settings during the build. This is quite easy as they are all stored in **lucee-server.xml** (for the web settings it slightly different but the process is roughly the same) and this file just need placing in the Docker when `$ docker build` is called

##### Setting up Lucee

To load our own **lucee-server.xml** in to the build we'll need to edit the **Dockerfile** and tell it to delete whats there and pull our file in just like we did with the nginx.conf files. To figure this out I went back to the git repos for the Lucee Docker to check out [the Dockerfile](http://https://github.com/lucee/lucee-dockerfiles/blob/8b6e8eda915c2e87421bef8ff6395dedaa12bfac/4.5/Dockerfile#L25) in the Tomcat (JAR deployment) container. [Line 25](http://https://github.com/lucee/lucee-dockerfiles/blob/8b6e8eda915c2e87421bef8ff6395dedaa12bfac/4.5/Dockerfile#L25) told me where the **lucee-server.xml** is stored so all I needed to do was to overwrite this in my **Dockerfile** in the **myFirstLuceeDocker** folder. 


1) Grab your **lucee-server.xml** and put it in the **myFirstLuceeDocker** folder

2) Edit the Dockerfile to delete the current xml file and use your own one

```
$ sudo nano Dockerfile
```

add the following before the `# Expose HTTP and HTTPS ports`

```
#Remove the default lucee-server.xml 
RUN rm -rf /opt/lucee/server/lucee-server/context/lucee-server.xml
#Copy your xml in to the build
COPY lucee-server.xml /opt/lucee/server/lucee-server/context/lucee-server.xml
```

3) Build and run

```
$ docker build -t lucee-server-custom .
$ docker run -p 80:80 -v ~/Projects/exmaple.com/www/:/var/www lucee-server-custom
```
4) Browse to [http://localhost](http://localhost) to see your site working.

##### Telling your Lucee Docker how to talk to MySQL on the host machine
Step 4 actually failed hard for me. Though the datasouce was now setup the container couldn't see my dev instance of MySQL running on the host. 

If you need your container talking to another machine you'll need to add a host during the run command.

```
#Find out your IP address
$ ifconfig | grep "inet "

#Add the --add-host option to our build 
$ docker run -p 80:80 -v ~/Projects/mytagio:/var/www --add-host=docker:192.168.0.12 lucee-server-custom

```



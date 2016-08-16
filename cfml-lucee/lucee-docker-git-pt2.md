This is part 2 and following on from [this post](http://www.andyjarrett.co.uk/2016/05/26/getting-lucee-and-docker-working-together/), if you're just getting in to Docker please read pt1 first. This post will follow on from that setup to give you a working (for local dev) container.

*I am using a Beta version of Docker to run natively on my Mac. References to localhost might need to change to the IP 192.168.99.101 if you are using something like Boot2Docker*

#### On to the post


To recap, our server is working with code loaded in to the container during the **docker build** but does this mean that when we make a change we have to re-build and re-run the container as well as reloading the browser just to see our changes.

###### Setting up your local environment

For my local dev I've been mounting a local folder (a git repos) to the Docker container as this means that I can make changes, run pull/push/branch etc all on the HOST machine and just refresh the browser to see the changes. All we need to do is add the **-v** option to share our hosts filesystems 

``
$ docker run -p 80:80 -v ~/Projects/example.com/www/:/var/www lucee-server
``

You can now make changes to your project in **~/Projects/example.com/www/** without the need to re-build your container.

##### Setting up Lucee

At this point you have a fresh install of Lucee, but what you really need is all your lucee-server settings migrated over as well. To achieve this we'll load our own Lucee config  during the build. This actually quite easy as they are  stored in **lucee-server.xml** (for the lucee-web settings it slightly different but the process is roughly the same) and this file just need placing in the Docker when `$ docker build` is called


To load our own **lucee-server.xml** in to the build we'll need to edit the **Dockerfile** and tell it to delete whats there and pull our file in just like we did with the nginx.conf files. To figure this out I went back to the git repos for the Lucee Docker to check out [the Dockerfile](http://https://github.com/lucee/lucee-dockerfiles/blob/8b6e8eda915c2e87421bef8ff6395dedaa12bfac/4.5/Dockerfile#L25) in the Tomcat (JAR deployment) container. [Line 25](https://github.com/lucee/lucee-dockerfiles/blob/8b6e8eda915c2e87421bef8ff6395dedaa12bfac/4.5/Dockerfile#L25) told me where the **lucee-server.xml** is stored so all I needed to do was to overwrite this in my **Dockerfile** which is in the **myFirstLuceeDocker** folder. 


1) Grab your own **lucee-server.xml** and put it in the **myFirstLuceeDocker** folder

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
$ docker run -p 80:80 -v ~/Projects/example.com/www/:/var/www lucee-server-custom
```
4) Browse to [http://localhost](http://localhost) to see your site working.

At this point you have your docker running from your own code (on the host machine) and using a current Lucee config. You can now take this container to any machine you want to develop on and get your environment up-and-running relatively quickly.


##### Fork me!
I'm still playing around with Docker but if i've done something wrong above or know a better way then forke copis of this post which I have put in my [scripts](https://github.com/andyj/scripts) project on Github.

* [https://github.com/andyj/scripts/blob/master/cfml-lucee/lucee-docker-git-pt1.md](https://github.com/andyj/scripts/blob/master/cfml-lucee/lucee-docker-git-pt1.md)
* [https://github.com/andyj/scripts/blob/master/cfml-lucee/lucee-docker-git-pt2.md](https://github.com/andyj/scripts/blob/master/cfml-lucee/lucee-docker-git-pt2.md)

If you have changes then please send pull-requests!


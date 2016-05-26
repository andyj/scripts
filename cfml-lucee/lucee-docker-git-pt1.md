As of writing this [Lucee v5](http://lucee.org/blog/welcome-and-lucee-5.html) is out. This post will be using v4.5 as that was the version I was converting my EC2 instance from. Also now that I've finished this post I can see it is going to be a 2 parter. In the next blog I'll explain how to get the container working with your own code  and how to customise the Lucee install with your own config settings.

Lets get going.

###### Clone the repo
```
$ git clone git@github.com:lucee/lucee-dockerfiles.git
$ cd lucee-dockerfiles/
```

###### Copy the nginx folder for your first container
I'm goin to be using the Lucee with Tomcat/NGINX (lucee/lucee4-nginx) container which gives us a stable release of Lucee 4.5+ on Tomcat 8 JRE8 combined with an integrated NGINX web server (it's the complete package). 

Once we have cloned the app we need to take of copy of the Docker build folder. This is the folder we'll be using to fire up and then later modify.

```
$ cp -R lucee-nginx/4.5/ myFirstLuceeDocker
$ cd myFirstLuceeDocker/
```

###### Lets build
Thats it essentially, at this point we can just build and run this via

```
$ docker build -t lucee-server .
$ docker run lucee-server
```

You can confirm its fired up by opening up a new terminal window and running

```
$ docker ps -l
```

and you should see something like this which gives you the container ID, Name and it also shows you what ports are exposed. 

![docker ps -l example](http://www.andyjarrett.co.uk/content/images/2016/05/Screenshot-2016-05-26-15-50-08.png)

The problem is you cannot connect to this box. To do that you need to tell your machine to connect to the container and you do that be defining which port to connect on. 

Before we continue we need to use either the name or ID that we just got to kill the Docker instance.

```
docker kill nostalgic_shockley
```

I'm going to map my machines (the host) port 80 to the containers port 80 with

```
$ docker run -p 80:80 lucee-server
```

You should now be able to go to [http://localhost](http://localhost) and see the test page. If your host machines port 80 is in use you can change the command to something like 

```
$ docker run -p l337:80 lucee-server
``` 

and browse to [http://localhost:1337/](http://localhost:1337/)



To kill this container we need to get the name or ID again, so run the following to get the information

```
$ docker ps -l
$ docker kill dd1f3e7aef76 #Use your ID from the command above
```

###### Configure the build (Nginx)

One problem here is that you won't be able to connect to the Lucee Admin as its blocked by default. To get around this (for development) we're going to change the **nginx.conf** file to allow us access.

```
$ sudo nano default.conf 
```

Change

```
location ^~ /lucee {
  deny  all;
}
```

to

```
#location ^~ /lucee {
  #deny  all;
#}
```

If you're paranoid you can change the above to allow your IP address just by following the [Nginx guide](https://www.nginx.com/resources/admin-guide/restricting-access/). Once you have made your change its time to rebuild and re-run the container:

```
$ docker build -t lucee-server .
$ docker run -p 80:80 lucee-server
```

To check in the nginx.conf change has worked go to  [http://localhost/lucee/admin/server.cfm](http://localhost/lucee/admin/server.cfm)


You could of changed the build name from **lucee-server** to **lucee-server-v2** if you are playing around with different setups or need mulitple instances of Lucee running on your machine. 

Hopefully this is enough to get you going. Even if you don't use it on your live server its a great tool to run locally without have everything installed on your machine. 

Don't forget to check out the main Git repo at [https://github.com/lucee/lucee-dockerfiles](https://github.com/lucee/lucee-dockerfiles) for more info.


And if you haven't already you can join in at the [CFML Slack ](http://cfml-slack.herokuapp.com/) where they also have a #docker and #lucee channel!

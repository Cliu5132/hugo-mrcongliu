---
title: "Docker Intro"
date: 2023-04-24
---

On my first day of work, my supervisor asked me to pull in our repository and run `docker-compose up` inside the root folder. After that, everything was ready, including the front-end, back-end, database, and caching. Although I was terrified by the logs thrown all over the screen, I was quite curious about what Docker was. This curiosity continued for over a year until I was thrown into a DevOps role to do a maintanance on our Lambda services.

If I could go back in time and talk to the my younger self, I would say that I should start learning Docker as early as possible. It is so neat and will help me throughout my career as a software developer. In terms of learning resources, the Docker official introduction is the best place to start.

Here are a few steps to grasp the basics of Docker.

### Download Docker Desktop

Go to [Docker's website](https://www.docker.com/) and download Docker Desktop.

There are a few versions out there, but I recommend using the desktop version. If your system or machine is not compatible, consider switching to a compatible one. 

One alternative is using Docker's online version, called `Play with Docker`. You can do everything with that without installing Docker Desktop locally.

### Create a Sample Container

After installation, you can open Docker Desktop and see there is a command to start a container using a `docker/getting-started` image. This container contains a very good tutorial for beginners.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/68310735-560c-8a42-948e-1e8a23a76f18.png)

After running that commmand in your terminal, you will see it with a randomly generated name. It is fine for testing purposes but in production you can set meaningful names for your containers. But for now, you can double check its image name and it matches `docker/getting-started`.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/68614138-c73d-de50-194c-617e33dd691b.png)

Personally, I would say this is a pretty boring UI. I would design it differently. 😁

Now, if you open your `localhost:80` in your browser, you will see a website running.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/239c5d61-d9c9-fe2e-114c-a6381e0d0428.png)

### Try using Docker online

When I learn something for the first time, I would not want to install a lot of staff locally. In this case, I want to do everything online without messing up my local environment. So let's try using an online [Docker playground](https://labs.play-with-docker.com/) to create the same container.

After signing in and start a session, everything is the same as using your local. Except the session will be closed after 4 hours.

Fill in the command and hit enter! 

```bash
docker run -d -p 80:80 docker/getting-started
```

It will hosted on the port 80 and you can open it to see what you just created.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/12a72013-bb84-90af-5c4d-f95ac542a146.png)

Notice the domain is different this time. And this is because you are using someone else's server other than your local machine.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/fffdc8e8-5bf3-394e-55c2-4ffe522481be.png)

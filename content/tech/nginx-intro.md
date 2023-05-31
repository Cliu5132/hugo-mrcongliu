---
title: "NGINX Intro"
date: 2023-05-30
---

Recently, I've been learning a Terraform course on Udemy. In which, I saw a file with weird syntax that I couldn't understand. So I fed that file to ChatGPT-4.

```shell
#!/bin/sh

set -e

envsubst < /etc/nginx/default.conf.tpl > /etc/nginx/conf.d/default.conf
nginx -g 'daemon off;'
```
The answer provided by ChatGPT-4 was quite satisfiying.

"The script performs the following actions:

1. **#!/bin/sh**: This line is called a shebang, and it tells the system to use the **/bin/sh** shell interpreter to execute the script.
2. **set -e**: This command sets the shell option **e**, which causes the script to exit immediately if any command in the script exits with a non-zero status. This is useful for ensuring that the script stops if an error occurs.
3. **envsubst < /etc/nginx/default.conf.tpl > /etc/nginx/conf.d/default.conf**: This command uses the **envsubst** utility to substitute environment variable values into the **/etc/nginx/default.conf.tpl** template file. The resulting file is saved as **/etc/nginx/conf.d/default.conf**. This allows you to dynamically configure the Nginx server based on environment variables passed to the Docker container.
4. **nginx -g 'daemon off;'**: This command starts the Nginx server with the **g** option, which sets global directives. In this case, the **daemon off;** directive tells Nginx to run in the foreground, which is necessary for Docker containers to keep running."

Then I requested a simple demo using NGINX from ChatGPT-4. It had done quite decent job!

Of course, the AI-coding-buddy made some mistakes here and there, but I was able to fix them because I am luckily a human.

First, create a project folder:

```bash
touch nginx-project
```

The files I created are shown as follows.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/15d3cfda-29c4-6076-69ba-d0068cae29ec.png)

Dockerfile:
```dockerfile
FROM nginx:alpine
COPY entrypoint.sh /entrypoint.sh
COPY default.conf.tpl /default.conf.tpl
RUN chmod +x /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh" ]
```

entrypoint.sh
```shell
#!/bin/sh

set -e

envsubst <./default.conf.tpl> default.conf
nginx -g 'daemon off;'
```

default.conf.tpl
```tpl
server {
  listen ${NGINX_LISTEN_PORT:-80};
  server_name localhost;

  location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
  }

  error_page 500 502 503 504 /50x.html;
  location = /50x.html {
    root /usr/share/nginx/html;
  }
}
```

In order to run this project in Docker, we need to buid a image. Remember, we need to have Docker Desktop running before use `docker` in our command line interface.

```bash
docker build -t nginx-image .
```

Then, run a container based on that image.

```bash
docker run -d -p 8080:80 --name nginx-container nginx-image
```

Now, if I go back to my Docker Desktop App. Great, I can see the container is running. Please ignore the 3 hours since I went out for something else before writing this blog.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/493f7148-0557-f2b5-d3ab-e35c63450abf.png)

If I go to `localhost:8080`, I can verify that NGINX web server is working.

![](https://hugo-mrcongliu.s3.ca-central-1.amazonaws.com/f04c661f-7ecb-b128-e2de-721125771e4c.png)

Overall, ChatGPT-4 is useful in terms of debugging and learning new knowledge. For me, I would use it more and keep learning Terraform!
---
layout: post
title: "Using Docker environment variables at JavaScript runtime"
subtitle: "This post is about accessing Docker environment variables at Javascript runtime. This technique can be used in any Javascript framework or technology."
date: 2023-09-04 00:00:00
categories: [Docker,Javascript]
tags: [Docker,Javascript]
author: "Anuraj"
image: /assets/images/2023/09/docker_static_webapp.png
---

When creating a container for a single-page application with JavaScript frameworks like Angular, React, or Vue.js, we may have to inject certain configuration variables based the deployment environment. A common scenario is using API endpoint URL for the Javascript application which can vary depend on the environment. In Angular, we can use environment file for this purpose. But again the problem is we have to hard code the API URL in the environment file. In this post we will explore how we can do configure container environment variable in vanilla Javascript. This technique can be used in any Javascript framework or technology.

For hosting and running we are using Nginx Alpine. Here is the Dockerfile

{% highlight YAML %}
{% raw %}

FROM nginx:mainline-alpine3.18-slim

COPY docker-entrypoint.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-entrypoint.sh

COPY index.html /usr/share/nginx/html/index.html

ENTRYPOINT ["/usr/local/bin/docker-entrypoint.sh"]

{% endraw %}
{% endhighlight %}

We are not using nginx directly for serving the index.html page, instead we are using a shell script - `docker-entrypoint.sh`. Here is the implementation of `docker-entrypoint.sh`.

{% highlight YAML %}
{% raw %}

#!/bin/sh
cat <<EOF > /usr/share/nginx/html/scripts/env.js
window.API_ENDPOINT="$API_ENDPOINT";
EOF

nginx -g "daemon off;"

{% endraw %}
{% endhighlight %}

In the shell script, we are adding setting a Javascript variable - `window.API_ENDPOINT` from the `$API_ENDPOINT` environment variable using the `cat` command. Then we are starting the `nginx` web server with daemon mode off. And in the Dockerfile we are using this shell script as entry point - so that the env.js file will be populated and we will be running the nginx web server.

Inside `scripts` folder we need to create an empty Javascript file with name `env.js`.

Here is the `index.html` file.

{% highlight HTML %}
{% raw %}

<!doctype html>
<html lang="en">

<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Blog</title>
    <script src="scripts/env.js"></script>
</head>

<body>
    <h1>Hello, world!</h1>
    <h2>Api Endpoint : <span id="apiendpoint"></span> </h2>
    <script src="scripts/index.js"></script>
</body>

</html>

{% endraw %}
{% endhighlight %}

Please note we are including the `env.js` file - which will set the API endpoint variable. 

And here is the `index.js` file

{% highlight Javascript %}
{% raw %}

document.getElementById("apiendpoint").innerHTML = "API Endpoint: " + window.API_ENDPOINT;

{% endraw %}
{% endhighlight %}

 And when can build the image using the following command - `docker build -t anuraj/staticweb:v1 .` - this command will build the image and we can run the container using following command - `docker run -p 8080:80 -e API_ENDPOINT="https://api.example.com" anuraj/staticweb:v1` - Please note we are passing the `API_ENDPOINT` as an environment variable. We can browse the application in http://localhost:8080. Here is the screenshot of the application running.

![Static Web App running]({{ site.url }}/assets/images/2023/09/docker_static_webapp.png)

This way we can consume environment variables in Javascript - which can be used to provide server side configuration to frontend applications. Full source code available here - [Docker samples](https://github.com/anuraj/Docker-Samples)

Happy Programming.
---
layout: post
title: "Create Containerized Build Agents with Azure DevOps Pipelines"
subtitle: "This post is about how to create containerized build agents with azure devops pipeline."
date: 2022-08-20 00:00:00
categories: [Azure,DevOps]
tags: [Azure,DevOps]
author: "Anuraj"
image: /assets/images/2022/08/build_agents.png
---

This post is about how to create containerized build agents with azure devops pipeline. Few days back I wrote a blog post on creating Azure Virtual Machine as Azure DevOps pipeline build agent. In this post we will be discussing about how to use Docker container as build agent. We can use this build agents for .NET Core and .NET Framework or any other languages / platforms. For this demo I am creating a build agent for a ASP.NET Core project.

Unlike other operating systems, Azure DevOps portal doesn't contains configuration for container build agents. But we can get the details from docs page - [Run a self-hosted agent in Docker](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops&WT.mc_id=AZ-MVP-5002040){:target="_blank"}. I am using the Docker file for Linux containers. Here is the [Dockerfile](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1&WT.mc_id=AZ-MVP-5002040){:target="_blank"} from the page.

{% highlight YAML %}
{% raw %}
FROM ubuntu:20.04
RUN DEBIAN_FRONTEND=noninteractive apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends \
    apt-transport-https \
    apt-utils \
    ca-certificates \
    curl \
    git \
    iputils-ping \
    jq \
    lsb-release \
    software-properties-common

RUN curl -sL https://aka.ms/InstallAzureCLIDeb | bash

# Can be 'linux-x64', 'linux-arm64', 'linux-arm', 'rhel.6-x64'.
ENV TARGETARCH=linux-x64

WORKDIR /azp

COPY ./start.sh .
RUN chmod +x start.sh

ENTRYPOINT [ "./start.sh" ]
{% endraw %}
{% endhighlight %}

There is a `start.sh` file. Here is the code for the shell script. We can find about the script file code [here](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops#create-and-build-the-dockerfile-1&WT.mc_id=AZ-MVP-5002040){:target="_blank"}

{% highlight Shell %}
{% raw %}

#!/bin/bash
set -e

if [ -z "$AZP_URL" ]; then
  echo 1>&2 "error: missing AZP_URL environment variable"
  exit 1
fi

if [ -z "$AZP_TOKEN_FILE" ]; then
  if [ -z "$AZP_TOKEN" ]; then
    echo 1>&2 "error: missing AZP_TOKEN environment variable"
    exit 1
  fi

  AZP_TOKEN_FILE=/azp/.token
  echo -n $AZP_TOKEN > "$AZP_TOKEN_FILE"
fi

unset AZP_TOKEN

if [ -n "$AZP_WORK" ]; then
  mkdir -p "$AZP_WORK"
fi

export AGENT_ALLOW_RUNASROOT="1"

cleanup() {
  if [ -e config.sh ]; then
    print_header "Cleanup. Removing Azure Pipelines agent..."

    # If the agent has some running jobs, the configuration removal process will fail.
    # So, give it some time to finish the job.
    while true; do
      ./config.sh remove --unattended --auth PAT --token $(cat "$AZP_TOKEN_FILE") && break

      echo "Retrying in 30 seconds..."
      sleep 30
    done
  fi
}

print_header() {
  lightcyan='\033[1;36m'
  nocolor='\033[0m'
  echo -e "${lightcyan}$1${nocolor}"
}

# Let the agent ignore the token env variables
export VSO_AGENT_IGNORE=AZP_TOKEN,AZP_TOKEN_FILE

print_header "1. Determining matching Azure Pipelines agent..."

AZP_AGENT_PACKAGES=$(curl -LsS \
    -u user:$(cat "$AZP_TOKEN_FILE") \
    -H 'Accept:application/json;' \
    "$AZP_URL/_apis/distributedtask/packages/agent?platform=$TARGETARCH&top=1")

AZP_AGENT_PACKAGE_LATEST_URL=$(echo "$AZP_AGENT_PACKAGES" | jq -r '.value[0].downloadUrl')

if [ -z "$AZP_AGENT_PACKAGE_LATEST_URL" -o "$AZP_AGENT_PACKAGE_LATEST_URL" == "null" ]; then
  echo 1>&2 "error: could not determine a matching Azure Pipelines agent"
  echo 1>&2 "check that account '$AZP_URL' is correct and the token is valid for that account"
  exit 1
fi

print_header "2. Downloading and extracting Azure Pipelines agent..."

curl -LsS $AZP_AGENT_PACKAGE_LATEST_URL | tar -xz & wait $!

source ./env.sh

print_header "3. Configuring Azure Pipelines agent..."

./config.sh --unattended \
  --agent "${AZP_AGENT_NAME:-$(hostname)}" \
  --url "$AZP_URL" \
  --auth PAT \
  --token $(cat "$AZP_TOKEN_FILE") \
  --pool "${AZP_POOL:-Default}" \
  --work "${AZP_WORK:-_work}" \
  --replace \
  --acceptTeeEula & wait $!

print_header "4. Running Azure Pipelines agent..."

trap 'cleanup; exit 0' EXIT
trap 'cleanup; exit 130' INT
trap 'cleanup; exit 143' TERM

chmod +x ./run-docker.sh

# To be aware of TERM and INT signals call run.sh
# Running it with the --once flag at the end will shut down the agent after the build is executed
./run-docker.sh "$@" & wait $!
{% endraw %}
{% endhighlight %}

This docker file and script file for a build agent in Docker. We can build the docker image using the command - `docker build -t anuraj/dotnetcorebuildagent:v1 .`, since my docker hub username is `anuraj`, I am tagging the image with the name, if your name is different use it. 

![Docker Build command]({{ site.url }}/assets/images/2022/08/docker_build.png)

As I am planning to use this build agent for building .NET Core project, I need to install the .NET Core SDK as well. To install the .NET Core SDK we need to add the following code in the Dockerfile.

{% highlight Shell %}
{% raw %}

RUN curl https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -o packages-microsoft-prod.deb
RUN dpkg -i packages-microsoft-prod.deb
RUN rm packages-microsoft-prod.deb

RUN apt-get update && \
   apt-get install -y dotnet-sdk-6.0

{% endraw %}
{% endhighlight %}

Now build the image again. And now we can run the docker container with few environment variables. Here are list of environment variables.

| Environment variable      | Description |
| ----------- | ----------- |
| AZP_URL      | The URL of the Azure DevOps or Azure DevOps Server instance. Ex: https://dev.azure.com/dotnetthoughts       |
| AZP_TOKEN   | Personal Access Token (PAT) with Agent Pools (read, manage) scope, created by a user who has permission to configure agents, at AZP_URL        |
| AZP_AGENT_NAME   | Agent name (default value: the container hostname).        |
| AZP_POOL   | Agent pool name (default value: Default).        |
| AZP_WORK   | Work directory (default value: _work).        |

Here is the way I am running my docker container with the environment variables like this.

{% highlight Shell %}
{% raw %}

docker run -e AZP_URL=https://dev.azure.com/dotnetthoughts `
-e AZP_TOKEN=YOUR-PERSONAL-ACCESS-TOKEN `
-e AZP_AGENT_NAME=dotnetthoughts-build-agent anuraj/dotnetcorebuildagent:v1

{% endraw %}
{% endhighlight %}

Once it is executed successfully we will be able to see something like this.

![Docker run command]({{ site.url }}/assets/images/2022/08/docker_run.png)

And we can see the build agent listed under the Azure DevOps Agent Pool. Since we didn't mentioned any specific pool, it will be under default pool.

![Build Agent]({{ site.url }}/assets/images/2022/08/build_agents.png)

Finally we need to configure the `Agent pool` inside the build definition. Like this.

![Build Pipeline - Build definition]({{ site.url }}/assets/images/2022/08/build_pipeline.png)

Once it is done, we can save and queue the build which will compile the ASP.NET Core project and run unit tests. Since the project doesn't have any unit tests that step is executed with a warning.

This way we will be able to configure the build agent in docker. Instead of running the build agent always we can execute this container and create build agents.

Happy Programming :)
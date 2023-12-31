---
layout: post
title: "Adding blog posts to your GitHub README with GitHub Actions"
subtitle: "This article will discuss about adding your blog posts to your Github readme with Github Actions."
date: 2021-07-16 00:00:00
categories: [GitHub,GitHubActions]
tags: [GitHub,GitHubActions]
author: "Anuraj"
image: /assets/images/2021/07/github_action_running.png
---
This article will discuss about adding your blog posts to your Github readme with Github Actions. You can add a README file to a repository to communicate important information about your project. You can find for more details about the GitHub readme from [here](https://docs.github.com/en/github/creating-cloning-and-archiving-repositories/creating-a-repository-on-github/about-readmes){:target="_blank"}

I was looking into some automation aspect related to GitHub readme automation - I was looking into some solution using Azure Functions and Logic Apps. And I found few solutions as well. But later I found one simple solution using GitHub Actions. Here is the GitHub Action which will run every day and use your blog RSS feed and update your Readme file.

First you need to modify the `Readme.md` file like this.

{% highlight Yaml %}
{% raw %}
### Recent Blog posts
<!-- BLOGPOSTS:START -->
<!-- BLOGPOSTS:END -->
{% endraw %}
{% endhighlight %}

Next you can create an GitHub Action and add the following code.

{% highlight Yaml %}
{% raw %}
name: Blog posts on ReadMe
on:
  schedule:
    # Runs every day at 9am UTC
    - cron: '0 4 * * *'

jobs:
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      - uses: actions/checkout@v2
      - name: Get RSS Feed
        uses: kohrongying/readme-the-rss@master
        with:
          feed_url: https://dotnetthoughts.net/feed
          count: 10
      - name: Commit file changes
        run: |
            git config --global user.name 'anuraj'
            git config --global user.email 'anuraj@example.com'
            git add .
            git diff --quiet --cached || git commit -m "Update README"    
      - name: Push changes
        uses: ad-m/github-push-action@master
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
{% endraw %}
{% endhighlight %}

And here is the screenshot of the GitHub Action running.

![Github Action Running]({{ site.url }}/assets/images/2021/07/github_action_running.png)

And here is the Updated Readme file.

![Updated Readme file]({{ site.url }}/assets/images/2021/07/updated_readme.png)

This way you will be able to update your blog posts automatically in to your GitHub readme file with the help of GitHub Actions.

Happy Programming :)
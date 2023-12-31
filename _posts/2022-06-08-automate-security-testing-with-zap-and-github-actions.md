---
layout: post
title: "Automate Security Testing with ZAP and GitHub Actions"
subtitle: "This post is about deploying a static sites to Azure storage account using Azure CLI and configuring GitHub actions to deploy the files."
date: 2022-06-08 00:00:00
categories: [DevSecOps,DevOps,Github Actions]
tags: [DevSecOps,DevOps,Github Actions]
author: "Anuraj"
image: /assets/images/2022/06/github_actions_new.png
---
This post is about running automated security tests on your web application with the help OWASP ZAP and GitHub Actions. In GitHub actions, OWASP ZAP provides a baseline scan feature which helps to find common security faults in a web application without doing any active attacks. The ZAP baseline action scans a target URL for vulnerabilities and maintains an issue in GitHub repository for the identified alerts. We can configure this action in Github public and private repositories. To get started first create an empty GitHub repository. And once it is created, click on the Actions tab. Either choose the `Skip this and set up a workflow yourself` option or select `Simple workflow` actions.

![Github Action - Create new action]({{ site.url }}/assets/images/2022/06/github_actions_new.png)

I am using the first option for this blog post. Next we can search for `ZAP Baseline scan`. And click on the `ZAP Baseline scan` and copy the content and paste it in the `main.yml`

![Zip Baseline Scan - Properties]({{ site.url }}/assets/images/2022/06/zap_baseline_scan.png)

In the step, we need to configure the `target` property. I am setting my blog url as the target. Here is the final github action workflow file.

{% highlight YAML %}
{% raw %}

name: CI
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
jobs:
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3
      
      - name: OWASP ZAP Baseline Scan
        # You may pin to the exact commit or the version.
        # uses: zaproxy/action-baseline@7cea08522cd386f6c675776d5e4296aecf61f33b
        uses: zaproxy/action-baseline@v0.7.0
        with:
          # Target URL
          target: "https://dotnetthoughts.net"

{% endraw %}
{% endhighlight %}

Once it is done, commit the file which will execute the GitHub action and once it is finished, we will be able to see the issues in the issues tab.

![Zip Baseline Scan - New Issue created]({{ site.url }}/assets/images/2022/06/zip_github_issues.png)

We can configure the scanning for a schedule or once we push some changes to QA / Staging / Production environment. We can also configure rules - to exclude or include certain web application alerts using `rules_file_name` property. You can create the rules with .tsv file. I created a `rules.tsv` file under `.zap` directory. Here is an example.

{% highlight Shell %}
{% raw %}
10202	IGNORE	(Absence of Anti-CSRF Tokens)
90022	IGNORE	(Application Error Disclosure)
{% endraw %}
{% endhighlight %}

And we can modify the GitHub action workflow file like this.

{% highlight YAML %}
{% raw %}

name: CI
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
  workflow_dispatch:
jobs:
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v3
      
      - name: OWASP ZAP Baseline Scan
        # You may pin to the exact commit or the version.
        # uses: zaproxy/action-baseline@7cea08522cd386f6c675776d5e4296aecf61f33b
        uses: zaproxy/action-baseline@v0.7.0
        with:
          # Target URL
          target: "https://dotnetthoughts.net"

{% endraw %}
{% endhighlight %}

Once it executed, you will be able to see an comment under the issue.

![Zip Baseline Scan - New Issue comment]({{ site.url }}/assets/images/2022/06/github_issues_comment.png)

This way we can configure security testing for your web application using OWASP ZAP Scanning using GitHub Actions. And the issues we can work on the track with GitHub. The ZAP baseline action scan runs the ZAP spider against the specified web application for (by default) 1 minute and then waits for the passive scanning to complete before reporting the results. This means that the script doesn’t perform any actual ‘attacks’ and will run for a relatively short period of time (a few minutes at most). You can configure the different ZAP scanning tools, like `OWASP ZAP API Scan` and `OWASP ZAP Full Scan` steps.

Happy Programming :)
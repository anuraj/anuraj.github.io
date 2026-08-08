---
layout: post
title: "GitHub Copilot - Automated Code Review on Pull Requests"
subtitle: "In this blog post, we will explore automated Code Review on Pull Requests using GitHub Copilot."
date: 2026-07-20 00:00:00
categories: [dotnet,codereview,github,copilot]
tags: [dotnet,codereview,github,copilot]
author: "Anuraj"
image: /assets/images/2026/07/github_action_main_expanded.png
---
In this blog post, we will explore automated Code Review on Pull Requests using GitHub Copilot. First we need to install the Copilot CLI in the GitHub Action and then we can execute the copilot CLI with prompt parameter and view the results.

```yaml
- name: Install Copilot CLI
    run: npm install -g @github/copilot
- name: Run Copilot review
    run: |
        set -e
        copilot --silent --yolo -p "Review the changes in this commit for bugs, security issues, and logic errors. Be specific about line numbers and provide detailed explanations for any issues found.
    env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

We need the `copilot-requests: write` permissions - which we need to add it in the beginning of the GitHub Action. Here is the screenshot of the application.

![GitHub Action - Running]({{ site.url }}/assets/images/2026/07/github_action_main.png)

And when we expand the Run Copilot review step, we can see the following content.

![GitHub Copilot Review details.]({{ site.url }}/assets/images/2026/07/github_action_main_expanded.png)

Now we can modify the action code and create the an issue in GitHub Action. For creating issues, we need the `issues: write` permission. Here is the full code which execute copilot CLI review and create an issue in GitHub.

```yaml
- name: Install Copilot CLI
  run: npm install -g @github/copilot
- name: Run Copilot review
  run: |
    set -e
    copilot --silent --yolo -p "Review the changes in this commit for bugs, security issues, and logic errors. Be specific about line numbers and provide detailed explanations for any issues found. In the output I only need Filename, LineNumber, Issue, Impact, Severity in markdown format" > /tmp/copilot_review.txt
    [[ -s /tmp/copilot_review.txt ]] || { echo "Copilot returned empty output" >&2; exit 1; }
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
- name: Create issue with Copilot review
  run: |
    gh issue create --title "Copilot Review: $GITHUB_SHA" --body-file /tmp/copilot_review.txt
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

Once the code is committed and execute the action, it will create a GitHub Issue with the following content.

![GitHub Action created issue]({{ site.url }}/assets/images/2026/07/github_created_issue.png)

This way we will be able to use GitHub Copilot CLI to review the code and create issue in GitHub using GitHub Actions. You can find the entire GitHub Action [here](https://github.com/anuraj/SqliteMcp)

Happy Programming.
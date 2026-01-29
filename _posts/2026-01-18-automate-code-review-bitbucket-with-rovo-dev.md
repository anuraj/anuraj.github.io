---
layout: post
title: "Automate Code Review in Bitbucket with Rovo Dev"
subtitle: "In this blog post we will learn how to enable code review in Bitbucket with Rovo Dev."
date: 2026-01-18 00:00:00
categories: [bitbucket,code-review]
tags: [bitbucket,code-review]
author: "Anuraj"
image: /assets/images/2026/01/rovo_code_review.png
---

In this blog post we will learn how to enable code review in Bitbucket with Rovo Dev. Rovo Dev introduced by Bitbucket recently. Rovo Dev can review pull requests in this repository and leave comments about potential improvements to quality, performance, security, and more.

To use this, first we need to enable Rovo Dev code review or AI code reviews in the bitbucket repository. We can do this from Bitbucket repository settings. Search for Code Review or we can find it under Features &gt; Rovo Dev Code Review menu.

![Enable Rovo Code Review]({{ site.url }}/assets/images/2026/01/rovo_code_review.png)

And toggle the `AI code reviews in this repository` option. I am not using `Acceptance criteria check` that we can turn on or turn off.

Once enabled, when ever we raise a Pull request, Rovo will start reviewing the code and add reviews comments in the Pull request.

Here is an example of a comment.

![Rovo Code Review comment]({{ site.url }}/assets/images/2026/01/rovo_code_review_comment.png)

We can further customize the behavior by setting custom instructions to the repository. To do this we need to add `.rovodev` directory in the repository root and then add a `.review-agent.md` file in the `.rovodev` folder and add clear, specific instructions for Rovo Dev to follow when reviewing pull requests in the repository.

For more information on how to set custom instructions check out the [atlassian support website](https://support.atlassian.com/rovo/docs/set-custom-instructions-for-code-reviews/)

This way we will be able to enable automated code review in Bitbucket and setup custom instructions.

Happy Programming.
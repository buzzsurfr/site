---
title: "Automatically Deploy Hugo Blog to Amazon S3"
date: 2019-01-19T12:00:00-05:00
draft: false
categories: ["Build Log"]
tags: ["aws", "github", "hugo", "s3"]
description: "I had grand aspirations of maintaining a personal blog on a weekly basis, but sometimes that isn't always possible. I've been using my iPad and [Working Copy..."
image: ""
---

I had grand aspirations of maintaining a personal blog on a weekly basis, but sometimes that isn't always possible. I've been using my iPad and [Working Copy](https://itunes.apple.com/us/app/working-copy/id896694807?mt=8) to write posts, but had to use my regular computer to build and publish. CI/CD pipelines help, but I couldn't find the right security and cost optimizations for my use case...until this year.

My prior model had my blog stored on GitLab because it enabled a free private repository (mainly to hide drafts and future posts). I was building locally using a Docker container and then uploading to Amazon S3 via a script.

At the beginning of the year, GitHub announced [free private repositories](https://github.blog/2019-01-07-new-year-new-github/) (for up to 3 contributors), and I promptly moved my repo to GitHub. (NOTE: I don't use CodeCommit because it's more difficult to plumb together with Working Copy.)

I was able to now plumb CodePipeline and CodeBuild to build my site, but fell short on deploying my blog to S3. I had to build a Lambda function to extract the artifact and upload to S3. The function is only 20 lines of Python, so it wasn't difficult.

But then, AWS announced [deploying to S3 from CodePipeline](https://aws.amazon.com/about-aws/whats-new/2019/01/aws-codepipeline-now-supports-deploying-to-amazon-s3/), meaning my Lambda function was useful for exactly 10 days!

Now, I can write a post from my iPad and publish it to my blog with a simple commit on the iPad! It's a good start to 2019, with (hopefully) more topics coming soon...
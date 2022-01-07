---
layout: post
title:  "10 - Now Hiring!"
date:   2022-01-01 00:00:01 -0800
categories: kringlecon2021 objective10 main
---

This is challenge number 10 in the 2021 SANS Holiday Hack Challenge (https://2021.kringlecon.com/). Objective:

>What is the secret access key for the Jack Frost Tower job applications server? Brave the perils of Jack's bathroom to get hints from Noxious O. D'or.


We start this challenge off by visiting Jack Frost's job application website at https://apply.jackfrosttower.com/

![Objective10 website](/assets/kringlecon2021/objective10/objective10_website.jpg)

Next we can start exploring the website to get an idea of what we might be able to do to obtain the secret access key. Noxious O. D'or gives us a hint that we should check out the AWS IMDS documentation. Given this knowledge and knowing that IMDS has been abused via SSRF vulnerabilities before we should start looking for any place that accepts a URL as a parameter. It doesn't take long before we can see that the career application has a field which takes a "URL to your public NLBI report"


![Objective10 website](/assets/kringlecon2021/objective10/objective10_application.jpg)


So, let's fill out the application and monitor the request in the browser. For the URL, we will plug in the IMDS IP address and try to find out if the application is running on an EC2 instance and has a role attached. http://169.254.169.254/latest/meta-data/iam/security-credentials. The request is accepted and the response returns an html document and an image called foo.jpg (which was the name I used when submitting the form). And if you look closely, the foo.jpg is showing up as a broken image on the page.

![Objective10 ssrf](/assets/kringlecon2021/objective10/objective10_ssrf.jpg)

The browser wasn't showing the details contents of the jpg, just that it was broken. So I configured burp and replayed the request to the jpeg to see what it returned. Ah-hah! We see "jf-deploy-role" in the response!

![Objective10 role](/assets/kringlecon2021/objective10/objective10_role.jpg)

So now let tweak our request to fetch credentials for this role. We go back and change our application request to point to URL http://169.254.169.254/latest/meta-data/iam/security-credentials/jf-deploy-role, and then make a request for foo.jpg again


![Objective10 credential](/assets/kringlecon2021/objective10/objective10_credential.jpg)

We have our answer! The secret access key is "CGgQcSdERePvGgr058r3PObPq3+0CfraKcsLREpX"

![Objective10 complete](/assets/kringlecon2021/objective10/objective10_complete.jpg)

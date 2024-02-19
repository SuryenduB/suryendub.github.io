---
layout: post
title: Containerization using Docker for IIQ
subtitle:  Containerization using Docker for IIQ - A Primer
cover-img: /assets/img/iiq-container6.png
thumbnail-img: /assets/img/iiq-container6.png
share-img: /assets/img/iiq-container6.png
tags: [ IAM, IGA , Docker, IIQ, Sailpoint, Containerization]
readtime: true

---


# Containerization using Docker for IIQ

Containerization using Docker has become increasingly popular in the world of software development and deployment. It addresses the challenges of deploying applications across different environments by providing a consistent and isolated runtime environment. With Docker, applications and their dependencies are packaged into containers, which encapsulate everything needed to run the application, including the code, runtime, system tools, and libraries. This eliminates the "it works on my machine" problem and ensures that applications run consistently across different environments, from development to production.

I recently came across a resource for containerizing IdentityIQ (IIQ) using Docker, and I wanted to share some insights on how this can be done.

1. Clone the [GitHub Repository](https://github.com/drosenbauer/sailpoint-iiq-docker).
1. Download the IdentityIQ zip file from the Sailpoint Compass Website.
1. Build the IdentityIQ war file by running the `build.sh` script.
1. The default `docker-compose.yml` file uses Microsoft SQL  as the database, however, in my test, I have realized that this resulted in an error due to recent changes in the MS-SQL docker image. Update the `docker-compose.yml` file to use MySQL as the database.
![iiq-component](/assets/img/iiq-container1.png)
![iiq-component](/assets/img/iiq-container2.png)
1. Run the `docker-compose up -d` command to start the containers.

1. Wait for the `iiq-init` container to complete the initialization process. ![iiq-component](/assets/img/iiq-container3.png)

1. Access IdentityIQ by navigating to `http://localhost:8080/identityiq`.
![iiq-component](/assets/img/iiq-container4.png)
![iiq-component](/assets/img/iiq-container5.png)

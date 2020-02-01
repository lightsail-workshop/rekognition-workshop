---
title: "Introduction"
weight: 50
---

## Deploying an AI Driven Application with Amazon Lightsail
In this workshop you're going to deploy memory matching game built using Amazon Rekogntion onto an Amazon Lightsail server. 

Game players click on pairs of face-down tiles to reveal pictures of celebrities. If the two celebrities match, the tiles remain turned face up. If they don't, they are returned face down and the player picks two more tiles until all the tiles are matched. 

The application itself is comprised of a web front-end written in React and an API server written in Node.js. The API server makes calls to the Amazon Rekogntion service. Rekognition takes as input the two photos the user clicked on (which are stored in a Amazon S3 bucket) and determines the name of the celebrity in each photo. If the names are the same, then the player has a match. 

At the end of the game the player's score is saved into a MySQL database. 

![](./images/architecture.png?classes=border)

For the purposes of this workshop all three application tiers (front-end, API server, and database) are all running on a single Lightsail instance. If you finish the workshop early, you may want to try and separate the front-end and API servers out onto their own Lightsail intances, and use an Amazon Lightsail mysql database to store the scores. 

### What is Amazon Lightsail

Amazon Lightsail is the easiest way to get started in the cloud. Lightsail provides compute, storage, and networking at a low fixed price. But, it doesn't stop there, your Lightsail instances are built on and can leverage the power of the rest of services that AWS offers. This means that you can start building more quickly than ever before, but are not limited in what you can do long term. 

### What is Amazon Rekognition
Amazon Rekognition makes it easy to add image and video analysis to your applications using proven, highly scalable, deep learning technology that requires no machine learning expertise to use. With Amazon Rekognition, you can identify objects, people, text, scenes, and activities in images and videos, as well as detect any inappropriate content. Amazon Rekognition also provides highly accurate facial analysis and facial search capabilities that you can use to detect, analyze, and compare faces for a wide variety of user verification, people counting, and public safety use cases.

### What is Amazon S3
Amazon Simple Storage Service (Amazon S3) is an object storage service that offers industry-leading scalability, data availability, security, and performance. This means customers of all sizes and industries can use it to store and protect any amount of data for a range of use cases, such as websites, mobile applications, backup and restore, archive, enterprise applications, IoT devices, and big data analytics. Amazon S3 provides easy-to-use management features so you can organize your data and configure finely-tuned access controls to meet your specific business, organizational, and compliance requirements


 
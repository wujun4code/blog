+++
authors = ["Jun Wu"]
title = "About Me"
date = "2024-01-19"
description = "About me"
+++

# Contact Information

- **Email:** junwu@shouyicheng.com
- **WeChat:** wing_wj

# Personal Information

- **Name:** Jun Wu
- **Gender:** Male
- **Birthdate:** 1989
- **Education:** Bachelor's in Software Engineering, Liaoning University of Technology (2007-2011)
- **Years of Experience:** 13
- **Technical Blog:** [http://blog.shouyicheng.com](http://blog.shouyicheng.com)
- **Github:** [http://github.com/wujun4code](http://github.com/wujun4code)

# Summary

I am a seasoned full-stack developer with a comprehensive background in C# across various product lines, including Unity, WPF, SystemCenter, WP, ASP.NET, and .NET Core. Proficient in backend microservices development using Kubernetes and Docker. My expertise extends to diverse client platforms such as iOS, Android, Hybrid, and Web. I have a strong ability to quickly adapt to and utilize unfamiliar frameworks and technologies, demonstrating proficiency in tool application and framework utilization. I excel in understanding and interpreting requirements, with robust communication skills to convey complex technical details to colleagues with different backgrounds. Effective communication with international teams in English is one of my strengths. I have experience in maintaining large technical documentation projects and am skilled in copywriting.

# Work Experience

## iHerb.com (January 2019 - January 2024)

### Catalog
As a full-stack developer, I utilized multiple languages to implement business functionalities in various business components. Implemented a GraphQL data API using TypeScript and apollographql framework for Web (Desktop+React Mobile Web)/Mobile App/Mini Program. Also responsible for maintaining Web pages using MVC.
- **Technology Stack:** C#/.NET Core/SQL Server/React/ASP.NET MVC

#### Project: Catalog GraphQL
Developed an Apollo-based Node.js backend middleware serving as the aggregation API for the entire website. Aggregated data from upstream APIs including orders, product information, inventory, payments, personal information, etc., and provided it downstream using GraphQL. Downstream includes Web frontend, SSR-rendering backend, Mobile App, and WeChat Mini Program.
- **Technology Stack:** Apollo/NodeJS/Dgraph/Kafka

#### Project: Welcome Mat
Implemented the backend for attracting new user registrations and providing the first order discount code. Designed and executed a comprehensive validation process during email registration, each requiring a different discount code. Independently completed the entire backend module from design, coding to deployment, and successfully launched.
- **Technology Stack:** C#/.NET Core/SQL Server

### UGC
Backend developer role responsible for maintaining user comments and Q&A. Used .NET Core 6.0 as the core API layer, with Elasticsearch as a full-text search engine, MongoDB for persistent storage, RabbitMQ for handling user-submitted comments and Q&A, and Hangfire for periodic data processing and adapting to new requirements.
- **Technology Stack:** .NET Core 6.0/Elasticsearch/MongoDB/RabbitMQ/Hangfire

#### Project: Gateway API
Developed an API for SEO on the server side. Redirected sub-sites based on different countries, regions, and languages, serving as the core SEO middleware for the entire UGC project. Independently completed API design, data source organization, cache strategy formulation, coding, unit testing, release, and deployment.
- **Technology Stack:** C#/Kubernetes/Jenkins CI/CD/MongoDB/.NET Core 3.0

#### Project: Incentive Review Mechanism
Functionality involved rewarding users with $1 in cash for submitting a valid review. Designed the entire process from user submission of a review to awarding $1. Implemented a thorough review process, including scanning user-added reviews, scoring reviews, verifying review effectiveness, and checking past review records. Utilized the Chain of Responsibility pattern to improve efficiency and code readability.
- **Technology Stack:** .NET Core 6.0/RabbitMQ/MongoDB

### BFF (Backend For Frontend)
As an architect and backend developer, provided data APIs to Mobile App and Mini Program. Built a Web API project based on ASP.NET Core, a database microservice using Parse Server, and utilized MongoDB for storage. Redis was used for frontend caching.
- **Technology Stack:** C#/Kubernetes/Jenkins CI/CD/MongoDB/.NET Core 2.2

## Leancloud (April 2014 - December 2018)

### C# SDK
Responsible for daily maintenance of the C# SDK for Unity/Windows Phone/Windows Desktop/WPF/Xamarin, developing various cross-platform SDKs using Visual Studio.
- **Technology Stack:** C#/Unity3D

### Realtime Chat Component SDK
Developed a C# SDK for a client-side real-time chat component based on WebSocket. The server-side used Kafka for message pushing. The client-side encapsulated server-side chat rooms, groups, private messages, and other functionalities, designing C# interface protocols and writing code.
- **Technology Stack:** C#

### LeanCloud Official Documentation - [https://docs.leancloud.cn/](https://docs.leancloud.cn/)
Mainly responsible for Chinese documentation writing and maintenance, as well as writing and reviewing example code.
- **Technology Stack:** Nunjucks/Markdown/jQuery

### Other Projects

## Weichuang (Beijing) Branch

### CloudBox
Exposed System Center's basic API services to internal WCF services, implementing the functionality of virtualized cloud desktops. Packaged System Center's Powershell API into WCF/REST APIs for client-side calls to create virtual desktop hosts, manage cloud resources, monitor subnets, etc. Main clients were banks and large enterprises, focusing on virtual desktop office solutions. The corresponding product is VMWare. The project involved exposure to Microsoft Azure's public and hybrid cloud, as well as an in-depth understanding of Azure's REST API, especially the encapsulation of asynchronous REST APIs for virtual machine creation.
- **Technology Stack:** C#/.NET/SQL Server/System Center/WCF/Azure REST API

## Symbio Beijing

Mainly responsible for developing Windows Phone apps based on Silverlight.

- **Technology Stack:** Silverlight for Windows Phone

## Nokia App Studio

Developed Windows Phone versions corresponding to iOS apps acquired by Nokia. Responsible for three games, using the last version of the XNA engine and Silverlight. This project marked the beginning of understanding Microsoft's XAML technology stack, XNA structure, and MVVM implementation. Although a beginner at the time, the project was extensive, involving twenty-plus games with a team of five to six people, each responsible for three to four games.
- **Technology Keywords:** C# WPF/Silverlight/XNA/XAML/MVVM WP/Cocos2d

## EverNote Windows Phone

Responsible for the development and iteration of EverNote's Windows Phone version.

- **Technology Stack:** Silverlight for Windows Phone

## Microsoft Ventures Accelerator Website

An outsourcing project on-site at Microsoft, building the official website for the Microsoft Ventures Accelerator. A showcase project using ASP.NET MVC/jQuery/Azure SQL Server.
- **Technology Stack:** ASP.NET MVC/jQuery/Azure SQL Server

# Open Source Projects and Works

## Open Source Projects

- [strapi-csharp-sdk](https://github.com/strapi-extensions/strapi-csharp-sdk): C# SDK for Strapi CMS, based on .NET 6, supporting both server-side and client-side.
- [localKube](https://github.com/kubernetes-go/localkube): CI/CD command-line tool based on Microk8s and Docker for Kubernetes. Supports multi-cluster multi-template rendering to generate Yaml files, relying on docker/helm/git for compilation, packaging, template rendering, deployment, and other convenient functions.
- [DoChat](https://github.com/wujun4code/DoChat): Live Chat Demo based on Ionic 2.

## Technical Articles

- [个人博客- 中文](https://blog.shouyicheng.com)

# Skills

- **Programming Languages:** C# (Proficient), TypeScript (Proficient)
- **Web Development:** React (Basic), Angular (Basic)
- **Desktop:** WPF (Basic)
- **Mobile Platforms:** Ionic (Proficient)
- **Web Frameworks:** ASP.NET Core (Proficient), Express (Basic)
- **Frontend Frameworks:** Angular 2+ (Basic), React (Basic)
- **Frontend Tools:** Gulp (Proficient), SaSS (Basic), Gatsby (Basic)
- **Databases:** MongoDB (Proficient), SQL Server (Basic)
- **Message Queues:** RabbitMQ (Proficient), Kafka (Basic)
- **Version Control/Deployment Tools:** Git (Proficient), Markdown (Proficient), Jenkins (Proficient), Azure DevOps (Proficient), GitHub Actions (Proficient)
- **Virtualization:** System Center (Basic), Kubernetes (Proficient)
- **Backend Components:** Kubernetes (Proficient), Redis (Basic), ELK (Proficient), Docker (Proficient)
- **Testing:** XUnit (Proficient), Mocha (Proficient)
- **Cloud Platforms:** GCP (Proficient), Azure (Proficient)
- **IDEs:** Visual Studio (Proficient), Visual Studio Code (Proficient)
- **Operating Systems:** Linux (ElementaryOS) (Proficient), macOS (Proficient), Windows (Proficient)
- **Language:** English (Fluent)

Feel free to let me know if you have any specific preferences or if there are any modifications you'd like!
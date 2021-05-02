# Introduction

A common question we encounter when describing FLARE is, what is the role of Git/GitHub in the system? The purpose of this document is to elaborate on the reasons behind the use of Git in FLARE, how it is used, and advantages and disadvantages compared to other related systems. We hope this will not only help clarify how FLARE leverages Git, but also help others in the community better understand if/where Git is a good choice for their systems.

# What is Git?

[Git is a free and open source distributed version control system](https://git-scm.com/). Git has been widely used by the open-source software development community - it is fast, lightweight, and there are open-source implementations widely available for Git clients and servers. 

Git is primarily used to store plain text files - such as source code - and keep track of every update (version) that is applied to files stored in the system. Git has been designed with a focus on text files because it tracks differences between updates - e.g. an update that is a single line of code inserted into a program's source code can be efficiently stored as just the difference. However, Git is not limited to source code - Git can be used to store other types of text files (e.g. many files used in FLARE are text CSV files), as well as arbitrary files (e.g. NetCDF). The main difference is, when handling non-text files, Git loses the ability to store differences only - each new version stores the whole new file.

Git can be used as a general-purpose cloud storage system to store arbitrary files, and it is particularly effective when handling of text files. 

Git follows a client-server design. The Git server holds all committed version of the data in persistent storage, while the client can be used to download, modify, and upload data to the server. Git [implements smart, reliable protocols](https://git-scm.com/book/en/v2/Git-Internals-Transfer-Protocols) for data transfer between client and server over the network, using URLs to identify the server and [HTTP or SSH as transports](https://git-scm.com/book/en/v2/Git-on-the-Server-The-Protocols). Git implementations do most of the heavy-lifting necessary to ensure reliable transfers between clients and servers - even in the presence of failures.

## What is the difference between Git and GitHub?

Git is an open-source protocol, and many implementations of Git exist. For instance, if you run a Linux server, it can be configured to run an open-source Git server, and if you run a Windows or Mac client, you can install a different implementation of a Git client software to access the server. Git is not proprietary and is not tied to any particular operating system, cloud vendor. 

A flagship Git-based service on the Internet is [GitHub](https://github.com). GitHub is a business, which offers free access to projects (up to a size). It is widely used - not just by the software development community, but by many other projects. It also offers a clean, intuitive Web interface to browse data repositories, and has built a large community of of users. You can think of GitHub as a large-scale, cloud-hosted Git server - perhaps the largest installation of a Git server in the world. Still, it is essentially a large Git server - it is no different fundamentally from a Git server you might run on your own network using open source software.

In summary:

* FLARE relies on the *open Git protocol*
* FLARE *does not require GitHub* to work
* A person deploying FLARE *may elect to use GitHub* as a repository for convenience
* A person deploying FLARE *may elect to use any other open-source Git server* as a repository

# Why is Git used in FLARE?

FLARE has been designed to use Git as a framework to handle data transfers and storage. Having covered the basics of Git, let's dive deeper into a discussion of the several aspects that have motivated this choice.

## Git is an open protocol with a robust open-source software ecosystem

One of the key motivations for using Git is that it's an open, mature, widely-used, and well-understood protocol. This means we can leverage Git for data transfers and storage, and expect a robust implementation. Getting reliable data transfer and storage right is a challenging problem - we don't want to reinvent the wheel with a custom solution.

Git has several open-source implementations, including [GitLab](https://gitlab.com/gitlab-org/gitlab-ce), [Gitea](https://gitea.io), [Gitbucket](https://github.com/gitbucket/gitbucket). In particular, GitLab has a Web interface and is deployable using containers, making it a good candidate for use in FLARE as an alternative (or complement to) GitHub.

## Git implements a reliable transfer mechanism

FLARE is an end-to-end distributed system that allows modules to run right next to where the data is captured by sensors. Small computers that run in the field - which we call *sensor gateways* - provide the starting point for data driving the execution of FLARE. Sensor gateways read from data loggers to retrieve raw sensor data, and need to securely transfer data to storage servers that "feed" the next stages in the FLARE workflow.

Think of sensor gateways as small, field-hardened computers that run Linux - an example device we use is the [fitlet2](https://fit-iot.com/web/products/fitlet2/). These gateways run at the *edge* of the network, in conditions that are very different from a conventional cloud system: they may be battery-powered and run only a fraction of the time, and they may be connected to the Internet via cell phone links that are unreliable. Bottom line, there are many chances for things to go wrong during transfers, such as: the cell link may disconnect for long periods of time, or the device may be powered off when a battery is drained. 

What we need is a mechanism that ensures that, no matter how many times a device restarts, or a cell link disconnects, that a transfer is accomplished in a reliable fashion. Git provides this capability: if a device powers off or gets disconnected during a transfer, Git will ensure that the transfer is properly resumed when the device is back on and connected. 

Git uses [cryptography techniques for data assurance](https://git-scm.com/about/info-assurance). Regardless of how many retries are needed to perform a transfer, once it completes, we can be assured that when the transfer is eventually completed, we can be assured the information stored in the server matches the information sent by the client.

## Git only transfers differences for text files

Files that are retrieved from data loggers by sensor gateways tend to be text-based - such as a CSV file where each row has a timestamp and columns with the various fields/observations read from sensors. 

It is simple to append new measurements to a text CSV file - data loggers can be programmed to do so. With Git, we can be assured that although a file grows in size with appends, only the differences are transferred. This reduces network usage (which is important when using cell data plans), while keeping the process of buffering and sending data simple. 

We want the sensor gateways to be as simple as possible in terms of the software they run - by appending measurements to a file and using Git to reliably transfer differences, the gateway software does not need to worry about corner cases such as restarts in the middle of multi-file transfers. 

## Git keeps track of versions

Another benefit of using Git is that we have visibility into every single update to a file. This is especially important when something unexpected happens that may require manual troubleshooting. For instance, if problems arise in a sensor that lead to problems in the data during a period of time, it is possible to isolate the problem (and possibly correct the data) by zooming in to a particular version.

## Git is familiar to many practitioners in the ecology and forecasting communities

For usability and productivity, it is important to expose interfaces that are relatively familiar to our target end users. One could argue that a custom system, or an alternative system to Git might perform better in terms of data transfers and storage. However, using a system that has little to no adoption in a community creates a barrier to entry. Git, however, is used by many in the ecology community, and provides a familiar interface. We have had success in having FLARE used by ecology students in our projects, in no small part due to the fact that data is available and can be accessed from Git repositories.

## Git servers can be public or private without changes to FLARE

FLARE is designed so that arbitrary Git servers can be configured, through a configuration file - all that is needed is the URL of the server, user name, and the SSH key to authenticate. Thus, users deploying FLARE have the option to deploy it on public services for convenience (e.g. GitHub) as well as on private servers for data privacy (e.g. a GitLab deployment) without changes to FLARE code.

# How is Git used in FLARE?

Git is used in various steps of data transfer and storage in support of the execution of a FLARE forecasting workflow:

* Git clients run on sensor gateways to reliably upload data
* Git clients run on FLARE containers to retrieve files generated by other containers in the workflow, and to upload files to be used by other containers
* Git server(s) run on the cloud, or on hosts "near" the FLARE containers (e.g. in the same local area network) to serve one or more repositories

Note that, while Git is used at the core of FLARE data transfers, it does not need to be the only protocol used in the overall data management of a FLARE end-to-end workflow. Git is used as a data framework to "glue" the various modules used in the operation of FLARE; other protocols may be used to expose data products (e.g. forecast results) using different platforms.

For example, a FLARE workflow may generate outputs that are committed to long-term data repositories, such as [EDI](https://environmentaldatainitiative.org/). Such repositories may have custom APIs and metadata requirements to publish data that is [FAIR - Findable, Accessible, Interoperable, and Reusable](https://www.go-fair.org/go-fair-initiative/). FLARE modules/containers can be designed to retrieve data from FLARE Git servers, prepare metadata, package and publish to a long-term repository such as EDI.

# What are the challenges/limitations in using Git in FLARE?

* Git excels for text data that is appended - such as sensor data logged as time series in a CSV file. For data that is binary (e.g. NetCDF files), Git still works, but it does not benefit from keeping track of differences between versions
* Git servers impose limits on file and repository sizes, posing limitations when storing large files. These limits depend on the provider of Git services; for instance, for public services, as of Jan 2019, GitHub has a limit of 100MB per file and 2GB per repository in its free tier, while Gitlab has a 10GB cap on file/repo sizes. Private servers can be configured with larger limits

Overall, Git is particularly well-suited for FLARE use cases of forecasting in aquatic systems - aggregating time-series sensor data from sensor gateways deployed in the field, as well as to hold model outputs (e.g. NetCDF files) of the order of MBs/file, if backed by a server provisioned with sufficient file/repository capacity limits. Other use cases that demand high-volume binary data (e.g. high-definition images) from sensors are not particularly well-served by Git.


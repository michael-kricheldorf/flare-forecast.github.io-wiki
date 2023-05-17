# S3 storage in FLARE

FLARE uses a de-facto standard for cloud storage - S3 (Amazon Simple Storage Service) to hold all persistent data, including FLARE forecast outputs and NOAA weather forecasts. Here's a an intrudoction to S3 and a quick start to using S3 in FLARE. 

## What is S3?

S3 is a standard protocol used to store and retrieve data from a cloud storage provider. It's in many ways similar to other cloud storage (e.g. Dropbox, iCloud, Google Drive), but uses a de-facto standard that is supported by multiple cloud vendors, as well as by open-source software such as [MinIO](https://min.io/). Using S3 allows FLARE data to be stored reliably on a cloud provider for access.

## What is an S3 bucket?

S3 bucket is the terminology used for a placeholder for all data stored in an S3 account. Think of a bucket as equivalent to a folder in your computer, where you can store many/large files (which in S3 terminology are called "objects"). An S3 bucket is referred to by the bucket's name, e.g. forecasts, drivers

## What is an S3 server?

An S3 server is the server on the Internet that holds the storage for all the objects in an S3 bucket. An S3 server is referred to by an URL, e.g. s3.flare-forecast.org

## S3 in FLARE

S3 storage is a key part of the FLARE workflow - it is used to store data persistently for access both by FLARE users (using desktop S3 clients such as Rclone and MinIO client) as well as by the FLARE daily forecasts (using the R package aws.s3). The S3 cloud storage allows access from anywhere - user clients or cloud computing workflows. It supports both anonymous, public read-only access to data, as well as authenticated access for read/write. All FLARE users have read-only access to data automatically - if you need authenticated read/write access, please contact the project PIs.

## How to use S3 in FLARE?

First off, it's important to understand the naming conventions for how files are organized in buckets in the FLARE S3 server. [FLARE S3 Storage Naming Convention](FLARE-S3-Storage-Naming-Convention)

As a user, there are two different ways you can browse data in the S3 buckets. For read-only access, you don't need credentials. For writable access, you need an S3 access key. The two clients you can use are [Rclone](Manage-Files-on-S3-Using-RClone) and 
[MinIO](Accessing-FLARE-S3-Storage-with-MinIO-Client); they have similar capabilities, see which one you are more comfortable installing and using.

As a developer, you can access S3 data [from R code](Accessing-FLARE-S3-Storage-from-R)






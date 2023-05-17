# S3 storage in FLARE

FLARE uses a de-facto standard for cloud storage - S3 (Amazon Simple Storage Service) to hold all persistent data, including FLARE forecast outputs and NOAA weather forecasts. Here's a an intrudoction to S3 and a quick start to using S3 in FLARE. 

## What is S3?

S3 is a standard protocol used to store and retrieve data from a cloud storage provider. It's in many ways similar to other cloud storage (e.g. Dropbox, iCloud, Google Drive), but uses a de-facto standard that is supported by multiple cloud vendors, as well as by open-source software such as [MinIO](https://min.io/). Using S3 allows FLARE data to be stored reliably on a cloud provider for access.

## What is an S3 bucket?

S3 bucket is the terminology used for a placeholder for all data stored in an S3 account. Think of a bucket as equivalent to a folder in your computer, where you can store many/large files (which in S3 terminology are called "objects"). An S3 bucket is referred to by the bucket's name, e.g. MyBucket

## What is an S3 server?

An S3 server is the server on the Internet that holds the storage for all the objects in an S3 bucket. An S3 server is referred to by an URL, e.g. mys3server.mydomain.org


## S3 in FLARE

Why do we use S3?

How do we use S3?

## How to use S3 in FLARE?

Naming convention

Access from R

Access from dekstops



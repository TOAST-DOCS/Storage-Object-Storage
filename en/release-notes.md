## Storage > Object Storage > Release Notes

### May 27, 2025
#### Added Features
* [Console] Added the feature to resume/suspend container replication

#### Feature Updates
* [API] Improved Amazon S3 API compatibility
    * Fixed issue where the LastModified value of each objects in the object list was displayed to the nearest millisecond

#### Bug Fixes
* [API] Fixed issue where multipart uploads to buckets with object locks using Amazon S3 compatible APIs would cause requests to fail and existing part objects to be deleted

### August 27, 2024
#### Feature Updates
* [Console][API] Added service gateway IP access control settings
    * You can handle IP ACL exceptions for requests through the service gateway
* [Console] Added the feature for replication to a different project in the same organization
* [API] Improved Amazon S3 API compatibility
    * Improved response compatibility for the following errors
        * Request to create a bucket with an invalid name
        * Container request with an invalid path
        * Object request with an invalid path


### May 28, 2024
#### Added Features
* [Console][API] Added Economy storage class - Korea (Pangyo) Region
    * You can select storage classes based on frequency of data access, cost requirements, etc.
* [Console][API] Added lifecycle expiration action
    * You can move objects whose lifecycle has expired to another container

#### Feature Updates
* [Console] Improved the feature to copy objects
    * You can select multiple objects to copy/move
* [Console] Changed container naming conventions
    * Minimum of 3 characters and maximum of 63 characters allowed
    * Lowercase letters, numbers, -, and . allowed
    * IP-formatted names not allowed

### February 27, 2024
#### Added Features
* [Console][API] Added the feature to empty containers

#### Feature Updates
* [Console] Improved the feature to delete objects

### November 28, 2023
#### Added Features
* [API] Added support for AWS Signature V4 Chunked Upload to Amazon S3 compatible API
* [Console] Added the feature to create signed URLs

### May 30, 2023
#### Added Features
* [Console][API] Added the IP ACL feature

### March 28, 2023
#### Feature Updates
* [API] Changed API endpoints

#### Bug Fixes
* [API] Fixed an issue where, when a segment object of a multipart object uploaded to PUBLIC containers is in another container, it could not be downloaded without a token
* [API] Fixed an issue where, when a multipart object with the same name is updated using the S3 compatible API, the previous part object could not be deleted 

### November 29, 2022
#### Added Features
* [Console][API] Added a feature to set an object lock container

### October 13, 2022
#### Added Features
* [Console] Added a feature to set a container ACL 

### September 27, 2022
#### Added Features
* [Console] Added a feature to set container encryption
* [Console] Added a feature to set Cross-Origin Resource Sharing (CORS)

#### Feature Updates
* [Console] Improved the UI of Get Container and Container Settings

### August 23, 2022
#### Added Features
* [API] Added a feature to set the RFC compliant ETag format

### March 29, 2022
#### Added Features
* [Console] Added the inter-region container replication feature

### January 25, 2022
#### Added Features
* [Console] Added a feature to obtain S3 API credentials

#### Feature Updates
* [Console] Applied batch deletion of segment objects when deleting multipart objects
* [API] Applied limit on characters that can be entered in container names
    * Some special characters (' " < > ;), spaces, and relative path characters (. ..) are not allowed

### October 26, 2021

#### Feature Updates
* [Console] Changed the limitation rule for characters that can be entered when setting up a static website
    * Up to 256 bytes, only alphanumeric characters and some special characters (- _ . /) allowed
* [Console] Changed the input restriction rule applied when copying objects
    * A path including subfolders can be entered
    * {Maximum length of the path} = 1024 - {Length of the object name} - 1

### February 23, 2021

#### Feature Updates
* [Console] Container settings improved
    * Feature improved to allow setting access policy, object life cycle, version control policy, state website, etc. in container unit

### November 24, 2020

#### Feature Updates
* [Console] Prefix search
    * Searches for containers, folders, or objects that begin with the prefix you entered

### February 25, 2020

#### Added Features
* [API] Provides Amazon S3 compatible API

### April 24, 2018

#### Feature Updates
* [Console] Restricts special characters for a folder name
    * Added slash (/) to the list of special characters not allowed for the name of container or folder

### October 26, 2017

#### Feature Updates
* [Console] Restricts special characters for a folder name
    * Modified not to allow some special characters (. .. & < > " ' ;) and space characters for the name of container or folder

### March 23, 2017

#### Feature Updates

* [Console] Specifies the size of uploadable files
    * Up to 5GB can be uploaded.

### December 22, 2016

#### Bug Fixes
* [Console] Fixed an issue where folders starting with "/" in the name were not properly shown when created
* [Console] Fixed an issue where files that are named the same as the folder name could not be deleted
* [Console] Fixed an issue where files including "#" as part of the title could not be downloaded
* [Console] Fixed an issue where the copy button on the pop-up for API Endpoint Setting did not work
* [Console] Fixed an issue where 0 byte files could not be downloaded

### December 8, 2016

#### Bug Fixes
* [Console] Modified the message guided when service is closed while resources remain

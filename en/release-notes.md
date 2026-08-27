<!-- machine_translated: true -->

<!-- pre-align:aligned sig=79f355249cbd -->

<a id="storage-object-storage-release-notes"></a>
## Storage > Object Storage > Release Notes { #storage-object-storage-release-notes }

<a id="august-25-2026"></a>
## August 25, 2026 { #august-25-2026 }
<a id="august-25-2026-added-features"></a>
### Added Features { #august-25-2026-added-features }
* [Console] Added features to view task history and stop tasks
    * Added features to view the history of empty container, copy/move/delete object tasks, and stop tasks.
* [API] Added the feature for container configuration using container policy documents
    * Added support for access policy, CORS, and object lock container settings.

<a id="august-25-2026-feature-updates"></a>
### Feature Updates { #august-25-2026-feature-updates }
* [Console][API] Added limits on the number of container configuration items
    * Up to 100 access policies (ACLs) can be configured per read, write, and query permission.
    * Up to 100 IP access control (IP ACL) entries can be configured for each of the allowed IP list and the blocked IP list.
    * CORS allowed origins and response headers to expose can each be configured up to 100.
    * Up to 30 lifecycle condition rules can be configured.
* [API] Improved Amazon S3 API compatibility
    * Added support for creating and configuring locked containers.

<a id="may-27-2026"></a>
## May 27, 2026 { #may-27-2026 }
<a id="may-27-2026-added-features"></a>
### Added Features { #may-27-2026-added-features }
* [Console] Added features to configure lifecycle rules and apply them in bulk
* [API] Added the feature for container configuration using container policy documents
    * Added support for configuring lifecycle rules.

<a id="may-27-2026-feature-updates"></a>
### Feature Updates { #may-27-2026-feature-updates }
* [API] Improved Amazon S3 API compatibility
    * Added support for domain-style (Virtual Hosted Domain) endpoints.
    * Added support for integrity verification for upload using Trailing Checksum.
    * Added support for CORS Preflight requests.

<a id="july-29-2025"></a>
## July 29, 2025 { #july-29-2025 }
<a id="july-29-2025-feature-updates"></a>
### Feature Updates { #july-29-2025-feature-updates }
* [API] Added a write request rate limit policy
    * A rate limiting policy is applied to write requests that exceed 500 requests per second, per storage account.

<a id="may-27-2025"></a>
## May 27, 2025 { #may-27-2025 }
<a id="may-27-2025-added-features"></a>
### Added Features { #may-27-2025-added-features }
* [Console] Added the feature to resume/suspend container replication

<a id="may-27-2025-feature-updates"></a>
### Feature Updates { #may-27-2025-feature-updates }
* [API] Improved Amazon S3 API compatibility
    * Fixed issue where the LastModified value of each objects in the object list was displayed to the nearest millisecond

<a id="may-27-2025-bug-fixes"></a>
### Bug Fixes { #may-27-2025-bug-fixes }
* [API] Fixed issue where multipart uploads to buckets with object locks using Amazon S3 compatible APIs would cause requests to fail and existing part objects to be deleted

<a id="august-27-2024"></a>
## August 27, 2024 { #august-27-2024 }
<a id="august-27-2024-feature-updates"></a>
### Feature Updates { #august-27-2024-feature-updates }
* [Console][API] Added service gateway IP access control settings
    * You can handle IP ACL exceptions for requests using the service gateway
* [Console] Added the feature for replication to a different project in the same organization
* [API] Improved Amazon S3 API compatibility
    * Improved response compatibility for the following errors
        * Request to create a bucket with an invalid name
        * Container request with an invalid path
        * Object request with an invalid path

<a id="may-28-2024"></a>
## May 28, 2024 { #may-28-2024 }
<a id="may-28-2024-added-features"></a>
### Added Features { #may-28-2024-added-features }
* [Console][API] Added Economy storage class - Korea (Pangyo) Region
    * You can select storage classes based on frequency of data access, cost requirements, etc.
* [Console][API] Added lifecycle expiration action
    * You can move objects whose lifecycle has expired to another container.

<a id="may-28-2024-feature-updates"></a>
### Feature Updates { #may-28-2024-feature-updates }
* [Console] Improved the feature to copy objects
    * You can select multiple objects to copy/move.
* [Console] Changed container naming conventions
    * Minimum of 3 characters and maximum of 63 characters allowed.
    * Lowercase letters, numbers, -, and . allowed.
    * IP-formatted names not allowed.

<a id="february-27-2024"></a>
## February 27, 2024 { #february-27-2024 }
<a id="february-27-2024-added-features"></a>
### Added Features { #february-27-2024-added-features }
* [Console][API] Added object upload policy configuration feature
* [Console] Added the feature to empty containers

<a id="february-27-2024-feature-updates"></a>
### Feature Updates { #february-27-2024-feature-updates }
* [Console] Improved the feature to delete objects

<a id="november-28-2023"></a>
## November 28, 2023 { #november-28-2023 }
<a id="november-28-2023-added-features"></a>
### Added Features { #november-28-2023-added-features }
* [API] Added support for AWS Signature V4 Chunked Upload to Amazon S3 compatible API
* [Console] Added the feature to create signed URLs

<a id="may-30-2023"></a>
## May 30, 2023 { #may-30-2023 }
<a id="may-30-2023-added-features"></a>
### Added Features { #may-30-2023-added-features }
* [Console][API] Added the IP ACL feature

<a id="march-28-2023"></a>
## March 28, 2023 { #march-28-2023 }
<a id="march-28-2023-feature-updates"></a>
### Feature Updates { #march-28-2023-feature-updates }
* [API] Changed API endpoints

<a id="march-28-2023-bug-fixes"></a>
### Bug Fixes { #march-28-2023-bug-fixes }
* [API] Fixed an issue where, when a segment object of a multipart object uploaded to PUBLIC containers is in another container, it could not be downloaded without a token
* [API] Fixed an issue where, when a multipart object with the same name is updated using the S3 compatible API, the previous part object could not be deleted 

<a id="november-29-2022"></a>
## November 29, 2022 { #november-29-2022 }
<a id="november-29-2022-added-features"></a>
### Added Features { #november-29-2022-added-features }
* [Console][API] Added a feature to set an object lock container

<a id="october-13-2022"></a>
## October 13, 2022 { #october-13-2022 }
<a id="october-13-2022-added-features"></a>
### Added Features { #october-13-2022-added-features }
* [Console] Added a feature to set a container ACL 

<a id="september-27-2022"></a>
## September 27, 2022 { #september-27-2022 }
<a id="september-27-2022-added-features"></a>
### Added Features { #september-27-2022-added-features }
* [Console] Added a feature to set container encryption
* [Console] Added a feature to set Cross-Origin Resource Sharing (CORS)

<a id="september-27-2022-feature-updates"></a>
### Feature Updates { #september-27-2022-feature-updates }
* [Console] Improved the UI of Get Container and Container Settings

<a id="august-23-2022"></a>
## August 23, 2022 { #august-23-2022 }
<a id="august-23-2022-added-features"></a>
### Added Features { #august-23-2022-added-features }
* [API] Added a feature to set the RFC compliant ETag format

<a id="march-29-2022"></a>
## March 29, 2022 { #march-29-2022 }
<a id="march-29-2022-added-features"></a>
### Added Features { #march-29-2022-added-features }
* [Console] Added the inter-region container replication feature

<a id="january-25-2022"></a>
## January 25, 2022 { #january-25-2022 }
<a id="january-25-2022-added-features"></a>
### Added Features { #january-25-2022-added-features }
* [Console] Added a feature to obtain S3 API credentials

<a id="january-25-2022-feature-updates"></a>
### Feature Updates { #january-25-2022-feature-updates }
* [Console] Applied batch deletion of segment objects when deleting multipart objects
* [API] Applied limit on characters that can be entered in container names
    * Some special characters (' " < > ;), spaces, and relative path characters (. ..) are not allowed

<a id="october-26-2021"></a>
## October 26, 2021 { #october-26-2021 }
<a id="october-26-2021-feature-updates"></a>
### Feature Updates { #october-26-2021-feature-updates }
* [Console] Changed the limitation rule for characters that can be entered when setting up a static website
    * Up to 256 bytes, only alphanumeric characters and some special characters (-, _, ., /) allowed.
* [Console] Changed the input restriction rule applied when copying objects
    * A path including subfolders can be entered.
    * {Maximum length of the path} = 1024 - {Length of the object name} - 1

<a id="february-23-2021"></a>
## February 23, 2021 { #february-23-2021 }
<a id="february-23-2021-feature-updates"></a>
### Feature Updates { #february-23-2021-feature-updates }
* [Console] Container settings improved
    * Feature improved to allow setting access policy, object lifecycle, version control policy, static website, etc. in container unit.

<a id="november-24-2020"></a>
## November 24, 2020 { #november-24-2020 }
<a id="november-24-2020-feature-updates"></a>
### Feature Updates { #november-24-2020-feature-updates }
* [Console] Prefix search
    * Searches for containers, folders, or objects that begin with the prefix you entered.

<a id="february-25-2020"></a>
## February 25, 2020 { #february-25-2020 }
<a id="february-25-2020-added-features"></a>
### Added Features { #february-25-2020-added-features }
* [API] Provides Amazon S3 compatible API

<a id="april-24-2018"></a>
## April 24, 2018 { #april-24-2018 }
<a id="april-24-2018-feature-updates"></a>
### Feature Updates { #april-24-2018-feature-updates }
* [Console] Restricts special characters for a folder name
    * Added slash (/) to the list of special characters not allowed for the name of container or folder.

<a id="october-26-2017"></a>
## October 26, 2017 { #october-26-2017 }
<a id="october-26-2017-feature-updates"></a>
### Feature Updates { #october-26-2017-feature-updates }
* [Console] Restricts special characters for a folder name
    * Modified not to allow some special characters (. .. & < > " ' ;) and space characters for the name of container or folder.

<a id="march-23-2017"></a>
## March 23, 2017 { #march-23-2017 }
<a id="march-23-2017-feature-updates"></a>
### Feature Updates { #march-23-2017-feature-updates }

* [Console] Specifies the size of uploadable files
    * Up to 5GB can be uploaded.

<a id="december-22-2016"></a>
## December 22, 2016 { #december-22-2016 }
<a id="december-22-2016-bug-fixes"></a>
### Bug Fixes { #december-22-2016-bug-fixes }
* [Console] Fixed an issue where folders starting with "/" in the name were not properly shown when created
* [Console] Fixed an issue where files that are named the same as the folder name could not be deleted
* [Console] Fixed an issue where files including "#" as part of the title could not be downloaded
* [Console] Fixed an issue where the copy button on the pop-up for API Endpoint Setting did not work
* [Console] Fixed an issue where 0 byte files could not be downloaded

<a id="december-8-2016"></a>
## December 8, 2016 { #december-8-2016 }
<a id="december-8-2016-bug-fixes"></a>
### Bug Fixes { #december-8-2016-bug-fixes }
* [Console] Modified the message guided when service is closed while resources remain

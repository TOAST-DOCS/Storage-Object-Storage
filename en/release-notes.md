## Storage > Object Storage > Release Notes

### October 26, 2021

#### Feature Updates
* Changed the limitation rule for characters that can be entered when setting up a static website
    * Up to 1024 bytes, only alphanumeric characters and some special characters (-, _, ., /) allowed
* Changed the input restriction rule applied when copying objects
    * A path including subfolders can be entered
    * {Maximum length of the path} = 1024 - {Length of the object name} - 1

### February 23, 2021
* Container settings improved
    * Feature improved to allow setting access policy, object life cycle, version control policy, state website, etc. in container unit

### November 24, 2020

#### Feature Updates
* Prefix search
    * Searches containers, folders, or objects that begin with the typed-in prefix

### February 25, 2020

#### More Features
* Provides APIs compatible with AWS S3

### April 24, 2018

#### Feature Updates
* Restricts special characters for a folder name
    * Added slash (/) onto the list of special characters not allowed for the name of container or folder

### Oct. 26, 2017

#### Feature Updates
* Restricts special characters for a folder name
    * Modified not to allow some special characters (e.g.., .., &, <, >, ", ', ;) and space characters for the name of container or folder

### March 23, 2017

#### Feature Updates

* Specifies the size of uploadable files
    * Up to 5GB can be uploaded.

### Dec. 22, 2016

#### Bug Fixes
* [Console] Fixed an issue in which folders starting with "/" in the name were not properly shown when created
* [Console] Fixed an issue in which files that are named the same as the folder name could not be deleted
* [Console] Fixed an issue where files including "#" as part of the title could not be downloaded
* [Console] Fixed an issue where the copy button on the pop-up for API Endpoint Setting did not work
* [Console] Fixed an issue where 0 byte files could not be downloaded

### Dec. 08, 2016

#### Bug Fixes
* [Console] Modified the message guided when service is closed while resources remain

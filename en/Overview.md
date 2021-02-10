## Storage > Object Storage > Overview

Object Storage refers to an online storage which can save large-capacity data.

## Features

### Scalable

Volume scale-up is almost limitless. Data can be saved whenever and as much as a user wants, without considering storage volume.

### Control Accessibility   

Access authority can be differently set for each container. Only allowed users can access a private container, whereas a public container can be accessed by anyone to internal objects, via public URL.

### Ensure Data Security

A total of three copies are created for each object to safely save data. Access to data is unavailable during copies are created, since integrity of copied data cannot be ensured. Hence, sending a request of GET right after data saving may result in failure response. If at least two copies are same, saving is completed as copying is considered to be successful, and access to data is allowed.    

### Provide Convenient Web Console

For user's convenience, web console is provided to manage data in the hierarchical directory structure. When an application approaches data, RESTful API is available as an object ID, while user can use the web console with familiar directory structure to approach data. You may use folders to manage objects under groups.

### Provide RESTful APIs

HTTP(s)-based REST API is provided to control containers and objects. With APIs, object storage becomes available on user's applications.


## Glossary
#### Object
Data to be saved, as a basic management unit of object storage: generally refers to a file.
#### Folder
A virtual unit that binds many objects: helps to manage objects in hierarchy, similar to folder of Windows or directory of Linux.
#### Container
Folder in the highest order, which serves as the basic unit for access authority setting. All objects must exist in a container.
#### Storage Account
Refers to user's account for object storage. NHN Cloud Object Storage is classified by the account unit.
#### API Endpoint
Refers to HTTP URL which is provided to access object storage via REST API. When requested with the URL, access is allowed to object storage.   

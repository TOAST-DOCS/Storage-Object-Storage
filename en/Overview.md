## Storage > Object Storage > Overview

Object Storage is an online storage service that can store large-capacity data.

## Features

### Scalable

Capacity can be expanded almost infinitely. Users can store as much data as they want when they need it, without worrying about storage capacity.

### Control Accessibility

Access permissions can be set per container. Containers set to private can only be accessed by authorized users. When a container is set to public, anyone can access the objects inside the container using a public URL.

### Ensure Data Reliability

To store data safely, a total of 3 copies of each object are created. While creating the copies, the integrity of the replicated data cannot be guaranteed and therefore the data is inaccessible. So, if you send a GET request immediately after storing data, you might get a failure response. If at least two copies are identical, the replication is considered successful, and the storing is completed and the data becomes accessible.

### Provide a Convenient Web Console

For user convenience, a web console is provided to manage data in a hierarchical directory structure. When an application accesses data, the RESTful API can be used with the object ID. When a user directly accesses data, they can use a web console with a familiar directory structure. You can manage objects by grouping them using folders.

### Provide RESTful API

HTTP(s)-based REST API is provided to control containers and objects. The API allows you to use object storage in your applications.


## Glossary
#### Object
The basic management unit of object storage, which refers to the data to be stored. Generally a file is used as an object.
#### Folder
A virtual unit that binds multiple objects. It helps to manage objects hierarchically, similar to folders in Windows or directories in Linux.
#### Container
Refers to the top-level folder. Basically, a container becomes the unit of access permission setting. All objects must exist in a container.
#### Storage Account
A user account for object storage. NHN Cloud Object Storage is isolated on a per-account basis.
#### API Endpoint
HTTP URL provided to access object storage with REST API. You can access object storage by making a request with the URL.

## Storage > Object Storage > Overview
Object storage is an object storage service that allows you to store various types of data as much as you want and import them whenever needed.

## Service Characteristics
### Outstanding Scalability
With unlimited capacity growth, you can store as much data as you need, when you need it, without having to consider the storage capacity.

### High Stability and Restoration Capability
To store data securely, store objects on multiple different hardware in duplicate. Even if some replicas have unintended problems, they are quickly restored from the intact replicas.

### Control Accessibility
Allows you to control access rights per container. You can set containers to public for anyone to view, and grant differential read/write permissions on a project or individual user basis by using Role-based access control (RBAC).

### Convenient Web Console
Provides a web console that allows you to manage data in a familiar hierarchical directory structure for user convenience. Allows you to create virtual folders within a container to group and manage objects.

### REST API
Provides an HTTP-based REST API for controlling containers and objects. By using REST API, you can use object storage in your application.

### Amazon S3-compatible API
Provides an API compatible with Amazon S3. S3-compatible API allows you to use Amazon Web Service Software Development Kit (SDK)-based applications and various third-party tools.

### Life Cycle Control
Provides a life cycle control feature on a container or individual object basis. Allows for efficient management of storage usage.

### Object Lock 
Provides an WORM (Write Once Read Many) feature to protect data from inadvertent overwriting and deletion requests by users.

### Version Management
Allows you to use a version management feature to manage the history of updating and deleting objects and restore them to previous versions.

### Disaster Recovery
The container replication feature allows you to replicate objects in a container to containers in different regions. You can prepare for unexpected disasters by using replicated data.

### Data encryption for Improved Security
The encryption feature from server-side can improve security by encrypting sensitive data. Symmetric keys used for encryption are managed securely by Secure Key Manager service of NHN Cloud.

### Convenient Cloud Accessibility
NHN Cloud's Service Gateway service provides access to object storage over a private network from instances within user VPCs isolated from external networks.

### Access History
Provides a history of access to object storage through NHN Cloud's CloudTrail service.

## Terms
#### Object
Basic data unit of object storage. Object consists of data, user-assigned metadata, and specific addresses.

#### Folder
A virtual unit that binds objects. Folder allows you to manage objects hierarchically, similar to folders in Windows or directories in Linux.

#### Container
Namespace for managing objects. Container serves as a unit of various management settings. All objects must exist in a container and have a name that is unique within the container.

#### Storage Account
User account for object storage. NHN Cloud object storage is isolated on an account basis.

#### API Endpoint
An HTTP URL provided to access object storage through REST API.
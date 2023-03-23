## Storage > Object Storage > Overview
Object storage is an object storage service that allows you to store various types of data as much as you want and import them whenever needed.

## Service Characteristics
### Outstanding Scalability
With unlimited capacity, you can store as much data as you need when needed without having to consider storage capacity.

### High Stability and Restoration Capability
Stores objects redundantly on different hardware to safely store data. Even if some replicas have unexpected problems, they are quickly restored from the intact replicas.

### Control Accessibility
You can control access rights per container. You can set containers to public for anyone to view, and Role-based access control (RBAC) enables you to grant differential read/write permissions to a project or individual user.

### Convenient Web Console
Provides a web console that allows you to manage data in a familiar hierarchical directory structure for user convenience. Allows you to create virtual folders within a container to group and manage objects.

### REST API
Provides HTTP-based REST APIs to control containers and objects. REST APIs allows you to use object storage in your application.

### Amazon S3-compatible API
Provides an API compatible with Amazon S3. S3-compatible API allows you to use Amazon Web Service Software Development Kit (SDK)-based applications and various third-party tools.

### Life Cycle Control
Provides a life cycle control feature on a container or individual object basis. This allows for efficient management of storage usage.

### Object Lock 
Provides an WORM (Write Once Read Many) feature to protect data from inadvertent overwriting and deletion requests by users.

### Version Management
The version management feature allows you to manage the history of updating and deleting objects and to restore the objects to their previous versions.

### Disaster Recovery
The container replication feature allows you to replicate objects in a container to another containers in different regions. Replicated data allows you to prepare for unexpected disasters.

### Data Encryption for Improved Security
The encryption feature from server-side can improve security by encrypting sensitive data. Symmetric keys used for encryption are managed securely by Secure Key Manager service of NHN Cloud.

### Convenient Cloud Accessibility
NHN Cloud's Service Gateway provides access to object storage over a private network from instances within user VPCs isolated from external networks.

### Access History
Provides a history of access to object storage through NHN Cloud's CloudTrail service.

## Terminology
#### Object
A basic data unit of object storage. It consists of data, user-assigned metadata, and unique addresses.

#### Folder
A virtual unit that binds objects. It allows you to manage objects hierarchically, similar to folders in Windows or directories in Linux.

#### Container
A namespace for managing objects. It serves as a unit for various management settings. All objects must exist in a container and have a name that is unique within the container.

#### Storage Account
A user account for object storage. NHN Cloud object storage is isolated on an account basis.

#### API Endpoint
An HTTP URL provided to access object storages through REST APIs.
## Storage > Object Storage > Console Guide


### Create Containers

To upload objects to object storage, at least one container is required.

* **Container Access Policy**
    * Private: Only allowed users can access objects within container.
    * Public: Anyone can access objects within container via public URL.
* **Storage Class**
    * Standard: Default

> [Note]
> Container name is limited to 256 in English, or 85 in Korean. 


### Delete Containers
Make sure the container is empty before it is deleted. Any remained objects in a container cannot be deleted.

> [Note]
> Objects that remain when folder is deleted shall not be deleted.

### Create Folders

Folder is a virtual unit to bind objects of object storage in groups. It helps to manage objects in the hierarchical order, similar to folder of Windows or directory of Linux.

> [Note]
> Folder name cannot exceed 256 characters in English or 85 in Korean: slash (/) serves as a delimiter between folders.


### Upload Objects

All objects must be uploaded to containers. One object cannot be larger than 5GB.

> [Note]
> Files exceeding 5GB cannot be uploaded in a web console.
> Objects exceeding 5GB must be split by using commands, like  `split`, or programmed to be divided into less than 5GB before uploaded.  
> For more details, refer to [Multipart Upload](api-guide/#multipart-upload) of API guide.

### Download Objects

For creating a container, if container access policy is set private, only allowed users can access objects. If it is set public, click **Actions > View Public URL** to check public URL of the object. This URL helps to create hyperlink of an object or directly download objects.   

**Example **

* Write Web Pages

```
# cat > index.html
<html>
<body> hello world!
<a href="https://kr1-api-object-storage.gov-nhncloudservice.com/v1/{account}/{container}/{object}">Download</a>
</body>
</html>
```

* Execute Web Servers

```
# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

Access through web browser and click **Download** to see if files are downloaded properly.


### Copy Objects
Copy objects to create  new objects. Create an object under a new name to the container which has an object to copy, or copy objects to another container.

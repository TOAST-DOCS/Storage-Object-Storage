## Storage > Object Storage > Console User Guide

<a id="container"></a>

## Container

<a id="create-container"></a>

### Create Container
Creates a container. You must have at least one container to upload objects to object storage. When encryption is configured, uploaded objects are automatically encrypted and stored.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="5">Create Container</td>
    <td>Name</td>
    <td>Container names must be between 3 and 63 characters long and can only contain lowercase letters, numbers, '-', '.', and '+'.<br/>Container names must start and end with letters or numbers.<br/>IP address format names cannot be used.</td>
  </tr>
  <tr>
    <td rowspan="2">Container Access Policy</td>
    <td><b>PRIVATE</b>: Only authorized users can access objects within the container.</td>
  </tr>
  <tr>
    <td><b>PUBLIC</b>: Anyone can access objects within the container through public URLs.</td>
  </tr>
  <tr>
    <td rowspan="2">Storage Class</td>
    <td><b>Standard</b>: The default class.</td>
  </tr>
  <tr>
    <td><b>Economy</b>: A class suitable for long-term storage of infrequently accessed data.</td>
  </tr>
  <tr>
    <td rowspan="2">Object Lock Settings</td>
    <td>Object Lock</td>
    <td>Select whether to use object lock.</td>
  </tr>
  <tr>
    <td>Lock Period</td>
    <td>Enter the object lock period in days.</td>
  </tr>
  <tr>
    <td rowspan="2">Encryption Settings</td>
    <td>Encryption</td>
    <td>Select whether to use object encryption.</td>
  </tr>
  <tr>
    <td>Symmetric Key ID</td>
    <td>Enter the symmetric key ID managed by the Secure Key Manager service.</td>
  </tr>
</table>

<a id="storage-class"></a>

### Storage Class
You can select a storage class based on data access frequency and cost requirements. We provide the Standard class for frequently accessed data and the Economy class for long-term storage of infrequently accessed data at a lower cost.

> [Note]
> The storage class of an already created container cannot be changed.
> Objects uploaded to Economy class containers have a minimum retention period of 30 days. Even for objects deleted before 30 days, charges are applied for the remaining retention period.
> Economy class containers are charged per 1,000 API requests (excluding HEAD/DELETE requests).

<a id="set-object-lock-cycle"></a>
#### Object Lock Configuration
Objects uploaded to object lock containers are stored using the **WORM (Write-Once-Read-Many)** model. Objects uploaded to object lock containers have a lock expiration date set. Objects cannot be overwritten or deleted before the lock expiration date set for each object.

<a id="set-object-encryption"></a>
#### Encryption Configuration
Objects uploaded to encrypted containers are encrypted using symmetric keys managed by NHN Cloud's Secure Key Manager service. Therefore, to create an encrypted container, you must first create a symmetric key in the Secure Key Manager service.

The policies applied to encrypted containers are as follows:

* Objects uploaded to encrypted containers are encrypted and stored using the configured symmetric key.
* When downloading encrypted objects, they are decrypted and transmitted.
* When copying objects from encrypted containers to other containers through copying or cross-region container replication, they are re-encrypted or decrypted and stored according to the target container's encryption settings.
* The symmetric key ID registered when creating an encrypted container cannot be changed. To change the symmetric key, you must use key rotation in the Secure Key Manager service.
* After rotating the symmetric key configured for an encrypted container in the Secure Key Manager service and uploading new objects, objects encrypted with the previous version key are re-encrypted with the rotated key. This operation may take a long time depending on usage. Be careful not to delete the previous version key before re-encryption is completed.

> [Caution]
If you delete the symmetric key configured for an encrypted container in the Secure Key Manager service, encrypted objects cannot be decrypted. You must carefully manage the symmetric key to avoid accidental deletion.

<a id="empty-a-container"></a>

### Empty Container
Deletes all objects inside the selected container.

> [Note]
> Objects whose lock expiration date has not passed will not be deleted.
> For multipart objects inside the selected container, only the manifest object is deleted. Segment objects located in other containers are not deleted.

> [Caution]
> If replication settings are in use, objects in the target container may also be deleted.
> Objects uploaded to a container while a container emptying operation is in progress may be deleted.

<a id="delete-container"></a>

### Delete Container
Deletes the selected container. Before deleting a container, you must verify that the container is empty. If objects remain in the container, it will not be deleted.

<a id="manage-container"></a>

### Container Management
View basic information about the selected container and manage its settings.

<a id="container-basic-info"></a>
#### Basic Information
You can view the basic information and encryption details of the container, and modify settings such as access policies, static website configuration, and cross-origin resource sharing.
<br/>

<a id="set-container-access-policy"></a>
##### Container Access Policy

Set default access policies and manage role-based access policies by tenant or individual user. For detailed information about access policies, refer to the [Access Policy Configuration Guide](acl-guide/).

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Default Access Policy</td>
    <td>PRIVATE</td>
    <td>Only authorized users can access objects inside the container.</td>
  </tr>
  <tr>
    <td>PUBLIC</td>
    <td>Anyone can access objects inside the container through public URLs.</td>
  </tr>
  <tr>
    <td rowspan="4">Role-based Access Policy Settings</td>
    <td></td>
    <td>Select whether to use individual access policies.</td>
  </tr>
  </tr>
  <tr>
    <td>Tenant ID</td>
    <td>Enter the Tenant ID or <code>*</code> to allow access. The Tenant ID can be found in the API endpoint settings dialog in the console.</td>
  </tr>
  <tr>
    <td>API User ID</td>
    <td>Enter the API User ID or <code>*</code> to allow access. The API User ID can be found in the API endpoint settings dialog in the console.</td>
  </tr>
  <tr>
    <td>Permission</td>
    <td>Select the access permissions to allow (<code>Read</code>, <code>Write</code>, <code>View</code>).</td>
  </tr>
</table>

<br/>

<a id="set-container-ip-acl"></a>
##### IP ACL

Manage IP-based access policies. For detailed information about access policies, refer to the [Access Policy Configuration Guide](acl-guide/).

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Whitelist</td>
    <td>IPv4</td>
    <td>Enter the IP to register in the whitelist. You can enter it in IP (192.168.0.1) or CIDR (192.168.0.0/24) format.</td>
  </tr>
  <tr>
    <td>Permission</td>
    <td>Select the access permissions to allow (Read, Write, View).</td>
  </tr>
  <tr>
    <td rowspan="2">Blacklist</td>
    <td>IPv4</td>
    <td>Enter the IP to register in the blacklist. You can enter it in IP (192.168.0.1) or CIDR (192.168.0.0/24) format.</td>
  </tr>
  <tr>
    <td>Permission</td>
    <td>Select the access permissions to deny (Read, Write).</td>
  </tr>
  <tr>
    <td rowspan="5">Service Gateway IP</td>
    <td>Not Set</td>
    <td>Do not apply access control settings for requests through the service gateway.</td>
  </tr>
  <tr>
    <td>Allow Read</td>
    <td>Allow read requests through the service gateway.</td>
  </tr>
  <tr>
    <td>Allow Write</td>
    <td>Allow write requests through the service gateway.</td>
  </tr>
  <tr>
    <td>Allow Read / Write</td>
    <td>Allow both read and write requests through the service gateway.</td>
  </tr>
  <tr>
    <td>Block</td>
    <td>Block all requests through the service gateway.</td>
  </tr>
</table>

<a id="set-container-static-website"></a>
##### Static Website Configuration

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Static Website Settings</td>
    <td>Index Document</td>
    <td>Enter the index document object for the static website. If the object is inside a folder, you must include the folder path.<br/>Maximum 256 bytes, only English letters, numbers, and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
  <tr>
    <td>Error Document</td>
    <td>Enter the suffix for the error document object of the static website. The error document suffix cannot include folder paths.<br/>Maximum 256 bytes, only English letters, numbers, and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
</table>

By setting the container's access policy to **PUBLIC** and entering index and error documents, you can host a static website from the container. The static website URL can be obtained by clicking the **Copy URL** button in the container list.

Objects used as index documents or error documents for static websites must have names consisting of one or more English letters, numbers, or some special characters (`-`, `_`, `.`, `/`), and must be in hypertext format with a `.html` file extension. If they don't meet these conditions, you may not be able to configure them or the static website may not function properly.

The error document name for static websites is in the format `{response code}{suffix}`. For example, if you set the error document to `error.html`, the error document name shown for a 404 error would be `404error.html`. You can upload and use error documents for each error situation. If an error document is not defined or there is no error document object matching the response code, the web browser's default error document will be displayed.
<br/>

<a id="set-container-cors"></a>
##### Cross-Origin Resource Sharing (CORS) Configuration

To directly call Object Storage APIs from a browser, cross-origin resource sharing configuration is required. Click the change button for the cross-origin resource sharing item to register allowed origin URLs. URLs must include the protocol (`https://` or `http://`). Registering `*` allows all origins.

<br/>

<a id="set-container-upload-policy"></a>
##### Upload Policy Configuration
Set object name-based upload policies for the container. Using upload policy settings, you can restrict uploads to only objects with specific extensions or keywords in their names, or prevent such uploads.

Upload policies can be set as either `whitelist` or `blacklist`, but both methods cannot be set simultaneously. You can set file extensions or keywords to be included in filenames for upload. However, for objects with paths, only the object name excluding the path is applied to the policy. Upload policies apply to newly uploaded objects after they are configured.

Setting a whitelist with extensions `exe` and `jpg` allows only objects with `.exe` and `.jpg` extensions to be uploaded. Adding filename `example` allows uploads only for objects that have both the configured filename and extensions, such as `exe_example.exe` and `image_example.jpg`.

For blacklists, setting extensions `exe` and `jpg` prevents all objects with `.exe` and `.jpg` extensions from being uploaded. Adding filename `example` prevents uploads of both files with restricted extensions like `test.exe` and `image.jpg`, as well as files with restricted keywords like `text_example.txt`.
<br/>

<a id="set-object-lifecycle"></a>
#### Lifecycle

You can view and modify lifecycle rules for objects stored in the container.
For detailed information about lifecycle configuration, refer to [How to Apply Lifecycle Rules](container-policy-guide/#lifecycle-apply).
<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Type</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan=3>Default Rule</td>
    <td>Object Lifecycle</td>
    <td>Enter the object lifecycle in days.</td>
  </tr>
  <tr>
    <td>Lifecycle Expiration Action</td>
    <td>Select how to handle objects when their lifecycle expires.</td>
  </tr>
  <tr>
    <td>Target Container</td>
    <td>If you select <b>Move Container</b> as the lifecycle expiration action, you must select the container to move objects to.</td>
  </tr>
  <tr>
    <td rowspan=5>Conditional Rule</td>
    <td>Rule Name</td>
    <td>Enter the name of the lifecycle rule.</td>
  </tr>
  <tr>
    <td>Condition</td>
    <td>Specify the condition for applying the rule.</td>
  </tr>
  <tr>
    <td>Object Lifecycle</td>
    <td>Enter the object lifecycle in days.</td>
  </tr>
  <tr>
    <td>Lifecycle Expiration Action</td>
    <td>Select how to handle objects when their lifecycle expires.</td>
  </tr>
  <tr>
    <td>Target Container</td>
    <td>If you select <b>Move Container</b> as the lifecycle expiration action, you must select the container to move objects to.</td>
  </tr>
</table>

> [Note]
> Object lifecycle settings only apply to objects uploaded after the configuration is set.
> You can move objects from Standard class containers to Economy class containers according to their lifecycle to reduce long-term storage costs.

<a id="set-object-lifecycle-batch"></a>
##### Batch Rule Application

Clicking the `Batch Rule Application` button allows you to batch reset the lifecycle of all objects in the container according to the rules.
Rules are applied in order of priority, and lifecycles are recalculated based on the batch application time.

<a id="set-object-versioning"></a>
#### Object Versioning

Through object versioning settings, you can preserve previous versions of objects. When objects are updated or deleted, previous versions are stored in the designated archive container. By setting a lifecycle for previous versions, those that exceed the lifecycle are automatically deleted.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Versioning Policy</td>
    <td>Select whether to use the versioning policy.</td>
  </tr>
  <tr>
    <td>Archive Container</td>
    <td>Enter the container to store previous versions of objects.</td>
  </tr>
  <tr>
    <td>Archived Object<br/>Lifecycle</td>
    <td>Enter the lifecycle for previous versions of objects in days. Leave blank to disable lifecycle settings.</td>
  </tr>
</table>

> [Warning]
> If you delete the archive container before the original container, errors will occur when updating or deleting objects in the original container. If already deleted, you can resolve this by creating a new archive container or disabling the versioning policy for the original container.
> If you designate an encrypted container as an archive container and then delete the symmetric key of the encrypted container from the Secure Key Manager service, object upload and deletion in the original container will fail.


<a id="change-object-lock-cycle"></a>
#### Object Lock

You can view and modify the object lock period for object lock containers. The lock period can be entered in days and cannot be disabled.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Object Lock Settings</td>
    <td>Object Lock</td>
    <td>Select whether to use object lock.</td>
  </tr>
  <tr>
    <td>Lock Period</td>
    <td>Enter the object lock period in days.</td>
  </tr>
</table>

> [Note]
> The modified object lock period applies to objects uploaded after the setting change.
> You cannot convert a regular container to an object lock container, or an object lock container to a regular container.
> Object lock containers cannot be designated as archive containers or replication target containers.

<a id="set-container-replication"></a>
#### Replication

Through replication settings, you can replicate container objects to containers in other regions. Replication is a feature for disaster recovery that identically replicates and manages objects from the source region to the target region. Replication runs in the background at regular intervals.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="6">Replication Settings</td>
    <td>Replication</td>
    <td>Select whether to use the replication feature.</td>
  </tr>
  <tr>
    <td rowspan="2">Project Classification</td>
    <td><b>Same Project</b>: Select a container from the same project as the replication target container.</td>
  </tr>
  <tr>
    <td><b>Different Project</b>: Select a container from a different project as the replication target container.</td>
  </tr>
  <tr>
    <td>Target Project</td>
    <td>Enter the replication target project. Click the search button to check the project's permissions.</td>
  </tr>
  <tr>
    <td>Target Region</td>
    <td>Select the replication target region. If project classification is same project, the currently used region is excluded.</td>
  </tr>
  <tr>
    <td>Target Container</td>
    <td>Enter the replication target container or click the browse button to select it.</td>
  </tr>
</table>

Replication policies are as follows:

* **Basic Behavior**
    * When objects are modified (uploaded, metadata updated, deleted) in the source container, the same changes are reflected in the replication target container.
    * Objects modified in the replication target container are not reflected in the source container.
    * Replication operates based on the object's last modification time. If an object in the replication target container was modified later than the source, it will not be replicated.
* **Configuration Considerations**
    * It is recommended to use an empty container as the replication target container. If the target container already has objects with the same names as objects in the source container, replication may not work smoothly.
    * If there is a history of objects with the same name being deleted in the replication target container, the recent update time of replicated objects may be changed to the replication configuration time.
    * It is recommended to set the source container and replication target container names identically. If container names are different, access to replicated large objects may fail.
    * If segment objects of large objects uploaded via multipart upload are stored in other containers, those containers containing segments must also be configured for replication in the same way to access replicated large objects.
* **Configuration Change Considerations**
    * Changing replication settings to disabled stops replication but maintains already replicated objects.
    * Switching replication direction restores objects from the replication target container to the source container. Objects deleted from the source container are also included in the restoration, and the recent update time of restored objects is changed to the replication configuration time.
    * If you delete the replication target container, replication will not resume even if you create a container with the same name again. To resume replication, you must reconfigure replication settings.
* **Limitations and Exceptions**
    * You cannot replicate the replication target container to another region or set it as a replication target container for other containers.
    * Objects whose lifecycle has expired but have not yet been deleted have their lifecycle settings removed when replicated to the target container. When they are subsequently deleted from the source container, the deletion is propagated to and executed in the target container.
    * Delete marker objects in archive containers are not replicated.

> [Warning]
> If you designate an encrypted container as a replication target container and then delete the symmetric key of the encrypted container from the Secure Key Manager service, replication to the encrypted container will fail.
<br/>

<a id="resume-container-replication"></a>
##### Start Replication

Resume stopped container replication. Replication continues from the point where it was interrupted.
<br/>

<a id="suspend-container-replication"></a>
##### Stop Replication

Stop container replication. While replication is stopped, objects deleted or modified in the source container are not replicated.

> [Warning]
> Objects deleted from the source container during the replication stop period may not be reflected in the target container.
<br/>

<a id="object"></a>

## Object

<a id="create-folder"></a>
### Create Folder
Creates a folder. A folder is a virtual unit for grouping objects within a container. It helps manage objects hierarchically, similar to folders in Windows or directories in Linux. Folder names are limited to 256 English characters or 85 Korean characters.

> [Note]
> Folders in Object Storage are different from directories provided by file systems. They are pseudo folders provided for convenience. When you create a folder, an empty object named `{folder-name}/` is actually created. Objects inside the folder have names in the format `{folder-name}/{object-name}`. When you copy objects to a new folder using the object copy feature, objects in the format `{folder-name}/{object-name}` are created directly without the empty `{folder-name}/` object. Therefore, if you delete this copied object, it appears as if the folder is also deleted. When copying to a pre-created folder, the folder remains even if you delete objects within the folder.

<a id="delete-folder"></a>
### Delete Folder
Deletes a folder. Deletes all objects within the folder and the folder object itself.
For multipart objects within the folder, only the manifest object is deleted, and segment objects not included in the selection are not deleted.

<a id="upload-object"></a>
### Upload Object
All objects must be uploaded to a container. The maximum capacity of a single object is limited to 5GB.

> [Note]
> Files exceeding 5GB cannot be uploaded through the web console. If the object you want to upload exceeds 5GB, you must divide it using command-line tools like `split`, or program your user application to divide and upload files in sizes of 5GB or less. For detailed usage instructions, refer to [Multipart Upload](api-guide/#multipart-upload) in the API guide.

<a id="download-object"></a>
### Download Object
Downloads the selected object. If you set the container access policy to **PRIVATE** when creating the container, only authorized users can access the object. If you set it to **PUBLIC**, you can check the object's public URL by clicking the `Copy URL` button in the list. You can create hyperlinks to the object or download the object directly using this URL.

<details style="padding-top: 15px; padding-bottom: 10px;">
<summary>Hyperlink Example</summary>
<ul style="padding-left: 10px; padding-top: 10px;">
<li>Creating a web page</li>

```
# cat > index.html
<html>
<body> hello world!
<a href="https://kr1-api-object-storage.nhncloudservice.com/v1/{account}/{container}/{object}">Download</a>
</body>
</html>
```

<li>Running a web server using Python3's http module</li>
```
# python -m http.server
Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

<li>Access <b>http://localhost:8000</b> through a web browser and click <b>Download</b> to verify that the file downloads normally</li>

</details>

<a id="copy-or-move-object"></a>
### Copy/Move Object
Copies or moves objects to the specified container. You can select multiple objects and copy or move them to another container or to a new path within the same container.

> [Note]
> The maximum length of the path you can enter varies depending on the object name. The combined length of the copy path and object name must be 1024 bytes or less.
> `{Maximum path length} = 1024 - {Object name length} - 1`
>
> For multipart objects, only the manifest object is copied or moved.

<a id="delete-object"></a>
### Delete Object
Deletes the selected object. You can select and delete multiple objects simultaneously.

> [Note]
> When deleting a multipart object, only the selected manifest object is deleted. Segment objects that are not selected are not deleted.

<a id="create-signed-url"></a>
### Create Signed URL
Creates a signed URL that allows free access to the specified object for a set period of time, regardless of role-based access policies.

> [Note]
> Only a single object can be selected, and folder objects cannot be selected.
> The validity period can be set up to 720 minutes in minute units.

> [Caution]
> If a signed URL is leaked, anyone can access the selected object, so it must be used carefully. It is recommended to set an appropriate validity period based on the situation and purpose to minimize damage even if the signed URL is leaked.

<a id="manage-object"></a>
### Manage Object
Checks basic information of the selected object and manages its properties.

> [Note]
> If you set both object expiration date and lock expiration date, the object expiration date must always be set after the lock expiration date.

<a id="set-object-expiration"></a>
#### Change Object Expiration Date

You can change the expiration date of the selected object.

<a id="set-object-lock-expiration"></a>
#### Change Object Lock Date

You can change the lock expiration date of the selected object. It cannot be changed to a date earlier than the previously set expiration date.

<a id="prefix-search"></a>

## Prefix Search
By entering a prefix in the search box and clicking the **Search** button, you can search for containers, folders, and objects that start with the entered prefix. In the container list, it searches for containers, and in the object list, it searches for folders and objects.

<a id="s3-api-credentials"></a>

## S3 API Credentials
You can obtain credentials for using the Amazon S3-compatible API. S3 API credentials have no expiration date, and each user can obtain up to 3 credentials per project.

> [Caution]
> If S3 API credential keys are leaked, anyone can use the leaked keys to access objects. If keys are leaked, it is recommended to delete the leaked credentials and issue new ones.
## Storage > Object Storage > Console Guide

## Container
### Create Container
Creates containers. Uploading objects in an object storage requires one or more containers. If you set encryption, the uploaded object is automatically encrypted and saved.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="4">Create Container</td>
    <td>Name</td>
    <td>Name of a container is limited to 256 characters in English or 85 characters in Korean.</td>
  </tr>
  <tr>
    <td rowspan="2">Container access policy</td>
    <td><b>PRIVATE</b>: Only permitted users can access objects within a container.</td>
  </tr>
  <tr>
    <td><b>PUBLIC</b>: Anyone with a public URL can access objects within a container.</td>
  </tr>
  <tr>
    <td>Storage class</td>
    <td><b>Standard</b>: This is the default class.</td>
  </tr>
   <tr>
    <td rowspan="2">Object lock settings</td>
    <td>Object lock</td>
    <td>Select whather to use the obejct lock.</td>
  </tr>
  <tr>
    <td>Lock cycle</td>
    <td>Enter the object lock cycle in days.</td>
  </tr>
  <tr>
    <td rowspan="2">Encryption settings</td>
    <td>Encryption</td>
    <td>Select whether to use object encryption.</td>
  </tr>
  <tr>
    <td>Symmetric key ID </td>
    <td>Enter the symmetric key ID managed by the Secure Key Manager service.</td>
  </tr>
</table>

#### Object Lock Settings
Objects uploaded to the Object Lock container are stored using the **WORM (Write-Once-Read-Many)** model. For objects uploaded to the object lock container, the lock expiration date is configured. You cannot overwrite or delete objects before the lock expiration date set on each object.

#### Encryption Settings
Objects uploaded to encryption containers are encrypted using a symmetric key managed by the NHN Cloud's Secure Key Manager service. Therefore, in order to create an encryption container, you must create a symmetric key in the Secure Key Manager service in advance.

The policies for encryption container are as follows.

* Objects uploaded to encryption containers are encrypted and stored using the configured symmetric key.
* If you download the encrypted object, it is sent after being decrypted. 
* If you copy an object of the encryption container or copy it to another container through the inter-region container replication, the object is stored re-encrypted or decrypted according to the encryption settings for the container.
* You cannot change the symmetric key ID that is registered when creating an encryption container. To change the symmetric key, you must use the key rotation feature of Secure Key Manager.
* If you rotate the symmetric key configured in an encryption container from Secure Key Manager and upload the key to a new object, the object encrypted with the previous version key is re-encrypted with the rotated key. This task can take a long time depending on usage. Be cautious not to delete the previous version key before re-encryption is complete.

> [Caution]
If you delete the symmetric key configured in an encryption container from Secure Key Manager, the encrypted object cannot be decrypted. You must carefully manage the symmetric key not to delete it accidentally.

### Delete Container
Deletes selected containers. Check if the containers are empty before deleting them. If any objects are left inside a container, you cannot delete the relevant container.

### Manage Container
Checks basic information of the selected containers and manage the settings.

#### Basic Information
You can view the container's basic and encryption information, and change settings such as access policies, static websites, and cross-origin resource sharing.
<br/>

##### Container Access Policy

Sets the basic access policy and manages role-based access policies for each tenant or user. For more details, refer to [ACL Configuration Guide](acl-guide/).

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Basic Access Policy</td>
    <td>PRIVATE</td>
    <td>Only permitted users can access objects within a container.</td>
  </tr>
  <tr>
    <td>PUBLIC</td>
    <td>Anyone can access objects within a container through a public URL.</td>
  </tr>
  <tr>
    <td rowspan="4">ACL settings</td>
    <td></td>
    <td>Selects whether to use an access policy.</td>
  </tr>
  </tr>
  <tr>
    <td>Tenant ID</td>
    <td>Enter the tenant ID or <code>*</code> to allow access. You can check the tenant ID in the API Endpoint setting dialog box on the console.</td>
  </tr>
  <tr>
    <td>API User ID</td>
    <td>Enter the tenant API user ID or <code>*</code> to allow access. You can check the API user ID in the API Endpoint setting dialog on the console.</td>
  </tr>
  <tr>
    <td>Permission</td>
    <td>Select access permissions (<code>Read</code>, <code>Write</code>) to allow.</td>
  </tr>
</table>

<br/>

##### IP ACL

Manages IP-based access policies. For more details, refer to [ACL Configuration Guide](acl-guide/).

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Whitelist</td>
    <td>IPv4</td>
    <td>Enter an IP to register in the whitelist. You can enter in IP (192.168.0.1) or CIDR (192.168.0.0/24) format.</td>
  </tr>
  <tr>
    <td>Access right</td>
    <td>Select access rights (Read, Write) to allow.</td>
  </tr>
  <tr>
    <td rowspan="2">Blacklist</td>
    <td>IPv4</td>
    <td>Enter an IP to register in the blacklist. You can enter in IP (192.168.0.1) or CIDR (192.168.0.0/24) format.</td>
  </tr>
  </tr>
  <tr>
    <td>Access right</td>
    <td>Select access rights not to allow (Read, Write).</td>
  </tr>
</table>


##### Static Website Settings

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Static Website Settings</td>
    <td>Index document</td>
    <td>Enter index document objects of a static website. If the object is within a folder, the folder path must be included.<br/>Up to 256 bytes, only alphanumeric characters and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
  <tr>
    <td>Error document</td>
    <td>Enter the suffix of an error document of a static website. A folder path cannot be included in the suffix of the error document.<br/>Up to 256 bytes, only alphanumeric characters and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
</table>

If you set the access policy of a container to **PUBLIC** and enter the index document and error document, you can host a static website in the container. You can get the URL of the static website by clicking the **Copy URL** button on the container list.

The name for an object to be used as an index document or error document for a static website must consist of one or more alphanumeric characters, or some special characters(`-`, `_`, `.`, `/`), and the file extension must be `html` in hypertext format. If the conditions are not satisfied, you cannot configure the settings or the static website may not work.

The name for an error document of a static website has the form of `{error code}{suffix}`. For example, if you configure the error document as `error.html`, the name for an error document to display when a 404 error occurs is `404error.html`. You can upload and use error documents for each error situation. If error documents are not defined or error objects that matches error codes do not exist, a default error document of a web browser will be displayed.
<br/>

##### Change Cross-Origin Resource Sharing (CORS)

To call the Object Storage API directly from the browser, you need to set Cross-Origin Resource Sharing (CORS). You can register the source URLs to allow by clicking the Change button of the cross-origin resource sharing item. The URL must include the protocols (`https://` or `http://`). You can allow all source URLs by entering `*`.

<br/>

#### Life Cycle and Version Management

Views the life cycle of objects stored in containers and version control policies.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Object life cycle</td>
    <td></td>
    <td>Enter the object life cycle in days. Life cycle setting will be cleared if it is not filled in.</td>
  </tr>
  <tr>
    <td rowspan="3">Object<br/>version control policy</td>
    <td>Version control policy</td>
    <td>Select whether to use a version control policy.</td>
  </tr>
  <tr>
    <td>Archive container</td>
    <td>Enter a container in which previous versions of an object are stored.</td>
  </tr>
  <tr>
    <td>Archiving object<br/>life cycle</td>
    <td>Enter the life cycle of previous versions of an object in days. Life cycle setting will be cleared if it is not filled in.</td>
  </tr>
</table>

##### Object Life Cycle

You can set a life cycle of the uploaded object. Objects that have passed the set life cycle are automatically deleted.

> [Note]
> It is applied only to objects uploaded after the object life cycle is set.

##### Object Version Control Settings

Object version control settings allow you to keep previous versions of objects. Previous versions are kept in the archive container when the object is updated or deleted. If you set the life cycle for previous versions, versions that exceed the set life cycle are automatically deleted.

> [Caution]
> If the archive container is deleted before the original container, an error occurs when updating or deleting objects in the original container. If the archive container has already been deleted, you can solve the issue by creating a new archive container or disabling the original container's version control policy.
> If you specify an encryption container as the archive container and then delete the symetric key from Secure Key Manager, the object of the original container fails to be uploaded and deleted.

#### Object Lock

You can check and change the object lock cycle of object lock containers. The object lock cycle can be entered in days, and cannot be turned off.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Object lock settings</td>
    <td>Objecy lock</td>
    <td>Select whether to use the object lock.</td>
  </tr>
  <tr>
    <td>Lock cycle</td>
    <td>Enter the object lock cycle in days.</td>
  </tr>
</table>

> [Note]
> The changed object lock cycle is applied to obejcts uploaded after changing the settings. 
> You cannot change a general container to an object lock container and vice versa.
> You cannot specify an object lock container as an archive container or replication target containger.

#### Replication

Replication settings allow you to replicate objects in a container to another container in a different region. Replication settings are for disaster recovery, and objects in the source region are replicated to the target region and managed. Replication proceed in the background at regular intervals.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="3">Replication settings</td>
    <td>Replication</td>
    <td>Select whether to use the replication feature.</td>
  </tr>
  <tr>
    <td>Target region</td>
    <td>Select a region to replicate to other than the currently used region.</td>
  </tr>
  <tr>
    <td>Target container</td>
    <td>Enter the target container for replication.</td>
  </tr>
</table>

The replication policies are as follows:

* Changes (upload, update, deletion) of objects in the source container are reflected identically in the target container.
* Changes made to the object in the target container are not reflected in the source container.
* If a change is made to a replicated object in the target container, that object may not be replicated even if there are subsequent changes to the source object.
* It is recommended to use an empty container for the target container. If the target container already has an object with the same name as the object in the source container, replication may not be performed properly.
* If the target container is deleted, replication will not resume even if you create a container with the same name again. To resume replication, you must set up replication again.
* The target container cannot be replicated to another region, and it cannot be set as another container's target container in duplicate.
* Changing the replication setting to **Disable** stops replication, but the objects that have already been replicated are maintained.
* It is recommended to set the name of the source container and the target container to be the same. If the container names are different, access to large replicated objects may fail.
* If the segment objects of the multipart-uploaded large object are stored in another container, you must also set up the replication for that container to access the replicated large object.
* Delete marker objects of the archive container are not replicated.

> [Caution]
> If you specify an encryption container as the replication target container and then delete the symmetric key from Secure Key Manager, the encryption container fails to be replicated.

## Object
### Create Folder
Create folders. Folders are virtual units to bundle objects within a container into a group. Similar to folders in Windows or directories in Linux, they help users to manage objects hierarchically. Folder names are limited to 256 letters in English or 85 characters in Korean.

> [Note]
> Folder for object storage is different from the directory provided by the file system. It is a pseudo folder provided for user's convenience. When a folder is created, an empty object named `{folder-name}/` is created. Objects within the folder will have names in the form of `{folder-name}/{object-name}`. Objects in the form of `{folder-name}/{object-name}` can be created directly without generating empty objects in the form of `{folder-name}/` by using the Copy Object function to copy objects into a new folder. If this copied object is deleted, it will appear as if the folder is also deleted. If you copy the object to a folder that you created in advance, the folder remains even if the object is deleted.

### Delete Folder
Delete folders. Check if the folders are empty before deleting them. If any objects are left inside a folder, you cannot delete the relevant folder.

### Upload Object
All objects must be uploaded to containers. One object cannot be larger than 5GB.

> [Note]
> Files exceeding 5GB cannot be uploaded in a web console. If the size of the object to be uploaded exceeds 5GB, it must be split by using a command-line tool such as `split`, or the user application must be programmed to divide the object into segments less than 5GB before uploading. For more details, refer to [Multipart Upload](api-guide/#multipart-upload) of the API guide.

### Download Object
Download selected objects. If you have set up the container access policy as **PRIVATE** at the time of creation, only permitted users can access the objects. If the access policy was set up as **PUBLIC**, click the `Copy URL` button on the list to check the public URL of the object. With this URL, it is possible to create a hyperlink of the object or directly download it.

<details style="padding-top: 15px; padding-bottom: 10px;">
<summary>Hyperlink Example</summary>
<ul style="padding-left: 10px; padding-top: 10px;">
<li>Write Web Page</li>

```
# cat > index.html
<html>
<body> hello world!
<a href="https://kr1-api-object-storage.nhncloudservice.com/v1/{account}/{container}/{object}">Download</a>
</body>
</html>
```

<li>Run web server using http module of Python3</li>
```
# python -m http.server
Serving HTTP on :: port 8000 (http://[::]:8000/) ...
```

<li>After accessing <b>http://localhost:8000</b> through a web browser click <b>Download</b> to confirm file is being downloaded properly</li>

</details>


### Copy Object
Copy objects to create new objects. Create an object with a new name in the container which has an object to copy, or copy objects to another container.

> [Note]
> The maximum length of the path that can be entered depends on the length of the object name. The length of the path to copy plus the object name must be 1024 bytes or less.
> `{Maximum length of the path} = 1024 - {Length of the object name} - 1`
>
> Multipart objects larger than 5 GB cannot be copied. 

### Delete Object
Delete selected objects. When a multipart object is deleted, the segment object is also deleted.

### Create Signed URL
Create a signed URL that allows free access to the specified object for the time you set, regardless of role-based access policies.

> [Note]
> Only single objects can be selected, not folder objects.
> The validity period can be set in minutes, up to 720 minutes.

> [Caution]
> Signed URLs should be used with caution because if they are exposed, anyone can access the selected object. It is recommended that you set an appropriate validity period for your situation and use it to reduce the damage if your signed URL is exposed.


### Manage Object
Check the selected object information and manage the properties.

> [Note]
> If you set both an object expiration date and a lock expiration date, the object expiration date must always be set after the lock expiration date.

##### Change Object Expriation Date

You can change the expiration date for selected objects.

#### Change Object Lock Date

You can change the lock expiration date for selected objects. It cannot be changed prior to the previously set expiration date.


## Prefix Search
If you enter a prefix in the search bar and click the **Search** button, you can search for containers, folders, and objects that begin with the prefix you entered. You can search for containers in the container list, and search for folders and objects in the object list.

## S3 API Credentials
You can obtain credentials required to use Amazon S3 compatible API. S3 API credentials have no expiration date, and up to 3 credentials can be issued per project for each user.

> [Caution]
> If the S3 API credentials key is leaked, anyone can access the object using the leaked key. If the key is leaked, it is recommended to delete the leaked credentials and obtain a new one.


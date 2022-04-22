## Storage > Object Storage > Console Guide

## Container
### Create Container
Creates containers. Uploading objects in an object storage requires one or more containers.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
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
</table>

### Delete Container
Delete selected containers. Check if the containers are empty before deleting them. If any objects are left inside a container, you cannot delete the relevant container.

### Container Details
Check detailed information of selected container. Information including basic information and settings of a container are available.

### Container Settings
Change settings of a selected container.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="2">Basic Settings</td>
    <td>Container access policy</td>
    <td>Select the container access policy between <b>PRIVATE</b> and <b>PUBLIC</b>. </td>
  </tr>
  <tr>
    <td>Object life cycle</td>
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
  <tr>
    <td rowspan="5">Static website settings</td>
    <td>Index document</td>
    <td>Enter index document objects of a static website. If the object is within a folder, the folder path must be included.<br/>Up to 1024 bytes, only alphanumeric characters and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
  <tr>
    <td>Error document</td>
    <td>Enter the suffix of an error document of a static website. A folder path cannot be included in the suffix of the error document.<br/>Up to 1024 bytes, only alphanumeric characters and some special characters (<code>-</code>, <code>_</code>, <code>.</code>, <code>/</code>) are allowed.</td>
  </tr>
</table>

#### Basic Settings

##### Access Policy

You can control access with the access policy.

* **PRIVATE**: Only permitted users can access objects within a container.
* **PUBLIC**: Anyone can access objects within a container through a public URL.

##### Object Life Cycle

If you set an object life cycle, the objects within the container are automatically deleted after the entered life cycle.

> [Note]
> It is applied only to objects uploaded after the object life cycle is set.

#### Object Version Control Settings

Object version control settings allow you to keep previous versions of objects. Previous versions are kept in the archive container when the object is updated, and you can enter a lifetime for previous versions through settings.
Previous versions that exceed the set life cycle are automatically deleted.

> [Caution]
> If the archive container is deleted before the original container, an error occurs when updating or deleting objects in the original container. If the archive container has already been deleted, you can solve the issue by creating a new archive container or disabling the original container's version control policy.

#### Replication Settings

Setting up replication allows objects in a container to be replicated to a container in another region. Replication Settings is a feature for disaster recovery, which replicates objects in the source region identically to the target region and manages the objects. Replication takes place at regular intervals in the background.

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

#### Static Website Settings

If you set the access policy of a container to **PUBLIC** and enter the index document and error document, you can host a static website in the container. You can get the URL of the static website by clicking the `Copy URL` button on the container list.

The object to be used as an index document or error document for a static website must have a name consisting of one or more alphanumeric characters or some special characters (`-`, `_`, `.`, `/`), and it must be in hypertext format with an `html` file extension. If the conditions are not met, you may not be able to configure the setting or the static website may not work.
The format of a static website's error document name is `{response code}{suffix}`. For example, if an error document is set as `error.html`, the name of the error document to be displayed when the 404 error occurs becomes `404error.html`. You can upload and use error documents according to each error condition. If an error document is not defined or an error object that matches the response code does not exist, the default error document of a web browser will be displayed.


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
<a href="https://api-storage.cloud.toast.com/v1/{account}/{container}/{object}">Download</a>
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

### Delete Object
Delete selected objects. When a multipart object is deleted, the segment object is also deleted.

## Prefix Search
If you enter a prefix in the search bar and click the **Search** button, you can search for containers, folders, and objects that begin with the prefix you entered. You can search for containers in the container list, and search for folders and objects in the object list.

## S3 API Credentials
You can obtain credentials required to use Amazon S3 compatible API. S3 API credentials have no expiration date, and up to 3 credentials can be issued per project for each user.

> [Caution]
> If the S3 API credentials key is leaked, anyone can access the object using the leaked key. If the key is leaked, it is recommended to delete the leaked credentials and obtain a new one.
>
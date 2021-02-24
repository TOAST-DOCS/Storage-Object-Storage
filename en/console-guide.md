## Storage > Object Storage > Console Guide

## Container
### Container Generation
Creates containers. Uploading objects in an object storage requires one or more containers.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Name</td>
    <td>Name of a container is limited to 255 characters in English or 85 characters in Korean.</td>
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

### Deleting Container
Delete selected containers. Check if the containers are empty before deleting them. If any objects are left inside a container, you cannot delete the relevant container.

### Container Details
Check detailed information of selected container. Information including basic information and settings of a container are available.

### Container settings
Change settings of a selected container.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Category</th>
    <th>Option</th>
    <th>Description</th>
  </tr>
  <tr>
    <td rowspan="3">Default Setting</td>
    <td rowspan="2">Container Access Policy</td>
    <td><b>PRIVATE</b>: Only permitted users can access objects within a container.</td>
  </tr>
  <tr>
    <td><b>PUBLIC</b>: Anyone with a public URL can access objects within a container.</td>
  </tr>
  <tr>
    <td>Object life cycle</td>
    <td>Enter object life cycle in days. Life cycle setting will be removed if it is not filled in.<br/>
    Objects that are uploaded to containers with object life cycle enabled up will be deleted automatically after the set life cycle is reached.</td>
  </tr>
  <tr>
    <td rowspan="3">Object<br/>Version Control Policy</td>
    <td>Version Control Policy</td>
    <td>When version control policy setting is enabled, you can store previous versions in a designated container when updating saved objects.</td>
  </tr>
  <tr>
    <td>Archive Container</td>
    <td>Enter a container in which previous versions of an object are stored.</td>
  </tr>
  <tr>
    <td>Archiving object<br/>life cycle</td>
    <td>Enter life cycle of previous versions of an object in days. Life cycle setting will be removed if it is not filled in.<br/>
    After the life cycle is enabled, previous versions of an object will be deleted automatically after the set life cycle is reached.</td>
  </tr>  
  <tr>
    <td rowspan="5">Static Website Settings</td>
    <td>Index Document</td>
    <td>Enter index document objects of a static website. If the object is within a folder, the folder path must be included.</td>
  </tr>
  <tr>
    <td>Error Document</td>
    <td>Enter the (suffix)of an error document from a static website.Folder path cannot be included in the suffix of the error document.</td>
  </tr>
</table>

> [Caution]
> If the version control policy has been set up, you must not delete the archive container before the original container. An error may occur as objects in the original container cannot save previous versions in the archive container during updates or deletion. If an error occurs because the archive container was deleted before the original container, create a new archive container or disable the version control policy of the original container.

<br/>

> [Note]
> If the access policy of a container is set to **PUBLIC** and index document and error document are entered, it becomes possible to host static websites in a container.
URL of static websites can be obtained by clicking the `Copy URL` button on the container list.

<br/>

> [Note]
> Error documents of static websites are named in the form of `{error code}{suffix}.` For example, an error document is set as `error.html,` the name of the error document will be displayed as `404error.html` when 404 error occurs. You can upload and use error documents according to each error conditions. If error documents are not defined or error objects that matches error codes do not exist, a default error document of a web browser will be displayed.


## Objects
### Create Folder
Create folders. Folders are virtual units to bundle objects within a container into a group. Similar to folders in Windows or directories in Linux, they help users to manage objects hierarchically. Folder names are limited to 255 letters in English or 85 characters in Korean.
> [Note]
> Folder for object storage is different from the directory provided by the file system. It is a pseudo folder provided for user's convenience. When a folder is created, an empty object named `{folder-name}/` is created. Objects within the folder will have names in the form of `{folder-name}/{object-name}`. Objects in the form of `{folder-name}/{object-name}` can be created directly without generating empty objects in the form of `{folder-name}/` by using the Copy Object function to copy objects into a new folder. If this copied object is deleted, it will appear as if the folder is also deleted. If you copy the object to a folder that you created in advance, the folder remains even if the object is deleted.

### Delete Folder
Delete folders. Check if the folders are empty before deleting them. If any objects are left inside a folder, you cannot delete the relevant folder.

### Upload Objects
All objects must be uploaded to containers. One object cannot be larger than 5GB.

> [Note]
> Files exceeding 5GB cannot be uploaded in a web console. Objects exceeding 5GB must be split by using commands, like  `split`, or programmed to be divided into less than 5GB before uploaded. Fore more details, refer to [Upload Multiple Parts](api-guide/#_10) of API guide.

### Download Objects
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


### Copy Objects
Copy objects to create  new objects. Create an object under a new name to the container which has an object to copy, or copy objects to another container.


### Delete Objects
Delete selected objects.

## Search Prefix
If you enter a prefix in the search bar and click the **Search**  button, you can see the search results of containers, folders, and objects that begin with the typed-in prefix. In the container list, you can search for containers, and in the object list, you can search for folders and objects.

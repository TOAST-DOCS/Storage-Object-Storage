## Third-Party Tools Usage Guide

This document describes how to use NHN Cloud Object Storage service as a third-party tool.

## Cyberduck

Cyberduck is an open-source cloud storage browser.

### Cyberduck Installation

Download and install the installation file for user’s operating system from the [Cyberduck download page ](https://cyberduck.io/download/).

### Object Storage Connection Settings

To connect to object storage, you must create a bookmark that stores connection information. By clicking `New Connection` button at the top of browser, you can create a new bookmark. Select `Openstack Swift (Keystone 2.0)` from drop-down menu, enter the required information, and click `Connect` button to create a bookmark.

<table class="it" style="padding-top: 15px; padding-bottom: 10px;">
  <tr>
    <th>Item</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Alias</td>
    <td>It is an alias of bookmark. You can freely set it to any desired value.</td>
  </tr>
  <tr>
    <td>Server</td>
    <td>It is an address of <b>Identity</b>. You can check it in the dialog box of <b>API Endpoint Settings</b> from on the Object Storage service page.</td>
  </tr>
  <tr>
    <td rowspan="2">Tenant ID: Access Key</td>
    <td><b>Tenant ID</b>: It is the project ID of the user. You can check it in <b>Project Management > Project Basic information</b> from the web console.</td>
  </tr>
  <tr>
    <td><b>Access ID</b>: Enter either NHN Cloud member ID (email format) or IAM user ID.</td>
  </tr>
  <tr>
    <td>Secret Key</td>
    <td>On the Object Storage Services page, enter the API password set in the <b> Set API Endpoint </b> dialog box.</td>
  </tr>
</table>

> \[Note] See [Set API Password](api-guide/#api) in the API guide for information on how to set API passwords.
### Connect Object Storage

Double-click the bookmark you want to connect in order to access the object storage.

### Retrieve Container/Object

When connecting to object storage, a list of containers for **all regions** appears in the browser. You can get a list of objects in a container by double-clicking the desired container.

> \[Note] If there are containers with the same name in different regions, multiple containers with the same name are displayed. Select **View>Column>Region** from the menu to display regions in the column entry.
### Create Container

Open the right-click menu in an empty space in the container list and open `New Folder...` to create a new container. After entering the container's name and region, click the `Create` button to create a container.

> \[Note] If there is no change in the container list after creating a container, you can update the container list by opening the right-click menu in the empty space of the list and clicking `Refresh`.
### Upload Object

Select a container and click `Action` > `Upload…` at the top of the browser, or open the right-click menu in the object list and click `Upload…` to select and upload the file. You can upload a file by dragging and dropping.

> \[Note] Uploading or creating a folder with Cyberduck creates another 0 byte object with the same folder name. The object can be found with the console or object storage APIs and can be deleted.
### Download Object

Select the object, open the right-click menu, and click `Download` to download it. You can download a file by dragging and dropping.

> \[Note] When downloading the object, it is stored in the local **Download** folder by default. Click Download to **Specified Location** in the right-click menu to download to the path you specify. When uploading or downloading an object, the **Send** window pops up to check its progress.
### Delete Container

Select the container you want to delete, open the right-click menu, and click `Delete` to delete it.

> \[Caution] Be careful because even if there are objects left inside the container, they will all be deleted.
### Delete Object

Select the object you want to delete, open the right-click menu, and click `Delete` to delete it.

### Synchronize Folder

After selecting a container or folder, you can synchronize the local folder with the selected container or folder by opening the right-click menu and clicking `Synchronize`. Provides three methods, such as download, upload, and mirror.

#### Download

Download changed or added objects from object storage locally.

#### Upload

Upload locally changed or added files to object storage.

#### Mirror

Compare local and object storage to upload or download changed or missing files or objects.

> \[Note] For more information on synchronization, see [Cyberduck Synchronize Folders](https://docs.cyberduck.io/cyberduck/sync/#synchronize-folders).
## SwiftStack Client

The SwiftStack Client is a client included in the object storage platform SwiftStack.

### Install SwiftStack Client

Download and install the appropriate installation file for user’s operating system from [SwiftStack Client web page ](https://platform.swiftstack.com/docs/install/swiftstack_client.html).

### Object Storage Connection Settings

#### Account Registration

To connect to object storage, you must register an account that stores connection information. If you click `Accounts` on the left side of the browser, an interface for registering an account and managing registered accounts appears. Click `Add account` to register an account. Enter the required information and click `Save` button at the bottom to register an account.

|Item | Description | |---|--- | Auth Type | Select**V2**. | | Username | Enter NHN Cloud member ID (email format) or IAM user ID. | | Password | Enter the API password set in the **Set API Endpoint** dialog box on the Object Storage Services page. | | Auth URL | On the Object Storage Services page, type **API Endpoint Settings** followed by the address of \*\*Identity** found in the dialog box with `/tokens`.| | Project Name | User's Project ID. <br/> You find it in the web console at <b>Project Management > Project Basic Information > Project ID</b>. | | Region | Select the region to use. |

> \[Note] See [API Password Settings](api-guide/#api) in the API guide for information on how to set API passwords. If the input value is not correct, an error occurs when selecting Region.
### Connect Object Storage

Double-click the account created to access object storage.

### Retrieve Container/Object

When connecting to object storage, a list of containers for the region set up appears in the browser. Double-click the desired container to view the list of objects in the container.

### Create Container 

Click `Add Container`>`Edit Before Creation` at the top of the browser and enter a container name in the dialog box. After entering a name, click `Create` at the bottom to create the container.

> \[Note] If there is no change in the container list after creating a container, you can update the container list by clicking `Refresh` at the top of the browser.
### Upload Object

Click the `Upload` button at the top of the browser in the container where you want to upload the object, and the Upload dialog box will appear. Click the `Choose File(s)` button to select a file and click `Upload` button to upload the selected file. You can upload a file by dragging and dropping.

> \[Note] If you select two or more files, you can put a prefix to the object name through the `Object Name Prefix` entry in the upload dialog box.
### Download Folder

Clicking the `Download` icon in `Actions` icon to the right of `Virtual Folder` object allows you to download the selected folder and all objects inside. To download multiple folders at once, check the desired folder in the list, and then click `Selection<4>`Download` at the top of the browser.

### Download Object

You can download one object by clicking on the object name. To download multiple objects at once, check the desired object in the list, and then click `Selection`>`Download` at the top of the browser.

### TempURLs

Clicking the `Copy Quick Temporary URL` icon in `Actions` icon on the right side of the object allows you to copy the expired access URL to clipboard. This URL allows you to create hyperlinks or download objects directly. The default time limit for TempURL is 24 hours and can be changed by clicking `Temp URLs` at the top of the browser.

> \[Caution] Anyone with TempURL can download the object within the time limit.
### Delete Container

Clicking `Delete Container` icon in `Actions` icon to the right of the container entry allows you to delete the desired container.

> \[Caution] Be careful because even if there are objects left inside the container, they will all be deleted.
### Delete Object

Clicking `Delete Object` icon in `Actions` icon to the right of the object item allows you to delete the desired object. To delete multiple objects at once, check the desired objects in the list, and then click `Selection`> `Delete` at the top of the browser.

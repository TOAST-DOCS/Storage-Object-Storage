## Third-Party Tools Usage Guide

This document describes how to use NHN Cloud Object Storage service as a third-party tool.

## Cyberduck

Cyberduck is an open-source cloud storage browser.

### Install Cyberduck

Download and install the installation file for user’s operating system from the [Cyberduck download page ](https://cyberduck.io/download/).

### Object Storage Connection Settings

To connect to object storage, you must create a bookmark that stores connection information. By clicking **New Connection** button at the top of browser, you can create a new bookmark. Select **Openstack Swift (Keystone 2.0)** from the drop-down menu, enter the required information, and click the **Connect** button to create a bookmark.

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

> [Note] 
> See [Set the API Password](api-guide/#api) in the API guide for information on how to set API passwords.

### Connect Object Storage

Double-click the bookmark you want to connect in order to access object storage.

### Retrieve Container/Object

When accessing object storage, a list of containers for **All Regions** appear in the browser. You can retrieve a list of objects in a container by double-clicking the desired container.

> [Note] 
> If there are containers with the same name in different regions, multiple containers with the same name are displayed. 
> Select **View > Column > Region** from the menu to display regions in the column entry.

### Create Container

You can create a new container by right-clicking an empty space in the container list and selecting **New Folder...**. After entering the container's name and region, click the **Create** button to create a container.

> [Note] 
>  You can refresh the container list by right-clicking an empty space in the list and selecting **View Again**.

### Upload Object

Select a container and click **Action** > **Upload…** at the top of the browser, or right-click the object list and click **Upload…** to select and upload the file.

> [Note] 
> If you upload or create a folder with Cyberduck, another 0 byte object with the same folder name is created. The object can be found with the console or object storage API and can be deleted.

### Download Object

Select and right-click an object to download, select **Download**. You can download the object by dragging and dropping.

> [Note] 
> When you download an object, it is saved to your local **Downloads** folder by default. Right-click it and select **Download to Specified Location** to download to the specified path.
> When uploading or downloading an object, the **Send** window pops up to check the progress.

### Delete Container

Select the container to delete and right-click and select **Delete** to delete it.

> [Caution] 
> When deleting containers, all objects in the containers are deleted.

### Delete Object

Select the object to delete and right-click and select **Delete**.

### Synchronize Folder

You can synchronize local folders with containers or folders. Select a container or folder and right-click and select **Synchronize**. 
Folder synchronization provides three methods of download, upload, and mirror.

#### Download

Download objects that are changed or added in object storage to your local.

#### Upload

Upload files that are changed or added in your local to object storage.

#### Mirror

Compare local and object storage to upload or download changed or missing files or objects.

> [Note] 
> For more information on synchronization, see [Cyberduck Synchronize Folders](https://docs.cyberduck.io/cyberduck/sync/#synchronize-folders).

## SwiftStack Client

The SwiftStack Client is a client included in the object storage platform SwiftStack.

### Install SwiftStack Client

Download and install the installation file suitable for your operating system from [SwiftStack Client web page ](https://platform.swiftstack.com/docs/install/swiftstack_client.html).

### Object Storage Connection Settings

#### Register Account

To connect to object storage, you must register an account that stores connection information. If you click **Accounts** on the left side of the browser, an interface for registering an account and managing registered accounts appears. Click **Add account** to register an account. Enter the required information and click **Save** button at the bottom to register an account.

|Item | Description | 
|---|--- |
| Auth Type | Select **V2**. | 
| Username | Enter NHN Cloud member ID (email format) or IAM member ID. | 
| Password | Enter the API password set in the **Set API Endpoint** dialog box on the object storage service page. | 
| Auth URL | Enter the address of the **identity service (identity)** checked in the **API Endpoint Settings** dialog box of the Object Storage Service page with `/tokens`.| 
| Project Name | User's Project ID. <br/> You find it in <b>Project Management > Project Basic Information > Project ID</b> from the web console. | 
| Region | Select the region to use. |

> [Note] 
> See [API Password Settings](api-guide/#api) in the API guide for information on how to set API passwords. 
> If the input value is invalid, an error occurs when selecting Region.

### Connect Object Storage

Double-click the created account to access object storage.

### Retrieve Container/Object

When accessing object storage, a list of containers for the region set up appears in the browser. Double-click the desired container to view the list of objects in the container.

### Create Container 

Click **Add Container** > **Edit Before Creation** at the top of the browser and enter a container name in the dialog box. After entering a name, click **Create** at the bottom to create the container.

> [Note] 
> You can refresh the container list by clicking **Refresh** at the top of the browser.

### Upload Object

Click the **Upload** button at the top of the browser in the container where you want to upload the object, and the Upload dialog box appears. Click **Choose File(s)** to select a file and click **Upload** to upload the selected file.

> [Note] 
> If you select two or more files, you can put a prefix to the object name through the **Object Name Prefix** entry in the upload dialog box.

### Download Folder

If you click the **Download** icon among the **Actions** icons at the right of the **Virtual Folder** object, you can download the selected folder and all objects inside. To download multiple folders at once, check a folder you want in the list, and then click **Selection** > **Download** at the top of the browser.

### Download Object

You can download one object by clicking the object name. To download multiple objects at once, check an object you want in the list, and then click **Selection** > **Download** at the top of the browser.

### TempURLs

If you click the **Copy Quick Temporary URL** icon among the **Actions** icons at the right of the object, the access URL with an expiration date is copied to the clipboard. This URL allows you to create hyperlinks or download objects directly. 
The default time limit for TempURL is 24 hours and can be changed by clicking **Temp URLs** at the top of the browser.

> [Caution] 
> Anyone with TempURL can download the object within the time limit.

### Delete Container

If you click the **Delete Container** icon in **Actions** icon to the right of the container entry, you can delete a container.

> [Caution] 
> Be careful because even if there are objects left inside the container, they will all be deleted.

### Delete Object

If you click the **Delete Object** icon among the **Actions** icons at the right of the object item, you can delete an object. To delete multiple objects at once, check objects you want to delete in the list, and then click **Selection** > **Delete** at the top of the browser.

## References
Cyberduck - [https://docs.cyberduck.io/cyberduck/](https://docs.cyberduck.io/cyberduck/)
SwiftStack Client - [https://platform.swiftstack.com/docs/install/swiftstack_client.html](https://platform.swiftstack.com/docs/install/swiftstack_client.html)

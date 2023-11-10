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

## References
Cyberduck - [https://docs.cyberduck.io/cyberduck/](https://docs.cyberduck.io/cyberduck/)

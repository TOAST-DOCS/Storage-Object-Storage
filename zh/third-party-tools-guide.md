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

## Terraform

Terraform is an open source tool that makes it easy to build infrastructure, make changes to it safely, and manage its geometry efficiently. For basic usage, see [User Guide > Compute > Instance > Terraform User Guide](https://docs.nhncloud.com/ko/Compute/Instance/ko/terraform-guide/).

### Resource Dependency

In general, each resource is independent, but it can also have dependencies on certain other resources. Terraform automatically establishes dependencies when a resource's label references information from another resource.
For example, the `object1 object` contained in the `conatiner1` container might be represented as follows

```hcl
# Container resource
resource "nhncloud_objectstorage_container_v1" "container_1" {
  name = "container1"
}

# Object resource
resource "nhncloud_objectstorage_object_v1" "object_1" {
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name = "object1"
  source = "/tmp/dummy"
}
```

> [Note]
> For information on how to specify explicit resource dependencies, see [Terraform's Resource dependencies](https://developer.hashicorp.com/terraform/tutorials/configuration-language/dependencies) documentation.

### Resources - Object Storage

#### Create a Container

```hcl
# Create a default container
resource "nhncloud_objectstorage_container_v1" "container_1" {
  region = "KR1"
  name = "tf-test-container-1"
}

# Create an object versioning container
resource "nhncloud_objectstorage_container_v1" "container_2" {
  region = "KR1"
  name = "tf-test-container-2"
  versioning_legacy {
    type = "history"
    location = resource.nhncloud_objectstorage_container_v1.container_1.name
  }
}

# Create a public container
resource "nhncloud_objectstorage_container_v1" "container_3" {
  region = "KR1"
  name = "tf-test-container-3"
  container_read = ".r:*,.rlistings"
}
```

| Name | Type | Required | Description |
| ---- | ---- | ---- | ---- |
| region | String | | Region to manage NHN Cloud resources |
| name | String | O | Container name |
| container_read | String | | Sets the role-based access rules for container read |
| container_write | String | | Role-based access rules for container writes |
| force_destroy | Boolean | | Whether to force container deletion, `true` or `false`<br>You can't recover objects that were deleted together. |
| versioning_legacy | Object | | Object Version Control Settings |
| versioning_legacy.type | String | | Specify as `history` |
| versioning_legacy.location | String | | Container name to store the previous version of the object |

#### Create an Object

```hcl
# Create the object
resource "nhncloud_objectstorage_object_v1" "object_1" {
  region = "KR1"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name = "test/test1.json"
  content_type = "application/json"
  content = <<JSON
               { "key" : "value
                 "key" : "value"
               }
JSON
}

# Upload a file
resource "nhncloud_objectstorage_object_v1" "object_2" {
  region = "KR2"
  container_name = nhncloud_objectstorage_container_v1.container_1.name
  name = "test/test2.json"
  source = "./test2.txt"
}
```

<table>
	<thead>
			<tr>
				<th>Name</th>
				<th>Type</th>
				<th>Required</th>
				<th>Description</th>
			</tr>
	</thead>
	<tbody>
		<tr>
			<td>name</td>
			<td>String</td>
			<td>O</td>
			<td>Object name</td>
		</tr>
		<tr>
			<td>container_name</td>
			<td>String</td>
			<td>O</td>
			<td>Container name</td>
		</tr>
		<tr>
			<td>source</td>
			<td>String</td>
			<td rowspan=4>O</td>
			<td> The path to the file on the local file system to upload<br>Cannot be used with <code>content</code>, <code>copy_from</code>, or <code>object_manifest</code>.<br>You must specify one of the following: <code>source</code>, <code>content</code>, <code>copy_from</code>, or <code>object_manifest</code>.</td>
		</tr>
		<tr>
			<td>content</td>
			<td>String</td>
			<td>Data content of the object to create<br>Cannot be used with <code>source</code>, <code>copy_from</code>, or <code>object_manifest</code>.</td>
		</tr>
		<tr>
			<td>copy_from</td>
			<td>String</td>
			<td>The original object to copy, <code>{container}/{object}</code><br>Cannot be used with <code>source</code>, <code>content</code>, or <code>object_manifest</code>.</td>
		</tr>
		<tr>
			<td>object_manifest</td>
			<td>String</td>
			<td>The path where segment objects are uploaded: <code>{Segment-Container}/{Segment-Object}/</code><br>Cannot be used with <code>source</code>, <code>content</code>, or <code>copy_from</code>.</td>
		</tr>
		<tr>
			<td>content_disposition</td>
			<td>String</td>
			<td></td>
			<td>Specify the <code>Content-Disposition</code> header</td>
		</tr>
		<tr>
			<td>content_encoding</td>
			<td>String</td>
			<td></td>
			<td>Specify the <code>Content-Encoding</code> header</td>
		</tr>
			<tr>
			<td>content_type</td>
			<td>String</td>
			<td></td>
			<td>Specify the <code>Content-Type</code> header</td>
		</tr>
		<tr>
			<td>delete_after</td>
			<td>Integer</td>
			<td></td>
			<td>Object's expiration time, unix time (seconds)</td>
		</tr>
		<tr>
			<td>delete_at</td>
			<td>String</td>
			<td></td>
			<td>Object expiration date, Unix time in seconds, RFC3339 formatted string</td>
		</tr>
		<tr>
			<td>detect_content_type</td>
			<td>Boolean</td>
			<td></td>
			<td>Whether to infer content type</br>The <code>content_type</code> is ignored when setting.</td>
		</tr>
	</tbody>
</table>

## Reference
Cyberduck - [https://docs.cyberduck.io/cyberduck/](https://docs.cyberduck.io/cyberduck/)
Terraform - [https://www.terraform.io/](https://www.terraform.io/)
Terraform Registry - [https://registry.terraform.io/][https://registry.terraform.io/]


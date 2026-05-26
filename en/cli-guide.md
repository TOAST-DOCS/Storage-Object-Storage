## Storage > Object Storage > CLI Guide

This guide explains how to use the NHN Cloud Object Storage service with the OpenStack Swift command-line interface (CLI).

<a id="python-swiftclient"></a>
## python-swiftclient

<a id="install"></a>
### Install

python-swiftclient is provided as a Python package. Install it using pip.

```
pip install python-swiftclient python-keystoneclient
```

> [Note]
> Python 3.6 or later is required. If Python is not installed, refer to the [Python download page](https://www.python.org/downloads/) for installation instructions.

Once the installation is complete, you can verify it with the following command.

```
$ swift --version
python-swiftclient x.x.x
```

<br/>

<a id="configuration"></a>
### Configuration

To use the Swift CLI, you must configure the environment variables required for authentication. Click the **API Endpoint Settings** button on the Object Storage service page to find the required information.

```
export OS_AUTH_URL=https://api-identity-infrastructure.nhncloudservice.com/v2.0
export OS_TENANT_ID=<Tenant ID>
export OS_USERNAME=<NHN Cloud account ID>
export OS_PASSWORD=<API password>
export OS_REGION_NAME=<Region name>
```

<br/>

| Environment Variable | Description |
| --- | --- |
| OS_AUTH_URL | Identity API URL<br/>https://api-identity-infrastructure.nhncloudservice.com/v2.0 |
| OS_TENANT_ID | Tenant ID available in **API Endpoint Settings** on the Object Storage service page |
| OS_USERNAME | NHN Cloud account ID (email format) or IAM account ID |
| OS_PASSWORD | API password configured in **API Endpoint Settings** |
| OS_REGION_NAME | Region name<br/>KR1 - Korea (Pangyo) region<br/>KR2 - Korea (Pyeongchon) region<br/>KR3 - Korea (Gwangju) region<br/>JP1 - Japan (Tokyo) region |

<br/>

> [Caution]
> Object Storage has a different tenant ID from the basic infrastructure service. Click the **API Endpoint Settings** button on the Object Storage service page to verify.

<!-- 개행을 위한 주석 -->

> [Note]
> The API password can be configured by clicking the **API Endpoint Settings** button on the Object Storage service page.

<br/>

Creating a configuration file allows you to use the CLI conveniently without setting environment variables each time.

```
$ cat ~/swiftrc
export OS_AUTH_URL=https://api-identity-infrastructure.nhncloudservice.com/v2.0
export OS_TENANT_ID=<Tenant ID>
export OS_USERNAME=<NHN Cloud account ID>
export OS_PASSWORD=<API password>
export OS_REGION_NAME=<Region name>
```

```
$ source ~/swiftrc
```

<br/>

To verify that the configuration is correct, run the `swift stat` command. If account information is displayed without errors, the configuration is set up correctly.

```
$ swift stat
        Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
     Containers: 0
        Objects: 0
          Bytes: 0
             ...
...
```

<br/>

<a id="basic-usage"></a>
## Basic usage
The basic usage is as follows.

```
swift <subcommand> [<options>] [<container> [<object>]]
```

<br/>

<a id="subcommands"></a>
### Subcommands
The main subcommands are as follows.

| Subcommand | Description |
| --- | --- |
| stat | Retrieves information about accounts, containers, or objects. |
| list | Retrieves a list of containers or objects. |
| post | Creates a container or configures metadata. |
| upload | Uploads objects. |
| download | Downloads objects. |
| copy | Copies objects. |
| delete | Deletes containers or objects. |
| tempurl | Generates a signed URL. |
| auth | Outputs authentication-related environment variables. |

<br/>

<a id="options"></a>
### Common options

These options can be used with all subcommands.

| Option | Description |
| --- | --- |
| --help | Displays usage instructions for each subcommand. |
| --debug | Displays the actual API call details. |
| --quiet | Suppresses status output. |
| --retries &lt;num&gt; | Specifies the number of retries when a request fails. |

<br/>

<a id="auth-info"></a>
### How it works and using authentication information

By default, the Swift CLI calls the Identity API each time a command is executed to obtain a token and service catalog, then calls the Swift API corresponding to the subcommand.

<br/>

Using the `auth` subcommand, you can check the storage URL and authentication token in advance.

```
$ swift auth
export OS_STORAGE_URL=https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067
export OS_AUTH_TOKEN=gAAAAABi...
```

<br/>

Applying the output environment variables to the shell allows subsequent commands to skip token issuance and catalog queries, resulting in faster execution.

```
$ eval $(swift auth)
```

> [Note]
> Authentication tokens have an expiration time. If a request fails due to a token expiry, run `eval $(swift auth)` again to renew it.

<br/>

<a id="stat"></a>
## View information
Retrieves information about storage accounts, containers, and objects.

```
swift stat [<options>] [<container> [<object>]]
```

<br/>

| Option | Description |
| --- | --- |
| --lh | Displays capacity in human-readable units (KB, MB, etc.). |

<br/>

<a id="stat-account"></a>
### View storage account information
Retrieves information about the storage account.

```
$ swift stat
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
            Containers: 3
               Objects: 4
                 Bytes: 807711
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
                    ...
```

<br/>

<a id="stat-container"></a>
### View container information
Retrieves information about a container.

```
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                 Bytes: 95983
              Read ACL:
             Write ACL:
          Content-Type: application/json; charset=utf-8
           X-Timestamp: 1772338036.37283
         Last-Modified: Sun, 01 Mar 2026 04:07:16 GMT
      X-Storage-Policy: standard
                    ...
```

<br/>

<a id="stat-object"></a>
### View object information
Retrieves information about an object.

```
$ swift stat media 797619b171a455e9eec8a87f94ee77f4.jpg
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
                Object: 797619b171a455e9eec8a87f94ee77f4.jpg
          Content Type: image/jpeg
        Content Length: 31661
         Last Modified: Sun, 01 Mar 2026 04:07:16 GMT
                  ETag: bd97323f11e8f2fc1655cbb8a1b9382e
           X-Timestamp: 1772338036.37283
                    ...
```

<br/>

<a id="list"></a>
## List
Retrieves a list of containers or objects.

```
swift list [container] [options]
```

<br/>

| Option | Description |
| --- | --- |
| --long | Retrieves a list with detailed information for each resource. |
| --lh | Similar to `--long`, but displays capacity in human-readable units. |
| --totals | When used with `--long` or `--lh`, outputs only the totals. |
| --json | Outputs the list in JSON format. |
| --prefix &lt;prefix&gt; | Retrieves only items that start with the specified prefix. |
| --delimiter &lt;delimiter&gt; | Can only be used when retrieving object lists.<br/>Groups and displays object names based on the delimiter.<br/>For example, using `--delimiter /` excludes objects with names such as `pics/9b171a455e9e.png` from the list and displays them as `pics/`, similar to a folder. |

<br/>

<a id="list-containers"></a>
### List containers
Retrieves a list of containers.

```
$ swift list
media
```

<br/>

<a id="list-objects"></a>
### List objects
Retrieves a list of objects.

```
$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="create-container"></a>
## Create container
Creates a new container.

```
swift post <container>
```

```
$ swift post docs
$ swift list
docs
media
```

<br/>

<a id="upload"></a>
## Upload objects
Uploads or updates (overwrites) objects. Multiple objects can be uploaded at once.

```
swift upload <container> <file_or_directory> [<file_or_directory>] [...]
```

<br/>

| Option | Description |
| --- | --- |
| --object-name &lt;name&gt; | Specifies the name of the object to upload.<br/>If omitted, the file name is used. |
| --changed | If an object with the same name exists in the container, compares the modification time and size, and uploads only if changed. |
| --skip-identical | Skips the upload if an identical object exists in the container. |
| --segment-size &lt;size&gt; | Splits the file into the specified size and uploads it as a multipart upload using the SLO method. |
| --segment-container &lt;container&gt; | Specifies the container in which to store segment objects. |
| --leave-segments | Retains existing segments without deleting them when overwriting an object. |
| --object-threads &lt;threads&gt; | Specifies the number of threads to use for object uploads. The default value is 10. |
| --segment-threads &lt;threads&gt; | Specifies the number of threads to use for segment uploads. The default value is 10. |
| --meta &lt;name:value&gt; | Configures metadata. Can be used multiple times. |
| --ignore-checksum | Disables checksum verification during upload. |

<br/>

```
$ swift upload media 4287b656254e69e948c3cab237f3fb03.jpg
4287b656254e69e948c3cab237f3fb03.jpg

$ swift list media
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
4287b656254e69e948c3cab237f3fb03.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="multipart-upload"></a>
### Multipart upload
Specifying the segment size with the `--segment-size` option splits the file and uploads it as a multipart upload.

```
swift upload --segment-size <size> [--segment-container <segment-container>] <container> <file>
```

<br/>

If `--segment-container` is not specified, a container named `{container}_segments` is created and the segment objects are uploaded to it.

```
$ swift upload --segment-size 10m media test.mp4
test.mp4 segment 1
test.mp4 segment 0
test.mp4

$ swift list
media
media_segments

$ swift list media
...
test.mp4

$ swift list media_segments
test.mp4/slo/1635487628.192050/20971520/10485760/00000000
test.mp4/slo/1635487628.192050/20971520/10485760/00000001
```

<br/>

<a id="download"></a>
## Download objects
Downloads objects to the current path. Multiple objects can be downloaded.

```
swift download <container> [<obj>] [...]
```

<br/>

| Option | Description |
| --- | --- |
| --output &lt;out_file&gt; | Specifies the file name to save the object as.<br/>Specifying `-` outputs the object content to standard output. |
| --output-dir &lt;out_directory&gt; | Specifies the path to download to. |
| --prefix &lt;prefix&gt; | Downloads only objects that start with the specified prefix. |
| --remove-prefix | When used with `--prefix`, downloads the objects with the prefix removed from the name. |
| --skip-identical | Skips the download if an identical file exists at the local path. |
| --object-threads &lt;threads&gt; | Specifies the number of threads to use for object downloads. The default value is 10. |
| --ignore-checksum | Disables checksum verification during download. |
| --no-download | Executes the download but does not write the file to disk. |

<br/>

```
$ swift download media 153206025029118e96e38b95b78281c8.jpg
153206025029118e96e38b95b78281c8.jpg [auth 0.117s, headers 0.288s, total 0.393s, 2.344 MB/s]
```

<br/>

<a id="download-all"></a>
### Download by container
If the object to download is omitted, all objects in the specified container are downloaded.

```
$ swift download media
300e5522b971c2159e65f95e3c89d981.jpg [auth 0.111s, headers 0.249s, total 0.264s, 0.376 MB/s]
797619b171a455e9eec8a87f94ee77f4.jpg [auth 0.303s, headers 0.451s, total 0.460s, 0.202 MB/s]
...
```

<br/>

<a id="copy"></a>
## Copy objects
Copies an object to the specified path. Metadata can be added or changed during the copy.

```
swift copy [--destination </container/object>] [--fresh-metadata] [--meta <name:value>] <container> <object>
```

<br/>

| Option | Description |
| --- | --- |
| --destination &lt;/container/object&gt; | Specifies the destination container and object name to copy to.<br/>If the object name is omitted, it is copied with the same name as the source. |
| --meta &lt;name:value&gt; | Configures metadata. Can be used multiple times. |
| --fresh-metadata | Does not copy the metadata from the source.<br/>When used with `--meta`, only the newly specified metadata is set. |

<br/>

<a id="copy-same-container"></a>
### Copy within the same container
The destination object name must be different from the source.

```
$ swift copy --destination "/media/copied_object.jpg" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

<br/>

<a id="copy-other-container"></a>
### Copy to a different container
If the destination object name is omitted, it is copied with the same name as the source.

```
$ swift copy --destination "/pics" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container pics
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /pics/797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="copy-with-metadata"></a>
### Copy with additional metadata
Metadata can be added during copying using the `--meta` option.

```
$ swift copy --destination "/media/copied_object.jpg" --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
created container media
/media/797619b171a455e9eec8a87f94ee77f4.jpg copied to /media/copied_object.jpg
```

Using the `--fresh-metadata` option together removes existing metadata and sets only the newly specified metadata.

<br/>

<a id="post"></a>
## Change settings
Changes the settings of a container or object.

<br/>

<a id="post-container"></a>
### Change container settings
Changes the settings of a container. Container settings can be verified with the [view container information command](cli-guide/#stat-container).

```
swift post [<options>] <container>
```

<br/>

| Option | Description |
| --- | --- |
| --meta &lt;key:value&gt; | Configures container metadata. Can be used multiple times. |
| --read-acl &lt;acl&gt; | Sets the read ACL for the container. |
| --write-acl &lt;acl&gt; | Sets the write ACL for the container. |
| --header &lt;key:value&gt; | Adds a custom HTTP header. Settings not provided as options by the Swift CLI can also be specified directly with this option. |

<br/>

> [Note]
> For ACL setting values, see the [Access policy configuration guide](acl-guide/).
>
> For the list of headers that can be set with the `--header` option, see the [Change container settings](api-guide/#change-container-settings) section in the API guide.

<br/>

```
$ swift post --meta "Type:photo" media
$ swift post --header "X-Container-Object-Lifecycle: 90" media
$ swift post --read-acl ".r:*" media
```

<br/>

<a id="post-object"></a>
### Change object metadata
Changes the metadata of an object. Object metadata can be verified with the [view object information command](cli-guide/#stat-object).

```
swift post [<options>] <container> <object>
```

<br/>

| Option | Description |
| --- | --- |
| --meta &lt;key:value&gt; | Configures object metadata. Can be used multiple times. |
| --header &lt;key:value&gt; | Adds a custom HTTP header. Object properties such as Content-Type can be changed. |

<br/>

```
$ swift post --meta "Color:Blue" media 797619b171a455e9eec8a87f94ee77f4.jpg
$ swift post --header "Content-Type:image/png" media 797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="delete"></a>
## Delete
Deletes containers or objects.

<br/>

<a id="delete-container"></a>
### Delete container
Deletes all objects inside the specified container, then deletes the container.

```
swift delete <container>
```

<br/>

| Option | Description |
| --- | --- |
| --prefix &lt;prefix&gt; | Deletes only objects that start with the specified prefix. The container is not deleted. |
| --object-threads &lt;threads&gt; | Specifies the number of threads to use for object deletion. The default value is 10. |
| --container-threads &lt;threads&gt; | Specifies the number of threads to use for container deletion. The default value is 10. |

<br/>

```
$ swift delete media
797619b171a455e9eec8a87f94ee77f4.jpg
153206025029118e96e38b95b78281c8.jpg
300e5522b971c2159e65f95e3c89d981.jpg
```

<br/>

<a id="delete-object"></a>
### Delete object
Deletes the specified object.

```
swift delete <container> <object> [<object>] [...]
```

| Option | Description |
| --- | --- |
| --leave-segments | Retains segments of the manifest object without deleting them. |
| --object-threads &lt;threads&gt; | Specifies the number of threads to use for object deletion. The default value is 10. |

<br/>

```
$ swift delete media 797619b171a455e9eec8a87f94ee77f4.jpg
797619b171a455e9eec8a87f94ee77f4.jpg
```

<br/>

<a id="tempurl"></a>
## Create signed URL
Generates a signed URL that allows access to a container or object without a token.

```
swift tempurl <method> <time> <path> <key>
```

<br/>

| Argument | Description |
| --- | --- |
| method | The HTTP method to allow. Generally `GET` or `PUT` is used. |
| time | The expiration time of the signed URL.<br/>Enter in ISO8601 format (`2026-03-01`, `2026-03-01T23:59:59`, `2026-03-01T23:59:59Z`).<br/>Can also be entered in seconds, in which case the URL expires after the specified duration from the current time. |
| path | Enter in the format `/v1/AUTH_*****/<container>` or `/v1/AUTH_*****/<container>/<object>`. |
| key | The Temp-URL-Key registered in the container in advance. |

<br/>

| Option | Description |
| --- | --- |
| --prefix-based | Generates a prefix-based signed URL. |
| --ip-range &lt;ip_range&gt; | Restricts access via the signed URL to a specific IP or IP range. |
| --digest &lt;algorithm&gt; | Specifies the hash algorithm.<br/>`sha256` (default) or `sha512` can be used. |
| --absolute | When `<time>` is entered in seconds, the value is interpreted as a Unix timestamp rather than a relative time from the current time. Does not affect ISO8601 format. |
| --iso8601 | Includes the expiration time in ISO8601 format in the generated URL instead of a Unix timestamp. |

<br/>

```
$ swift tempurl GET 2026-03-05T23:59:59 /v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg 398dded8-b1be
/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg?temp_url_sig=8244bff5037316dbe8aebcda9cd679c1b331e479&temp_url_expires=1772755199

$ curl https://kr1-api-object-storage.nhncloudservice.com/v1/AUTH_6dbc368b94894416bec4cdfc65b5e067/media/797619b171a455e9eec8a87f94ee77f4.jpg\?temp_url_sig\=8244bff5037316dbe8aebcda9cd679c1b331e479\&temp_url_expires\=1772755199 --output 797619b171a455e9eec8a87f94ee77f4.jpg
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 31661  100 31661    0     0  1043k      0 --:--:-- --:--:-- --:--:-- 1058k
```

<br/>

<a id="tempurl-key"></a>
### Register Temp-URL-Key
To generate a signed URL, a Temp-URL-Key must be registered in the container in advance. The configured key can be verified with the [view container information command](cli-guide/#stat-container).

```
swift post --meta "Temp-URL-Key: <key>" <container>
```

```
$ swift post --meta "Temp-URL-Key: 398dded8-b1be" media
$ swift stat media
               Account: AUTH_6dbc368b94894416bec4cdfc65b5e067
             Container: media
               Objects: 3
                    ...
     Meta Temp-Url-Key: 398dded8-b1be
                    ...
```

<a id="reference"></a>
## References
Object Storage service (swift) command-line client - [https://docs.openstack.org/ocata/cli-reference/swift.html](https://docs.openstack.org/ocata/cli-reference/swift.html)
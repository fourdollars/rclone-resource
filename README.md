 [![GitHub: fourdollars/rclone-resource](https://img.shields.io/badge/GitHub-fourdollars%2Frclone%E2%80%90resource-green.svg)](https://github.com/fourdollars/rclone-resource/) [![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT) [![Bash](https://img.shields.io/badge/Language-Bash-red.svg)](https://www.gnu.org/software/bash/) ![Docker](https://github.com/fourdollars/rclone-resource/workflows/Docker/badge.svg) [![Docker Pulls](https://img.shields.io/docker/pulls/fourdollars/rclone-resource.svg)](https://hub.docker.com/r/fourdollars/rclone-resource/)
# rclone-resource
[rclone](https://rclone.org/)'s resource

* Known Issue
  * It doesn't support WebDAV for digest authentication. [webdav: Support for digest authentication #2110](https://github.com/rclone/rclone/issues/2110)

## Config

### Resource Type

```yaml
resource_types:
- name: resource-rclone
  type: registry-image
  source:
    repository: fourdollars/rclone-resource
    tag: latest
```

or

```yaml
resource_types:
- name: resource-rclone
  type: registry-image
  source:
    repository: ghcr.io/fourdollars/rclone-resource
    tag: latest
```

### Resource

* remote: **required**
* config: **required**
* path: optional, if not specified, it will watch the root folder.
* files: optional, if not specified, it will watch the whole folder.
* args: optional, the arguments list passed to `rclone lsjson` and `rclone copy`.

```yaml
resources:
- name: storage
  type: resource-rclone
  source:
    remote: webdavRemote
    config: |
      [webdavRemote]
      type = webdav
      url = https://webdav.some.where/share/project/
      vendor = other
      user = hello-kitty
      pass = e3b0c44298fc1-149afbf4c8996fb92427ae41
    path: First/Path
```

### check step

It will use `rclone lsjson` to watch the changes.

```shell
# It acts like the following command.
$ rclone lsjson webdavRemote:First/Path
```

### get step

* path: optional
* files: optional, if specified, it will overwrite the files in source.
* skip: optional, set true if you just want to list files and folders.
* args: optional, the arguments list passed to `rclone copy`.

```yaml
- get: storage
  params:
    folder: Second/Folder
    files:
      - file1.txt
      - file2.txt
```
```shell
# It acts like the following commands.
$ cd /tmp/build/get
$ rclone copy webdavRemote:First/Path/Second/Folder/file1.txt .
$ rclone copy webdavRemote:First/Path/Second/Folder/file2.txt .
```

### put step

* from: **required**
* files: optional, if not specified, it will copy all files under 'from'.
* folder: optional
* args: optional, the arguments list passed to `rclone copy`.
* get_params:
  * skip: optional if you don't want the [implicit get step](https://concourse-ci.org/jobs.html#put-step) after the put step to download the same content again in order to save the execution time.

```yaml
- put: storage
  params:
    args:
      - --ignore-times
    from: SomeFolderInTask
    files:
      - file1.txt
      - file2.txt
    folder: Second/Folder
  get_params:
    skip: true
```
```shell
# It acts like the following commands.
$ cd /tmp/build/put
$ rclone copy --ignore-times SomeFolderInTask/file1.txt webdavRemote:First/Path/Second/Folder
$ rclone copy --ignore-times SomeFolderInTask/file2.txt webdavRemote:First/Path/Second/Folder
```

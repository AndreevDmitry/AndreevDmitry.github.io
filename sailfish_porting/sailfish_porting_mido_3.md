Далее необходимо установить цель "mido" для Scratchbox2, которая будет использоваться для упаковки адаптационных пакетов аппаратного обеспечения mido.
```console
PlatformSDK:~/hadk$ sdk-assistant create $VENDOR-$DEVICE http://releases.sailfishos.org/sdk/latest/Jolla-latest-Sailfish_SDK_Tooling-i486.tar.bz2
```
<details>
Creating tooling [xiaomi-mido]<br>
Using tarball [http://releases.sailfishos.org/sdk/latest/Jolla-latest-Sailfish_SDK_Tooling-i486.tar.bz2]<br>
Do you want to continue? (y/n) y<br>
Downloading 'Jolla-latest-Sailfish_SDK_Tooling-i486.tar.bz2'<br>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                                 Dload  Upload   Total   Spent    Left  Speed<br>
100  223M  100  223M    0     0  15.7M      0  0:00:14  0:00:14 --:--:-- 18.0M<br>
INFO: md5sum matches - download ok<br>
Unpacking tooling ...<br>
Initializing machine ID for tooling 'xiaomi-mido'<br>
NOTICE: removing unexpected non-empty /etc/machine-id in the 'xiaomi-mido' tooling<br>
Initializing machine ID from random generator.<br>
Tooling 'xiaomi-mido' now setup<br>
</details><br>

```console
PlatformSDK:~/hadk$ sdk-assistant create $VENDOR-$DEVICE-$PORT_ARCH http://releases.sailfishos.org/sdk/latest/Jolla-latest-Sailfish_SDK_Target-$PORT_ARCH.tar.bz2
```
<details>
Creating target [xiaomi-mido-armv7hl]<br>
Using tarball [http://releases.sailfishos.org/sdk/latest/Jolla-latest-Sailfish_SDK_Target-armv7hl.tar.bz2]<br>
Do you want to continue? (y/n) y<br>
Downloading 'Jolla-latest-Sailfish_SDK_Target-armv7hl.tar.bz2'<br>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                                 Dload  Upload   Total   Spent    Left  Speed<br>
100  198M  100  198M    0     0  11.1M      0  0:00:17  0:00:17 --:--:-- 11.8M<br>
INFO: md5sum matches - download ok<br>
<br>
Unpacking target ...<br>
Using 'xiaomi-mido' tooling for this target<br>
Making sure the right toolchain exists in 'xiaomi-mido' tooling<br>
Setting up SB2<br>
Using /srv/mer/toolings/xiaomi-mido//opt/cross/bin/armv7hl-meego-linux-gnueabi-gcc to detect target architecture:<br>
Finished writing sb2.gcc.config<br>
gcc configured.<br>
sb2-init: Target architecture is 'armv7hl'<br>
sb2-init: Host architecture is 'i[3456]86'<br>
which: no gcc in (/home/stalker/bin:/home/stalker/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin)<br>
sb2-init: no host-gcc.<br>
Finished writing sb2.config<br>
sb2-init: Creating Debian build system settings for this target:<br>
Initializing machine ID for target 'xiaomi-mido-armv7hl'<br>
NOTICE: removing unexpected non-empty /etc/machine-id in the 'xiaomi-mido-armv7hl' target<br>
qemu: Unsupported syscall: 384<br>
Initializing machine ID from random generator.<br>
Target 'xiaomi-mido-armv7hl' now setup<br>
</details><br>


Проверим, все ли установлось корректно на примере Hello World
```console
PlatformSDK:~/hadk$ cd $HOME
PlatformSDK:~$ cat > main.c << EOF
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  printf("Hello, world!\n");
  return EXIT_SUCCESS;
}
EOF
```

Скомпилируем нашу программу под mido
```console
PlatformSDK:~$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH gcc main.c -o test
```
И проверим, что она работает
```console
PlatformSDK:~$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH ./test
```
<details>
Hello, world!<br>
</details><br>


Далее упакуем наши результаты сборки ядра Android как RPM пакет и создадим локальный RPM репозиторий.
```console
PlatformSDK:~$ cd $ANDROID_ROOT
PlatformSDK:~/hadk$ mkdir rpm
PlatformSDK:~/hadk$ cd rpm
PlatformSDK:~/hadk/rpm$ git init
```
<details>
Initialized empty Git repository in /home/stalker/hadk/rpm/.git/<br>
PlatformSDK:~/hadk/rpm$ git submodule add https://github.com/mer-hybris/droid-hal-device dhd<br>
Cloning into 'dhd'...<br>
remote: Enumerating objects: 6, done.<br>
remote: Counting objects: 100% (6/6), done.<br>
remote: Compressing objects: 100% (4/4), done.<br>
remote: Total 2847 (delta 2), reused 6 (delta 2), pack-reused 2841<br>
Receiving objects: 100% (2847/2847), 669.47 KiB | 0 bytes/s, done.<br>
Resolving deltas: 100% (1419/1419), done.<br>
</details><br>

```console
PlatformSDK:~/hadk/rpm$ sed -e "s/@DEVICE@/mido/" \
-e "s/@VENDOR@/xiaomi/" \
-e "s/@DEVICE_PRETTY@/Redmi Note 4/" \
-e "s/@VENDOR_PRETTY@/Xiaomi/" \
dhd/droid-hal-@DEVICE@.spec.template > droid-hal-mido.spec
PlatformSDK:~/hadk/rpm$ git add .
PlatformSDK:~/hadk/rpm$ git commit -m "[dhd] Initial content"
```
<details>
[master (root-commit) ccdedfa] [dhd] Initial content<br>
 3 files changed, 21 insertions(+)<br>
 create mode 100644 .gitmodules<br>
 create mode 160000 dhd<br>
 create mode 100644 droid-hal-mido.spec<br>
 </details><br>

```console
PlatformSDK:~/hadk/rpm$ git remote add droid-hal-mido https://github.com/AndreevDmitry/droid-hal-mido.git
PlatformSDK:~/hadk/rpm$ git push dhm master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
remote: Repository not found.<br>
fatal: repository 'https://github.com/AndreevDmitry/droid-hal-mido.git/' not found<br>
</details><br>

Как видим git ругается, что такого репозитория не существует, создадим его
```console
PlatformSDK:~/hadk/rpm$ curl -u 'AndreevDmitry' https://api.github.com/user/repos -d'{"name":"droid-hal-mido"}'
```
<details>
Enter host password for user 'AndreevDmitry':<br>
{<br>
  "id": 194708778,<br>
  "node_id": "MDEwOlJlcG9zaXRvcnkxOTQ3MDg3Nzg=",<br>
  "name": "droid-hal-mido",<br>
  "full_name": "AndreevDmitry/droid-hal-mido",<br>
  "private": false,<br>
  "owner": {<br>
    "login": "AndreevDmitry",<br>
    "id": 46165782,<br>
    "node_id": "MDQ6VXNlcjQ2MTY1Nzgy",<br>
    "avatar_url": "https://avatars3.githubusercontent.com/u/46165782?v=4",<br>
    "gravatar_id": "",<br>
    "url": "https://api.github.com/users/AndreevDmitry",<br>
    "html_url": "https://github.com/AndreevDmitry",<br>
    "followers_url": "https://api.github.com/users/AndreevDmitry/followers",<br>
    "following_url": "https://api.github.com/users/AndreevDmitry/following{/other_user}",<br>
    "gists_url": "https://api.github.com/users/AndreevDmitry/gists{/gist_id}",<br>
    "starred_url": "https://api.github.com/users/AndreevDmitry/starred{/owner}{/repo}",<br>
    "subscriptions_url": "https://api.github.com/users/AndreevDmitry/subscriptions",<br>
    "organizations_url": "https://api.github.com/users/AndreevDmitry/orgs",<br>
    "repos_url": "https://api.github.com/users/AndreevDmitry/repos",<br>
    "events_url": "https://api.github.com/users/AndreevDmitry/events{/privacy}",<br>
    "received_events_url": "https://api.github.com/users/AndreevDmitry/received_events",<br>
    "type": "User",<br>
    "site_admin": false<br>
  },<br>
  "html_url": "https://github.com/AndreevDmitry/droid-hal-mido",<br>
  "description": null,<br>
  "fork": false,<br>
  "url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido",<br>
  "forks_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/forks",<br>
  "keys_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/keys{/key_id}",<br>
  "collaborators_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/collaborators{/collaborator}",<br>
  "teams_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/teams",<br>
  "hooks_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/hooks",<br>
  "issue_events_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/issues/events{/number}",<br>
  "events_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/events",<br>
  "assignees_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/assignees{/user}",<br>
  "branches_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/branches{/branch}",<br>
  "tags_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/tags",<br>
  "blobs_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/git/blobs{/sha}",<br>
  "git_tags_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/git/tags{/sha}",<br>
  "git_refs_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/git/refs{/sha}",<br>
  "trees_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/git/trees{/sha}",<br>
  "statuses_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/statuses/{sha}",<br>
  "languages_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/languages",<br>
  "stargazers_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/stargazers",<br>
  "contributors_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/contributors",<br>
  "subscribers_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/subscribers",<br>
  "subscription_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/subscription",<br>
  "commits_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/commits{/sha}",<br>
  "git_commits_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/git/commits{/sha}",<br>
  "comments_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/comments{/number}",<br>
  "issue_comment_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/issues/comments{/number}",<br>
  "contents_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/contents/{+path}",<br>
  "compare_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/compare/{base}...{head}",<br>
  "merges_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/merges",<br>
  "archive_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/{archive_format}{/ref}",<br>
  "downloads_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/downloads",<br>
  "issues_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/issues{/number}",<br>
  "pulls_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/pulls{/number}",<br>
  "milestones_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/milestones{/number}",<br>
  "notifications_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/notifications{?since,all,participating}",<br>
  "labels_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/labels{/name}",<br>
  "releases_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/releases{/id}",<br>
  "deployments_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-mido/deployments",<br>
  "created_at": "2019-07-01T16:34:40Z",<br>
  "updated_at": "2019-07-01T16:34:40Z",<br>
  "pushed_at": "2019-07-01T16:34:41Z",<br>
  "git_url": "git://github.com/AndreevDmitry/droid-hal-mido.git",<br>
  "ssh_url": "git@github.com:AndreevDmitry/droid-hal-mido.git",<br>
  "clone_url": "https://github.com/AndreevDmitry/droid-hal-mido.git",<br>
  "svn_url": "https://github.com/AndreevDmitry/droid-hal-mido",<br>
  "homepage": null,<br>
  "size": 0,<br>
  "stargazers_count": 0,<br>
  "watchers_count": 0,<br>
  "language": null,<br>
  "has_issues": true,<br>
  "has_projects": true,<br>
  "has_downloads": true,<br>
  "has_wiki": true,<br>
  "has_pages": false,<br>
  "forks_count": 0,<br>
  "mirror_url": null,<br>
  "archived": false,<br>
  "disabled": false,<br>
  "open_issues_count": 0,<br>
  "license": null,<br>
  "forks": 0,<br>
  "open_issues": 0,<br>
  "watchers": 0,<br>
  "default_branch": "master",<br>
  "permissions": {<br>
    "admin": true,<br>
    "push": true,<br>
    "pull": true<br>
  },<br>
  "allow_squash_merge": true,<br>
  "allow_merge_commit": true,<br>
  "allow_rebase_merge": true,<br>
  "network_count": 0,<br>
  "subscribers_count": 0<br>
}<br>
</details><br>

Отправим изменения в созданный репозиторий
```console
PlatformSDK:~/hadk/rpm$ git push droid-hal-mido master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 4, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (4/4), done.<br>
Writing objects: 100% (4/4), 712 bytes | 0 bytes/s, done.<br>
Total 4 (delta 0), reused 0 (delta 0)<br>
To https://github.com/AndreevDmitry/droid-hal-mido.git<br>
 * [new branch]      master -> master<br>
</details><br>

```console
PlatformSDK:~/hadk/rpm$ cd -
PlatformSDK:~/hadk$ mkdir -p hybris/droid-configs
PlatformSDK:~/hadk$ cd hybris/droid-configs
PlatformSDK:~/hadk/hybris/droid-configs$ git init
```
<details>
Initialized empty Git repository in /home/stalker/hadk/hybris/droid-configs/.git/<br>
PlatformSDK:~/hadk/hybris/droid-configs$ git submodule add https://github.com/mer-hybris/droid-hal-configs droid-configs-device<br>
Cloning into 'droid-configs-device'...<br>
remote: Enumerating objects: 1493, done.<br>
remote: Total 1493 (delta 0), reused 0 (delta 0), pack-reused 1493<br>
Receiving objects: 100% (1493/1493), 280.76 KiB | 0 bytes/s, done.<br>
Resolving deltas: 100% (649/649), done.<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ mkdir rpm
PlatformSDK:~/hadk/hybris/droid-configs$ sed -e "s/@DEVICE@/mido/" \
-e "s/@VENDOR@/xiaomi/" \
-e "s/@DEVICE_PRETTY@/Redmi Note 4/" \
-e "s/@VENDOR_PRETTY@/Xiaomi/" \
droid-configs-device/droid-config-@DEVICE@.spec.template > \
rpm/droid-config-mido.spec
PlatformSDK:~/hadk/hybris/droid-configs$ git add .
PlatformSDK:~/hadk/hybris/droid-configs$ git commit -m "[dcd] Initial content"
```
<details>
[master (root-commit) 8797725] [dcd] Initial content<br><br>
 3 files changed, 29 insertions(+)<br><br>
 create mode 100644 .gitmodules<br><br>
 create mode 160000 droid-configs-device<br><br>
 create mode 100644 rpm/droid-config-mido.spec<br><br>
 </details><br><br>

 ```console
PlatformSDK:~/hadk/hybris/droid-configs$ curl -u 'AndreevDmitry' https://api.github.com/user/repos -d'{"name":"droid-config-mido"}'
```
<details>
Enter host password for user 'AndreevDmitry':<br>
{<br>
  "id": 194710241,<br>
  "node_id": "MDEwOlJlcG9zaXRvcnkxOTQ3MTAyNDE=",<br>
  "name": "droid-config-mido",<br>
  "full_name": "AndreevDmitry/droid-config-mido",<br>
  "private": false,<br>
  "owner": {<br>
    "login": "AndreevDmitry",<br>
    "id": 46165782,<br>
    "node_id": "MDQ6VXNlcjQ2MTY1Nzgy",<br>
    "avatar_url": "https://avatars3.githubusercontent.com/u/46165782?v=4",<br>
    "gravatar_id": "",<br>
    "url": "https://api.github.com/users/AndreevDmitry",<br>
    "html_url": "https://github.com/AndreevDmitry",<br>
    "followers_url": "https://api.github.com/users/AndreevDmitry/followers",<br>
    "following_url": "https://api.github.com/users/AndreevDmitry/following{/other_user}",<br>
    "gists_url": "https://api.github.com/users/AndreevDmitry/gists{/gist_id}",<br>
    "starred_url": "https://api.github.com/users/AndreevDmitry/starred{/owner}{/repo}",<br>
    "subscriptions_url": "https://api.github.com/users/AndreevDmitry/subscriptions",<br>
    "organizations_url": "https://api.github.com/users/AndreevDmitry/orgs",<br>
    "repos_url": "https://api.github.com/users/AndreevDmitry/repos",<br>
    "events_url": "https://api.github.com/users/AndreevDmitry/events{/privacy}",<br>
    "received_events_url": "https://api.github.com/users/AndreevDmitry/received_events",<br>
    "type": "User",<br>
    "site_admin": false<br>
  },<br>
  "html_url": "https://github.com/AndreevDmitry/droid-config-mido",<br>
  "description": null,<br>
  "fork": false,<br>
  "url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido",<br>
  "forks_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/forks",<br>
  "keys_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/keys{/key_id}",<br>
  "collaborators_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/collaborators{/collaborator}",<br>
  "teams_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/teams",<br>
  "hooks_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/hooks",<br>
  "issue_events_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/issues/events{/number}",<br>
  "events_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/events",<br>
  "assignees_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/assignees{/user}",<br>
  "branches_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/branches{/branch}",<br>
  "tags_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/tags",<br>
  "blobs_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/git/blobs{/sha}",<br>
  "git_tags_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/git/tags{/sha}",<br>
  "git_refs_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/git/refs{/sha}",<br>
  "trees_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/git/trees{/sha}",<br>
  "statuses_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/statuses/{sha}",<br>
  "languages_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/languages",<br>
  "stargazers_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/stargazers",<br>
  "contributors_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/contributors",<br>
  "subscribers_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/subscribers",<br>
  "subscription_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/subscription",<br>
  "commits_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/commits{/sha}",<br>
  "git_commits_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/git/commits{/sha}",<br>
  "comments_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/comments{/number}",<br>
  "issue_comment_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/issues/comments{/number}",<br>
  "contents_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/contents/{+path}",<br>
  "compare_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/compare/{base}...{head}",<br>
  "merges_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/merges",<br>
  "archive_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/{archive_format}{/ref}",<br>
  "downloads_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/downloads",<br>
  "issues_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/issues{/number}",<br>
  "pulls_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/pulls{/number}",<br>
  "milestones_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/milestones{/number}",<br>
  "notifications_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/notifications{?since,all,participating}",<br>
  "labels_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/labels{/name}",<br>
  "releases_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/releases{/id}",<br>
  "deployments_url": "https://api.github.com/repos/AndreevDmitry/droid-config-mido/deployments",<br>
  "created_at": "2019-07-01T16:45:46Z",<br>
  "updated_at": "2019-07-01T16:45:46Z",<br>
  "pushed_at": "2019-07-01T16:45:47Z",<br>
  "git_url": "git://github.com/AndreevDmitry/droid-config-mido.git",<br>
  "ssh_url": "git@github.com:AndreevDmitry/droid-config-mido.git",<br>
  "clone_url": "https://github.com/AndreevDmitry/droid-config-mido.git",<br>
  "svn_url": "https://github.com/AndreevDmitry/droid-config-mido",<br>
  "homepage": null,<br>
  "size": 0,<br>
  "stargazers_count": 0,<br>
  "watchers_count": 0,<br>
  "language": null,<br>
  "has_issues": true,<br>
  "has_projects": true,<br>
  "has_downloads": true,<br>
  "has_wiki": true,<br>
  "has_pages": false,<br>
  "forks_count": 0,<br>
  "mirror_url": null,<br>
  "archived": false,<br>
  "disabled": false,<br>
  "open_issues_count": 0,<br>
  "license": null,<br>
  "forks": 0,<br>
  "open_issues": 0,<br>
  "watchers": 0,<br>
  "default_branch": "master",<br>
  "permissions": {<br>
    "admin": true,<br>
    "push": true,<br>
    "pull": true<br>
  },<br>
  "allow_squash_merge": true,<br>
  "allow_merge_commit": true,<br>
  "allow_rebase_merge": true,<br>
  "network_count": 0,<br>
  "subscribers_count": 0<br>
}<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ git remote add dcm https://github.com/AndreevDmitry/droid-config-mido.git
PlatformSDK:~/hadk/hybris/droid-configs$ git push dcm master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 5, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (5/5), done.<br>
Writing objects: 100% (5/5), 1.02 KiB | 0 bytes/s, done.<br>
Total 5 (delta 0), reused 0 (delta 0)<br>
To https://github.com/AndreevDmitry/droid-config-mido.git<br>
 * [new branch]      master -> master<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ cd -
PlatformSDK:~/hadk$ rpm/dhd/helpers/add_new_device.sh
```
<details>
Creating the following nodes:<br>
sparse/<br>
patterns/<br>
patterns/jolla-hw-adaptation-mido.yaml<br>
patterns/jolla-configuration-mido.yaml<br>
/home/stalker/hadk<br>
</details><br>

```console
PlatformSDK:~/hadk$ cd hybris/droid-configs
PlatformSDK:~/hadk/hybris/droid-configs$ COMPOSITOR_CFGS=sparse/var/lib/environment/compositor
PlatformSDK:~/hadk/hybris/droid-configs$ mkdir -p $COMPOSITOR_CFGS
PlatformSDK:~/hadk/hybris/droid-configs$ cat <<EOF >$COMPOSITOR_CFGS/droid-hal-device.conf
#Config for xiaomi/mido
EGL_PLATFORM=hwcomposer
QT_QPA_PLATFORM=hwcomposer
LIPSTICK_OPTIONS=-plugin evdevtouch:/dev/input/event1 -plugin evdevkeyboard:keymap=/usr/share/qt5/keymaps/droid.qmap
EOF
PlatformSDK:~/hadk/hybris/droid-configs$ git add .
PlatformSDK:~/hadk/hybris/droid-configs$ git commit -m "[dcd] Patterns and compositor config"
```
<details>
[master 395a722] [dcd] Patterns and compositor config<br>
 3 files changed, 130 insertions(+)<br>
 create mode 100644 patterns/jolla-configuration-mido.yaml<br>
 create mode 100644 patterns/jolla-hw-adaptation-mido.yaml<br>
 create mode 100644 sparse/var/lib/environment/compositor/droid-hal-device.conf<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ git push dcm master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 12, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (6/6), done.<br>
Writing objects: 100% (11/11), 2.51 KiB | 0 bytes/s, done.<br>
Total 11 (delta 1), reused 0 (delta 0)<br>
remote: Resolving deltas: 100% (1/1), completed with 1 local object.<br>
To https://github.com/AndreevDmitry/droid-config-mido.git<br>
   8797725..395a722  master -> master<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ cd -
PlatformSDK:~/hadk$ mkdir -p hybris/droid-hal-version-mido
PlatformSDK:~/hadk$ cd hybris/droid-hal-version-mido
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git init
```
<details>
Initialized empty Git repository in /home/stalker/hadk/hybris/droid-hal-version-mido/.git/<br>
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git submodule add https://github.com/mer-hybris/droid-hal-version<br>
Cloning into 'droid-hal-version'...<br>
remote: Enumerating objects: 81, done.<br>
remote: Total 81 (delta 0), reused 0 (delta 0), pack-reused 81<br>
Unpacking objects: 100% (81/81), done.<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ mkdir rpm
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ sed -e "s/@DEVICE@/mido/" \
-e "s/@VENDOR@/xiaomi/" \
-e "s/@DEVICE_PRETTY@/Redmi Note 4/" \
-e "s/@VENDOR_PRETTY@/Xiaomi/" \
droid-hal-version/droid-hal-version-@DEVICE@.spec.template > \
rpm/droid-hal-version-mido.spec
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git add .
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git commit -m "[dvd] Initial content"
```
<details>
[master (root-commit) a9825d4] [dvd] Initial content<br>
 3 files changed, 23 insertions(+)<br>
 create mode 100644 .gitmodules<br>
 create mode 160000 droid-hal-version<br>
 create mode 100644 rpm/droid-hal-version-mido.spec<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ curl -u 'AndreevDmitry' https://api.github.com/user/repos -d'{"name":"droid-hal-version-mido"}'
```
<details>
Enter host password for user 'AndreevDmitry':<br>
{<br>
  "id": 194713090,<br>
  "node_id": "MDEwOlJlcG9zaXRvcnkxOTQ3MTMwOTA=",<br>
  "name": "droid-hal-version-mido",<br>
  "full_name": "AndreevDmitry/droid-hal-version-mido",<br>
  "private": false,<br>
  "owner": {<br>
    "login": "AndreevDmitry",<br>
    "id": 46165782,<br>
    "node_id": "MDQ6VXNlcjQ2MTY1Nzgy",<br>
    "avatar_url": "https://avatars3.githubusercontent.com/u/46165782?v=4",<br>
    "gravatar_id": "",<br>
    "url": "https://api.github.com/users/AndreevDmitry",<br>
    "html_url": "https://github.com/AndreevDmitry",<br>
    "followers_url": "https://api.github.com/users/AndreevDmitry/followers",<br>
    "following_url": "https://api.github.com/users/AndreevDmitry/following{/other_user}",<br>
    "gists_url": "https://api.github.com/users/AndreevDmitry/gists{/gist_id}",<br>
    "starred_url": "https://api.github.com/users/AndreevDmitry/starred{/owner}{/repo}",<br>
    "subscriptions_url": "https://api.github.com/users/AndreevDmitry/subscriptions",<br>
    "organizations_url": "https://api.github.com/users/AndreevDmitry/orgs",<br>
    "repos_url": "https://api.github.com/users/AndreevDmitry/repos",<br>
    "events_url": "https://api.github.com/users/AndreevDmitry/events{/privacy}",<br>
    "received_events_url": "https://api.github.com/users/AndreevDmitry/received_events",<br>
    "type": "User",<br>
    "site_admin": false<br>
  },<br>
  "html_url": "https://github.com/AndreevDmitry/droid-hal-version-mido",<br>
  "description": null,<br>
  "fork": false,<br>
  "url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido",<br>
  "forks_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/forks",<br>
  "keys_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/keys{/key_id}",<br>
  "collaborators_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/collaborators{/collaborator}",<br>
  "teams_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/teams",<br>
  "hooks_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/hooks",<br>
  "issue_events_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/issues/events{/number}",<br>
  "events_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/events",<br>
  "assignees_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/assignees{/user}",<br>
  "branches_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/branches{/branch}",<br>
  "tags_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/tags",<br>
  "blobs_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/git/blobs{/sha}",<br>
  "git_tags_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/git/tags{/sha}",<br>
  "git_refs_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/git/refs{/sha}",<br>
  "trees_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/git/trees{/sha}",<br>
  "statuses_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/statuses/{sha}",<br>
  "languages_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/languages",<br>
  "stargazers_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/stargazers",<br>
  "contributors_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/contributors",<br>
  "subscribers_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/subscribers",<br>
  "subscription_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/subscription",<br>
  "commits_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/commits{/sha}",<br>
  "git_commits_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/git/commits{/sha}",<br>
  "comments_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/comments{/number}",<br>
  "issue_comment_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/issues/comments{/number}",<br>
  "contents_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/contents/{+path}",<br>
  "compare_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/compare/{base}...{head}",<br>
  "merges_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/merges",<br>
  "archive_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/{archive_format}{/ref}",<br>
  "downloads_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/downloads",<br>
  "issues_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/issues{/number}",<br>
  "pulls_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/pulls{/number}",<br>
  "milestones_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/milestones{/number}",<br>
  "notifications_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/notifications{?since,all,participating}",<br>
  "labels_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/labels{/name}",<br>
  "releases_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/releases{/id}",<br>
  "deployments_url": "https://api.github.com/repos/AndreevDmitry/droid-hal-version-mido/deployments",<br>
  "created_at": "2019-07-01T17:07:35Z",<br>
  "updated_at": "2019-07-01T17:07:35Z",<br>
  "pushed_at": "2019-07-01T17:07:36Z",<br>
  "git_url": "git://github.com/AndreevDmitry/droid-hal-version-mido.git",<br>
  "ssh_url": "git@github.com:AndreevDmitry/droid-hal-version-mido.git",<br>
  "clone_url": "https://github.com/AndreevDmitry/droid-hal-version-mido.git",<br>
  "svn_url": "https://github.com/AndreevDmitry/droid-hal-version-mido",<br>
  "homepage": null,<br>
  "size": 0,<br>
  "stargazers_count": 0,<br>
  "watchers_count": 0,<br>
  "language": null,<br>
  "has_issues": true,<br>
  "has_projects": true,<br>
  "has_downloads": true,<br>
  "has_wiki": true,<br>
  "has_pages": false,<br>
  "forks_count": 0,<br>
  "mirror_url": null,<br>
  "archived": false,<br>
  "disabled": false,<br>
  "open_issues_count": 0,<br>
  "license": null,<br>
  "forks": 0,<br>
  "open_issues": 0,<br>
  "watchers": 0,<br>
  "default_branch": "master",<br>
  "permissions": {<br>
    "admin": true,<br>
    "push": true,<br>
    "pull": true<br>
  },<br>
  "allow_squash_merge": true,<br>
  "allow_merge_commit": true,<br>
  "allow_rebase_merge": true,<br>
  "network_count": 0,<br>
  "subscribers_count": 0<br>
}<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git remote add dhvm https://github.com/AndreevDmitry/droid-hal-version-mido.git
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ git push dhvm master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 5, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (5/5), done.<br>
Writing objects: 100% (5/5), 819 bytes | 0 bytes/s, done.<br>
Total 5 (delta 0), reused 0 (delta 0)<br>
To https://github.com/AndreevDmitry/droid-hal-version-mido.git<br>
 * [new branch]      master -> master<br>
</details><br>


Обновим манифест и добавим наши репозитории, он должен быть следующим
```console
PlatformSDK:~/hadk/hybris/droid-hal-version-mido$ cd -
PlatformSDK:~/hadk$ cat .repo/local_manifests/mido.xml
```
<details>

`<?xml version="1.0" encoding="UTF-8"?>`<br>
`<manifest>`<br>
`  <project path="device/xiaomi/mido" name="LineageOS/android_device_xiaomi_mido" revision="cm-14.1" />`<br>
`  <project path="kernel/xiaomi/msm8953" name="LineageOS/android_kernel_xiaomi_msm8953" revision="cm-14.1" />`<br>
`  <project path="vendor/xiaomi" name="TheMuppets/proprietary_vendor_xiaomi" revision="cm-14.1" />`<br>
`  <project path="rpm/"name="AndreevDmitry/droid-hal-mido" revision="master" />`<br>
`  <project path="hybris/droid-configs"name="AndreevDmitry/droid-config-mido" revision="master" />`<br>
`  <project path="hybris/droid-hal-version-mido"name="AndreevDmitry/droid-hal-version-mido" revision="master" />`<br>
`</manifest>`<br>
</details><br>

```console
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details>
* Installing required Platform SDK packages<br>
Loading repository data...<br>
Reading installed packages...<br>
'tar' is already installed.<br>
No update candidate for 'tar-1.17-1.3.1.jolla.i486'. The highest available version is already installed.<br>
'rpm-python' is already installed.<br>
No update candidate for 'rpm-python-4.14.1+git7-1.5.1.jolla.i486'. The highest available version is already installed.<br>
'zip' is already installed.<br>
No update candidate for 'zip-3.0-1.2.1.jolla.i486'. The highest available version is already installed.<br>
'android-tools-hadk' is already installed.<br>
No update candidate for 'android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486'. The highest available version is already installed.<br>
Resolving package dependencies...<br>
<br>
The following 2 NEW packages are going to be installed:<br>
  createrepo_c createrepo_c-libs<br>
<br>
2 new packages to install.<br>
Overall download size: 116.2 KiB. Already cached: 0 B. After the operation, additional 299.9 KiB will be used.<br>
Continue? [y/n/...? shows all options] (y): y<br>
Retrieving package createrepo_c-libs-0.10.0+git1-1.2.8.jolla.i486                                  (1/2),  73.9 KiB (194.6 KiB unpacked)<br>
Retrieving: createrepo_c-libs-0.10.0+git1-1.2.8.jolla.i486.rpm .........................................................[done (586 B/s)]<br>
Retrieving package createrepo_c-0.10.0+git1-1.2.8.jolla.i486                                       (2/2),  42.3 KiB (105.3 KiB unpacked)<br>
Retrieving: createrepo_c-0.10.0+git1-1.2.8.jolla.i486.rpm ........................................................................[done]<br>
Checking for file conflicts: .....................................................................................................[done]<br>
(1/2) Installing: createrepo_c-libs-0.10.0+git1-1.2.8.jolla.i486 .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/createrepo_c-libs-0.10.0+git1-1.2.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(2/2) Installing: createrepo_c-0.10.0+git1-1.2.8.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/createrepo_c-0.10.0+git1-1.2.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
* Building rpm/droid-hal-mido.spec<br>
+ export CXXFLAGS<br>
+ FFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -Wformat -Wformat-security -fmessage-length=0 -march=armv7-a -mfloat-abi=hard -mfpu=neon -mthumb -Wno-psabi -I/usr/lib/gfortran/modules'<br>
+ export FFLAGS<br>
+ LD_AS_NEEDED=1<br>
+ export LD_AS_NEEDED<br>
+ echo _target_cpu is armv7hl<br>
_target_cpu is armv7hl<br>
+ grep -q '^TARGET_ARCH := arm64' ./device/xiaomi/mido/BoardConfig.mk<br>
+ echo -e '\n' 'IMPORTANT: some devices in your Android tree are 64bit targets. If your device is aarch64,\n' '           please define droid_target_aarch64 in your .spec, otherwise define droid_target_armv7hl\n' 'NOTE: Currently there is no Sailfish OS ARM 64bit target, so leave PORT_ARCH as armv7hl\n' '      Mixed builds of 64bit Android+Linux Kernel and 32bit Sailfish OS work just fine.'<br>
<br>
 IMPORTANT: some devices in your Android tree are 64bit targets. If your device is aarch64,<br>
            please define droid_target_aarch64 in your .spec, otherwise define droid_target_armv7hl<br>
 NOTE: Currently there is no Sailfish OS ARM 64bit target, so leave PORT_ARCH as armv7hl<br>
       Mixed builds of 64bit Android+Linux Kernel and 32bit Sailfish OS work just fine.<br>
+ exit 1<br>
error: Bad exit status from /var/tmp/rpm-tmp.KP6D3J (%build)<br>
<br>
<br>
RPM build errors:<br>
    Bad exit status from /var/tmp/rpm-tmp.KP6D3J (%build)<br>
* Check /home/stalker/hadk/droid-hal-mido.log for full log.<br>
!! building of package failed<br>
</details><br>


Похоже имеет место несовместимость архитектуры процессора в конфигурации ядра и конфигурации droid-hal. Исправим это добавив строку<br>
` %define droid_target_aarch64 1`<br>
в `$ANDROID_ROOT/rpm/droid-hal-mido.spec`

В итоге получим следующее
```console
PlatformSDK:~/hadk$ cat rpm/droid-hal-mido.spec
```
<details>
# These and other macros are documented in dhd/droid-hal-device.inc<br>
# Feel free to cleanup this file by removing comments, once you have memorised them ;)<br>
<br>
%define device mido<br>
%define vendor xiaomi<br>
<br>
%define vendor_pretty Xiaomi<br>
%define device_pretty Redmi Note 4<br>
<br>
%define droid_target_aarch64 1<br>
<br>
%define installable_zip 1<br>
<br>
%include rpm/dhd/droid-hal-device.inc<br>
<br>
# IMPORTANT if you want to comment out any macros in your .spec, delete the %<br>
# sign, otherwise they will remain defined! E.g.:<br>
#define some_macro "I'll not be defined because I don't have % in front"<br>
</details><br>

Заливаем наши изменения на github (это очень важно, т.к. в дальнейшем скрипт сборщика пакетов будет синхронизироваться с github)
```console
PlatformSDK:~/hadk$ cd rpm
PlatformSDK:~/hadk/rpm$ git add .
PlatformSDK:~/hadk/rpm$ git commit -m "Add aarch64 target"
```
<details>
[master 3453226] Add aarch64 target<br>
 1 file changed, 2 insertions(+)<br>
</details><br>

```console
PlatformSDK:~/hadk/rpm$ git push droid-hal-mido master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 5, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (3/3), done.<br>
Writing objects: 100% (3/3), 317 bytes | 0 bytes/s, done.<br>
Total 3 (delta 2), reused 0 (delta 0)<br>
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.<br>
To https://github.com/AndreevDmitry/droid-hal-mido.git<br>
   ad8ef90..3453226  master -> master<br>
</details><br>

Запускаем снова сборку пакетов
```console
PlatformSDK:~/hadk/rpm$ cd ..
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details>
* Building rpm/droid-hal-mido.spec<br>
   /init.qcom.usb.sh<br>
   /property_contexts<br>
   /sdcard<br>
   /selinux_version<br>
   /service_contexts<br>
   /vendor<br>
<br>
<br>
RPM build errors:<br>
    Installed (but unpackaged) file(s) found:<br>
   /bugreports<br>
   /d<br>
   /file_contexts.bin<br>
   /init.qcom.sh<br>
   /init.qcom.usb.sh<br>
   /property_contexts<br>
   /sdcard<br>
   /selinux_version<br>
   /service_contexts<br>
   /vendor<br>
* Check /home/stalker/hadk/droid-hal-mido.log for full log.<br>
!! building of package failed<br>
</details><br>


В HADK это ситуация описана в пункте 7.2.2, решается добавлением
```c
%define straggler_files \
/init.qcom.sh \
/init.qcom.usb.sh \
/bugreports \
/d \
/file_contexts.bin \
/property_contexts \
/sdcard \
/selinux_version \
/service_contexts \
/vendor \
%{nil}
```
в тот же `$ANDROID_ROOT/rpm/droid-hal-mido.spec`

```console
PlatformSDK:~/hadk$ cat rpm/droid-hal-mido.spec
```
<details>
# These and other macros are documented in dhd/droid-hal-device.inc<br>
# Feel free to cleanup this file by removing comments, once you have memorised them ;)<br>
<br>
%define device mido<br>
%define vendor xiaomi<br>
<br>
%define vendor_pretty Xiaomi<br>
%define device_pretty Redmi Note 4<br>
<br>
%define droid_target_aarch64 1<br>
<br>
%define installable_zip 1<br>
<br>
%define straggler_files \<br>
/init.qcom.sh \<br>
/init.qcom.usb.sh \<br>
/bugreports \<br>
/d \<br>
/file_contexts.bin \<br>
/property_contexts \<br>
/sdcard \<br>
/selinux_version \<br>
/service_contexts \<br>
/vendor \<br>
%{nil}<br>
<br>
%include rpm/dhd/droid-hal-device.inc<br>
<br>
# IMPORTANT if you want to comment out any macros in your .spec, delete the %<br>
# sign, otherwise they will remain defined! E.g.:<br>
#define some_macro "I'll not be defined because I don't have % in front"<br>
</details><br>


Заливаем изменения на github
```console
PlatformSDK:~/hadk$ cd rpm
PlatformSDK:~/hadk/rpm$ git add .
PlatformSDK:~/hadk/rpm$ git commit -m "Add straggler files"
```
<details>
[master 8be11af] Add straggler files<br>
 1 file changed, 13 insertions(+)<br>
</details><br>
```console
PlatformSDK:~/hadk/rpm$ git push droid-hal-mido master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 5, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (3/3), done.<br>
Writing objects: 100% (3/3), 424 bytes | 0 bytes/s, done.<br>
Total 3 (delta 2), reused 0 (delta 0)<br>
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.<br>
To https://github.com/AndreevDmitry/droid-hal-mido.git<br>
   3453226..8be11af  master -> master<br>
</details><br>

Также согласно HADK добавим<br>
`- droid-hal-mido-detritus`<br>
в `$ANDROID_ROOT/hybris/droid-configs/patterns/jolla-hw-adaptation-mido.yaml`

Получим следующее
```console
PlatformSDK:~/hadk$ cat hybris/droid-configs/patterns/jolla-hw-adaptation-mido.yaml
```
<details>
# Feel free to disable non-critical HA parts during devel by commenting lines out<br>
# Generated in hadk by executing: rpm/dhd/helpers/add_new_device.sh<br>
<br>
Description: Pattern with packages for mido HW Adaptation<br>
Name: jolla-hw-adaptation-mido<br>
Requires:<br>
- droid-hal-mido<br>
- droid-hal-mido-img-boot<br>
- droid-hal-mido-kernel-modules<br>
- droid-config-mido-sailfish<br>
- droid-config-mido-pulseaudio-settings<br>
- droid-config-mido-policy-settings<br>
- droid-config-mido-preinit-plugin<br>
- droid-config-mido-flashing<br>
- droid-config-mido-bluez5<br>
- droid-hal-version-mido<br>
- droid-hal-mido-detritus<br>
<br>
# Hybris packages<br>
- libhybris-libEGL<br>
- libhybris-libGLESv2<br>
- libhybris-libwayland-egl<br>
<br>
# Sensors<br>
- hybris-libsensorfw-qt5<br>
<br>
# Vibra<br>
- ngfd-plugin-native-vibrator<br>
- qt5-feedback-haptics-native-vibrator<br>
<br>
# Needed for /dev/touchscreen symlink<br>
- qt5-plugin-generic-evdev<br>
<br>
- pulseaudio-modules-droid<br>
# for audio recording to work:<br>
- qt5-qtmultimedia-plugin-mediaservice-gstmediacapture<br>
<br>
# These need to be per-device due to differing backends (fbdev, eglfs, hwc, ..?)<br>
- qt5-qtwayland-wayland_egl<br>
- qt5-qpa-hwcomposer-plugin<br>
- qtscenegraph-adaptation<br>
<br>
# Add GStreamer v1.0 as standard<br>
- gstreamer1.0<br>
- gstreamer1.0-plugins-good<br>
- gstreamer1.0-plugins-base<br>
- gstreamer1.0-plugins-bad<br>
- nemo-gstreamer1.0-interfaces<br>
# For devices with droidmedia and gst-droid built, see HADK pdf for more information<br>
#- gstreamer1.0-droid<br>
<br>
# This is needed for notification LEDs<br>
- mce-plugin-libhybris<br>
<br>
## USB mode controller<br>
# Enables mode selector upon plugging USB cable:<br>
- usb-moded<br>
- usb-moded-defaults-android<br>
- usb-moded-developer-mode-android<br>
<br>
# Extra useful modes not officially supported:<br>
# might need some configuration to get working<br>
#- usb-moded-mass-storage-android-config<br>
# working but careful with roaming!<br>
- usb-moded-connection-sharing-android-config<br>
# android diag mode only usable for certain android tools<br>
#- usb-moded-diag-mode-android<br>
<br>
# hammerhead, grouper, and maguro use this in scripts, so include for all<br>
- rfkill<br>
<br>
# enable device lock and allow to select untrusted software<br>
- jolla-devicelock-daemon-encsfa<br>
<br>
# For GPS<br>
- geoclue-provider-hybris<br>
<br>
# For FM radio on some QCOM devices<br>
#- qt5-qtmultimedia-plugin-mediaservice-irisradio<br>
#- jolla-mediaplayer-radio<br>
<br>
# For devices with SD Card<br>
#- sd-utils<br>
<br>
Summary: Jolla HW Adaptation mido<br>
</details><br>

Добавим изменения в наш репозиторий
```console
PlatformSDK:~/hadk$ cd hybris/droid-configs/
PlatformSDK:~/hadk/hybris/droid-configs$ git add .
PlatformSDK:~/hadk/hybris/droid-configs$ git commit -m "Add droid-hal-mido-detritus"
```
<details>
[master b6fd7ed] Add droid-hal-mido-detritus<br>
 1 file changed, 1 insertion(+)<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs$ git push dcm master
```
<details>
Username for 'https://github.com': AndreevDmitry<br>
Password for 'https://AndreevDmitry@github.com':<br>
Counting objects: 7, done.<br>
Delta compression using up to 16 threads.<br>
Compressing objects: 100% (4/4), done.<br>
Writing objects: 100% (4/4), 389 bytes | 0 bytes/s, done.<br>
Total 4 (delta 3), reused 0 (delta 0)<br>
remote: Resolving deltas: 100% (3/3), completed with 3 local objects.<br>
To https://github.com/AndreevDmitry/droid-config-mido.git<br>
   395a722..b6fd7ed  master -> master<br>
</details><br>

Снова запускаем сборку пакетов
```console
PlatformSDK:~/hadk/hybris/droid-configs$ cd -
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details>
* Building rpm/droid-hal-mido.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of droid-hal-mido finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/community-adaptation-localbuild.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of community-adaptation finished successfully<br>
* Building rpm/droid-config-mido.spec<br>
warning: Macro expanded in comment on line 157: %{name} contains ssu.ini file which is needed to build kickstarts<br>
<br>
Package 'ssu-kickstart-configuration' not found.<br>
NOTICE: No tags describe the HEAD, will not fix package version.<br>
warning: Macro expanded in comment on line 157: %{name} contains ssu.ini file which is needed to build kickstarts<br>
<br>
error: Failed build dependencies:<br>
	community-adaptation is needed by droid-config-mido-1-1.armv7hl<br>
	pkgconfig(android-headers) is needed by droid-config-mido-1-1.armv7hl<br>
	repomd-pattern-builder is needed by droid-config-mido-1-1.armv7hl<br>
	ssu-kickstart-configuration is needed by droid-config-mido-1-1.armv7hl<br>
Building target platforms: armv7hl-meego-linux<br>
Building for target armv7hl-meego-linux<br>
* Check /home/stalker/hadk/hybris/droid-configs.log for full log.<br>
!! building of package failed<br>
</details><br>


Теперь сборщик не может найти пакет "ssu-kickstart-configuration", т.к раньше он назывался "ssu-kickstart-configuration-jolla".<br>
Решение описано здесь http://www.merproject.org/logs/%23sailfishos-porters/%23sailfishos-porters.2019-05-10.log.html
```console
PlatformSDK:~/hadk$ cd hybris/droid-configs/droid-configs-device/
PlatformSDK:~/hadk/hybris/droid-configs/droid-configs-device$ git reset --hard 746de04dd318127044d163a4f4f69b1867789eb8
```
<details>
HEAD is now at 746de04 Merge pull request #149 from jusa/bluez5<br>
</details><br>

```console
PlatformSDK:~/hadk/hybris/droid-configs/droid-configs-device$ cd -
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details><br>
* Building rpm/droid-hal-mido.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of droid-hal-mido finished successfully<br>
* pulling updates...<br>
* Building rpm/community-adaptation-localbuild.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of community-adaptation finished successfully<br>
* Building rpm/droid-config-mido.spec<br>
* Building successful, adding packages to repo<br>
Retrieving repository 'adaptation-community-common' metadata .....................................................................[done]<br>
Building repository 'adaptation-community-common' cache ..........................................................................[done]<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of droid-configs finished successfully<br>
Changing domain from sailfish to sales<br>
DBus unavailable, falling back to libssu<br>
Forcing raw metadata refresh<br>
Retrieving repository 'adaptation-community-common' metadata .....................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'adaptation-community-common' cache ..........................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'hotfixes' metadata ........................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'hotfixes' cache .............................................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'jolla' metadata ...........................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'jolla' cache ................................................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
Loading repository data...<br>
Reading installed packages...<br>
'droid-hal-mido-devel' is already installed.<br>
No update candidate for 'droid-hal-mido-devel-0.0.6-201907021704.armv7hl'. The highest available version is already installed.<br>
Resolving package dependencies...<br>
<br>
Nothing to do.<br>
Build libhybris? [Y/n/all]all<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* enabling debugging in libhybris...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/libhybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of libhybris finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/pulseaudio-modules-droid.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of pulseaudio-modules-droid finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/mce-plugin-libhybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of mce-plugin-libhybris finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/ngfd-plugin-native-vibrator.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of ngfd-plugin-droid-vibrator finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/qt5-feedback-haptics-native-vibrator.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-feedback-haptics-droid-vibrator finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/qt5-qpa-hwcomposer-plugin.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-qpa-hwcomposer-plugin finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/qt5-qpa-surfaceflinger-plugin.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-qpa-surfaceflinger-plugin finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/qtscenegraph-adaptation-droid.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qtscenegraph-adaptation finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/sensorfw-qt5-hybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of sensorfw finished successfully<br>
* Source code directory doesn't exist, cloning repository<br>
* pulling updates...<br>
* Building rpm/geoclue-providers-hybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ..................................................................................[done]<br>
Building repository 'local-mido-hal' cache .......................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of geoclue-providers-hybris finished successfully<br>
* Building rpm/droid-hal-version-mido.spec<br>
Choose from above solutions by number or cancel [1/2/c] (c): c<br>
NOTICE: No tags describe the HEAD, will not fix package version.<br>
error: Failed build dependencies:<br>
	droid-config is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-preinit-plugins is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-pulseaudio-settings is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-sailfish is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-img-boot is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-img-recovery is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-kernel is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	hybris-libsensorfw-qt5 is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	mce-plugin-libhybris is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	ngfd-plugin-native-vibrator is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	pulseaudio-modules-droid is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qt5-feedback-haptics-native-vibrator is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qt5-qpa-hwcomposer-plugin is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qtscenegraph-adaptation is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
Building target platforms: armv7hl-meego-linux<br>
Building for target armv7hl-meego-linux<br>
* Check /home/stalker/hadk/hybris/droid-hal-version-mido.log for full log.<br>
!! building of package failed<br>
</details><br>


Посмотрим что случилось во время сборки droid-hal-verion-mido
```console
PlatformSDK:~/hadk$ cat hybris/droid-hal-version-mido.log
```
<details>
Problem: droid-config-mido-pulseaudio-settings-1-1.armv7hl requires pulseaudio-modules-nemo-mainvolume >= 11.1.24, but this requirement cannot be provided<br>
  uninstallable providers: pulseaudio-modules-nemo-mainvolume-11.1.25-1.5.1.jolla.armv7hl[jolla]<br>
 Solution 1: do not ask to install a solvable providing droid-config-pulseaudio-settings<br>
 Solution 2: break droid-config-mido-pulseaudio-settings-1-1.armv7hl by ignoring some of its dependencies<br>
<br>
Choose from above solutions by number or cancel [1/2/c] (c): c<br>
NOTICE: No tags describe the HEAD, will not fix package version.<br>
error: Failed build dependencies:<br>
	droid-config is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-preinit-plugins is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-pulseaudio-settings is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-config-sailfish is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-img-boot is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-img-recovery is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	droid-hal-kernel is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	hybris-libsensorfw-qt5 is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	mce-plugin-libhybris is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	ngfd-plugin-native-vibrator is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	pulseaudio-modules-droid is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qt5-feedback-haptics-native-vibrator is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qt5-qpa-hwcomposer-plugin is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
	qtscenegraph-adaptation is needed by droid-hal-version-mido-0.0.1-1.armv7hl<br>
Building target platforms: armv7hl-meego-linux<br>
Building for target armv7hl-meego-linux<br>
</details><br>


Также почитав http://www.merproject.org/logs/%23sailfishos-porters/%23sailfishos-porters.2019-05-10.log.html возникла мысль:<br>
конфиг ждет, что пакет `pulseaudio-modules-nemo` будет больше версии `11.1.24`, найдем какой версии Sailfish соответствует текущая версия в истории изменений
https://together.jolla.com/question/184470/changelog-220-mouhijoki-released/#184470-pulseaudio-modules-nemo
Получается текущие пакеты для версии `Sailfish 2.2`

Проверим версию песочницы
```console
PlatformSDK:~/hadk$ sudo ssu re
Device release is currently: 3.0.2.8
```

Проверим версию песочнницы для `xiaomi-mido-armv7hl`
```console
PlatformSDK:~/hadk$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R ssu re
Device release is currently: 2.2.1.18
```

Видим, что версии не совпадают, обновим все до последней (на момент написания перевода) версии 3.0.3.10
```console
PlatformSDK:~/hadk$ sudo ssu re 3.0.3.10
```
<details>
Changing release from 3.0.2.8 to 3.0.3.10<br>
Your device is now in release mode!<br>
DBus unavailable, falling back to libssu<br>
</details><br>

```console
PlatformSDK:~/hadk$ sudo zypper ref
```
<details>
Retrieving repository 'adaptation0' metadata .....................................................................................[done]<br>
Building repository 'adaptation0' cache ..........................................................................................[done]<br>
Retrieving repository 'customer-jolla' metadata ..................................................................................[done]<br>
Building repository 'customer-jolla' cache .......................................................................................[done]<br>
Retrieving repository 'hotfixes' metadata ........................................................................................[done]<br>
Building repository 'hotfixes' cache .............................................................................................[done]<br>
Retrieving repository 'jolla' metadata ...........................................................................................[done]<br>
Building repository 'jolla' cache ................................................................................................[done]<br>
Retrieving repository 'sdk' metadata .............................................................................................[done]<br>
Building repository 'sdk' cache ..................................................................................................[done]<br>
All repositories have been refreshed.<br>
</details><br>

```console
PlatformSDK:~/hadk$ sudo zypper dup
```
<details>
Warning: You are about to do a distribution upgrade with all enabled repositories. Make sure these repositories are compatible before you continue. See 'man zypper' for more information about this command.<br>
Loading repository data...<br>
Reading installed packages...<br>
Computing distribution upgrade...<br>
<br>
The following 3 NEW packages are going to be installed:<br>
  cryptsetup-libs json-c libicu<br>
<br>
The following package is going to be REMOVED:<br>
  libicu52<br>
<br>
The following 81 packages are going to be upgraded:<br>
  PackageKit PackageKit-glib PackageKit-zypp btrfs-progs connman connman-configs-sailfish-sdk cpio curl e2fsprogs e2fsprogs-libs glibc<br>
  glibc-common iptables iptables-ipv6 kbd libblkid libcom_err libcurl libdbusaccess libfdisk libgcc libgrilio libmount libpcap libsb2<br>
  libsmartcols libss libstdc++ libuuid libzypp lsof mobile-broadband-provider-info ncurses ncurses-base ncurses-libs nss nss-sysinit<br>
  ofono ofono-configs-build-engine openssh openssh-clients openssl-libs p11-kit p11-kit-nss-ckbi p11-kit-trust<br>
  patterns-sailfish-configuration-platform-sdk-chroot patterns-sailfish-core patterns-sailfish-image-creation-tools<br>
  patterns-sailfish-packaging-tools patterns-sailfish-platform-sdk patterns-sailfish-sb2-host pcre platform-sdk-support python<br>
  python-libs qt5-qtcore qt5-qtdbus qt5-qtnetwork qt5-qtxml rpm rpm-build rpm-libs rpm-python sailfish-version sailfish-version-variant<br>
  scratchbox2 sdk-chroot sdk-sb2-config sdk-utils shared-mime-info sqlite-libs ssu-vendor-data-jolla systemd systemd-config-sailfish<br>
  systemd-libs util-linux vim-common vim-enhanced vim-filesystem vm-configs zlib<br>
<br>
The following 191 packages are going to be downgraded:<br>
  android-tools-hadk augeas-libs basesystem bash binutils bluez bluez-libs boost-filesystem boost-system boost-thread bsdtar busybox<br>
  busybox-symlinks-gzip bzip2 bzip2-libs ca-certificates connman-qt5 cor coreutils crda createrepo_c createrepo_c-libs db4 db4-utils<br>
  dbus dbus-libs deltarpm desktop-file-utils device-mapper device-mapper-event device-mapper-event-libs device-mapper-libs diffutils<br>
  dosfstools elfutils elfutils-libelf elfutils-libs expat fakeroot feature-jolla-sdk file file-libs filesystem findutils fontconfig<br>
  fontpackages-filesystem freetype fuse fuse-libs gawk gdbm git glib2 gnupg2 gnutls gpgme grep groff hwdata info iproute iw jolla-ca<br>
  kf5bluezqt-bluez4 kmod kmod-libs kpartx less libacl libarchive libattr libcap libdbuslogserver-dbus libffi libgcrypt libglibutil<br>
  libgofono libgofonoext libgpg-error libgsupplicant libiphb libksba liblua libmce-glib libnl libqtaround2 libshadowutils libsolv-tools<br>
  libsolv0 libtasn1 libusb libuser libutempter libwspcodec libxml2 libxslt lvm2 lvm2-libs lzo mic mtools net-tools nspr obex-capability<br>
  obexd oneshot openconnect openvpn osc p7zip-full pacrunner pacrunner-python pam parted passwd patch perl perl-Error perl-Filter<br>
  perl-Git perl-Module-Pluggable perl-Pod-Escapes perl-Pod-Parser perl-Pod-Perldoc perl-Pod-Simple perl-Scalar-List-Utils perl-Socket<br>
  perl-libs perl-macros perl-parent perl-threads perl-threads-shared pkgconfig polkit popt ppp ppp-libs pptp procps psmisc pth<br>
  python-M2Crypto python-cheetah python-distro python-lxml python-pycurl python-setuptools python-urlgrabber python-yaml python-zypp<br>
  qemu-usermode qemu-usermode-common qemu-usermode-static qt5-qtsysteminfo readline repomd-pattern-builder rootfiles rsync sailfish-ca<br>
  sdk-configs sdk-register sed setup shadow-utils spectacle squashfs-tools ssu ssu-network-proxy-plugin statefs<br>
  statefs-contextkit-subscriber statefs-pp statefs-provider-inout-power statefs-qt5 strace sudo syslinux systemd-user-session-targets<br>
  tar tzdata unzip vpnc wireless-regdb wpa_supplicant xdg-user-dirs xdg-utils xl2tpd xz xz-libs xz-lzma-compat zip zypper<br>
<br>
81 packages to upgrade, 191 to downgrade, 3 new, 1 to remove.<br>
Overall download size: 98.5 MiB. Already cached: 0 B. After the operation, additional 3.2 MiB will be used.<br>
Continue? [y/n/...? shows all options] (y): y<br>
Retrieving package ofono-configs-build-engine-0.15.8-1.9.11.jolla.noarch                         (1/275),  24.1 KiB (   26   B unpacked)<br>
Retrieving: ofono-configs-build-engine-0.15.8-1.9.11.jolla.noarch.rpm ................................................[done (4.0 KiB/s)]<br>
Retrieving package fontpackages-filesystem-1.44-1.1.8.jolla.noarch                               (2/275),   9.0 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/fontpackages-filesystem-1.44-1.2.1.jolla_1.44_1.1.8.jolla.noarch.drpm, 5.0 KiB<br>
Retrieving: fontpackages-filesystem-1.44-1.2.1.jolla_1.44_1.1.8.jolla.noarch.drpm ....................................[done (1.2 KiB/s)]<br>
Applying delta: ./fontpackages-filesystem-1.44-1.2.1.jolla_1.44_1.1.8.jolla.noarch.drpm ..........................................[done]<br>
Retrieving package jolla-ca-0.9-1.2.9.jolla.noarch                                               (3/275),  15.1 KiB ( 22.5 KiB unpacked)<br>
Retrieving delta: ./drpms/jolla-ca-0.9-1.3.2.jolla_0.9_1.2.9.jolla.noarch.drpm, 3.8 KiB<br>
Retrieving: jolla-ca-0.9-1.3.2.jolla_0.9_1.2.9.jolla.noarch.drpm .................................................................[done]<br>
Applying delta: ./jolla-ca-0.9-1.3.2.jolla_0.9_1.2.9.jolla.noarch.drpm ...........................................................[done]<br>
Retrieving package libgcc-4.9.4-1.2.4.jolla.i486                                                 (4/275),  71.1 KiB (126.0 KiB unpacked)<br>
Retrieving delta: ./drpms/libgcc-4.8.3-1.2.1.jolla_4.9.4_1.2.4.jolla.i486.drpm, 57.8 KiB<br>
Retrieving: libgcc-4.8.3-1.2.1.jolla_4.9.4_1.2.4.jolla.i486.drpm .....................................................[done (2.6 KiB/s)]<br>
Applying delta: ./libgcc-4.8.3-1.2.1.jolla_4.9.4_1.2.4.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch              (5/275),  55.8 KiB (329.0 KiB unpacked)<br>
Retrieving delta: ./drpms/mobile-broadband-provider-info-20131125+git66-1.3.1.jolla_20131125+git68_1.3.2.jolla.noarch.drpm, 14.9 KiB<br>
Retrieving: mobile-broadband-provider-info-20131125+git66-1.3.1.jolla_20131125+git68_1.3.2.jolla.noarch.drpm .....................[done]<br>
Applying delta: ./mobile-broadband-provider-info-20131125+git66-1.3.1.jolla_20131125+git68_1.3.2.jolla.noarch.drpm ...............[done]<br>
Retrieving package ncurses-base-6.1+git1-1.3.4.jolla.i486                                        (6/275),  57.6 KiB (320.4 KiB unpacked)<br>
Retrieving delta: ./drpms/ncurses-base-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm, 43.3 KiB<br>
Retrieving: ncurses-base-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./ncurses-base-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm ....................................................[done]<br>
Retrieving package rootfiles-8.1+git2-1.2.7.jolla.noarch                                         (7/275),   8.3 KiB (  599   B unpacked)<br>
Retrieving delta: ./drpms/rootfiles-8.1+git2-1.3.1.jolla_8.1+git2_1.2.7.jolla.noarch.drpm, 4.4 KiB<br>
Retrieving: rootfiles-8.1+git2-1.3.1.jolla_8.1+git2_1.2.7.jolla.noarch.drpm ......................................................[done]<br>
Applying delta: ./rootfiles-8.1+git2-1.3.1.jolla_8.1+git2_1.2.7.jolla.noarch.drpm ................................................[done]<br>
Retrieving package sailfish-ca-0.1.1-1.2.9.jolla.noarch                                          (8/275),  11.3 KiB (  9.1 KiB unpacked)<br>
Retrieving delta: ./drpms/sailfish-ca-0.1.1-1.3.2.jolla_0.1.1_1.2.9.jolla.noarch.drpm, 3.6 KiB<br>
Retrieving: sailfish-ca-0.1.1-1.3.2.jolla_0.1.1_1.2.9.jolla.noarch.drpm ..........................................................[done]<br>
Applying delta: ./sailfish-ca-0.1.1-1.3.2.jolla_0.1.1_1.2.9.jolla.noarch.drpm ....................................................[done]<br>
Retrieving package setup-2.8.56-1.2.4.jolla.noarch                                               (9/275), 116.1 KiB (663.0 KiB unpacked)<br>
Retrieving delta: ./drpms/setup-2.8.56-1.3.1.jolla_2.8.56_1.2.4.jolla.noarch.drpm, 111.3 KiB<br>
Retrieving: setup-2.8.56-1.3.1.jolla_2.8.56_1.2.4.jolla.noarch.drpm ..............................................................[done]<br>
Applying delta: ./setup-2.8.56-1.3.1.jolla_2.8.56_1.2.4.jolla.noarch.drpm ........................................................[done]<br>
Retrieving package tzdata-2017b-1.1.8.jolla.noarch                                              (10/275), 346.5 KiB (  2.3 MiB unpacked)<br>
Retrieving: tzdata-2017b-1.1.8.jolla.noarch.rpm ..................................................................................[done]<br>
Retrieving package vim-filesystem-7.3.629-1.2.3.jolla.i486                                      (11/275),  10.0 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/vim-filesystem-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm, 5.9 KiB<br>
Retrieving: vim-filesystem-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm ...........................................[done (154 B/s)]<br>
Applying delta: ./vim-filesystem-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm ...............................................[done]<br>
Retrieving package wireless-regdb-2016.06.10+git1-1.2.6.jolla.noarch                            (12/275),  11.2 KiB (  6.6 KiB unpacked)<br>
Retrieving delta: ./drpms/wireless-regdb-2016.06.10+git1-1.3.1.jolla_2016.06.10+git1_1.2.6.jolla.noarch.drpm, 5.0 KiB<br>
Retrieving: wireless-regdb-2016.06.10+git1-1.3.1.jolla_2016.06.10+git1_1.2.6.jolla.noarch.drpm ...................................[done]<br>
Applying delta: ./wireless-regdb-2016.06.10+git1-1.3.1.jolla_2016.06.10+git1_1.2.6.jolla.noarch.drpm .............................[done]<br>
Retrieving package filesystem-3.1-1.1.8.jolla.noarch                                            (13/275), 910.7 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/filesystem-3.1-1.2.1.jolla_3.1_1.1.8.jolla.noarch.drpm, 902.9 KiB<br>
Retrieving: filesystem-3.1-1.2.1.jolla_3.1_1.1.8.jolla.noarch.drpm .....................................................[done (154 B/s)]<br>
Applying delta: ./filesystem-3.1-1.2.1.jolla_3.1_1.1.8.jolla.noarch.drpm .........................................................[done]<br>
Retrieving package glibc-2.25+git5-1.4.1.jolla.i486                                             (14/275),   2.5 MiB ( 10.6 MiB unpacked)<br>
Retrieving delta: ./drpms/glibc-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm, 1.6 MiB<br>
Retrieving: glibc-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm ............................................[done (1.2 MiB/s)]<br>
Applying delta: ./glibc-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm ..................................................[done]<br>
Retrieving package basesystem-11+git1-1.2.7.jolla.noarch                                        (15/275),   8.0 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/basesystem-11+git1-1.3.1.jolla_11+git1_1.2.7.jolla.noarch.drpm, 4.1 KiB<br>
Retrieving: basesystem-11+git1-1.3.1.jolla_11+git1_1.2.7.jolla.noarch.drpm .......................................................[done]<br>
Applying delta: ./basesystem-11+git1-1.3.1.jolla_11+git1_1.2.7.jolla.noarch.drpm .................................................[done]<br>
Retrieving package ncurses-libs-6.1+git1-1.3.4.jolla.i486                                       (16/275), 238.7 KiB (756.3 KiB unpacked)<br>
Retrieving delta: ./drpms/ncurses-libs-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm, 226.8 KiB<br>
Retrieving: ncurses-libs-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm ................................................[done (154 B/s)]<br>
Applying delta: ./ncurses-libs-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm ....................................................[done]<br>
Retrieving package bash-1:3.2.57-1.2.4.jolla.i486                                               (17/275), 322.2 KiB (778.8 KiB unpacked)<br>
Retrieving delta: ./drpms/bash-3.2.57-1.3.1.jolla_3.2.57_1.2.4.jolla.i486.drpm, 310.0 KiB<br>
Retrieving: bash-3.2.57-1.3.1.jolla_3.2.57_1.2.4.jolla.i486.drpm .....................................................[done (1.2 KiB/s)]<br>
Applying delta: ./bash-3.2.57-1.3.1.jolla_3.2.57_1.2.4.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package glibc-common-2.25+git5-1.4.1.jolla.i486                                      (18/275),   4.1 MiB ( 14.1 MiB unpacked)<br>
Retrieving delta: ./drpms/glibc-common-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm, 1.4 MiB<br>
Retrieving: glibc-common-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm .................................................[done]<br>
Applying delta: ./glibc-common-2.19+6.13.1-1.3.1.jolla_2.25+git5_1.4.1.jolla.i486.drpm ...........................................[done]<br>
Retrieving package zlib-1.2.11+git1-1.4.4.jolla.i486                                            (19/275),  70.2 KiB (139.0 KiB unpacked)<br>
Retrieving delta: ./drpms/zlib-1.2.8+git3-1.4.1.jolla_1.2.11+git1_1.4.4.jolla.i486.drpm, 57.0 KiB<br>
Retrieving: zlib-1.2.8+git3-1.4.1.jolla_1.2.11+git1_1.4.4.jolla.i486.drpm ............................................[done (2.6 KiB/s)]<br>
Applying delta: ./zlib-1.2.8+git3-1.4.1.jolla_1.2.11+git1_1.4.4.jolla.i486.drpm ..................................................[done]<br>
Retrieving package zip-3.0-1.1.10.jolla.i486                                                    (20/275), 239.3 KiB (798.3 KiB unpacked)<br>
Retrieving delta: ./drpms/zip-3.0-1.2.1.jolla_3.0_1.1.10.jolla.i486.drpm, 129.3 KiB<br>
Retrieving: zip-3.0-1.2.1.jolla_3.0_1.1.10.jolla.i486.drpm .......................................................................[done]<br>
Applying delta: ./zip-3.0-1.2.1.jolla_3.0_1.1.10.jolla.i486.drpm .................................................................[done]<br>
Retrieving package xz-libs-5.0.4-1.2.6.jolla.i486                                               (21/275),  73.6 KiB (139.1 KiB unpacked)<br>
Retrieving delta: ./drpms/xz-libs-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm, 51.2 KiB<br>
Retrieving: xz-libs-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./xz-libs-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package xdg-user-dirs-0.16+git1-1.2.7.jolla.i486                                     (22/275),  52.9 KiB (152.1 KiB unpacked)<br>
Retrieving delta: ./drpms/xdg-user-dirs-0.16+git1-1.3.1.jolla_0.16+git1_1.2.7.jolla.i486.drpm, 26.9 KiB<br>
Retrieving: xdg-user-dirs-0.16+git1-1.3.1.jolla_0.16+git1_1.2.7.jolla.i486.drpm ..................................................[done]<br>
Applying delta: ./xdg-user-dirs-0.16+git1-1.3.1.jolla_0.16+git1_1.2.7.jolla.i486.drpm ............................................[done]<br>
Retrieving package vim-common-7.3.629-1.2.3.jolla.i486                                          (23/275),   4.9 MiB ( 18.4 MiB unpacked)<br>
Retrieving delta: ./drpms/vim-common-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm, 238.1 KiB<br>
Retrieving: vim-common-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./vim-common-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm ...................................................[done]<br>
Retrieving package unzip-6.0-1.2.4.jolla.i486                                                   (24/275),  95.5 KiB (269.7 KiB unpacked)<br>
Retrieving delta: ./drpms/unzip-6.0-1.3.1.jolla_6.0_1.2.4.jolla.i486.drpm, 89.2 KiB<br>
Retrieving: unzip-6.0-1.3.1.jolla_6.0_1.2.4.jolla.i486.drpm ......................................................................[done]<br>
Applying delta: ./unzip-6.0-1.3.1.jolla_6.0_1.2.4.jolla.i486.drpm ................................................................[done]<br>
Retrieving package tar-1.17-1.2.4.jolla.i486                                                    (25/275), 321.4 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/tar-1.17-1.3.1.jolla_1.17_1.2.4.jolla.i486.drpm, 121.8 KiB<br>
Retrieving: tar-1.17-1.3.1.jolla_1.17_1.2.4.jolla.i486.drpm ......................................................................[done]<br>
Applying delta: ./tar-1.17-1.3.1.jolla_1.17_1.2.4.jolla.i486.drpm ................................................................[done]<br>
Retrieving package strace-4.22+git1-1.2.18.jolla.i486                                           (26/275), 198.4 KiB (558.7 KiB unpacked)<br>
Retrieving delta: ./drpms/strace-4.22+git1-1.3.2.jolla_4.22+git1_1.2.18.jolla.i486.drpm, 192.3 KiB<br>
Retrieving: strace-4.22+git1-1.3.2.jolla_4.22+git1_1.2.18.jolla.i486.drpm ..............................................[done (154 B/s)]<br>
Applying delta: ./strace-4.22+git1-1.3.2.jolla_4.22+git1_1.2.18.jolla.i486.drpm ..................................................[done]<br>
Retrieving package shadow-utils-4.6-1.2.4.jolla.i486                                            (27/275), 173.0 KiB (  1.0 MiB unpacked)<br>
Retrieving delta: ./drpms/shadow-utils-4.6-1.3.1.jolla_4.6_1.2.4.jolla.i486.drpm, 169.5 KiB<br>
Retrieving: shadow-utils-4.6-1.3.1.jolla_4.6_1.2.4.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./shadow-utils-4.6-1.3.1.jolla_4.6_1.2.4.jolla.i486.drpm .........................................................[done]<br>
Retrieving package sed-1:4.1.5-1.2.5.jolla.i486                                                 (28/275),  34.0 KiB ( 61.1 KiB unpacked)<br>
Retrieving delta: ./drpms/sed-4.1.5-1.3.1.jolla_4.1.5_1.2.5.jolla.i486.drpm, 23.9 KiB<br>
Retrieving: sed-4.1.5-1.3.1.jolla_4.1.5_1.2.5.jolla.i486.drpm ....................................................................[done]<br>
Applying delta: ./sed-4.1.5-1.3.1.jolla_4.1.5_1.2.5.jolla.i486.drpm ..............................................................[done]<br>
Retrieving package readline-5.2-1.2.6.jolla.i486                                                (29/275), 101.1 KiB (263.9 KiB unpacked)<br>
Retrieving delta: ./drpms/readline-5.2-1.3.1.jolla_5.2_1.2.6.jolla.i486.drpm, 87.8 KiB<br>
Retrieving: readline-5.2-1.3.1.jolla_5.2_1.2.6.jolla.i486.drpm .........................................................[done (154 B/s)]<br>
Applying delta: ./readline-5.2-1.3.1.jolla_5.2_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Retrieving package pth-2.0.7-1.1.8.jolla.i486                                                   (30/275),  48.9 KiB (101.3 KiB unpacked)<br>
Retrieving delta: ./drpms/pth-2.0.7-1.2.1.jolla_2.0.7_1.1.8.jolla.i486.drpm, 33.3 KiB<br>
Retrieving: pth-2.0.7-1.2.1.jolla_2.0.7_1.1.8.jolla.i486.drpm ....................................................................[done]<br>
Applying delta: ./pth-2.0.7-1.2.1.jolla_2.0.7_1.1.8.jolla.i486.drpm ..............................................................[done]<br>
Retrieving package psmisc-22.13-1.3.6.jolla.i486                                                (31/275),  43.7 KiB ( 91.6 KiB unpacked)<br>
Retrieving delta: ./drpms/psmisc-22.13-1.4.1.jolla_22.13_1.3.6.jolla.i486.drpm, 33.7 KiB<br>
Retrieving: psmisc-22.13-1.4.1.jolla_22.13_1.3.6.jolla.i486.drpm .......................................................[done (154 B/s)]<br>
Applying delta: ./psmisc-22.13-1.4.1.jolla_22.13_1.3.6.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package procps-3.2.8-1.2.6.jolla.i486                                                (32/275), 130.4 KiB (344.6 KiB unpacked)<br>
Retrieving delta: ./drpms/procps-3.2.8-1.3.1.jolla_3.2.8_1.2.6.jolla.i486.drpm, 119.3 KiB<br>
Retrieving: procps-3.2.8-1.3.1.jolla_3.2.8_1.2.6.jolla.i486.drpm .................................................................[done]<br>
Applying delta: ./procps-3.2.8-1.3.1.jolla_3.2.8_1.2.6.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package pptp-1.8.0+git4-1.1.14.jolla.i486                                            (33/275),  31.9 KiB ( 60.6 KiB unpacked)<br>
Retrieving delta: ./drpms/pptp-1.8.0+git4-1.2.2.jolla_1.8.0+git4_1.1.14.jolla.i486.drpm, 27.9 KiB<br>
Retrieving: pptp-1.8.0+git4-1.2.2.jolla_1.8.0+git4_1.1.14.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./pptp-1.8.0+git4-1.2.2.jolla_1.8.0+git4_1.1.14.jolla.i486.drpm ..................................................[done]<br>
Retrieving package popt-1.16-1.1.9.jolla.i486                                                   (34/275),  30.1 KiB ( 44.8 KiB unpacked)<br>
Retrieving delta: ./drpms/popt-1.16-1.2.1.jolla_1.16_1.1.9.jolla.i486.drpm, 23.8 KiB<br>
Retrieving: popt-1.16-1.2.1.jolla_1.16_1.1.9.jolla.i486.drpm .....................................................................[done]<br>
Applying delta: ./popt-1.16-1.2.1.jolla_1.16_1.1.9.jolla.i486.drpm ...............................................................[done]<br>
Retrieving package pkgconfig-0.27.1-1.2.6.jolla.i486                                            (35/275), 162.3 KiB (523.5 KiB unpacked)<br>
Retrieving delta: ./drpms/pkgconfig-0.27.1-1.3.1.jolla_0.27.1_1.2.6.jolla.i486.drpm, 150.2 KiB<br>
Retrieving: pkgconfig-0.27.1-1.3.1.jolla_0.27.1_1.2.6.jolla.i486.drpm ..................................................[done (154 B/s)]<br>
Applying delta: ./pkgconfig-0.27.1-1.3.1.jolla_0.27.1_1.2.6.jolla.i486.drpm ......................................................[done]<br>
Retrieving package patch-2.7.5+git1-1.1.8.jolla.i486                                            (36/275),  97.3 KiB (188.1 KiB unpacked)<br>
Retrieving delta: ./drpms/patch-2.7.5+git1-1.2.1.jolla_2.7.5+git1_1.1.8.jolla.i486.drpm, 75.4 KiB<br>
Retrieving: patch-2.7.5+git1-1.2.1.jolla_2.7.5+git1_1.1.8.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./patch-2.7.5+git1-1.2.1.jolla_2.7.5+git1_1.1.8.jolla.i486.drpm ..................................................[done]<br>
Retrieving package nspr-4.20.0-1.3.6.jolla.i486                                                 (37/275), 118.4 KiB (279.1 KiB unpacked)<br>
Retrieving delta: ./drpms/nspr-4.20.0-1.4.1.jolla_4.20.0_1.3.6.jolla.i486.drpm, 101.3 KiB<br>
Retrieving: nspr-4.20.0-1.4.1.jolla_4.20.0_1.3.6.jolla.i486.drpm .................................................................[done]<br>
Applying delta: ./nspr-4.20.0-1.4.1.jolla_4.20.0_1.3.6.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package net-tools-1.60-1.2.4.jolla.i486                                              (38/275),  92.5 KiB (302.0 KiB unpacked)<br>
Retrieving delta: ./drpms/net-tools-1.60-1.3.1.jolla_1.60_1.2.4.jolla.i486.drpm, 82.3 KiB<br>
Retrieving: net-tools-1.60-1.3.1.jolla_1.60_1.2.4.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./net-tools-1.60-1.3.1.jolla_1.60_1.2.4.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package ncurses-6.1+git1-1.3.4.jolla.i486                                            (39/275),  70.9 KiB (191.0 KiB unpacked)<br>
Retrieving delta: ./drpms/ncurses-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm, 67.3 KiB<br>
Retrieving: ncurses-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./ncurses-6.0-1.3.1.jolla_6.1+git1_1.3.4.jolla.i486.drpm .........................................................[done]<br>
Retrieving package lzo-2.09-1.1.9.jolla.i486                                                    (40/275),  42.4 KiB (188.4 KiB unpacked)<br>
Retrieving delta: ./drpms/lzo-2.09-1.2.1.jolla_2.09_1.1.9.jolla.i486.drpm, 20.0 KiB<br>
Retrieving: lzo-2.09-1.2.1.jolla_2.09_1.1.9.jolla.i486.drpm ......................................................................[done]<br>
Applying delta: ./lzo-2.09-1.2.1.jolla_2.09_1.1.9.jolla.i486.drpm ................................................................[done]<br>
Retrieving package lsof-4.91+git1-1.3.3.jolla.i486                                              (41/275),  71.2 KiB (150.9 KiB unpacked)<br>
Retrieving delta: ./drpms/lsof-4.86+git1-1.3.1.jolla_4.91+git1_1.3.3.jolla.i486.drpm, 67.3 KiB<br>
Retrieving: lsof-4.86+git1-1.3.1.jolla_4.91+git1_1.3.3.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./lsof-4.86+git1-1.3.1.jolla_4.91+git1_1.3.3.jolla.i486.drpm .....................................................[done]<br>
Retrieving package libtasn1-4.13+git1-1.3.6.jolla.i486                                          (42/275),  48.0 KiB ( 97.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libtasn1-4.13+git1-1.4.1.jolla_4.13+git1_1.3.6.jolla.i486.drpm, 33.1 KiB<br>
Retrieving: libtasn1-4.13+git1-1.4.1.jolla_4.13+git1_1.3.6.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libtasn1-4.13+git1-1.4.1.jolla_4.13+git1_1.3.6.jolla.i486.drpm .................................................[done]<br>
Retrieving package libstdc++-4.9.4-1.2.4.jolla.i486                                             (43/275), 265.4 KiB (992.4 KiB unpacked)<br>
Retrieving: libstdc++-4.9.4-1.2.4.jolla.i486.rpm .................................................................................[done]<br>
Retrieving package libshadowutils-0.0.2-1.3.6.jolla.i486                                        (44/275),  13.9 KiB ( 15.3 KiB unpacked)<br>
Retrieving delta: ./drpms/libshadowutils-0.0.2-1.4.2.jolla_0.0.2_1.3.6.jolla.i486.drpm, 7.4 KiB<br>
Retrieving: libshadowutils-0.0.2-1.4.2.jolla_0.0.2_1.3.6.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./libshadowutils-0.0.2-1.4.2.jolla_0.0.2_1.3.6.jolla.i486.drpm ...................................................[done]<br>
Retrieving package libsb2-2.3.90+git17-1.5.3.jolla.i486                                         (45/275), 116.2 KiB (375.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libsb2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm, 101.1 KiB<br>
Retrieving: libsb2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm .......................................[done (6.8 KiB/s)]<br>
Applying delta: ./libsb2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm .............................................[done]<br>
Retrieving package libpcap-1.8.1+git2-1.2.4.jolla.i486                                          (46/275),  99.1 KiB (245.6 KiB unpacked)<br>
Retrieving delta: ./drpms/libpcap-1.8.1+git2-1.2.1.jolla_1.8.1+git2_1.2.4.jolla.i486.drpm, 70.1 KiB<br>
Retrieving: libpcap-1.8.1+git2-1.2.1.jolla_1.8.1+git2_1.2.4.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./libpcap-1.8.1+git2-1.2.1.jolla_1.8.1+git2_1.2.4.jolla.i486.drpm ................................................[done]<br>
Retrieving package libnl-3.4.0-1.2.6.jolla.i486                                                 (47/275), 250.1 KiB (829.7 KiB unpacked)<br>
Retrieving delta: ./drpms/libnl-3.4.0-1.3.1.jolla_3.4.0_1.2.6.jolla.i486.drpm, 221.3 KiB<br>
Retrieving: libnl-3.4.0-1.3.1.jolla_3.4.0_1.2.6.jolla.i486.drpm ..................................................................[done]<br>
Applying delta: ./libnl-3.4.0-1.3.1.jolla_3.4.0_1.2.6.jolla.i486.drpm ............................................................[done]<br>
Retrieving package liblua-5.1.5-1.1.11.jolla.i486                                               (48/275),  81.0 KiB (172.7 KiB unpacked)<br>
Retrieving delta: ./drpms/liblua-5.1.5-1.2.1.jolla_5.1.5_1.1.11.jolla.i486.drpm, 72.1 KiB<br>
Retrieving: liblua-5.1.5-1.2.1.jolla_5.1.5_1.1.11.jolla.i486.drpm ...................................................[done (11.1 KiB/s)]<br>
Applying delta: ./liblua-5.1.5-1.2.1.jolla_5.1.5_1.1.11.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package libiphb-1.2.5+git1-1.3.6.jolla.i486                                          (49/275),  21.4 KiB ( 32.4 KiB unpacked)<br>
Retrieving delta: ./drpms/libiphb-1.2.5+git1-1.4.1.jolla_1.2.5+git1_1.3.6.jolla.i486.drpm, 8.5 KiB<br>
Retrieving: libiphb-1.2.5+git1-1.4.1.jolla_1.2.5+git1_1.3.6.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./libiphb-1.2.5+git1-1.4.1.jolla_1.2.5+git1_1.3.6.jolla.i486.drpm ................................................[done]<br>
Retrieving package libgpg-error-1.27+git2-1.3.4.jolla.i486                                      (50/275), 129.9 KiB (577.6 KiB unpacked)<br>
Retrieving delta: ./drpms/libgpg-error-1.27+git2-1.4.1.jolla_1.27+git2_1.3.4.jolla.i486.drpm, 47.2 KiB<br>
Retrieving: libgpg-error-1.27+git2-1.4.1.jolla_1.27+git2_1.3.4.jolla.i486.drpm .......................................[done (1.2 KiB/s)]<br>
Applying delta: ./libgpg-error-1.27+git2-1.4.1.jolla_1.27+git2_1.3.4.jolla.i486.drpm .............................................[done]<br>
Retrieving package libffi-3.2.1+git1-1.2.9.jolla.i486                                           (51/275),  23.6 KiB ( 27.1 KiB unpacked)<br>
Retrieving delta: ./drpms/libffi-3.2.1+git1-1.3.1.jolla_3.2.1+git1_1.2.9.jolla.i486.drpm, 18.1 KiB<br>
Retrieving: libffi-3.2.1+git1-1.3.1.jolla_3.2.1+git1_1.2.9.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libffi-3.2.1+git1-1.3.1.jolla_3.2.1+git1_1.2.9.jolla.i486.drpm .................................................[done]<br>
Retrieving package libcom_err-1.45.0+git1-1.3.4.jolla.i486                                      (52/275),  14.4 KiB ( 10.6 KiB unpacked)<br>
Retrieving delta: ./drpms/libcom_err-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm, 10.0 KiB<br>
Retrieving: libcom_err-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm .....................................[done (9.6 KiB/s)]<br>
Applying delta: ./libcom_err-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm ...........................................[done]<br>
Retrieving package libcap-2.24+git1-1.3.6.jolla.i486                                            (53/275),  35.4 KiB ( 72.7 KiB unpacked)<br>
Retrieving delta: ./drpms/libcap-2.24+git1-1.4.1.jolla_2.24+git1_1.3.6.jolla.i486.drpm, 24.9 KiB<br>
Retrieving: libcap-2.24+git1-1.4.1.jolla_2.24+git1_1.3.6.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./libcap-2.24+git1-1.4.1.jolla_2.24+git1_1.3.6.jolla.i486.drpm ...................................................[done]<br>
Retrieving package libattr-2.4.47+git1-1.3.6.jolla.i486                                         (54/275),  23.8 KiB ( 42.0 KiB unpacked)<br>
Retrieving delta: ./drpms/libattr-2.4.47+git1-1.4.1.jolla_2.4.47+git1_1.3.6.jolla.i486.drpm, 11.1 KiB<br>
Retrieving: libattr-2.4.47+git1-1.4.1.jolla_2.4.47+git1_1.3.6.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./libattr-2.4.47+git1-1.4.1.jolla_2.4.47+git1_1.3.6.jolla.i486.drpm ..............................................[done]<br>
Retrieving package less-436+mer1-1.1.24.jolla.i486                                              (55/275),  98.7 KiB (183.8 KiB unpacked)<br>
Retrieving delta: ./drpms/less-436+mer1-1.2.2.jolla_436+mer1_1.1.24.jolla.i486.drpm, 69.7 KiB<br>
Retrieving: less-436+mer1-1.2.2.jolla_436+mer1_1.1.24.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./less-436+mer1-1.2.2.jolla_436+mer1_1.1.24.jolla.i486.drpm ......................................................[done]<br>
Retrieving package json-c-0.12-1.1.8.jolla.i486                                                 (56/275),  25.0 KiB ( 42.3 KiB unpacked)<br>
Retrieving: json-c-0.12-1.1.8.jolla.i486.rpm .....................................................................................[done]<br>
Retrieving package iptables-1.8.2+git1-1.4.2.jolla.i486                                         (57/275), 230.3 KiB (874.2 KiB unpacked)<br>
Retrieving delta: ./drpms/iptables-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm, 218.2 KiB<br>
Retrieving: iptables-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm ...........................................[done (154 B/s)]<br>
Applying delta: ./iptables-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm ...............................................[done]<br>
Retrieving package iproute-3.7.0+git4-1.2.4.jolla.i486                                          (58/275), 260.1 KiB (770.6 KiB unpacked)<br>
Retrieving delta: ./drpms/iproute-3.7.0+git4-1.3.1.jolla_3.7.0+git4_1.2.4.jolla.i486.drpm, 230.4 KiB<br>
Retrieving: iproute-3.7.0+git4-1.3.1.jolla_3.7.0+git4_1.2.4.jolla.i486.drpm .........................................[done (15.5 KiB/s)]<br>
Applying delta: ./iproute-3.7.0+git4-1.3.1.jolla_3.7.0+git4_1.2.4.jolla.i486.drpm ................................................[done]<br>
Retrieving package gdbm-1.8.3-1.2.6.jolla.i486                                                  (59/275),  24.3 KiB ( 37.7 KiB unpacked)<br>
Retrieving delta: ./drpms/gdbm-1.8.3-1.3.1.jolla_1.8.3_1.2.6.jolla.i486.drpm, 13.2 KiB<br>
Retrieving: gdbm-1.8.3-1.3.1.jolla_1.8.3_1.2.6.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./gdbm-1.8.3-1.3.1.jolla_1.8.3_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Retrieving package freetype-2.8.0-1.1.8.jolla.i486                                              (60/275), 314.8 KiB (693.2 KiB unpacked)<br>
Retrieving delta: ./drpms/freetype-2.8.0-1.2.1.jolla_2.8.0_1.1.8.jolla.i486.drpm, 266.6 KiB<br>
Retrieving: freetype-2.8.0-1.2.1.jolla_2.8.0_1.1.8.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./freetype-2.8.0-1.2.1.jolla_2.8.0_1.1.8.jolla.i486.drpm .........................................................[done]<br>
Retrieving package findutils-4.2.31-1.2.4.jolla.i486                                            (61/275), 129.9 KiB (515.0 KiB unpacked)<br>
Retrieving delta: ./drpms/findutils-4.2.31-1.3.1.jolla_4.2.31_1.2.4.jolla.i486.drpm, 53.9 KiB<br>
Retrieving: findutils-4.2.31-1.3.1.jolla_4.2.31_1.2.4.jolla.i486.drpm ...............................................[done (11.3 KiB/s)]<br>
Applying delta: ./findutils-4.2.31-1.3.1.jolla_4.2.31_1.2.4.jolla.i486.drpm ......................................................[done]<br>
Retrieving package expat-2.1.0-1.1.9.jolla.i486                                                 (62/275),  68.3 KiB (173.4 KiB unpacked)<br>
Retrieving delta: ./drpms/expat-2.1.0-1.2.1.jolla_2.1.0_1.1.9.jolla.i486.drpm, 63.1 KiB<br>
Retrieving: expat-2.1.0-1.2.1.jolla_2.1.0_1.1.9.jolla.i486.drpm ..................................................................[done]<br>
Applying delta: ./expat-2.1.0-1.2.1.jolla_2.1.0_1.1.9.jolla.i486.drpm ............................................................[done]<br>
Retrieving package dosfstools-3.0.10+git1-1.1.25.jolla.i486                                     (63/275),  66.6 KiB (168.0 KiB unpacked)<br>
Retrieving delta: ./drpms/dosfstools-3.0.10+git1-1.2.2.jolla_3.0.10+git1_1.1.25.jolla.i486.drpm, 42.2 KiB<br>
Retrieving: dosfstools-3.0.10+git1-1.2.2.jolla_3.0.10+git1_1.1.25.jolla.i486.drpm ......................................[done (154 B/s)]<br>
Applying delta: ./dosfstools-3.0.10+git1-1.2.2.jolla_3.0.10+git1_1.1.25.jolla.i486.drpm ..........................................[done]<br>
Retrieving package diffutils-2.8.1-1.2.5.jolla.i486                                             (64/275), 121.2 KiB (471.9 KiB unpacked)<br>
Retrieving delta: ./drpms/diffutils-2.8.1-1.3.1.jolla_2.8.1_1.2.5.jolla.i486.drpm, 63.8 KiB<br>
Retrieving: diffutils-2.8.1-1.3.1.jolla_2.8.1_1.2.5.jolla.i486.drpm ..............................................................[done]<br>
Applying delta: ./diffutils-2.8.1-1.3.1.jolla_2.8.1_1.2.5.jolla.i486.drpm ........................................................[done]<br>
Retrieving package db4-4.8.30-1.3.8.jolla.i486                                                  (65/275), 549.6 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/db4-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm, 507.5 KiB<br>
Retrieving: db4-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm ..................................................................[done]<br>
Applying delta: ./db4-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm ............................................................[done]<br>
Retrieving package cpio-2.12+git1-1.3.3.jolla.i486                                              (66/275),  75.5 KiB (162.5 KiB unpacked)<br>
Retrieving delta: ./drpms/cpio-2.11-1.3.1.jolla_2.12+git1_1.3.3.jolla.i486.drpm, 60.9 KiB<br>
Retrieving: cpio-2.11-1.3.1.jolla_2.12+git1_1.3.3.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./cpio-2.11-1.3.1.jolla_2.12+git1_1.3.3.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package bzip2-libs-1.0.6-1.2.6.jolla.i486                                            (67/275),  38.5 KiB ( 73.0 KiB unpacked)<br>
Retrieving delta: ./drpms/bzip2-libs-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm, 30.5 KiB<br>
Retrieving: bzip2-libs-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Applying delta: ./bzip2-libs-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm .......................................................[done]<br>
Retrieving package busybox-1.29.3+git5-1.1.4.jolla.i486                                         (68/275),  80.6 KiB (144.5 KiB unpacked)<br>
Retrieving delta: ./drpms/busybox-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm, 70.3 KiB<br>
Retrieving: busybox-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./busybox-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm ..............................................[done]<br>
Retrieving package bluez-libs-4.101+git76-1.2.12.jolla.i486                                     (69/275),  64.2 KiB (127.4 KiB unpacked)<br>
Retrieving delta: ./drpms/bluez-libs-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm, 43.6 KiB<br>
Retrieving: bluez-libs-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./bluez-libs-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm ..........................................[done]<br>
Retrieving package libxml2-2.9.8+git2-1.3.6.jolla.i486                                          (70/275), 522.9 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/libxml2-2.9.8+git2-1.4.1.jolla_2.9.8+git2_1.3.6.jolla.i486.drpm, 488.0 KiB<br>
Retrieving: libxml2-2.9.8+git2-1.4.1.jolla_2.9.8+git2_1.3.6.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./libxml2-2.9.8+git2-1.4.1.jolla_2.9.8+git2_1.3.6.jolla.i486.drpm ................................................[done]<br>
Retrieving package info-4.13a-1.2.6.jolla.i486                                                  (71/275), 109.0 KiB (255.2 KiB unpacked)<br>
Retrieving delta: ./drpms/info-4.13a-1.3.1.jolla_4.13a_1.2.6.jolla.i486.drpm, 94.3 KiB<br>
Retrieving: info-4.13a-1.3.1.jolla_4.13a_1.2.6.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./info-4.13a-1.3.1.jolla_4.13a_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Retrieving package file-libs-5.35+git2-1.2.6.jolla.i486                                         (72/275), 473.1 KiB (  6.2 MiB unpacked)<br>
Retrieving delta: ./drpms/file-libs-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm, 62.0 KiB<br>
Retrieving: file-libs-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm ............................................[done (154 B/s)]<br>
Applying delta: ./file-libs-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm ................................................[done]<br>
Retrieving package elfutils-libelf-0.170+git1-1.3.6.jolla.i486                                  (73/275),  58.9 KiB ( 97.7 KiB unpacked)<br>
Retrieving delta: ./drpms/elfutils-libelf-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm, 51.1 KiB<br>
Retrieving: elfutils-libelf-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./elfutils-libelf-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm ........................................[done]<br>
Retrieving package binutils-2.25-1.3.6.jolla.i486                                               (74/275),   3.6 MiB ( 20.3 MiB unpacked)<br>
Retrieving delta: ./drpms/binutils-2.25-1.4.1.jolla_2.25_1.3.6.jolla.i486.drpm, 2.4 MiB<br>
Retrieving: binutils-2.25-1.4.1.jolla_2.25_1.3.6.jolla.i486.drpm ...................................................[done (715.3 KiB/s)]<br>
Applying delta: ./binutils-2.25-1.4.1.jolla_2.25_1.3.6.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package xz-5.0.4-1.2.6.jolla.i486                                                    (75/275),  59.6 KiB (160.6 KiB unpacked)<br>
Retrieving delta: ./drpms/xz-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm, 33.4 KiB<br>
Retrieving: xz-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm .....................................................................[done]<br>
Applying delta: ./xz-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm ...............................................................[done]<br>
Retrieving package libutempter-1.1.5+git1-1.1.9.jolla.i486                                      (76/275),  21.9 KiB ( 36.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libutempter-1.1.5+git1-1.2.1.jolla_1.1.5+git1_1.1.9.jolla.i486.drpm, 9.2 KiB<br>
Retrieving: libutempter-1.1.5+git1-1.2.1.jolla_1.1.5+git1_1.1.9.jolla.i486.drpm ..................................................[done]<br>
Applying delta: ./libutempter-1.1.5+git1-1.2.1.jolla_1.1.5+git1_1.1.9.jolla.i486.drpm ............................................[done]<br>
Retrieving package pcre-8.42+git1-1.3.4.jolla.i486                                              (77/275), 284.8 KiB (967.4 KiB unpacked)<br>
Retrieving delta: ./drpms/pcre-8.31-1.3.1.jolla_8.42+git1_1.3.4.jolla.i486.drpm, 287.0 KiB<br>
Retrieving: pcre-8.31-1.3.1.jolla_8.42+git1_1.3.4.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./pcre-8.31-1.3.1.jolla_8.42+git1_1.3.4.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package p7zip-full-16.02+git1-1.2.1.jolla.i486                                       (78/275),   1.1 MiB (  4.2 MiB unpacked)<br>
Retrieving: p7zip-full-16.02+git1-1.2.1.jolla.i486.rpm ...........................................................................[done]<br>
Retrieving package libusb-0.1.12-1.2.4.jolla.i486                                               (79/275),  33.8 KiB ( 70.4 KiB unpacked)<br>
Retrieving delta: ./drpms/libusb-0.1.12-1.3.1.jolla_0.1.12_1.2.4.jolla.i486.drpm, 19.4 KiB<br>
Retrieving: libusb-0.1.12-1.3.1.jolla_0.1.12_1.2.4.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./libusb-0.1.12-1.3.1.jolla_0.1.12_1.2.4.jolla.i486.drpm .........................................................[done]<br>
Retrieving package boost-system-1.66.0-1.3.8.jolla.i486                                         (80/275),  19.3 KiB ( 29.4 KiB unpacked)<br>
Retrieving delta: ./drpms/boost-system-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm, 13.2 KiB<br>
Retrieving: boost-system-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./boost-system-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm ...................................................[done]<br>
Retrieving package ppp-2.4.7+git3-1.1.15.jolla.i486                                             (81/275), 169.8 KiB (404.6 KiB unpacked)<br>
Retrieving delta: ./drpms/ppp-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm, 165.8 KiB<br>
Retrieving: ppp-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./ppp-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm ...................................................[done]<br>
Retrieving package iw-4.1+git2-1.2.4.jolla.i486                                                 (82/275),  63.4 KiB (145.1 KiB unpacked)<br>
Retrieving delta: ./drpms/iw-4.1+git2-1.3.1.jolla_4.1+git2_1.2.4.jolla.i486.drpm, 59.0 KiB<br>
Retrieving: iw-4.1+git2-1.3.1.jolla_4.1+git2_1.2.4.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./iw-4.1+git2-1.3.1.jolla_4.1+git2_1.2.4.jolla.i486.drpm .........................................................[done]<br>
Retrieving package libksba-1.3.5+git2-1.2.4.jolla.i486                                          (83/275),  94.2 KiB (230.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libksba-1.3.5+git2-1.3.1.jolla_1.3.5+git2_1.2.4.jolla.i486.drpm, 74.4 KiB<br>
Retrieving: libksba-1.3.5+git2-1.3.1.jolla_1.3.5+git2_1.2.4.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./libksba-1.3.5+git2-1.3.1.jolla_1.3.5+git2_1.2.4.jolla.i486.drpm ................................................[done]<br>
Retrieving package libgcrypt-1.5.6+git1-1.1.11.jolla.i486                                       (84/275), 221.0 KiB (520.3 KiB unpacked)<br>
Retrieving delta: ./drpms/libgcrypt-1.5.6+git1-1.2.1.jolla_1.5.6+git1_1.1.11.jolla.i486.drpm, 156.4 KiB<br>
Retrieving: libgcrypt-1.5.6+git1-1.2.1.jolla_1.5.6+git1_1.1.11.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./libgcrypt-1.5.6+git1-1.2.1.jolla_1.5.6+git1_1.1.11.jolla.i486.drpm .............................................[done]<br>
Retrieving package p11-kit-0.23.12+git1-1.3.4.jolla.i486                                        (85/275), 177.7 KiB (  1.2 MiB unpacked)<br>
Retrieving delta: ./drpms/p11-kit-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm, 156.9 KiB<br>
Retrieving: p11-kit-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm ...........................................[done (1.2 KiB/s)]<br>
Applying delta: ./p11-kit-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm .................................................[done]<br>
Retrieving package libss-1.45.0+git1-1.3.4.jolla.i486                                           (86/275),  29.1 KiB ( 63.6 KiB unpacked)<br>
Retrieving delta: ./drpms/libss-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm, 13.6 KiB<br>
Retrieving: libss-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./libss-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm ................................................[done]<br>
Retrieving package e2fsprogs-libs-1.45.0+git1-1.3.4.jolla.i486                                  (87/275), 207.6 KiB (520.7 KiB unpacked)<br>
Retrieving delta: ./drpms/e2fsprogs-libs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm, 172.8 KiB<br>
Retrieving: e2fsprogs-libs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm .............................................[done]<br>
Applying delta: ./e2fsprogs-libs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm .......................................[done]<br>
Retrieving package libacl-2.2.53-1.2.6.jolla.i486                                               (88/275),  21.5 KiB ( 28.9 KiB unpacked)<br>
Retrieving delta: ./drpms/libacl-2.2.53-1.3.1.jolla_2.2.53_1.2.6.jolla.i486.drpm, 16.4 KiB<br>
Retrieving: libacl-2.2.53-1.3.1.jolla_2.2.53_1.2.6.jolla.i486.drpm ...............................................................[done]<br>
Applying delta: ./libacl-2.2.53-1.3.1.jolla_2.2.53_1.2.6.jolla.i486.drpm .........................................................[done]<br>
Retrieving package iptables-ipv6-1.8.2+git1-1.4.2.jolla.i486                                    (89/275),  45.7 KiB (139.2 KiB unpacked)<br>
Retrieving delta: ./drpms/iptables-ipv6-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm, 44.2 KiB<br>
Retrieving: iptables-ipv6-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./iptables-ipv6-1.6.1+git3-1.4.1.jolla_1.8.2+git1_1.4.2.jolla.i486.drpm ..........................................[done]<br>
Retrieving package db4-utils-4.8.30-1.3.8.jolla.i486                                            (90/275), 103.1 KiB (280.3 KiB unpacked)<br>
Retrieving delta: ./drpms/db4-utils-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm, 99.2 KiB<br>
Retrieving: db4-utils-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./db4-utils-4.8.30-1.4.1.jolla_4.8.30_1.3.8.jolla.i486.drpm ......................................................[done]<br>
Retrieving package bzip2-1.0.6-1.2.6.jolla.i486                                                 (91/275),  32.1 KiB ( 43.9 KiB unpacked)<br>
Retrieving delta: ./drpms/bzip2-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm, 25.5 KiB<br>
Retrieving: bzip2-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm ..................................................................[done]<br>
Applying delta: ./bzip2-1.0.6-1.3.1.jolla_1.0.6_1.2.6.jolla.i486.drpm ............................................................[done]<br>
Retrieving package busybox-symlinks-gzip-1.29.3+git5-1.1.4.jolla.i486                           (92/275),   9.5 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/busybox-symlinks-gzip-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm, 5.6 KiB<br>
Retrieving: busybox-symlinks-gzip-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm ......................................[done]<br>
Applying delta: ./busybox-symlinks-gzip-1.29.3+git5-1.2.1.jolla_1.29.3+git5_1.1.4.jolla.i486.drpm ................................[done]<br>
Retrieving package libxslt-1.1.29-1.2.6.jolla.i486                                              (93/275), 131.7 KiB (331.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libxslt-1.1.29-1.3.1.jolla_1.1.29_1.2.6.jolla.i486.drpm, 116.4 KiB<br>
Retrieving: libxslt-1.1.29-1.3.1.jolla_1.1.29_1.2.6.jolla.i486.drpm ..............................................................[done]<br>
Applying delta: ./libxslt-1.1.29-1.3.1.jolla_1.1.29_1.2.6.jolla.i486.drpm ........................................................[done]<br>
Retrieving package augeas-libs-1.6.0+git1-1.2.5.jolla.i486                                      (94/275), 472.9 KiB (  1.8 MiB unpacked)<br>
Retrieving delta: ./drpms/augeas-libs-1.6.0+git1-1.3.1.jolla_1.6.0+git1_1.2.5.jolla.i486.drpm, 193.2 KiB<br>
Retrieving: augeas-libs-1.6.0+git1-1.3.1.jolla_1.6.0+git1_1.2.5.jolla.i486.drpm ..................................................[done]<br>
Applying delta: ./augeas-libs-1.6.0+git1-1.3.1.jolla_1.6.0+git1_1.2.5.jolla.i486.drpm ............................................[done]<br>
Retrieving package mtools-4.0.12+mer1-1.1.24.jolla.i486                                         (95/275), 184.8 KiB (300.0 KiB unpacked)<br>
Retrieving delta: ./drpms/mtools-4.0.12+mer1-1.2.2.jolla_4.0.12+mer1_1.1.24.jolla.i486.drpm, 86.2 KiB<br>
Retrieving: mtools-4.0.12+mer1-1.2.2.jolla_4.0.12+mer1_1.1.24.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./mtools-4.0.12+mer1-1.2.2.jolla_4.0.12+mer1_1.1.24.jolla.i486.drpm ..............................................[done]<br>
Retrieving package file-5.35+git2-1.2.6.jolla.i486                                              (96/275),  15.5 KiB ( 17.3 KiB unpacked)<br>
Retrieving delta: ./drpms/file-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm, 10.8 KiB<br>
Retrieving: file-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./file-5.35+git2-1.3.1.jolla_5.35+git2_1.2.6.jolla.i486.drpm .....................................................[done]<br>
Retrieving package xz-lzma-compat-5.0.4-1.2.6.jolla.i486                                        (97/275),  14.1 KiB ( 14.3 KiB unpacked)<br>
Retrieving delta: ./drpms/xz-lzma-compat-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm, 10.2 KiB<br>
Retrieving: xz-lzma-compat-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./xz-lzma-compat-5.0.4-1.3.1.jolla_5.0.4_1.2.6.jolla.i486.drpm ...................................................[done]<br>
Retrieving package squashfs-tools-4.3.0-1.1.9.jolla.i486                                        (98/275), 108.8 KiB (271.5 KiB unpacked)<br>
Retrieving delta: ./drpms/squashfs-tools-4.3.0-1.2.1.jolla_4.3.0_1.1.9.jolla.i486.drpm, 98.5 KiB<br>
Retrieving: squashfs-tools-4.3.0-1.2.1.jolla_4.3.0_1.1.9.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./squashfs-tools-4.3.0-1.2.1.jolla_4.3.0_1.1.9.jolla.i486.drpm ...................................................[done]<br>
Retrieving package kmod-libs-21-1.2.7.jolla.i486                                                (99/275),  45.9 KiB ( 91.0 KiB unpacked)<br>
Retrieving delta: ./drpms/kmod-libs-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm, 39.9 KiB<br>
Retrieving: kmod-libs-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm ....................................................................[done]<br>
Applying delta: ./kmod-libs-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm ..............................................................[done]<br>
Retrieving package kmod-21-1.2.7.jolla.i486                                                    (100/275),  69.3 KiB (143.0 KiB unpacked)<br>
Retrieving delta: ./drpms/kmod-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm, 64.2 KiB<br>
Retrieving: kmod-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm .........................................................................[done]<br>
Applying delta: ./kmod-21-1.3.1.jolla_21_1.2.7.jolla.i486.drpm ...................................................................[done]<br>
Retrieving package elfutils-libs-0.170+git1-1.3.6.jolla.i486                                   (101/275), 238.6 KiB (694.0 KiB unpacked)<br>
Retrieving delta: ./drpms/elfutils-libs-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm, 207.0 KiB<br>
Retrieving: elfutils-libs-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./elfutils-libs-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm ..........................................[done]<br>
Retrieving package grep-1:2.5.1a-1.2.4.jolla.i486                                              (102/275),  64.4 KiB (103.6 KiB unpacked)<br>
Retrieving delta: ./drpms/grep-2.5.1a-1.3.1.jolla_2.5.1a_1.2.4.jolla.i486.drpm, 54.3 KiB<br>
Retrieving: grep-2.5.1a-1.3.1.jolla_2.5.1a_1.2.4.jolla.i486.drpm .................................................................[done]<br>
Applying delta: ./grep-2.5.1a-1.3.1.jolla_2.5.1a_1.2.4.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package glib2-2.56.1+git3-1.3.6.jolla.i486                                          (103/275),   1.2 MiB (  3.7 MiB unpacked)<br>
Retrieving delta: ./drpms/glib2-2.56.1+git3-1.4.1.jolla_2.56.1+git3_1.3.6.jolla.i486.drpm, 1.0 MiB<br>
Retrieving: glib2-2.56.1+git3-1.4.1.jolla_2.56.1+git3_1.3.6.jolla.i486.drpm .........................................[done (11.1 KiB/s)]<br>
Applying delta: ./glib2-2.56.1+git3-1.4.1.jolla_2.56.1+git3_1.3.6.jolla.i486.drpm ................................................[done]<br>
Retrieving package boost-thread-1.66.0-1.3.8.jolla.i486                                        (104/275),  80.9 KiB (330.1 KiB unpacked)<br>
Retrieving delta: ./drpms/boost-thread-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm, 56.5 KiB<br>
Retrieving: boost-thread-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./boost-thread-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm ...................................................[done]<br>
Retrieving package boost-filesystem-1.66.0-1.3.8.jolla.i486                                    (105/275),  55.7 KiB (266.0 KiB unpacked)<br>
Retrieving delta: ./drpms/boost-filesystem-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm, 42.6 KiB<br>
Retrieving: boost-filesystem-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./boost-filesystem-1.66.0-1.4.1.jolla_1.66.0_1.3.8.jolla.i486.drpm ...............................................[done]<br>
Retrieving package ppp-libs-2.4.7+git3-1.1.15.jolla.i486                                       (106/275),  60.0 KiB (155.5 KiB unpacked)<br>
Retrieving delta: ./drpms/ppp-libs-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm, 51.5 KiB<br>
Retrieving: ppp-libs-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./ppp-libs-2.4.7+git3-1.2.2.jolla_2.4.7+git3_1.1.15.jolla.i486.drpm ..............................................[done]<br>
Retrieving package vpnc-0.5.3-1.2.4.jolla.i486                                                 (107/275),  68.9 KiB (156.0 KiB unpacked)<br>
Retrieving delta: ./drpms/vpnc-0.5.3-1.3.1.jolla_0.5.3_1.2.4.jolla.i486.drpm, 58.4 KiB<br>
Retrieving: vpnc-0.5.3-1.3.1.jolla_0.5.3_1.2.4.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./vpnc-0.5.3-1.3.1.jolla_0.5.3_1.2.4.jolla.i486.drpm .............................................................[done]<br>
Retrieving package rsync-3.1.0+git2-1.3.5.jolla.i486                                           (108/275), 210.9 KiB (460.5 KiB unpacked)<br>
Retrieving delta: ./drpms/rsync-3.1.0+git2-1.4.2.jolla_3.1.0+git2_1.3.5.jolla.i486.drpm, 196.3 KiB<br>
Retrieving: rsync-3.1.0+git2-1.4.2.jolla_3.1.0+git2_1.3.5.jolla.i486.drpm ...........................................[done (15.5 KiB/s)]<br>
Applying delta: ./rsync-3.1.0+git2-1.4.2.jolla_3.1.0+git2_1.3.5.jolla.i486.drpm ..................................................[done]<br>
Retrieving package coreutils-1:6.9-1.2.5.jolla.i486                                            (109/275), 549.2 KiB (  2.8 MiB unpacked)<br>
Retrieving delta: ./drpms/coreutils-6.9-1.3.1.jolla_6.9_1.2.5.jolla.i486.drpm, 540.3 KiB<br>
Retrieving: coreutils-6.9-1.3.1.jolla_6.9_1.2.5.jolla.i486.drpm ..................................................................[done]<br>
Applying delta: ./coreutils-6.9-1.3.1.jolla_6.9_1.2.5.jolla.i486.drpm ............................................................[done]<br>
Retrieving package hwdata-0.291+git1-1.1.8.jolla.noarch                                        (110/275),   1.2 MiB (  6.5 MiB unpacked)<br>
Retrieving delta: ./drpms/hwdata-0.291+git1-1.2.1.jolla_0.291+git1_1.1.8.jolla.noarch.drpm, 5.0 KiB<br>
Retrieving: hwdata-0.291+git1-1.2.1.jolla_0.291+git1_1.1.8.jolla.noarch.drpm .....................................................[done]<br>
Applying delta: ./hwdata-0.291+git1-1.2.1.jolla_0.291+git1_1.1.8.jolla.noarch.drpm ...............................................[done]<br>
Retrieving package elfutils-0.170+git1-1.3.6.jolla.i486                                        (111/275), 241.5 KiB (661.3 KiB unpacked)<br>
Retrieving delta: ./drpms/elfutils-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm, 221.9 KiB<br>
Retrieving: elfutils-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./elfutils-0.170+git1-1.4.1.jolla_0.170+git1_1.3.6.jolla.i486.drpm ...............................................[done]<br>
Retrieving package shared-mime-info-1.12-1.4.4.jolla.i486                                      (112/275), 321.2 KiB (  4.8 MiB unpacked)<br>
Retrieving delta: ./drpms/shared-mime-info-1.9+git1-1.4.1.jolla_1.12_1.4.4.jolla.i486.drpm, 107.2 KiB<br>
Retrieving: shared-mime-info-1.9+git1-1.4.1.jolla_1.12_1.4.4.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./shared-mime-info-1.9+git1-1.4.1.jolla_1.12_1.4.4.jolla.i486.drpm ...............................................[done]<br>
Retrieving package libwspcodec-2.2.1-1.2.8.jolla.i486                                          (113/275),  14.3 KiB ( 16.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libwspcodec-2.2.1-1.3.1.jolla_2.2.1_1.2.8.jolla.i486.drpm, 9.1 KiB<br>
Retrieving: libwspcodec-2.2.1-1.3.1.jolla_2.2.1_1.2.8.jolla.i486.drpm ................................................[done (9.1 KiB/s)]<br>
Applying delta: ./libwspcodec-2.2.1-1.3.1.jolla_2.2.1_1.2.8.jolla.i486.drpm ......................................................[done]<br>
Retrieving package libglibutil-1.0.35-1.8.4.jolla.i486                                         (114/275),  30.0 KiB ( 47.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libglibutil-1.0.35-1.9.2.jolla_1.0.35_1.8.4.jolla.i486.drpm, 25.4 KiB<br>
Retrieving: libglibutil-1.0.35-1.9.2.jolla_1.0.35_1.8.4.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./libglibutil-1.0.35-1.9.2.jolla_1.0.35_1.8.4.jolla.i486.drpm ....................................................[done]<br>
Retrieving package desktop-file-utils-0.23+git1-1.2.10.jolla.i486                              (115/275),  44.3 KiB (143.6 KiB unpacked)<br>
Retrieving delta: ./drpms/desktop-file-utils-0.23+git1-1.3.1.jolla_0.23+git1_1.2.10.jolla.i486.drpm, 34.3 KiB<br>
Retrieving: desktop-file-utils-0.23+git1-1.3.1.jolla_0.23+git1_1.2.10.jolla.i486.drpm ............................................[done]<br>
Applying delta: ./desktop-file-utils-0.23+git1-1.3.1.jolla_0.23+git1_1.2.10.jolla.i486.drpm ......................................[done]<br>
Retrieving package xl2tpd-1.3.8+git3-1.1.17.jolla.i486                                         (116/275),  49.2 KiB (106.7 KiB unpacked)<br>
Retrieving delta: ./drpms/xl2tpd-1.3.8+git3-1.2.2.jolla_1.3.8+git3_1.1.17.jolla.i486.drpm, 45.2 KiB<br>
Retrieving: xl2tpd-1.3.8+git3-1.2.2.jolla_1.3.8+git3_1.1.17.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./xl2tpd-1.3.8+git3-1.2.2.jolla_1.3.8+git3_1.1.17.jolla.i486.drpm ................................................[done]<br>
Retrieving package pam-1.1.8+git5-1.2.4.jolla.i486                                             (117/275), 314.9 KiB (  1.3 MiB unpacked)<br>
Retrieving delta: ./drpms/pam-1.1.8+git5-1.3.1.jolla_1.1.8+git5_1.2.4.jolla.i486.drpm, 207.2 KiB<br>
Retrieving: pam-1.1.8+git5-1.3.1.jolla_1.1.8+git5_1.2.4.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./pam-1.1.8+git5-1.3.1.jolla_1.1.8+git5_1.2.4.jolla.i486.drpm ....................................................[done]<br>
Retrieving package libmce-glib-1.0.5-1.1.15.jolla.i486                                         (118/275),  20.0 KiB ( 39.3 KiB unpacked)<br>
Retrieving delta: ./drpms/libmce-glib-1.0.5-1.2.2.jolla_1.0.5_1.1.15.jolla.i486.drpm, 14.0 KiB<br>
Retrieving: libmce-glib-1.0.5-1.2.2.jolla_1.0.5_1.1.15.jolla.i486.drpm ..............................................[done (14.0 KiB/s)]<br>
Applying delta: ./libmce-glib-1.0.5-1.2.2.jolla_1.0.5_1.1.15.jolla.i486.drpm .....................................................[done]<br>
Retrieving package libgsupplicant-1.0.11-1.4.7.jolla.i486                                      (119/275),  65.0 KiB (230.9 KiB unpacked)<br>
Retrieving delta: ./drpms/libgsupplicant-1.0.11-1.5.2.jolla_1.0.11_1.4.7.jolla.i486.drpm, 55.0 KiB<br>
Retrieving: libgsupplicant-1.0.11-1.5.2.jolla_1.0.11_1.4.7.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libgsupplicant-1.0.11-1.5.2.jolla_1.0.11_1.4.7.jolla.i486.drpm .................................................[done]<br>
Retrieving package libgrilio-1.0.29-1.9.1.jolla.i486                                           (120/275),  32.1 KiB ( 53.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libgrilio-1.0.26-1.7.2.jolla_1.0.29_1.9.1.jolla.i486.drpm, 26.8 KiB<br>
Retrieving: libgrilio-1.0.26-1.7.2.jolla_1.0.29_1.9.1.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./libgrilio-1.0.26-1.7.2.jolla_1.0.29_1.9.1.jolla.i486.drpm ......................................................[done]<br>
Retrieving package libgofono-2.0.6-1.2.12.jolla.i486                                           (121/275),  54.1 KiB (192.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libgofono-2.0.6-1.3.2.jolla_2.0.6_1.2.12.jolla.i486.drpm, 44.6 KiB<br>
Retrieving: libgofono-2.0.6-1.3.2.jolla_2.0.6_1.2.12.jolla.i486.drpm .............................................................[done]<br>
Applying delta: ./libgofono-2.0.6-1.3.2.jolla_2.0.6_1.2.12.jolla.i486.drpm .......................................................[done]<br>
Retrieving package libdbusaccess-1.0.7-1.3.4.jolla.i486                                        (122/275),  22.6 KiB ( 36.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libdbusaccess-1.0.6-1.3.2.jolla_1.0.7_1.3.4.jolla.i486.drpm, 16.3 KiB<br>
Retrieving: libdbusaccess-1.0.6-1.3.2.jolla_1.0.7_1.3.4.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./libdbusaccess-1.0.6-1.3.2.jolla_1.0.7_1.3.4.jolla.i486.drpm ....................................................[done]<br>
Retrieving package systemd-libs-225+git13-1.4.4.jolla.i486                                     (123/275), 780.3 KiB (  5.2 MiB unpacked)<br>
Retrieving delta: ./drpms/systemd-libs-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm, 824.3 KiB<br>
Retrieving: systemd-libs-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./systemd-libs-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm .............................................[done]<br>
Retrieving package sudo-1.8.20p2-1.1.25.jolla.i486                                             (124/275), 358.2 KiB (980.4 KiB unpacked)<br>
Retrieving delta: ./drpms/sudo-1.8.20p2-1.2.2.jolla_1.8.20p2_1.1.25.jolla.i486.drpm, 237.0 KiB<br>
Retrieving: sudo-1.8.20p2-1.2.2.jolla_1.8.20p2_1.1.25.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./sudo-1.8.20p2-1.2.2.jolla_1.8.20p2_1.1.25.jolla.i486.drpm ......................................................[done]<br>
Retrieving package p11-kit-trust-0.23.12+git1-1.3.4.jolla.i486                                 (125/275), 110.9 KiB (343.4 KiB unpacked)<br>
Retrieving delta: ./drpms/p11-kit-trust-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm, 100.5 KiB<br>
Retrieving: p11-kit-trust-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm .................................................[done]<br>
Applying delta: ./p11-kit-trust-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm ...........................................[done]<br>
Retrieving package libuser-0.62+git1-1.2.6.jolla.i486                                          (126/275), 264.1 KiB (  1.6 MiB unpacked)<br>
Retrieving delta: ./drpms/libuser-0.62+git1-1.3.1.jolla_0.62+git1_1.2.6.jolla.i486.drpm, 98.7 KiB<br>
Retrieving: libuser-0.62+git1-1.3.1.jolla_0.62+git1_1.2.6.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./libuser-0.62+git1-1.3.1.jolla_0.62+git1_1.2.6.jolla.i486.drpm ..................................................[done]<br>
Retrieving package libsmartcols-2.33+git1-1.4.4.jolla.i486                                     (127/275),  97.6 KiB (228.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libsmartcols-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 80.7 KiB<br>
Retrieving: libsmartcols-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./libsmartcols-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .............................................[done]<br>
Retrieving package kbd-2.0.4-1.3.3.jolla.i486                                                  (128/275),   1.1 MiB (  2.5 MiB unpacked)<br>
Retrieving delta: ./drpms/kbd-1.15.3-1.3.1.jolla_2.0.4_1.3.3.jolla.i486.drpm, 324.9 KiB<br>
Retrieving: kbd-1.15.3-1.3.1.jolla_2.0.4_1.3.3.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./kbd-1.15.3-1.3.1.jolla_2.0.4_1.3.3.jolla.i486.drpm .............................................................[done]<br>
Retrieving package groff-1.18.1.4-1.1.13.jolla.i486                                            (129/275),   1.4 MiB (  5.0 MiB unpacked)<br>
Retrieving delta: ./drpms/groff-1.18.1.4-1.2.1.jolla_1.18.1.4_1.1.13.jolla.i486.drpm, 540.0 KiB<br>
Retrieving: groff-1.18.1.4-1.2.1.jolla_1.18.1.4_1.1.13.jolla.i486.drpm ..............................................[done (11.1 KiB/s)]<br>
Applying delta: ./groff-1.18.1.4-1.2.1.jolla_1.18.1.4_1.1.13.jolla.i486.drpm .....................................................[done]<br>
Retrieving package gawk-1:3.1.5-1.2.6.jolla.i486                                               (130/275), 182.0 KiB (678.5 KiB unpacked)<br>
Retrieving delta: ./drpms/gawk-3.1.5-1.3.1.jolla_3.1.5_1.2.6.jolla.i486.drpm, 167.1 KiB<br>
Retrieving: gawk-3.1.5-1.3.1.jolla_3.1.5_1.2.6.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./gawk-3.1.5-1.3.1.jolla_3.1.5_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Retrieving package fontconfig-2.12.4-1.2.10.jolla.i486                                         (131/275), 153.7 KiB (391.4 KiB unpacked)<br>
Retrieving delta: ./drpms/fontconfig-2.12.4-1.3.1.jolla_2.12.4_1.2.10.jolla.i486.drpm, 122.6 KiB<br>
Retrieving: fontconfig-2.12.4-1.3.1.jolla_2.12.4_1.2.10.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./fontconfig-2.12.4-1.3.1.jolla_2.12.4_1.2.10.jolla.i486.drpm ....................................................[done]<br>
Retrieving package libgofonoext-1.0.10-1.2.11.jolla.i486                                       (132/275),  25.3 KiB ( 65.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libgofonoext-1.0.10-1.3.2.jolla_1.0.10_1.2.11.jolla.i486.drpm, 18.3 KiB<br>
Retrieving: libgofonoext-1.0.10-1.3.2.jolla_1.0.10_1.2.11.jolla.i486.drpm ...........................................[done (13.9 KiB/s)]<br>
Applying delta: ./libgofonoext-1.0.10-1.3.2.jolla_1.0.10_1.2.11.jolla.i486.drpm ..................................................[done]<br>
Retrieving package dbus-libs-1.10.8+git1-1.1.12.jolla.i486                                     (133/275), 116.6 KiB (312.3 KiB unpacked)<br>
Retrieving delta: ./drpms/dbus-libs-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm, 103.8 KiB<br>
Retrieving: dbus-libs-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm .................................................[done]<br>
Applying delta: ./dbus-libs-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm ...........................................[done]<br>
Retrieving package cor-0.1.18-1.1.16.jolla.i486                                                (134/275),  61.3 KiB (185.5 KiB unpacked)<br>
Retrieving delta: ./drpms/cor-0.1.18-1.2.2.jolla_0.1.18_1.1.16.jolla.i486.drpm, 53.4 KiB<br>
Retrieving: cor-0.1.18-1.2.2.jolla_0.1.18_1.1.16.jolla.i486.drpm .......................................................[done (154 B/s)]<br>
Applying delta: ./cor-0.1.18-1.2.2.jolla_0.1.18_1.1.16.jolla.i486.drpm ...........................................................[done]<br>
Retrieving package p11-kit-nss-ckbi-0.23.12+git1-1.3.4.jolla.i486                              (135/275),   7.7 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/p11-kit-nss-ckbi-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm, 3.7 KiB<br>
Retrieving: p11-kit-nss-ckbi-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./p11-kit-nss-ckbi-0.23.12-1.3.1.jolla_0.23.12+git1_1.3.4.jolla.i486.drpm ........................................[done]<br>
Retrieving package passwd-0.79+git1-1.3.6.jolla.i486                                           (136/275),  83.0 KiB (393.0 KiB unpacked)<br>
Retrieving delta: ./drpms/passwd-0.79+git1-1.4.1.jolla_0.79+git1_1.3.6.jolla.i486.drpm, 24.1 KiB<br>
Retrieving: passwd-0.79+git1-1.4.1.jolla_0.79+git1_1.3.6.jolla.i486.drpm ............................................[done (13.9 KiB/s)]<br>
Applying delta: ./passwd-0.79+git1-1.4.1.jolla_0.79+git1_1.3.6.jolla.i486.drpm ...................................................[done]<br>
Retrieving package libuuid-2.33+git1-1.4.4.jolla.i486                                          (137/275),  22.9 KiB ( 26.3 KiB unpacked)<br>
Retrieving delta: ./drpms/libuuid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 18.5 KiB<br>
Retrieving: libuuid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./libuuid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm ..................................................[done]<br>
Retrieving package perl-macros-2:5.16.1-1.1.15.jolla.i486                                      (138/275),  10.9 KiB (  5.0 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-macros-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm, 4.7 KiB<br>
Retrieving: perl-macros-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./perl-macros-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm ...................................................[done]<br>
Retrieving package libdbuslogserver-dbus-1.0.15-1.4.7.jolla.i486                               (139/275),  24.6 KiB ( 44.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libdbuslogserver-dbus-1.0.15-1.5.2.jolla_1.0.15_1.4.7.jolla.i486.drpm, 19.2 KiB<br>
Retrieving: libdbuslogserver-dbus-1.0.15-1.5.2.jolla_1.0.15_1.4.7.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./libdbuslogserver-dbus-1.0.15-1.5.2.jolla_1.0.15_1.4.7.jolla.i486.drpm ..........................................[done]<br>
Retrieving package statefs-pp-0.3.35-1.1.19.jolla.i486                                         (140/275),  31.2 KiB ( 47.4 KiB unpacked)<br>
Retrieving delta: ./drpms/statefs-pp-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm, 26.9 KiB<br>
Retrieving: statefs-pp-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm ..............................................[done (4.0 KiB/s)]<br>
Applying delta: ./statefs-pp-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm ....................................................[done]<br>
Retrieving package gnutls-2.12.23.4-1.2.5.jolla.i486                                           (141/275), 315.1 KiB (  1.0 MiB unpacked)<br>
Retrieving delta: ./drpms/gnutls-2.12.23.4-1.3.1.jolla_2.12.23.4_1.2.5.jolla.i486.drpm, 255.6 KiB<br>
Retrieving: gnutls-2.12.23.4-1.3.1.jolla_2.12.23.4_1.2.5.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./gnutls-2.12.23.4-1.3.1.jolla_2.12.23.4_1.2.5.jolla.i486.drpm ...................................................[done]<br>
Retrieving package ca-certificates-2018.2.24-1.2.11.jolla.noarch                               (142/275), 342.0 KiB (928.0 KiB unpacked)<br>
Retrieving delta: ./drpms/ca-certificates-2018.2.24-1.3.1.jolla_2018.2.24_1.2.11.jolla.noarch.drpm, 14.2 KiB<br>
Retrieving: ca-certificates-2018.2.24-1.3.1.jolla_2018.2.24_1.2.11.jolla.noarch.drpm .............................................[done]<br>
Applying delta: ./ca-certificates-2018.2.24-1.3.1.jolla_2018.2.24_1.2.11.jolla.noarch.drpm .......................................[done]<br>
Retrieving package libblkid-2.33+git1-1.4.4.jolla.i486                                         (143/275), 134.7 KiB (316.5 KiB unpacked)<br>
Retrieving delta: ./drpms/libblkid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 125.9 KiB<br>
Retrieving: libblkid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libblkid-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .................................................[done]<br>
Retrieving package perl-libs-2:5.16.1-1.1.15.jolla.i486                                        (144/275), 583.9 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/perl-libs-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm, 544.8 KiB<br>
Retrieving: perl-libs-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./perl-libs-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm .....................................................[done]<br>
Retrieving package openssl-libs-1.0.2o+git2-1.4.4.jolla.i486                                   (145/275), 793.4 KiB (  2.3 MiB unpacked)<br>
Retrieving delta: ./drpms/openssl-libs-1.0.2o-1.4.1.jolla_1.0.2o+git2_1.4.4.jolla.i486.drpm, 709.0 KiB<br>
Retrieving: openssl-libs-1.0.2o-1.4.1.jolla_1.0.2o+git2_1.4.4.jolla.i486.drpm ..........................................[done (154 B/s)]<br>
Applying delta: ./openssl-libs-1.0.2o-1.4.1.jolla_1.0.2o+git2_1.4.4.jolla.i486.drpm ..............................................[done]<br>
Retrieving package libfdisk-2.33+git1-1.4.4.jolla.i486                                         (146/275), 171.6 KiB (427.8 KiB unpacked)<br>
Retrieving delta: ./drpms/libfdisk-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 155.6 KiB<br>
Retrieving: libfdisk-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libfdisk-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .................................................[done]<br>
Retrieving package perl-2:5.16.1-1.1.15.jolla.i486                                             (147/275),   8.7 MiB ( 27.8 MiB unpacked)<br>
Retrieving delta: ./drpms/perl-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm, 1.2 MiB<br>
Retrieving: perl-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./perl-5.16.1-1.2.1.jolla_5.16.1_1.1.15.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package libcurl-7.64.0+git1-1.8.4.jolla.i486                                        (148/275), 217.7 KiB (525.0 KiB unpacked)<br>
Retrieving delta: ./drpms/libcurl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm, 199.6 KiB<br>
Retrieving: libcurl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm .........................................................[done]<br>
Applying delta: ./libcurl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm ...................................................[done]<br>
Retrieving package libarchive-3.3.3+git1-1.2.6.jolla.i486                                      (149/275), 301.4 KiB (740.7 KiB unpacked)<br>
Retrieving delta: ./drpms/libarchive-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm, 274.6 KiB<br>
Retrieving: libarchive-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm ......................................[done (13.9 KiB/s)]<br>
Applying delta: ./libarchive-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package libmount-2.33+git1-1.4.4.jolla.i486                                         (150/275), 140.8 KiB (348.3 KiB unpacked)<br>
Retrieving delta: ./drpms/libmount-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 126.9 KiB<br>
Retrieving: libmount-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./libmount-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .................................................[done]<br>
Retrieving package perl-Scalar-List-Utils-2:1.25-1.1.15.jolla.i486                             (151/275),  33.0 KiB ( 43.3 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Scalar-List-Utils-1.25-1.2.1.jolla_1.25_1.1.15.jolla.i486.drpm, 13.5 KiB<br>
Retrieving: perl-Scalar-List-Utils-1.25-1.2.1.jolla_1.25_1.1.15.jolla.i486.drpm ..................................................[done]<br>
Applying delta: ./perl-Scalar-List-Utils-1.25-1.2.1.jolla_1.25_1.1.15.jolla.i486.drpm ............................................[done]<br>
Retrieving package qemu-usermode-common-2.1.0-1.2.6.jolla.i486                                 (152/275), 458.7 KiB (  2.8 MiB unpacked)<br>
Retrieving delta: ./drpms/qemu-usermode-common-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm, 455.5 KiB<br>
Retrieving: qemu-usermode-common-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm .....................................[done (424.0 KiB/s)]<br>
Applying delta: ./qemu-usermode-common-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package pacrunner-0.15+git1-1.3.6.jolla.i486                                        (153/275), 191.7 KiB (454.7 KiB unpacked)<br>
Retrieving delta: ./drpms/pacrunner-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm, 178.4 KiB<br>
Retrieving: pacrunner-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./pacrunner-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm ................................................[done]<br>
Retrieving package gnupg2-1:2.0.4+git2-1.4.4.jolla.i486                                        (154/275),   1.0 MiB (  4.9 MiB unpacked)<br>
Retrieving delta: ./drpms/gnupg2-2.0.4+git2-1.5.1.jolla_2.0.4+git2_1.4.4.jolla.i486.drpm, 616.6 KiB<br>
Retrieving: gnupg2-2.0.4+git2-1.5.1.jolla_2.0.4+git2_1.4.4.jolla.i486.drpm ...........................................[done (8.7 KiB/s)]<br>
Applying delta: ./gnupg2-2.0.4+git2-1.5.1.jolla_2.0.4+git2_1.4.4.jolla.i486.drpm .................................................[done]<br>
Retrieving package curl-7.64.0+git1-1.8.4.jolla.i486                                           (155/275), 115.8 KiB (350.5 KiB unpacked)<br>
Retrieving delta: ./drpms/curl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm, 111.4 KiB<br>
Retrieving: curl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./curl-7.63.0-1.8.1.jolla_7.64.0+git1_1.8.4.jolla.i486.drpm ......................................................[done]<br>
Retrieving package bsdtar-3.3.3+git1-1.2.6.jolla.i486                                          (156/275),  30.3 KiB ( 52.8 KiB unpacked)<br>
Retrieving delta: ./drpms/bsdtar-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm, 26.4 KiB<br>
Retrieving: bsdtar-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./bsdtar-3.3.3+git1-1.3.1.jolla_3.3.3+git1_1.2.6.jolla.i486.drpm .................................................[done]<br>
Retrieving package util-linux-2.33+git1-1.4.4.jolla.i486                                       (157/275), 795.9 KiB (  3.6 MiB unpacked)<br>
Retrieving delta: ./drpms/util-linux-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm, 784.9 KiB<br>
Retrieving: util-linux-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm .......................................[done (105.8 KiB/s)]<br>
Applying delta: ./util-linux-2.31+git2-1.4.1.jolla_2.33+git1_1.4.4.jolla.i486.drpm ...............................................[done]<br>
Retrieving package perl-Pod-Escapes-2:1.04-1.1.15.jolla.noarch                                 (158/275),  18.2 KiB ( 20.6 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Pod-Escapes-1.04-1.2.1.jolla_1.04_1.1.15.jolla.noarch.drpm, 5.1 KiB<br>
Retrieving: perl-Pod-Escapes-1.04-1.2.1.jolla_1.04_1.1.15.jolla.noarch.drpm ......................................................[done]<br>
Applying delta: ./perl-Pod-Escapes-1.04-1.2.1.jolla_1.04_1.1.15.jolla.noarch.drpm ................................................[done]<br>
Retrieving package qemu-usermode-static-2.1.0-1.2.6.jolla.i486                                 (159/275),   1.4 MiB (  9.3 MiB unpacked)<br>
Retrieving: qemu-usermode-static-2.1.0-1.2.6.jolla.i486.rpm ......................................................................[done]<br>
Retrieving package qemu-usermode-2.1.0-1.2.6.jolla.i486                                        (160/275), 822.1 KiB (  5.0 MiB unpacked)<br>
Retrieving delta: ./drpms/qemu-usermode-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm, 817.9 KiB<br>
Retrieving: qemu-usermode-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm ..............................................[done (2.6 KiB/s)]<br>
Applying delta: ./qemu-usermode-2.1.0-1.3.1.jolla_2.1.0_1.2.6.jolla.i486.drpm ....................................................[done]<br>
Retrieving package gpgme-1.2.0+git6-1.3.4.jolla.i486                                           (161/275), 111.1 KiB (454.5 KiB unpacked)<br>
Retrieving delta: ./drpms/gpgme-1.2.0+git6-1.4.1.jolla_1.2.0+git6_1.3.4.jolla.i486.drpm, 94.4 KiB<br>
Retrieving: gpgme-1.2.0+git6-1.4.1.jolla_1.2.0+git6_1.3.4.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./gpgme-1.2.0+git6-1.4.1.jolla_1.2.0+git6_1.3.4.jolla.i486.drpm ..................................................[done]<br>
Retrieving package rpm-libs-4.14.1+git9-1.5.6.jolla.i486                                       (162/275), 304.1 KiB (780.6 KiB unpacked)<br>
Retrieving delta: ./drpms/rpm-libs-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm, 286.2 KiB<br>
Retrieving: rpm-libs-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./rpm-libs-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package xdg-utils-1.1.2+git1-1.2.7.jolla.noarch                                     (163/275),  43.8 KiB (257.9 KiB unpacked)<br>
Retrieving delta: ./drpms/xdg-utils-1.1.2+git1-1.3.1.jolla_1.1.2+git1_1.2.7.jolla.noarch.drpm, 8.2 KiB<br>
Retrieving: xdg-utils-1.1.2+git1-1.3.1.jolla_1.1.2+git1_1.2.7.jolla.noarch.drpm ..................................................[done]<br>
Applying delta: ./xdg-utils-1.1.2+git1-1.3.1.jolla_1.1.2+git1_1.2.7.jolla.noarch.drpm ............................................[done]<br>
Retrieving package vim-enhanced-7.3.629-1.2.3.jolla.i486                                       (164/275), 774.1 KiB (  1.7 MiB unpacked)<br>
Retrieving delta: ./drpms/vim-enhanced-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm, 770.0 KiB<br>
Retrieving: vim-enhanced-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./vim-enhanced-7.3.629-1.2.1.jolla_7.3.629_1.2.3.jolla.i486.drpm .................................................[done]<br>
Retrieving package parted-3.0+mer4-1.2.12.jolla.i486                                           (165/275), 148.2 KiB (386.7 KiB unpacked)<br>
Retrieving delta: ./drpms/parted-3.0+mer4-1.3.2.jolla_3.0+mer4_1.2.12.jolla.i486.drpm, 129.4 KiB<br>
Retrieving: parted-3.0+mer4-1.3.2.jolla_3.0+mer4_1.2.12.jolla.i486.drpm ..........................................................[done]<br>
Applying delta: ./parted-3.0+mer4-1.3.2.jolla_3.0+mer4_1.2.12.jolla.i486.drpm ....................................................[done]<br>
Retrieving package fuse-2.9.0+git1-1.2.6.jolla.i486                                            (166/275),  32.1 KiB ( 57.2 KiB unpacked)<br>
Retrieving delta: ./drpms/fuse-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm, 22.0 KiB<br>
Retrieving: fuse-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm ............................................[done (15.6 KiB/s)]<br>
Applying delta: ./fuse-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm ...................................................[done]<br>
Retrieving package fakeroot-1.12.4-1.1.9.jolla.i486                                            (167/275),  87.3 KiB (233.8 KiB unpacked)<br>
Retrieving delta: ./drpms/fakeroot-1.12.4-1.2.1.jolla_1.12.4_1.1.9.jolla.i486.drpm, 35.2 KiB<br>
Retrieving: fakeroot-1.12.4-1.2.1.jolla_1.12.4_1.1.9.jolla.i486.drpm .............................................................[done]<br>
Applying delta: ./fakeroot-1.12.4-1.2.1.jolla_1.12.4_1.1.9.jolla.i486.drpm .......................................................[done]<br>
Retrieving package e2fsprogs-1.45.0+git1-1.3.4.jolla.i486                                      (168/275), 334.9 KiB (970.1 KiB unpacked)<br>
Retrieving delta: ./drpms/e2fsprogs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm, 321.0 KiB<br>
Retrieving: e2fsprogs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm ......................................[done (2.6 KiB/s)]<br>
Applying delta: ./e2fsprogs-1.43.1+git2-1.3.1.jolla_1.45.0+git1_1.3.4.jolla.i486.drpm ............................................[done]<br>
Retrieving package btrfs-progs-3.16+git2-1.4.2.jolla.i486                                      (169/275), 388.1 KiB (  2.9 MiB unpacked)<br>
Retrieving delta: ./drpms/btrfs-progs-3.16+git1-1.4.2.jolla_3.16+git2_1.4.2.jolla.i486.drpm, 389.8 KiB<br>
Retrieving: btrfs-progs-3.16+git1-1.4.2.jolla_3.16+git2_1.4.2.jolla.i486.drpm ......................................[done (156.7 KiB/s)]<br>
Applying delta: ./btrfs-progs-3.16+git1-1.4.2.jolla_3.16+git2_1.4.2.jolla.i486.drpm ..............................................[done]<br>
Retrieving package perl-Pod-Simple-2:3.20-1.1.15.jolla.noarch                                  (170/275), 198.5 KiB (488.0 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Pod-Simple-3.20-1.2.1.jolla_3.20_1.1.15.jolla.noarch.drpm, 14.9 KiB<br>
Retrieving: perl-Pod-Simple-3.20-1.2.1.jolla_3.20_1.1.15.jolla.noarch.drpm .......................................................[done]<br>
Applying delta: ./perl-Pod-Simple-3.20-1.2.1.jolla_3.20_1.1.15.jolla.noarch.drpm .................................................[done]<br>
Retrieving package rpm-4.14.1+git9-1.5.6.jolla.i486                                            (171/275), 332.5 KiB (  1.8 MiB unpacked)<br>
Retrieving delta: ./drpms/rpm-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm, 62.1 KiB<br>
Retrieving: rpm-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm ..............................................[done (154 B/s)]<br>
Applying delta: ./rpm-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm ..................................................[done]<br>
Retrieving package fuse-libs-2.9.0+git1-1.2.6.jolla.i486                                       (172/275),  82.1 KiB (227.8 KiB unpacked)<br>
Retrieving delta: ./drpms/fuse-libs-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm, 63.6 KiB<br>
Retrieving: fuse-libs-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./fuse-libs-2.9.0+git1-1.3.2.jolla_2.9.0+git1_1.2.6.jolla.i486.drpm ..............................................[done]<br>
Retrieving package perl-parent-2:0.225-1.1.15.jolla.noarch                                     (173/275),  13.3 KiB (  5.6 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-parent-0.225-1.2.1.jolla_0.225_1.1.15.jolla.noarch.drpm, 5.2 KiB<br>
Retrieving: perl-parent-0.225-1.2.1.jolla_0.225_1.1.15.jolla.noarch.drpm .........................................................[done]<br>
Applying delta: ./perl-parent-0.225-1.2.1.jolla_0.225_1.1.15.jolla.noarch.drpm ...................................................[done]<br>
Retrieving package libicu-63.1+git5-1.1.5.jolla.i486                                           (174/275),   7.8 MiB ( 30.6 MiB unpacked)<br>
Retrieving: libicu-63.1+git5-1.1.5.jolla.i486.rpm ....................................................................[done (4.2 MiB/s)]<br>
Retrieving package perl-Pod-Perldoc-2:3.17.00-1.1.15.jolla.noarch                              (175/275),  75.3 KiB (143.0 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Pod-Perldoc-3.17.00-1.2.1.jolla_3.17.00_1.1.15.jolla.noarch.drpm, 10.6 KiB<br>
Retrieving: perl-Pod-Perldoc-3.17.00-1.2.1.jolla_3.17.00_1.1.15.jolla.noarch.drpm ................................................[done]<br>
Applying delta: ./perl-Pod-Perldoc-3.17.00-1.2.1.jolla_3.17.00_1.1.15.jolla.noarch.drpm ..........................................[done]<br>
Retrieving package sqlite-libs-3.13.0+git3-1.3.5.jolla.i486                                    (176/275), 457.9 KiB (971.6 KiB unpacked)<br>
Retrieving delta: ./drpms/sqlite-libs-3.13.0+git2-1.4.2.jolla_3.13.0+git3_1.3.5.jolla.i486.drpm, 430.1 KiB<br>
Retrieving: sqlite-libs-3.13.0+git2-1.4.2.jolla_3.13.0+git3_1.3.5.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./sqlite-libs-3.13.0+git2-1.4.2.jolla_3.13.0+git3_1.3.5.jolla.i486.drpm ..........................................[done]<br>
Retrieving package qt5-qtcore-5.6.3+git9-1.9.1.jolla.i486                                      (177/275),   1.6 MiB (  4.6 MiB unpacked)<br>
Retrieving delta: ./drpms/qt5-qtcore-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm, 1.2 MiB<br>
Retrieving: qt5-qtcore-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./qt5-qtcore-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm .............................................[done]<br>
Retrieving package perl-Pod-Parser-2:1.51-1.1.15.jolla.noarch                                  (178/275), 124.7 KiB (305.0 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Pod-Parser-1.51-1.2.1.jolla_1.51_1.1.15.jolla.noarch.drpm, 9.1 KiB<br>
Retrieving: perl-Pod-Parser-1.51-1.2.1.jolla_1.51_1.1.15.jolla.noarch.drpm .............................................[done (154 B/s)]<br>
Applying delta: ./perl-Pod-Parser-1.51-1.2.1.jolla_1.51_1.1.15.jolla.noarch.drpm .................................................[done]<br>
Retrieving package nss-3.39-1.3.5.jolla.i486                                                   (179/275), 871.6 KiB (  2.7 MiB unpacked)<br>
Retrieving delta: ./drpms/nss-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm, 803.9 KiB<br>
Retrieving: nss-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm ......................................................................[done]<br>
Applying delta: ./nss-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm ................................................................[done]<br>
Retrieving package qt5-qtdbus-5.6.3+git9-1.9.1.jolla.i486                                      (180/275), 164.8 KiB (492.8 KiB unpacked)<br>
Retrieving delta: ./drpms/qt5-qtdbus-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm, 155.6 KiB<br>
Retrieving: qt5-qtdbus-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./qt5-qtdbus-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm .............................................[done]<br>
Retrieving package perl-Filter-2:1.40-1.1.15.jolla.i486                                        (181/275),  40.6 KiB ( 57.4 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Filter-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm, 9.3 KiB<br>
Retrieving: perl-Filter-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm .............................................................[done]<br>
Applying delta: ./perl-Filter-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm .......................................................[done]<br>
Retrieving package python-2.7.15+git2-1.3.5.jolla.i486                                         (182/275),  17.8 KiB ( 16.2 KiB unpacked)<br>
Retrieving delta: ./drpms/python-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm, 10.1 KiB<br>
Retrieving: python-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./python-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm ...............................................[done]<br>
Retrieving package openvpn-2.4.5+git2-1.3.5.jolla.i486                                         (183/275), 305.7 KiB (803.9 KiB unpacked)<br>
Retrieving delta: ./drpms/openvpn-2.4.5+git2-1.4.1.jolla_2.4.5+git2_1.3.5.jolla.i486.drpm, 286.8 KiB<br>
Retrieving: openvpn-2.4.5+git2-1.4.1.jolla_2.4.5+git2_1.3.5.jolla.i486.drpm ..........................................[done (2.6 KiB/s)]<br>
Applying delta: ./openvpn-2.4.5+git2-1.4.1.jolla_2.4.5+git2_1.3.5.jolla.i486.drpm ................................................[done]<br>
Retrieving package openconnect-7.08+git1-1.2.4.jolla.i486                                      (184/275), 374.6 KiB (  1.9 MiB unpacked)<br>
Retrieving delta: ./drpms/openconnect-7.08+git1-1.3.1.jolla_7.08+git1_1.2.4.jolla.i486.drpm, 98.0 KiB<br>
Retrieving: openconnect-7.08+git1-1.3.1.jolla_7.08+git1_1.2.4.jolla.i486.drpm ........................................[done (5.4 KiB/s)]<br>
Applying delta: ./openconnect-7.08+git1-1.3.1.jolla_7.08+git1_1.2.4.jolla.i486.drpm ..............................................[done]<br>
Retrieving package nss-sysinit-3.39-1.3.5.jolla.i486                                           (185/275),  15.8 KiB ( 10.9 KiB unpacked)<br>
Retrieving delta: ./drpms/nss-sysinit-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm, 10.1 KiB<br>
Retrieving: nss-sysinit-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm ....................................................[done (154 B/s)]<br>
Applying delta: ./nss-sysinit-3.39-1.3.2.jolla_3.39_1.3.5.jolla.i486.drpm ........................................................[done]<br>
Retrieving package libsolv0-0.6.35+git2-1.4.6.jolla.i486                                       (186/275), 337.5 KiB (794.4 KiB unpacked)<br>
Retrieving delta: ./drpms/libsolv0-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm, 318.7 KiB<br>
Retrieving: libsolv0-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./libsolv0-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package deltarpm-3.5-1.2.6.jolla.i486                                               (187/275),  65.4 KiB (177.5 KiB unpacked)<br>
Retrieving delta: ./drpms/deltarpm-3.5-1.3.1.jolla_3.5_1.2.6.jolla.i486.drpm, 60.9 KiB<br>
Retrieving: deltarpm-3.5-1.3.1.jolla_3.5_1.2.6.jolla.i486.drpm ...................................................................[done]<br>
Applying delta: ./deltarpm-3.5-1.3.1.jolla_3.5_1.2.6.jolla.i486.drpm .............................................................[done]<br>
Retrieving package cryptsetup-libs-2.1.0+git1-1.3.4.jolla.i486                                 (188/275), 315.9 KiB (  1.4 MiB unpacked)<br>
Retrieving: cryptsetup-libs-2.1.0+git1-1.3.4.jolla.i486.rpm ..........................................................[done (5.4 KiB/s)]<br>
Retrieving package createrepo_c-libs-0.10.0+git1-1.1.47.jolla.i486                             (189/275),  73.8 KiB (186.8 KiB unpacked)<br>
Retrieving: createrepo_c-libs-0.10.0+git1-1.1.47.jolla.i486.rpm ..................................................................[done]<br>
Retrieving package qt5-qtnetwork-5.6.3+git9-1.9.1.jolla.i486                                   (190/275), 416.2 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/qt5-qtnetwork-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm, 400.4 KiB<br>
Retrieving: qt5-qtnetwork-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./qt5-qtnetwork-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ..........................................[done]<br>
Retrieving package perl-Module-Pluggable-2:4.00-1.1.15.jolla.noarch                            (191/275),  25.6 KiB ( 30.3 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Module-Pluggable-4.00-1.2.1.jolla_4.00_1.1.15.jolla.noarch.drpm, 6.1 KiB<br>
Retrieving: perl-Module-Pluggable-4.00-1.2.1.jolla_4.00_1.1.15.jolla.noarch.drpm .................................................[done]<br>
Applying delta: ./perl-Module-Pluggable-4.00-1.2.1.jolla_4.00_1.1.15.jolla.noarch.drpm ...........................................[done]<br>
Retrieving package python-libs-2.7.15+git2-1.3.5.jolla.i486                                    (192/275),   6.6 MiB ( 26.5 MiB unpacked)<br>
Retrieving delta: ./drpms/python-libs-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm, 1.4 MiB<br>
Retrieving: python-libs-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm ................................................[done]<br>
Applying delta: ./python-libs-2.7.15+git1-1.3.2.jolla_2.7.15+git2_1.3.5.jolla.i486.drpm ..........................................[done]<br>
Retrieving package libsolv-tools-0.6.35+git2-1.4.6.jolla.i486                                  (193/275),  51.6 KiB (173.2 KiB unpacked)<br>
Retrieving delta: ./drpms/libsolv-tools-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm, 47.6 KiB<br>
Retrieving: libsolv-tools-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./libsolv-tools-0.6.35+git2-1.5.1.jolla_0.6.35+git2_1.4.6.jolla.i486.drpm ........................................[done]<br>
Retrieving package systemd-config-sailfish-0.8.15-1.7.2.jolla.noarch                           (194/275),  33.9 KiB (  3.7 KiB unpacked)<br>
Retrieving delta: ./drpms/systemd-config-sailfish-0.8.14-1.7.2.jolla_0.8.15_1.7.2.jolla.noarch.drpm, 30.0 KiB<br>
Retrieving: systemd-config-sailfish-0.8.14-1.7.2.jolla_0.8.15_1.7.2.jolla.noarch.drpm ............................................[done]<br>
Applying delta: ./systemd-config-sailfish-0.8.14-1.7.2.jolla_0.8.15_1.7.2.jolla.noarch.drpm ......................................[done]<br>
Retrieving package createrepo_c-0.10.0+git1-1.1.47.jolla.i486                                  (195/275),  43.2 KiB (101.5 KiB unpacked)<br>
Retrieving: createrepo_c-0.10.0+git1-1.1.47.jolla.i486.rpm .......................................................................[done]<br>
Retrieving package qt5-qtxml-5.6.3+git9-1.9.1.jolla.i486                                       (196/275),  82.8 KiB (229.0 KiB unpacked)<br>
Retrieving delta: ./drpms/qt5-qtxml-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm, 78.1 KiB<br>
Retrieving: qt5-qtxml-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./qt5-qtxml-5.6.3+git8-1.8.2.jolla_5.6.3+git9_1.9.1.jolla.i486.drpm ..............................................[done]<br>
Retrieving package libqtaround2-0.2.8-1.2.1.jolla.i486                                         (197/275),  80.2 KiB (224.5 KiB unpacked)<br>
Retrieving: libqtaround2-0.2.8-1.2.1.jolla.i486.rpm .................................................................[done (15.5 KiB/s)]<br>
Retrieving package perl-threads-2:1.86-1.1.15.jolla.i486                                       (198/275),  46.6 KiB ( 78.6 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-threads-1.86-1.2.1.jolla_1.86_1.1.15.jolla.i486.drpm, 17.5 KiB<br>
Retrieving: perl-threads-1.86-1.2.1.jolla_1.86_1.1.15.jolla.i486.drpm ............................................................[done]<br>
Applying delta: ./perl-threads-1.86-1.2.1.jolla_1.86_1.1.15.jolla.i486.drpm ......................................................[done]<br>
Retrieving package rpm-python-4.14.1+git9-1.4.4.jolla.i486                                     (199/275),  63.6 KiB (158.6 KiB unpacked)<br>
Retrieving delta: ./drpms/rpm-python-4.14.1+git7-1.5.1.jolla_4.14.1+git9_1.4.4.jolla.i486.drpm, 44.8 KiB<br>
Retrieving: rpm-python-4.14.1+git7-1.5.1.jolla_4.14.1+git9_1.4.4.jolla.i486.drpm .................................................[done]<br>
Applying delta: ./rpm-python-4.14.1+git7-1.5.1.jolla_4.14.1+git9_1.4.4.jolla.i486.drpm ...........................................[done]<br>
Retrieving package python-yaml-3.10-1.1.10.jolla.i486                                          (200/275),  89.5 KiB (427.4 KiB unpacked)<br>
Retrieving delta: ./drpms/python-yaml-3.10-1.2.1.jolla_3.10_1.1.10.jolla.i486.drpm, 13.2 KiB<br>
Retrieving: python-yaml-3.10-1.2.1.jolla_3.10_1.1.10.jolla.i486.drpm .............................................................[done]<br>
Applying delta: ./python-yaml-3.10-1.2.1.jolla_3.10_1.1.10.jolla.i486.drpm .......................................................[done]<br>
Retrieving package python-setuptools-0.6c11-1.1.10.jolla.noarch                                (201/275), 241.3 KiB (981.5 KiB unpacked)<br>
Retrieving delta: ./drpms/python-setuptools-0.6c11-1.2.1.jolla_0.6c11_1.1.10.jolla.noarch.drpm, 20.5 KiB<br>
Retrieving: python-setuptools-0.6c11-1.2.1.jolla_0.6c11_1.1.10.jolla.noarch.drpm .................................................[done]<br>
Applying delta: ./python-setuptools-0.6c11-1.2.1.jolla_0.6c11_1.1.10.jolla.noarch.drpm ...........................................[done]<br>
Retrieving package python-pycurl-7.19.0+mer1-1.1.45.jolla.i486                                 (202/275),  72.7 KiB (221.6 KiB unpacked)<br>
Retrieving delta: ./drpms/python-pycurl-7.19.0+mer1-1.2.3.jolla_7.19.0+mer1_1.1.45.jolla.i486.drpm, 27.4 KiB<br>
Retrieving: python-pycurl-7.19.0+mer1-1.2.3.jolla_7.19.0+mer1_1.1.45.jolla.i486.drpm .............................................[done]<br>
Applying delta: ./python-pycurl-7.19.0+mer1-1.2.3.jolla_7.19.0+mer1_1.1.45.jolla.i486.drpm .......................................[done]<br>
Retrieving package python-lxml-3.2.0-1.1.12.jolla.i486                                         (203/275),   1.7 MiB ( 22.9 MiB unpacked)<br>
Retrieving delta: ./drpms/python-lxml-3.2.0-1.2.1.jolla_3.2.0_1.1.12.jolla.i486.drpm, 539.0 KiB<br>
Retrieving: python-lxml-3.2.0-1.2.1.jolla_3.2.0_1.1.12.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./python-lxml-3.2.0-1.2.1.jolla_3.2.0_1.1.12.jolla.i486.drpm .....................................................[done]<br>
Retrieving package python-cheetah-2.4.4+mer1-1.1.31.jolla.i486                                 (204/275), 307.1 KiB (  1.9 MiB unpacked)<br>
Retrieving delta: ./drpms/python-cheetah-2.4.4+mer1-1.2.4.jolla_2.4.4+mer1_1.1.31.jolla.i486.drpm, 35.0 KiB<br>
Retrieving: python-cheetah-2.4.4+mer1-1.2.4.jolla_2.4.4+mer1_1.1.31.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./python-cheetah-2.4.4+mer1-1.2.4.jolla_2.4.4+mer1_1.1.31.jolla.i486.drpm ........................................[done]<br>
Retrieving package python-M2Crypto-0.23.0-1.1.11.jolla.i486                                    (205/275), 216.4 KiB (  1.1 MiB unpacked)<br>
Retrieving delta: ./drpms/python-M2Crypto-0.23.0-1.2.1.jolla_0.23.0_1.1.11.jolla.i486.drpm, 100.8 KiB<br>
Retrieving: python-M2Crypto-0.23.0-1.2.1.jolla_0.23.0_1.1.11.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./python-M2Crypto-0.23.0-1.2.1.jolla_0.23.0_1.1.11.jolla.i486.drpm ...............................................[done]<br>
Retrieving package pacrunner-python-0.15+git1-1.3.6.jolla.i486                                 (206/275),  12.6 KiB (  9.3 KiB unpacked)<br>
Retrieving delta: ./drpms/pacrunner-python-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm, 5.9 KiB<br>
Retrieving: pacrunner-python-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm ...............................................[done]<br>
Applying delta: ./pacrunner-python-0.15+git1-1.4.1.jolla_0.15+git1_1.3.6.jolla.i486.drpm .........................................[done]<br>
Retrieving package libzypp-17.3.1+git4-1.6.2.jolla.i486                                        (207/275),   1.8 MiB (  7.1 MiB unpacked)<br>
Retrieving delta: ./drpms/libzypp-17.3.1-1.6.1.jolla_17.3.1+git4_1.6.2.jolla.i486.drpm, 1.4 MiB<br>
Retrieving: libzypp-17.3.1-1.6.1.jolla_17.3.1+git4_1.6.2.jolla.i486.drpm ............................................[done (13.9 KiB/s)]<br>
Applying delta: ./libzypp-17.3.1-1.6.1.jolla_17.3.1+git4_1.6.2.jolla.i486.drpm ...................................................[done]<br>
Retrieving package device-mapper-libs-2.02.177+git3-1.3.6.jolla.i486                           (208/275), 129.7 KiB (331.5 KiB unpacked)<br>
Retrieving delta: ./drpms/device-mapper-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 112.4 KiB<br>
Retrieving: device-mapper-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm .....................................[done]<br>
Applying delta: ./device-mapper-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ...............................[done]<br>
Retrieving package statefs-qt5-0.3.5-1.4.1.jolla.i486                                          (209/275),  29.0 KiB ( 52.1 KiB unpacked)<br>
Retrieving: statefs-qt5-0.3.5-1.4.1.jolla.i486.rpm ..................................................................[done (15.5 KiB/s)]<br>
Retrieving package perl-Socket-2:2.001-1.1.15.jolla.i486                                       (210/275),  38.5 KiB ( 70.8 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Socket-2.001-1.2.1.jolla_2.001_1.1.15.jolla.i486.drpm, 14.3 KiB<br>
Retrieving: perl-Socket-2.001-1.2.1.jolla_2.001_1.1.15.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./perl-Socket-2.001-1.2.1.jolla_2.001_1.1.15.jolla.i486.drpm .....................................................[done]<br>
Retrieving package python-distro-1.0.4+mer1-1.1.10.jolla.noarch                                (211/275),  23.0 KiB ( 81.2 KiB unpacked)<br>
Retrieving delta: ./drpms/python-distro-1.0.4+mer1-1.2.1.jolla_1.0.4+mer1_1.1.10.jolla.noarch.drpm, 4.9 KiB<br>
Retrieving: python-distro-1.0.4+mer1-1.2.1.jolla_1.0.4+mer1_1.1.10.jolla.noarch.drpm .............................................[done]<br>
Applying delta: ./python-distro-1.0.4+mer1-1.2.1.jolla_1.0.4+mer1_1.1.10.jolla.noarch.drpm .......................................[done]<br>
Retrieving package repomd-pattern-builder-0.3.3-1.2.7.jolla.i486                               (212/275),  10.8 KiB (  6.8 KiB unpacked)<br>
Retrieving delta: ./drpms/repomd-pattern-builder-0.3.3-1.3.1.jolla_0.3.3_1.2.7.jolla.i486.drpm, 4.4 KiB<br>
Retrieving: repomd-pattern-builder-0.3.3-1.3.1.jolla_0.3.3_1.2.7.jolla.i486.drpm .................................................[done]<br>
Applying delta: ./repomd-pattern-builder-0.3.3-1.3.1.jolla_0.3.3_1.2.7.jolla.i486.drpm ...........................................[done]<br>
Retrieving package python-urlgrabber-3.9.1+mer1-1.1.26.jolla.noarch                            (213/275),  78.0 KiB (313.2 KiB unpacked)<br>
Retrieving delta: ./drpms/python-urlgrabber-3.9.1+mer1-1.2.3.jolla_3.9.1+mer1_1.1.26.jolla.noarch.drpm, 7.5 KiB<br>
Retrieving: python-urlgrabber-3.9.1+mer1-1.2.3.jolla_3.9.1+mer1_1.1.26.jolla.noarch.drpm .........................................[done]<br>
Applying delta: ./python-urlgrabber-3.9.1+mer1-1.2.3.jolla_3.9.1+mer1_1.1.26.jolla.noarch.drpm ...................................[done]<br>
Retrieving package zypper-1.14.6+git3-1.3.6.jolla.i486                                         (214/275),   1.2 MiB (  6.7 MiB unpacked)<br>
Retrieving delta: ./drpms/zypper-1.14.6+git3-1.4.1.jolla_1.14.6+git3_1.3.6.jolla.i486.drpm, 610.1 KiB<br>
Retrieving: zypper-1.14.6+git3-1.4.1.jolla_1.14.6+git3_1.3.6.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./zypper-1.14.6+git3-1.4.1.jolla_1.14.6+git3_1.3.6.jolla.i486.drpm ...............................................[done]<br>
Retrieving package python-zypp-0.7.4+git3-1.2.9.jolla.i486                                     (215/275), 844.0 KiB (  5.4 MiB unpacked)<br>
Retrieving: python-zypp-0.7.4+git3-1.2.9.jolla.i486.rpm ..........................................................................[done]<br>
Retrieving package device-mapper-event-libs-2.02.177+git3-1.3.6.jolla.i486                     (216/275),  16.8 KiB ( 21.6 KiB unpacked)<br>
Retrieving delta: ./drpms/device-mapper-event-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 11.5 KiB<br>
Retrieving: device-mapper-event-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ...............................[done]<br>
Applying delta: ./device-mapper-event-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm .........................[done]<br>
Retrieving package perl-threads-shared-2:1.40-1.1.15.jolla.i486                                (217/275),  35.9 KiB ( 59.4 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-threads-shared-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm, 17.4 KiB<br>
Retrieving: perl-threads-shared-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./perl-threads-shared-1.40-1.2.1.jolla_1.40_1.1.15.jolla.i486.drpm ...............................................[done]<br>
Retrieving package osc-0.146.0-1.1.33.jolla.noarch                                             (218/275), 381.3 KiB (  1.7 MiB unpacked)<br>
Retrieving delta: ./drpms/osc-0.146.0-1.2.5.jolla_0.146.0_1.1.33.jolla.noarch.drpm, 19.6 KiB<br>
Retrieving: osc-0.146.0-1.2.5.jolla_0.146.0_1.1.33.jolla.noarch.drpm .............................................................[done]<br>
Applying delta: ./osc-0.146.0-1.2.5.jolla_0.146.0_1.1.33.jolla.noarch.drpm .......................................................[done]<br>
Retrieving package lvm2-libs-2.02.177+git3-1.3.6.jolla.i486                                    (219/275), 700.0 KiB (  3.1 MiB unpacked)<br>
Retrieving delta: ./drpms/lvm2-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 606.9 KiB<br>
Retrieving: lvm2-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ....................................[done (154 B/s)]<br>
Applying delta: ./lvm2-libs-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ........................................[done]<br>
Retrieving package spectacle-0.30-1.1.24.jolla.noarch                                          (220/275), 132.4 KiB (726.7 KiB unpacked)<br>
Retrieving delta: ./drpms/spectacle-0.30-1.2.4.jolla_0.30_1.1.24.jolla.noarch.drpm, 23.5 KiB<br>
Retrieving: spectacle-0.30-1.2.4.jolla_0.30_1.1.24.jolla.noarch.drpm ...................................................[done (154 B/s)]<br>
Applying delta: ./spectacle-0.30-1.2.4.jolla_0.30_1.1.24.jolla.noarch.drpm .......................................................[done]<br>
Retrieving package scratchbox2-2.3.90+git17-1.5.3.jolla.i486                                   (221/275), 227.8 KiB (794.4 KiB unpacked)<br>
Retrieving delta: ./drpms/scratchbox2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm, 126.3 KiB<br>
Retrieving: scratchbox2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm ....................................[done (154 B/s)]<br>
Applying delta: ./scratchbox2-2.3.90+git15-1.5.1.jolla_2.3.90+git17_1.5.3.jolla.i486.drpm ........................................[done]<br>
Retrieving package rpm-build-4.14.1+git9-1.5.6.jolla.i486                                      (222/275), 107.5 KiB (260.9 KiB unpacked)<br>
Retrieving delta: ./drpms/rpm-build-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm, 54.4 KiB<br>
Retrieving: rpm-build-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm ..................................................[done]<br>
Applying delta: ./rpm-build-4.14.1+git7-1.6.1.jolla_4.14.1+git9_1.5.6.jolla.i486.drpm ............................................[done]<br>
Retrieving package perl-Error-0.17020+mer2-1.1.15.jolla.noarch                                 (223/275),  30.7 KiB ( 48.7 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Error-0.17020+mer2-1.2.2.jolla_0.17020+mer2_1.1.15.jolla.noarch.drpm, 6.2 KiB<br>
Retrieving: perl-Error-0.17020+mer2-1.2.2.jolla_0.17020+mer2_1.1.15.jolla.noarch.drpm ............................................[done]<br>
Applying delta: ./perl-Error-0.17020+mer2-1.2.2.jolla_0.17020+mer2_1.1.15.jolla.noarch.drpm ......................................[done]<br>
Retrieving package android-tools-hadk-5.1.1+git2-1.1.30.jolla.i486                             (224/275), 122.6 KiB (291.8 KiB unpacked)<br>
Retrieving delta: ./drpms/android-tools-hadk-5.1.1+git2-1.2.3.jolla_5.1.1+git2_1.1.30.jolla.i486.drpm, 113.3 KiB<br>
Retrieving: android-tools-hadk-5.1.1+git2-1.2.3.jolla_5.1.1+git2_1.1.30.jolla.i486.drpm ..........................................[done]<br>
Applying delta: ./android-tools-hadk-5.1.1+git2-1.2.3.jolla_5.1.1+git2_1.1.30.jolla.i486.drpm ....................................[done]<br>
Retrieving package lvm2-2.02.177+git3-1.3.6.jolla.i486                                         (225/275), 634.5 KiB (  2.0 MiB unpacked)<br>
Retrieving delta: ./drpms/lvm2-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 604.9 KiB<br>
Retrieving: lvm2-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./lvm2-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package systemd-225+git13-1.4.4.jolla.i486                                          (226/275),   4.8 MiB ( 39.5 MiB unpacked)<br>
Retrieving delta: ./drpms/systemd-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm, 4.0 MiB<br>
Retrieving: systemd-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm ..........................................[done (879.7 KiB/s)]<br>
Applying delta: ./systemd-225+git12-1.4.1.jolla_225+git13_1.4.4.jolla.i486.drpm ..................................................[done]<br>
Retrieving package dbus-1.10.8+git1-1.1.12.jolla.i486                                          (227/275), 132.5 KiB (371.1 KiB unpacked)<br>
Retrieving delta: ./drpms/dbus-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm, 120.0 KiB<br>
Retrieving: dbus-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./dbus-1.10.8+git1-1.2.1.jolla_1.10.8+git1_1.1.12.jolla.i486.drpm ................................................[done]<br>
Retrieving package polkit-0.105+git2-1.2.5.jolla.i486                                          (228/275), 113.8 KiB (339.5 KiB unpacked)<br>
Retrieving delta: ./drpms/polkit-0.105+git2-1.3.1.jolla_0.105+git2_1.2.5.jolla.i486.drpm, 93.2 KiB<br>
Retrieving: polkit-0.105+git2-1.3.1.jolla_0.105+git2_1.2.5.jolla.i486.drpm ...........................................[done (1.2 KiB/s)]<br>
Applying delta: ./polkit-0.105+git2-1.3.1.jolla_0.105+git2_1.2.5.jolla.i486.drpm .................................................[done]<br>
Retrieving package device-mapper-2.02.177+git3-1.3.6.jolla.i486                                (229/275),  65.3 KiB (180.7 KiB unpacked)<br>
Retrieving delta: ./drpms/device-mapper-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 47.9 KiB<br>
Retrieving: device-mapper-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ..........................................[done]<br>
Applying delta: ./device-mapper-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ....................................[done]<br>
Retrieving package PackageKit-glib-1.1.9+git5-1.7.2.jolla.i486                                 (230/275), 105.7 KiB (343.6 KiB unpacked)<br>
Retrieving delta: ./drpms/PackageKit-glib-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm, 91.5 KiB<br>
Retrieving: PackageKit-glib-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm ..................................[done (6.8 KiB/s)]<br>
Applying delta: ./PackageKit-glib-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm ........................................[done]<br>
Retrieving package device-mapper-event-2.02.177+git3-1.3.6.jolla.i486                          (231/275),  23.1 KiB ( 33.4 KiB unpacked)<br>
Retrieving delta: ./drpms/device-mapper-event-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm, 19.1 KiB<br>
Retrieving: device-mapper-event-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ....................................[done]<br>
Applying delta: ./device-mapper-event-2.02.177+git3-1.4.1.jolla_2.02.177+git3_1.3.6.jolla.i486.drpm ..............................[done]<br>
Retrieving package PackageKit-1.1.9+git5-1.7.2.jolla.i486                                      (232/275), 385.5 KiB (  2.0 MiB unpacked)<br>
Retrieving delta: ./drpms/PackageKit-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm, 176.7 KiB<br>
Retrieving: PackageKit-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./PackageKit-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm .............................................[done]<br>
Retrieving package wpa_supplicant-2.6+git5-1.3.6.jolla.i486                                    (233/275), 560.4 KiB (  1.5 MiB unpacked)<br>
Retrieving delta: ./drpms/wpa_supplicant-2.6+git5-1.4.1.jolla_2.6+git5_1.3.6.jolla.i486.drpm, 555.9 KiB<br>
Retrieving: wpa_supplicant-2.6+git5-1.4.1.jolla_2.6+git5_1.3.6.jolla.i486.drpm ...................................................[done]<br>
Applying delta: ./wpa_supplicant-2.6+git5-1.4.1.jolla_2.6+git5_1.3.6.jolla.i486.drpm .............................................[done]<br>
Retrieving package systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch                       (234/275),   8.7 KiB (  1.3 KiB unpacked)<br>
Retrieving delta: ./drpms/systemd-user-session-targets-0.0.2-1.4.1.jolla_0.0.2_1.3.5.jolla.noarch.drpm, 4.4 KiB<br>
Retrieving: systemd-user-session-targets-0.0.2-1.4.1.jolla_0.0.2_1.3.5.jolla.noarch.drpm .........................................[done]<br>
Applying delta: ./systemd-user-session-targets-0.0.2-1.4.1.jolla_0.0.2_1.3.5.jolla.noarch.drpm ...................................[done]<br>
Retrieving package openssh-7.9p1+git2-1.5.4.jolla.i486                                         (235/275), 243.0 KiB (  1.4 MiB unpacked)<br>
Retrieving delta: ./drpms/openssh-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm, 234.7 KiB<br>
Retrieving: openssh-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm .........................................[done (15.5 KiB/s)]<br>
Applying delta: ./openssh-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm ................................................[done]<br>
Retrieving package ofono-1.21+git44-1.19.1.jolla.i486                                          (236/275), 520.4 KiB (  1.3 MiB unpacked)<br>
Retrieving delta: ./drpms/ofono-1.21+git38-1.18.2.jolla_1.21+git44_1.19.1.jolla.i486.drpm, 510.0 KiB<br>
Retrieving: ofono-1.21+git38-1.18.2.jolla_1.21+git44_1.19.1.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./ofono-1.21+git38-1.18.2.jolla_1.21+git44_1.19.1.jolla.i486.drpm ................................................[done]<br>
Retrieving package kpartx-0.4.9+mer1-1.1.41.jolla.i486                                         (237/275),  23.9 KiB ( 40.0 KiB unpacked)<br>
Retrieving delta: ./drpms/kpartx-0.4.9+mer1-1.2.3.jolla_0.4.9+mer1_1.1.41.jolla.i486.drpm, 18.1 KiB<br>
Retrieving: kpartx-0.4.9+mer1-1.2.3.jolla_0.4.9+mer1_1.1.41.jolla.i486.drpm ......................................................[done]<br>
Applying delta: ./kpartx-0.4.9+mer1-1.2.3.jolla_0.4.9+mer1_1.1.41.jolla.i486.drpm ................................................[done]<br>
Retrieving package crda-4.14+git2-1.3.6.jolla.i486                                             (238/275),  22.6 KiB ( 30.7 KiB unpacked)<br>
Retrieving delta: ./drpms/crda-4.14+git2-1.4.1.jolla_4.14+git2_1.3.6.jolla.i486.drpm, 18.3 KiB<br>
Retrieving: crda-4.14+git2-1.4.1.jolla_4.14+git2_1.3.6.jolla.i486.drpm ...........................................................[done]<br>
Applying delta: ./crda-4.14+git2-1.4.1.jolla_4.14+git2_1.3.6.jolla.i486.drpm .....................................................[done]<br>
Retrieving package PackageKit-zypp-1.1.9+git5-1.7.2.jolla.i486                                 (239/275), 104.2 KiB (272.8 KiB unpacked)<br>
Retrieving delta: ./drpms/PackageKit-zypp-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm, 91.7 KiB<br>
Retrieving: PackageKit-zypp-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./PackageKit-zypp-1.1.9+git3-1.6.2.jolla_1.1.9+git5_1.7.2.jolla.i486.drpm ........................................[done]<br>
Retrieving package oneshot-0.4.8-1.2.8.jolla.noarch                                            (240/275),  15.2 KiB (  8.3 KiB unpacked)<br>
Retrieving delta: ./drpms/oneshot-0.4.8-1.3.1.jolla_0.4.8_1.2.8.jolla.noarch.drpm, 8.9 KiB<br>
Retrieving: oneshot-0.4.8-1.3.1.jolla_0.4.8_1.2.8.jolla.noarch.drpm ..............................................................[done]<br>
Applying delta: ./oneshot-0.4.8-1.3.1.jolla_0.4.8_1.2.8.jolla.noarch.drpm ........................................................[done]<br>
Retrieving package openssh-clients-7.9p1+git2-1.5.4.jolla.i486                                 (241/275), 424.5 KiB (  2.2 MiB unpacked)<br>
Retrieving delta: ./drpms/openssh-clients-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm, 417.0 KiB<br>
Retrieving: openssh-clients-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm ..............................................[done]<br>
Applying delta: ./openssh-clients-7.7p1+git4-1.5.1.jolla_7.9p1+git2_1.5.4.jolla.i486.drpm ........................................[done]<br>
Retrieving package statefs-0.3.35-1.1.19.jolla.i486                                            (242/275), 168.1 KiB (642.4 KiB unpacked)<br>
Retrieving delta: ./drpms/statefs-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm, 163.5 KiB<br>
Retrieving: statefs-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm ................................................[done (15.5 KiB/s)]<br>
Applying delta: ./statefs-0.3.35-1.2.2.jolla_0.3.35_1.1.19.jolla.i486.drpm .......................................................[done]<br>
Retrieving package perl-Git-1.8.3+mer2-1.1.22.jolla.i486                                       (243/275),  24.0 KiB ( 45.1 KiB unpacked)<br>
Retrieving delta: ./drpms/perl-Git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm, 5.7 KiB<br>
Retrieving: perl-Git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./perl-Git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm ..............................................[done]<br>
Retrieving package statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch                      (244/275),  19.0 KiB (  795   B unpacked)<br>
Retrieving delta: ./drpms/statefs-provider-inout-power-0.3.17-1.3.8.jolla_0.3.17_1.3.1.jolla.noarch.drpm, 14.8 KiB<br>
Retrieving: statefs-provider-inout-power-0.3.17-1.3.8.jolla_0.3.17_1.3.1.jolla.noarch.drpm .......................................[done]<br>
Applying delta: ./statefs-provider-inout-power-0.3.17-1.3.8.jolla_0.3.17_1.3.1.jolla.noarch.drpm .................................[done]<br>
Retrieving package statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.i486                        (245/275),  55.6 KiB (147.5 KiB unpacked)<br>
Retrieving: statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.i486.rpm .............................................................[done]<br>
Retrieving package git-1.8.3+mer2-1.1.22.jolla.i486                                            (246/275),   2.1 MiB ( 12.4 MiB unpacked)<br>
Retrieving delta: ./drpms/git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm, 1.4 MiB<br>
Retrieving: git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm ...........................................[done (166.9 KiB/s)]<br>
Applying delta: ./git-1.8.3+mer2-1.2.3.jolla_1.8.3+mer2_1.1.22.jolla.i486.drpm ...................................................[done]<br>
Retrieving package syslinux-4.06+git1-1.1.11.jolla.i486                                        (247/275), 562.2 KiB (  1.8 MiB unpacked)<br>
Retrieving: syslinux-4.06+git1-1.1.11.jolla.i486.rpm ...............................................................[done (274.2 KiB/s)]<br>
Retrieving package platform-sdk-support-0.15.8-1.9.11.jolla.noarch                             (248/275),  24.0 KiB (    7   B unpacked)<br>
Retrieving: platform-sdk-support-0.15.8-1.9.11.jolla.noarch.rpm ..................................................................[done]<br>
Retrieving package connman-configs-sailfish-sdk-0.15.8-1.9.11.jolla.noarch                     (249/275),  24.3 KiB (  233   B unpacked)<br>
Retrieving: connman-configs-sailfish-sdk-0.15.8-1.9.11.jolla.noarch.rpm ..........................................................[done]<br>
Retrieving package sdk-chroot-1.2.53.1-1.16.1.jolla.noarch                                     (250/275),  34.5 KiB ( 16.8 KiB unpacked)<br>
Retrieving: sdk-chroot-1.2.53.1-1.16.1.jolla.noarch.rpm ................................................................[done (154 B/s)]<br>
Retrieving package sdk-sb2-config-1.2.53.1-1.16.1.jolla.noarch                                 (251/275),  27.6 KiB (    0   B unpacked)<br>
Retrieving: sdk-sb2-config-1.2.53.1-1.16.1.jolla.noarch.rpm ......................................................................[done]<br>
Retrieving package mic-0.14+git6-1.3.18.jolla.noarch                                           (252/275), 385.2 KiB (  1.9 MiB unpacked)<br>
Retrieving delta: ./drpms/mic-0.14+git6-1.4.3.jolla_0.14+git6_1.3.18.jolla.noarch.drpm, 65.7 KiB<br>
Retrieving: mic-0.14+git6-1.4.3.jolla_0.14+git6_1.3.18.jolla.noarch.drpm .............................................[done (8.2 KiB/s)]<br>
Applying delta: ./mic-0.14+git6-1.4.3.jolla_0.14+git6_1.3.18.jolla.noarch.drpm ...................................................[done]<br>
Retrieving package connman-1.32+git65-1.25.1.jolla.i486                                        (253/275), 481.3 KiB (  1.3 MiB unpacked)<br>
Retrieving delta: ./drpms/connman-1.32+git61-1.22.1.jolla_1.32+git65_1.25.1.jolla.i486.drpm, 472.2 KiB<br>
Retrieving: connman-1.32+git61-1.22.1.jolla_1.32+git65_1.25.1.jolla.i486.drpm ....................................................[done]<br>
Applying delta: ./connman-1.32+git61-1.22.1.jolla_1.32+git65_1.25.1.jolla.i486.drpm ..............................................[done]<br>
Retrieving package patterns-sailfish-sb2-host-1.0.18-1.10.2.jolla.noarch                       (254/275),  47.4 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/patterns-sailfish-sb2-host-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm, 43.5 KiB<br>
Retrieving: patterns-sailfish-sb2-host-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm .......................................[done]<br>
Applying delta: ./patterns-sailfish-sb2-host-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm .................................[done]<br>
Retrieving package patterns-sailfish-image-creation-tools-1.0.18-1.10.2.jolla.noarch           (255/275),  47.4 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/patterns-sailfish-image-creation-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm, 43.5 KiB<br>
Retrieving: patterns-sailfish-image-creation-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm ...........................[done]<br>
Applying delta: ./patterns-sailfish-image-creation-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm .....................[done]<br>
Retrieving package connman-qt5-1.2.16-1.10.1.jolla.i486                                        (256/275), 146.8 KiB (446.6 KiB unpacked)<br>
Retrieving delta: ./drpms/connman-qt5-1.2.17-1.11.2.jolla_1.2.16_1.10.1.jolla.i486.drpm, 133.0 KiB<br>
Retrieving: connman-qt5-1.2.17-1.11.2.jolla_1.2.16_1.10.1.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./connman-qt5-1.2.17-1.11.2.jolla_1.2.16_1.10.1.jolla.i486.drpm ..................................................[done]<br>
Retrieving package ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.i486                           (257/275),  25.3 KiB (  4.0 KiB unpacked)<br>
Retrieving delta: ./drpms/ssu-network-proxy-plugin-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm, 20.7 KiB<br>
Retrieving: ssu-network-proxy-plugin-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm ...........................................[done]<br>
Applying delta: ./ssu-network-proxy-plugin-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm .....................................[done]<br>
Retrieving package ssu-0.43.12-1.8.1.jolla.i486                                                (258/275), 181.8 KiB (517.4 KiB unpacked)<br>
Retrieving delta: ./drpms/ssu-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm, 165.4 KiB<br>
Retrieving: ssu-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm ................................................................[done]<br>
Applying delta: ./ssu-0.43.12-1.8.3.jolla_0.43.12_1.8.1.jolla.i486.drpm ..........................................................[done]<br>
Retrieving package ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch                              (259/275),  26.6 KiB ( 11.3 KiB unpacked)<br>
Retrieving delta: ./drpms/ssu-vendor-data-jolla-0.107-1.6.1.jolla_0.108_1.6.1.jolla.noarch.drpm, 17.7 KiB<br>
Retrieving: ssu-vendor-data-jolla-0.107-1.6.1.jolla_0.108_1.6.1.jolla.noarch.drpm ....................................[done (2.6 KiB/s)]<br>
Applying delta: ./ssu-vendor-data-jolla-0.107-1.6.1.jolla_0.108_1.6.1.jolla.noarch.drpm ..........................................[done]<br>
Retrieving package sailfish-version-variant-3.0.3-1.11.10.jolla.noarch                         (260/275),  16.5 KiB (   97   B unpacked)<br>
Retrieving delta: ./drpms/sailfish-version-variant-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm, 12.6 KiB<br>
Retrieving: sailfish-version-variant-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm ..........................................[done]<br>
Applying delta: ./sailfish-version-variant-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm ....................................[done]<br>
Retrieving package qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.i486                                (261/275),  72.2 KiB (248.1 KiB unpacked)<br>
Retrieving: qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.i486.rpm .....................................................................[done]<br>
Retrieving package sailfish-version-3.0.3-1.11.10.jolla.noarch                                 (262/275),  17.7 KiB (  2.0 KiB unpacked)<br>
Retrieving delta: ./drpms/sailfish-version-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm, 13.0 KiB<br>
Retrieving: sailfish-version-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm ........................................[done (154 B/s)]<br>
Applying delta: ./sailfish-version-3.0.2-1.10.8.jolla_3.0.3_1.11.10.jolla.noarch.drpm ............................................[done]<br>
Retrieving package obex-capability-0.0.2-1.3.2.jolla.i486                                      (263/275),  17.4 KiB ( 26.5 KiB unpacked)<br>
Retrieving delta: ./drpms/obex-capability-0.0.2-1.3.8.jolla_0.0.2_1.3.2.jolla.i486.drpm, 13.5 KiB<br>
Retrieving: obex-capability-0.0.2-1.3.8.jolla_0.0.2_1.3.2.jolla.i486.drpm ........................................................[done]<br>
Applying delta: ./obex-capability-0.0.2-1.3.8.jolla_0.0.2_1.3.2.jolla.i486.drpm ..................................................[done]<br>
Retrieving package patterns-sailfish-core-1.0.18-1.10.2.jolla.noarch                           (264/275),  47.9 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/patterns-sailfish-core-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm, 44.0 KiB<br>
Retrieving: patterns-sailfish-core-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm ..............................[done (15.6 KiB/s)]<br>
Applying delta: ./patterns-sailfish-core-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm .....................................[done]<br>
Retrieving package obexd-0.48+git17-1.1.15.jolla.i486                                          (265/275),  78.1 KiB (183.2 KiB unpacked)<br>
Retrieving delta: ./drpms/obexd-0.48+git17-1.2.2.jolla_0.48+git17_1.1.15.jolla.i486.drpm, 64.4 KiB<br>
Retrieving: obexd-0.48+git17-1.2.2.jolla_0.48+git17_1.1.15.jolla.i486.drpm .......................................................[done]<br>
Applying delta: ./obexd-0.48+git17-1.2.2.jolla_0.48+git17_1.1.15.jolla.i486.drpm .................................................[done]<br>
Retrieving package kf5bluezqt-bluez4-5.24.0+git15-1.3.1.jolla.i486                             (266/275), 219.7 KiB (835.1 KiB unpacked)<br>
Retrieving: kf5bluezqt-bluez4-5.24.0+git15-1.3.1.jolla.i486.rpm ....................................................[done (153.6 KiB/s)]<br>
Retrieving package sdk-register-0.5-1.3.4.jolla.i486                                           (267/275),  13.5 KiB ( 14.2 KiB unpacked)<br>
Retrieving: sdk-register-0.5-1.3.4.jolla.i486.rpm ................................................................................[done]<br>
Retrieving package sdk-configs-0.114-1.9.4.jolla.noarch                                        (268/275), 122.5 KiB (  3.1 MiB unpacked)<br>
Retrieving: sdk-configs-0.114-1.9.4.jolla.noarch.rpm .............................................................................[done]<br>
Retrieving package sdk-utils-1.2.53.1-1.16.1.jolla.noarch                                      (269/275),  86.2 KiB (296.6 KiB unpacked)<br>
Retrieving: sdk-utils-1.2.53.1-1.16.1.jolla.noarch.rpm ...........................................................................[done]<br>
Retrieving package feature-jolla-sdk-0.1.3-1.2.10.jolla.i486                                   (270/275),   8.1 KiB (  501   B unpacked)<br>
Retrieving delta: ./drpms/feature-jolla-sdk-0.1.3-1.3.2.jolla_0.1.3_1.2.10.jolla.i486.drpm, 3.9 KiB<br>
Retrieving: feature-jolla-sdk-0.1.3-1.3.2.jolla_0.1.3_1.2.10.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./feature-jolla-sdk-0.1.3-1.3.2.jolla_0.1.3_1.2.10.jolla.i486.drpm ...............................................[done]<br>
Retrieving package vm-configs-0.15.8-1.9.11.jolla.noarch                                       (271/275),  26.7 KiB (  1.0 KiB unpacked)<br>
Retrieving: vm-configs-0.15.8-1.9.11.jolla.noarch.rpm ............................................................................[done]<br>
Retrieving package patterns-sailfish-packaging-tools-1.0.18-1.10.2.jolla.noarch                (272/275),  47.3 KiB (    0   B unpacked)<br>
Retrieving delta: ./drpms/patterns-sailfish-packaging-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm, 43.4 KiB<br>
Retrieving: patterns-sailfish-packaging-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm ................................[done]<br>
Applying delta: ./patterns-sailfish-packaging-tools-1.0.17-1.10.2.jolla_1.0.18_1.10.2.jolla.noarch.drpm ..........................[done]<br>
Retrieving package bluez-4.101+git76-1.2.12.jolla.i486                                         (273/275), 554.3 KiB (  2.2 MiB unpacked)<br>
Retrieving delta: ./drpms/bluez-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm, 556.0 KiB<br>
Retrieving: bluez-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm .....................................................[done]<br>
Applying delta: ./bluez-4.101+git76-1.3.1.jolla_4.101+git76_1.2.12.jolla.i486.drpm ...............................................[done]<br>
Retrieving package patterns-sailfish-platform-sdk-0.15.8-1.9.11.jolla.noarch                   (274/275),  23.7 KiB (    0   B unpacked)<br>
Retrieving: patterns-sailfish-platform-sdk-0.15.8-1.9.11.jolla.noarch.rpm ........................................................[done]<br>
Retrieving package patterns-sailfish-configuration-platform-sdk-chroot-0.15.8-1.9.11.jolla.noarch<br>
                                                                                               (275/275),  23.5 KiB (    0   B unpacked)<br>
Retrieving: patterns-sailfish-configuration-platform-sdk-chroot-0.15.8-1.9.11.jolla.noarch.rpm ...................................[done]<br>
Checking for file conflicts: .....................................................................................................[done]<br>
(  1/275) Installing: ofono-configs-build-engine-0.15.8-1.9.11.jolla.noarch ......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/ofono-configs-build-engine-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  2/275) Installing: fontpackages-filesystem-1.44-1.1.8.jolla.noarch ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/fontpackages-filesystem-1.44-1.1.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  3/275) Installing: jolla-ca-0.9-1.2.9.jolla.noarch ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/jolla-ca-0.9-1.2.9.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  4/275) Installing: libgcc-4.9.4-1.2.4.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgcc-4.9.4-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  5/275) Installing: mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch ...........................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  6/275) Installing: ncurses-base-6.1+git1-1.3.4.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/ncurses-base-6.1+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  7/275) Installing: rootfiles-8.1+git2-1.2.7.jolla.noarch ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/rootfiles-8.1+git2-1.2.7.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  8/275) Installing: sailfish-ca-0.1.1-1.2.9.jolla.noarch .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-ca-0.1.1-1.2.9.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(  9/275) Installing: setup-2.8.56-1.2.4.jolla.noarch ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/setup-2.8.56-1.2.4.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: /etc/shadow created as /etc/shadow.rpmnew<br>
<br>
<br>
( 10/275) Installing: tzdata-2017b-1.1.8.jolla.noarch ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/tzdata-2017b-1.1.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 11/275) Installing: vim-filesystem-7.3.629-1.2.3.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/vim-filesystem-7.3.629-1.2.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 12/275) Installing: wireless-regdb-2016.06.10+git1-1.2.6.jolla.noarch ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/wireless-regdb-2016.06.10+git1-1.2.6.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 13/275) Installing: filesystem-3.1-1.1.8.jolla.noarch ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/filesystem-3.1-1.1.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 14/275) Installing: glibc-2.25+git5-1.4.1.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/glibc-2.25+git5-1.4.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 15/275) Installing: basesystem-11+git1-1.2.7.jolla.noarch ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/basesystem-11+git1-1.2.7.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 16/275) Installing: ncurses-libs-6.1+git1-1.3.4.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/ncurses-libs-6.1+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 17/275) Installing: bash-1:3.2.57-1.2.4.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bash-3.2.57-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 18/275) Installing: glibc-common-2.25+git5-1.4.1.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/glibc-common-2.25+git5-1.4.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 19/275) Installing: zlib-1.2.11+git1-1.4.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/zlib-1.2.11+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 20/275) Installing: zip-3.0-1.1.10.jolla.i486 ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/zip-3.0-1.1.10.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 21/275) Installing: xz-libs-5.0.4-1.2.6.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/xz-libs-5.0.4-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 22/275) Installing: xdg-user-dirs-0.16+git1-1.2.7.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/xdg-user-dirs-0.16+git1-1.2.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 23/275) Installing: vim-common-7.3.629-1.2.3.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/vim-common-7.3.629-1.2.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 24/275) Installing: unzip-6.0-1.2.4.jolla.i486 .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/unzip-6.0-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 25/275) Installing: tar-1.17-1.2.4.jolla.i486 ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/tar-1.17-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 26/275) Installing: strace-4.22+git1-1.2.18.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/strace-4.22+git1-1.2.18.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 27/275) Installing: shadow-utils-4.6-1.2.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/shadow-utils-4.6-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 28/275) Installing: sed-1:4.1.5-1.2.5.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/sed-4.1.5-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 29/275) Installing: readline-5.2-1.2.6.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/readline-5.2-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 30/275) Installing: pth-2.0.7-1.1.8.jolla.i486 .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pth-2.0.7-1.1.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 31/275) Installing: psmisc-22.13-1.3.6.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/psmisc-22.13-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 32/275) Installing: procps-3.2.8-1.2.6.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/procps-3.2.8-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 33/275) Installing: pptp-1.8.0+git4-1.1.14.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/pptp-1.8.0+git4-1.1.14.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 34/275) Installing: popt-1.16-1.1.9.jolla.i486 .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/popt-1.16-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 35/275) Installing: pkgconfig-0.27.1-1.2.6.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pkgconfig-0.27.1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 36/275) Installing: patch-2.7.5+git1-1.1.8.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/patch-2.7.5+git1-1.1.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 37/275) Installing: nspr-4.20.0-1.3.6.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/nspr-4.20.0-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 38/275) Installing: net-tools-1.60-1.2.4.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/net-tools-1.60-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 39/275) Installing: ncurses-6.1+git1-1.3.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/ncurses-6.1+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 40/275) Installing: lzo-2.09-1.1.9.jolla.i486 ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/lzo-2.09-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 41/275) Installing: lsof-4.91+git1-1.3.3.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/lsof-4.91+git1-1.3.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 42/275) Installing: libtasn1-4.13+git1-1.3.6.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libtasn1-4.13+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 43/275) Installing: libstdc++-4.9.4-1.2.4.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libstdc++-4.9.4-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 44/275) Installing: libshadowutils-0.0.2-1.3.6.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/libshadowutils-0.0.2-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 45/275) Installing: libsb2-2.3.90+git17-1.5.3.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libsb2-2.3.90+git17-1.5.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 46/275) Installing: libpcap-1.8.1+git2-1.2.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libpcap-1.8.1+git2-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 47/275) Installing: libnl-3.4.0-1.2.6.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libnl-3.4.0-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 48/275) Installing: liblua-5.1.5-1.1.11.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/liblua-5.1.5-1.1.11.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 49/275) Installing: libiphb-1.2.5+git1-1.3.6.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libiphb-1.2.5+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 50/275) Installing: libgpg-error-1.27+git2-1.3.4.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgpg-error-1.27+git2-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 51/275) Installing: libffi-3.2.1+git1-1.2.9.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libffi-3.2.1+git1-1.2.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 52/275) Installing: libcom_err-1.45.0+git1-1.3.4.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libcom_err-1.45.0+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 53/275) Installing: libcap-2.24+git1-1.3.6.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libcap-2.24+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 54/275) Installing: libattr-2.4.47+git1-1.3.6.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libattr-2.4.47+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 55/275) Installing: less-436+mer1-1.1.24.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/less-436+mer1-1.1.24.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 56/275) Installing: json-c-0.12-1.1.8.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/json-c-0.12-1.1.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 57/275) Installing: iptables-1.8.2+git1-1.4.2.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/iptables-1.8.2+git1-1.4.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 58/275) Installing: iproute-3.7.0+git4-1.2.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/iproute-3.7.0+git4-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 59/275) Installing: gdbm-1.8.3-1.2.6.jolla.i486 ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/gdbm-1.8.3-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 60/275) Installing: freetype-2.8.0-1.1.8.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/freetype-2.8.0-1.1.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 61/275) Installing: findutils-4.2.31-1.2.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/findutils-4.2.31-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 62/275) Installing: expat-2.1.0-1.1.9.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/expat-2.1.0-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 63/275) Installing: dosfstools-3.0.10+git1-1.1.25.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/dosfstools-3.0.10+git1-1.1.25.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 64/275) Installing: diffutils-2.8.1-1.2.5.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/diffutils-2.8.1-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 65/275) Installing: db4-4.8.30-1.3.8.jolla.i486 ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/db4-4.8.30-1.3.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 66/275) Installing: cpio-2.12+git1-1.3.3.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/cpio-2.12+git1-1.3.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 67/275) Installing: bzip2-libs-1.0.6-1.2.6.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bzip2-libs-1.0.6-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 68/275) Installing: busybox-1.29.3+git5-1.1.4.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/busybox-1.29.3+git5-1.1.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 69/275) Installing: bluez-libs-4.101+git76-1.2.12.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bluez-libs-4.101+git76-1.2.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 70/275) Installing: libxml2-2.9.8+git2-1.3.6.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libxml2-2.9.8+git2-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 71/275) Installing: info-4.13a-1.2.6.jolla.i486 ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/info-4.13a-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: /usr/share/info/dir saved as /usr/share/info/dir.rpmsave<br>
<br>
<br>
( 72/275) Installing: file-libs-5.35+git2-1.2.6.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/file-libs-5.35+git2-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 73/275) Installing: elfutils-libelf-0.170+git1-1.3.6.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/elfutils-libelf-0.170+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 74/275) Installing: binutils-2.25-1.3.6.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/binutils-2.25-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 75/275) Installing: xz-5.0.4-1.2.6.jolla.i486 ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/xz-5.0.4-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 76/275) Installing: libutempter-1.1.5+git1-1.1.9.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libutempter-1.1.5+git1-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 77/275) Installing: pcre-8.42+git1-1.3.4.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pcre-8.42+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 78/275) Installing: p7zip-full-16.02+git1-1.2.1.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/p7zip-full-16.02+git1-1.2.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 79/275) Installing: libusb-0.1.12-1.2.4.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libusb-0.1.12-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 80/275) Installing: boost-system-1.66.0-1.3.8.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/boost-system-1.66.0-1.3.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 81/275) Installing: ppp-2.4.7+git3-1.1.15.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/ppp-2.4.7+git3-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 82/275) Installing: iw-4.1+git2-1.2.4.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/iw-4.1+git2-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 83/275) Installing: libksba-1.3.5+git2-1.2.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libksba-1.3.5+git2-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 84/275) Installing: libgcrypt-1.5.6+git1-1.1.11.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgcrypt-1.5.6+git1-1.1.11.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 85/275) Installing: p11-kit-0.23.12+git1-1.3.4.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/p11-kit-0.23.12+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 86/275) Installing: libss-1.45.0+git1-1.3.4.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libss-1.45.0+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 87/275) Installing: e2fsprogs-libs-1.45.0+git1-1.3.4.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/e2fsprogs-libs-1.45.0+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 88/275) Installing: libacl-2.2.53-1.2.6.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libacl-2.2.53-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 89/275) Installing: iptables-ipv6-1.8.2+git1-1.4.2.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/iptables-ipv6-1.8.2+git1-1.4.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 90/275) Installing: db4-utils-4.8.30-1.3.8.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/db4-utils-4.8.30-1.3.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 91/275) Installing: bzip2-1.0.6-1.2.6.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bzip2-1.0.6-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 92/275) Installing: busybox-symlinks-gzip-1.29.3+git5-1.1.4.jolla.i486 .........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/busybox-symlinks-gzip-1.29.3+git5-1.1.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 93/275) Installing: libxslt-1.1.29-1.2.6.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libxslt-1.1.29-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 94/275) Installing: augeas-libs-1.6.0+git1-1.2.5.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/augeas-libs-1.6.0+git1-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 95/275) Installing: mtools-4.0.12+mer1-1.1.24.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/mtools-4.0.12+mer1-1.1.24.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 96/275) Installing: file-5.35+git2-1.2.6.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/file-5.35+git2-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 97/275) Installing: xz-lzma-compat-5.0.4-1.2.6.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/xz-lzma-compat-5.0.4-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 98/275) Installing: squashfs-tools-4.3.0-1.1.9.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/squashfs-tools-4.3.0-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
( 99/275) Installing: kmod-libs-21-1.2.7.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/kmod-libs-21-1.2.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(100/275) Installing: kmod-21-1.2.7.jolla.i486 ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/kmod-21-1.2.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(101/275) Installing: elfutils-libs-0.170+git1-1.3.6.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/elfutils-libs-0.170+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(102/275) Installing: grep-1:2.5.1a-1.2.4.jolla.i486 .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/grep-2.5.1a-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(103/275) Installing: glib2-2.56.1+git3-1.3.6.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/glib2-2.56.1+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(104/275) Installing: boost-thread-1.66.0-1.3.8.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/boost-thread-1.66.0-1.3.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(105/275) Installing: boost-filesystem-1.66.0-1.3.8.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/boost-filesystem-1.66.0-1.3.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(106/275) Installing: ppp-libs-2.4.7+git3-1.1.15.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/ppp-libs-2.4.7+git3-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(107/275) Installing: vpnc-0.5.3-1.2.4.jolla.i486 ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/vpnc-0.5.3-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(108/275) Installing: rsync-3.1.0+git2-1.3.5.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/oss/i486/rsync-3.1.0+git2-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(109/275) Installing: coreutils-1:6.9-1.2.5.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/coreutils-6.9-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(110/275) Installing: hwdata-0.291+git1-1.1.8.jolla.noarch .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/hwdata-0.291+git1-1.1.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(111/275) Installing: elfutils-0.170+git1-1.3.6.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/elfutils-0.170+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(112/275) Installing: shared-mime-info-1.12-1.4.4.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/shared-mime-info-1.12-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(113/275) Installing: libwspcodec-2.2.1-1.2.8.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libwspcodec-2.2.1-1.2.8.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(114/275) Installing: libglibutil-1.0.35-1.8.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libglibutil-1.0.35-1.8.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(115/275) Installing: desktop-file-utils-0.23+git1-1.2.10.jolla.i486 .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/desktop-file-utils-0.23+git1-1.2.10.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(116/275) Installing: xl2tpd-1.3.8+git3-1.1.17.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/xl2tpd-1.3.8+git3-1.1.17.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(117/275) Installing: pam-1.1.8+git5-1.2.4.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pam-1.1.8+git5-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(118/275) Installing: libmce-glib-1.0.5-1.1.15.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libmce-glib-1.0.5-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(119/275) Installing: libgsupplicant-1.0.11-1.4.7.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgsupplicant-1.0.11-1.4.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(120/275) Installing: libgrilio-1.0.29-1.9.1.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgrilio-1.0.29-1.9.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(121/275) Installing: libgofono-2.0.6-1.2.12.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgofono-2.0.6-1.2.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(122/275) Installing: libdbusaccess-1.0.7-1.3.4.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libdbusaccess-1.0.7-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(123/275) Installing: systemd-libs-225+git13-1.4.4.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/systemd-libs-225+git13-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(124/275) Installing: sudo-1.8.20p2-1.1.25.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/sudo-1.8.20p2-1.1.25.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(125/275) Installing: p11-kit-trust-0.23.12+git1-1.3.4.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/p11-kit-trust-0.23.12+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(126/275) Installing: libuser-0.62+git1-1.2.6.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libuser-0.62+git1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(127/275) Installing: libsmartcols-2.33+git1-1.4.4.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libsmartcols-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(128/275) Installing: kbd-2.0.4-1.3.3.jolla.i486 .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/kbd-2.0.4-1.3.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(129/275) Installing: groff-1.18.1.4-1.1.13.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/groff-1.18.1.4-1.1.13.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(130/275) Installing: gawk-1:3.1.5-1.2.6.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/gawk-3.1.5-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(131/275) Installing: fontconfig-2.12.4-1.2.10.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/fontconfig-2.12.4-1.2.10.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(132/275) Installing: libgofonoext-1.0.10-1.2.11.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libgofonoext-1.0.10-1.2.11.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(133/275) Installing: dbus-libs-1.10.8+git1-1.1.12.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/dbus-libs-1.10.8+git1-1.1.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(134/275) Installing: cor-0.1.18-1.1.16.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/cor-0.1.18-1.1.16.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(135/275) Installing: p11-kit-nss-ckbi-0.23.12+git1-1.3.4.jolla.i486 .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/p11-kit-nss-ckbi-0.23.12+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(136/275) Installing: passwd-0.79+git1-1.3.6.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/passwd-0.79+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(137/275) Installing: libuuid-2.33+git1-1.4.4.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libuuid-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(138/275) Installing: perl-macros-2:5.16.1-1.1.15.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-macros-5.16.1-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(139/275) Installing: libdbuslogserver-dbus-1.0.15-1.4.7.jolla.i486 ..............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libdbuslogserver-dbus-1.0.15-1.4.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(140/275) Installing: statefs-pp-0.3.35-1.1.19.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/statefs-pp-0.3.35-1.1.19.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(141/275) Installing: gnutls-2.12.23.4-1.2.5.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/gnutls-2.12.23.4-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(142/275) Installing: ca-certificates-2018.2.24-1.2.11.jolla.noarch ..............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/ca-certificates-2018.2.24-1.2.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(143/275) Installing: libblkid-2.33+git1-1.4.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libblkid-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(144/275) Installing: perl-libs-2:5.16.1-1.1.15.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-libs-5.16.1-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(145/275) Installing: openssl-libs-1.0.2o+git2-1.4.4.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/openssl-libs-1.0.2o+git2-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(146/275) Installing: libfdisk-2.33+git1-1.4.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libfdisk-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(147/275) Installing: perl-2:5.16.1-1.1.15.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-5.16.1-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(148/275) Installing: libcurl-7.64.0+git1-1.8.4.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libcurl-7.64.0+git1-1.8.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(149/275) Installing: libarchive-3.3.3+git1-1.2.6.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libarchive-3.3.3+git1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(150/275) Installing: libmount-2.33+git1-1.4.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libmount-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(151/275) Installing: perl-Scalar-List-Utils-2:1.25-1.1.15.jolla.i486 ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-Scalar-List-Utils-1.25-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(152/275) Installing: qemu-usermode-common-2.1.0-1.2.6.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/qemu-usermode-common-2.1.0-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(153/275) Installing: pacrunner-0.15+git1-1.3.6.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pacrunner-0.15+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(154/275) Installing: gnupg2-1:2.0.4+git2-1.4.4.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/gnupg2-2.0.4+git2-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(155/275) Installing: curl-7.64.0+git1-1.8.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/curl-7.64.0+git1-1.8.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(156/275) Installing: bsdtar-3.3.3+git1-1.2.6.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bsdtar-3.3.3+git1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(157/275) Installing: util-linux-2.33+git1-1.4.4.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/util-linux-2.33+git1-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(158/275) Installing: perl-Pod-Escapes-2:1.04-1.1.15.jolla.noarch ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Escapes-1.04-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(159/275) Installing: qemu-usermode-static-2.1.0-1.2.6.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/qemu-usermode-static-2.1.0-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(160/275) Installing: qemu-usermode-2.1.0-1.2.6.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/qemu-usermode-2.1.0-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(161/275) Installing: gpgme-1.2.0+git6-1.3.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/gpgme-1.2.0+git6-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(162/275) Installing: rpm-libs-4.14.1+git9-1.5.6.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/rpm-libs-4.14.1+git9-1.5.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(163/275) Installing: xdg-utils-1.1.2+git1-1.2.7.jolla.noarch ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/xdg-utils-1.1.2+git1-1.2.7.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(164/275) Installing: vim-enhanced-7.3.629-1.2.3.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/vim-enhanced-7.3.629-1.2.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(165/275) Installing: parted-3.0+mer4-1.2.12.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/parted-3.0+mer4-1.2.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(166/275) Installing: fuse-2.9.0+git1-1.2.6.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/fuse-2.9.0+git1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(167/275) Installing: fakeroot-1.12.4-1.1.9.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/fakeroot-1.12.4-1.1.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(168/275) Installing: e2fsprogs-1.45.0+git1-1.3.4.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/e2fsprogs-1.45.0+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(169/275) Installing: btrfs-progs-3.16+git2-1.4.2.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/btrfs-progs-3.16+git2-1.4.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(170/275) Installing: perl-Pod-Simple-2:3.20-1.1.15.jolla.noarch .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Simple-3.20-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(171/275) Installing: rpm-4.14.1+git9-1.5.6.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/rpm-4.14.1+git9-1.5.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(172/275) Installing: fuse-libs-2.9.0+git1-1.2.6.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/fuse-libs-2.9.0+git1-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(173/275) Installing: perl-parent-2:0.225-1.1.15.jolla.noarch ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-parent-0.225-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(174/275) Installing: libicu-63.1+git5-1.1.5.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libicu-63.1+git5-1.1.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(175/275) Installing: perl-Pod-Perldoc-2:3.17.00-1.1.15.jolla.noarch .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Perldoc-3.17.00-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(176/275) Installing: sqlite-libs-3.13.0+git3-1.3.5.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/sqlite-libs-3.13.0+git3-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(177/275) Installing: qt5-qtcore-5.6.3+git9-1.9.1.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/i486/qt5-qtcore-5.6.3+git9-1.9.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(178/275) Installing: perl-Pod-Parser-2:1.51-1.1.15.jolla.noarch .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Parser-1.51-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(179/275) Installing: nss-3.39-1.3.5.jolla.i486 ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/nss-3.39-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(180/275) Installing: qt5-qtdbus-5.6.3+git9-1.9.1.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/i486/qt5-qtdbus-5.6.3+git9-1.9.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(181/275) Installing: perl-Filter-2:1.40-1.1.15.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-Filter-1.40-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(182/275) Installing: python-2.7.15+git2-1.3.5.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-2.7.15+git2-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(183/275) Installing: openvpn-2.4.5+git2-1.3.5.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/openvpn-2.4.5+git2-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(184/275) Installing: openconnect-7.08+git1-1.2.4.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/openconnect-7.08+git1-1.2.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(185/275) Installing: nss-sysinit-3.39-1.3.5.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/nss-sysinit-3.39-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(186/275) Installing: libsolv0-0.6.35+git2-1.4.6.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libsolv0-0.6.35+git2-1.4.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(187/275) Installing: deltarpm-3.5-1.2.6.jolla.i486 ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/deltarpm-3.5-1.2.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(188/275) Installing: cryptsetup-libs-2.1.0+git1-1.3.4.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/cryptsetup-libs-2.1.0+git1-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(189/275) Installing: createrepo_c-libs-0.10.0+git1-1.1.47.jolla.i486 ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/createrepo_c-libs-0.10.0+git1-1.1.47.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(190/275) Installing: qt5-qtnetwork-5.6.3+git9-1.9.1.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/i486/qt5-qtnetwork-5.6.3+git9-1.9.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(191/275) Installing: perl-Module-Pluggable-2:4.00-1.1.15.jolla.noarch ...........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Module-Pluggable-4.00-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(192/275) Installing: python-libs-2.7.15+git2-1.3.5.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-libs-2.7.15+git2-1.3.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(193/275) Installing: libsolv-tools-0.6.35+git2-1.4.6.jolla.i486 .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libsolv-tools-0.6.35+git2-1.4.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(194/275) Installing: systemd-config-sailfish-0.8.15-1.7.2.jolla.noarch ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/systemd-config-sailfish-0.8.15-1.7.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(195/275) Installing: createrepo_c-0.10.0+git1-1.1.47.jolla.i486 .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/createrepo_c-0.10.0+git1-1.1.47.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(196/275) Installing: qt5-qtxml-5.6.3+git9-1.9.1.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/i486/qt5-qtxml-5.6.3+git9-1.9.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(197/275) Installing: libqtaround2-0.2.8-1.2.1.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/libqtaround2-0.2.8-1.2.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(198/275) Installing: perl-threads-2:1.86-1.1.15.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-threads-1.86-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(199/275) Installing: rpm-python-4.14.1+git9-1.4.4.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/rpm-python-4.14.1+git9-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(200/275) Installing: python-yaml-3.10-1.1.10.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-yaml-3.10-1.1.10.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(201/275) Installing: python-setuptools-0.6c11-1.1.10.jolla.noarch ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/python-setuptools-0.6c11-1.1.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(202/275) Installing: python-pycurl-7.19.0+mer1-1.1.45.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/python-pycurl-7.19.0+mer1-1.1.45.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(203/275) Installing: python-lxml-3.2.0-1.1.12.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-lxml-3.2.0-1.1.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(204/275) Installing: python-cheetah-2.4.4+mer1-1.1.31.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/python-cheetah-2.4.4+mer1-1.1.31.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(205/275) Installing: python-M2Crypto-0.23.0-1.1.11.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-M2Crypto-0.23.0-1.1.11.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(206/275) Installing: pacrunner-python-0.15+git1-1.3.6.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/pacrunner-python-0.15+git1-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(207/275) Installing: libzypp-17.3.1+git4-1.6.2.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/libzypp-17.3.1+git4-1.6.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(208/275) Installing: device-mapper-libs-2.02.177+git3-1.3.6.jolla.i486 ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/device-mapper-libs-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(209/275) Installing: statefs-qt5-0.3.5-1.4.1.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/statefs-qt5-0.3.5-1.4.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(210/275) Installing: perl-Socket-2:2.001-1.1.15.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-Socket-2.001-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(211/275) Installing: python-distro-1.0.4+mer1-1.1.10.jolla.noarch ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/python-distro-1.0.4+mer1-1.1.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(212/275) Installing: repomd-pattern-builder-0.3.3-1.2.7.jolla.i486 ..............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/repomd-pattern-builder-0.3.3-1.2.7.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(213/275) Installing: python-urlgrabber-3.9.1+mer1-1.1.26.jolla.noarch ...........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/noarch/python-urlgrabber-3.9.1+mer1-1.1.26.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(214/275) Installing: zypper-1.14.6+git3-1.3.6.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/zypper-1.14.6+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(215/275) Installing: python-zypp-0.7.4+git3-1.2.9.jolla.i486 ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/python-zypp-0.7.4+git3-1.2.9.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(216/275) Installing: device-mapper-event-libs-2.02.177+git3-1.3.6.jolla.i486 ....................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/device-mapper-event-libs-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(217/275) Installing: perl-threads-shared-2:1.40-1.1.15.jolla.i486 ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/perl-threads-shared-1.40-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(218/275) Installing: osc-0.146.0-1.1.33.jolla.noarch ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/noarch/osc-0.146.0-1.1.33.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(219/275) Installing: lvm2-libs-2.02.177+git3-1.3.6.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/lvm2-libs-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(220/275) Installing: spectacle-0.30-1.1.24.jolla.noarch .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/noarch/spectacle-0.30-1.1.24.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(221/275) Installing: scratchbox2-2.3.90+git17-1.5.3.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/scratchbox2-2.3.90+git17-1.5.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(222/275) Installing: rpm-build-4.14.1+git9-1.5.6.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/rpm-build-4.14.1+git9-1.5.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(223/275) Installing: perl-Error-0.17020+mer2-1.1.15.jolla.noarch ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/noarch/perl-Error-0.17020+mer2-1.1.15.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(224/275) Installing: android-tools-hadk-5.1.1+git2-1.1.30.jolla.i486 ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/android-tools-hadk-5.1.1+git2-1.1.30.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(225/275) Installing: lvm2-2.02.177+git3-1.3.6.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/lvm2-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(226/275) Installing: systemd-225+git13-1.4.4.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/systemd-225+git13-1.4.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(227/275) Installing: dbus-1.10.8+git1-1.1.12.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/dbus-1.10.8+git1-1.1.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
<br>
<br>
(228/275) Installing: polkit-0.105+git2-1.2.5.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/polkit-0.105+git2-1.2.5.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(229/275) Installing: device-mapper-2.02.177+git3-1.3.6.jolla.i486 ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/device-mapper-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(230/275) Installing: PackageKit-glib-1.1.9+git5-1.7.2.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/PackageKit-glib-1.1.9+git5-1.7.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(231/275) Installing: device-mapper-event-2.02.177+git3-1.3.6.jolla.i486 .........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/device-mapper-event-2.02.177+git3-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(232/275) Installing: PackageKit-1.1.9+git5-1.7.2.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/PackageKit-1.1.9+git5-1.7.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(233/275) Installing: wpa_supplicant-2.6+git5-1.3.6.jolla.i486 ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/wpa_supplicant-2.6+git5-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(234/275) Installing: systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch ......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(235/275) Installing: openssh-7.9p1+git2-1.5.4.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/openssh-7.9p1+git2-1.5.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(236/275) Installing: ofono-1.21+git44-1.19.1.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/ofono-1.21+git44-1.19.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: user radio does not exist - using root<br>
warning: group radio does not exist - using root<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
<br>
<br>
(237/275) Installing: kpartx-0.4.9+mer1-1.1.41.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/kpartx-0.4.9+mer1-1.1.41.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(238/275) Installing: crda-4.14+git2-1.3.6.jolla.i486 ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/crda-4.14+git2-1.3.6.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(239/275) Installing: PackageKit-zypp-1.1.9+git5-1.7.2.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/PackageKit-zypp-1.1.9+git5-1.7.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(240/275) Installing: oneshot-0.4.8-1.2.8.jolla.noarch ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/oneshot-0.4.8-1.2.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(241/275) Installing: openssh-clients-7.9p1+git2-1.5.4.jolla.i486 ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/openssh-clients-7.9p1+git2-1.5.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(242/275) Installing: statefs-0.3.35-1.1.19.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/statefs-0.3.35-1.1.19.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
WARNING: there is no privileged group, failed<br>
<br>
<br>
(243/275) Installing: perl-Git-1.8.3+mer2-1.1.22.jolla.i486 ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/perl-Git-1.8.3+mer2-1.1.22.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(244/275) Installing: statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch .....................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/noarch/statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
unregister<br>
Register inout_power<br>
Trying to dump inout provider "/usr/share/statefs/inout_power.conf"<br>
Can't find inout loader<br>
Can't retrieve information from /usr/share/statefs/inout_power.conf<br>
add-oneshot: /usr/lib/oneshot.d/statefs-03-register-inout_power - run OK<br>
<br>
<br>
(245/275) Installing: statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.i486 .......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(246/275) Installing: git-1.8.3+mer2-1.1.22.jolla.i486 ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/git-1.8.3+mer2-1.1.22.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(247/275) Installing: syslinux-4.06+git1-1.1.11.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/i486/syslinux-4.06+git1-1.1.11.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(248/275) Installing: platform-sdk-support-0.15.8-1.9.11.jolla.noarch ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/platform-sdk-support-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(249/275) Installing: connman-configs-sailfish-sdk-0.15.8-1.9.11.jolla.noarch ....................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/connman-configs-sailfish-sdk-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(250/275) Installing: sdk-chroot-1.2.53.1-1.16.1.jolla.noarch ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-chroot-1.2.53.1-1.16.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(251/275) Installing: sdk-sb2-config-1.2.53.1-1.16.1.jolla.noarch ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-sb2-config-1.2.53.1-1.16.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(252/275) Installing: mic-0.14+git6-1.3.18.jolla.noarch ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/noarch/mic-0.14+git6-1.3.18.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(253/275) Installing: connman-1.32+git65-1.25.1.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/connman-1.32+git65-1.25.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
find: /var/lib/connman: No such file or directory<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
<br>
<br>
(254/275) Installing: patterns-sailfish-sb2-host-1.0.18-1.10.2.jolla.noarch ......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-sb2-host-1.0.18-1.10.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(255/275) Installing: patterns-sailfish-image-creation-tools-1.0.18-1.10.2.jolla.noarch ..........................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-image-creation-tools-1.0.18-1.10.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(256/275) Installing: connman-qt5-1.2.16-1.10.1.jolla.i486 .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/connman-qt5-1.2.16-1.10.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(257/275) Installing: ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.i486 ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(258/275) Installing: ssu-0.43.12-1.8.1.jolla.i486 ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/ssu-0.43.12-1.8.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(259/275) Installing: ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/oss/noarch/ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
DBus unavailable, falling back to libssu<br>
add-oneshot: /usr/lib/oneshot.d/ssu-update-repos - run OK<br>
<br>
<br>
(260/275) Installing: sailfish-version-variant-3.0.3-1.11.10.jolla.noarch ........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-version-variant-3.0.3-1.11.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(261/275) Installing: qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.i486 ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(262/275) Installing: sailfish-version-3.0.3-1.11.10.jolla.noarch ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-version-3.0.3-1.11.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(263/275) Installing: obex-capability-0.0.2-1.3.2.jolla.i486 .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/obex-capability-0.0.2-1.3.2.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(264/275) Installing: patterns-sailfish-core-1.0.18-1.10.2.jolla.noarch ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-core-1.0.18-1.10.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(265/275) Installing: obexd-0.48+git17-1.1.15.jolla.i486 .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/obexd-0.48+git17-1.1.15.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(266/275) Installing: kf5bluezqt-bluez4-5.24.0+git15-1.3.1.jolla.i486 ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/i486/kf5bluezqt-bluez4-5.24.0+git15-1.3.1.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(267/275) Installing: sdk-register-0.5-1.3.4.jolla.i486 ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/i486/sdk-register-0.5-1.3.4.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(268/275) Installing: sdk-configs-0.114-1.9.4.jolla.noarch .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-configs-0.114-1.9.4.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(269/275) Installing: sdk-utils-1.2.53.1-1.16.1.jolla.noarch .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-utils-1.2.53.1-1.16.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(270/275) Installing: feature-jolla-sdk-0.1.3-1.2.10.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/customer-jolla/i486/feature-jolla-sdk-0.1.3-1.2.10.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(271/275) Installing: vm-configs-0.15.8-1.9.11.jolla.noarch ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/vm-configs-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
DBus unavailable, falling back to libssu<br>
add-oneshot: /usr/lib/oneshot.d/ssu-update-repos - run OK<br>
<br>
<br>
(272/275) Installing: patterns-sailfish-packaging-tools-1.0.18-1.10.2.jolla.noarch ...............................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-packaging-tools-1.0.18-1.10.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(273/275) Installing: bluez-4.101+git76-1.2.12.jolla.i486 ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/i486/bluez-4.101+git76-1.2.12.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
Running in chroot, ignoring request.<br>
<br>
<br>
(274/275) Installing: patterns-sailfish-platform-sdk-0.15.8-1.9.11.jolla.noarch ..................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/patterns-sailfish-platform-sdk-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
(275/275) Installing: patterns-sailfish-configuration-platform-sdk-chroot-0.15.8-1.9.11.jolla.noarch .............................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/adaptation0/noarch/patterns-sailfish-configuration-platform-sdk-chroot-0.15.8-1.9.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Executing %posttrans scripts .....................................................................................................[done]<br>
There are some running programs that might use files deleted by recent upgrade. You may wish to check and restart some of them. Run 'zypper ps -s' to list these programs<br>
</details>

```console
PlatformSDK:~$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R ssu re 3.0.3.10
```
<details>
Changing release from 2.2.1.18 to 3.0.3.10<br>
Your device is now in release mode!<br>
DBus unavailable, falling back to libssu<br>
</details><br>

```console
PlatformSDK:~/hadk$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper ref
```
<details>
Retrieving repository 'hotfixes' metadata --------------------------------------------------------------------------------------------[\]<br>
Download (curl) error for 'https://releases.jolla.com/releases/3.0.3.10/hotfixes/armv7hl/repodata/repomd.xml':<br>
Error code: Connection failed<br>
Error message: Could not resolve host: releases.jolla.com<br>
<br>
Abort, retry, ignore? [a/r/i/?] (a): a<br>
Retrieving repository 'hotfixes' metadata ........................................................................................[error]<br>
Repository 'hotfixes' is invalid.<br>
[|] Valid metadata not found at specified URL(s)<br>
Please check if the URIs defined for this repository are pointing to a valid repository.<br>
Skipping repository 'hotfixes' because of the above error.<br>
Retrieving repository 'jolla' metadata -----------------------------------------------------------------------------------------------[|]<br>
Download (curl) error for 'https://releases.jolla.com/releases/3.0.3.10/jolla/armv7hl/repodata/repomd.xml':<br>
Error code: Connection failed<br>
Error message: Could not resolve host: releases.jolla.com<br>
<br>
Abort, retry, ignore? [a/r/i/?] (a): a<br>
Retrieving repository 'jolla' metadata ...........................................................................................[error]<br>
Repository 'jolla' is invalid.<br>
[|] Valid metadata not found at specified URL(s)<br>
Please check if the URIs defined for this repository are pointing to a valid repository.<br>
Skipping repository 'jolla' because of the above error.<br>
Retrieving repository 'sdk' metadata -------------------------------------------------------------------------------------------------[/]<br>
Download (curl) error for 'https://releases.jolla.com/releases/3.0.3.10/sdk/armv7hl/repodata/repomd.xml':<br>
Error code: Connection failed<br>
Error message: Could not resolve host: releases.jolla.com<br>
<br>
Abort, retry, ignore? [a/r/i/?] (a): a<br>
Retrieving repository 'sdk' metadata .............................................................................................[error]<br>
Repository 'sdk' is invalid.<br>
[|] Valid metadata not found at specified URL(s)<br>
Please check if the URIs defined for this repository are pointing to a valid repository.<br>
Skipping repository 'sdk' because of the above error.<br>
Could not refresh the repositories because of errors.<br>
</details><br>

Такое иногда бывает выходим из песочницы и заходим снова
```console
PlatformSDK:~/hadk$ exit
HOST:~$ sfossdk
```
<details>
[sudo] пароль для stalker:<br>
Mounting system directories...<br>
Mounting /srv/mer/targets as /srv/mer/targets<br>
Mounting /srv/mer/toolings as /srv/mer/toolings<br>
Mounting / as /parentroot<br>
Mounting home directory: /home/stalker<br>
Entering chroot as stalker<br>
Last login: Mon Jul  8 09:51:22 EDT 2019 on pts/0<br>
Env setup for mido<br>
</details><br>

```console
PlatformSDK:~$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper ref
```
<details>
Retrieving repository 'hotfixes' metadata .........................................................................................[done]<br>
Building repository 'hotfixes' cache ..............................................................................................[done]<br>
Retrieving repository 'jolla' metadata ............................................................................................[done]<br>
Building repository 'jolla' cache .................................................................................................[done]<br>
Retrieving repository 'sdk' metadata ..............................................................................................[done]<br>
Building repository 'sdk' cache ...................................................................................................[done]<br>
All repositories have been refreshed.<br>
</details><br>

```console
PlatformSDK:~$ sb2 -t $VENDOR-$DEVICE-$PORT_ARCH -m sdk-install -R zypper dup
```
<details>
Warning: You are about to do a distribution upgrade with all enabled repositories. Make sure these repositories are compatible before you continue. See 'man zypper' for more information about this command.<br>
Loading repository data...<br>
Reading installed packages...<br>
Computing distribution upgrade...<br>
<br>
The following NEW packages are going to be installed:<br>
  boost-thread busybox-symlinks-diffutils busybox-symlinks-findutils busybox-symlinks-grep buteo-mtp-qt5<br>
  buteo-mtp-qt5-sample-vendor-configuration buteo-syncfw-qt5 cryptsetup-libs ffmpeg gpgme iptables-ipv6 json-c libaccounts-qt5-devel<br>
  libicu libmce-qt5 libpython3_7m1_0 libqt5sparql libqt5sparql-tracker libqt5sparql-tracker-direct libsignon-qt5-devel nspr-devel<br>
  nss-devel poppler-qt5 sailfish-minui-resources-z1.0 sdk-harbour-rpmvalidator sqlite-libs tar thumbnaild<br>
<br>
The following packages are going to be REMOVED:<br>
  diffutils findutils grep libav libicu52 libpython3_4m1_0 sqlite<br>
<br>
The following packages are going to be upgraded:<br>
  PackageKit PackageKit-Qt5 PackageKit-glib PackageKit-zypp alsa-lib ambienced ambienced-devel augeas-libs basesystem bash binutils<br>
  boost-filesystem boost-system busybox busybox-symlinks-dhcp busybox-symlinks-gzip bzip2 bzip2-libs ca-certificates cairo connman<br>
  connman-configs-mer connman-qt5 cor coreutils cpio curl db4 db4-utils dbus dbus-devel dbus-glib dbus-libs dbusextended-qt5<br>
  declarative-transferengine-qt5 desktop-file-utils device-mapper device-mapper-event device-mapper-event-libs device-mapper-libs dsme<br>
  elfutils elfutils-libelf elfutils-libs exempi expat expat-devel file file-libs filesystem flac fontconfig fontconfig-devel<br>
  fontpackages-filesystem freetype freetype-devel fuse fuse-libs gawk gdbm giflib glib-networking glib2 glibc glibc-common glibc-devel<br>
  glibc-headers gmime gnupg2 gnutls groff gstreamer1.0 gstreamer1.0-plugins-base info iptables jolla-ambient-sound-theme jolla-ca<br>
  kernel-headers kmod-libs libaccounts-glib libaccounts-qt5 libacl libarchive libasyncns libattr libblkid libcanberra libcap<br>
  libcontentaction-qt5 libcurl libdbus-qeventloop-qt5 libdbusaccess libdbuslogserver-dbus libdrm libdsme libexif libfdisk libffi libgcc<br>
  libgcrypt libglibutil libgofono libgofonoext libgpg-error libgrilio libgsf libgsupplicant libiodata-qt5 libiphb libiptcdata<br>
  libjollasignonuiservice-qt5 libjollasignonuiservice-qt5-plugin libjpeg-turbo libkeepalive libkf5archive libksba liblua libmce-glib<br>
  libmediaart libmlocale-qt5 libmount libnemotransferengine-qt5 libngf libngf-qt5 libngf-qt5-declarative libnl libogg libpng<br>
  libqmfclient1-qt5 libqmfmessageserver1-qt5 libqtaround2 libqtwebkit5 libqtwebkit5-devel libqtwebkit5-widgets libquillmetadata-qt5<br>
  libresource libresourceqt-qt5 libsailfishkeyprovider libsbc libshadowutils libsignon-qt5 libsmartcols libsndfile libsolv-tools<br>
  libsolv0 libsoup libstdc++ libtasn1 libtheora libtiff libtool-ltdl libusb libusb-moded-qt5 libuser libutempter libuuid libvorbis<br>
  libvpx libwebp libwspcodec libxkbcommon libxml2 libxslt libzypp lsof lvm2 lvm2-libs maliit-framework-wayland<br>
  maliit-framework-wayland-devel maliit-framework-wayland-inputcontext mapplauncherd mapplauncherd-qt5 mapplauncherd-qt5-devel mce<br>
  meego-rpm-config mesa-llvmpipe mesa-llvmpipe-libEGL mesa-llvmpipe-libEGL-devel mesa-llvmpipe-libGLESv2 mesa-llvmpipe-libGLESv2-devel<br>
  mesa-llvmpipe-libglapi mesa-llvmpipe-libwayland-egl mlite-qt5 mlite-qt5-devel mobile-broadband-provider-info mpris-qt5<br>
  mpris-qt5-qml-plugin mtdev ncurses ncurses-base ncurses-libs nemo-gstreamer1.0-interfaces nemo-qml-plugin-contentaction<br>
  nemo-qml-plugin-dbus-qt5 nemo-qml-plugin-filemanager nemo-qml-plugin-models-qt5 nemo-qml-plugin-notifications-qt5<br>
  nemo-qml-plugin-systemsettings nemo-qml-plugin-thumbnailer-qt5 nemo-transferengine-qt5 ngfd ngfd-settings-sailfish nspr nss<br>
  nss-softokn-freebl nss-sysinit ofono ofono-configs-mer oneshot openjpeg openssl-libs opus orc p11-kit p11-kit-nss-ckbi p11-kit-trust<br>
  pacrunner pam passwd patch patterns-sailfish-configuration-sdk-target patterns-sailfish-qt5-devel-basic<br>
  patterns-sailfish-qt5-devel-full patterns-sailfish-silica-devel patterns-sailfish-target-support pcre perl perl-Filter<br>
  perl-Module-Pluggable perl-Pod-Escapes perl-Pod-Parser perl-Pod-Perldoc perl-Pod-Simple perl-Scalar-List-Utils perl-Socket perl-libs<br>
  perl-macros perl-parent perl-threads perl-threads-shared pixman pkgconfig polkit poppler poppler-glib popt procps profiled psmisc pth<br>
  pulseaudio pyotherside-qml-plugin-python3-qt5 python python-libs python3-base qmf-qt5-devel qml-rpm-macros qt5-default<br>
  qt5-plugin-bearer-generic qt5-plugin-imageformat-jpeg qt5-plugin-platform-minimal qt5-plugin-sqldriver-sqlite qt5-qmake<br>
  qt5-qtconcurrent qt5-qtconcurrent-devel qt5-qtconnectivity-qtbluetooth qt5-qtconnectivity-qtbluetooth-devel qt5-qtcore<br>
  qt5-qtcore-devel qt5-qtdbus qt5-qtdbus-devel qt5-qtdeclarative qt5-qtdeclarative-devel qt5-qtdeclarative-import-folderlistmodel<br>
  qt5-qtdeclarative-import-localstorageplugin qt5-qtdeclarative-import-location qt5-qtdeclarative-import-models2<br>
  qt5-qtdeclarative-import-multimedia qt5-qtdeclarative-import-particles2 qt5-qtdeclarative-import-positioning<br>
  qt5-qtdeclarative-import-qtquick2plugin qt5-qtdeclarative-import-qttest qt5-qtdeclarative-import-sensors<br>
  qt5-qtdeclarative-import-window2 qt5-qtdeclarative-import-xmllistmodel qt5-qtdeclarative-pim-contacts qt5-qtdeclarative-pim-organizer<br>
  qt5-qtdeclarative-plugin-qmlinspector qt5-qtdeclarative-qtquick qt5-qtdeclarative-qtquick-devel qt5-qtdeclarative-qtquickparticles<br>
  qt5-qtdeclarative-qtquickparticles-devel qt5-qtdeclarative-qtquicktest qt5-qtdocgallery qt5-qtdocgallery-devel qt5-qtfeedback<br>
  qt5-qtfeedback-devel qt5-qtgraphicaleffects qt5-qtgui qt5-qtgui-devel qt5-qtlocation qt5-qtlocation-devel qt5-qtmultimedia<br>
  qt5-qtmultimedia-devel qt5-qtmultimedia-gsttools qt5-qtnetwork qt5-qtnetwork-devel qt5-qtopengl qt5-qtopengl-devel qt5-qtpim-contacts<br>
  qt5-qtpim-contacts-devel qt5-qtpim-organizer qt5-qtpim-organizer-devel qt5-qtpim-versit qt5-qtpim-versit-devel<br>
  qt5-qtpim-versitorganizer qt5-qtpim-versitorganizer-devel qt5-qtplatformsupport-devel qt5-qtpositioning qt5-qtpositioning-devel<br>
  qt5-qtprintsupport qt5-qtqml-import-webkitplugin qt5-qtqml-import-webkitplugin-experimental qt5-qtquickcontrols-layouts qt5-qtsensors<br>
  qt5-qtsensors-devel qt5-qtsql qt5-qtsql-devel qt5-qtsvg qt5-qtsvg-devel qt5-qttest qt5-qttools qt5-qttools-linguist<br>
  qt5-qtwayland-wayland_egl qt5-qtwayland-wayland_egl-devel qt5-qtwebkit-uiprocess-launcher qt5-qtwidgets qt5-qtwidgets-devel qt5-qtxml<br>
  qt5-qtxml-devel qt5-qtxmlpatterns qt5-qtxmlpatterns-devel qt5-tools qtchooser readline rpm rpm-build rpm-libs sailfish-ca<br>
  sailfish-components-filemanager sailfish-components-gallery-qt5 sailfish-components-pickers-qt5 sailfish-content-graphics-closed<br>
  sailfish-content-graphics-closed-z1.0 sailfish-content-graphics-default sailfish-content-graphics-default-base<br>
  sailfish-content-graphics-default-z1.0 sailfish-content-graphics-default-z1.0-base sailfish-content-profiled-settings-default<br>
  sailfish-content-tones-default sailfish-fonts sailfish-silica-background-qt5 sailfish-upgrade-ui-resources-z1.0 sailfish-version<br>
  sailfish-version-variant sailfishsilica-qt5 sdk-configs sdk-register sdk-target-configs sed setup shadow-utils shared-mime-info<br>
  signon-qt5 sound-theme-freedesktop speex speexdsp ssu ssu-network-proxy-plugin ssu-sysinfo ssu-vendor-data-jolla systemd<br>
  systemd-config-sailfish systemd-libs taglib timed-qt5 totem-pl-parser tracker tzdata tzdata-timed unzip usb-moded usb-moded-defaults<br>
  usb-moded-developer-mode util-linux wayland xdg-utils xkeyboard-config xz xz-libs xz-lzma-compat zlib zlib-devel zypper<br>
<br>
The following packages are going to be downgraded:<br>
  dconf gstreamer1.0-plugins-good libmpg123 signon-plugin-oauth2-qt5 statefs-provider-inout-power systemd-user-session-targets<br>
  wpa_supplicant<br>
<br>
The following packages are going to be reinstalled:<br>
  contextkit-declarative-qt5 libsailfishapp libsailfishapp-devel nemo-qml-plugin-configuration-qt5<br>
  nemo-qtmultimedia-plugins-gstvideotexturebackend qt5-qtdeclarative-publishsubscribe qt5-qtdeclarative-serviceframework<br>
  qt5-qtdeclarative-systeminfo qt5-qtpublishsubscribe qt5-qtpublishsubscribe-devel qt5-qtserviceframework qt5-qtserviceframework-devel<br>
  qt5-qtsysteminfo sailfish-components-media-qt5 statefs statefs-contextkit-subscriber statefs-pp statefs-qt5<br>
<br>
404 packages to upgrade, 7 to downgrade, 28 new, 18 to reinstall, 7 to remove.<br>
Overall download size: 168.0 MiB. After the operation, additional 45.7 MiB will be used.<br>
Continue? [y/n/?] (y): y<br>
Retrieving package buteo-mtp-qt5-sample-vendor-configuration-0.7.0-1.6.1.jolla.armv7hl            (1/457),  28.0 KiB (108.6 KiB unpacked)<br>
Retrieving: buteo-mtp-qt5-sample-vendor-configuration-0.7.0-1.6.1.jolla.armv7hl.rpm ...............................................[done]<br>
Retrieving package fontpackages-filesystem-1.44-1.1.9.jolla.noarch                                  (2/457),   8.9 KiB (    0 B unpacked)<br>
Retrieving: fontpackages-filesystem-1.44-1.1.9.jolla.noarch.rpm ...................................................................[done]<br>
Retrieving package jolla-ca-0.9-1.3.1.jolla.noarch                                                (3/457),  15.1 KiB ( 22.5 KiB unpacked)<br>
Retrieving: jolla-ca-0.9-1.3.1.jolla.noarch.rpm ...................................................................................[done]<br>
Retrieving package kernel-headers-3.18.136-1.2.3.jolla.armv7hl                                    (4/457), 793.3 KiB (  3.2 MiB unpacked)<br>
Retrieving: kernel-headers-3.18.136-1.2.3.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libgcc-4.9.4-1.2.5.jolla.armv7hl                                               (5/457),  47.7 KiB ( 84.9 KiB unpacked)<br>
Retrieving: libgcc-4.9.4-1.2.5.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch               (6/457),  55.8 KiB (329.0 KiB unpacked)<br>
Retrieving: mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch.rpm ..................................................[done]<br>
Retrieving package ncurses-base-6.1+git1-1.3.5.jolla.armv7hl                                      (7/457),  57.6 KiB (320.4 KiB unpacked)<br>
Retrieving: ncurses-base-6.1+git1-1.3.5.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package ofono-configs-mer-1.21+git44-1.19.1.jolla.armv7hl                              (8/457),  68.1 KiB (  8.0 KiB unpacked)<br>
Retrieving: ofono-configs-mer-1.21+git44-1.19.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package qt5-qtplatformsupport-devel-5.6.3+git9-1.9.2.jolla.armv7hl                     (9/457), 237.2 KiB (  1.2 MiB unpacked)<br>
Retrieving: qt5-qtplatformsupport-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package sailfish-ca-0.1.1-1.3.1.jolla.noarch                                          (10/457),  11.3 KiB (  9.1 KiB unpacked)<br>
Retrieving: sailfish-ca-0.1.1-1.3.1.jolla.noarch.rpm ..............................................................................[done]<br>
Retrieving package setup-2.8.56-1.2.5.jolla.noarch                                               (11/457), 115.0 KiB (663.0 KiB unpacked)<br>
Retrieving: setup-2.8.56-1.2.5.jolla.noarch.rpm ...................................................................................[done]<br>
Retrieving package tzdata-2017b-1.1.11.jolla.noarch                                              (12/457), 304.2 KiB (  1.1 MiB unpacked)<br>
Retrieving: tzdata-2017b-1.1.11.jolla.noarch.rpm ..................................................................................[done]<br>
Retrieving package usb-moded-developer-mode-0.86.0+mer33-1.3.2.jolla.armv7hl                       (13/457),  42.5 KiB (  110 B unpacked)<br>
Retrieving: usb-moded-developer-mode-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package xkeyboard-config-2.10.1+git6-1.2.7.jolla.noarch                               (14/457), 311.1 KiB (  2.4 MiB unpacked)<br>
Retrieving: xkeyboard-config-2.10.1+git6-1.2.7.jolla.noarch.rpm ...................................................................[done]<br>
Retrieving package filesystem-3.1-1.1.9.jolla.noarch                                               (15/457), 911.0 KiB (    0 B unpacked)<br>
Retrieving: filesystem-3.1-1.1.9.jolla.noarch.rpm .................................................................................[done]<br>
Retrieving package tzdata-timed-2017b.2-1.3.1.jolla.noarch                                       (16/457),  17.0 KiB ( 35.9 KiB unpacked)<br>
Retrieving: tzdata-timed-2017b.2-1.3.1.jolla.noarch.rpm ...........................................................................[done]<br>
Retrieving package glibc-2.25+git5-1.4.1.jolla.armv7hl                                           (17/457),   2.3 MiB (  8.7 MiB unpacked)<br>
Retrieving: glibc-2.25+git5-1.4.1.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package usb-moded-defaults-0.86.0+mer33-1.3.2.jolla.armv7hl                             (18/457),  41.8 KiB (    0 B unpacked)<br>
Retrieving: usb-moded-defaults-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package basesystem-11+git1-1.2.8.jolla.noarch                                           (19/457),   8.0 KiB (    0 B unpacked)<br>
Retrieving: basesystem-11+git1-1.2.8.jolla.noarch.rpm .............................................................................[done]<br>
Retrieving package ncurses-libs-6.1+git1-1.3.5.jolla.armv7hl                                     (20/457), 201.6 KiB (504.4 KiB unpacked)<br>
Retrieving: ncurses-libs-6.1+git1-1.3.5.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package sailfish-content-graphics-closed-z1.0-0.6.1-1.2.1.jolla.noarch                (21/457), 416.0 KiB (389.0 KiB unpacked)<br>
Retrieving: sailfish-content-graphics-closed-z1.0-0.6.1-1.2.1.jolla.noarch.rpm ....................................................[done]<br>
Retrieving package bash-1:3.2.57-1.2.5.jolla.armv7hl                                             (22/457), 277.7 KiB (542.1 KiB unpacked)<br>
Retrieving: bash-3.2.57-1.2.5.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package sailfish-content-graphics-closed-0.6.1-1.2.1.jolla.noarch                     (23/457), 175.9 KiB (157.7 KiB unpacked)<br>
Retrieving: sailfish-content-graphics-closed-0.6.1-1.2.1.jolla.noarch.rpm .........................................................[done]<br>
Retrieving package glibc-common-2.25+git5-1.4.1.jolla.armv7hl                                    (24/457),   4.0 MiB ( 13.6 MiB unpacked)<br>
Retrieving: glibc-common-2.25+git5-1.4.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package zlib-1.2.11+git1-1.4.5.jolla.armv7hl                                          (25/457),  68.4 KiB (106.2 KiB unpacked)<br>
Retrieving: zlib-1.2.11+git1-1.4.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package sailfish-minui-resources-z1.0-0.0.4-1.4.2.jolla.armv7hl                       (26/457),  40.9 KiB ( 31.9 KiB unpacked)<br>
Retrieving: sailfish-minui-resources-z1.0-0.0.4-1.4.2.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package qt5-qttools-5.6.3+git1-1.4.1.jolla.armv7hl                                      (27/457),   7.5 KiB (    0 B unpacked)<br>
Retrieving: qt5-qttools-5.6.3+git1-1.4.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package qml-rpm-macros-0.0.5-1.1.9.jolla.armv7hl                                      (28/457),   8.8 KiB (  3.9 KiB unpacked)<br>
Retrieving: qml-rpm-macros-0.0.5-1.1.9.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package info-4.13a-1.2.7.jolla.armv7hl                                                (29/457),  95.3 KiB (187.3 KiB unpacked)<br>
Retrieving: info-4.13a-1.2.7.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package sailfish-upgrade-ui-resources-z1.0-0.1.1-1.5.2.jolla.armv7hl                  (30/457), 869.1 KiB (869.1 KiB unpacked)<br>
Retrieving: sailfish-upgrade-ui-resources-z1.0-0.1.1-1.5.2.jolla.armv7hl.rpm ......................................................[done]<br>
Retrieving package glibc-headers-2.25+git5-1.4.1.jolla.armv7hl                                   (31/457), 433.3 KiB (  2.1 MiB unpacked)<br>
Retrieving: glibc-headers-2.25+git5-1.4.1.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package glibc-devel-2.25+git5-1.4.1.jolla.armv7hl                                     (32/457),  29.2 KiB ( 59.7 KiB unpacked)<br>
Retrieving: glibc-devel-2.25+git5-1.4.1.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package xz-libs-5.0.4-1.2.8.jolla.armv7hl                                             (33/457),  64.6 KiB ( 91.4 KiB unpacked)<br>
Retrieving: xz-libs-5.0.4-1.2.8.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package unzip-6.0-1.2.5.jolla.armv7hl                                                 (34/457),  84.0 KiB (182.3 KiB unpacked)<br>
Retrieving: unzip-6.0-1.2.5.jolla.armv7hl.rpm .....................................................................................[done]<br>
Retrieving package ssu-sysinfo-1.1.3-1.3.7.jolla.armv7hl                                         (35/457),  20.9 KiB ( 34.0 KiB unpacked)<br>
Retrieving: ssu-sysinfo-1.1.3-1.3.7.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package speexdsp-1.2.0+git2-1.1.7.jolla.armv7hl                                       (36/457),  43.9 KiB ( 56.4 KiB unpacked)<br>
Retrieving: speexdsp-1.2.0+git2-1.1.7.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package speex-1.2.0+git1-1.2.7.jolla.armv7hl                                          (37/457),  52.2 KiB ( 69.1 KiB unpacked)<br>
Retrieving: speex-1.2.0+git1-1.2.7.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package shadow-utils-4.6-1.2.5.jolla.armv7hl                                          (38/457), 148.9 KiB (749.1 KiB unpacked)<br>
Retrieving: shadow-utils-4.6-1.2.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package sed-1:4.1.5-1.2.6.jolla.armv7hl                                               (39/457),  31.4 KiB ( 49.4 KiB unpacked)<br>
Retrieving: sed-4.1.5-1.2.6.jolla.armv7hl.rpm .....................................................................................[done]<br>
Retrieving package readline-5.2-1.2.7.jolla.armv7hl                                              (40/457),  89.8 KiB (190.1 KiB unpacked)<br>
Retrieving: readline-5.2-1.2.7.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package pth-2.0.7-1.1.9.jolla.armv7hl                                                 (41/457),  44.2 KiB ( 76.7 KiB unpacked)<br>
Retrieving: pth-2.0.7-1.1.9.jolla.armv7hl.rpm .....................................................................................[done]<br>
Retrieving package psmisc-22.13-1.3.6.jolla.armv7hl                                              (42/457),  40.6 KiB ( 80.5 KiB unpacked)<br>
Retrieving: psmisc-22.13-1.3.6.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package procps-3.2.8-1.2.7.jolla.armv7hl                                              (43/457), 122.9 KiB (287.1 KiB unpacked)<br>
Retrieving: procps-3.2.8-1.2.7.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package popt-1.16-1.1.10.jolla.armv7hl                                                (44/457),  26.7 KiB ( 31.7 KiB unpacked)<br>
Retrieving: popt-1.16-1.1.10.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package pkgconfig-0.27.1-1.2.7.jolla.armv7hl                                          (45/457), 139.7 KiB (415.6 KiB unpacked)<br>
Retrieving: pkgconfig-0.27.1-1.2.7.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package pixman-0.34.0-1.1.9.jolla.armv7hl                                             (46/457), 154.8 KiB (487.6 KiB unpacked)<br>
Retrieving: pixman-0.34.0-1.1.9.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package patch-2.7.5+git1-1.1.9.jolla.armv7hl                                          (47/457),  85.7 KiB (136.2 KiB unpacked)<br>
Retrieving: patch-2.7.5+git1-1.1.9.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package orc-0.4.26+git1-1.1.9.jolla.armv7hl                                           (48/457), 123.1 KiB (354.7 KiB unpacked)<br>
Retrieving: orc-0.4.26+git1-1.1.9.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package opus-1.2.1-1.2.7.jolla.armv7hl                                                (49/457), 149.6 KiB (208.8 KiB unpacked)<br>
Retrieving: opus-1.2.1-1.2.7.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package openjpeg-2.3.0+git1-1.2.5.jolla.armv7hl                                       (50/457), 130.3 KiB (246.8 KiB unpacked)<br>
Retrieving: openjpeg-2.3.0+git1-1.2.5.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package nspr-4.20.0-1.3.7.jolla.armv7hl                                               (51/457),  98.5 KiB (178.5 KiB unpacked)<br>
Retrieving: nspr-4.20.0-1.3.7.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package ncurses-6.1+git1-1.3.5.jolla.armv7hl                                          (52/457),  64.9 KiB (149.9 KiB unpacked)<br>
Retrieving: ncurses-6.1+git1-1.3.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package mtdev-1.1.3-1.1.9.jolla.armv7hl                                               (53/457),  15.2 KiB ( 13.8 KiB unpacked)<br>
Retrieving: mtdev-1.1.3-1.1.9.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package lsof-4.91+git1-1.3.4.jolla.armv7hl                                            (54/457),  68.4 KiB (118.9 KiB unpacked)<br>
Retrieving: lsof-4.91+git1-1.3.4.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package libxml2-2.9.8+git2-1.3.10.jolla.armv7hl                                       (55/457), 454.0 KiB (924.2 KiB unpacked)<br>
Retrieving: libxml2-2.9.8+git2-1.3.10.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libxkbcommon-0.5.0+git1-1.2.8.jolla.armv7hl                                   (56/457),  77.3 KiB (220.4 KiB unpacked)<br>
Retrieving: libxkbcommon-0.5.0+git1-1.2.8.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libwebp-0.6.1+git3-1.3.6.jolla.armv7hl                                        (57/457), 164.4 KiB (364.4 KiB unpacked)<br>
Retrieving: libwebp-0.6.1+git3-1.3.6.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libtool-ltdl-2.4.6-1.2.9.jolla.armv7hl                                        (58/457),  42.1 KiB ( 48.7 KiB unpacked)<br>
Retrieving: libtool-ltdl-2.4.6-1.2.9.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libtasn1-4.13+git1-1.3.9.jolla.armv7hl                                        (59/457),  42.7 KiB ( 70.3 KiB unpacked)<br>
Retrieving: libtasn1-4.13+git1-1.3.9.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libstdc++-4.9.4-1.2.5.jolla.armv7hl                                           (60/457), 217.2 KiB (681.2 KiB unpacked)<br>
Retrieving: libstdc++-4.9.4-1.2.5.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package libshadowutils-0.0.2-1.4.1.jolla.armv7hl                                      (61/457),  13.4 KiB ( 13.4 KiB unpacked)<br>
Retrieving: libshadowutils-0.0.2-1.4.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libsbc-1.3-1.3.6.jolla.armv7hl                                                (62/457),  34.5 KiB ( 54.0 KiB unpacked)<br>
Retrieving: libsbc-1.3-1.3.6.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package libpng-1.6.34-1.2.9.jolla.armv7hl                                             (63/457),  88.4 KiB (141.7 KiB unpacked)<br>
Retrieving: libpng-1.6.34-1.2.9.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package libogg-1.3.3-1.2.7.jolla.armv7hl                                              (64/457),  17.3 KiB ( 15.9 KiB unpacked)<br>
Retrieving: libogg-1.3.3-1.2.7.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package libnl-3.4.0-1.2.7.jolla.armv7hl                                               (65/457), 215.3 KiB (567.4 KiB unpacked)<br>
Retrieving: libnl-3.4.0-1.2.7.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package libmpg123-1.25.10-1.1.7.jolla.armv7hl                                         (66/457), 145.5 KiB (294.3 KiB unpacked)<br>
Retrieving: libmpg123-1.25.10-1.1.7.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package liblua-5.1.5-1.1.11.jolla.armv7hl                                             (67/457),  64.0 KiB (100.3 KiB unpacked)<br>
Retrieving: liblua-5.1.5-1.1.11.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package libjpeg-turbo-1.5.3+git1-1.4.5.jolla.armv7hl                                  (68/457), 116.5 KiB (364.4 KiB unpacked)<br>
Retrieving: libjpeg-turbo-1.5.3+git1-1.4.5.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package libiptcdata-1.0.4+git1-1.3.1.jolla.armv7hl                                    (69/457),  42.3 KiB (118.5 KiB unpacked)<br>
Retrieving: libiptcdata-1.0.4+git1-1.3.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package libiphb-1.2.5+git1-1.3.3.jolla.armv7hl                                        (70/457),  21.2 KiB ( 31.6 KiB unpacked)<br>
Retrieving: libiphb-1.2.5+git1-1.3.3.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libgpg-error-1.27+git2-1.3.5.jolla.armv7hl                                    (71/457), 123.5 KiB (538.2 KiB unpacked)<br>
Retrieving: libgpg-error-1.27+git2-1.3.5.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package libffi-3.2.1+git1-1.2.13.jolla.armv7hl                                        (72/457),  23.2 KiB ( 22.0 KiB unpacked)<br>
Retrieving: libffi-3.2.1+git1-1.2.13.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libexif-0.6.21+git2-1.2.5.jolla.armv7hl                                       (73/457), 294.8 KiB (  1.7 MiB unpacked)<br>
Retrieving: libexif-0.6.21+git2-1.2.5.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libdrm-2.4.39-1.1.9.jolla.armv7hl                                             (74/457),  28.4 KiB ( 39.2 KiB unpacked)<br>
Retrieving: libdrm-2.4.39-1.1.9.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package libattr-2.4.47+git1-1.3.7.jolla.armv7hl                                       (75/457),  23.2 KiB ( 37.9 KiB unpacked)<br>
Retrieving: libattr-2.4.47+git1-1.3.7.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libasyncns-0.8-1.2.5.jolla.armv7hl                                            (76/457),  23.9 KiB ( 39.8 KiB unpacked)<br>
Retrieving: libasyncns-0.8-1.2.5.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package json-c-0.12-1.1.9.jolla.armv7hl                                               (77/457),  22.3 KiB ( 29.1 KiB unpacked)<br>
Retrieving: json-c-0.12-1.1.9.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package iptables-1.8.2+git1-1.4.3.jolla.armv7hl                                       (78/457), 211.7 KiB (731.2 KiB unpacked)<br>
Retrieving: iptables-1.8.2+git1-1.4.3.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package giflib-4.2.3+git2-1.3.5.jolla.armv7hl                                         (79/457),  21.5 KiB ( 25.9 KiB unpacked)<br>
Retrieving: giflib-4.2.3+git2-1.3.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package gdbm-1.8.3-1.2.10.jolla.armv7hl                                               (80/457),  23.0 KiB ( 32.4 KiB unpacked)<br>
Retrieving: gdbm-1.8.3-1.2.10.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package freetype-2.8.0-1.1.9.jolla.armv7hl                                            (81/457), 262.1 KiB (433.3 KiB unpacked)<br>
Retrieving: freetype-2.8.0-1.1.9.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package file-libs-5.35+git2-1.2.8.jolla.armv7hl                                       (82/457), 465.3 KiB (  6.2 MiB unpacked)<br>
Retrieving: file-libs-5.35+git2-1.2.8.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package expat-2.1.0-1.1.10.jolla.armv7hl                                              (83/457),  56.6 KiB (106.4 KiB unpacked)<br>
Retrieving: expat-2.1.0-1.1.10.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package elfutils-libelf-0.170+git1-1.3.10.jolla.armv7hl                               (84/457),  53.2 KiB ( 65.8 KiB unpacked)<br>
Retrieving: elfutils-libelf-0.170+git1-1.3.10.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package db4-4.8.30-1.3.12.jolla.armv7hl                                               (85/457), 475.4 KiB (926.2 KiB unpacked)<br>
Retrieving: db4-4.8.30-1.3.12.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package cpio-2.12+git1-1.3.4.jolla.armv7hl                                            (86/457),  67.3 KiB (122.7 KiB unpacked)<br>
Retrieving: cpio-2.12+git1-1.3.4.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package bzip2-libs-1.0.6-1.2.9.jolla.armv7hl                                          (87/457),  37.5 KiB ( 49.0 KiB unpacked)<br>
Retrieving: bzip2-libs-1.0.6-1.2.9.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package busybox-1.29.3+git5-1.1.5.jolla.armv7hl                                       (88/457),  75.8 KiB (112.7 KiB unpacked)<br>
Retrieving: busybox-1.29.3+git5-1.1.5.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package binutils-2.25-1.3.11.jolla.armv7hl                                            (89/457),   3.1 MiB ( 15.7 MiB unpacked)<br>
Retrieving: binutils-2.25-1.3.11.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package xz-5.0.4-1.2.8.jolla.armv7hl                                                  (90/457),  56.5 KiB (148.1 KiB unpacked)<br>
Retrieving: xz-5.0.4-1.2.8.jolla.armv7hl.rpm ......................................................................................[done]<br>
Retrieving package libutempter-1.1.5+git1-1.1.12.jolla.armv7hl                                   (91/457),  21.4 KiB ( 35.4 KiB unpacked)<br>
Retrieving: libutempter-1.1.5+git1-1.1.12.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package zlib-devel-1.2.11+git1-1.4.5.jolla.armv7hl                                    (92/457),  35.4 KiB (110.1 KiB unpacked)<br>
Retrieving: zlib-devel-1.2.11+git1-1.4.5.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package nspr-devel-4.20.0-1.3.7.jolla.armv7hl                                         (93/457), 106.2 KiB (447.8 KiB unpacked)<br>
Retrieving: nspr-devel-4.20.0-1.3.7.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package libxslt-1.1.29-1.2.6.jolla.armv7hl                                            (94/457), 119.4 KiB (240.9 KiB unpacked)<br>
Retrieving: libxslt-1.1.29-1.2.6.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package augeas-libs-1.6.0+git1-1.2.5.jolla.armv7hl                                    (95/457), 449.5 KiB (  1.6 MiB unpacked)<br>
Retrieving: augeas-libs-1.6.0+git1-1.2.5.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package taglib-1.11.1+git1-1.3.1.jolla.armv7hl                                        (96/457), 220.6 KiB (652.2 KiB unpacked)<br>
Retrieving: taglib-1.11.1+git1-1.3.1.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package qtchooser-26-1.1.9.jolla.armv7hl                                              (97/457),  37.3 KiB ( 79.0 KiB unpacked)<br>
Retrieving: qtchooser-26-1.1.9.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package pcre-8.42+git1-1.3.5.jolla.armv7hl                                            (98/457), 265.9 KiB (710.4 KiB unpacked)<br>
Retrieving: pcre-8.42+git1-1.3.5.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package libvpx-1.7.0+git1-1.2.5.jolla.armv7hl                                         (99/457), 928.5 KiB (  2.0 MiB unpacked)<br>
Retrieving: libvpx-1.7.0+git1-1.2.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package libusb-0.1.12-1.2.5.jolla.armv7hl                                            (100/457),  32.2 KiB ( 59.4 KiB unpacked)<br>
Retrieving: libusb-0.1.12-1.2.5.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package libsailfishkeyprovider-0.0.14-1.2.1.jolla.armv7hl                            (101/457),  17.4 KiB ( 20.3 KiB unpacked)<br>
Retrieving: libsailfishkeyprovider-0.0.14-1.2.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package libvorbis-1.3.6+git1-1.2.5.jolla.armv7hl                                     (102/457), 149.0 KiB (670.2 KiB unpacked)<br>
Retrieving: libvorbis-1.3.6+git1-1.2.5.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libtheora-1.2.0alpha1+git-1.2.8.jolla.armv7hl                                (103/457), 140.9 KiB (469.6 KiB unpacked)<br>
Retrieving: libtheora-1.2.0alpha1+git-1.2.8.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package flac-1.3.2-1.2.7.jolla.armv7hl                                               (104/457), 171.4 KiB (340.3 KiB unpacked)<br>
Retrieving: flac-1.3.2-1.2.7.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package libtiff-4.0.10+git1-1.2.6.jolla.armv7hl                                      (105/457), 136.1 KiB (354.7 KiB unpacked)<br>
Retrieving: libtiff-4.0.10+git1-1.2.6.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libksba-1.3.5+git2-1.2.5.jolla.armv7hl                                       (106/457),  80.9 KiB (158.4 KiB unpacked)<br>
Retrieving: libksba-1.3.5+git2-1.2.5.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libgcrypt-1.5.6+git1-1.1.11.jolla.armv7hl                                    (107/457), 213.8 KiB (372.2 KiB unpacked)<br>
Retrieving: libgcrypt-1.5.6+git1-1.1.11.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package p11-kit-0.23.12+git1-1.3.5.jolla.armv7hl                                     (108/457), 175.3 KiB (768.0 KiB unpacked)<br>
Retrieving: p11-kit-0.23.12+git1-1.3.5.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libcap-2.24+git1-1.3.6.jolla.armv7hl                                         (109/457),  33.2 KiB ( 65.4 KiB unpacked)<br>
Retrieving: libcap-2.24+git1-1.3.6.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libacl-2.2.53-1.2.7.jolla.armv7hl                                            (110/457),  18.9 KiB ( 19.1 KiB unpacked)<br>
Retrieving: libacl-2.2.53-1.2.7.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package iptables-ipv6-1.8.2+git1-1.4.3.jolla.armv7hl                                 (111/457),  44.1 KiB (124.5 KiB unpacked)<br>
Retrieving: iptables-ipv6-1.8.2+git1-1.4.3.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package file-5.35+git2-1.2.8.jolla.armv7hl                                           (112/457),  15.1 KiB ( 15.9 KiB unpacked)<br>
Retrieving: file-5.35+git2-1.2.8.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package wayland-1.6.0+git1-1.2.8.jolla.armv7hl                                       (113/457),  45.9 KiB (111.0 KiB unpacked)<br>
Retrieving: wayland-1.6.0+git1-1.2.8.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package expat-devel-2.1.0-1.1.10.jolla.armv7hl                                       (114/457),  22.1 KiB ( 44.3 KiB unpacked)<br>
Retrieving: expat-devel-2.1.0-1.1.10.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package exempi-2.4.3-1.2.1.jolla.armv7hl                                             (115/457), 486.9 KiB (  1.2 MiB unpacked)<br>
Retrieving: exempi-2.4.3-1.2.1.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package db4-utils-4.8.30-1.3.12.jolla.armv7hl                                        (116/457),  96.7 KiB (222.7 KiB unpacked)<br>
Retrieving: db4-utils-4.8.30-1.3.12.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package bzip2-1.0.6-1.2.9.jolla.armv7hl                                              (117/457),  31.7 KiB ( 38.4 KiB unpacked)<br>
Retrieving: bzip2-1.0.6-1.2.9.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package busybox-symlinks-gzip-1.29.3+git5-1.1.5.jolla.armv7hl                          (118/457),   9.5 KiB (    0 B unpacked)<br>
Retrieving: busybox-symlinks-gzip-1.29.3+git5-1.1.5.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package busybox-symlinks-grep-1.29.3+git5-1.1.5.jolla.armv7hl                          (119/457),   9.4 KiB (    0 B unpacked)<br>
Retrieving: busybox-symlinks-grep-1.29.3+git5-1.1.5.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package busybox-symlinks-findutils-1.29.3+git5-1.1.5.jolla.armv7hl                     (120/457),   9.4 KiB (    0 B unpacked)<br>
Retrieving: busybox-symlinks-findutils-1.29.3+git5-1.1.5.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package busybox-symlinks-diffutils-1.29.3+git5-1.1.5.jolla.armv7hl                     (121/457),   9.3 KiB (    0 B unpacked)<br>
Retrieving: busybox-symlinks-diffutils-1.29.3+git5-1.1.5.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package busybox-symlinks-dhcp-1.29.3+git5-1.1.5.jolla.armv7hl                          (122/457),   9.5 KiB (  151 B unpacked)<br>
Retrieving: busybox-symlinks-dhcp-1.29.3+git5-1.1.5.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package xz-lzma-compat-5.0.4-1.2.8.jolla.armv7hl                                     (123/457),  13.9 KiB ( 13.8 KiB unpacked)<br>
Retrieving: xz-lzma-compat-5.0.4-1.2.8.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package kmod-libs-21-1.2.8.jolla.armv7hl                                             (124/457),  37.3 KiB ( 54.6 KiB unpacked)<br>
Retrieving: kmod-libs-21-1.2.8.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package elfutils-libs-0.170+git1-1.3.10.jolla.armv7hl                                (125/457), 205.0 KiB (496.4 KiB unpacked)<br>
Retrieving: elfutils-libs-0.170+git1-1.3.10.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package freetype-devel-2.8.0-1.1.9.jolla.armv7hl                                     (126/457), 155.4 KiB (  1.2 MiB unpacked)<br>
Retrieving: freetype-devel-2.8.0-1.1.9.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package qt5-default-5.6.3+git9-1.9.2.jolla.armv7hl                                     (127/457),  10.9 KiB (    0 B unpacked)<br>
Retrieving: qt5-default-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package glib2-2.56.1+git3-1.3.6.jolla.armv7hl                                        (128/457),   1.1 MiB (  2.7 MiB unpacked)<br>
Retrieving: glib2-2.56.1+git3-1.3.6.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package tar-1.17-1.2.5.jolla.armv7hl                                                 (129/457), 303.3 KiB (  1.3 MiB unpacked)<br>
Retrieving: tar-1.17-1.2.5.jolla.armv7hl.rpm ......................................................................................[done]<br>
Retrieving package coreutils-1:6.9-1.2.6.jolla.armv7hl                                          (130/457), 492.4 KiB (  2.1 MiB unpacked)<br>
Retrieving: coreutils-6.9-1.2.6.jolla.armv7hl.rpm .................................................................................[done]<br>
Retrieving package elfutils-0.170+git1-1.3.10.jolla.armv7hl                                     (131/457), 217.8 KiB (541.0 KiB unpacked)<br>
Retrieving: elfutils-0.170+git1-1.3.10.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package shared-mime-info-1.12-1.4.2.jolla.armv7hl                                    (132/457), 318.4 KiB (  4.7 MiB unpacked)<br>
Retrieving: shared-mime-info-1.12-1.4.2.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package libwspcodec-2.2.1-1.2.7.jolla.armv7hl                                        (133/457),  12.8 KiB ( 12.1 KiB unpacked)<br>
Retrieving: libwspcodec-2.2.1-1.2.7.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package libgsf-1.14.36+git3-1.3.1.jolla.armv7hl                                      (134/457), 180.8 KiB (734.4 KiB unpacked)<br>
Retrieving: libgsf-1.14.36+git3-1.3.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libglibutil-1.0.35-1.8.4.jolla.armv7hl                                       (135/457),  26.3 KiB ( 31.9 KiB unpacked)<br>
Retrieving: libglibutil-1.0.35-1.8.4.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libdsme-0.66.1-1.4.4.jolla.armv7hl                                           (136/457),  25.4 KiB ( 42.9 KiB unpacked)<br>
Retrieving: libdsme-0.66.1-1.4.4.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package gstreamer1.0-1.14.1-1.3.5.jolla.armv7hl                                      (137/457), 756.3 KiB (  1.9 MiB unpacked)<br>
Retrieving: gstreamer1.0-1.14.1-1.3.5.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package gmime-2.6.20-1.2.1.jolla.armv7hl                                             (138/457), 122.4 KiB (310.3 KiB unpacked)<br>
Retrieving: gmime-2.6.20-1.2.1.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package desktop-file-utils-0.23+git1-1.2.9.jolla.armv7hl                             (139/457),  41.0 KiB (125.3 KiB unpacked)<br>
Retrieving: desktop-file-utils-0.23+git1-1.2.9.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package pam-1.1.8+git5-1.2.5.jolla.armv7hl                                           (140/457), 295.2 KiB (  1.2 MiB unpacked)<br>
Retrieving: pam-1.1.8+git5-1.2.5.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package libmce-glib-1.0.5-1.1.14.jolla.armv7hl                                       (141/457),  18.2 KiB ( 27.3 KiB unpacked)<br>
Retrieving: libmce-glib-1.0.5-1.1.14.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libgsupplicant-1.0.11-1.4.6.jolla.armv7hl                                    (142/457),  55.4 KiB (150.0 KiB unpacked)<br>
Retrieving: libgsupplicant-1.0.11-1.4.6.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package libgrilio-1.0.29-1.9.1.jolla.armv7hl                                         (143/457),  27.9 KiB ( 34.5 KiB unpacked)<br>
Retrieving: libgrilio-1.0.29-1.9.1.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libgofono-2.0.6-1.2.11.jolla.armv7hl                                         (144/457),  47.3 KiB (136.3 KiB unpacked)<br>
Retrieving: libgofono-2.0.6-1.2.11.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libdbusaccess-1.0.7-1.3.4.jolla.armv7hl                                      (145/457),  19.9 KiB ( 25.0 KiB unpacked)<br>
Retrieving: libdbusaccess-1.0.7-1.3.4.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package systemd-libs-225+git13-1.4.2.jolla.armv7hl                                   (146/457), 573.9 KiB (  3.6 MiB unpacked)<br>
Retrieving: systemd-libs-225+git13-1.4.2.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package sound-theme-freedesktop-0.8-1.3.1.jolla.noarch                               (147/457), 377.0 KiB (460.5 KiB unpacked)<br>
Retrieving: sound-theme-freedesktop-0.8-1.3.1.jolla.noarch.rpm ....................................................................[done]<br>
Retrieving package p11-kit-trust-0.23.12+git1-1.3.5.jolla.armv7hl                               (148/457),  96.7 KiB (264.9 KiB unpacked)<br>
Retrieving: p11-kit-trust-0.23.12+git1-1.3.5.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package libuser-0.62+git1-1.2.5.jolla.armv7hl                                        (149/457), 259.9 KiB (  1.5 MiB unpacked)<br>
Retrieving: libuser-0.62+git1-1.2.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package libsmartcols-2.33+git1-1.4.5.jolla.armv7hl                                   (150/457),  87.8 KiB (157.4 KiB unpacked)<br>
Retrieving: libsmartcols-2.33+git1-1.4.5.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package groff-1.18.1.4-1.1.14.jolla.armv7hl                                          (151/457),   1.3 MiB (  4.4 MiB unpacked)<br>
Retrieving: groff-1.18.1.4-1.1.14.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package gawk-1:3.1.5-1.2.7.jolla.armv7hl                                             (152/457), 157.6 KiB (488.5 KiB unpacked)<br>
Retrieving: gawk-3.1.5-1.2.7.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package fontconfig-2.12.4-1.2.9.jolla.armv7hl                                        (153/457), 135.4 KiB (308.7 KiB unpacked)<br>
Retrieving: fontconfig-2.12.4-1.2.9.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package alsa-lib-1.0.26-1.2.6.jolla.armv7hl                                          (154/457), 301.1 KiB (797.5 KiB unpacked)<br>
Retrieving: alsa-lib-1.0.26-1.2.6.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package libgofonoext-1.0.10-1.2.10.jolla.armv7hl                                     (155/457),  23.0 KiB ( 49.1 KiB unpacked)<br>
Retrieving: libgofonoext-1.0.10-1.2.10.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package mesa-llvmpipe-libglapi-9.2.5+git3-1.1.10.jolla.armv7hl                       (156/457),  41.6 KiB (192.8 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libglapi-9.2.5+git3-1.1.10.jolla.armv7hl.rpm ............................................................[done]<br>
Retrieving package dbus-libs-1.10.8+git1-1.1.12.jolla.armv7hl                                   (157/457),  97.1 KiB (192.3 KiB unpacked)<br>
Retrieving: dbus-libs-1.10.8+git1-1.1.12.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package cor-0.1.18-1.2.1.jolla.armv7hl                                               (158/457),  53.8 KiB (142.4 KiB unpacked)<br>
Retrieving: cor-0.1.18-1.2.1.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package p11-kit-nss-ckbi-0.23.12+git1-1.3.5.jolla.armv7hl                              (159/457),   7.7 KiB (    0 B unpacked)<br>
Retrieving: p11-kit-nss-ckbi-0.23.12+git1-1.3.5.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package passwd-0.79+git1-1.3.5.jolla.armv7hl                                         (160/457),  83.3 KiB (389.2 KiB unpacked)<br>
Retrieving: passwd-0.79+git1-1.3.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libuuid-2.33+git1-1.4.5.jolla.armv7hl                                        (161/457),  22.5 KiB ( 20.1 KiB unpacked)<br>
Retrieving: libuuid-2.33+git1-1.4.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package perl-macros-2:5.16.1-1.1.16.jolla.armv7hl                                    (162/457),  10.9 KiB (  5.0 KiB unpacked)<br>
Retrieving: perl-macros-5.16.1-1.1.16.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package sailfish-fonts-0.1.5-1.3.1.jolla.noarch                                      (163/457),  12.1 MiB ( 32.9 MiB unpacked)<br>
Retrieving: sailfish-fonts-0.1.5-1.3.1.jolla.noarch.rpm ...........................................................................[done]<br>
Retrieving package fontconfig-devel-2.12.4-1.2.9.jolla.armv7hl                                  (164/457),  25.9 KiB ( 32.8 KiB unpacked)<br>
Retrieving: fontconfig-devel-2.12.4-1.2.9.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libsndfile-1.0.25-1.2.5.jolla.armv7hl                                        (165/457), 167.6 KiB (425.1 KiB unpacked)<br>
Retrieving: libsndfile-1.0.25-1.2.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package mesa-llvmpipe-libEGL-9.2.5+git3-1.1.10.jolla.armv7hl                         (166/457),  38.9 KiB ( 67.5 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libEGL-9.2.5+git3-1.1.10.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package libdbuslogserver-dbus-1.0.15-1.4.4.jolla.armv7hl                             (167/457),  21.7 KiB ( 30.8 KiB unpacked)<br>
Retrieving: libdbuslogserver-dbus-1.0.15-1.4.4.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package dbus-glib-0.100.2-1.2.7.jolla.armv7hl                                        (168/457),  84.9 KiB (211.6 KiB unpacked)<br>
Retrieving: dbus-glib-0.100.2-1.2.7.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package statefs-pp-0.3.35-1.2.1.jolla.armv7hl                                        (169/457),  29.1 KiB ( 35.6 KiB unpacked)<br>
Retrieving: statefs-pp-0.3.35-1.2.1.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package gnutls-2.12.23.4-1.2.5.jolla.armv7hl                                         (170/457), 301.8 KiB (801.1 KiB unpacked)<br>
Retrieving: gnutls-2.12.23.4-1.2.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package ca-certificates-2018.2.24-1.2.14.jolla.noarch                                (171/457), 342.5 KiB (928.0 KiB unpacked)<br>
Retrieving: ca-certificates-2018.2.24-1.2.14.jolla.noarch.rpm .....................................................................[done]<br>
Retrieving package libblkid-2.33+git1-1.4.5.jolla.armv7hl                                       (172/457), 120.6 KiB (208.3 KiB unpacked)<br>
Retrieving: libblkid-2.33+git1-1.4.5.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package perl-libs-2:5.16.1-1.1.16.jolla.armv7hl                                      (173/457), 497.9 KiB (952.6 KiB unpacked)<br>
Retrieving: perl-libs-5.16.1-1.1.16.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package mesa-llvmpipe-9.2.5+git3-1.1.10.jolla.armv7hl                                (174/457),   5.2 MiB ( 16.3 MiB unpacked)<br>
Retrieving: mesa-llvmpipe-9.2.5+git3-1.1.10.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package libresource-0.23.1+git1-1.2.3.jolla.armv7hl                                  (175/457),  30.8 KiB ( 53.9 KiB unpacked)<br>
Retrieving: libresource-0.23.1+git1-1.2.3.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package glib-networking-2.42.0-1.2.2.jolla.armv7hl                                   (176/457),  82.4 KiB (278.8 KiB unpacked)<br>
Retrieving: glib-networking-2.42.0-1.2.2.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package openssl-libs-1.0.2o+git2-1.4.5.jolla.armv7hl                                 (177/457), 692.3 KiB (  1.5 MiB unpacked)<br>
Retrieving: openssl-libs-1.0.2o+git2-1.4.5.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package libfdisk-2.33+git1-1.4.5.jolla.armv7hl                                       (178/457), 159.4 KiB (300.7 KiB unpacked)<br>
Retrieving: libfdisk-2.33+git1-1.4.5.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package perl-2:5.16.1-1.1.16.jolla.armv7hl                                           (179/457),   8.6 MiB ( 27.1 MiB unpacked)<br>
Retrieving: perl-5.16.1-1.1.16.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package mesa-llvmpipe-libwayland-egl-9.2.5+git3-1.1.10.jolla.armv7hl                 (180/457),  10.3 KiB (  3.5 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libwayland-egl-9.2.5+git3-1.1.10.jolla.armv7hl.rpm ......................................................[done]<br>
Retrieving package mesa-llvmpipe-libGLESv2-9.2.5+git3-1.1.10.jolla.armv7hl                      (181/457),  16.6 KiB ( 31.7 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libGLESv2-9.2.5+git3-1.1.10.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package mesa-llvmpipe-libEGL-devel-9.2.5+git3-1.1.10.jolla.armv7hl                   (182/457),  21.2 KiB ( 60.7 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libEGL-devel-9.2.5+git3-1.1.10.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package libcurl-7.64.0+git1-1.8.5.jolla.armv7hl                                      (183/457), 171.5 KiB (287.3 KiB unpacked)<br>
Retrieving: libcurl-7.64.0+git1-1.8.5.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libarchive-3.3.3+git1-1.2.9.jolla.armv7hl                                    (184/457), 249.4 KiB (471.5 KiB unpacked)<br>
Retrieving: libarchive-3.3.3+git1-1.2.9.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package libmount-2.33+git1-1.4.5.jolla.armv7hl                                       (185/457), 128.9 KiB (235.8 KiB unpacked)<br>
Retrieving: libmount-2.33+git1-1.4.5.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package perl-Scalar-List-Utils-2:1.25-1.1.16.jolla.armv7hl                           (186/457),  31.5 KiB ( 38.4 KiB unpacked)<br>
Retrieving: perl-Scalar-List-Utils-1.25-1.1.16.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package mesa-llvmpipe-libGLESv2-devel-9.2.5+git3-1.1.10.jolla.armv7hl                (187/457),  29.4 KiB (187.0 KiB unpacked)<br>
Retrieving: mesa-llvmpipe-libGLESv2-devel-9.2.5+git3-1.1.10.jolla.armv7hl.rpm .....................................................[done]<br>
Retrieving package gstreamer1.0-plugins-base-1.14.1-1.2.5.jolla.armv7hl                         (188/457),   1.3 MiB (  3.4 MiB unpacked)<br>
Retrieving: gstreamer1.0-plugins-base-1.14.1-1.2.5.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package cairo-1.14.6+git1-1.2.3.jolla.armv7hl                                        (189/457), 399.3 KiB (797.6 KiB unpacked)<br>
Retrieving: cairo-1.14.6+git1-1.2.3.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package pacrunner-0.15+git1-1.3.3.jolla.armv7hl                                      (190/457), 163.9 KiB (318.2 KiB unpacked)<br>
Retrieving: pacrunner-0.15+git1-1.3.3.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package gnupg2-1:2.0.4+git2-1.4.5.jolla.armv7hl                                      (191/457), 949.6 KiB (  4.2 MiB unpacked)<br>
Retrieving: gnupg2-2.0.4+git2-1.4.5.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package curl-7.64.0+git1-1.8.5.jolla.armv7hl                                         (192/457), 107.0 KiB (307.8 KiB unpacked)<br>
Retrieving: curl-7.64.0+git1-1.8.5.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package util-linux-2.33+git1-1.4.5.jolla.armv7hl                                     (193/457), 673.2 KiB (  2.6 MiB unpacked)<br>
Retrieving: util-linux-2.33+git1-1.4.5.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package perl-Pod-Escapes-2:1.04-1.1.16.jolla.noarch                                  (194/457),  18.2 KiB ( 20.6 KiB unpacked)<br>
Retrieving: perl-Pod-Escapes-1.04-1.1.16.jolla.noarch.rpm .........................................................................[done]<br>
Retrieving package nemo-gstreamer1.0-interfaces-0.20150126.0-1.3.3.jolla.armv7hl                (195/457),  12.0 KiB (  9.5 KiB unpacked)<br>
Retrieving: nemo-gstreamer1.0-interfaces-0.20150126.0-1.3.3.jolla.armv7hl.rpm .....................................................[done]<br>
Retrieving package gpgme-1.2.0+git6-1.3.4.jolla.armv7hl                                         (196/457),  91.0 KiB (293.9 KiB unpacked)<br>
Retrieving: gpgme-1.2.0+git6-1.3.4.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package rpm-libs-4.14.1+git9-1.5.7.jolla.armv7hl                                     (197/457), 254.8 KiB (504.9 KiB unpacked)<br>
Retrieving: rpm-libs-4.14.1+git9-1.5.7.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package xdg-utils-1.1.2+git1-1.2.8.jolla.noarch                                      (198/457),  43.8 KiB (257.9 KiB unpacked)<br>
Retrieving: xdg-utils-1.1.2+git1-1.2.8.jolla.noarch.rpm ...........................................................................[done]<br>
Retrieving package fuse-2.9.0+git1-1.3.1.jolla.armv7hl                                          (199/457),  30.2 KiB ( 51.5 KiB unpacked)<br>
Retrieving: fuse-2.9.0+git1-1.3.1.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package cryptsetup-libs-2.1.0+git1-1.3.5.jolla.armv7hl                               (200/457), 298.2 KiB (  1.3 MiB unpacked)<br>
Retrieving: cryptsetup-libs-2.1.0+git1-1.3.5.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package perl-Pod-Simple-2:3.20-1.1.16.jolla.noarch                                   (201/457), 198.4 KiB (488.0 KiB unpacked)<br>
Retrieving: perl-Pod-Simple-3.20-1.1.16.jolla.noarch.rpm ..........................................................................[done]<br>
Retrieving package rpm-4.14.1+git9-1.5.7.jolla.armv7hl                                          (202/457), 331.3 KiB (  1.8 MiB unpacked)<br>
Retrieving: rpm-4.14.1+git9-1.5.7.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package fuse-libs-2.9.0+git1-1.3.1.jolla.armv7hl                                     (203/457),  70.4 KiB (166.2 KiB unpacked)<br>
Retrieving: fuse-libs-2.9.0+git1-1.3.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package systemd-config-sailfish-0.8.15-1.8.1.jolla.noarch                            (204/457),  33.9 KiB (  3.7 KiB unpacked)<br>
Retrieving: systemd-config-sailfish-0.8.15-1.8.1.jolla.noarch.rpm .................................................................[done]<br>
Retrieving package perl-parent-2:0.225-1.1.16.jolla.noarch                                      (205/457),  13.2 KiB (  5.6 KiB unpacked)<br>
Retrieving: perl-parent-0.225-1.1.16.jolla.noarch.rpm .............................................................................[done]<br>
Retrieving package device-mapper-libs-2.02.177+git3-1.3.7.jolla.armv7hl                         (206/457), 124.9 KiB (239.3 KiB unpacked)<br>
Retrieving: device-mapper-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package perl-Pod-Perldoc-2:3.17.00-1.1.16.jolla.noarch                               (207/457),  75.3 KiB (143.0 KiB unpacked)<br>
Retrieving: perl-Pod-Perldoc-3.17.00-1.1.16.jolla.noarch.rpm ......................................................................[done]<br>
Retrieving package device-mapper-event-libs-2.02.177+git3-1.3.7.jolla.armv7hl                   (208/457),  16.5 KiB ( 17.8 KiB unpacked)<br>
Retrieving: device-mapper-event-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package perl-Pod-Parser-2:1.51-1.1.16.jolla.noarch                                   (209/457), 124.6 KiB (305.0 KiB unpacked)<br>
Retrieving: perl-Pod-Parser-1.51-1.1.16.jolla.noarch.rpm ..........................................................................[done]<br>
Retrieving package lvm2-2.02.177+git3-1.3.7.jolla.armv7hl                                       (210/457), 618.2 KiB (  1.6 MiB unpacked)<br>
Retrieving: lvm2-2.02.177+git3-1.3.7.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package perl-Filter-2:1.40-1.1.16.jolla.armv7hl                                      (211/457),  40.2 KiB ( 56.0 KiB unpacked)<br>
Retrieving: perl-Filter-1.40-1.1.16.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package systemd-225+git13-1.4.2.jolla.armv7hl                                        (212/457),   3.8 MiB ( 31.3 MiB unpacked)<br>
Retrieving: systemd-225+git13-1.4.2.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package perl-Module-Pluggable-2:4.00-1.1.16.jolla.noarch                             (213/457),  25.6 KiB ( 30.3 KiB unpacked)<br>
Retrieving: perl-Module-Pluggable-4.00-1.1.16.jolla.noarch.rpm ....................................................................[done]<br>
Retrieving package dbus-1.10.8+git1-1.1.12.jolla.armv7hl                                        (214/457), 113.1 KiB (278.0 KiB unpacked)<br>
Retrieving: dbus-1.10.8+git1-1.1.12.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package perl-threads-2:1.86-1.1.16.jolla.armv7hl                                     (215/457),  45.2 KiB ( 72.0 KiB unpacked)<br>
Retrieving: perl-threads-1.86-1.1.16.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package polkit-0.105+git2-1.2.3.jolla.armv7hl                                        (216/457), 103.1 KiB (268.9 KiB unpacked)<br>
Retrieving: polkit-0.105+git2-1.2.3.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package device-mapper-2.02.177+git3-1.3.7.jolla.armv7hl                              (217/457),  67.8 KiB (160.9 KiB unpacked)<br>
Retrieving: device-mapper-2.02.177+git3-1.3.7.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package dbus-devel-1.10.8+git1-1.1.12.jolla.armv7hl                                  (218/457),  30.2 KiB (116.4 KiB unpacked)<br>
Retrieving: dbus-devel-1.10.8+git1-1.1.12.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package perl-Socket-2:2.001-1.1.16.jolla.armv7hl                                     (219/457),  37.6 KiB ( 66.8 KiB unpacked)<br>
Retrieving: perl-Socket-2.001-1.1.16.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package device-mapper-event-2.02.177+git3-1.3.7.jolla.armv7hl                        (220/457),  22.0 KiB ( 29.6 KiB unpacked)<br>
Retrieving: device-mapper-event-2.02.177+git3-1.3.7.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package perl-threads-shared-2:1.40-1.1.16.jolla.armv7hl                              (221/457),  34.5 KiB ( 50.6 KiB unpacked)<br>
Retrieving: perl-threads-shared-1.40-1.1.16.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package lvm2-libs-2.02.177+git3-1.3.7.jolla.armv7hl                                  (222/457), 670.2 KiB (  2.3 MiB unpacked)<br>
Retrieving: lvm2-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package qt5-qmake-5.6.3+git9-1.9.2.jolla.armv7hl                                     (223/457),   1.1 MiB (  2.7 MiB unpacked)<br>
Retrieving: qt5-qmake-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package meego-rpm-config-0.18-1.2.7.jolla.noarch                                     (224/457),  49.4 KiB (132.3 KiB unpacked)<br>
Retrieving: meego-rpm-config-0.18-1.2.7.jolla.noarch.rpm ..........................................................................[done]<br>
Retrieving package wpa_supplicant-2.6+git5-1.3.5.jolla.armv7hl                                  (225/457), 483.4 KiB (  1.0 MiB unpacked)<br>
Retrieving: wpa_supplicant-2.6+git5-1.3.5.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package usb-moded-0.86.0+mer33-1.3.2.jolla.armv7hl                                   (226/457), 100.8 KiB (138.3 KiB unpacked)<br>
Retrieving: usb-moded-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch                        (227/457),   8.7 KiB (  1.3 KiB unpacked)<br>
Retrieving: systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch.rpm .............................................................[done]<br>
Retrieving package pulseaudio-12.2+git1-1.7.2.jolla.armv7hl                                     (228/457),   1.3 MiB (  5.6 MiB unpacked)<br>
Retrieving: pulseaudio-12.2+git1-1.7.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package ofono-1.21+git44-1.19.1.jolla.armv7hl                                        (229/457), 443.3 KiB (834.2 KiB unpacked)<br>
Retrieving: ofono-1.21+git44-1.19.1.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package oneshot-0.4.8-1.2.8.jolla.noarch                                             (230/457),  15.2 KiB (  8.3 KiB unpacked)<br>
Retrieving: oneshot-0.4.8-1.2.8.jolla.noarch.rpm ..................................................................................[done]<br>
Retrieving package mapplauncherd-4.1.30-1.6.1.jolla.armv7hl                                     (231/457),  53.4 KiB ( 87.9 KiB unpacked)<br>
Retrieving: mapplauncherd-4.1.30-1.6.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libcanberra-0.30+git1-1.4.1.jolla.armv7hl                                    (232/457),  48.4 KiB (103.5 KiB unpacked)<br>
Retrieving: libcanberra-0.30+git1-1.4.1.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package connman-configs-mer-1.32+git65-1.25.2.jolla.armv7hl                            (233/457),  62.4 KiB (   94 B unpacked)<br>
Retrieving: connman-configs-mer-1.32+git65-1.25.2.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package ffmpeg-4.1.1+git1-1.2.1.jolla.armv7hl                                        (234/457),   3.4 MiB (  7.9 MiB unpacked)<br>
Retrieving: ffmpeg-4.1.1+git1-1.2.1.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package dconf-0.28.0-1.1.2.jolla.armv7hl                                             (235/457),  56.5 KiB (146.1 KiB unpacked)<br>
Retrieving: dconf-0.28.0-1.1.2.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package boost-system-1.66.0-1.3.8.jolla.armv7hl                                      (236/457),  18.8 KiB ( 23.9 KiB unpacked)<br>
Retrieving: boost-system-1.66.0-1.3.8.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package connman-1.32+git65-1.25.2.jolla.armv7hl                                      (237/457), 417.5 KiB (874.1 KiB unpacked)<br>
Retrieving: connman-1.32+git65-1.25.2.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libicu-63.1+git5-1.1.6.jolla.armv7hl                                         (238/457),   7.5 MiB ( 29.1 MiB unpacked)<br>
Retrieving: libicu-63.1+git5-1.1.6.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package jolla-ambient-sound-theme-0.0.17-1.2.1.jolla.noarch                          (239/457), 774.7 KiB (  1.8 MiB unpacked)<br>
Retrieving: jolla-ambient-sound-theme-0.0.17-1.2.1.jolla.noarch.rpm ...............................................................[done]<br>
Retrieving package boost-filesystem-1.66.0-1.3.8.jolla.armv7hl                                  (240/457),  46.4 KiB (217.3 KiB unpacked)<br>
Retrieving: boost-filesystem-1.66.0-1.3.8.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package sqlite-libs-3.13.0+git3-1.3.6.jolla.armv7hl                                  (241/457), 381.7 KiB (591.0 KiB unpacked)<br>
Retrieving: sqlite-libs-3.13.0+git3-1.3.6.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package qt5-qtcore-5.6.3+git9-1.9.2.jolla.armv7hl                                    (242/457),   1.4 MiB (  3.4 MiB unpacked)<br>
Retrieving: qt5-qtcore-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package statefs-0.3.35-1.2.1.jolla.armv7hl                                           (243/457), 145.0 KiB (497.1 KiB unpacked)<br>
Retrieving: statefs-0.3.35-1.2.1.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package nss-3.39-1.3.6.jolla.armv7hl                                                 (244/457), 746.1 KiB (  1.7 MiB unpacked)<br>
Retrieving: nss-3.39-1.3.6.jolla.armv7hl.rpm ......................................................................................[done]<br>
Retrieving package qt5-qtconcurrent-5.6.3+git9-1.9.2.jolla.armv7hl                              (245/457),  18.6 KiB ( 16.4 KiB unpacked)<br>
Retrieving: qt5-qtconcurrent-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch                         (246/457),  19.0 KiB (  795 B unpacked)<br>
Retrieving: statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch.rpm ............................................................[done]<br>
Retrieving package boost-thread-1.66.0-1.3.8.jolla.armv7hl                                      (247/457),  66.7 KiB (274.2 KiB unpacked)<br>
Retrieving: boost-thread-1.66.0-1.3.8.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package nss-sysinit-3.39-1.3.6.jolla.armv7hl                                         (248/457),  15.7 KiB ( 11.0 KiB unpacked)<br>
Retrieving: nss-sysinit-3.39-1.3.6.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package qt5-qtdbus-5.6.3+git9-1.9.2.jolla.armv7hl                                    (249/457), 141.9 KiB (351.3 KiB unpacked)<br>
Retrieving: qt5-qtdbus-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package nss-devel-3.39-1.3.6.jolla.armv7hl                                           (250/457), 220.6 KiB (  1.1 MiB unpacked)<br>
Retrieving: nss-devel-3.39-1.3.6.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package qt5-qtgui-5.6.3+git9-1.9.2.jolla.armv7hl                                     (251/457),   1.4 MiB (  3.6 MiB unpacked)<br>
Retrieving: qt5-qtgui-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package nss-softokn-freebl-3.39-1.3.6.jolla.armv7hl                                  (252/457), 202.5 KiB (671.2 KiB unpacked)<br>
Retrieving: nss-softokn-freebl-3.39-1.3.6.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package qt5-qtnetwork-5.6.3+git9-1.9.2.jolla.armv7hl                                 (253/457), 357.4 KiB (962.8 KiB unpacked)<br>
Retrieving: qt5-qtnetwork-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package python-2.7.15+git2-1.3.7.jolla.armv7hl                                       (254/457),  17.7 KiB ( 16.1 KiB unpacked)<br>
Retrieving: python-2.7.15+git2-1.3.7.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package poppler-0.74.0+git1-1.5.1.jolla.armv7hl                                      (255/457), 770.2 KiB (  2.1 MiB unpacked)<br>
Retrieving: poppler-0.74.0+git1-1.5.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libsoup-2.54.1+git3-1.2.4.jolla.armv7hl                                      (256/457), 255.4 KiB (851.2 KiB unpacked)<br>
Retrieving: libsoup-2.54.1+git3-1.2.4.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libsolv0-0.6.35+git2-1.4.5.jolla.armv7hl                                     (257/457), 290.0 KiB (513.9 KiB unpacked)<br>
Retrieving: libsolv0-0.6.35+git2-1.4.5.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libaccounts-glib-1.18+git1-1.1.8.jolla.armv7hl                               (258/457),  42.9 KiB ( 79.1 KiB unpacked)<br>
Retrieving: libaccounts-glib-1.18+git1-1.1.8.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package PackageKit-glib-1.1.9+git5-1.8.1.jolla.armv7hl                               (259/457),  98.5 KiB (240.6 KiB unpacked)<br>
Retrieving: PackageKit-glib-1.1.9+git5-1.8.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtwidgets-5.6.3+git9-1.9.2.jolla.armv7hl                                 (260/457),   1.7 MiB (  4.1 MiB unpacked)<br>
Retrieving: qt5-qtwidgets-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package python-libs-2.7.15+git2-1.3.7.jolla.armv7hl                                  (261/457),   6.5 MiB ( 25.3 MiB unpacked)<br>
Retrieving: python-libs-2.7.15+git2-1.3.7.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package poppler-glib-0.74.0+git1-1.5.1.jolla.armv7hl                                 (262/457), 103.8 KiB (296.1 KiB unpacked)<br>
Retrieving: poppler-glib-0.74.0+git1-1.5.1.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package totem-pl-parser-3.26.1+git1-1.4.1.jolla.armv7hl                              (263/457), 146.8 KiB (503.3 KiB unpacked)<br>
Retrieving: totem-pl-parser-3.26.1+git1-1.4.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package gstreamer1.0-plugins-good-1.14.1-1.2.8.jolla.armv7hl                         (264/457),   1.3 MiB (  3.2 MiB unpacked)<br>
Retrieving: gstreamer1.0-plugins-good-1.14.1-1.2.8.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package libsolv-tools-0.6.35+git2-1.4.5.jolla.armv7hl                                (265/457),  46.3 KiB (144.5 KiB unpacked)<br>
Retrieving: libsolv-tools-0.6.35+git2-1.4.5.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package qt5-qtopengl-5.6.3+git9-1.9.2.jolla.armv7hl                                  (266/457),  91.2 KiB (232.1 KiB unpacked)<br>
Retrieving: qt5-qtopengl-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package rpm-build-4.14.1+git9-1.5.7.jolla.armv7hl                                    (267/457), 105.1 KiB (250.6 KiB unpacked)<br>
Retrieving: rpm-build-4.14.1+git9-1.5.7.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package ngfd-settings-sailfish-0.8.15-1.8.1.jolla.noarch                             (268/457),  41.4 KiB ( 25.8 KiB unpacked)<br>
Retrieving: ngfd-settings-sailfish-0.8.15-1.8.1.jolla.noarch.rpm ..................................................................[done]<br>
Retrieving package qt5-qtsql-5.6.3+git9-1.9.2.jolla.armv7hl                                     (269/457),  78.2 KiB (180.6 KiB unpacked)<br>
Retrieving: qt5-qtsql-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package qt5-tools-5.6.3+git9-1.9.2.jolla.armv7hl                                     (270/457), 671.9 KiB (  2.3 MiB unpacked)<br>
Retrieving: qt5-tools-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package qt5-qtxmlpatterns-5.6.3+git2-1.2.1.jolla.armv7hl                             (271/457), 776.1 KiB (  2.5 MiB unpacked)<br>
Retrieving: qt5-qtxmlpatterns-5.6.3+git2-1.2.1.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package qt5-qtxml-5.6.3+git9-1.9.2.jolla.armv7hl                                     (272/457),  68.3 KiB (149.6 KiB unpacked)<br>
Retrieving: qt5-qtxml-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package qt5-qttest-5.6.3+git9-1.9.2.jolla.armv7hl                                    (273/457),  69.8 KiB (159.5 KiB unpacked)<br>
Retrieving: qt5-qttest-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package qt5-qtsvg-5.6.2+git1-1.2.1.jolla.armv7hl                                     (274/457),  84.8 KiB (199.3 KiB unpacked)<br>
Retrieving: qt5-qtsvg-5.6.2+git1-1.2.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package qt5-qtserviceframework-5.2.0+git9-1.2.1.jolla.armv7hl                        (275/457), 123.0 KiB (317.1 KiB unpacked)<br>
Retrieving: qt5-qtserviceframework-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtsensors-5.2.1+git17-1.3.1.jolla.armv7hl                                (276/457),  52.2 KiB (153.3 KiB unpacked)<br>
Retrieving: qt5-qtsensors-5.2.1+git17-1.3.1.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package qt5-qtpublishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl                        (277/457),  21.3 KiB ( 28.7 KiB unpacked)<br>
Retrieving: qt5-qtpublishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtprintsupport-5.6.3+git9-1.9.2.jolla.armv7hl                            (278/457), 126.6 KiB (311.9 KiB unpacked)<br>
Retrieving: qt5-qtprintsupport-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package qt5-qtpositioning-5.2.1+git31-1.4.1.jolla.armv7hl                            (279/457),  64.9 KiB (157.1 KiB unpacked)<br>
Retrieving: qt5-qtpositioning-5.2.1+git31-1.4.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package qt5-qtpim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl                           (280/457), 119.0 KiB (426.8 KiB unpacked)<br>
Retrieving: qt5-qtpim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ................................................................[done]<br>
Retrieving package qt5-qtpim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl                            (281/457), 109.3 KiB (356.2 KiB unpacked)<br>
Retrieving: qt5-qtpim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package qt5-plugin-sqldriver-sqlite-5.6.3+git9-1.9.2.jolla.armv7hl                   (282/457),  24.9 KiB ( 37.6 KiB unpacked)<br>
Retrieving: qt5-plugin-sqldriver-sqlite-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package qt5-plugin-platform-minimal-5.6.3+git9-1.9.2.jolla.armv7hl                   (283/457),  46.8 KiB ( 91.4 KiB unpacked)<br>
Retrieving: qt5-plugin-platform-minimal-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package qt5-plugin-imageformat-jpeg-5.6.3+git9-1.9.2.jolla.armv7hl                   (284/457),  24.3 KiB ( 32.0 KiB unpacked)<br>
Retrieving: qt5-plugin-imageformat-jpeg-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package qt5-plugin-bearer-generic-5.6.3+git9-1.9.2.jolla.armv7hl                     (285/457),  29.2 KiB ( 55.3 KiB unpacked)<br>
Retrieving: qt5-plugin-bearer-generic-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package mlite-qt5-0.2.25-1.4.1.jolla.armv7hl                                         (286/457),  80.2 KiB (199.5 KiB unpacked)<br>
Retrieving: mlite-qt5-0.2.25-1.4.1.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libusb-moded-qt5-1.8-1.4.1.jolla.armv7hl                                     (287/457),  30.3 KiB ( 71.0 KiB unpacked)<br>
Retrieving: libusb-moded-qt5-1.8-1.4.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libqtaround2-0.2.8-1.2.1.jolla.armv7hl                                       (288/457),  68.9 KiB (165.0 KiB unpacked)<br>
Retrieving: libqtaround2-0.2.8-1.2.1.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package libmediaart-1.9.4-1.4.1.jolla.armv7hl                                        (289/457),  27.2 KiB ( 43.9 KiB unpacked)<br>
Retrieving: libmediaart-1.9.4-1.4.1.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package libmce-qt5-1.1.1-1.4.1.jolla.armv7hl                                         (290/457),  39.8 KiB (129.1 KiB unpacked)<br>
Retrieving: libmce-qt5-1.1.1-1.4.1.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libkf5archive-5.41.0+git4-1.4.1.jolla.armv7hl                                (291/457),  82.3 KiB (196.4 KiB unpacked)<br>
Retrieving: libkf5archive-5.41.0+git4-1.4.1.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package libiodata-qt5-0.19.10-1.3.1.jolla.armv7hl                                    (292/457),  58.9 KiB (122.9 KiB unpacked)<br>
Retrieving: libiodata-qt5-0.19.10-1.3.1.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package libdbus-qeventloop-qt5-1.30.3-1.4.1.jolla.armv7hl                            (293/457),  17.3 KiB ( 18.2 KiB unpacked)<br>
Retrieving: libdbus-qeventloop-qt5-1.30.3-1.4.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package dbusextended-qt5-0.0.3-1.3.1.jolla.armv7hl                                   (294/457),  30.9 KiB ( 67.1 KiB unpacked)<br>
Retrieving: dbusextended-qt5-0.0.3-1.3.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package connman-qt5-1.2.16-1.10.1.jolla.armv7hl                                      (295/457), 126.7 KiB (324.9 KiB unpacked)<br>
Retrieving: connman-qt5-1.2.16-1.10.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package qt5-qtcore-devel-5.6.3+git9-1.9.2.jolla.armv7hl                              (296/457), 601.5 KiB (  3.9 MiB unpacked)<br>
Retrieving: qt5-qtcore-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package poppler-qt5-0.74.0+git1-1.5.1.jolla.armv7hl                                  (297/457), 133.3 KiB (347.6 KiB unpacked)<br>
Retrieving: poppler-qt5-0.74.0+git1-1.5.1.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libaccounts-qt5-1.13+git1-1.3.1.jolla.armv7hl                                (298/457),  38.4 KiB ( 87.6 KiB unpacked)<br>
Retrieving: libaccounts-qt5-1.13+git1-1.3.1.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package qt5-qtdeclarative-5.6.3+git7-1.5.1.jolla.armv7hl                             (299/457), 958.7 KiB (  2.5 MiB unpacked)<br>
Retrieving: qt5-qtdeclarative-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package qt5-qttools-linguist-5.6.3+git1-1.4.1.jolla.armv7hl                          (300/457), 653.6 KiB (  2.3 MiB unpacked)<br>
Retrieving: qt5-qttools-linguist-5.6.3+git1-1.4.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package qt5-qtpim-versit-5.2.0+git2-1.2.1.jolla.armv7hl                              (301/457),  93.5 KiB (258.0 KiB unpacked)<br>
Retrieving: qt5-qtpim-versit-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package libquillmetadata-qt5-1.111111.10-1.5.1.jolla.armv7hl                         (302/457),  44.6 KiB ( 95.2 KiB unpacked)<br>
Retrieving: libquillmetadata-qt5-1.111111.10-1.5.1.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package sailfish-silica-background-qt5-0.9.14-1.4.1.jolla.armv7hl                    (303/457),  24.4 KiB ( 29.5 KiB unpacked)<br>
Retrieving: sailfish-silica-background-qt5-0.9.14-1.4.1.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package statefs-qt5-0.3.5-1.4.1.jolla.armv7hl                                        (304/457),  27.4 KiB ( 46.2 KiB unpacked)<br>
Retrieving: statefs-qt5-0.3.5-1.4.1.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package tracker-1.12.4+git9-1.7.1.jolla.armv7hl                                      (305/457), 981.0 KiB (  4.3 MiB unpacked)<br>
Retrieving: tracker-1.12.4+git9-1.7.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libresourceqt-qt5-1.30.3-1.4.1.jolla.armv7hl                                 (306/457),  38.7 KiB ( 91.7 KiB unpacked)<br>
Retrieving: libresourceqt-qt5-1.30.3-1.4.1.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.armv7hl                         (307/457),  25.3 KiB (  3.8 KiB unpacked)<br>
Retrieving: ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package qt5-qtxml-devel-5.6.3+git9-1.9.2.jolla.armv7hl                               (308/457),  26.1 KiB ( 49.2 KiB unpacked)<br>
Retrieving: qt5-qtxml-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtsql-devel-5.6.3+git9-1.9.2.jolla.armv7hl                               (309/457),  31.7 KiB (113.4 KiB unpacked)<br>
Retrieving: qt5-qtsql-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtserviceframework-devel-5.2.0+git9-1.2.1.jolla.armv7hl                  (310/457),  32.2 KiB (135.8 KiB unpacked)<br>
Retrieving: qt5-qtserviceframework-devel-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .......................................................[done]<br>
Retrieving package qt5-qtsensors-devel-5.2.1+git17-1.3.1.jolla.armv7hl                          (311/457),  38.9 KiB (148.9 KiB unpacked)<br>
Retrieving: qt5-qtsensors-devel-5.2.1+git17-1.3.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package qt5-qtpublishsubscribe-devel-5.2.0+git9-1.2.1.jolla.armv7hl                  (312/457),  19.9 KiB ( 48.4 KiB unpacked)<br>
Retrieving: qt5-qtpublishsubscribe-devel-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .......................................................[done]<br>
Retrieving package qt5-qtpositioning-devel-5.2.1+git31-1.4.1.jolla.armv7hl                      (313/457),  30.9 KiB (122.5 KiB unpacked)<br>
Retrieving: qt5-qtpositioning-devel-5.2.1+git31-1.4.1.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package qt5-qtpim-organizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl                     (314/457),  55.8 KiB (338.5 KiB unpacked)<br>
Retrieving: qt5-qtpim-organizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package qt5-qtpim-contacts-devel-5.2.0+git2-1.2.1.jolla.armv7hl                      (315/457),  57.6 KiB (338.7 KiB unpacked)<br>
Retrieving: qt5-qtpim-contacts-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package qt5-qtnetwork-devel-5.6.3+git9-1.9.2.jolla.armv7hl                           (316/457),  87.6 KiB (491.3 KiB unpacked)<br>
Retrieving: qt5-qtnetwork-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ................................................................[done]<br>
Retrieving package qt5-qtgui-devel-5.6.3+git9-1.9.2.jolla.armv7hl                               (317/457), 458.0 KiB (  6.5 MiB unpacked)<br>
Retrieving: qt5-qtgui-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtdbus-devel-5.6.3+git9-1.9.2.jolla.armv7hl                              (318/457), 136.7 KiB (423.8 KiB unpacked)<br>
Retrieving: qt5-qtdbus-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package qt5-qtconcurrent-devel-5.6.3+git9-1.9.2.jolla.armv7hl                        (319/457),  30.7 KiB (187.0 KiB unpacked)<br>
Retrieving: qt5-qtconcurrent-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtdeclarative-qtquick-5.6.3+git7-1.5.1.jolla.armv7hl                     (320/457), 827.0 KiB (  2.4 MiB unpacked)<br>
Retrieving: qt5-qtdeclarative-qtquick-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package qt5-qtpim-versitorganizer-5.2.0+git2-1.2.1.jolla.armv7hl                     (321/457),  50.9 KiB (120.9 KiB unpacked)<br>
Retrieving: qt5-qtpim-versitorganizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package timed-qt5-3.5.1-1.4.1.jolla.armv7hl                                          (322/457), 286.0 KiB (772.7 KiB unpacked)<br>
Retrieving: timed-qt5-3.5.1-1.4.1.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.armv7hl                      (323/457),  48.2 KiB (111.9 KiB unpacked)<br>
Retrieving: statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package libmlocale-qt5-0.7.0-1.3.1.jolla.armv7hl                                     (324/457), 117.0 KiB (356.0 KiB unpacked)<br>
Retrieving: libmlocale-qt5-0.7.0-1.3.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package libzypp-17.3.1+git4-1.6.2.jolla.armv7hl                                      (325/457),   1.6 MiB (  5.6 MiB unpacked)<br>
Retrieving: libzypp-17.3.1+git4-1.6.2.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libaccounts-qt5-devel-1.13+git1-1.3.1.jolla.armv7hl                          (326/457),  18.4 KiB ( 30.5 KiB unpacked)<br>
Retrieving: libaccounts-qt5-devel-1.13+git1-1.3.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package qt5-qtpim-versit-devel-5.2.0+git2-1.2.1.jolla.armv7hl                        (327/457),  29.8 KiB (117.2 KiB unpacked)<br>
Retrieving: qt5-qtpim-versit-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtxmlpatterns-devel-5.6.3+git2-1.2.1.jolla.armv7hl                       (328/457), 266.4 KiB (  2.2 MiB unpacked)<br>
Retrieving: qt5-qtxmlpatterns-devel-5.6.3+git2-1.2.1.jolla.armv7hl.rpm ............................................................[done]<br>
Retrieving package qt5-qtwidgets-devel-5.6.3+git9-1.9.2.jolla.armv7hl                           (329/457), 270.2 KiB (  1.6 MiB unpacked)<br>
Retrieving: qt5-qtwidgets-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm ................................................................[done]<br>
Retrieving package qt5-qtsvg-devel-5.6.2+git1-1.2.1.jolla.armv7hl                               (330/457),  24.6 KiB ( 77.1 KiB unpacked)<br>
Retrieving: qt5-qtsvg-devel-5.6.2+git1-1.2.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package mlite-qt5-devel-0.2.25-1.4.1.jolla.armv7hl                                   (331/457),  23.9 KiB ( 48.4 KiB unpacked)<br>
Retrieving: mlite-qt5-devel-0.2.25-1.4.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package qt5-qtdeclarative-qtquickparticles-5.6.3+git7-1.5.1.jolla.armv7hl            (332/457), 128.7 KiB (348.5 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-qtquickparticles-5.6.3+git7-1.5.1.jolla.armv7hl.rpm .................................................[done]<br>
Retrieving package PackageKit-1.1.9+git5-1.8.1.jolla.armv7hl                                    (333/457), 378.2 KiB (  1.9 MiB unpacked)<br>
Retrieving: PackageKit-1.1.9+git5-1.8.1.jolla.armv7hl.rpm .........................................................................[done]<br>
Retrieving package qt5-qtpim-versitorganizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl               (334/457),  21.5 KiB ( 61.8 KiB unpacked)<br>
Retrieving: qt5-qtpim-versitorganizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package qt5-qtopengl-devel-5.6.3+git9-1.9.2.jolla.armv7hl                            (335/457), 114.7 KiB (  1.2 MiB unpacked)<br>
Retrieving: qt5-qtopengl-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package qt5-qtdeclarative-qtquicktest-5.6.3+git7-1.5.1.jolla.armv7hl                 (336/457),  41.4 KiB ( 85.5 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-qtquicktest-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ......................................................[done]<br>
Retrieving package PackageKit-zypp-1.1.9+git5-1.8.1.jolla.armv7hl                               (337/457),  92.2 KiB (196.6 KiB unpacked)<br>
Retrieving: PackageKit-zypp-1.1.9+git5-1.8.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package sailfish-content-profiled-settings-default-0.15.3-1.3.1.jolla.noarch         (338/457),  11.7 KiB (  3.1 KiB unpacked)<br>
Retrieving: sailfish-content-profiled-settings-default-0.15.3-1.3.1.jolla.noarch.rpm ..............................................[done]<br>
Retrieving package qt5-qtwayland-wayland_egl-5.4.0+git48-1.3.1.jolla.armv7hl                    (339/457), 279.1 KiB (  1.0 MiB unpacked)<br>
Retrieving: qt5-qtwayland-wayland_egl-5.4.0+git48-1.3.1.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package qt5-qtquickcontrols-layouts-5.2.1+git3-1.2.1.jolla.armv7hl                   (340/457),  43.1 KiB (101.4 KiB unpacked)<br>
Retrieving: qt5-qtquickcontrols-layouts-5.2.1+git3-1.2.1.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package qt5-qtmultimedia-5.6.2+git8-1.4.1.jolla.armv7hl                              (341/457), 237.4 KiB (809.7 KiB unpacked)<br>
Retrieving: qt5-qtmultimedia-5.6.2+git8-1.4.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package qt5-qtlocation-5.2.1+git31-1.4.1.jolla.armv7hl                               (342/457), 120.6 KiB (372.8 KiB unpacked)<br>
Retrieving: qt5-qtlocation-5.2.1+git31-1.4.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtgraphicaleffects-5.6.2+git2-1.2.1.jolla.armv7hl                        (343/457),  51.6 KiB (360.3 KiB unpacked)<br>
Retrieving: qt5-qtgraphicaleffects-5.6.2+git2-1.2.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtfeedback-5.2.0+git6-1.3.1.jolla.armv7hl                                (344/457),  32.8 KiB ( 78.6 KiB unpacked)<br>
Retrieving: qt5-qtfeedback-5.2.0+git6-1.3.1.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package qt5-qtdocgallery-5.2.0+git8-1.4.1.jolla.armv7hl                              (345/457), 107.1 KiB (358.0 KiB unpacked)<br>
Retrieving: qt5-qtdocgallery-5.2.0+git8-1.4.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package qt5-qtdeclarative-serviceframework-5.2.0+git9-1.2.1.jolla.armv7hl            (346/457),  30.7 KiB ( 62.2 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-serviceframework-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .................................................[done]<br>
Retrieving package qt5-qtdeclarative-qtquickparticles-devel-5.6.3+git7-1.5.1.jolla.armv7hl      (347/457),  30.3 KiB (131.9 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-qtquickparticles-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ...........................................[done]<br>
Retrieving package qt5-qtdeclarative-publishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl            (348/457),  19.4 KiB ( 25.6 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-publishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .................................................[done]<br>
Retrieving package qt5-qtdeclarative-plugin-qmlinspector-5.6.3+git7-1.5.1.jolla.armv7hl         (349/457), 125.8 KiB (354.7 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-plugin-qmlinspector-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ..............................................[done]<br>
Retrieving package qt5-qtdeclarative-pim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl               (350/457),  85.5 KiB (327.5 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-pim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package qt5-qtdeclarative-pim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl                (351/457),  90.9 KiB (348.9 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-pim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl.rpm .....................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-xmllistmodel-5.6.3+git7-1.5.1.jolla.armv7hl         (352/457),  33.3 KiB ( 71.8 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-xmllistmodel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ..............................................[done]<br>
Retrieving package qt5-qtdeclarative-import-window2-5.6.3+git7-1.5.1.jolla.armv7hl              (353/457),  14.3 KiB ( 17.0 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-window2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ...................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-sensors-5.2.1+git17-1.3.1.jolla.armv7hl             (354/457),  39.0 KiB (137.8 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-sensors-5.2.1+git17-1.3.1.jolla.armv7hl.rpm ..................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-qttest-5.6.3+git7-1.5.1.jolla.armv7hl               (355/457),  30.1 KiB ( 96.0 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-qttest-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-qtquick2plugin-5.6.3+git7-1.5.1.jolla.armv7hl       (356/457),  26.7 KiB (176.0 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-qtquick2plugin-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ............................................[done]<br>
Retrieving package qt5-qtdeclarative-import-positioning-5.2.1+git31-1.4.1.jolla.armv7hl         (357/457),  30.1 KiB ( 62.9 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-positioning-5.2.1+git31-1.4.1.jolla.armv7hl.rpm ..............................................[done]<br>
Retrieving package qt5-qtdeclarative-import-particles2-5.6.3+git7-1.5.1.jolla.armv7hl           (358/457),  16.7 KiB ( 54.1 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-particles2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-models2-5.6.3+git7-1.5.1.jolla.armv7hl              (359/457),  14.6 KiB ( 27.8 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-models2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ...................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-localstorageplugin-5.6.3+git7-1.5.1.jolla.armv7hl   (360/457),  24.9 KiB ( 39.8 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-localstorageplugin-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ........................................[done]<br>
Retrieving package qt5-qtdeclarative-import-folderlistmodel-5.6.3+git7-1.5.1.jolla.armv7hl      (361/457),  26.1 KiB ( 55.6 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-folderlistmodel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ...........................................[done]<br>
Retrieving package qt5-qtdeclarative-devel-5.6.3+git7-1.5.1.jolla.armv7hl                       (362/457), 220.2 KiB (  1.5 MiB unpacked)<br>
Retrieving: qt5-qtdeclarative-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ............................................................[done]<br>
Retrieving package qt5-qtconnectivity-qtbluetooth-5.6.2+git0-1.2.1.jolla.armv7hl                (363/457), 199.1 KiB (545.2 KiB unpacked)<br>
Retrieving: qt5-qtconnectivity-qtbluetooth-5.6.2+git0-1.2.1.jolla.armv7hl.rpm .....................................................[done]<br>
Retrieving package nemo-qml-plugin-thumbnailer-qt5-1.0.0-1.6.1.jolla.armv7hl                    (364/457),  40.9 KiB ( 74.0 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-thumbnailer-qt5-1.0.0-1.6.1.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package nemo-qml-plugin-notifications-qt5-1.1.6-1.5.1.jolla.armv7hl                  (365/457),  49.5 KiB ( 99.0 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-notifications-qt5-1.1.6-1.5.1.jolla.armv7hl.rpm .......................................................[done]<br>
Retrieving package nemo-qml-plugin-models-qt5-0.1.3-1.4.1.jolla.armv7hl                         (366/457),  78.0 KiB (189.6 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-models-qt5-0.1.3-1.4.1.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package nemo-qml-plugin-filemanager-0.1.11-1.7.1.jolla.armv7hl                       (367/457),  81.6 KiB (222.8 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-filemanager-0.1.11-1.7.1.jolla.armv7hl.rpm ............................................................[done]<br>
Retrieving package nemo-qml-plugin-dbus-qt5-2.1.20-1.6.1.jolla.armv7hl                          (368/457),  78.7 KiB (189.0 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-dbus-qt5-2.1.20-1.6.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package nemo-qml-plugin-configuration-qt5-0.2.2-1.3.1.jolla.armv7hl                  (369/457),  21.3 KiB ( 27.7 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-configuration-qt5-0.2.2-1.3.1.jolla.armv7hl.rpm .......................................................[done]<br>
Retrieving package mpris-qt5-1.0.0-1.4.1.jolla.armv7hl                                          (370/457),  76.3 KiB (233.4 KiB unpacked)<br>
Retrieving: mpris-qt5-1.0.0-1.4.1.jolla.armv7hl.rpm ...............................................................................[done]<br>
Retrieving package mapplauncherd-qt5-1.1.14-1.3.1.jolla.armv7hl                                 (371/457),  19.3 KiB ( 20.0 KiB unpacked)<br>
Retrieving: mapplauncherd-qt5-1.1.14-1.3.1.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package libqtwebkit5-5.6.2+git8-1.5.1.jolla.armv7hl                                  (372/457),   7.1 MiB ( 23.3 MiB unpacked)<br>
Retrieving: libqtwebkit5-5.6.2+git8-1.5.1.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libqt5sparql-0.2.18-1.2.1.jolla.armv7hl                                      (373/457),  84.0 KiB (206.8 KiB unpacked)<br>
Retrieving: libqt5sparql-0.2.18-1.2.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libpython3_7m1_0-3.7.2+git2-1.3.5.jolla.armv7hl                              (374/457), 732.0 KiB (  1.8 MiB unpacked)<br>
Retrieving: libpython3_7m1_0-3.7.2+git2-1.3.5.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package contextkit-declarative-qt5-0.3.5-1.4.1.jolla.armv7hl                         (375/457),  19.0 KiB ( 21.2 KiB unpacked)<br>
Retrieving: contextkit-declarative-qt5-0.3.5-1.4.1.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package ssu-0.43.12-1.8.1.jolla.armv7hl                                              (376/457), 161.8 KiB (424.8 KiB unpacked)<br>
Retrieving: ssu-0.43.12-1.8.1.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package PackageKit-Qt5-0.9.6+git-1.3.1.jolla.armv7hl                                 (377/457),  69.8 KiB (181.9 KiB unpacked)<br>
Retrieving: PackageKit-Qt5-0.9.6+git-1.3.1.jolla.armv7hl.rpm ......................................................................[done]<br>
Retrieving package profiled-1.0.6-1.2.1.jolla.armv7hl                                           (378/457),  27.8 KiB ( 40.9 KiB unpacked)<br>
Retrieving: profiled-1.0.6-1.2.1.jolla.armv7hl.rpm ................................................................................[done]<br>
Retrieving package qt5-qtwayland-wayland_egl-devel-5.4.0+git48-1.3.1.jolla.armv7hl              (379/457), 127.1 KiB (815.3 KiB unpacked)<br>
Retrieving: qt5-qtwayland-wayland_egl-devel-5.4.0+git48-1.3.1.jolla.armv7hl.rpm ...................................................[done]<br>
Retrieving package qt5-qtmultimedia-gsttools-5.6.2+git8-1.4.1.jolla.armv7hl                     (380/457),  52.6 KiB (143.4 KiB unpacked)<br>
Retrieving: qt5-qtmultimedia-gsttools-5.6.2+git8-1.4.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-location-5.2.1+git31-1.4.1.jolla.armv7hl            (381/457), 155.5 KiB (502.2 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-location-5.2.1+git31-1.4.1.jolla.armv7hl.rpm .................................................[done]<br>
Retrieving package qt5-qtfeedback-devel-5.2.0+git6-1.3.1.jolla.armv7hl                          (382/457),  19.2 KiB ( 36.7 KiB unpacked)<br>
Retrieving: qt5-qtfeedback-devel-5.2.0+git6-1.3.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package qt5-qtdocgallery-devel-5.2.0+git8-1.4.1.jolla.armv7hl                        (383/457),  30.4 KiB (130.6 KiB unpacked)<br>
Retrieving: qt5-qtdocgallery-devel-5.2.0+git8-1.4.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtdeclarative-qtquick-devel-5.6.3+git7-1.5.1.jolla.armv7hl              (384/457), 153.4 KiB (1008.6 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-qtquick-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package qt5-qtconnectivity-qtbluetooth-devel-5.6.2+git0-1.2.1.jolla.armv7hl          (385/457),  83.7 KiB (495.7 KiB unpacked)<br>
Retrieving: qt5-qtconnectivity-qtbluetooth-devel-5.6.2+git0-1.2.1.jolla.armv7hl.rpm ...............................................[done]<br>
Retrieving package thumbnaild-0.0.6-1.4.1.jolla.armv7hl                                         (386/457),  34.1 KiB ( 71.9 KiB unpacked)<br>
Retrieving: thumbnaild-0.0.6-1.4.1.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package libnemotransferengine-qt5-1.0.1-1.5.1.jolla.armv7hl                          (387/457),  64.6 KiB (157.0 KiB unpacked)<br>
Retrieving: libnemotransferengine-qt5-1.0.1-1.5.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package mpris-qt5-qml-plugin-1.0.0-1.4.1.jolla.armv7hl                               (388/457),  17.0 KiB ( 33.6 KiB unpacked)<br>
Retrieving: mpris-qt5-qml-plugin-1.0.0-1.4.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package signon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl                                   (389/457), 218.1 KiB (560.4 KiB unpacked)<br>
Retrieving: signon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl.rpm ........................................................................[done]<br>
Retrieving package mapplauncherd-qt5-devel-1.1.14-1.3.1.jolla.armv7hl                           (390/457),  12.4 KiB (  3.6 KiB unpacked)<br>
Retrieving: mapplauncherd-qt5-devel-1.1.14-1.3.1.jolla.armv7hl.rpm ................................................................[done]<br>
Retrieving package maliit-framework-wayland-inputcontext-0.99.1+git4-1.4.1.jolla.armv7hl        (391/457),  59.4 KiB (152.7 KiB unpacked)<br>
Retrieving: maliit-framework-wayland-inputcontext-0.99.1+git4-1.4.1.jolla.armv7hl.rpm .............................................[done]<br>
Retrieving package libsailfishapp-1.2.8-1.4.1.jolla.armv7hl                                     (392/457),  19.3 KiB ( 14.0 KiB unpacked)<br>
Retrieving: libsailfishapp-1.2.8-1.4.1.jolla.armv7hl.rpm ..........................................................................[done]<br>
Retrieving package buteo-syncfw-qt5-0.8.18-1.7.1.jolla.armv7hl                                  (393/457), 182.7 KiB (481.7 KiB unpacked)<br>
Retrieving: buteo-syncfw-qt5-0.8.18-1.7.1.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package libqtwebkit5-widgets-5.6.2+git8-1.5.1.jolla.armv7hl                          (394/457),  79.8 KiB (290.2 KiB unpacked)<br>
Retrieving: libqtwebkit5-widgets-5.6.2+git8-1.5.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package libqt5sparql-tracker-direct-0.2.18-1.2.1.jolla.armv7hl                       (395/457),  34.1 KiB ( 49.4 KiB unpacked)<br>
Retrieving: libqt5sparql-tracker-direct-0.2.18-1.2.1.jolla.armv7hl.rpm ............................................................[done]<br>
Retrieving package libqt5sparql-tracker-0.2.18-1.2.1.jolla.armv7hl                              (396/457),  32.7 KiB ( 47.0 KiB unpacked)<br>
Retrieving: libqt5sparql-tracker-0.2.18-1.2.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package python3-base-3.7.2+git2-1.3.5.jolla.armv7hl                                  (397/457),   9.0 MiB ( 38.5 MiB unpacked)<br>
Retrieving: python3-base-3.7.2+git2-1.3.5.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package zypper-1.14.6+git3-1.3.4.jolla.armv7hl                                       (398/457),   1.1 MiB (  6.1 MiB unpacked)<br>
Retrieving: zypper-1.14.6+git3-1.3.4.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch                               (399/457),  26.6 KiB ( 11.3 KiB unpacked)<br>
Retrieving: ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch.rpm ....................................................................[done]<br>
Retrieving package ngfd-1.1.1-1.5.1.jolla.armv7hl                                               (400/457), 110.2 KiB (245.6 KiB unpacked)<br>
Retrieving: ngfd-1.1.1-1.5.1.jolla.armv7hl.rpm ....................................................................................[done]<br>
Retrieving package qt5-qtmultimedia-devel-5.6.2+git8-1.4.1.jolla.armv7hl                        (401/457),  96.4 KiB (521.5 KiB unpacked)<br>
Retrieving: qt5-qtmultimedia-devel-5.6.2+git8-1.4.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package qt5-qtdeclarative-import-multimedia-5.6.2+git8-1.4.1.jolla.armv7hl           (402/457),  69.7 KiB (270.1 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-import-multimedia-5.6.2+git8-1.4.1.jolla.armv7hl.rpm ................................................[done]<br>
Retrieving package nemo-qtmultimedia-plugins-gstvideotexturebackend-0.0.10-1.3.1.jolla.armv7hl  (403/457),  20.4 KiB ( 27.1 KiB unpacked)<br>
Retrieving: nemo-qtmultimedia-plugins-gstvideotexturebackend-0.0.10-1.3.1.jolla.armv7hl.rpm .......................................[done]<br>
Retrieving package qt5-qtlocation-devel-5.2.1+git31-1.4.1.jolla.armv7hl                         (404/457),  56.4 KiB (344.2 KiB unpacked)<br>
Retrieving: qt5-qtlocation-devel-5.2.1+git31-1.4.1.jolla.armv7hl.rpm ..............................................................[done]<br>
Retrieving package nemo-transferengine-qt5-1.0.1-1.5.1.jolla.armv7hl                            (405/457),  63.2 KiB (132.7 KiB unpacked)<br>
Retrieving: nemo-transferengine-qt5-1.0.1-1.5.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package libsignon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl                                (406/457),  73.0 KiB (171.7 KiB unpacked)<br>
Retrieving: libsignon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl.rpm .....................................................................[done]<br>
Retrieving package maliit-framework-wayland-0.99.1+git4-1.4.1.jolla.armv7hl                     (407/457), 152.3 KiB (449.6 KiB unpacked)<br>
Retrieving: maliit-framework-wayland-0.99.1+git4-1.4.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package libsailfishapp-devel-1.2.8-1.4.1.jolla.armv7hl                               (408/457),  16.1 KiB ( 10.2 KiB unpacked)<br>
Retrieving: libsailfishapp-devel-1.2.8-1.4.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package qt5-qtwebkit-uiprocess-launcher-5.6.2+git8-1.5.1.jolla.armv7hl               (409/457),  15.2 KiB (  8.2 KiB unpacked)<br>
Retrieving: qt5-qtwebkit-uiprocess-launcher-5.6.2+git8-1.5.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package pyotherside-qml-plugin-python3-qt5-1.5.1+git2-1.3.1.jolla.armv7hl            (410/457),  70.0 KiB (189.6 KiB unpacked)<br>
Retrieving: pyotherside-qml-plugin-python3-qt5-1.5.1+git2-1.3.1.jolla.armv7hl.rpm .................................................[done]<br>
Retrieving package sailfish-version-variant-3.0.3-1.11.10.jolla.noarch                            (411/457),  16.5 KiB (   97 B unpacked)<br>
Retrieving: sailfish-version-variant-3.0.3-1.11.10.jolla.noarch.rpm ...............................................................[done]<br>
Retrieving package qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.armv7hl                              (412/457),  63.8 KiB (176.4 KiB unpacked)<br>
Retrieving: qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package libngf-qt5-0.6.2-1.4.1.jolla.armv7hl                                         (413/457),  22.3 KiB ( 33.2 KiB unpacked)<br>
Retrieving: libngf-qt5-0.6.2-1.4.1.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package signon-plugin-oauth2-qt5-0.21.5-1.2.1.jolla.armv7hl                          (414/457),  63.3 KiB (139.7 KiB unpacked)<br>
Retrieving: signon-plugin-oauth2-qt5-0.21.5-1.2.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package libsignon-qt5-devel-8.57.5+git3-1.5.1.jolla.armv7hl                          (415/457),  22.8 KiB ( 59.0 KiB unpacked)<br>
Retrieving: libsignon-qt5-devel-8.57.5+git3-1.5.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package libqmfclient1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl                          (416/457), 514.5 KiB (  1.5 MiB unpacked)<br>
Retrieving: libqmfclient1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl.rpm ...............................................................[done]<br>
Retrieving package maliit-framework-wayland-devel-0.99.1+git4-1.4.1.jolla.armv7hl               (417/457),  25.9 KiB ( 53.1 KiB unpacked)<br>
Retrieving: maliit-framework-wayland-devel-0.99.1+git4-1.4.1.jolla.armv7hl.rpm ....................................................[done]<br>
Retrieving package qt5-qtqml-import-webkitplugin-experimental-5.6.2+git8-1.5.1.jolla.armv7hl    (418/457),  24.6 KiB ( 59.2 KiB unpacked)<br>
Retrieving: qt5-qtqml-import-webkitplugin-experimental-5.6.2+git8-1.5.1.jolla.armv7hl.rpm .........................................[done]<br>
Retrieving package qt5-qtqml-import-webkitplugin-5.6.2+git8-1.5.1.jolla.armv7hl                 (419/457),  23.0 KiB ( 43.7 KiB unpacked)<br>
Retrieving: qt5-qtqml-import-webkitplugin-5.6.2+git8-1.5.1.jolla.armv7hl.rpm ......................................................[done]<br>
Retrieving package libqtwebkit5-devel-5.6.2+git8-1.5.1.jolla.armv7hl                            (420/457),  45.7 KiB (145.4 KiB unpacked)<br>
Retrieving: libqtwebkit5-devel-5.6.2+git8-1.5.1.jolla.armv7hl.rpm .................................................................[done]<br>
Retrieving package sailfish-version-3.0.3-1.11.10.jolla.noarch                                  (421/457),  17.7 KiB (  2.0 KiB unpacked)<br>
Retrieving: sailfish-version-3.0.3-1.11.10.jolla.noarch.rpm .......................................................................[done]<br>
Retrieving package qt5-qtdeclarative-systeminfo-5.2.0+git9-1.2.1.jolla.armv7hl                  (422/457),  21.2 KiB ( 52.1 KiB unpacked)<br>
Retrieving: qt5-qtdeclarative-systeminfo-5.2.0+git9-1.2.1.jolla.armv7hl.rpm .......................................................[done]<br>
Retrieving package libcontentaction-qt5-0.3.9-1.6.1.jolla.armv7hl                               (423/457),  74.9 KiB (158.7 KiB unpacked)<br>
Retrieving: libcontentaction-qt5-0.3.9-1.6.1.jolla.armv7hl.rpm ....................................................................[done]<br>
Retrieving package buteo-mtp-qt5-0.7.0-1.6.1.jolla.armv7hl                                      (424/457), 219.1 KiB (581.6 KiB unpacked)<br>
Retrieving: buteo-mtp-qt5-0.7.0-1.6.1.jolla.armv7hl.rpm ...........................................................................[done]<br>
Retrieving package libngf-qt5-declarative-0.6.2-1.4.1.jolla.armv7hl                             (425/457),  17.4 KiB ( 16.6 KiB unpacked)<br>
Retrieving: libngf-qt5-declarative-0.6.2-1.4.1.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package nemo-qml-plugin-contentaction-0.3.9-1.6.1.jolla.armv7hl                      (426/457),  20.6 KiB ( 19.7 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-contentaction-0.3.9-1.6.1.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package sailfish-content-graphics-default-base-1.0.17-1.15.1.jolla.noarch            (427/457), 951.6 KiB (  1.0 MiB unpacked)<br>
Retrieving: sailfish-content-graphics-default-base-1.0.17-1.15.1.jolla.noarch.rpm .................................................[done]<br>
Retrieving package sailfish-content-graphics-default-z1.0-base-1.0.17-1.15.1.jolla.noarch       (428/457),   1.1 MiB (  1.1 MiB unpacked)<br>
Retrieving: sailfish-content-graphics-default-z1.0-base-1.0.17-1.15.1.jolla.noarch.rpm ............................................[done]<br>
Retrieving package sailfish-content-graphics-default-z1.0-1.0.17-1.15.1.jolla.noarch              (429/457),  56.4 KiB (    0 B unpacked)<br>
Retrieving: sailfish-content-graphics-default-z1.0-1.0.17-1.15.1.jolla.noarch.rpm .................................................[done]<br>
Retrieving package sailfish-content-graphics-default-1.0.17-1.15.1.jolla.noarch                   (430/457),  56.3 KiB (    0 B unpacked)<br>
Retrieving: sailfish-content-graphics-default-1.0.17-1.15.1.jolla.noarch.rpm ......................................................[done]<br>
Retrieving package ambienced-0.29.10-1.9.2.jolla.armv7hl                                        (431/457), 237.9 KiB (770.8 KiB unpacked)<br>
Retrieving: ambienced-0.29.10-1.9.2.jolla.armv7hl.rpm .............................................................................[done]<br>
Retrieving package sailfish-content-tones-default-0.15.3-1.3.1.jolla.noarch                     (432/457),  22.3 MiB ( 22.8 MiB unpacked)<br>
Retrieving: sailfish-content-tones-default-0.15.3-1.3.1.jolla.noarch.rpm ..........................................................[done]<br>
Retrieving package sailfishsilica-qt5-1.0.51.1-1.22.1.jolla.armv7hl                             (433/457), 570.3 KiB (  1.4 MiB unpacked)<br>
Retrieving: sailfishsilica-qt5-1.0.51.1-1.22.1.jolla.armv7hl.rpm ..................................................................[done]<br>
Retrieving package sailfish-components-media-qt5-0.2.2-1.5.1.jolla.armv7hl                      (434/457),  35.9 KiB ( 63.7 KiB unpacked)<br>
Retrieving: sailfish-components-media-qt5-0.2.2-1.5.1.jolla.armv7hl.rpm ...........................................................[done]<br>
Retrieving package sailfish-components-filemanager-0.2.3-1.7.1.jolla.armv7hl                    (435/457),  24.8 KiB ( 48.8 KiB unpacked)<br>
Retrieving: sailfish-components-filemanager-0.2.3-1.7.1.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package libngf-0.26-1.1.8.jolla.armv7hl                                              (436/457),  22.5 KiB ( 36.4 KiB unpacked)<br>
Retrieving: libngf-0.26-1.1.8.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package libjollasignonuiservice-qt5-0.4.6-1.6.1.jolla.armv7hl                        (437/457),  66.4 KiB (141.6 KiB unpacked)<br>
Retrieving: libjollasignonuiservice-qt5-0.4.6-1.6.1.jolla.armv7hl.rpm .............................................................[done]<br>
Retrieving package declarative-transferengine-qt5-0.3.9-1.8.1.jolla.armv7hl                     (438/457),  63.5 KiB (147.2 KiB unpacked)<br>
Retrieving: declarative-transferengine-qt5-0.3.9-1.8.1.jolla.armv7hl.rpm ..........................................................[done]<br>
Retrieving package ambienced-devel-0.29.10-1.9.2.jolla.armv7hl                                  (439/457),  31.6 KiB (  3.4 KiB unpacked)<br>
Retrieving: ambienced-devel-0.29.10-1.9.2.jolla.armv7hl.rpm .......................................................................[done]<br>
Retrieving package dsme-0.79.4-1.6.2.jolla.armv7hl                                              (440/457), 154.8 KiB (349.1 KiB unpacked)<br>
Retrieving: dsme-0.79.4-1.6.2.jolla.armv7hl.rpm ...................................................................................[done]<br>
Retrieving package libjollasignonuiservice-qt5-plugin-0.4.6-1.6.1.jolla.armv7hl                 (441/457),  21.6 KiB ( 14.3 KiB unpacked)<br>
Retrieving: libjollasignonuiservice-qt5-plugin-0.4.6-1.6.1.jolla.armv7hl.rpm ......................................................[done]<br>
Retrieving package patterns-sailfish-silica-devel-1.0.18-1.11.1.jolla.noarch                      (442/457),  47.5 KiB (    0 B unpacked)<br>
Retrieving: patterns-sailfish-silica-devel-1.0.18-1.11.1.jolla.noarch.rpm .........................................................[done]<br>
Retrieving package mce-1.100.2-1.16.1.jolla.armv7hl                                             (443/457), 335.1 KiB (727.8 KiB unpacked)<br>
Retrieving: mce-1.100.2-1.16.1.jolla.armv7hl.rpm ..................................................................................[done]<br>
Retrieving package sailfish-components-gallery-qt5-1.0.2-1.15.1.jolla.armv7hl                   (444/457),  68.9 KiB (148.6 KiB unpacked)<br>
Retrieving: sailfish-components-gallery-qt5-1.0.2-1.15.1.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package nemo-qml-plugin-systemsettings-0.5.11-1.18.1.jolla.armv7hl                   (445/457), 252.4 KiB (662.3 KiB unpacked)<br>
Retrieving: nemo-qml-plugin-systemsettings-0.5.11-1.18.1.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package libkeepalive-1.6.2-1.6.1.jolla.armv7hl                                       (446/457),  39.3 KiB ( 88.9 KiB unpacked)<br>
Retrieving: libkeepalive-1.6.2-1.6.1.jolla.armv7hl.rpm ............................................................................[done]<br>
Retrieving package sailfish-components-pickers-qt5-1.0.0-1.6.1.jolla.armv7hl                    (447/457),  43.0 KiB (103.8 KiB unpacked)<br>
Retrieving: sailfish-components-pickers-qt5-1.0.0-1.6.1.jolla.armv7hl.rpm .........................................................[done]<br>
Retrieving package libqmfmessageserver1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl                   (448/457), 367.8 KiB (  1.0 MiB unpacked)<br>
Retrieving: libqmfmessageserver1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl.rpm ........................................................[done]<br>
Retrieving package qmf-qt5-devel-4.0.4+git111-1.10.1.jolla.armv7hl                              (449/457), 115.1 KiB (534.8 KiB unpacked)<br>
Retrieving: qmf-qt5-devel-4.0.4+git111-1.10.1.jolla.armv7hl.rpm ...................................................................[done]<br>
Retrieving package patterns-sailfish-qt5-devel-basic-1.0.18-1.11.1.jolla.noarch                   (450/457),  49.0 KiB (    0 B unpacked)<br>
Retrieving: patterns-sailfish-qt5-devel-basic-1.0.18-1.11.1.jolla.noarch.rpm ......................................................[done]<br>
Retrieving package patterns-sailfish-qt5-devel-full-1.0.18-1.11.1.jolla.noarch                    (451/457),  47.7 KiB (    0 B unpacked)<br>
Retrieving: patterns-sailfish-qt5-devel-full-1.0.18-1.11.1.jolla.noarch.rpm .......................................................[done]<br>
Retrieving package sdk-target-configs-0.114-1.9.4.jolla.noarch                                  (452/457),  11.0 KiB (  4.2 KiB unpacked)<br>
Retrieving: sdk-target-configs-0.114-1.9.4.jolla.noarch.rpm .......................................................................[done]<br>
Retrieving package sdk-harbour-rpmvalidator-1.49.1-1.8.1.jolla.noarch                           (453/457),  32.2 KiB ( 50.0 KiB unpacked)<br>
Retrieving: sdk-harbour-rpmvalidator-1.49.1-1.8.1.jolla.noarch.rpm ................................................................[done]<br>
Retrieving package sdk-register-0.5-1.3.4.jolla.armv7hl                                         (454/457),  12.6 KiB ( 12.0 KiB unpacked)<br>
Retrieving: sdk-register-0.5-1.3.4.jolla.armv7hl.rpm ..............................................................................[done]<br>
Retrieving package sdk-configs-0.114-1.9.4.jolla.noarch                                         (455/457), 123.1 KiB (  3.1 MiB unpacked)<br>
Retrieving: sdk-configs-0.114-1.9.4.jolla.noarch.rpm ..............................................................................[done]<br>
Retrieving package patterns-sailfish-target-support-1.0.18-1.11.1.jolla.noarch                    (456/457),  47.5 KiB (    0 B unpacked)<br>
Retrieving: patterns-sailfish-target-support-1.0.18-1.11.1.jolla.noarch.rpm .......................................................[done]<br>
Retrieving package patterns-sailfish-configuration-sdk-target-1.0.18-1.11.1.jolla.noarch          (457/457),  47.6 KiB (    0 B unpacked)<br>
Retrieving: patterns-sailfish-configuration-sdk-target-1.0.18-1.11.1.jolla.noarch.rpm .............................................[done]<br>
Installing: buteo-mtp-qt5-sample-vendor-configuration-0.7.0-1.6.1.jolla ...........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/buteo-mtp-qt5-sample-vendor-configuration-0.7.0-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: fontpackages-filesystem-1.44-1.1.9.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/fontpackages-filesystem-1.44-1.1.9.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: jolla-ca-0.9-1.3.1.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/jolla-ca-0.9-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: kernel-headers-3.18.136-1.2.3.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/kernel-headers-3.18.136-1.2.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgcc-4.9.4-1.2.5.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgcc-4.9.4-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mobile-broadband-provider-info-20131125+git68-1.3.2.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/mobile-broadband-provider-info-20131125+git68-1.3.2.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ncurses-base-6.1+git1-1.3.5.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ncurses-base-6.1+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ofono-configs-mer-1.21+git44-1.19.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ofono-configs-mer-1.21+git44-1.19.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtplatformsupport-devel-5.6.3+git9-1.9.2.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtplatformsupport-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-ca-0.1.1-1.3.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-ca-0.1.1-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: setup-2.8.56-1.2.5.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/setup-2.8.56-1.2.5.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: /etc/group created as /etc/group.rpmnew<br>
warning: /etc/gshadow created as /etc/gshadow.rpmnew<br>
warning: /etc/passwd created as /etc/passwd.rpmnew<br>
warning: /etc/shadow created as /etc/shadow.rpmnew<br>
<br>
<br>
Installing: tzdata-2017b-1.1.11.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/tzdata-2017b-1.1.11.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: usb-moded-developer-mode-0.86.0+mer33-1.3.2.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/usb-moded-developer-mode-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: xkeyboard-config-2.10.1+git6-1.2.7.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/xkeyboard-config-2.10.1+git6-1.2.7.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: filesystem-3.1-1.1.9.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/filesystem-3.1-1.1.9.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: tzdata-timed-2017b.2-1.3.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/noarch/tzdata-timed-2017b.2-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glibc-2.25+git5-1.4.1.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glibc-2.25+git5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: usb-moded-defaults-0.86.0+mer33-1.3.2.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/usb-moded-defaults-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: basesystem-11+git1-1.2.8.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/basesystem-11+git1-1.2.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ncurses-libs-6.1+git1-1.3.5.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ncurses-libs-6.1+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-graphics-closed-z1.0-0.6.1-1.2.1.jolla ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-closed-z1.0-0.6.1-1.2.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: bash-1:3.2.57-1.2.5.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/bash-3.2.57-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-graphics-closed-0.6.1-1.2.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-closed-0.6.1-1.2.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glibc-common-2.25+git5-1.4.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glibc-common-2.25+git5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: zlib-1.2.11+git1-1.4.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/zlib-1.2.11+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-minui-resources-z1.0-0.0.4-1.4.2.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-minui-resources-z1.0-0.0.4-1.4.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qttools-5.6.3+git1-1.4.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qttools-5.6.3+git1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qml-rpm-macros-0.0.5-1.1.9.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/qml-rpm-macros-0.0.5-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: info-4.13a-1.2.7.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/info-4.13a-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: /usr/share/info/dir saved as /usr/share/info/dir.rpmsave<br>
<br>
<br>
Installing: sailfish-upgrade-ui-resources-z1.0-0.1.1-1.5.2.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-upgrade-ui-resources-z1.0-0.1.1-1.5.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glibc-headers-2.25+git5-1.4.1.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glibc-headers-2.25+git5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glibc-devel-2.25+git5-1.4.1.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glibc-devel-2.25+git5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: xz-libs-5.0.4-1.2.8.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/xz-libs-5.0.4-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: unzip-6.0-1.2.5.jolla .................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/unzip-6.0-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ssu-sysinfo-1.1.3-1.3.7.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ssu-sysinfo-1.1.3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: speexdsp-1.2.0+git2-1.1.7.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/speexdsp-1.2.0+git2-1.1.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: speex-1.2.0+git1-1.2.7.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/speex-1.2.0+git1-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: shadow-utils-4.6-1.2.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/shadow-utils-4.6-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sed-1:4.1.5-1.2.6.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/sed-4.1.5-1.2.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: readline-5.2-1.2.7.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/readline-5.2-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pth-2.0.7-1.1.9.jolla .................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pth-2.0.7-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: psmisc-22.13-1.3.6.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/psmisc-22.13-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: procps-3.2.8-1.2.7.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/procps-3.2.8-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: popt-1.16-1.1.10.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/popt-1.16-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pkgconfig-0.27.1-1.2.7.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pkgconfig-0.27.1-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pixman-0.34.0-1.1.9.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pixman-0.34.0-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patch-2.7.5+git1-1.1.9.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/patch-2.7.5+git1-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: orc-0.4.26+git1-1.1.9.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/orc-0.4.26+git1-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: opus-1.2.1-1.2.7.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/opus-1.2.1-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: openjpeg-2.3.0+git1-1.2.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/openjpeg-2.3.0+git1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nspr-4.20.0-1.3.7.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nspr-4.20.0-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ncurses-6.1+git1-1.3.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ncurses-6.1+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mtdev-1.1.3-1.1.9.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mtdev-1.1.3-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: lsof-4.91+git1-1.3.4.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/lsof-4.91+git1-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libxml2-2.9.8+git2-1.3.10.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libxml2-2.9.8+git2-1.3.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libxkbcommon-0.5.0+git1-1.2.8.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libxkbcommon-0.5.0+git1-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libwebp-0.6.1+git3-1.3.6.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libwebp-0.6.1+git3-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libtool-ltdl-2.4.6-1.2.9.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libtool-ltdl-2.4.6-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libtasn1-4.13+git1-1.3.9.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libtasn1-4.13+git1-1.3.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libstdc++-4.9.4-1.2.5.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libstdc++-4.9.4-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libshadowutils-0.0.2-1.4.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libshadowutils-0.0.2-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsbc-1.3-1.3.6.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsbc-1.3-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libpng-1.6.34-1.2.9.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libpng-1.6.34-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libogg-1.3.3-1.2.7.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libogg-1.3.3-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libnl-3.4.0-1.2.7.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libnl-3.4.0-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmpg123-1.25.10-1.1.7.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libmpg123-1.25.10-1.1.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: liblua-5.1.5-1.1.11.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/liblua-5.1.5-1.1.11.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libjpeg-turbo-1.5.3+git1-1.4.5.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libjpeg-turbo-1.5.3+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libiptcdata-1.0.4+git1-1.3.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libiptcdata-1.0.4+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libiphb-1.2.5+git1-1.3.3.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libiphb-1.2.5+git1-1.3.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgpg-error-1.27+git2-1.3.5.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgpg-error-1.27+git2-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libffi-3.2.1+git1-1.2.13.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libffi-3.2.1+git1-1.2.13.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libexif-0.6.21+git2-1.2.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libexif-0.6.21+git2-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libdrm-2.4.39-1.1.9.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libdrm-2.4.39-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libattr-2.4.47+git1-1.3.7.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libattr-2.4.47+git1-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libasyncns-0.8-1.2.5.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libasyncns-0.8-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: json-c-0.12-1.1.9.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/json-c-0.12-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: iptables-1.8.2+git1-1.4.3.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/iptables-1.8.2+git1-1.4.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: giflib-4.2.3+git2-1.3.5.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/giflib-4.2.3+git2-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gdbm-1.8.3-1.2.10.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gdbm-1.8.3-1.2.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: freetype-2.8.0-1.1.9.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/freetype-2.8.0-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: file-libs-5.35+git2-1.2.8.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/file-libs-5.35+git2-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: expat-2.1.0-1.1.10.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/expat-2.1.0-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: elfutils-libelf-0.170+git1-1.3.10.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/elfutils-libelf-0.170+git1-1.3.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: db4-4.8.30-1.3.12.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/db4-4.8.30-1.3.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: cpio-2.12+git1-1.3.4.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/cpio-2.12+git1-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: bzip2-libs-1.0.6-1.2.9.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/bzip2-libs-1.0.6-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-1.29.3+git5-1.1.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: binutils-2.25-1.3.11.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/binutils-2.25-1.3.11.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: xz-5.0.4-1.2.8.jolla ..................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/xz-5.0.4-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libutempter-1.1.5+git1-1.1.12.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libutempter-1.1.5+git1-1.1.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: zlib-devel-1.2.11+git1-1.4.5.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/zlib-devel-1.2.11+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nspr-devel-4.20.0-1.3.7.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nspr-devel-4.20.0-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libxslt-1.1.29-1.2.6.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libxslt-1.1.29-1.2.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: augeas-libs-1.6.0+git1-1.2.5.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/augeas-libs-1.6.0+git1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: taglib-1.11.1+git1-1.3.1.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/taglib-1.11.1+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qtchooser-26-1.1.9.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/qtchooser-26-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pcre-8.42+git1-1.3.5.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pcre-8.42+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libvpx-1.7.0+git1-1.2.5.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libvpx-1.7.0+git1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libusb-0.1.12-1.2.5.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libusb-0.1.12-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsailfishkeyprovider-0.0.14-1.2.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libsailfishkeyprovider-0.0.14-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libvorbis-1.3.6+git1-1.2.5.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libvorbis-1.3.6+git1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libtheora-1.2.0alpha1+git-1.2.8.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libtheora-1.2.0alpha1+git-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: flac-1.3.2-1.2.7.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/flac-1.3.2-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libtiff-4.0.10+git1-1.2.6.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libtiff-4.0.10+git1-1.2.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libksba-1.3.5+git2-1.2.5.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libksba-1.3.5+git2-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgcrypt-1.5.6+git1-1.1.11.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgcrypt-1.5.6+git1-1.1.11.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: p11-kit-0.23.12+git1-1.3.5.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/p11-kit-0.23.12+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libcap-2.24+git1-1.3.6.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libcap-2.24+git1-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libacl-2.2.53-1.2.7.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libacl-2.2.53-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: iptables-ipv6-1.8.2+git1-1.4.3.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/iptables-ipv6-1.8.2+git1-1.4.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: file-5.35+git2-1.2.8.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/file-5.35+git2-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: wayland-1.6.0+git1-1.2.8.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/wayland-1.6.0+git1-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: expat-devel-2.1.0-1.1.10.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/expat-devel-2.1.0-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: exempi-2.4.3-1.2.1.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/exempi-2.4.3-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: db4-utils-4.8.30-1.3.12.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/db4-utils-4.8.30-1.3.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: bzip2-1.0.6-1.2.9.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/bzip2-1.0.6-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-symlinks-gzip-1.29.3+git5-1.1.5.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-symlinks-gzip-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-symlinks-grep-1.29.3+git5-1.1.5.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-symlinks-grep-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-symlinks-findutils-1.29.3+git5-1.1.5.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-symlinks-findutils-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-symlinks-diffutils-1.29.3+git5-1.1.5.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-symlinks-diffutils-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: busybox-symlinks-dhcp-1.29.3+git5-1.1.5.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/busybox-symlinks-dhcp-1.29.3+git5-1.1.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: xz-lzma-compat-5.0.4-1.2.8.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/xz-lzma-compat-5.0.4-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: kmod-libs-21-1.2.8.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/kmod-libs-21-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: elfutils-libs-0.170+git1-1.3.10.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/elfutils-libs-0.170+git1-1.3.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: freetype-devel-2.8.0-1.1.9.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/freetype-devel-2.8.0-1.1.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-default-5.6.3+git9-1.9.2.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-default-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glib2-2.56.1+git3-1.3.6.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glib2-2.56.1+git3-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: tar-1.17-1.2.5.jolla ..................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/tar-1.17-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: coreutils-1:6.9-1.2.6.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/coreutils-6.9-1.2.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: elfutils-0.170+git1-1.3.10.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/elfutils-0.170+git1-1.3.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: shared-mime-info-1.12-1.4.2.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/shared-mime-info-1.12-1.4.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libwspcodec-2.2.1-1.2.7.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libwspcodec-2.2.1-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgsf-1.14.36+git3-1.3.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libgsf-1.14.36+git3-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libglibutil-1.0.35-1.8.4.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libglibutil-1.0.35-1.8.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libdsme-0.66.1-1.4.4.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libdsme-0.66.1-1.4.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gstreamer1.0-1.14.1-1.3.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gstreamer1.0-1.14.1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gmime-2.6.20-1.2.1.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/gmime-2.6.20-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: desktop-file-utils-0.23+git1-1.2.9.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/desktop-file-utils-0.23+git1-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pam-1.1.8+git5-1.2.5.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pam-1.1.8+git5-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmce-glib-1.0.5-1.1.14.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libmce-glib-1.0.5-1.1.14.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgsupplicant-1.0.11-1.4.6.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgsupplicant-1.0.11-1.4.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgrilio-1.0.29-1.9.1.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgrilio-1.0.29-1.9.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgofono-2.0.6-1.2.11.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgofono-2.0.6-1.2.11.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libdbusaccess-1.0.7-1.3.4.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libdbusaccess-1.0.7-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: systemd-libs-225+git13-1.4.2.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/systemd-libs-225+git13-1.4.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sound-theme-freedesktop-0.8-1.3.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/noarch/sound-theme-freedesktop-0.8-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: p11-kit-trust-0.23.12+git1-1.3.5.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/p11-kit-trust-0.23.12+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libuser-0.62+git1-1.2.5.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libuser-0.62+git1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsmartcols-2.33+git1-1.4.5.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsmartcols-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: groff-1.18.1.4-1.1.14.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/groff-1.18.1.4-1.1.14.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gawk-1:3.1.5-1.2.7.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gawk-3.1.5-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: fontconfig-2.12.4-1.2.9.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/fontconfig-2.12.4-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: alsa-lib-1.0.26-1.2.6.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/alsa-lib-1.0.26-1.2.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libgofonoext-1.0.10-1.2.10.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libgofonoext-1.0.10-1.2.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libglapi-9.2.5+git3-1.1.10.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libglapi-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dbus-libs-1.10.8+git1-1.1.12.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dbus-libs-1.10.8+git1-1.1.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: cor-0.1.18-1.2.1.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/cor-0.1.18-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: p11-kit-nss-ckbi-0.23.12+git1-1.3.5.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/p11-kit-nss-ckbi-0.23.12+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: passwd-0.79+git1-1.3.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/passwd-0.79+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libuuid-2.33+git1-1.4.5.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libuuid-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-macros-2:5.16.1-1.1.16.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-macros-5.16.1-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-fonts-0.1.5-1.3.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-fonts-0.1.5-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: fontconfig-devel-2.12.4-1.2.9.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/fontconfig-devel-2.12.4-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsndfile-1.0.25-1.2.5.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsndfile-1.0.25-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libEGL-9.2.5+git3-1.1.10.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libEGL-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libdbuslogserver-dbus-1.0.15-1.4.4.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libdbuslogserver-dbus-1.0.15-1.4.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dbus-glib-0.100.2-1.2.7.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dbus-glib-0.100.2-1.2.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: statefs-pp-0.3.35-1.2.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/statefs-pp-0.3.35-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gnutls-2.12.23.4-1.2.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gnutls-2.12.23.4-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ca-certificates-2018.2.24-1.2.14.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/ca-certificates-2018.2.24-1.2.14.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libblkid-2.33+git1-1.4.5.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libblkid-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-libs-2:5.16.1-1.1.16.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-libs-5.16.1-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-9.2.5+git3-1.1.10.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libresource-0.23.1+git1-1.2.3.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libresource-0.23.1+git1-1.2.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: glib-networking-2.42.0-1.2.2.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/glib-networking-2.42.0-1.2.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: openssl-libs-1.0.2o+git2-1.4.5.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/openssl-libs-1.0.2o+git2-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libfdisk-2.33+git1-1.4.5.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libfdisk-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-2:5.16.1-1.1.16.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-5.16.1-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libwayland-egl-9.2.5+git3-1.1.10.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libwayland-egl-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libGLESv2-9.2.5+git3-1.1.10.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libGLESv2-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libEGL-devel-9.2.5+git3-1.1.10.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libEGL-devel-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libcurl-7.64.0+git1-1.8.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libcurl-7.64.0+git1-1.8.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libarchive-3.3.3+git1-1.2.9.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libarchive-3.3.3+git1-1.2.9.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmount-2.33+git1-1.4.5.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libmount-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Scalar-List-Utils-2:1.25-1.1.16.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-Scalar-List-Utils-1.25-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mesa-llvmpipe-libGLESv2-devel-9.2.5+git3-1.1.10.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mesa-llvmpipe-libGLESv2-devel-9.2.5+git3-1.1.10.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gstreamer1.0-plugins-base-1.14.1-1.2.5.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gstreamer1.0-plugins-base-1.14.1-1.2.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: cairo-1.14.6+git1-1.2.3.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/cairo-1.14.6+git1-1.2.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pacrunner-0.15+git1-1.3.3.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pacrunner-0.15+git1-1.3.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gnupg2-1:2.0.4+git2-1.4.5.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gnupg2-2.0.4+git2-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: curl-7.64.0+git1-1.8.5.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/curl-7.64.0+git1-1.8.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: util-linux-2.33+git1-1.4.5.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/util-linux-2.33+git1-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Pod-Escapes-2:1.04-1.1.16.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Escapes-1.04-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-gstreamer1.0-interfaces-0.20150126.0-1.3.3.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nemo-gstreamer1.0-interfaces-0.20150126.0-1.3.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gpgme-1.2.0+git6-1.3.4.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gpgme-1.2.0+git6-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: rpm-libs-4.14.1+git9-1.5.7.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/rpm-libs-4.14.1+git9-1.5.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: xdg-utils-1.1.2+git1-1.2.8.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/xdg-utils-1.1.2+git1-1.2.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: fuse-2.9.0+git1-1.3.1.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/fuse-2.9.0+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: cryptsetup-libs-2.1.0+git1-1.3.5.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/cryptsetup-libs-2.1.0+git1-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Pod-Simple-2:3.20-1.1.16.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Simple-3.20-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: rpm-4.14.1+git9-1.5.7.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/rpm-4.14.1+git9-1.5.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: fuse-libs-2.9.0+git1-1.3.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/fuse-libs-2.9.0+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: systemd-config-sailfish-0.8.15-1.8.1.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/systemd-config-sailfish-0.8.15-1.8.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-parent-2:0.225-1.1.16.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-parent-0.225-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: device-mapper-libs-2.02.177+git3-1.3.7.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/device-mapper-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Pod-Perldoc-2:3.17.00-1.1.16.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Perldoc-3.17.00-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: device-mapper-event-libs-2.02.177+git3-1.3.7.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/device-mapper-event-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Pod-Parser-2:1.51-1.1.16.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Pod-Parser-1.51-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: lvm2-2.02.177+git3-1.3.7.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/lvm2-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Filter-2:1.40-1.1.16.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-Filter-1.40-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: systemd-225+git13-1.4.2.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/systemd-225+git13-1.4.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Module-Pluggable-2:4.00-1.1.16.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/perl-Module-Pluggable-4.00-1.1.16.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dbus-1.10.8+git1-1.1.12.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dbus-1.10.8+git1-1.1.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: perl-threads-2:1.86-1.1.16.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-threads-1.86-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: polkit-0.105+git2-1.2.3.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/polkit-0.105+git2-1.2.3.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: device-mapper-2.02.177+git3-1.3.7.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/device-mapper-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dbus-devel-1.10.8+git1-1.1.12.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dbus-devel-1.10.8+git1-1.1.12.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-Socket-2:2.001-1.1.16.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-Socket-2.001-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: device-mapper-event-2.02.177+git3-1.3.7.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/device-mapper-event-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: perl-threads-shared-2:1.40-1.1.16.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/perl-threads-shared-1.40-1.1.16.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: lvm2-libs-2.02.177+git3-1.3.7.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/lvm2-libs-2.02.177+git3-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qmake-5.6.3+git9-1.9.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qmake-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: meego-rpm-config-0.18-1.2.7.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/meego-rpm-config-0.18-1.2.7.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: wpa_supplicant-2.6+git5-1.3.5.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/wpa_supplicant-2.6+git5-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: usb-moded-0.86.0+mer33-1.3.2.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/usb-moded-0.86.0+mer33-1.3.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: systemd-user-session-targets-0.0.2-1.3.5.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/systemd-user-session-targets-0.0.2-1.3.5.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pulseaudio-12.2+git1-1.7.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/pulseaudio-12.2+git1-1.7.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ofono-1.21+git44-1.19.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/ofono-1.21+git44-1.19.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: user radio does not exist - using root<br>
warning: group radio does not exist - using root<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: oneshot-0.4.8-1.2.8.jolla .............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/noarch/oneshot-0.4.8-1.2.8.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mapplauncherd-4.1.30-1.6.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/mapplauncherd-4.1.30-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libcanberra-0.30+git1-1.4.1.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libcanberra-0.30+git1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: connman-configs-mer-1.32+git65-1.25.2.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/connman-configs-mer-1.32+git65-1.25.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ffmpeg-4.1.1+git1-1.2.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/ffmpeg-4.1.1+git1-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dconf-0.28.0-1.1.2.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dconf-0.28.0-1.1.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/0/dconf-update - job saved OK<br>
<br>
<br>
Installing: boost-system-1.66.0-1.3.8.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/boost-system-1.66.0-1.3.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: connman-1.32+git65-1.25.2.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/connman-1.32+git65-1.25.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
rm: cannot remove `/etc/resolv.conf': Read-only file system<br>
ln: creating symbolic link `/etc/resolv.conf': Read-only file system<br>
find: /var/lib/connman: No such file or directory<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: libicu-63.1+git5-1.1.6.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libicu-63.1+git5-1.1.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: jolla-ambient-sound-theme-0.0.17-1.2.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/jolla-ambient-sound-theme-0.0.17-1.2.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/0/dconf-update - job saved OK<br>
<br>
<br>
Installing: boost-filesystem-1.66.0-1.3.8.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/boost-filesystem-1.66.0-1.3.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sqlite-libs-3.13.0+git3-1.3.6.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/sqlite-libs-3.13.0+git3-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtcore-5.6.3+git9-1.9.2.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtcore-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: statefs-0.3.35-1.2.1.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/statefs-0.3.35-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
STATEFS_GID=995<br>
STATEFS_UMASK=0002<br>
STATEFS_GID=995<br>
STATEFS_UMASK=0002<br>
Loader register default<br>
Register default<br>
Dumping loader "/usr/lib/statefs/libloader-default.so"<br>
Dumping loader "/usr/lib/statefs/libloader-default.so"<br>
Dumping loader "/usr/lib/statefs/libloader-inout.so"<br>
Dumping loader "/usr/lib/statefs/libloader-inout.so"<br>
Trying to dump inout provider "/usr/share/statefs/inout_power.conf"<br>
provider-power-inout<br>
Trying to dump inout provider "/etc/timed-statefs.conf"<br>
provider-timed-qt5<br>
add-oneshot: /usr/lib/oneshot.d/statefs-02-register-default - run OK<br>
<br>
<br>
Installing: nss-3.39-1.3.6.jolla ..................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nss-3.39-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtconcurrent-5.6.3+git9-1.9.2.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtconcurrent-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: statefs-provider-inout-power-0.3.17-1.3.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/noarch/statefs-provider-inout-power-0.3.17-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
unregister<br>
Register inout_power<br>
Trying to dump inout provider "/usr/share/statefs/inout_power.conf"<br>
provider-power-inout<br>
add-oneshot: /usr/lib/oneshot.d/statefs-03-register-inout_power - run OK<br>
<br>
<br>
Installing: boost-thread-1.66.0-1.3.8.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/boost-thread-1.66.0-1.3.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nss-sysinit-3.39-1.3.6.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nss-sysinit-3.39-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdbus-5.6.3+git9-1.9.2.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdbus-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nss-devel-3.39-1.3.6.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nss-devel-3.39-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtgui-5.6.3+git9-1.9.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtgui-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nss-softokn-freebl-3.39-1.3.6.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/nss-softokn-freebl-3.39-1.3.6.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtnetwork-5.6.3+git9-1.9.2.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtnetwork-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: python-2.7.15+git2-1.3.7.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/python-2.7.15+git2-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: poppler-0.74.0+git1-1.5.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/poppler-0.74.0+git1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsoup-2.54.1+git3-1.2.4.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsoup-2.54.1+git3-1.2.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsolv0-0.6.35+git2-1.4.5.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsolv0-0.6.35+git2-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libaccounts-glib-1.18+git1-1.1.8.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libaccounts-glib-1.18+git1-1.1.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: PackageKit-glib-1.1.9+git5-1.8.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/PackageKit-glib-1.1.9+git5-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtwidgets-5.6.3+git9-1.9.2.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtwidgets-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: python-libs-2.7.15+git2-1.3.7.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/python-libs-2.7.15+git2-1.3.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: poppler-glib-0.74.0+git1-1.5.1.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/poppler-glib-0.74.0+git1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: totem-pl-parser-3.26.1+git1-1.4.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/totem-pl-parser-3.26.1+git1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: gstreamer1.0-plugins-good-1.14.1-1.2.8.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/gstreamer1.0-plugins-good-1.14.1-1.2.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsolv-tools-0.6.35+git2-1.4.5.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libsolv-tools-0.6.35+git2-1.4.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtopengl-5.6.3+git9-1.9.2.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtopengl-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: rpm-build-4.14.1+git9-1.5.7.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/rpm-build-4.14.1+git9-1.5.7.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ngfd-settings-sailfish-0.8.15-1.8.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/ngfd-settings-sailfish-0.8.15-1.8.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsql-5.6.3+git9-1.9.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsql-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-tools-5.6.3+git9-1.9.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-tools-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtxmlpatterns-5.6.3+git2-1.2.1.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtxmlpatterns-5.6.3+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtxml-5.6.3+git9-1.9.2.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtxml-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qttest-5.6.3+git9-1.9.2.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qttest-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsvg-5.6.2+git1-1.2.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsvg-5.6.2+git1-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtserviceframework-5.2.0+git9-1.2.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtserviceframework-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsensors-5.2.1+git17-1.3.1.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsensors-5.2.1+git17-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpublishsubscribe-5.2.0+git9-1.2.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtpublishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtprintsupport-5.6.3+git9-1.9.2.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtprintsupport-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpositioning-5.2.1+git31-1.4.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpositioning-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-organizer-5.2.0+git2-1.2.1.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-contacts-5.2.0+git2-1.2.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-plugin-sqldriver-sqlite-5.6.3+git9-1.9.2.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-plugin-sqldriver-sqlite-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-plugin-platform-minimal-5.6.3+git9-1.9.2.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-plugin-platform-minimal-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-plugin-imageformat-jpeg-5.6.3+git9-1.9.2.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-plugin-imageformat-jpeg-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-plugin-bearer-generic-5.6.3+git9-1.9.2.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-plugin-bearer-generic-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mlite-qt5-0.2.25-1.4.1.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/mlite-qt5-0.2.25-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libusb-moded-qt5-1.8-1.4.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libusb-moded-qt5-1.8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqtaround2-0.2.8-1.2.1.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqtaround2-0.2.8-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmediaart-1.9.4-1.4.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libmediaart-1.9.4-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmce-qt5-1.1.1-1.4.1.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libmce-qt5-1.1.1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libkf5archive-5.41.0+git4-1.4.1.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libkf5archive-5.41.0+git4-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libiodata-qt5-0.19.10-1.3.1.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libiodata-qt5-0.19.10-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libdbus-qeventloop-qt5-1.30.3-1.4.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libdbus-qeventloop-qt5-1.30.3-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dbusextended-qt5-0.0.3-1.3.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/dbusextended-qt5-0.0.3-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: connman-qt5-1.2.16-1.10.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/connman-qt5-1.2.16-1.10.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtcore-devel-5.6.3+git9-1.9.2.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtcore-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: poppler-qt5-0.74.0+git1-1.5.1.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/poppler-qt5-0.74.0+git1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libaccounts-qt5-1.13+git1-1.3.1.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libaccounts-qt5-1.13+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-5.6.3+git7-1.5.1.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qttools-linguist-5.6.3+git1-1.4.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qttools-linguist-5.6.3+git1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-versit-5.2.0+git2-1.2.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-versit-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libquillmetadata-qt5-1.111111.10-1.5.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libquillmetadata-qt5-1.111111.10-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-silica-background-qt5-0.9.14-1.4.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-silica-background-qt5-0.9.14-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: statefs-qt5-0.3.5-1.4.1.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/statefs-qt5-0.3.5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: tracker-1.12.4+git9-1.7.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/tracker-1.12.4+git9-1.7.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
su: cannot set groups: Operation not permitted<br>
su: cannot set groups: Operation not permitted<br>
add-oneshot: /etc/oneshot.d/default/tracker-configs.sh - job saved OK<br>
<br>
<br>
Removing libav-12.2+git1-1.2.17.jolla .............................................................................................[done]<br>
Installing: libresourceqt-qt5-1.30.3-1.4.1.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libresourceqt-qt5-1.30.3-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ssu-network-proxy-plugin-0.43.12-1.8.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/ssu-network-proxy-plugin-0.43.12-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtxml-devel-5.6.3+git9-1.9.2.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtxml-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsql-devel-5.6.3+git9-1.9.2.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsql-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtserviceframework-devel-5.2.0+git9-1.2.1.jolla ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtserviceframework-devel-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsensors-devel-5.2.1+git17-1.3.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsensors-devel-5.2.1+git17-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpublishsubscribe-devel-5.2.0+git9-1.2.1.jolla ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtpublishsubscribe-devel-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpositioning-devel-5.2.1+git31-1.4.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpositioning-devel-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-organizer-devel-5.2.0+git2-1.2.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-organizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-contacts-devel-5.2.0+git2-1.2.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-contacts-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtnetwork-devel-5.6.3+git9-1.9.2.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtnetwork-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtgui-devel-5.6.3+git9-1.9.2.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtgui-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdbus-devel-5.6.3+git9-1.9.2.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdbus-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtconcurrent-devel-5.6.3+git9-1.9.2.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtconcurrent-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-qtquick-5.6.3+git7-1.5.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-qtquick-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-versitorganizer-5.2.0+git2-1.2.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-versitorganizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: timed-qt5-3.5.1-1.4.1.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/timed-qt5-3.5.1-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
unable to set CAP_SETFCAP effective capability: Operation not permitted<br>
add-oneshot: /usr/lib/oneshot.d/setcaps-timed-qt5.sh - could not be run, save for later<br>
add-oneshot: /etc/oneshot.d/0/setcaps-timed-qt5.sh - job saved OK<br>
su: cannot set groups: Operation not permitted<br>
su: cannot set groups: Operation not permitted<br>
Register timed<br>
Trying to dump inout provider "/etc/timed-statefs.conf"<br>
provider-timed-qt5<br>
add-oneshot: /usr/lib/oneshot.d/statefs-03-register-timed - run OK<br>
<br>
<br>
Installing: statefs-contextkit-subscriber-0.3.5-1.4.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/statefs-contextkit-subscriber-0.3.5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libmlocale-qt5-0.7.0-1.3.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libmlocale-qt5-0.7.0-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libzypp-17.3.1+git4-1.6.2.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libzypp-17.3.1+git4-1.6.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
warning: /etc/zypp/zypp.conf created as /etc/zypp/zypp.conf.rpmnew<br>
<br>
<br>
Installing: libaccounts-qt5-devel-1.13+git1-1.3.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libaccounts-qt5-devel-1.13+git1-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-versit-devel-5.2.0+git2-1.2.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-versit-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtxmlpatterns-devel-5.6.3+git2-1.2.1.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtxmlpatterns-devel-5.6.3+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtwidgets-devel-5.6.3+git9-1.9.2.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtwidgets-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsvg-devel-5.6.2+git1-1.2.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtsvg-devel-5.6.2+git1-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mlite-qt5-devel-0.2.25-1.4.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/mlite-qt5-devel-0.2.25-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-qtquickparticles-5.6.3+git7-1.5.1.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-qtquickparticles-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: PackageKit-1.1.9+git5-1.8.1.jolla .....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/PackageKit-1.1.9+git5-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtpim-versitorganizer-devel-5.2.0+git2-1.2.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtpim-versitorganizer-devel-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtopengl-devel-5.6.3+git9-1.9.2.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtopengl-devel-5.6.3+git9-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-qtquicktest-5.6.3+git7-1.5.1.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-qtquicktest-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: PackageKit-zypp-1.1.9+git5-1.8.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/PackageKit-zypp-1.1.9+git5-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-profiled-settings-default-0.15.3-1.3.1.jolla .........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-profiled-settings-default-0.15.3-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/default/rename-profile-settings - job saved OK<br>
<br>
<br>
Installing: qt5-qtwayland-wayland_egl-5.4.0+git48-1.3.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtwayland-wayland_egl-5.4.0+git48-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtquickcontrols-layouts-5.2.1+git3-1.2.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtquickcontrols-layouts-5.2.1+git3-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtmultimedia-5.6.2+git8-1.4.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtmultimedia-5.6.2+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtlocation-5.2.1+git31-1.4.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtlocation-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtgraphicaleffects-5.6.2+git2-1.2.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtgraphicaleffects-5.6.2+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtfeedback-5.2.0+git6-1.3.1.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtfeedback-5.2.0+git6-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdocgallery-5.2.0+git8-1.4.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtdocgallery-5.2.0+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-serviceframework-5.2.0+git9-1.2.1.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtdeclarative-serviceframework-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-qtquickparticles-devel-5.6.3+git7-1.5.1.jolla .......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-qtquickparticles-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-publishsubscribe-5.2.0+git9-1.2.1.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtdeclarative-publishsubscribe-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-plugin-qmlinspector-5.6.3+git7-1.5.1.jolla ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-plugin-qmlinspector-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-pim-organizer-5.2.0+git2-1.2.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-pim-organizer-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-pim-contacts-5.2.0+git2-1.2.1.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-pim-contacts-5.2.0+git2-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-xmllistmodel-5.6.3+git7-1.5.1.jolla ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-xmllistmodel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-window2-5.6.3+git7-1.5.1.jolla ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-window2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-sensors-5.2.1+git17-1.3.1.jolla ..............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-sensors-5.2.1+git17-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-qttest-5.6.3+git7-1.5.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-qttest-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-qtquick2plugin-5.6.3+git7-1.5.1.jolla ........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-qtquick2plugin-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-positioning-5.2.1+git31-1.4.1.jolla ..........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-positioning-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-particles2-5.6.3+git7-1.5.1.jolla ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-particles2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-models2-5.6.3+git7-1.5.1.jolla ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-models2-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-localstorageplugin-5.6.3+git7-1.5.1.jolla ....................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-localstorageplugin-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-folderlistmodel-5.6.3+git7-1.5.1.jolla .......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-folderlistmodel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-devel-5.6.3+git7-1.5.1.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtconnectivity-qtbluetooth-5.6.2+git0-1.2.1.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtconnectivity-qtbluetooth-5.6.2+git0-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-thumbnailer-qt5-1.0.0-1.6.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-thumbnailer-qt5-1.0.0-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
/bin/rm: cannot remove `/home/nemo/.cache/.nemothumbs': Permission denied<br>
add-oneshot: /usr/lib/oneshot.d/remove-obsolete-nemothumbs-cache-dir - could not be run, save for later<br>
add-oneshot: /etc/oneshot.d/0/remove-obsolete-nemothumbs-cache-dir - job saved OK<br>
<br>
<br>
Installing: nemo-qml-plugin-notifications-qt5-1.1.6-1.5.1.jolla ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-notifications-qt5-1.1.6-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-models-qt5-0.1.3-1.4.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-models-qt5-0.1.3-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-filemanager-0.1.11-1.7.1.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-filemanager-0.1.11-1.7.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-dbus-qt5-2.1.20-1.6.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-dbus-qt5-2.1.20-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-configuration-qt5-0.2.2-1.3.1.jolla ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-configuration-qt5-0.2.2-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mpris-qt5-1.0.0-1.4.1.jolla ...........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/mpris-qt5-1.0.0-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mapplauncherd-qt5-1.1.14-1.3.1.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/mapplauncherd-qt5-1.1.14-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqtwebkit5-5.6.2+git8-1.5.1.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libqtwebkit5-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqt5sparql-0.2.18-1.2.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqt5sparql-0.2.18-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libpython3_7m1_0-3.7.2+git2-1.3.5.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libpython3_7m1_0-3.7.2+git2-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: contextkit-declarative-qt5-0.3.5-1.4.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/contextkit-declarative-qt5-0.3.5-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ssu-0.43.12-1.8.1.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/ssu-0.43.12-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: PackageKit-Qt5-0.9.6+git-1.3.1.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/PackageKit-Qt5-0.9.6+git-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: profiled-1.0.6-1.2.1.jolla ............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/profiled-1.0.6-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtwayland-wayland_egl-devel-5.4.0+git48-1.3.1.jolla ...............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtwayland-wayland_egl-devel-5.4.0+git48-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtmultimedia-gsttools-5.6.2+git8-1.4.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtmultimedia-gsttools-5.6.2+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-location-5.2.1+git31-1.4.1.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-location-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtfeedback-devel-5.2.0+git6-1.3.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtfeedback-devel-5.2.0+git6-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdocgallery-devel-5.2.0+git8-1.4.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtdocgallery-devel-5.2.0+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-qtquick-devel-5.6.3+git7-1.5.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-qtquick-devel-5.6.3+git7-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtconnectivity-qtbluetooth-devel-5.6.2+git0-1.2.1.jolla ...........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtconnectivity-qtbluetooth-devel-5.6.2+git0-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: thumbnaild-0.0.6-1.4.1.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/thumbnaild-0.0.6-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
/bin/rm: cannot remove `/home/nemo/.thumbnails': Permission denied<br>
add-oneshot: /usr/lib/oneshot.d/remove-obsolete-tumbler-cache-dir - could not be run, save for later<br>
add-oneshot: /etc/oneshot.d/0/remove-obsolete-tumbler-cache-dir - job saved OK<br>
<br>
<br>
Installing: libnemotransferengine-qt5-1.0.1-1.5.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libnemotransferengine-qt5-1.0.1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mpris-qt5-qml-plugin-1.0.0-1.4.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/mpris-qt5-qml-plugin-1.0.0-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: signon-qt5-8.57.5+git3-1.5.1.jolla ....................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/signon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/0/signon-storage-perm - job saved OK<br>
<br>
<br>
Installing: mapplauncherd-qt5-devel-1.1.14-1.3.1.jolla ............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/mapplauncherd-qt5-devel-1.1.14-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: maliit-framework-wayland-inputcontext-0.99.1+git4-1.4.1.jolla .........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/maliit-framework-wayland-inputcontext-0.99.1+git4-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsailfishapp-1.2.8-1.4.1.jolla ......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/oss/armv7hl/libsailfishapp-1.2.8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: buteo-syncfw-qt5-0.8.18-1.7.1.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/buteo-syncfw-qt5-0.8.18-1.7.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/0/msyncd-storage-perm - job saved OK<br>
su: cannot set groups: Operation not permitted<br>
su: cannot set groups: Operation not permitted<br>
<br>
<br>
Installing: libqtwebkit5-widgets-5.6.2+git8-1.5.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libqtwebkit5-widgets-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqt5sparql-tracker-direct-0.2.18-1.2.1.jolla ........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqt5sparql-tracker-direct-0.2.18-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqt5sparql-tracker-0.2.18-1.2.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqt5sparql-tracker-0.2.18-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: python3-base-3.7.2+git2-1.3.5.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/python3-base-3.7.2+git2-1.3.5.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: zypper-1.14.6+git3-1.3.4.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/zypper-1.14.6+git3-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ssu-vendor-data-jolla-0.108-1.6.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/oss/noarch/ssu-vendor-data-jolla-0.108-1.6.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
DBus unavailable, falling back to libssu<br>
add-oneshot: /usr/lib/oneshot.d/ssu-update-repos - run OK<br>
<br>
<br>
Installing: ngfd-1.1.1-1.5.1.jolla ................................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/ngfd-1.1.1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
su: cannot set groups: Operation not permitted<br>
su: cannot set groups: Operation not permitted<br>
<br>
<br>
Installing: qt5-qtmultimedia-devel-5.6.2+git8-1.4.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtmultimedia-devel-5.6.2+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-import-multimedia-5.6.2+git8-1.4.1.jolla ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtdeclarative-import-multimedia-5.6.2+git8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qtmultimedia-plugins-gstvideotexturebackend-0.0.10-1.3.1.jolla ...................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qtmultimedia-plugins-gstvideotexturebackend-0.0.10-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtlocation-devel-5.2.1+git31-1.4.1.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtlocation-devel-5.2.1+git31-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-transferengine-qt5-1.0.1-1.5.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-transferengine-qt5-1.0.1-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsignon-qt5-8.57.5+git3-1.5.1.jolla .................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libsignon-qt5-8.57.5+git3-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: maliit-framework-wayland-0.99.1+git4-1.4.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/maliit-framework-wayland-0.99.1+git4-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsailfishapp-devel-1.2.8-1.4.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/oss/armv7hl/libsailfishapp-devel-1.2.8-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtwebkit-uiprocess-launcher-5.6.2+git8-1.5.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtwebkit-uiprocess-launcher-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: pyotherside-qml-plugin-python3-qt5-1.5.1+git2-1.3.1.jolla .............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/pyotherside-qml-plugin-python3-qt5-1.5.1+git2-1.3.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-version-variant-3.0.3-1.11.10.jolla ..........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-version-variant-3.0.3-1.11.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtsysteminfo-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libngf-qt5-0.6.2-1.4.1.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libngf-qt5-0.6.2-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: signon-plugin-oauth2-qt5-0.21.5-1.2.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/signon-plugin-oauth2-qt5-0.21.5-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libsignon-qt5-devel-8.57.5+git3-1.5.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libsignon-qt5-devel-8.57.5+git3-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqmfclient1-qt5-4.0.4+git111-1.10.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqmfclient1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: maliit-framework-wayland-devel-0.99.1+git4-1.4.1.jolla ................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/maliit-framework-wayland-devel-0.99.1+git4-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtqml-import-webkitplugin-experimental-5.6.2+git8-1.5.1.jolla .....................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtqml-import-webkitplugin-experimental-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtqml-import-webkitplugin-5.6.2+git8-1.5.1.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/qt5-qtqml-import-webkitplugin-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqtwebkit5-devel-5.6.2+git8-1.5.1.jolla .............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/qt/armv7hl/libqtwebkit5-devel-5.6.2+git8-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-version-3.0.3-1.11.10.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-version-3.0.3-1.11.10.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qt5-qtdeclarative-systeminfo-5.2.0+git9-1.2.1.jolla ...................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qt5-qtdeclarative-systeminfo-5.2.0+git9-1.2.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libcontentaction-qt5-0.3.9-1.6.1.jolla ................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libcontentaction-qt5-0.3.9-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: buteo-mtp-qt5-0.7.0-1.6.1.jolla .......................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/buteo-mtp-qt5-0.7.0-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libngf-qt5-declarative-0.6.2-1.4.1.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libngf-qt5-declarative-0.6.2-1.4.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-contentaction-0.3.9-1.6.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-contentaction-0.3.9-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-graphics-default-base-1.0.17-1.15.1.jolla ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-default-base-1.0.17-1.15.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/0/dconf-update - job saved OK<br>
<br>
<br>
Installing: sailfish-content-graphics-default-z1.0-base-1.0.17-1.15.1.jolla .......................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-default-z1.0-base-1.0.17-1.15.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-graphics-default-z1.0-1.0.17-1.15.1.jolla ............................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-default-z1.0-1.0.17-1.15.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-content-graphics-default-1.0.17-1.15.1.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-graphics-default-1.0.17-1.15.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ambienced-0.29.10-1.9.2.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/ambienced-0.29.10-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
add-oneshot: /etc/oneshot.d/default/late/reset-ambience-profile - job saved OK<br>
<br>
<br>
Installing: sailfish-content-tones-default-0.15.3-1.3.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/sailfish-content-tones-default-0.15.3-1.3.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfishsilica-qt5-1.0.51.1-1.22.1.jolla ..............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfishsilica-qt5-1.0.51.1-1.22.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-components-media-qt5-0.2.2-1.5.1.jolla .......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-components-media-qt5-0.2.2-1.5.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-components-filemanager-0.2.3-1.7.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-components-filemanager-0.2.3-1.7.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libngf-0.26-1.1.8.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/libngf-0.26-1.1.8.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libjollasignonuiservice-qt5-0.4.6-1.6.1.jolla .........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/libjollasignonuiservice-qt5-0.4.6-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: declarative-transferengine-qt5-0.3.9-1.8.1.jolla ......................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/declarative-transferengine-qt5-0.3.9-1.8.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: ambienced-devel-0.29.10-1.9.2.jolla ...................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/ambienced-devel-0.29.10-1.9.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: dsme-0.79.4-1.6.2.jolla ...............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/dsme-0.79.4-1.6.2.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: libjollasignonuiservice-qt5-plugin-0.4.6-1.6.1.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/libjollasignonuiservice-qt5-plugin-0.4.6-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patterns-sailfish-silica-devel-1.0.18-1.11.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-silica-devel-1.0.18-1.11.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: mce-1.100.2-1.16.1.jolla ..............................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/core/armv7hl/mce-1.100.2-1.16.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
Failed to get D-Bus connection: Operation not permitted<br>
<br>
<br>
Installing: sailfish-components-gallery-qt5-1.0.2-1.15.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-components-gallery-qt5-1.0.2-1.15.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: nemo-qml-plugin-systemsettings-0.5.11-1.18.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/nemo-qml-plugin-systemsettings-0.5.11-1.18.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libkeepalive-1.6.2-1.6.1.jolla ........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libkeepalive-1.6.2-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sailfish-components-pickers-qt5-1.0.0-1.6.1.jolla .....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/armv7hl/sailfish-components-pickers-qt5-1.0.0-1.6.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: libqmfmessageserver1-qt5-4.0.4+git111-1.10.1.jolla ....................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/libqmfmessageserver1-qt5-4.0.4+git111-1.10.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: qmf-qt5-devel-4.0.4+git111-1.10.1.jolla ...............................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/mw/armv7hl/qmf-qt5-devel-4.0.4+git111-1.10.1.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patterns-sailfish-qt5-devel-basic-1.0.18-1.11.1.jolla .................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-qt5-devel-basic-1.0.18-1.11.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patterns-sailfish-qt5-devel-full-1.0.18-1.11.1.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-qt5-devel-full-1.0.18-1.11.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sdk-target-configs-0.114-1.9.4.jolla ..................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-target-configs-0.114-1.9.4.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sdk-harbour-rpmvalidator-1.49.1-1.8.1.jolla ...........................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-harbour-rpmvalidator-1.49.1-1.8.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sdk-register-0.5-1.3.4.jolla ..........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/armv7hl/sdk-register-0.5-1.3.4.jolla.armv7hl.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: sdk-configs-0.114-1.9.4.jolla .........................................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/sdk/sdk/noarch/sdk-configs-0.114-1.9.4.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patterns-sailfish-target-support-1.0.18-1.11.1.jolla ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-target-support-1.0.18-1.11.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
Installing: patterns-sailfish-configuration-sdk-target-1.0.18-1.11.1.jolla ........................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/non-oss/noarch/patterns-sailfish-configuration-sdk-target-1.0.18-1.11.1.jolla.noarch.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY<br>
<br>
<br>
There are some running programs that use files deleted by recent upgrade. You may wish to restart some of them. Run 'zypper ps' to list these programs.<br>
</details><br>

Также в ходе выполнения последней команды возникала ошибка с `nothing provides default-rpm-validation-suite needed by patterns-sailfish-target-support...` - решить удалось с помощью полной переустановки песочницы

Запускаем снова сборку пакетов
```console
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details>
* Building rpm/droid-hal-mido.spec<br>
/usr/share/misc/magic, 14816: Warning: type `clear		x' invalid<br>
/usr/share/misc/magic, 19673: Warning: missing ')' in indirect offset<br>
/usr/share/misc/magic, 19673: Warning: invalid string op: ,<br>
/usr/share/misc/magic, 22433: Warning: type `clear	x' invalid<br>
/usr/share/misc/magic, 27164: Warning: Printf format `#' is not valid for type `lelong' in description `version %#x (MVP)'<br>
/usr/share/misc/magic, 27165: Warning: Printf format `#' is not valid for type `lelong' in description `version %#x'<br>
/usr/share/misc/magic, 28422: Warning: type `clear	x' invalid<br>
/usr/share/misc/magic, 28568: Warning: Printf format `#' is not valid for type `leshort' in description `[%#x]'<br>
/usr/share/misc/magic, 28588: Warning: Printf format `#' is not valid for type `leshort' in description `v?[%#x]'<br>
error: magic_load failed: File 5.14 supports only version 10 magic files. `/usr/share/misc/magic.mgc' is version 14<br>
Provides: droid-hal droid-hal-mido = 0.0.6-201907081020 droid-hal-mido(armv7hl-32) = 0.0.6-201907081020<br>
Requires(interp): /bin/sh /bin/sh<br>
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1<br>
Requires(post): /bin/grep /bin/ln /bin/sed /bin/sh /bin/touch /etc/login.defs /usr/bin/add-oneshot /usr/bin/getent systemd<br>
Requires(preun): /bin/sh systemd<br>
Requires(postun): systemd<br>
<br>
<br>
RPM build errors:<br>
    magic_load failed: File 5.14 supports only version 10 magic files. `/usr/share/misc/magic.mgc' is version 14<br>
* Check /home/stalker/hadk/droid-hal-mido.log for full log.<br>
!! building of package failed<br>
</details><br>

Решение описано в https://wiki.merproject.org/wiki/Platform_SDK_and_SB2
```console
PlatformSDK:~/hadk$ mv /parentroot/srv/mer/targets/xiaomi-mido-armv7hl/usr/share/misc/magic.mgc /parentroot/srv/mer/targets/xiaomi-mido-armv7hl/usr/share/misc/magic.mgc.orig
PlatformSDK:~/hadk$ cp /srv/mer/sdks/ubuntu/usr/share/misc/magic.mgc /parentroot/srv/mer/targets/xiaomi-mido-armv7hl/usr/share/misc/magic.mgc
PlatformSDK:~/hadk$ rpm/dhd/helpers/build_packages.sh
```
<details>
* Building rpm/droid-hal-mido.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of droid-hal-mido finished successfully<br>
* pulling updates...<br>
* Building rpm/community-adaptation-localbuild.spec<br>
* Building successful, adding packages to repo<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of community-adaptation finished successfully<br>
* Building rpm/droid-config-mido.spec<br>
* Building successful, adding packages to repo<br>
Retrieving repository 'adaptation-community-common' metadata ......................................................................[done]<br>
Building repository 'adaptation-community-common' cache ...........................................................................[done]<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
Repository 'sdk' is up to date.<br>
All repositories have been refreshed.<br>
* Building of droid-configs finished successfully<br>
Changing domain from sailfish to sales<br>
DBus unavailable, falling back to libssu<br>
Forcing raw metadata refresh<br>
Retrieving repository 'adaptation-community-common' metadata ......................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'adaptation-community-common' cache ...........................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'hotfixes' metadata .........................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'hotfixes' cache ..............................................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'jolla' metadata ............................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'jolla' cache .................................................................................................[done]<br>
Forcing raw metadata refresh<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Forcing building of repository cache<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
Loading repository data...<br>
Reading installed packages...<br>
'droid-hal-mido-devel' is already installed.<br>
No update candidate for 'droid-hal-mido-devel-0.0.6-201907081048.armv7hl'. The highest available version is already installed.<br>
Resolving package dependencies...<br>
<br>
Nothing to do.<br>
Build libhybris? [Y/n/all]all<br>
* pulling updates...<br>
* enabling debugging in libhybris...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/libhybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of libhybris finished successfully<br>
* pulling updates...<br>
* Building rpm/pulseaudio-modules-droid.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of pulseaudio-modules-droid finished successfully<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/mce-plugin-libhybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of mce-plugin-libhybris finished successfully<br>
* pulling updates...<br>
* Building rpm/ngfd-plugin-native-vibrator.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of ngfd-plugin-droid-vibrator finished successfully<br>
* pulling updates...<br>
* Building rpm/qt5-feedback-haptics-native-vibrator.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-feedback-haptics-droid-vibrator finished successfully<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/qt5-qpa-hwcomposer-plugin.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-qpa-hwcomposer-plugin finished successfully<br>
* pulling updates...<br>
* No spec file for package building specified, building all I can find.<br>
* Building rpm/qt5-qpa-surfaceflinger-plugin.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qt5-qpa-surfaceflinger-plugin finished successfully<br>
* pulling updates...<br>
* Building rpm/qtscenegraph-adaptation-droid.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of qtscenegraph-adaptation finished successfully<br>
* pulling updates...<br>
* Building rpm/sensorfw-qt5-hybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of sensorfw finished successfully<br>
* pulling updates...<br>
* Building rpm/geoclue-providers-hybris.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of geoclue-providers-hybris finished successfully<br>
* Building rpm/droid-hal-version-mido.spec<br>
* Building successful, adding packages to repo<br>
Repository 'adaptation-community-common' is up to date.<br>
Repository 'hotfixes' is up to date.<br>
Repository 'jolla' is up to date.<br>
Retrieving repository 'local-mido-hal' metadata ...................................................................................[done]<br>
Building repository 'local-mido-hal' cache ........................................................................................[done]<br>
All repositories have been refreshed.<br>
* Building of droid-hal-version-mido finished successfully<br>
----------------------DONE! Now proceed on creating the rootfs------------------<br>
</details><br>

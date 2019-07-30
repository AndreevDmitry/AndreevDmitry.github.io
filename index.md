###Вольный перевод Sailfish OS HADK версии 3.0.1.0 (https://sailfishos.org/content/uploads/2019/03/SailfishOS-HardwareAdaptationDevelopmentKit-3.0.1.0.pdf) от 15 марта 2019 с примером портирования на Xiaomi Mido (а также проблемами, решениями и выводами вводимых команд)

**Любые действия Вы делаете на свой страх и риск, помните, что при неумелом обращении с некоторыми командами (например `sudo rm -rf / srv`) вы можете удалить корень системы, лучше делайте копипаст и проверяйте, а не перепечатывайте.**

**Пожалуйста, прежде чем делать что-либо дочитайте руководство до конца, чтобы не повторять описанных ошибок.**

Портирование производилось на Ubuntu 18.04 x64 cо следующими предустановленными программами:
- adb
- fastboot

#Установка переменных окружения
```bash
`stalker@stalkerPC:~$ cat <<'EOF' > $HOME/.hadk.env`
`export PLATFORM_SDK_ROOT="/srv/mer"`
`export ANDROID_ROOT="$HOME/hadk"`
`export VENDOR="xiaomi"`
`export DEVICE="mido"`
`# Set arch to armv7hl even if you are porting a 64bit device`
`export PORT_ARCH="armv7hl"`
`EOF`
```
На выходе получаем скрытый файл .hadk.env в вашей домашней директории

`stalker@stalkerPC:~$ cat <<'EOF' >> $HOME/.mersdkubu.profile`
`function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }`
`export PS1="HABUILD_SDK [\${DEVICE}] $PS1"`
`hadk`
`EOF`

На выходе получаем скрытый файл .mersdkubu.profile в вашей домашней директории

#Устанавливаем Platform SDK 2.1.1
Скачиваем архив песочницы крайней на момент написания перевода версии 2.1.1 в которой в дальшейшем будем собирать Sailfish
`stalker@stalkerPC:~$ curl -k -O http://releases.sailfishos.org/sdk/installers/latest/Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 ;`
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100  137M  100  137M    0     0  14.4M      0  0:00:09  0:00:09 --:--:-- 17.9M
</details>

Распаковываем скачанный архив
`stalker@stalkerPC:~$ sudo tar --numeric-owner -p -xjf Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk;`
Проверяем, что все распаковалось
`stalker@stalkerPC:~$ ls -lah /srv/mer/sdks/sfossdk/`
<details>
итого 96K
drwxr-xr-x 20 root root 4,0K мая  6 17:49 .
drwxr-xr-x  3 root root 4,0K июн 20 22:13 ..
drwxr-xr-x  2 root root 4,0K мая  6 17:49 bin
drwxr-xr-x  2 root root 4,0K мар 28 07:35 boot
drwxr-xr-x  3 root root 4,0K мар 28 07:35 dev
drwxr-xr-x 51 root root 4,0K мая  6 17:49 etc
drwxr-xr-x  3 root root 4,0K мая  6 17:49 home
drwxr-xr-x  6 root root 4,0K мая  6 17:49 lib
drwxr-xr-x  2 root root 4,0K мар 28 07:35 media
-rwxr-xr-x  1 root root  754 апр 16 09:26 mer-bash-setup
-rwxr-xr-x  1 root root  11K апр 16 09:26 mer-sdk-chroot
drwxr-xr-x  2 root root 4,0K мар 28 07:35 mnt
drwxr-xr-x  2 root root 4,0K мар 28 07:35 opt
drwxr-xr-x  2 root root 4,0K мая  6 17:48 proc
drwxr-x---  2 root root 4,0K мая  6 17:49 root
drwxr-xr-x 15 root root 4,0K мая  6 17:49 run
drwxr-xr-x  2 root root 4,0K мая  6 17:49 sbin
drwxr-xr-x  3 root root 4,0K мая  6 17:49 srv
drwxr-xr-x  2 root root 4,0K мая  6 17:48 sys
drwxrwxrwt  2 root root 4,0K мая  6 17:49 tmp
drwxr-xr-x 12 root root 4,0K мая  6 17:49 usr
drwxr-xr-x 17 root root 4,0K мая  6 17:49 var
</details>

Создаем короткое имя (alias) для входа в песочницу Sailfish OS SDK (она же Platform SDK, она же Mer SDK)
`stalker@stalkerPC:~$ echo "export PLATFORM_SDK_ROOT=$PLATFORM_SDK_ROOT" >> ~/.bashrc`
`stalker@stalkerPC:~$ echo 'alias sfossdk=$PLATFORM_SDK_ROOT/sdks/sfossdk/mer-sdk-chroot' >> ~/.bashrc ; exec bash ;`

Соответственно в конец файла .bashrc должны были добавиться 2 строки, проверяем
`stalker@stalkerPC:~$ tail -n2 .bashrc`
<details>
export PLATFORM_SDK_ROOT=/srv/mer
alias sfossdk=$PLATFORM_SDK_ROOT/sdks/sfossdk/mer-sdk-chroot
</details>

Создаем вспомогательный файл для идентификации работы в песочнице
`stalker@stalkerPC:~$ echo 'PS1="PlatformSDK $PS1"' > ~/.mersdk.profile ;`
`stalker@stalkerPC:~$ echo '[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done'  >> ~/.mersdk.profile ;`

Проверяем
`stalker@stalkerPC:~$ cat .mersdk.profile`
<details>
PS1="PlatformSDK $PS1"
[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done
</details>

Заходим в песочницу,
`stalker@stalkerPC:~$ sfossdk`
<details>
SDK targets location '/srv/mer/targets' does not exist - about to create it.
Continue, abort? [c/a] (c)
SDK toolings location '/srv/mer/toolings' does not exist - about to create it.
Continue, abort? [c/a] (c)
ls: невозможно открыть каталог '/proc/sys/fs/binfmt_misc': Слишком много уровней символьных ссылок
Mounting system directories...
mount: /srv/mer/sdks/sfossdk/proc/sys/fs/binfmt_misc: mount(2) system call failed: Слишком много уровней символьных ссылок.
Mounting /srv/mer/targets as /srv/mer/targets
Mounting /srv/mer/toolings as /srv/mer/toolings
Mounting / as /parentroot
Mounting home directory: /home/stalker
Initializing machine ID from random generator.
Entering chroot as stalker
PlatformSDK stalker@stalkerPC:~$
</details>

Соответственно видим, что все вводимые команды выполнятся из окружения Platform SDK (и как следствие имеем набор команд и параметров который предустановлен в данной песочнице), но находимся мы в домашнем каталоге.

Выходим из песочницы
`stalker@stalkerPC:~$ exit`
<details>
exit
stalker@stalkerPC:~$
</details>

Добавляем идентификацию устройства на которое будем портировать Sailfish
`stalker@stalkerPC:~$ cat <<'EOF' >> $HOME/.mersdk.profile`
`function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }`
`hadk`
`EOF`

Соответственно в конце файла .mersdk.profile добавились 2 строки, проверяем
`stalker@stalkerPC:~$ tail -n2 .mersdk.profile`
<details>
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }
hadk
</details>

Заходим снова в PlatformSDK
`stalker@stalkerPC:~$ sfossdk`
<details>
Mounting system directories...
Mounting /srv/mer/targets as /srv/mer/targets
Mounting /srv/mer/toolings as /srv/mer/toolings
Mounting / as /parentroot
Mounting home directory: /home/stalker
Entering chroot as stalker
Last login: Thu Jun 20 17:39:38 UTC 2019 on pts/0
Env setup for mido
</details>

Пытаемся обновить репозитории нашей песочницы
`PlatformSDK stalker@stalkerPC:~$ sudo zypper ref`
<details>
Retrieving repository 'adaptation0' metadata .....................................................................................[error]
Repository 'adaptation0' is invalid.
[adaptation0|plugin:/ssu?repo=adaptation0] Valid metadata not found at specified URL
Please check if the URIs defined for this repository are pointing to a valid repository.
Skipping repository 'adaptation0' because of the above error.
Retrieving repository 'customer-jolla' metadata ...................................................................................[done]
Building repository 'customer-jolla' cache ........................................................................................[done]
Retrieving repository 'hotfixes' metadata .........................................................................................[done]
Building repository 'hotfixes' cache ..............................................................................................[done]
Retrieving repository 'jolla' metadata ............................................................................................[done]
Building repository 'jolla' cache .................................................................................................[done]
Retrieving repository 'sdk' metadata ..............................................................................................[done]
Building repository 'sdk' cache ...................................................................................................[done]
Some of the repositories have not been refreshed because of an error.
</details>

Выходим из sdk
`PlatformSDK stalker@stalkerPC:~$ exit`

Удаляем ее
`stalker@stalkerPC:~$ sudo rm -rf /srv`

Скачиваем версию 2.0 для сборки Sailfish 3.0.2.8

`stalker@stalkerPC:~$ curl -k -O  http://releases.sailfishos.org/sdk/installers/2.0/Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2`
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100  135M  100  135M    0     0  10.2M      0  0:00:13  0:00:13 --:--:-- 11.6M
</details>

Создаем папку для установки песочницы
`stalker@stalkerPC:~$ sudo mkdir -p $PLATFORM_SDK_ROOT/sdks/sfossdk ;`

Распаковываем скачанный архив
`stalker@stalkerPC:~$ sudo tar --numeric-owner -p -xjf Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk  ;`

Проверяем что все распаковалось, должно быть примерно так
`stalker@stalkerPC:~$ ls -lah /srv/mer/sdks/sfossdk/`
<details>
итого 96K
drwxr-xr-x 20 root root 4,0K мар 19 17:34 .
drwxr-xr-x  3 root root 4,0K июн 21 21:29 ..
drwxr-xr-x  2 root root 4,0K мар 19 17:33 bin
drwxr-xr-x  2 root root 4,0K мар  4 04:43 boot
drwxr-xr-x  3 root root 4,0K мар  4 04:43 dev
drwxr-xr-x 51 root root 4,0K мар 19 17:34 etc
drwxr-xr-x  3 root root 4,0K мар 19 17:34 home
drwxr-xr-x  7 root root 4,0K мар 19 17:33 lib
drwxr-xr-x  2 root root 4,0K мар  4 04:43 media
-rwxr-xr-x  1 root root  754 мар 13 00:39 mer-bash-setup
-rwxr-xr-x  1 root root  11K мар 13 00:39 mer-sdk-chroot
drwxr-xr-x  2 root root 4,0K мар  4 04:43 mnt
drwxr-xr-x  2 root root 4,0K мар  4 04:43 opt
drwxr-xr-x  2 root root 4,0K мар 19 17:33 proc
drwxr-x---  2 root root 4,0K мар 19 17:33 root
drwxr-xr-x 14 root root 4,0K мар 19 17:34 run
drwxr-xr-x  2 root root 4,0K мар 19 17:34 sbin
drwxr-xr-x  3 root root 4,0K мар 19 17:33 srv
drwxr-xr-x  2 root root 4,0K мар 19 17:33 sys
drwxrwxrwt  2 root root 4,0K мар 19 17:34 tmp
drwxr-xr-x 12 root root 4,0K мар 19 17:33 usr
drwxr-xr-x 17 root root 4,0K мар 19 17:33 var
</details>

Заходим в sdk
`stalker@stalkerPC:~$ sfossdk`
<details>
SDK targets location '/srv/mer/targets' does not exist - about to create it.
Continue, abort? [c/a] (c)
SDK toolings location '/srv/mer/toolings' does not exist - about to create it.
Continue, abort? [c/a] (c)
Mounting system directories...
Mounting /srv/mer/targets as /srv/mer/targets
Mounting /srv/mer/toolings as /srv/mer/toolings
Mounting / as /parentroot
Mounting home directory: /home/stalker
Initializing machine ID from random generator.
Entering chroot as stalker
Env setup for mido

Did you know…?

Deploying with `rpm -Uvh --force` can hide bugs. `mb2` produces evergrowing
version numbers unless told otherwise, so `zypper -p <rpms-dir> -v dup`
usually works well and obeys the dependencies.

Learn more on <https://sailfishos.org/wiki/SDK_Tips#Deploying_without_force>.
</details>

Пытаемся обновить репозитории нашей песочницы
`PlatformSDK stalker@stalkerPC:~$ sudo zypper ref`
<details>
Retrieving repository 'adaptation0' metadata ......................................................................................[done]
Building repository 'adaptation0' cache ...........................................................................................[done]
Retrieving repository 'customer-jolla' metadata ...................................................................................[done]
Building repository 'customer-jolla' cache ........................................................................................[done]
Retrieving repository 'hotfixes' metadata .........................................................................................[done]
Building repository 'hotfixes' cache ..............................................................................................[done]
Retrieving repository 'jolla' metadata ............................................................................................[done]
Building repository 'jolla' cache .................................................................................................[done]
Retrieving repository 'sdk' metadata ..............................................................................................[done]
Building repository 'sdk' cache ...................................................................................................[done]
All repositories have been refreshed.
</details>

Обновляем компоненты SDK
`PlatformSDK stalker@stalkerPC:~$ sudo zypper up`
<details>
Loading repository data...
Reading installed packages...

Nothing to do.
</details>

Проверяем, что используем версию 3.0.3.10 Platform SDK
`PlatformSDK stalker@stalkerPC:~$** cat /etc/os-release`
<details>
NAME="Sailfish OS"
ID=sailfishos
VERSION="3.0.3.10 (Hossa)"
VERSION_ID=3.0.3.10
PRETTY_NAME="Sailfish OS 3.0.3.10 (Hossa)"
SAILFISH_BUILD=10
SAILFISH_FLAVOUR=release
HOME_URL="https://sailfishos.org/"
</details>

Устанавливаем необходимые утилиты для работы с Android SDK (далее):
`PlatformSDK stalker@stalkerPC:~$ sudo zypper in android-tools-hadk tar`
<details>
Loading repository data...
Reading installed packages...
'tar' is already installed.
No update candidate for 'tar-1.17-1.3.1.jolla.i486'. The highest available version is already installed.
Resolving package dependencies...

The following NEW package is going to be installed:
  android-tools-hadk

1 new package to install.
Overall download size: 122.0 KiB. Already cached: 0 B. After the operation, additional 306.5 KiB will be used.
Continue? [y/n/...? shows all options] (y): y
Retrieving package android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486                                   (1/1), 122.0 KiB (306.5 KiB unpacked)
Retrieving: android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486.rpm ..........................................................[done (154 B/s)]
Checking for file conflicts: ......................................................................................................[done]
(1/1) Installing: android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486 ..................................................................[done]
Additional rpm output:
warning: /var/cache/zypp/packages/jolla/tools/i486/android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY
</details>

Насколько я понял на warning'и можно не обращать внимание, т.к. у нас не установен ключ для проверки подлинности устанавливаемых пакетов (см. http://www.rhd.ru/docs/manuals/enterprise/RHEL-4-Manual/sysadmin-guide/s1-rpm-using.html)

Далее устанавливаем среду для сборки ядра Android.
`PlatformSDK stalker@stalkerPC:~$ TARBALL=ubuntu-trusty-20180613-android-rootfs.tar.bz2`
`PlatformSDK stalker@stalkerPC:~$ curl -O https://releases.sailfishos.org/ubu/$TARBALL`
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                               Dload  Upload   Total   Spent    Left  Speed
100  440M  100  440M    0     0  16.7M      0  0:00:26  0:00:26 --:--:-- 17.2M
</details>

`PlatformSDK stalker@stalkerPC:~$ UBUNTU_CHROOT=$PLATFORM_SDK_ROOT/sdks/ubuntu`
`PlatformSDK stalker@stalkerPC:~$ sudo mkdir -p $UBUNTU_CHROOT`

Проверяем, комплектацию среды
`PlatformSDK stalker@stalkerPC:~$ ls -lah /srv/mer/sdks/ubuntu/`
<details>
total 96K
drwxr-xr-x 24 root root 4.0K Jun 13  2018 .
drwxr-xr-x  3 root root 4.0K Jun 21 17:38 ..
drwxr-xr-x  2 root root 4.0K Jun 13  2018 bin
drwxr-xr-x  2 root root 4.0K Apr 10  2014 boot
drwxr-xr-x  3 root root 4.0K Jun 13  2018 dev
drwxr-xr-x 82 root root 4.0K Jun 13  2018 etc
drwxr-xr-x  2 root root 4.0K Jun 13  2018 home
drwxr-xr-x 14 root root 4.0K Jun 13  2018 lib
drwxr-xr-x  2 root root 4.0K Jun 13  2018 lib32
drwxr-xr-x  2 root root 4.0K Jun 13  2018 lib64
drwxr-xr-x  2 root root 4.0K Jun 13  2018 libx32
drwxr-xr-x  2 root root 4.0K Jun 13  2018 media
drwxr-xr-x  2 root root 4.0K Apr 10  2014 mnt
drwxr-xr-x  2 root root 4.0K Jun 13  2018 opt
drwxr-xr-x  2 root root 4.0K Jun 13  2018 parentroot
drwxr-xr-x  2 root root 4.0K Apr 10  2014 proc
drwx------  2 root root 4.0K Jun 13  2018 root
drwxr-xr-x  8 root root 4.0K Jun 13  2018 run
drwxr-xr-x  2 root root 4.0K Jun 13  2018 sbin
drwxr-xr-x  2 root root 4.0K Jun 13  2018 srv
drwxr-xr-x  2 root root 4.0K Mar 13  2014 sys
drwxrwxrwt  5 root root 4.0K Jun 13  2018 tmp
drwxr-xr-x 15 root root 4.0K Jun 13  2018 usr
drwxr-xr-x 11 root root 4.0K Jun 13  2018 var
</details>

Заходим в среду
PlatformSDK stalker@stalkerPC:~$ ubu-chroot -r $PLATFORM_SDK_ROOT/sdks/ubuntu
<details>
Env setup for mido
HABUILD_SDK [mido] stalker@stalkerPC:~$
</details>

Позволю себе напомнить, что находясь в среде сборки ядра мы также имеем в распоряжении только те инструменты и параметры, которые установлены в этой среде

Если вы еще не зарегистрировались на github.com, то сделаейте это, т.к. в дальнейшем будем использовать git.
Установим имя и электронную почту для git
`HABUILD_SDK [mido] stalker@stalkerPC:~$ git config --global user.name "Your Name"`
`HABUILD_SDK [mido] stalker@stalkerPC:~$ git config --global user.email "you@example.com"`

Далее установим программу для синхронизации локального хранилища с глобальным
`HABUILD_SDK [mido] stalker@stalkerPC:~$ mkdir ~/bin`
`HABUILD_SDK [mido] stalker@stalkerPC:~$ PATH=~/bin:$PATH`
`HABUILD_SDK [mido] stalker@stalkerPC:~$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo`
<details>
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 29142  100 29142    0     0  47424      0 --:--:-- --:--:-- --:--:-- 47462
</details>
`HABUILD_SDK [mido] stalker@stalkerPC:~$ chmod a+x ~/bin/repo`

Теперь нужно определиться с какой версией hybris будем синхронизироваться, т.к. на Mido есть две версии Android: (6 и 7).
Смотрим какая установлена у вас в Настройки-О телефоне, если 6 то нужно использовать hybris-13.0 (либо лучше обновиться до 7), если 7 то hybris-14.1.
Дальнейшее описание будет выполнено для версии 14.1

Создаем папку, где будет располагаться все, что нам нужно для сборки
`HABUILD_SDK [mido] stalker@stalkerPC:~$ sudo mkdir -p $ANDROID_ROOT`
`HABUILD_SDK [mido] stalker@stalkerPC:~$ sudo chown -R $USER $ANDROID_ROOT`
`HABUILD_SDK [mido] stalker@stalkerPC:~$ cd $ANDROID_ROOT`
Синхронизируемся с  репозиторием
`HABUILD_SDK [mido] stalker@stalkerPC:~/hadk$ repo init -u git://github.com/mer-hybris/android.git -b hybris-14.1`
<details>
Get https://gerrit.googlesource.com/git-repo/clone.bundle
Get https://gerrit.googlesource.com/git-repo
Get git://github.com/mer-hybris/android.git
remote: Enumerating objects: 13, done.
remote: Counting objects: 100% (13/13), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 5464 (delta 5), reused 9 (delta 4), pack-reused 5451
Receiving objects: 100% (5464/5464), 1.82 MiB | 3.59 MiB/s, done.
Resolving deltas: 100% (2009/2009), done.
From git://github.com/mer-hybris/android
...
Syncing work tree: 100% (172/172), done.
</details>
Подготовим среду для сборки
`HABUILD_SDK [mido] stalker@stalkerPC:~/hadk$ source build/envsetup.sh`
<details>
including vendor/cm/vendorsetup.sh
including vendor/cm/bash_completion/git.bash
including vendor/cm/bash_completion/repo.bash
</details>

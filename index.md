# Вольный перевод Sailfish OS HADK версии 3.0.1.0 (https://sailfishos.org/content/uploads/2019/03/SailfishOS-HardwareAdaptationDevelopmentKit-3.0.1.0.pdf) от 15 марта 2019 с примером портирования на Xiaomi Mido (а также проблемами, решениями и выводами вводимых команд)

**Любые действия Вы делаете на свой страх и риск, помните, что при неумелом обращении с некоторыми командами (например `sudo rm -rf / srv`) вы можете удалить корень системы, лучше делайте копипаст и проверяйте, а не перепечатывайте.**

**Пожалуйста, прежде чем делать что-либо дочитайте руководство до конца, чтобы не повторять описанных ошибок.**

Портирование производилось на Ubuntu 18.04 x64 cо следующими предустановленными программами:
- adb
- fastboot

# Установка переменных окружения
```console
HOST:~$ cat <<'EOF' > $HOME/.hadk.env
export PLATFORM_SDK_ROOT="/srv/mer"
export ANDROID_ROOT="$HOME/hadk"
export VENDOR="xiaomi"
export DEVICE="mido"
# Set arch to armv7hl even if you are porting a 64bit device
export PORT_ARCH="armv7hl"
EOF
```
На выходе получаем скрытый файл .hadk.env в вашей домашней директории
```console
HOST:~$ cat <<'EOF' >> $HOME/.mersdkubu.profile
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }
export PS1="HABUILD_SDK [\${DEVICE}] $PS1"
hadk
EOF
```

На выходе получаем скрытый файл .mersdkubu.profile в вашей домашней директории

# Устанавливаем Platform SDK 2.1.1
Скачиваем архив песочницы крайней на момент написания перевода версии 2.1.1 в которой в дальшейшем будем собирать Sailfish
```console
HOST:~$ curl -k -O http://releases.sailfishos.org/sdk/installers/latest/Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 ;
```

<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
 100  137M  100  137M    0     0  14.4M      0  0:00:09  0:00:09 --:--:-- 17.9M<br>
</details><br>

Распаковываем скачанный архив
```console
HOST:~$ sudo tar --numeric-owner -p -xjf Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk;
```
Проверяем, что все распаковалось
```console
HOST:~$ ls -lah /srv/mer/sdks/sfossdk/
```

<details>
итого 96K<br>
drwxr-xr-x 20 root root 4,0K мая  6 17:49 .<br>
drwxr-xr-x  3 root root 4,0K июн 20 22:13 ..<br>
drwxr-xr-x  2 root root 4,0K мая  6 17:49 bin<br>
drwxr-xr-x  2 root root 4,0K мар 28 07:35 boot<br>
drwxr-xr-x  3 root root 4,0K мар 28 07:35 dev<br>
drwxr-xr-x 51 root root 4,0K мая  6 17:49 etc<br>
drwxr-xr-x  3 root root 4,0K мая  6 17:49 home<br>
drwxr-xr-x  6 root root 4,0K мая  6 17:49 lib<br>
drwxr-xr-x  2 root root 4,0K мар 28 07:35 media<br>
-rwxr-xr-x  1 root root  754 апр 16 09:26 mer-bash-setup<br>
-rwxr-xr-x  1 root root  11K апр 16 09:26 mer-sdk-chroot<br>
drwxr-xr-x  2 root root 4,0K мар 28 07:35 mnt<br>
drwxr-xr-x  2 root root 4,0K мар 28 07:35 opt<br>
drwxr-xr-x  2 root root 4,0K мая  6 17:48 proc<br>
drwxr-x---  2 root root 4,0K мая  6 17:49 root<br>
drwxr-xr-x 15 root root 4,0K мая  6 17:49 run<br>
drwxr-xr-x  2 root root 4,0K мая  6 17:49 sbin<br>
drwxr-xr-x  3 root root 4,0K мая  6 17:49 srv<br>
drwxr-xr-x  2 root root 4,0K мая  6 17:48 sys<br>
drwxrwxrwt  2 root root 4,0K мая  6 17:49 tmp<br>
drwxr-xr-x 12 root root 4,0K мая  6 17:49 usr<br>
drwxr-xr-x 17 root root 4,0K мая  6 17:49 var<br>
</details><br>


Создаем короткое имя (alias) для входа в песочницу Sailfish OS SDK (она же Platform SDK, она же Mer SDK)
```console
HOST:~$ echo "export PLATFORM_SDK_ROOT=$PLATFORM_SDK_ROOT" >> ~/.bashrc
HOST:~$ echo 'alias sfossdk=$PLATFORM_SDK_ROOT/sdks/sfossdk/mer-sdk-chroot' >> ~/.bashrc ; exec bash ;
```

Соответственно в конец файла .bashrc должны были добавиться 2 строки, проверяем
```console
HOST:~$ tail -n2 .bashrc
```
<details>
export PLATFORM_SDK_ROOT=/srv/mer<br>
alias sfossdk=$PLATFORM_SDK_ROOT/sdks/sfossdk/mer-sdk-chroot<br>
</details><br>

Создаем вспомогательный файл для идентификации работы в песочнице
```console
HOST:~$ echo 'PS1="PlatformSDK $PS1"' > ~/.mersdk.profile ;
HOST:~$ echo '[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done'  >> ~/.mersdk.profile ;
```

Проверяем
```console
HOST:~$ cat .mersdk.profile
```

<details>
```console
PS1="PlatformSDK $PS1"
[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done
```
</details><br>


Заходим в песочницу,
```console
HOST:~$ sfossdk
```

<details>
```console
SDK targets location '/srv/mer/targets' does not exist - about to create it.<br>
Continue, abort? [c/a] (c)<br>
SDK toolings location '/srv/mer/toolings' does not exist - about to create it.<br>
Continue, abort? [c/a] (c)<br>
ls: невозможно открыть каталог '/proc/sys/fs/binfmt_misc': Слишком много уровней символьных ссылок<br>
Mounting system directories...<br>
mount: /srv/mer/sdks/sfossdk/proc/sys/fs/binfmt_misc: mount(2) system call failed: Слишком много уровней символьных ссылок.<br>
Mounting /srv/mer/targets as /srv/mer/targets<br>
Mounting /srv/mer/toolings as /srv/mer/toolings<br>
Mounting / as /parentroot<br>
Mounting home directory: /home/stalker<br>
Initializing machine ID from random generator.<br>
Entering chroot as stalker<br>
PlatformSDK:~$<br>
```
</details><br>

Соответственно видим, что все вводимые команды выполнятся из окружения Platform SDK (и как следствие имеем набор команд и параметров который предустановлен в данной песочнице), но находимся мы в домашнем каталоге.

Выходим из песочницы
```console
PlatformSDK:~$ exit
```
Добавляем идентификацию устройства на которое будем портировать Sailfish
```console
HOST:~$ cat <<'EOF' >> $HOME/.mersdk.profile
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }
hadk
EOF
```


Соответственно в конце файла .mersdk.profile добавились 2 строки, проверяем
```console
HOST:~$ tail -n2 .mersdk.profile
```

<details>
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }<br>
hadk<br>
</details><br>

Заходим снова в PlatformSDK
```console
HOST:~$ sfossdk
```

<details>
Mounting system directories...<br>
Mounting /srv/mer/targets as /srv/mer/targets<br>
Mounting /srv/mer/toolings as /srv/mer/toolings<br>
Mounting / as /parentroot<br>
Mounting home directory: /home/stalker<br>
Entering chroot as stalker<br>
Last login: Thu Jun 20 17:39:38 UTC 2019 on pts/0<br>
Env setup for mido<br>
</details><br>

Пытаемся обновить репозитории нашей песочницы
```console
PlatformSDK:~$ sudo zypper ref
```

<details>
Retrieving repository 'adaptation0' metadata .....................................................................................[error]<br>
Repository 'adaptation0' is invalid.<br>
[adaptation0|plugin:/ssu?repo=adaptation0] Valid metadata not found at specified URL<br>
Please check if the URIs defined for this repository are pointing to a valid repository.<br>
Skipping repository 'adaptation0' because of the above error.<br>
Retrieving repository 'customer-jolla' metadata ...................................................................................[done]<br>
Building repository 'customer-jolla' cache ........................................................................................[done]<br>
Retrieving repository 'hotfixes' metadata .........................................................................................[done]<br>
Building repository 'hotfixes' cache ..............................................................................................[done]<br>
Retrieving repository 'jolla' metadata ............................................................................................[done]<br>
Building repository 'jolla' cache .................................................................................................[done]<br>
Retrieving repository 'sdk' metadata ..............................................................................................[done]<br>
Building repository 'sdk' cache ...................................................................................................[done]<br>
Some of the repositories have not been refreshed because of an error.<br>
</details><br>

Выходим из sdk
```console
PlatformSDK:~$ exit
```

Удаляем ее
```console
HOST:~$ sudo rm -rf /srv
```

Скачиваем версию 2.0 для сборки Sailfish 3.0.2.8

```console
HOST:~$ curl -k -O  http://releases.sailfishos.org/sdk/installers/2.0/Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2
```

<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100  135M  100  135M    0     0  10.2M      0  0:00:13  0:00:13 --:--:-- 11.6M
</details><br>

Создаем папку для установки песочницы
```console
HOST:~$ sudo mkdir -p $PLATFORM_SDK_ROOT/sdks/sfossdk ;
```

Распаковываем скачанный архив
```console
HOST:~$ sudo tar --numeric-owner -p -xjf Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk  ;
```

Проверяем что все распаковалось, должно быть примерно так
```console
HOST:~$ ls -lah /srv/mer/sdks/sfossdk/
```
<details>
итого 96K<br>
drwxr-xr-x 20 root root 4,0K мар 19 17:34 .<br>
drwxr-xr-x  3 root root 4,0K июн 21 21:29 ..<br>
drwxr-xr-x  2 root root 4,0K мар 19 17:33 bin<br>
drwxr-xr-x  2 root root 4,0K мар  4 04:43 boot<br>
drwxr-xr-x  3 root root 4,0K мар  4 04:43 dev<br>
drwxr-xr-x 51 root root 4,0K мар 19 17:34 etc<br>
drwxr-xr-x  3 root root 4,0K мар 19 17:34 home<br>
drwxr-xr-x  7 root root 4,0K мар 19 17:33 lib<br>
drwxr-xr-x  2 root root 4,0K мар  4 04:43 media<br>
-rwxr-xr-x  1 root root  754 мар 13 00:39 mer-bash-setup<br>
-rwxr-xr-x  1 root root  11K мар 13 00:39 mer-sdk-chroot<br>
drwxr-xr-x  2 root root 4,0K мар  4 04:43 mnt<br>
drwxr-xr-x  2 root root 4,0K мар  4 04:43 opt<br>
drwxr-xr-x  2 root root 4,0K мар 19 17:33 proc<br>
drwxr-x---  2 root root 4,0K мар 19 17:33 root<br>
drwxr-xr-x 14 root root 4,0K мар 19 17:34 run<br>
drwxr-xr-x  2 root root 4,0K мар 19 17:34 sbin<br>
drwxr-xr-x  3 root root 4,0K мар 19 17:33 srv<br>
drwxr-xr-x  2 root root 4,0K мар 19 17:33 sys<br>
drwxrwxrwt  2 root root 4,0K мар 19 17:34 tmp<br>
drwxr-xr-x 12 root root 4,0K мар 19 17:33 usr<br>
drwxr-xr-x 17 root root 4,0K мар 19 17:33 var
</details><br>

Заходим в sdk
```console
HOST:~$ sfossdk
```
<details>
SDK targets location '/srv/mer/targets' does not exist - about to create it.<br>
Continue, abort? [c/a] (c)<br>
SDK toolings location '/srv/mer/toolings' does not exist - about to create it.<br>
Continue, abort? [c/a] (c)<br>
Mounting system directories...<br>
Mounting /srv/mer/targets as /srv/mer/targets<br>
Mounting /srv/mer/toolings as /srv/mer/toolings<br>
Mounting / as /parentroot<br>
Mounting home directory: /home/stalker<br>
Initializing machine ID from random generator.<br>
Entering chroot as stalker<br>
Env setup for mido<br>
</details><br>

Пытаемся обновить репозитории нашей песочницы
```console
PlatformSDK:~$ sudo zypper ref
```
<details>
Retrieving repository 'adaptation0' metadata ......................................................................................[done]<br>
Building repository 'adaptation0' cache ...........................................................................................[done]<br>
Retrieving repository 'customer-jolla' metadata ...................................................................................[done]<br>
Building repository 'customer-jolla' cache ........................................................................................[done]<br>
Retrieving repository 'hotfixes' metadata .........................................................................................[done]<br>
Building repository 'hotfixes' cache ..............................................................................................[done]<br>
Retrieving repository 'jolla' metadata ............................................................................................[done]<br>
Building repository 'jolla' cache .................................................................................................[done]<br>
Retrieving repository 'sdk' metadata ..............................................................................................[done]<br>
Building repository 'sdk' cache ...................................................................................................[done]<br>
All repositories have been refreshed.
</details><br>

Обновляем компоненты SDK
```console
PlatformSDK:~$ sudo zypper up
```
<details>
Loading repository data...<br>
Reading installed packages...<br>
<br>
Nothing to do.
</details><br>

Проверяем, что используем версию 3.0.3.10 Platform SDK
```console
PlatformSDK:~$** cat /etc/os-release
```
<details>
NAME="Sailfish OS"<br>
ID=sailfishos<br>
VERSION="3.0.3.10 (Hossa)"<br>
VERSION_ID=3.0.3.10<br>
PRETTY_NAME="Sailfish OS 3.0.3.10 (Hossa)"<br>
SAILFISH_BUILD=10<br>
SAILFISH_FLAVOUR=release<br>
HOME_URL="https://sailfishos.org/"
</details><br>

Устанавливаем необходимые утилиты для работы с Android SDK (далее):
```console
PlatformSDK:~$ sudo zypper in android-tools-hadk tar
```
<details>
Loading repository data...<br>
Reading installed packages...<br>
'tar' is already installed.<br>
No update candidate for 'tar-1.17-1.3.1.jolla.i486'. The highest available version is already installed.<br>
Resolving package dependencies...<br>
<br>
The following NEW package is going to be installed:<br>
  android-tools-hadk<br>
<br>
1 new package to install.<br>
Overall download size: 122.0 KiB. Already cached: 0 B. After the operation, additional 306.5 KiB will be used.<br>
Continue? [y/n/...? shows all options] (y): y<br>
Retrieving package android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486                                   (1/1), 122.0 KiB (306.5 KiB unpacked)<br>
Retrieving: android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486.rpm ..........................................................[done (154 B/s)]<br>
Checking for file conflicts: ......................................................................................................[done]<br>
(1/1) Installing: android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486 ..................................................................[done]<br>
Additional rpm output:<br>
warning: /var/cache/zypp/packages/jolla/tools/i486/android-tools-hadk-5.1.1+git2-1.2.3.jolla.i486.rpm: Header V3 DSA/SHA1 Signature, key ID f2633ee0: NOKEY
</details><br>

Насколько я понял на warning'и можно не обращать внимание, т.к. у нас не установен ключ для проверки подлинности устанавливаемых пакетов (см. http://www.rhd.ru/docs/manuals/enterprise/RHEL-4-Manual/sysadmin-guide/s1-rpm-using.html)

Далее устанавливаем среду для сборки ядра Android.
```console
PlatformSDK:~$ TARBALL=ubuntu-trusty-20180613-android-rootfs.tar.bz2
PlatformSDK:~$ curl -O https://releases.sailfishos.org/ubu/$TARBALL
```
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100  440M  100  440M    0     0  16.7M      0  0:00:26  0:00:26 --:--:-- 17.2M
</details><br>

```console
PlatformSDK:~$ UBUNTU_CHROOT=$PLATFORM_SDK_ROOT/sdks/ubuntu
PlatformSDK:~$ sudo mkdir -p $UBUNTU_CHROOT
```

Проверяем, комплектацию среды
```console
PlatformSDK:~$ ls -lah /srv/mer/sdks/ubuntu/
```
<details>
total 96K
drwxr-xr-x 24 root root 4.0K Jun 13  2018 .<br>
drwxr-xr-x  3 root root 4.0K Jun 21 17:38 ..<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 bin<br>
drwxr-xr-x  2 root root 4.0K Apr 10  2014 boot<br>
drwxr-xr-x  3 root root 4.0K Jun 13  2018 dev<br>
drwxr-xr-x 82 root root 4.0K Jun 13  2018 etc<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 home<br>
drwxr-xr-x 14 root root 4.0K Jun 13  2018 lib<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 lib32<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 lib64<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 libx32<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 media<br>
drwxr-xr-x  2 root root 4.0K Apr 10  2014 mnt<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 opt<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 parentroot<br>
drwxr-xr-x  2 root root 4.0K Apr 10  2014 proc<br>
drwx------  2 root root 4.0K Jun 13  2018 root<br>
drwxr-xr-x  8 root root 4.0K Jun 13  2018 run<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 sbin<br>
drwxr-xr-x  2 root root 4.0K Jun 13  2018 srv<br>
drwxr-xr-x  2 root root 4.0K Mar 13  2014 sys<br>
drwxrwxrwt  5 root root 4.0K Jun 13  2018 tmp<br>
drwxr-xr-x 15 root root 4.0K Jun 13  2018 usr<br>
drwxr-xr-x 11 root root 4.0K Jun 13  2018 var
</details><br>

Заходим в среду
```console
PlatformSDK:~$ ubu-chroot -r $PLATFORM_SDK_ROOT/sdks/ubuntu
```
<details>
Env setup for mido<br>
HABUILD_SDK [mido]:~$
</details><br>

Позволю себе напомнить, что находясь в среде сборки ядра мы также имеем в распоряжении только те инструменты и параметры, которые установлены в этой среде

Если вы еще не зарегистрировались на github.com, то сделаейте это, т.к. в дальнейшем будем использовать git.
Установим имя и электронную почту для git
```console
HABUILD_SDK [mido]:~$ git config --global user.name "Your Name"
HABUILD_SDK [mido]:~$ git config --global user.email "you@example.com"
```

Далее установим программу для синхронизации локального хранилища с глобальным
```console
HABUILD_SDK [mido]:~$ mkdir ~/bin
HABUILD_SDK [mido]:~$ PATH=~/bin:$PATH
HABUILD_SDK [mido]:~$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
```
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100 29142  100 29142    0     0  47424      0 --:--:-- --:--:-- --:--:-- 47462
</details><br>

Сделаем ее исполняемой
```bash
HABUILD_SDK [mido]:~$ chmod a+x ~/bin/repo
```

Теперь нужно определиться с какой версией hybris будем синхронизироваться, т.к. на Mido есть две версии Android: (6 и 7).
Смотрим какая установлена у вас в Настройки-О телефоне, если 6 то нужно использовать hybris-13.0 (либо лучше обновиться до 7), если 7 то hybris-14.1.
Дальнейшее описание будет выполнено для версии 14.1

Создаем папку, где будет располагаться все, что нам нужно для сборки
```console
HABUILD_SDK [mido]:~$ sudo mkdir -p $ANDROID_ROOT
HABUILD_SDK [mido]:~$ sudo chown -R $USER $ANDROID_ROOT
HABUILD_SDK [mido]:~$ cd $ANDROID_ROOT
```
Синхронизируемся с  репозиторием
```console
HABUILD_SDK [mido]:~/hadk$ repo init -u git://github.com/mer-hybris/android.git -b hybris-14.1
```
<details>
Get https://gerrit.googlesource.com/git-repo/clone.bundle<br>
Get https://gerrit.googlesource.com/git-repo<br>
Get git://github.com/mer-hybris/android.git<br>
remote: Enumerating objects: 13, done.<br>
remote: Counting objects: 100% (13/13), done.<br>
remote: Compressing objects: 100% (9/9), done.<br>
remote: Total 5464 (delta 5), reused 9 (delta 4), pack-reused 5451<br>
Receiving objects: 100% (5464/5464), 1.82 MiB | 3.59 MiB/s, done.<br>
Resolving deltas: 100% (2009/2009), done.<br>
From git://github.com/mer-hybris/android<br>
...<br>
Syncing work tree: 100% (172/172), done.
</details><br>

Подготовим среду для сборки
```bash
HABUILD_SDK [mido]:~/hadk$ source build/envsetup.sh
```
<details>
including vendor/cm/vendorsetup.sh<br>
including vendor/cm/bash_completion/git.bash<br>
including vendor/cm/bash_completion/repo.bash
</details><br>

Запустим конфигурацию сборки под mido (более подробно можно прочитать http://www.trcompu.com/MySmartPhone/AndroidKitchen/Breakfast-Brunch-Lunch.html)
```console
HABUILD_SDK [mido]:~/hadk$ breakfast $DEVICE
```
<details>
including vendor/cm/vendorsetup.sh<br>
build/core/product_config.mk:254: *** _nic.PRODUCTS.[[device/xiaomi/mido/lineage.mk]]: "vendor/xiaomi/mido/mido-vendor.mk" does not exist.  Stop.<br>
build/core/product_config.mk:254: *** _nic.PRODUCTS.[[device/xiaomi/mido/lineage.mk]]: "vendor/xiaomi/mido/mido-vendor.mk" does not exist.  Stop.<br>
build/core/product_config.mk:254: *** _nic.PRODUCTS.[[device/xiaomi/mido/lineage.mk]]: "vendor/xiaomi/mido/mido-vendor.mk" does not exist.  Stop.<br>
Device mido not found. Attempting to retrieve device repository from LineageOS Github (http://github.com/LineageOS).<br>
Found repository: android_device_xiaomi_mido<br>
Default revision: cm-14.1<br>
Checking branch info<br>
Checking if device/xiaomi/mido is fetched from android_device_xiaomi_mido<br>
Adding dependency: LineageOS/android_device_xiaomi_mido -> device/xiaomi/mido<br>
Using default branch for android_device_xiaomi_mido<br>
Syncing repository to retrieve project.<br>
fatal: duplicate path device/xiaomi/mido in /home/stalker/hadk/.repo/manifest.xml<br>
Repository synced!<br>
Looking for dependencies in device/xiaomi/mido<br>
Adding dependencies to manifest<br>
Checking if kernel/xiaomi/msm8953 is fetched from android_kernel_xiaomi_msm8953<br>
Adding dependency: LineageOS/android_kernel_xiaomi_msm8953 -> kernel/xiaomi/msm8953<br>
Using default branch for android_kernel_xiaomi_msm8953<br>
Syncing dependencies<br>
fatal: duplicate path device/xiaomi/mido in /home/stalker/hadk/.repo/manifest.xml<br>
Looking for dependencies in device/qcom/common<br>
Dependencies file not found, bailing out.<br>
Looking for dependencies in kernel/xiaomi/msm8953<br>
Dependencies file not found, bailing out.<br>
Done<br>
build/core/product_config.mk:254: *** _nic.PRODUCTS.[[device/xiaomi/mido/lineage.mk]]: "vendor/xiaomi/mido/mido-vendor.mk" does not exist.  Stop.<br>
build/core/product_config.mk:254: *** _nic.PRODUCTS.[[device/xiaomi/mido/lineage.mk]]: "vendor/xiaomi/mido/mido-vendor.mk" does not exist.  Stop.<br>
<br>
** Don't have a product spec for: 'lineage_mido'<br>
** Do you have the right repo manifest?
</details><br>

Похоже не хватает блобов (blobs - binary large objects) решение описано здесь https://4pda.ru/forum/lofiversion/index.php?t209610-22980.html

Клонируем необходимые файлы
```console
HABUILD_SDK [mido]:~/hadk$ git clone -b cm-14.1 https://github.com/TheMuppets/proprietary_vendor_xiaomi.git vendor/xiaomi
```
<details>
Cloning into 'vendor/xiaomi'...<br>
remote: Enumerating objects: 28, done.<br>
remote: Counting objects: 100% (28/28), done.<br>
remote: Compressing objects: 100% (10/10), done.<br>
remote: Total 35780 (delta 20), reused 18 (delta 18), pack-reused 35752<br>
Receiving objects: 100% (35780/35780), 3.22 GiB | 4.50 MiB/s, done.<br>
Resolving deltas: 100% (19055/19055), done.<br>
Checking out files: 100% (8528/8528), done.<br>
</details><br>


Чтобы при синхронизации с нуля такой ошибки не было нужно обновить файл `$ANDROID_ROOT/.repo/local_manifests/$DEVICE.xml` и добавить туда строку<br>
`<project path="vendor/xiaomi" name="TheMuppets/proprietary_vendor_xiaomi" revision="cm-14.1" />`

Повторно приготовим завтрак)))
```console
HABUILD_SDK [mido]:~/hadk$ breakfast $DEVICE
```
<details>
including vendor/cm/vendorsetup.sh<br>
Looking for dependencies in device/xiaomi/mido<br>
Looking for dependencies in device/qcom/common<br>
Dependencies file not found, bailing out.<br>
<br>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190625-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
</details><br>

Пытаемся собрать ядро
```console
HABUILD_SDK [mido]:~/hadk$ make -j$(nproc --all) hybris-hal
```
<details>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190626-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/affinity.o build/kati/affinity.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/command.o build/kati/command.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/dep.o build/kati/dep.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/eval.o build/kati/eval.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/exec.o build/kati/exec.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/expr.o build/kati/expr.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file.o build/kati/file.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file_cache.o build/kati/file_cache.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/fileutil.o build/kati/fileutil.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/find.o build/kati/find.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/flags.o build/kati/flags.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/func.o build/kati/func.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/io.o build/kati/io.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/log.o build/kati/log.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/main.o build/kati/main.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/ninja.o build/kati/ninja.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/parser.o build/kati/parser.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/regen.o build/kati/regen.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/rule.o build/kati/rule.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stats.o build/kati/stats.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stmt.o build/kati/stmt.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/string_piece.o build/kati/string_piece.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stringprintf.o build/kati/stringprintf.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/strutil.o build/kati/strutil.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/symtab.o build/kati/symtab.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/thread_pool.o build/kati/thread_pool.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/timeutil.o build/kati/timeutil.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/var.o build/kati/var.cc<br>
echo '// +build ignore' > /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc<br>
echo >> /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc<br>
echo 'const char* kGitVersion = "bc43789a938c10cb00b81ddf08239c1b4cea48bb";' >> /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -Wall -Werror -MMD -MP -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/makeparallel_intermediates/makeparallel.o build/tools/makeparallel/makeparallel.cpp<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -m64 -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -Wl,--no-undefined-version    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/lib64/ -target x86_64-linux-gnu -static -std=c++11 -Wall -Werror -MMD -MP -o /home/stalker/hadk/out/host/linux-x86/bin/makeparallel /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/makeparallel_intermediates/makeparallel.o -lrt -lpthread<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -m64 -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -Wl,--no-undefined-version    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/lib64/ -target x86_64-linux-gnu -static -Wl,--whole-archive -lpthread -Wl,--no-whole-archive -ldl -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/bin/ckati /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/affinity.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/command.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/dep.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/eval.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/exec.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/expr.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file_cache.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/fileutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/find.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/flags.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/func.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/io.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/log.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/main.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/ninja.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/parser.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/regen.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/rule.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stats.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stmt.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/string_piece.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stringprintf.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/strutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/symtab.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/thread_pool.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/timeutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/var.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.o -lrt -lpthread<br>
sem_open.c:333: warning: the use of `mktemp' is dangerous, better use `mkstemp' or `mkdtemp'<br>
Running kati to generate build-lineage_mido.ninja...<br>
/home/stalker/hadk/out/build-lineage_mido.ninja is missing, regenerating...<br>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190626-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
Checking build tools versions...<br>
build/core/binary.mk:37: hal3-test-app uses kernel headers, but does not depend on them!<br>
external/speex/Android.mk:56: TODOArm64: enable neon in libspeex<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
hybris/hybris-boot/Android.mk:71: ********************* /boot appears to live on /dev/block/bootdevice/by-name/boot<br>
hybris/hybris-boot/Android.mk:72: ********************* /data appears to live on /dev/block/bootdevice/by-name/userdata<br>
**********************************************<br>
The boot animation could not be generated as<br>
ImageMagick is not installed in your system.<br>
<br>
Please install ImageMagick from this website:<br>
https://imagemagick.org/script/binary-releases.php<br>
**********************************************<br>
vendor/cm/bootanimation/Android.mk:40: *** stop.<br>
make: *** [/home/stalker/hadk/out/build-lineage_mido.ninja] Error 1<br>
<br>
#### make failed to build some targets (12 seconds) ####<br>
</details><br>

Пишет что не хватает ImageMagic, обновим репозитории
```console
HABUILD_SDK [mido]:~/hadk$ sudo apt-get update
```
<details>
Ign http://archive.ubuntu.com trusty InRelease<br>
Get:1 http://ppa.launchpad.net trusty InRelease [20.8 kB]<br>
Get:2 http://archive.ubuntu.com trusty-security InRelease [65.9 kB]<br>
Ign http://ppa.launchpad.net trusty InRelease<br>
Get:3 http://ppa.launchpad.net trusty InRelease [15.4 kB]<br>
Ign http://ppa.launchpad.net trusty/main amd64 Packages/DiffIndex<br>
Get:4 http://archive.ubuntu.com trusty-updates InRelease [65.9 kB]<br>
Ign http://ppa.launchpad.net trusty/main i386 Packages/DiffIndex<br>
Get:5 http://ppa.launchpad.net trusty/main Translation-en [2588 B]<br>
Hit http://archive.ubuntu.com trusty Release.gpg<br>
Get:6 http://archive.ubuntu.com trusty-security/main amd64 Packages [835 kB]<br>
Get:7 http://ppa.launchpad.net trusty/main amd64 Packages [7349 B]<br>
Get:8 http://ppa.launchpad.net trusty/main i386 Packages [7447 B]<br>
Get:9 http://ppa.launchpad.net trusty/main Translation-en [2854 B]<br>
Get:10 http://ppa.launchpad.net trusty/main amd64 Packages [3418 B]<br>
Get:11 http://archive.ubuntu.com trusty-security/universe amd64 Packages [294 kB]<br>
Get:12 http://ppa.launchpad.net trusty/main i386 Packages [3415 B]<br>
Get:13 http://archive.ubuntu.com trusty-security/multiverse amd64 Packages [4806 B]<br>
Get:14 http://archive.ubuntu.com trusty-security/restricted amd64 Packages [14.2 kB]<br>
Get:15 http://archive.ubuntu.com trusty-security/main i386 Packages [748 kB]<br>
Get:16 http://archive.ubuntu.com trusty-security/universe i386 Packages [276 kB]<br>
Get:17 http://archive.ubuntu.com trusty-security/multiverse i386 Packages [4969 B]<br>
Get:18 http://archive.ubuntu.com trusty-security/restricted i386 Packages [13.9 kB]<br>
Get:19 http://archive.ubuntu.com trusty-security/main Translation-en [448 kB]<br>
Get:20 http://archive.ubuntu.com trusty-security/multiverse Translation-en [2564 B]<br>
Get:21 http://archive.ubuntu.com trusty-security/restricted Translation-en [3556 B]<br>
Get:22 http://archive.ubuntu.com trusty-security/universe Translation-en [162 kB]<br>
Get:23 http://archive.ubuntu.com trusty-updates/main amd64 Packages [1177 kB]<br>
Get:24 http://archive.ubuntu.com trusty-updates/universe amd64 Packages [526 kB]<br>
Get:25 http://archive.ubuntu.com trusty-updates/multiverse amd64 Packages [14.6 kB]<br>
Get:26 http://archive.ubuntu.com trusty-updates/restricted amd64 Packages [17.2 kB]<br>
Get:27 http://archive.ubuntu.com trusty-updates/main i386 Packages [1089 kB]<br>
Get:28 http://archive.ubuntu.com trusty-updates/universe i386 Packages [504 kB]<br>
Get:29 http://archive.ubuntu.com trusty-updates/multiverse i386 Packages [15.1 kB]<br>
Get:30 http://archive.ubuntu.com trusty-updates/restricted i386 Packages [17.0 kB]<br>
Get:31 http://archive.ubuntu.com trusty-updates/main Translation-en [582 kB]<br>
Get:32 http://archive.ubuntu.com trusty-updates/multiverse Translation-en [7616 B]<br>
Get:33 http://archive.ubuntu.com trusty-updates/restricted Translation-en [4028 B]<br>
Get:34 http://archive.ubuntu.com trusty-updates/universe Translation-en [281 kB]<br>
Hit http://archive.ubuntu.com trusty Release<br>
Hit http://archive.ubuntu.com trusty/main amd64 Packages<br>
Hit http://archive.ubuntu.com trusty/universe amd64 Packages<br>
Hit http://archive.ubuntu.com trusty/multiverse amd64 Packages<br>
Hit http://archive.ubuntu.com trusty/restricted amd64 Packages<br>
Hit http://archive.ubuntu.com trusty/main i386 Packages<br>
Hit http://archive.ubuntu.com trusty/universe i386 Packages<br>
Hit http://archive.ubuntu.com trusty/multiverse i386 Packages<br>
Hit http://archive.ubuntu.com trusty/restricted i386 Packages<br>
Hit http://archive.ubuntu.com trusty/main Translation-en<br>
Hit http://archive.ubuntu.com trusty/multiverse Translation-en<br>
Hit http://archive.ubuntu.com trusty/restricted Translation-en<br>
Hit http://archive.ubuntu.com trusty/universe Translation-en<br>
Fetched 7237 kB in 8s (848 kB/s)<br>
Reading package lists... Done<br>
W: GPG error: http://ppa.launchpad.net trusty InRelease: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY A1715D88E1DF1F24<br>
</details><br>

и установим ImageMagick
```console
HABUILD_SDK [mido]:~/hadk$ sudo apt-get install imagemagick
```
<details>
Reading package lists... Done<br>
Building dependency tree<br>
Reading state information... Done<br>
The following extra packages will be installed:<br>
 ghostscript gsfonts imagemagick-common libcroco3 libcups2 libcupsfilters1<br>
 libcupsimage2 libdjvulibre-text libdjvulibre21 libfftw3-double3 libgs9<br>
 libgs9-common libijs-0.35 libilmbase6 libjbig2dec0 liblqr-1-0 libmagickcore5<br>
 libmagickcore5-extra libmagickwand5 libnetpbm10 libopenexr6 libpaper-utils<br>
 libpaper1 librsvg2-2 librsvg2-common libwmf0.2-7 netpbm poppler-data<br>
Suggested packages:<br>
 ghostscript-x hpijs imagemagick-doc autotrace cups-bsd lpr lprng enscript<br>
 ffmpeg gimp gnuplot grads groff-base hp2xx html2ps libwmf-bin mplayer povray<br>
 radiance sane-utils texlive-base-bin transfig xdg-utils ufraw-batch<br>
 cups-common libfftw3-bin libfftw3-dev fonts-droid librsvg2-bin<br>
 libwmf0.2-7-gtk poppler-utils fonts-japanese-mincho fonts-ipafont-mincho<br>
 fonts-japanese-gothic fonts-ipafont-gothic fonts-arphic-ukai<br>
 fonts-arphic-uming fonts-unfonts-core<br>
The following NEW packages will be installed:<br>
 ghostscript gsfonts imagemagick imagemagick-common libcroco3 libcupsfilters1<br>
 libcupsimage2 libdjvulibre-text libdjvulibre21 libfftw3-double3 libgs9<br>
 libgs9-common libijs-0.35 libilmbase6 libjbig2dec0 liblqr-1-0 libmagickcore5<br>
 libmagickcore5-extra libmagickwand5 libnetpbm10 libopenexr6 libpaper-utils<br>
 libpaper1 librsvg2-2 librsvg2-common libwmf0.2-7 netpbm poppler-data<br>
The following packages will be upgraded:<br>
 libcups2<br>
1 upgraded, 28 newly installed, 0 to remove and 109 not upgraded.<br>
Need to get 18.0 MB of archives.<br>
After this operation, 62.4 MB of additional disk space will be used.<br>
Do you want to continue? [Y/n] y<br>
Get:1 http://archive.ubuntu.com/ubuntu/ trusty-security/main imagemagick-common all 8:6.7.7.10-6ubuntu3.13 [38.4 kB]<br>
Get:2 http://archive.ubuntu.com/ubuntu/ trusty/main libcroco3 amd64 0.6.8-2ubuntu1 [82.4 kB]<br>
Get:3 http://archive.ubuntu.com/ubuntu/ trusty-security/main libcups2 amd64 1.7.2-0ubuntu1.11 [178 kB]<br>
Get:4 http://archive.ubuntu.com/ubuntu/ trusty-security/main libcupsfilters1 amd64 1.0.52-0ubuntu1.8 [73.7 kB]<br>
Get:5 http://archive.ubuntu.com/ubuntu/ trusty-security/main libcupsimage2 amd64 1.7.2-0ubuntu1.11 [15.4 kB]<br>
Get:6 http://archive.ubuntu.com/ubuntu/ trusty/main libdjvulibre-text all 3.5.25.4-3 [48.8 kB]<br>
Get:7 http://archive.ubuntu.com/ubuntu/ trusty/main libdjvulibre21 amd64 3.5.25.4-3 [553 kB]<br>
Get:8 http://archive.ubuntu.com/ubuntu/ trusty/main libfftw3-double3 amd64 3.3.3-7ubuntu3 [702 kB]<br>
Get:9 http://archive.ubuntu.com/ubuntu/ trusty/main libilmbase6 amd64 1.0.1-6ubuntu1 [53.6 kB]<br>
Get:10 http://archive.ubuntu.com/ubuntu/ trusty/main liblqr-1-0 amd64 0.4.1-2ubuntu1 [23.4 kB]<br>
Get:11 http://archive.ubuntu.com/ubuntu/ trusty-security/main libmagickcore5 amd64 8:6.7.7.10-6ubuntu3.13 [1480 kB]<br>
Get:12 http://archive.ubuntu.com/ubuntu/ trusty-security/main libmagickwand5 amd64 8:6.7.7.10-6ubuntu3.13 [267 kB]<br>
Get:13 http://archive.ubuntu.com/ubuntu/ trusty/main libopenexr6 amd64 1.6.1-7ubuntu1 [164 kB]<br>
Get:14 http://archive.ubuntu.com/ubuntu/ trusty/main librsvg2-2 amd64 2.40.2-1 [88.3 kB]<br>
Get:15 http://archive.ubuntu.com/ubuntu/ trusty-security/main libwmf0.2-7 amd64 0.2.8.4-10.3ubuntu1.14.04.1 [143 kB]<br>
Get:16 http://archive.ubuntu.com/ubuntu/ trusty-security/main libmagickcore5-extra amd64 8:6.7.7.10-6ubuntu3.13 [58.4 kB]<br>
Get:17 http://archive.ubuntu.com/ubuntu/ trusty/main libpaper1 amd64 1.1.24+nmu2ubuntu3 [13.4 kB]<br>
Get:18 http://archive.ubuntu.com/ubuntu/ trusty/main poppler-data all 0.4.6-4 [1479 kB]<br>
Get:19 http://archive.ubuntu.com/ubuntu/ trusty/main libijs-0.35 amd64 0.35-8build1 [16.8 kB]<br>
Get:20 http://archive.ubuntu.com/ubuntu/ trusty-security/main libjbig2dec0 amd64 0.11+20120125-1ubuntu1.1 [43.0 kB]<br>
Get:21 http://archive.ubuntu.com/ubuntu/ trusty-security/main libgs9-common all 9.26~dfsg+0-0ubuntu0.14.04.8 [5098 kB]<br>
Get:22 http://archive.ubuntu.com/ubuntu/ trusty-security/main libgs9 amd64 9.26~dfsg+0-0ubuntu0.14.04.8 [2355 kB]<br>
Get:23 http://archive.ubuntu.com/ubuntu/ trusty/main gsfonts all 1:8.11+urwcyr1.0.7~pre44-4.2ubuntu1 [3374 kB]<br>
Get:24 http://archive.ubuntu.com/ubuntu/ trusty-security/main ghostscript amd64 9.26~dfsg+0-0ubuntu0.14.04.8 [47.1 kB]<br>
Get:25 http://archive.ubuntu.com/ubuntu/ trusty-security/main imagemagick amd64 8:6.7.7.10-6ubuntu3.13 [189 kB]<br>
Get:26 http://archive.ubuntu.com/ubuntu/ trusty/main libnetpbm10 amd64 2:10.0-15ubuntu2 [69.0 kB]<br>
Get:27 http://archive.ubuntu.com/ubuntu/ trusty/main libpaper-utils amd64 1.1.24+nmu2ubuntu3 [8244 B]<br>
Get:28 http://archive.ubuntu.com/ubuntu/ trusty/main librsvg2-common amd64 2.40.2-1 [4990 B]<br>
Get:29 http://archive.ubuntu.com/ubuntu/ trusty/main netpbm amd64 2:10.0-15ubuntu2 [1341 kB]<br>
Fetched 18.0 MB in 6s (2712 kB/s)<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
 LANGUAGE = (unset),<br>
 LC_ALL = (unset),<br>
 LC_CTYPE = "en_US.UTF-8",<br>
 LC_MESSAGES = "en_US.UTF-8",<br>
 LANG = (unset)<br>
   are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
locale: Cannot set LC_CTYPE to default locale: No such file or directory<br>
locale: Cannot set LC_MESSAGES to default locale: No such file or directory<br>
locale: Cannot set LC_ALL to default locale: No such file or directory<br>
Preconfiguring packages ...<br>
Selecting previously unselected package imagemagick-common.<br>
(Reading database ... 30574 files and directories currently installed.)<br>
Preparing to unpack .../imagemagick-common_8%3a6.7.7.10-6ubuntu3.13_all.deb ...<br>
Unpacking imagemagick-common (8:6.7.7.10-6ubuntu3.13) ...<br>
Selecting previously unselected package libcroco3:amd64.<br>
Preparing to unpack .../libcroco3_0.6.8-2ubuntu1_amd64.deb ...<br>
Unpacking libcroco3:amd64 (0.6.8-2ubuntu1) ...<br>
Preparing to unpack .../libcups2_1.7.2-0ubuntu1.11_amd64.deb ...<br>
Unpacking libcups2:amd64 (1.7.2-0ubuntu1.11) over (1.7.2-0ubuntu1.9) ...<br>
Selecting previously unselected package libcupsfilters1:amd64.<br>
Preparing to unpack .../libcupsfilters1_1.0.52-0ubuntu1.8_amd64.deb ...<br>
Unpacking libcupsfilters1:amd64 (1.0.52-0ubuntu1.8) ...<br>
Selecting previously unselected package libcupsimage2:amd64.<br>
Preparing to unpack .../libcupsimage2_1.7.2-0ubuntu1.11_amd64.deb ...<br>
Unpacking libcupsimage2:amd64 (1.7.2-0ubuntu1.11) ...<br>
Selecting previously unselected package libdjvulibre-text.<br>
Preparing to unpack .../libdjvulibre-text_3.5.25.4-3_all.deb ...<br>
Unpacking libdjvulibre-text (3.5.25.4-3) ...<br>
Selecting previously unselected package libdjvulibre21:amd64.<br>
Preparing to unpack .../libdjvulibre21_3.5.25.4-3_amd64.deb ...<br>
Unpacking libdjvulibre21:amd64 (3.5.25.4-3) ...<br>
Selecting previously unselected package libfftw3-double3:amd64.<br>
Preparing to unpack .../libfftw3-double3_3.3.3-7ubuntu3_amd64.deb ...<br>
Unpacking libfftw3-double3:amd64 (3.3.3-7ubuntu3) ...<br>
Selecting previously unselected package libilmbase6:amd64.<br>
Preparing to unpack .../libilmbase6_1.0.1-6ubuntu1_amd64.deb ...<br>
Unpacking libilmbase6:amd64 (1.0.1-6ubuntu1) ...<br>
Selecting previously unselected package liblqr-1-0:amd64.<br>
Preparing to unpack .../liblqr-1-0_0.4.1-2ubuntu1_amd64.deb ...<br>
Unpacking liblqr-1-0:amd64 (0.4.1-2ubuntu1) ...<br>
Selecting previously unselected package libmagickcore5:amd64.<br>
Preparing to unpack .../libmagickcore5_8%3a6.7.7.10-6ubuntu3.13_amd64.deb ...<br>
Unpacking libmagickcore5:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Selecting previously unselected package libmagickwand5:amd64.<br>
Preparing to unpack .../libmagickwand5_8%3a6.7.7.10-6ubuntu3.13_amd64.deb ...<br>
Unpacking libmagickwand5:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Selecting previously unselected package libopenexr6:amd64.<br>
Preparing to unpack .../libopenexr6_1.6.1-7ubuntu1_amd64.deb ...<br>
Unpacking libopenexr6:amd64 (1.6.1-7ubuntu1) ...<br>
Selecting previously unselected package librsvg2-2:amd64.<br>
Preparing to unpack .../librsvg2-2_2.40.2-1_amd64.deb ...<br>
Unpacking librsvg2-2:amd64 (2.40.2-1) ...<br>
Selecting previously unselected package libwmf0.2-7:amd64.<br>
Preparing to unpack .../libwmf0.2-7_0.2.8.4-10.3ubuntu1.14.04.1_amd64.deb ...<br>
Unpacking libwmf0.2-7:amd64 (0.2.8.4-10.3ubuntu1.14.04.1) ...<br>
Selecting previously unselected package libmagickcore5-extra:amd64.<br>
Preparing to unpack .../libmagickcore5-extra_8%3a6.7.7.10-6ubuntu3.13_amd64.deb ...<br>
Unpacking libmagickcore5-extra:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Selecting previously unselected package libpaper1:amd64.<br>
Preparing to unpack .../libpaper1_1.1.24+nmu2ubuntu3_amd64.deb ...<br>
Unpacking libpaper1:amd64 (1.1.24+nmu2ubuntu3) ...<br>
Selecting previously unselected package poppler-data.<br>
Preparing to unpack .../poppler-data_0.4.6-4_all.deb ...<br>
Unpacking poppler-data (0.4.6-4) ...<br>
Selecting previously unselected package libijs-0.35.<br>
Preparing to unpack .../libijs-0.35_0.35-8build1_amd64.deb ...<br>
Unpacking libijs-0.35 (0.35-8build1) ...<br>
Selecting previously unselected package libjbig2dec0.<br>
Preparing to unpack .../libjbig2dec0_0.11+20120125-1ubuntu1.1_amd64.deb ...<br>
Unpacking libjbig2dec0 (0.11+20120125-1ubuntu1.1) ...<br>
Selecting previously unselected package libgs9-common.<br>
Preparing to unpack .../libgs9-common_9.26~dfsg+0-0ubuntu0.14.04.8_all.deb ...<br>
Unpacking libgs9-common (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
Selecting previously unselected package libgs9.<br>
Preparing to unpack .../libgs9_9.26~dfsg+0-0ubuntu0.14.04.8_amd64.deb ...<br>
Unpacking libgs9 (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
Selecting previously unselected package gsfonts.<br>
Preparing to unpack .../gsfonts_1%3a8.11+urwcyr1.0.7~pre44-4.2ubuntu1_all.deb ...<br>
Unpacking gsfonts (1:8.11+urwcyr1.0.7~pre44-4.2ubuntu1) ...<br>
Selecting previously unselected package ghostscript.<br>
Preparing to unpack .../ghostscript_9.26~dfsg+0-0ubuntu0.14.04.8_amd64.deb ...<br>
Unpacking ghostscript (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
Selecting previously unselected package imagemagick.<br>
Preparing to unpack .../imagemagick_8%3a6.7.7.10-6ubuntu3.13_amd64.deb ...<br>
Unpacking imagemagick (8:6.7.7.10-6ubuntu3.13) ...<br>
Selecting previously unselected package libnetpbm10.<br>
Preparing to unpack .../libnetpbm10_2%3a10.0-15ubuntu2_amd64.deb ...<br>
Unpacking libnetpbm10 (2:10.0-15ubuntu2) ...<br>
Selecting previously unselected package libpaper-utils.<br>
Preparing to unpack .../libpaper-utils_1.1.24+nmu2ubuntu3_amd64.deb ...<br>
Unpacking libpaper-utils (1.1.24+nmu2ubuntu3) ...<br>
Selecting previously unselected package librsvg2-common:amd64.<br>
Preparing to unpack .../librsvg2-common_2.40.2-1_amd64.deb ...<br>
Unpacking librsvg2-common:amd64 (2.40.2-1) ...<br>
Selecting previously unselected package netpbm.<br>
Preparing to unpack .../netpbm_2%3a10.0-15ubuntu2_amd64.deb ...<br>
Unpacking netpbm (2:10.0-15ubuntu2) ...<br>
Processing triggers for fontconfig (2.11.0-0ubuntu4.2) ...<br>
Processing triggers for mime-support (3.54ubuntu1.1) ...<br>
Processing triggers for hicolor-icon-theme (0.13-1) ...<br>
Processing triggers for libgdk-pixbuf2.0-0:amd64 (2.30.7-0ubuntu1.8) ...<br>
Setting up imagemagick-common (8:6.7.7.10-6ubuntu3.13) ...<br>
Setting up libcroco3:amd64 (0.6.8-2ubuntu1) ...<br>
Setting up libcups2:amd64 (1.7.2-0ubuntu1.11) ...<br>
Setting up libcupsfilters1:amd64 (1.0.52-0ubuntu1.8) ...<br>
Setting up libcupsimage2:amd64 (1.7.2-0ubuntu1.11) ...<br>
Setting up libdjvulibre-text (3.5.25.4-3) ...<br>
Setting up libdjvulibre21:amd64 (3.5.25.4-3) ...<br>
Setting up libfftw3-double3:amd64 (3.3.3-7ubuntu3) ...<br>
Setting up libilmbase6:amd64 (1.0.1-6ubuntu1) ...<br>
Setting up liblqr-1-0:amd64 (0.4.1-2ubuntu1) ...<br>
Setting up libmagickcore5:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Setting up libmagickwand5:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Setting up libopenexr6:amd64 (1.6.1-7ubuntu1) ...<br>
Setting up librsvg2-2:amd64 (2.40.2-1) ...<br>
Setting up libwmf0.2-7:amd64 (0.2.8.4-10.3ubuntu1.14.04.1) ...<br>
Setting up libmagickcore5-extra:amd64 (8:6.7.7.10-6ubuntu3.13) ...<br>
Setting up libpaper1:amd64 (1.1.24+nmu2ubuntu3) ...<br>
locale: Cannot set LC_CTYPE to default locale: No such file or directory<br>
locale: Cannot set LC_MESSAGES to default locale: No such file or directory<br>
locale: Cannot set LC_ALL to default locale: No such file or directory<br>
<br>
Creating config file /etc/papersize with new version<br>
Setting up poppler-data (0.4.6-4) ...<br>
Setting up libijs-0.35 (0.35-8build1) ...<br>
Setting up libjbig2dec0 (0.11+20120125-1ubuntu1.1) ...<br>
Setting up libgs9-common (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
update-alternatives: using /usr/share/ghostscript/9.26 to provide /usr/share/ghostscript/current (ghostscript-current) in auto mode<br>
Setting up libgs9 (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
Setting up gsfonts (1:8.11+urwcyr1.0.7~pre44-4.2ubuntu1) ...<br>
Setting up ghostscript (9.26~dfsg+0-0ubuntu0.14.04.8) ...<br>
Setting up imagemagick (8:6.7.7.10-6ubuntu3.13) ...<br>
update-alternatives: using /usr/bin/compare.im6 to provide /usr/bin/compare (compare) in auto mode<br>
update-alternatives: using /usr/bin/animate.im6 to provide /usr/bin/animate (animate) in auto mode<br>
update-alternatives: using /usr/bin/convert.im6 to provide /usr/bin/convert (convert) in auto mode<br>
update-alternatives: using /usr/bin/composite.im6 to provide /usr/bin/composite (composite) in auto mode<br>
update-alternatives: using /usr/bin/conjure.im6 to provide /usr/bin/conjure (conjure) in auto mode<br>
update-alternatives: using /usr/bin/import.im6 to provide /usr/bin/import (import) in auto mode<br>
update-alternatives: using /usr/bin/identify.im6 to provide /usr/bin/identify (identify) in auto mode<br>
update-alternatives: using /usr/bin/stream.im6 to provide /usr/bin/stream (stream) in auto mode<br>
update-alternatives: using /usr/bin/display.im6 to provide /usr/bin/display (display) in auto mode<br>
update-alternatives: using /usr/bin/montage.im6 to provide /usr/bin/montage (montage) in auto mode<br>
update-alternatives: using /usr/bin/mogrify.im6 to provide /usr/bin/mogrify (mogrify) in auto mode<br>
Setting up libnetpbm10 (2:10.0-15ubuntu2) ...<br>
Setting up libpaper-utils (1.1.24+nmu2ubuntu3) ...<br>
Setting up librsvg2-common:amd64 (2.40.2-1) ...<br>
Setting up netpbm (2:10.0-15ubuntu2) ...<br>
Processing triggers for libc-bin (2.19-0ubuntu6.14) ...<br>
Processing triggers for libgdk-pixbuf2.0-0:amd64 (2.30.7-0ubuntu1.8) ...<br>
</details><br>

Соберем ядро

```console
HABUILD_SDK [mido]:~/hadk$ make -j$(nproc --all) hybris-hal
```
<details><br>
...<br>
#### make completed successfully (02:30 (mm:ss)) ####<br>
</details><br>

Проверим, что наше ядро сконфигурировано корректно для работы с Sailfish
```console
HABUILD_SDK [mido]:~/hadk$ hybris/mer-kernel-check/mer_verify_kernel_config ./out/target/product/mido/obj/KERNEL_OBJ/.config
```
<details>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
WARNING: CONFIG_LOCKD is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS support<br>
<br>
ERROR: CONFIG_DEVTMPFS_MOUNT is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: Required by hybris-boot init-script<br>
<br>
WARNING: CONFIG_NET_CLS_CGROUP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_PID_NS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, enables kernel namespaces for systemd-nspawn containers<br>
<br>
WARNING: CONFIG_NFS_FS is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_UTS_NS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, enables kernel namespaces for systemd-nspawn containers<br>
<br>
ERROR: CONFIG_FHANDLE is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: systemd: http://cgit.freedesktop.org/systemd/systemd/commit/README?id=001809282918f273d372f1ee09d10b783c18a474<br>
<br>
WARNING: CONFIG_WATCHDOG_NOWAYOUT is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: If device uses watchdogs with dsme (https://github.com/nemomobile/dsme), this option should be enabled or watchdog does not protect the device in case dsme crashes.<br>
<br>
WARNING: CONFIG_NETPRIO_CGROUP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_CGROUP_PERF is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_LOCKD_V4 is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_SECURITY_SELINUX_BOOTPARAM is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: Required by hybris, SELinux needs to be disabled. Leave as not set, if you have unset AUDIT (read more about the CONFIG_AUDIT flag)<br>
<br>
WARNING: CONFIG_NFS_V3 is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR_KMEM is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_SECURITY_YAMA is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, prevents user's processes from ptracing each other<br>
<br>
WARNING: CONFIG_LBDAF is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: ext4 filesystem requires this in order to support filesysetms with huge_file feature, which is enabled by default by mke2fs.ext4<br>
<br>
WARNING: CONFIG_NETFILTER_XT_MATCH_NFACCT is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: connman (optional): for routing and statistic support in sessions, http://git.kernel.org/cgit/network/connman/connman.git/commit/README?id=41f37125887cb9208da2441e350e1e3324c17ee6<br>
<br>
WARNING: CONFIG_MEMCG_KMEM is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional, but recommended): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version < 3.10<br>
<br>
WARNING: CONFIG_NFS_COMMON is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_BT_HCIUART_H4 is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: Bluez (optional): Needed if bluez used as bluetooth stack<br>
<br>
WARNING: CONFIG_SECURITY_YAMA_STACKED is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, ignore for kernel version >= 4.3<br>
<br>
WARNING: CONFIG_FW_LOADER_USER_HELPER is invalid<br>
Value is: y<br>
Allowed values : n, !<br>
Comment says: it's actually needed by some Lollipop based devices; systemd(optional): http://cgit.freedesktop.org/systemd/systemd/commit/README?id=713bc0cfa477ca1df8769041cb3dbc83c10eace2<br>
<br>
WARNING: CONFIG_NFS_V4_1 is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_NFS_V4 is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_SUNRPC is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_CHECKPOINT_RESTORE is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: rich-core-dumper (https://github.com/mer-tools/sp-rich-core/) needs this to collect all data for environment recreation.<br>
<br>
WARNING: CONFIG_SCHED_DEBUG is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd-bootchart (optional): http://cgit.freedesktop.org/systemd/systemd/commit/README?id=f1c24fea94e19cf2108abbeed1d36ded7102ab98<br>
<br>
WARNING: CONFIG_UDF_FS is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional extra filesystem (DVD & portable USB)<br>
<br>
ERROR: CONFIG_IKCONFIG_PROC is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: Required by hybris-boot init-script<br>
<br>
WARNING: CONFIG_BLK_DEV_NBD is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS & CIFS support<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR_SWAP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_AUDIT is invalid<br>
Value is: y<br>
Allowed values : n, !<br>
Comment says: This will disable SELinux! That's ok, because hybris adaptations must not have SELinux, but if your device needs its support in kernel, set AUDIT=y and SELINUX_BOOTPARAM=y. Then disable them via kernel cmdline: audit=0 selinux=0. You can also leave audit enabled, if you don't plan to use systemd's containers: http://cgit.freedesktop.org/systemd/systemd/commit/README?id=77b6e19458f37cfde127ec6aa9494c0ac45ad890<br>
<br>
WARNING: CONFIG_SUNRPC_GSS is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_FANOTIFY is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, required for systemd readahead.<br>
<br>
WARNING: CONFIG_NFS_ACL_SUPPORT is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional, for NFS support<br>
<br>
ERROR: CONFIG_DUMMY is invalid<br>
Value is: y<br>
Allowed values : n<br>
Use of uninitialized value in concatenation (.) or string at hybris/mer-kernel-check/mer_verify_kernel_config line 111, <> line 4545.<br>
Comment says:<br>
<br>
WARNING: CONFIG_NETFILTER_NETLINK_ACCT is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: connman (optional): for routing and statistic support in sessions, http://git.kernel.org/cgit/network/connman/connman.git/commit/README?id=41f37125887cb9208da2441e350e1e3324c17ee6<br>
<br>
WARNING: CONFIG_ISO9660_FS is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: optional extra filesystem (CD-ROM)<br>
<br>
WARNING: CONFIG_BTRFS_FS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional extra filesystem (BTRFS)<br>
<br>
ERROR: CONFIG_VT is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: Required for virtual consoles<br>
<br>
WARNING: CONFIG_BLK_CGROUP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_IPC_NS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, enables kernel namespaces for systemd-nspawn containers<br>
<br>
ERROR: CONFIG_DEVTMPFS is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: systemd: http://cgit.freedesktop.org/systemd/systemd/commit/README?id=713bc0cfa477ca1df8769041cb3dbc83c10eace2<br>
<br>
ERROR: CONFIG_SYSVIPC is invalid<br>
It is unset<br>
Allowed values : y<br>
Comment says: Inter Process Communication option is required to run Mer<br>
<br>
WARNING: CONFIG_BT_HCIUART is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: Bluez (optional): Needed if bluez used as bluetooth stack<br>
<br>
WARNING: CONFIG_RTC_DRV_CMOS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, but highly recommended<br>
<br>
WARNING: CONFIG_NFS_USE_KERNEL_DNS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_CGROUP_DEVICE is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_NFS_V3_ACL is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, for NFS support<br>
<br>
WARNING: CONFIG_AUTOFS4_FS is invalid<br>
It is unset<br>
Allowed values : y, m, !<br>
Comment says: systemd (optional): http://cgit.freedesktop.org/systemd/systemd/commit/README?id=713bc0cfa477ca1df8769041cb3dbc83c10eace2<br>
</details><br>


Как видим, присутствует несколько ошибок и много предупреждений, желательно все это исправить. Как это сделать описано в блоге neochapay (https://neochapay.ru/blogs/zapiski-utkonosa-programmista/konfiguracija-jadra-dlja-sailfish-os.html). Воспользуемся этой инструкцией

Сперва открываем файл `~/hadk/device/xiaomi/mido/BoardConfig.mk`
И видим что
```console
...
TARGET_ARCH := arm64
...
TARGET_KERNEL_CONFIG := mido_defconfig
...
```

Соответственно создаем конфиг ядра
```bash
HABUILD_SDK [mido]:~/hadk/kernel/xiaomi/msm8953$ ARCH=arm64 make mido_defconfig
```
<details>
HOSTCC  scripts/basic/fixdep<br>
HOSTCC  scripts/kconfig/conf.o<br>
SHIPPED scripts/kconfig/zconf.tab.c<br>
SHIPPED scripts/kconfig/zconf.lex.c<br>
SHIPPED scripts/kconfig/zconf.hash.c<br>
HOSTCC  scripts/kconfig/zconf.tab.o<br>
HOSTLD  scripts/kconfig/conf<br>
#<br>
# configuration written to .config<br>
#<br>
</details><br>

```bash
HABUILD_SDK [mido]:~/hadk/kernel/xiaomi/msm8953$ ARCH=arm64 make menuconfig
```

<details>
HOSTCC  scripts/basic/fixdep<br>
HOSTCC  scripts/kconfig/mconf.o<br>
SHIPPED scripts/kconfig/zconf.tab.c<br>
SHIPPED scripts/kconfig/zconf.lex.c<br>
SHIPPED scripts/kconfig/zconf.hash.c<br>
HOSTCC  scripts/kconfig/zconf.tab.o<br>
HOSTCC  scripts/kconfig/lxdialog/checklist.o<br>
HOSTCC  scripts/kconfig/lxdialog/util.o<br>
HOSTCC  scripts/kconfig/lxdialog/inputbox.o<br>
HOSTCC  scripts/kconfig/lxdialog/textbox.o<br>
HOSTCC  scripts/kconfig/lxdialog/yesno.o<br>
HOSTCC  scripts/kconfig/lxdialog/menubox.o<br>
HOSTLD  scripts/kconfig/mconf<br>
scripts/kconfig/mconf Kconfig<br>
</details><br>

Производить поиск параметра можно с помощью символа "/", в поле ввода вставляем имя конфига, но без префикса "CONFIG_". Для того чтобы перейти на требуемый пункт меню из режима поиска можно воспользоваться соответствующими цифровыми клавишами (1) (2) и т.д.  и ставим "Y". Помните, что удовлетворить нужно все зависимости, например
```console
│ Symbol: WATCHDOG_NOWAYOUT [=n]
│ Type  : boolean
│ Prompt: Disable watchdog shutdown on close
│   Location:
│     -> Device Drivers
│ (1)   -> Watchdog Timer Support (WATCHDOG [=n])
│   Defined at drivers/watchdog/Kconfig:39
│   Depends on: WATCHDOG [=n]
```
Т.е. нужно также включить WATCHDOG, если нажать 1, то перейдем собственно к параметру WATCHDOG, где его можно активировать, ну а затем нужно будет войти в подменю и соответственно включить "Disable watchdog shutdown on close"

Или другой пример
```console
│ Symbol: NFS_ACL_SUPPORT [=n]
│ Type  : tristate
│   Defined at fs/Kconfig:254
│   Depends on: NETWORK_FILESYSTEMS [=y]
│   Selects: FS_POSIX_ACL [=y]
│   Selected by: NFS_FS [=y] && NETWORK_FILESYSTEMS [=y] && INET [=y] && FILE_LOCKING [=y] && NFS_V3_ACL [=n] || NFSD [=n] && NETWORK_FILESYSTEMS [=y] && INET [=y] && FILE_LOCKING [=y] && NFSD_V2_ACL [=n]
```
NFS_ACL_SUPPORT активируестя если выполнены условия из Selected by. Соответственно активируем NFS_V3_ACL (т.к он тоже фигурирует в нашем списке)


И так в том же духе...

Повторно запускаем проверку конфигурации ядра.
```console
HABUILD_SDK [mido]:~/hadk/kernel/xiaomi/msm8953$ $ANDROID_ROOT/hybris/mer-kernel-check/mer_verify_kernel_config .config
```
<details>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
WARNING: CONFIG_NETPRIO_CGROUP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR_KMEM is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR_SWAP is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_FW_LOADER_USER_HELPER is invalid<br>
Value is: y<br>
Allowed values : n, !<br>
Comment says: it's actually needed by some Lollipop based devices; systemd(optional): http://cgit.freedesktop.org/systemd/systemd/commit/README?id=713bc0cfa477ca1df8769041cb3dbc83c10eace2<br>
<br>
WARNING: CONFIG_CGROUP_MEM_RES_CTLR is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: systemd (optional): http://0pointer.de/blog/projects/cgroups-vs-cgroups.html, ignore if kernel version >= 3.10<br>
<br>
WARNING: CONFIG_LBDAF is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: ext4 filesystem requires this in order to support filesysetms with huge_file feature, which is enabled by default by mke2fs.ext4<br>
<br>
WARNING: CONFIG_SECURITY_SELINUX_BOOTPARAM is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: Required by hybris, SELinux needs to be disabled. Leave as not set, if you have unset AUDIT (read more about the CONFIG_AUDIT flag)<br>
<br>
WARNING: CONFIG_AUDIT is invalid<br>
Value is: y<br>
Allowed values : n, !<br>
Comment says: This will disable SELinux! That's ok, because hybris adaptations must not have SELinux, but if your device needs its support in kernel, set AUDIT=y and SELINUX_BOOTPARAM=y. Then disable them via kernel cmdline: audit=0 selinux=0. You can also leave audit enabled, if you don't plan to use systemd's containers: http://cgit.freedesktop.org/systemd/systemd/commit/README?id=77b6e19458f37cfde127ec6aa9494c0ac45ad890<br>
<br>
WARNING: CONFIG_RTC_DRV_CMOS is invalid<br>
It is unset<br>
Allowed values : y, !<br>
Comment says: optional, but highly recommended<br>
</details><br>

Осталось несколько предупреждений для конфигураций,<br>
1. которые похоже отсутствуют в ядре, т.к. они были актуальны для старых версий ядра (CONFIG_NETPRIO_CGROUP, CONFIG_CGROUP_MEM_RES_CTLR_KMEM, CONFIG_CGROUP_MEM_RES_CTLR, CONFIG_CGROUP_MEM_RES_CTLR_SWAP, CONFIG_FW_LOADER_USER_HELPER)<br>
2. Из-за архитектуры (CONFIG_RTC_DRV_CMOS, CONFIG_LBDAF)<br>
3. чтобы отключить selinux (CONFIG_AUDIT, CONFIG_SECURITY_SELINUX_BOOTPARAM)<br>

Далее открываем `$ANDROID_ROOT/device/xiaomi/mido/BoardConfig.mk` и правим параметр `BOARD_KERNEL_CMDLINE` добавляя `" selinux=0 audit=0"` должно получиться так:<br>
`BOARD_KERNEL_CMDLINE := androidboot.hardware=qcom msm_rtb.filter=0x237 ehci-hcd.park=3 lpm_levels.sleep_disabled=1 androidboot.bootdevice=7824900.sdhci earlycon=msm_hsl_uart,0x78af000 selinux=0 audit=0`<br>
а также правим параметр `TARGET_KERNEL_CONFIG := mido_sf_defconfig`. Сохраняем изменения. Теперь копируем наш исправленный конфиг, чтобы он не затерся при повторных сборках
```console
HABUILD_SDK [mido]:~/hadk$ cp kernel/xiaomi/msm8953/.config kernel/xiaomi/msm8953/arch/arm64/configs/mido_sf_defconfig
```

Текущий конфиг ядра выглядит так
```console
HABUILD_SDK [mido]:~/hadk$ cat kernel/xiaomi/msm8953/arch/arm64/configs/mido_sf_defconfig
```
<details>
#<br>
# Automatically generated file; DO NOT EDIT.<br>
# Linux/arm64 3.18.31 Kernel Configuration<br>
#<br>
CONFIG_ARM64=y<br>
CONFIG_64BIT=y<br>
CONFIG_ARCH_PHYS_ADDR_T_64BIT=y<br>
CONFIG_MMU=y<br>
CONFIG_ARCH_MMAP_RND_BITS_MIN=18<br>
CONFIG_ARCH_MMAP_RND_BITS_MAX=24<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MIN=11<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MAX=16<br>
CONFIG_ILLEGAL_POINTER_VALUE=0xdead000000000000<br>
CONFIG_STACKTRACE_SUPPORT=y<br>
CONFIG_LOCKDEP_SUPPORT=y<br>
CONFIG_TRACE_IRQFLAGS_SUPPORT=y<br>
CONFIG_RWSEM_XCHGADD_ALGORITHM=y<br>
CONFIG_GENERIC_BUG=y<br>
CONFIG_GENERIC_HWEIGHT=y<br>
CONFIG_GENERIC_CSUM=y<br>
CONFIG_GENERIC_CALIBRATE_DELAY=y<br>
CONFIG_ZONE_DMA=y<br>
CONFIG_HAVE_GENERIC_RCU_GUP=y<br>
CONFIG_ARCH_DMA_ADDR_T_64BIT=y<br>
CONFIG_NEED_DMA_MAP_STATE=y<br>
CONFIG_NEED_SG_DMA_LENGTH=y<br>
CONFIG_ARM64_DMA_USE_IOMMU=y<br>
CONFIG_ARM64_DMA_IOMMU_ALIGNMENT=8<br>
CONFIG_SWIOTLB=y<br>
CONFIG_IOMMU_HELPER=y<br>
CONFIG_KERNEL_MODE_NEON=y<br>
CONFIG_FIX_EARLYCON_MEM=y<br>
CONFIG_PGTABLE_LEVELS=3<br>
CONFIG_DEFCONFIG_LIST="/lib/modules/$UNAME_RELEASE/.config"<br>
CONFIG_IRQ_WORK=y<br>
CONFIG_BUILDTIME_EXTABLE_SORT=y<br>
<br>
#<br>
# General setup<br>
#<br>
CONFIG_BROKEN=y<br>
CONFIG_BROKEN_ON_SMP=y<br>
CONFIG_INIT_ENV_ARG_LIMIT=32<br>
CONFIG_CROSS_COMPILE=""<br>
# CONFIG_COMPILE_TEST is not set<br>
CONFIG_LOCALVERSION="-perf"<br>
CONFIG_LOCALVERSION_AUTO=y<br>
CONFIG_DEFAULT_HOSTNAME="(none)"<br>
CONFIG_SWAP=y<br>
CONFIG_SYSVIPC=y<br>
CONFIG_SYSVIPC_SYSCTL=y<br>
# CONFIG_POSIX_MQUEUE is not set<br>
CONFIG_CROSS_MEMORY_ATTACH=y<br>
CONFIG_FHANDLE=y<br>
CONFIG_USELIB=y<br>
CONFIG_AUDIT=y<br>
CONFIG_HAVE_ARCH_AUDITSYSCALL=y<br>
CONFIG_AUDITSYSCALL=y<br>
CONFIG_AUDIT_WATCH=y<br>
CONFIG_AUDIT_TREE=y<br>
<br>
#<br>
# IRQ subsystem<br>
#<br>
CONFIG_GENERIC_IRQ_PROBE=y<br>
CONFIG_GENERIC_IRQ_SHOW=y<br>
CONFIG_HARDIRQS_SW_RESEND=y<br>
CONFIG_IRQ_DOMAIN=y<br>
CONFIG_IRQ_DOMAIN_HIERARCHY=y<br>
CONFIG_GENERIC_MSI_IRQ=y<br>
CONFIG_GENERIC_MSI_IRQ_DOMAIN=y<br>
CONFIG_HANDLE_DOMAIN_IRQ=y<br>
# CONFIG_IRQ_DOMAIN_DEBUG is not set<br>
CONFIG_SPARSE_IRQ=y<br>
CONFIG_GENERIC_TIME_VSYSCALL=y<br>
CONFIG_GENERIC_CLOCKEVENTS=y<br>
CONFIG_GENERIC_CLOCKEVENTS_BUILD=y<br>
CONFIG_ARCH_HAS_TICK_BROADCAST=y<br>
CONFIG_GENERIC_CLOCKEVENTS_BROADCAST=y<br>
<br>
#<br>
# Timers subsystem<br>
#<br>
CONFIG_TICK_ONESHOT=y<br>
CONFIG_NO_HZ_COMMON=y<br>
# CONFIG_HZ_PERIODIC is not set<br>
CONFIG_NO_HZ_IDLE=y<br>
# CONFIG_NO_HZ_FULL is not set<br>
CONFIG_NO_HZ=y<br>
CONFIG_HIGH_RES_TIMERS=y<br>
<br>
#<br>
# CPU/Task time and stats accounting<br>
#<br>
# CONFIG_TICK_CPU_ACCOUNTING is not set<br>
# CONFIG_VIRT_CPU_ACCOUNTING_GEN is not set<br>
CONFIG_IRQ_TIME_ACCOUNTING=y<br>
# CONFIG_BSD_PROCESS_ACCT is not set<br>
# CONFIG_TASKSTATS is not set<br>
<br>
#<br>
# RCU Subsystem<br>
#<br>
CONFIG_TREE_PREEMPT_RCU=y<br>
CONFIG_PREEMPT_RCU=y<br>
# CONFIG_TASKS_RCU is not set<br>
CONFIG_RCU_STALL_COMMON=y<br>
# CONFIG_RCU_USER_QS is not set<br>
CONFIG_RCU_FANOUT=64<br>
CONFIG_RCU_FANOUT_LEAF=16<br>
# CONFIG_RCU_FANOUT_EXACT is not set<br>
CONFIG_RCU_FAST_NO_HZ=y<br>
# CONFIG_TREE_RCU_TRACE is not set<br>
# CONFIG_RCU_BOOST is not set<br>
CONFIG_RCU_NOCB_CPU=y<br>
# CONFIG_RCU_NOCB_CPU_NONE is not set<br>
# CONFIG_RCU_NOCB_CPU_ZERO is not set<br>
CONFIG_RCU_NOCB_CPU_ALL=y<br>
CONFIG_BUILD_BIN2C=y<br>
CONFIG_IKCONFIG=y<br>
CONFIG_IKCONFIG_PROC=y<br>
CONFIG_LOG_BUF_SHIFT=17<br>
# CONFIG_CONSOLE_FLUSH_ON_HOTPLUG is not set<br>
CONFIG_LOG_CPU_MAX_BUF_SHIFT=12<br>
CONFIG_GENERIC_SCHED_CLOCK=y<br>
CONFIG_CGROUPS=y<br>
# CONFIG_CGROUP_DEBUG is not set<br>
CONFIG_CGROUP_FREEZER=y<br>
CONFIG_CGROUP_DEVICE=y<br>
CONFIG_CPUSETS=y<br>
CONFIG_PROC_PID_CPUSET=y<br>
CONFIG_CGROUP_CPUACCT=y<br>
CONFIG_RESOURCE_COUNTERS=y<br>
CONFIG_MEMCG=y<br>
CONFIG_MEMCG_SWAP=y<br>
CONFIG_MEMCG_SWAP_ENABLED=y<br>
CONFIG_MEMCG_KMEM=y<br>
CONFIG_CGROUP_PERF=y<br>
CONFIG_CGROUP_SCHED=y<br>
CONFIG_FAIR_GROUP_SCHED=y<br>
CONFIG_CFS_BANDWIDTH=y<br>
CONFIG_RT_GROUP_SCHED=y<br>
CONFIG_BLK_CGROUP=y<br>
# CONFIG_DEBUG_BLK_CGROUP is not set<br>
CONFIG_SCHED_HMP=y<br>
# CONFIG_SCHED_HMP_CSTATE_AWARE is not set<br>
# CONFIG_SCHED_CORE_CTL is not set<br>
# CONFIG_SCHED_QHMP is not set<br>
CONFIG_CHECKPOINT_RESTORE=y<br>
CONFIG_NAMESPACES=y<br>
CONFIG_UTS_NS=y<br>
CONFIG_IPC_NS=y<br>
# CONFIG_USER_NS is not set<br>
CONFIG_PID_NS=y<br>
CONFIG_NET_NS=y<br>
# CONFIG_SCHED_AUTOGROUP is not set<br>
# CONFIG_SYSFS_DEPRECATED is not set<br>
# CONFIG_RELAY is not set<br>
CONFIG_BLK_DEV_INITRD=y<br>
CONFIG_INITRAMFS_SOURCE=""<br>
CONFIG_RD_GZIP=y<br>
CONFIG_RD_BZIP2=y<br>
CONFIG_RD_LZMA=y<br>
# CONFIG_RD_XZ is not set<br>
# CONFIG_RD_LZO is not set<br>
# CONFIG_RD_LZ4 is not set<br>
CONFIG_CC_OPTIMIZE_FOR_SIZE=y<br>
CONFIG_SYSCTL=y<br>
CONFIG_ANON_INODES=y<br>
CONFIG_HAVE_UID16=y<br>
CONFIG_SYSCTL_EXCEPTION_TRACE=y<br>
CONFIG_BPF=y<br>
CONFIG_EXPERT=y<br>
CONFIG_UID16=y<br>
# CONFIG_SGETMASK_SYSCALL is not set<br>
CONFIG_SYSFS_SYSCALL=y<br>
# CONFIG_SYSCTL_SYSCALL is not set<br>
CONFIG_KALLSYMS=y<br>
CONFIG_KALLSYMS_ALL=y<br>
CONFIG_PRINTK=y<br>
CONFIG_BUG=y<br>
CONFIG_ELF_CORE=y<br>
CONFIG_BASE_FULL=y<br>
CONFIG_FUTEX=y<br>
CONFIG_EPOLL=y<br>
CONFIG_SIGNALFD=y<br>
CONFIG_TIMERFD=y<br>
CONFIG_EVENTFD=y<br>
# CONFIG_BPF_SYSCALL is not set<br>
CONFIG_SHMEM=y<br>
CONFIG_AIO=y<br>
CONFIG_ADVISE_SYSCALLS=y<br>
CONFIG_PCI_QUIRKS=y<br>
CONFIG_EMBEDDED=y<br>
CONFIG_HAVE_PERF_EVENTS=y<br>
CONFIG_PERF_USE_VMALLOC=y<br>
<br>
#<br>
# Kernel Performance Events And Counters<br>
#<br>
CONFIG_PERF_EVENTS=y<br>
# CONFIG_DEBUG_PERF_USE_VMALLOC is not set<br>
CONFIG_VM_EVENT_COUNTERS=y<br>
# CONFIG_SLUB_DEBUG is not set<br>
CONFIG_COMPAT_BRK=y<br>
# CONFIG_SLAB is not set<br>
CONFIG_SLUB=y<br>
# CONFIG_SLOB is not set<br>
CONFIG_SLUB_CPU_PARTIAL=y<br>
CONFIG_SYSTEM_TRUSTED_KEYRING=y<br>
CONFIG_PROFILING=y<br>
CONFIG_TRACEPOINTS=y<br>
# CONFIG_JUMP_LABEL is not set<br>
# CONFIG_UPROBES is not set<br>
# CONFIG_HAVE_64BIT_ALIGNED_ACCESS is not set<br>
CONFIG_HAVE_EFFICIENT_UNALIGNED_ACCESS=y<br>
CONFIG_HAVE_ARCH_TRACEHOOK=y<br>
CONFIG_HAVE_DMA_ATTRS=y<br>
CONFIG_HAVE_DMA_CONTIGUOUS=y<br>
CONFIG_GENERIC_SMP_IDLE_THREAD=y<br>
CONFIG_HAVE_CLK=y<br>
CONFIG_HAVE_DMA_API_DEBUG=y<br>
CONFIG_HAVE_PERF_REGS=y<br>
CONFIG_HAVE_PERF_USER_STACK_DUMP=y<br>
CONFIG_HAVE_ARCH_JUMP_LABEL=y<br>
CONFIG_HAVE_RCU_TABLE_FREE=y<br>
CONFIG_HAVE_ALIGNED_STRUCT_PAGE=y<br>
CONFIG_HAVE_CMPXCHG_DOUBLE=y<br>
CONFIG_ARCH_WANT_COMPAT_IPC_PARSE_VERSION=y<br>
CONFIG_HAVE_ARCH_SECCOMP_FILTER=y<br>
CONFIG_SECCOMP_FILTER=y<br>
CONFIG_HAVE_CC_STACKPROTECTOR=y<br>
CONFIG_CC_STACKPROTECTOR=y<br>
# CONFIG_CC_STACKPROTECTOR_NONE is not set<br>
# CONFIG_CC_STACKPROTECTOR_REGULAR is not set<br>
CONFIG_CC_STACKPROTECTOR_STRONG=y<br>
CONFIG_HAVE_CONTEXT_TRACKING=y<br>
CONFIG_HAVE_VIRT_CPU_ACCOUNTING_GEN=y<br>
CONFIG_HAVE_IRQ_TIME_ACCOUNTING=y<br>
CONFIG_HAVE_ARCH_TRANSPARENT_HUGEPAGE=y<br>
CONFIG_MODULES_USE_ELF_RELA=y<br>
CONFIG_HAVE_ARCH_MMAP_RND_BITS=y<br>
CONFIG_ARCH_MMAP_RND_BITS=18<br>
CONFIG_HAVE_ARCH_MMAP_RND_COMPAT_BITS=y<br>
CONFIG_ARCH_MMAP_RND_COMPAT_BITS=16<br>
CONFIG_CLONE_BACKWARDS=y<br>
CONFIG_OLD_SIGSUSPEND3=y<br>
CONFIG_COMPAT_OLD_SIGACTION=y<br>
<br>
#<br>
# GCOV-based kernel profiling<br>
#<br>
# CONFIG_GCOV_KERNEL is not set<br>
CONFIG_HAVE_GENERIC_DMA_COHERENT=y<br>
CONFIG_RT_MUTEXES=y<br>
CONFIG_BASE_SMALL=0<br>
# CONFIG_MODULES is not set<br>
CONFIG_STOP_MACHINE=y<br>
CONFIG_BLOCK=y<br>
CONFIG_BLK_DEV_BSG=y<br>
# CONFIG_BLK_DEV_BSGLIB is not set<br>
# CONFIG_BLK_DEV_INTEGRITY is not set<br>
# CONFIG_BLK_DEV_THROTTLING is not set<br>
# CONFIG_BLK_CMDLINE_PARSER is not set<br>
<br>
#<br>
# Partition Types<br>
#<br>
CONFIG_PARTITION_ADVANCED=y<br>
# CONFIG_ACORN_PARTITION is not set<br>
# CONFIG_AIX_PARTITION is not set<br>
# CONFIG_OSF_PARTITION is not set<br>
# CONFIG_AMIGA_PARTITION is not set<br>
# CONFIG_ATARI_PARTITION is not set<br>
# CONFIG_MAC_PARTITION is not set<br>
CONFIG_MSDOS_PARTITION=y<br>
# CONFIG_BSD_DISKLABEL is not set<br>
# CONFIG_MINIX_SUBPARTITION is not set<br>
# CONFIG_SOLARIS_X86_PARTITION is not set<br>
# CONFIG_UNIXWARE_DISKLABEL is not set<br>
# CONFIG_LDM_PARTITION is not set<br>
# CONFIG_SGI_PARTITION is not set<br>
# CONFIG_ULTRIX_PARTITION is not set<br>
# CONFIG_SUN_PARTITION is not set<br>
# CONFIG_KARMA_PARTITION is not set<br>
CONFIG_EFI_PARTITION=y<br>
# CONFIG_SYSV68_PARTITION is not set<br>
# CONFIG_CMDLINE_PARTITION is not set<br>
CONFIG_BLOCK_COMPAT=y<br>
<br>
#<br>
# IO Schedulers<br>
#<br>
CONFIG_IOSCHED_NOOP=y<br>
# CONFIG_IOSCHED_TEST is not set<br>
CONFIG_IOSCHED_DEADLINE=y<br>
CONFIG_IOSCHED_CFQ=y<br>
# CONFIG_CFQ_GROUP_IOSCHED is not set<br>
# CONFIG_DEFAULT_DEADLINE is not set<br>
CONFIG_DEFAULT_CFQ=y<br>
# CONFIG_DEFAULT_NOOP is not set<br>
CONFIG_DEFAULT_IOSCHED="cfq"<br>
CONFIG_ASN1=y<br>
CONFIG_UNINLINE_SPIN_UNLOCK=y<br>
CONFIG_ARCH_SUPPORTS_ATOMIC_RMW=y<br>
CONFIG_MUTEX_SPIN_ON_OWNER=y<br>
CONFIG_RWSEM_SPIN_ON_OWNER=y<br>
CONFIG_FREEZER=y<br>
<br>
#<br>
# Platform selection<br>
#<br>
# CONFIG_ARCH_THUNDER is not set<br>
# CONFIG_ARCH_VEXPRESS is not set<br>
# CONFIG_ARCH_XGENE is not set<br>
CONFIG_ARCH_MSM=y<br>
# CONFIG_ARCH_MSM8916 is not set<br>
# CONFIG_ARCH_MSM8917 is not set<br>
# CONFIG_ARCH_MSM8920 is not set<br>
# CONFIG_ARCH_MSM8940 is not set<br>
CONFIG_ARCH_MSM8953=y<br>
# CONFIG_ARCH_SDM450 is not set<br>
# CONFIG_ARCH_MSM8937 is not set<br>
# CONFIG_ARCH_MSM8996 is not set<br>
# CONFIG_ARCH_MSMCOBALT is not set<br>
<br>
#<br>
# Bus support<br>
#<br>
CONFIG_ARM_AMBA=y<br>
CONFIG_PCI=y<br>
CONFIG_PCI_DOMAINS=y<br>
CONFIG_PCI_DOMAINS_GENERIC=y<br>
CONFIG_PCI_SYSCALL=y<br>
CONFIG_PCI_MSI=y<br>
CONFIG_PCI_MSI_IRQ_DOMAIN=y<br>
# CONFIG_PCI_DEBUG is not set<br>
# CONFIG_PCI_REALLOC_ENABLE_AUTO is not set<br>
# CONFIG_PCI_STUB is not set<br>
# CONFIG_PCI_IOV is not set<br>
# CONFIG_PCI_PRI is not set<br>
# CONFIG_PCI_PASID is not set<br>
CONFIG_PCI_MSM=y<br>
CONFIG_PCI_LABEL=y<br>
<br>
#<br>
# PCI host controller drivers<br>
#<br>
# CONFIG_PCI_HOST_GENERIC is not set<br>
# CONFIG_PCIEPORTBUS is not set<br>
# CONFIG_HOTPLUG_PCI is not set<br>
<br>
#<br>
# Kernel Features<br>
#<br>
<br>
#<br>
# ARM errata workarounds via the alternatives framework<br>
#<br>
CONFIG_ARM64_ERRATUM_826319=y<br>
CONFIG_ARM64_ERRATUM_827319=y<br>
CONFIG_ARM64_ERRATUM_824069=y<br>
CONFIG_ARM64_ERRATUM_819472=y<br>
CONFIG_ARM64_ERRATUM_832075=y<br>
CONFIG_ARM64_ERRATUM_845719=y<br>
CONFIG_ARM64_4K_PAGES=y<br>
# CONFIG_ARM64_64K_PAGES is not set<br>
# CONFIG_ARM64_DCACHE_DISABLE is not set<br>
# CONFIG_ARM64_ICACHE_DISABLE is not set<br>
CONFIG_ARCH_MSM8953_SOC_SETTINGS=y<br>
CONFIG_ARM64_VA_BITS_39=y<br>
CONFIG_ARM64_VA_BITS=39<br>
CONFIG_ARM64_PGTABLE_LEVELS=3<br>
# CONFIG_CPU_BIG_ENDIAN is not set<br>
# CONFIG_ARM64_SEV_IN_LOCK_UNLOCK is not set<br>
CONFIG_SMP=y<br>
CONFIG_SCHED_MC=y<br>
# CONFIG_SCHED_SMT is not set<br>
CONFIG_NR_CPUS=8<br>
CONFIG_HOTPLUG_CPU=y<br>
CONFIG_ARCH_NR_GPIO=1024<br>
# CONFIG_PREEMPT_NONE is not set<br>
# CONFIG_PREEMPT_VOLUNTARY is not set<br>
CONFIG_PREEMPT=y<br>
CONFIG_PREEMPT_COUNT=y<br>
CONFIG_HZ=100<br>
CONFIG_ARCH_HAS_HOLES_MEMORYMODEL=y<br>
CONFIG_ARCH_SPARSEMEM_ENABLE=y<br>
CONFIG_ARCH_SPARSEMEM_DEFAULT=y<br>
CONFIG_ARCH_SELECT_MEMORY_MODEL=y<br>
CONFIG_HAVE_ARCH_PFN_VALID=y<br>
CONFIG_HW_PERF_EVENTS=y<br>
# CONFIG_PERF_EVENTS_USERMODE is not set<br>
# CONFIG_PERF_EVENTS_RESET_PMU_DEBUGFS is not set<br>
# CONFIG_ARM64_REG_REBALANCE_ON_CTX_SW is not set<br>
CONFIG_SYS_SUPPORTS_HUGETLBFS=y<br>
CONFIG_ARCH_WANT_GENERAL_HUGETLB=y<br>
CONFIG_ARCH_WANT_HUGE_PMD_SHARE=y<br>
CONFIG_ARCH_HAS_CACHE_LINE_SIZE=y<br>
CONFIG_SELECT_MEMORY_MODEL=y<br>
CONFIG_SPARSEMEM_MANUAL=y<br>
CONFIG_SPARSEMEM=y<br>
CONFIG_HAVE_MEMORY_PRESENT=y<br>
CONFIG_SPARSEMEM_EXTREME=y<br>
CONFIG_SPARSEMEM_VMEMMAP_ENABLE=y<br>
CONFIG_SPARSEMEM_VMEMMAP=y<br>
CONFIG_HAVE_MEMBLOCK=y<br>
CONFIG_NO_BOOTMEM=y<br>
CONFIG_MEMORY_ISOLATION=y<br>
# CONFIG_HAVE_BOOTMEM_INFO_NODE is not set<br>
CONFIG_PAGEFLAGS_EXTENDED=y<br>
CONFIG_SPLIT_PTLOCK_CPUS=4<br>
CONFIG_COMPACTION=y<br>
CONFIG_MIGRATION=y<br>
CONFIG_PHYS_ADDR_T_64BIT=y<br>
CONFIG_ZONE_DMA_FLAG=1<br>
CONFIG_BOUNCE=y<br>
# CONFIG_KSM is not set<br>
CONFIG_DEFAULT_MMAP_MIN_ADDR=4096<br>
# CONFIG_TRANSPARENT_HUGEPAGE is not set<br>
CONFIG_CLEANCACHE=y<br>
# CONFIG_FRONTSWAP is not set<br>
CONFIG_CMA=y<br>
# CONFIG_CMA_DEBUG is not set<br>
CONFIG_CMA_DEBUGFS=y<br>
CONFIG_CMA_AREAS=7<br>
# CONFIG_ZPOOL is not set<br>
CONFIG_ZBUD=y<br>
CONFIG_ZSMALLOC=y<br>
# CONFIG_PGTABLE_MAPPING is not set<br>
# CONFIG_ZSMALLOC_STAT is not set<br>
CONFIG_GENERIC_EARLY_IOREMAP=y<br>
CONFIG_ZCACHE=y<br>
# CONFIG_BALANCE_ANON_FILE_RECLAIM is not set<br>
CONFIG_KSWAPD_CPU_AFFINITY_MASK=""<br>
# CONFIG_FORCE_ALLOC_FROM_DMA_ZONE is not set<br>
CONFIG_PROCESS_RECLAIM=y<br>
CONFIG_SECCOMP=y<br>
# CONFIG_XEN is not set<br>
CONFIG_FORCE_MAX_ZONEORDER=11<br>
CONFIG_ARM64_PAN=y<br>
CONFIG_ARMV8_DEPRECATED=y<br>
CONFIG_SWP_EMULATION=y<br>
CONFIG_CP15_BARRIER_EMULATION=y<br>
CONFIG_SETEND_EMULATION=y<br>
<br>
#<br>
# Boot options<br>
#<br>
CONFIG_CMDLINE=""<br>
CONFIG_EFI_STUB=y<br>
CONFIG_EFI=y<br>
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE=y<br>
CONFIG_BUILD_ARM64_APPENDED_DTB_IMAGE_NAMES=""<br>
CONFIG_DMI=y<br>
<br>
#<br>
# Userspace binary formats<br>
#<br>
CONFIG_BINFMT_ELF=y<br>
CONFIG_COMPAT_BINFMT_ELF=y<br>
# CONFIG_CORE_DUMP_DEFAULT_ELF_HEADERS is not set<br>
CONFIG_BINFMT_SCRIPT=y<br>
# CONFIG_HAVE_AOUT is not set<br>
# CONFIG_BINFMT_MISC is not set<br>
CONFIG_COREDUMP=y<br>
CONFIG_COMPAT=y<br>
CONFIG_SYSVIPC_COMPAT=y<br>
<br>
#<br>
# Power management options<br>
#<br>
CONFIG_SUSPEND=y<br>
CONFIG_SUSPEND_FREEZER=y<br>
CONFIG_WAKELOCK=y<br>
CONFIG_PM_SLEEP=y<br>
CONFIG_PM_SLEEP_SMP=y<br>
CONFIG_PM_AUTOSLEEP=y<br>
CONFIG_PM_WAKELOCKS=y<br>
CONFIG_PM_WAKELOCKS_LIMIT=0<br>
# CONFIG_PM_WAKELOCKS_GC is not set<br>
CONFIG_PM_RUNTIME=y<br>
CONFIG_PM=y<br>
CONFIG_PM_DEBUG=y<br>
# CONFIG_PM_ADVANCED_DEBUG is not set<br>
# CONFIG_PM_TEST_SUSPEND is not set<br>
CONFIG_PM_SLEEP_DEBUG=y<br>
# CONFIG_DPM_WATCHDOG is not set<br>
CONFIG_PM_OPP=y<br>
CONFIG_PM_CLK=y<br>
# CONFIG_WQ_POWER_EFFICIENT_DEFAULT is not set<br>
CONFIG_CPU_PM=y<br>
CONFIG_SUSPEND_TIME=y<br>
CONFIG_ARCH_SUSPEND_POSSIBLE=y<br>
CONFIG_ARM64_CPU_SUSPEND=y<br>
<br>
#<br>
# CPU Power Management<br>
#<br>
<br>
#<br>
# CPU Idle<br>
#<br>
CONFIG_CPU_IDLE=y<br>
CONFIG_CPU_IDLE_MULTIPLE_DRIVERS=y<br>
CONFIG_CPU_IDLE_GOV_LADDER=y<br>
CONFIG_CPU_IDLE_GOV_MENU=y<br>
<br>
#<br>
# ARM64 CPU Idle Drivers<br>
#<br>
# CONFIG_ARM64_CPUIDLE is not set<br>
# CONFIG_ARCH_NEEDS_CPU_IDLE_COUPLED is not set<br>
<br>
#<br>
# CPU Frequency scaling<br>
#<br>
CONFIG_CPU_FREQ=y<br>
CONFIG_CPU_FREQ_GOV_COMMON=y<br>
CONFIG_SCHED_FREQ_INPUT=y<br>
CONFIG_CPU_FREQ_STAT=y<br>
# CONFIG_CPU_FREQ_STAT_DETAILS is not set<br>
CONFIG_CPU_FREQ_DEFAULT_GOV_PERFORMANCE=y<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_POWERSAVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_USERSPACE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_ONDEMAND is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_CONSERVATIVE is not set<br>
# CONFIG_CPU_FREQ_DEFAULT_GOV_INTERACTIVE is not set<br>
CONFIG_CPU_FREQ_GOV_PERFORMANCE=y<br>
CONFIG_CPU_FREQ_GOV_POWERSAVE=y<br>
CONFIG_CPU_FREQ_GOV_USERSPACE=y<br>
CONFIG_CPU_FREQ_GOV_ONDEMAND=y<br>
CONFIG_CPU_FREQ_GOV_INTERACTIVE=y<br>
CONFIG_CPU_FREQ_GOV_CONSERVATIVE=y<br>
# CONFIG_CPUFREQ_DT is not set<br>
# CONFIG_CPU_BOOST is not set<br>
<br>
#<br>
# ARM CPU frequency scaling drivers<br>
#<br>
# CONFIG_ARM_KIRKWOOD_CPUFREQ is not set<br>
CONFIG_CPU_FREQ_MSM=y<br>
CONFIG_NET=y<br>
CONFIG_COMPAT_NETLINK_MESSAGES=y<br>
# CONFIG_DISABLE_NET_SKB_FRAG_CACHE is not set<br>
<br>
#<br>
# Networking options<br>
#<br>
CONFIG_PACKET=y<br>
# CONFIG_PACKET_DIAG is not set<br>
CONFIG_UNIX=y<br>
# CONFIG_UNIX_DIAG is not set<br>
CONFIG_XFRM=y<br>
CONFIG_XFRM_ALGO=y<br>
CONFIG_XFRM_USER=y<br>
# CONFIG_XFRM_SUB_POLICY is not set<br>
# CONFIG_XFRM_MIGRATE is not set<br>
CONFIG_XFRM_STATISTICS=y<br>
CONFIG_XFRM_IPCOMP=y<br>
CONFIG_NET_KEY=y<br>
# CONFIG_NET_KEY_MIGRATE is not set<br>
CONFIG_INET=y<br>
CONFIG_IP_MULTICAST=y<br>
CONFIG_IP_ADVANCED_ROUTER=y<br>
# CONFIG_IP_FIB_TRIE_STATS is not set<br>
CONFIG_IP_MULTIPLE_TABLES=y<br>
# CONFIG_IP_ROUTE_MULTIPATH is not set<br>
CONFIG_IP_ROUTE_VERBOSE=y<br>
CONFIG_IP_PNP=y<br>
CONFIG_IP_PNP_DHCP=y<br>
# CONFIG_IP_PNP_BOOTP is not set<br>
# CONFIG_IP_PNP_RARP is not set<br>
# CONFIG_NET_IPIP is not set<br>
# CONFIG_NET_IPGRE_DEMUX is not set<br>
CONFIG_NET_IP_TUNNEL=y<br>
# CONFIG_IP_MROUTE is not set<br>
# CONFIG_SYN_COOKIES is not set<br>
# CONFIG_NET_IPVTI is not set<br>
CONFIG_NET_UDP_TUNNEL=y<br>
# CONFIG_NET_FOU is not set<br>
# CONFIG_GENEVE is not set<br>
CONFIG_INET_AH=y<br>
CONFIG_INET_ESP=y<br>
CONFIG_INET_IPCOMP=y<br>
CONFIG_INET_XFRM_TUNNEL=y<br>
CONFIG_INET_TUNNEL=y<br>
CONFIG_INET_XFRM_MODE_TRANSPORT=y<br>
CONFIG_INET_XFRM_MODE_TUNNEL=y<br>
# CONFIG_INET_XFRM_MODE_BEET is not set<br>
# CONFIG_INET_LRO is not set<br>
CONFIG_INET_DIAG=y<br>
CONFIG_INET_TCP_DIAG=y<br>
# CONFIG_INET_UDP_DIAG is not set<br>
CONFIG_INET_DIAG_DESTROY=y<br>
# CONFIG_TCP_CONG_ADVANCED is not set<br>
CONFIG_TCP_CONG_CUBIC=y<br>
CONFIG_DEFAULT_TCP_CONG="cubic"<br>
# CONFIG_TCP_MD5SIG is not set<br>
CONFIG_IPV6=y<br>
CONFIG_IPV6_ROUTER_PREF=y<br>
CONFIG_IPV6_ROUTE_INFO=y<br>
CONFIG_IPV6_OPTIMISTIC_DAD=y<br>
CONFIG_INET6_AH=y<br>
CONFIG_INET6_ESP=y<br>
CONFIG_INET6_IPCOMP=y<br>
CONFIG_IPV6_MIP6=y<br>
CONFIG_INET6_XFRM_TUNNEL=y<br>
CONFIG_INET6_TUNNEL=y<br>
CONFIG_INET6_XFRM_MODE_TRANSPORT=y<br>
CONFIG_INET6_XFRM_MODE_TUNNEL=y<br>
CONFIG_INET6_XFRM_MODE_BEET=y<br>
# CONFIG_INET6_XFRM_MODE_ROUTEOPTIMIZATION is not set<br>
# CONFIG_IPV6_VTI is not set<br>
CONFIG_IPV6_SIT=y<br>
# CONFIG_IPV6_SIT_6RD is not set<br>
CONFIG_IPV6_NDISC_NODETYPE=y<br>
# CONFIG_IPV6_TUNNEL is not set<br>
# CONFIG_IPV6_GRE is not set<br>
CONFIG_IPV6_MULTIPLE_TABLES=y<br>
CONFIG_IPV6_SUBTREES=y<br>
# CONFIG_IPV6_MROUTE is not set<br>
# CONFIG_NETLABEL is not set<br>
CONFIG_ANDROID_PARANOID_NETWORK=y<br>
CONFIG_NET_ACTIVITY_STATS=y<br>
CONFIG_NETWORK_SECMARK=y<br>
# CONFIG_NET_PTP_CLASSIFY is not set<br>
# CONFIG_NETWORK_PHY_TIMESTAMPING is not set<br>
CONFIG_NETFILTER=y<br>
# CONFIG_NETFILTER_DEBUG is not set<br>
CONFIG_NETFILTER_ADVANCED=y<br>
CONFIG_BRIDGE_NETFILTER=y<br>
<br>
#<br>
# Core Netfilter Configuration<br>
#<br>
CONFIG_NETFILTER_NETLINK=y<br>
CONFIG_NETFILTER_NETLINK_ACCT=y<br>
CONFIG_NETFILTER_NETLINK_QUEUE=y<br>
CONFIG_NETFILTER_NETLINK_LOG=y<br>
CONFIG_NF_CONNTRACK=y<br>
CONFIG_NF_LOG_COMMON=y<br>
CONFIG_NF_CONNTRACK_MARK=y<br>
CONFIG_NF_CONNTRACK_SECMARK=y<br>
# CONFIG_NF_CONNTRACK_ZONES is not set<br>
CONFIG_NF_CONNTRACK_PROCFS=y<br>
CONFIG_NF_CONNTRACK_EVENTS=y<br>
# CONFIG_NF_CONNTRACK_TIMEOUT is not set<br>
# CONFIG_NF_CONNTRACK_TIMESTAMP is not set<br>
CONFIG_NF_CT_PROTO_DCCP=y<br>
CONFIG_NF_CT_PROTO_GRE=y<br>
CONFIG_NF_CT_PROTO_SCTP=y<br>
CONFIG_NF_CT_PROTO_UDPLITE=y<br>
CONFIG_NF_CONNTRACK_AMANDA=y<br>
CONFIG_NF_CONNTRACK_FTP=y<br>
CONFIG_NF_CONNTRACK_H323=y<br>
CONFIG_NF_CONNTRACK_IRC=y<br>
CONFIG_NF_CONNTRACK_BROADCAST=y<br>
CONFIG_NF_CONNTRACK_NETBIOS_NS=y<br>
# CONFIG_NF_CONNTRACK_SNMP is not set<br>
CONFIG_NF_CONNTRACK_PPTP=y<br>
CONFIG_NF_CONNTRACK_SANE=y<br>
# CONFIG_NF_CONNTRACK_SIP is not set<br>
CONFIG_NF_CONNTRACK_TFTP=y<br>
CONFIG_NF_CT_NETLINK=y<br>
# CONFIG_NF_CT_NETLINK_TIMEOUT is not set<br>
# CONFIG_NETFILTER_NETLINK_QUEUE_CT is not set<br>
CONFIG_NF_NAT=y<br>
CONFIG_NF_NAT_NEEDED=y<br>
CONFIG_NF_NAT_PROTO_DCCP=y<br>
CONFIG_NF_NAT_PROTO_UDPLITE=y<br>
CONFIG_NF_NAT_PROTO_SCTP=y<br>
CONFIG_NF_NAT_AMANDA=y<br>
CONFIG_NF_NAT_FTP=y<br>
CONFIG_NF_NAT_IRC=y<br>
# CONFIG_NF_NAT_SIP is not set<br>
CONFIG_NF_NAT_TFTP=y<br>
# CONFIG_NF_TABLES is not set<br>
CONFIG_NETFILTER_XTABLES=y<br>
<br>
#<br>
# Xtables combined modules<br>
#<br>
CONFIG_NETFILTER_XT_MARK=y<br>
CONFIG_NETFILTER_XT_CONNMARK=y<br>
<br>
#<br>
# Xtables targets<br>
#<br>
# CONFIG_NETFILTER_XT_TARGET_AUDIT is not set<br>
# CONFIG_NETFILTER_XT_TARGET_CHECKSUM is not set<br>
CONFIG_NETFILTER_XT_TARGET_CLASSIFY=y<br>
CONFIG_NETFILTER_XT_TARGET_CONNMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_CONNSECMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_CT=y<br>
# CONFIG_NETFILTER_XT_TARGET_DSCP is not set<br>
# CONFIG_NETFILTER_XT_TARGET_HL is not set<br>
# CONFIG_NETFILTER_XT_TARGET_HMARK is not set<br>
CONFIG_NETFILTER_XT_TARGET_IDLETIMER=y<br>
CONFIG_NETFILTER_XT_TARGET_HARDIDLETIMER=y<br>
# CONFIG_NETFILTER_XT_TARGET_LED is not set<br>
CONFIG_NETFILTER_XT_TARGET_LOG=y<br>
CONFIG_NETFILTER_XT_TARGET_MARK=y<br>
CONFIG_NETFILTER_XT_NAT=y<br>
CONFIG_NETFILTER_XT_TARGET_NETMAP=y<br>
CONFIG_NETFILTER_XT_TARGET_NFLOG=y<br>
CONFIG_NETFILTER_XT_TARGET_NFQUEUE=y<br>
CONFIG_NETFILTER_XT_TARGET_NOTRACK=y<br>
# CONFIG_NETFILTER_XT_TARGET_RATEEST is not set<br>
CONFIG_NETFILTER_XT_TARGET_REDIRECT=y<br>
CONFIG_NETFILTER_XT_TARGET_TEE=y<br>
CONFIG_NETFILTER_XT_TARGET_TPROXY=y<br>
CONFIG_NETFILTER_XT_TARGET_TRACE=y<br>
CONFIG_NETFILTER_XT_TARGET_SECMARK=y<br>
CONFIG_NETFILTER_XT_TARGET_TCPMSS=y<br>
# CONFIG_NETFILTER_XT_TARGET_TCPOPTSTRIP is not set<br>
<br>
#<br>
# Xtables matches<br>
#<br>
# CONFIG_NETFILTER_XT_MATCH_ADDRTYPE is not set<br>
# CONFIG_NETFILTER_XT_MATCH_BPF is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CGROUP is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CLUSTER is not set<br>
CONFIG_NETFILTER_XT_MATCH_COMMENT=y<br>
# CONFIG_NETFILTER_XT_MATCH_CONNBYTES is not set<br>
# CONFIG_NETFILTER_XT_MATCH_CONNLABEL is not set<br>
CONFIG_NETFILTER_XT_MATCH_CONNLIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_CONNMARK=y<br>
CONFIG_NETFILTER_XT_MATCH_CONNTRACK=y<br>
# CONFIG_NETFILTER_XT_MATCH_CPU is not set<br>
# CONFIG_NETFILTER_XT_MATCH_DCCP is not set<br>
# CONFIG_NETFILTER_XT_MATCH_DEVGROUP is not set<br>
CONFIG_NETFILTER_XT_MATCH_DSCP=y<br>
CONFIG_NETFILTER_XT_MATCH_ECN=y<br>
CONFIG_NETFILTER_XT_MATCH_ESP=y<br>
CONFIG_NETFILTER_XT_MATCH_HASHLIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_HELPER=y<br>
CONFIG_NETFILTER_XT_MATCH_HL=y<br>
# CONFIG_NETFILTER_XT_MATCH_IPCOMP is not set<br>
CONFIG_NETFILTER_XT_MATCH_IPRANGE=y<br>
CONFIG_NETFILTER_XT_MATCH_L2TP=y<br>
CONFIG_NETFILTER_XT_MATCH_LENGTH=y<br>
CONFIG_NETFILTER_XT_MATCH_LIMIT=y<br>
CONFIG_NETFILTER_XT_MATCH_MAC=y<br>
CONFIG_NETFILTER_XT_MATCH_MARK=y<br>
CONFIG_NETFILTER_XT_MATCH_MULTIPORT=y<br>
CONFIG_NETFILTER_XT_MATCH_NFACCT=y<br>
# CONFIG_NETFILTER_XT_MATCH_OSF is not set<br>
# CONFIG_NETFILTER_XT_MATCH_OWNER is not set<br>
CONFIG_NETFILTER_XT_MATCH_POLICY=y<br>
# CONFIG_NETFILTER_XT_MATCH_PHYSDEV is not set<br>
CONFIG_NETFILTER_XT_MATCH_PKTTYPE=y<br>
CONFIG_NETFILTER_XT_MATCH_QTAGUID=y<br>
CONFIG_NETFILTER_XT_MATCH_QUOTA=y<br>
CONFIG_NETFILTER_XT_MATCH_QUOTA2=y<br>
# CONFIG_NETFILTER_XT_MATCH_RATEEST is not set<br>
# CONFIG_NETFILTER_XT_MATCH_REALM is not set<br>
# CONFIG_NETFILTER_XT_MATCH_RECENT is not set<br>
# CONFIG_NETFILTER_XT_MATCH_SCTP is not set<br>
CONFIG_NETFILTER_XT_MATCH_SOCKET=y<br>
CONFIG_NETFILTER_XT_MATCH_STATE=y<br>
CONFIG_NETFILTER_XT_MATCH_STATISTIC=y<br>
CONFIG_NETFILTER_XT_MATCH_STRING=y<br>
# CONFIG_NETFILTER_XT_MATCH_TCPMSS is not set<br>
CONFIG_NETFILTER_XT_MATCH_TIME=y<br>
CONFIG_NETFILTER_XT_MATCH_U32=y<br>
# CONFIG_IP_SET is not set<br>
# CONFIG_IP_VS is not set<br>
<br>
#<br>
# IP: Netfilter Configuration<br>
#<br>
CONFIG_NF_DEFRAG_IPV4=y<br>
CONFIG_NF_CONNTRACK_IPV4=y<br>
CONFIG_NF_CONNTRACK_PROC_COMPAT=y<br>
# CONFIG_NF_LOG_ARP is not set<br>
CONFIG_NF_LOG_IPV4=y<br>
CONFIG_NF_REJECT_IPV4=y<br>
CONFIG_NF_NAT_IPV4=y<br>
CONFIG_NF_NAT_MASQUERADE_IPV4=y<br>
CONFIG_NF_NAT_PROTO_GRE=y<br>
CONFIG_NF_NAT_PPTP=y<br>
CONFIG_NF_NAT_H323=y<br>
CONFIG_IP_NF_IPTABLES=y<br>
CONFIG_IP_NF_MATCH_AH=y<br>
CONFIG_IP_NF_MATCH_ECN=y<br>
# CONFIG_IP_NF_MATCH_RPFILTER is not set<br>
CONFIG_IP_NF_MATCH_TTL=y<br>
CONFIG_IP_NF_FILTER=y<br>
CONFIG_IP_NF_TARGET_REJECT=y<br>
# CONFIG_IP_NF_TARGET_SYNPROXY is not set<br>
CONFIG_IP_NF_NAT=y<br>
CONFIG_IP_NF_TARGET_MASQUERADE=y<br>
CONFIG_IP_NF_TARGET_NATTYPE_MODULE=y<br>
CONFIG_IP_NF_TARGET_NETMAP=y<br>
CONFIG_IP_NF_TARGET_REDIRECT=y<br>
CONFIG_IP_NF_MANGLE=y<br>
# CONFIG_IP_NF_TARGET_CLUSTERIP is not set<br>
# CONFIG_IP_NF_TARGET_ECN is not set<br>
# CONFIG_IP_NF_TARGET_TTL is not set<br>
CONFIG_IP_NF_RAW=y<br>
CONFIG_IP_NF_SECURITY=y<br>
CONFIG_IP_NF_ARPTABLES=y<br>
CONFIG_IP_NF_ARPFILTER=y<br>
CONFIG_IP_NF_ARP_MANGLE=y<br>
<br>
#<br>
# IPv6: Netfilter Configuration<br>
#<br>
CONFIG_NF_DEFRAG_IPV6=y<br>
CONFIG_NF_CONNTRACK_IPV6=y<br>
CONFIG_NF_REJECT_IPV6=y<br>
CONFIG_NF_LOG_IPV6=y<br>
# CONFIG_NF_NAT_IPV6 is not set<br>
CONFIG_IP6_NF_IPTABLES=y<br>
# CONFIG_IP6_NF_MATCH_AH is not set<br>
# CONFIG_IP6_NF_MATCH_EUI64 is not set<br>
# CONFIG_IP6_NF_MATCH_FRAG is not set<br>
# CONFIG_IP6_NF_MATCH_OPTS is not set<br>
# CONFIG_IP6_NF_MATCH_HL is not set<br>
# CONFIG_IP6_NF_MATCH_IPV6HEADER is not set<br>
# CONFIG_IP6_NF_MATCH_MH is not set<br>
CONFIG_IP6_NF_MATCH_RPFILTER=y<br>
# CONFIG_IP6_NF_MATCH_RT is not set<br>
# CONFIG_IP6_NF_TARGET_HL is not set<br>
CONFIG_IP6_NF_FILTER=y<br>
CONFIG_IP6_NF_TARGET_REJECT=y<br>
# CONFIG_IP6_NF_TARGET_SYNPROXY is not set<br>
CONFIG_IP6_NF_MANGLE=y<br>
CONFIG_IP6_NF_RAW=y<br>
# CONFIG_IP6_NF_SECURITY is not set<br>
# CONFIG_IP6_NF_NAT is not set<br>
CONFIG_BRIDGE_NF_EBTABLES=y<br>
CONFIG_BRIDGE_EBT_BROUTE=y<br>
# CONFIG_BRIDGE_EBT_T_FILTER is not set<br>
# CONFIG_BRIDGE_EBT_T_NAT is not set<br>
# CONFIG_BRIDGE_EBT_802_3 is not set<br>
# CONFIG_BRIDGE_EBT_AMONG is not set<br>
# CONFIG_BRIDGE_EBT_ARP is not set<br>
# CONFIG_BRIDGE_EBT_IP is not set<br>
# CONFIG_BRIDGE_EBT_IP6 is not set<br>
# CONFIG_BRIDGE_EBT_LIMIT is not set<br>
# CONFIG_BRIDGE_EBT_MARK is not set<br>
# CONFIG_BRIDGE_EBT_PKTTYPE is not set<br>
# CONFIG_BRIDGE_EBT_STP is not set<br>
# CONFIG_BRIDGE_EBT_VLAN is not set<br>
# CONFIG_BRIDGE_EBT_ARPREPLY is not set<br>
# CONFIG_BRIDGE_EBT_DNAT is not set<br>
# CONFIG_BRIDGE_EBT_MARK_T is not set<br>
# CONFIG_BRIDGE_EBT_REDIRECT is not set<br>
# CONFIG_BRIDGE_EBT_SNAT is not set<br>
# CONFIG_BRIDGE_EBT_LOG is not set<br>
# CONFIG_BRIDGE_EBT_NFLOG is not set<br>
# CONFIG_IP_DCCP is not set<br>
# CONFIG_IP_SCTP is not set<br>
# CONFIG_RDS is not set<br>
# CONFIG_TIPC is not set<br>
# CONFIG_ATM is not set<br>
CONFIG_L2TP=y<br>
CONFIG_L2TP_DEBUGFS=y<br>
CONFIG_L2TP_V3=y<br>
CONFIG_L2TP_IP=y<br>
CONFIG_L2TP_ETH=y<br>
CONFIG_STP=y<br>
CONFIG_BRIDGE=y<br>
CONFIG_BRIDGE_IGMP_SNOOPING=y<br>
CONFIG_HAVE_NET_DSA=y<br>
# CONFIG_VLAN_8021Q is not set<br>
# CONFIG_DECNET is not set<br>
CONFIG_LLC=y<br>
# CONFIG_LLC2 is not set<br>
# CONFIG_IPX is not set<br>
# CONFIG_ATALK is not set<br>
# CONFIG_X25 is not set<br>
# CONFIG_LAPB is not set<br>
# CONFIG_PHONET is not set<br>
# CONFIG_6LOWPAN is not set<br>
# CONFIG_IEEE802154 is not set<br>
CONFIG_NET_SCHED=y<br>
<br>
#<br>
# Queueing/Scheduling<br>
#<br>
# CONFIG_NET_SCH_CBQ is not set<br>
CONFIG_NET_SCH_HTB=y<br>
# CONFIG_NET_SCH_HFSC is not set<br>
CONFIG_NET_SCH_PRIO=y<br>
# CONFIG_NET_SCH_MULTIQ is not set<br>
# CONFIG_NET_SCH_RED is not set<br>
# CONFIG_NET_SCH_SFB is not set<br>
# CONFIG_NET_SCH_SFQ is not set<br>
# CONFIG_NET_SCH_TEQL is not set<br>
# CONFIG_NET_SCH_TBF is not set<br>
# CONFIG_NET_SCH_GRED is not set<br>
# CONFIG_NET_SCH_DSMARK is not set<br>
# CONFIG_NET_SCH_NETEM is not set<br>
# CONFIG_NET_SCH_DRR is not set<br>
# CONFIG_NET_SCH_MQPRIO is not set<br>
# CONFIG_NET_SCH_CHOKE is not set<br>
# CONFIG_NET_SCH_QFQ is not set<br>
# CONFIG_NET_SCH_CODEL is not set<br>
# CONFIG_NET_SCH_FQ_CODEL is not set<br>
# CONFIG_NET_SCH_FQ is not set<br>
# CONFIG_NET_SCH_HHF is not set<br>
# CONFIG_NET_SCH_PIE is not set<br>
# CONFIG_NET_SCH_INGRESS is not set<br>
# CONFIG_NET_SCH_PLUG is not set<br>
<br>
#<br>
# Classification<br>
#<br>
CONFIG_NET_CLS=y<br>
# CONFIG_NET_CLS_BASIC is not set<br>
# CONFIG_NET_CLS_TCINDEX is not set<br>
# CONFIG_NET_CLS_ROUTE4 is not set<br>
CONFIG_NET_CLS_FW=y<br>
CONFIG_NET_CLS_U32=y<br>
# CONFIG_CLS_U32_PERF is not set<br>
CONFIG_CLS_U32_MARK=y<br>
# CONFIG_NET_CLS_RSVP is not set<br>
# CONFIG_NET_CLS_RSVP6 is not set<br>
CONFIG_NET_CLS_FLOW=y<br>
CONFIG_NET_CLS_CGROUP=y<br>
# CONFIG_NET_CLS_BPF is not set<br>
CONFIG_NET_EMATCH=y<br>
CONFIG_NET_EMATCH_STACK=32<br>
CONFIG_NET_EMATCH_CMP=y<br>
CONFIG_NET_EMATCH_NBYTE=y<br>
CONFIG_NET_EMATCH_U32=y<br>
CONFIG_NET_EMATCH_META=y<br>
CONFIG_NET_EMATCH_TEXT=y<br>
CONFIG_NET_CLS_ACT=y<br>
# CONFIG_NET_ACT_POLICE is not set<br>
# CONFIG_NET_ACT_GACT is not set<br>
# CONFIG_NET_ACT_MIRRED is not set<br>
# CONFIG_NET_ACT_IPT is not set<br>
# CONFIG_NET_ACT_NAT is not set<br>
# CONFIG_NET_ACT_PEDIT is not set<br>
# CONFIG_NET_ACT_SIMP is not set<br>
# CONFIG_NET_ACT_SKBEDIT is not set<br>
# CONFIG_NET_ACT_CSUM is not set<br>
# CONFIG_NET_CLS_IND is not set<br>
CONFIG_NET_SCH_FIFO=y<br>
# CONFIG_DCB is not set<br>
CONFIG_DNS_RESOLVER=y<br>
# CONFIG_BATMAN_ADV is not set<br>
# CONFIG_OPENVSWITCH is not set<br>
# CONFIG_VSOCKETS is not set<br>
# CONFIG_NETLINK_MMAP is not set<br>
# CONFIG_NETLINK_DIAG is not set<br>
# CONFIG_NET_MPLS_GSO is not set<br>
# CONFIG_HSR is not set<br>
CONFIG_RMNET_DATA=y<br>
CONFIG_RMNET_DATA_FC=y<br>
CONFIG_RMNET_DATA_DEBUG_PKT=y<br>
CONFIG_RPS=y<br>
CONFIG_RFS_ACCEL=y<br>
CONFIG_XPS=y<br>
# CONFIG_CGROUP_NET_PRIO is not set<br>
CONFIG_CGROUP_NET_CLASSID=y<br>
CONFIG_NET_RX_BUSY_POLL=y<br>
CONFIG_BQL=y<br>
CONFIG_NET_FLOW_LIMIT=y<br>
CONFIG_SOCKEV_NLMCAST=y<br>
<br>
#<br>
# Network testing<br>
#<br>
# CONFIG_NET_PKTGEN is not set<br>
# CONFIG_NET_DROP_MONITOR is not set<br>
# CONFIG_HAMRADIO is not set<br>
# CONFIG_CAN is not set<br>
# CONFIG_IRDA is not set<br>
CONFIG_BT=y<br>
CONFIG_BT_RFCOMM=y<br>
CONFIG_BT_RFCOMM_TTY=y<br>
CONFIG_BT_BNEP=y<br>
CONFIG_BT_BNEP_MC_FILTER=y<br>
CONFIG_BT_BNEP_PROTO_FILTER=y<br>
CONFIG_BT_HIDP=y<br>
<br>
#<br>
# Bluetooth device drivers<br>
#<br>
# CONFIG_BT_HCIBTUSB is not set<br>
# CONFIG_BT_HCIBTSDIO is not set<br>
CONFIG_BT_HCIUART=y<br>
CONFIG_BT_HCIUART_H4=y<br>
# CONFIG_BT_HCIUART_BCSP is not set<br>
# CONFIG_BT_HCIUART_ATH3K is not set<br>
# CONFIG_BT_HCIUART_LL is not set<br>
# CONFIG_BT_HCIUART_3WIRE is not set<br>
# CONFIG_BT_HCIUART_IBS is not set<br>
# CONFIG_BT_HCIBCM203X is not set<br>
# CONFIG_BT_HCIBPA10X is not set<br>
# CONFIG_BT_HCIBFUSB is not set<br>
# CONFIG_BT_HCIVHCI is not set<br>
# CONFIG_BT_MRVL is not set<br>
CONFIG_MSM_BT_POWER=y<br>
# CONFIG_BTFM_SLIM is not set<br>
# CONFIG_AF_RXRPC is not set<br>
CONFIG_FIB_RULES=y<br>
CONFIG_WIRELESS=y<br>
CONFIG_WIRELESS_EXT=y<br>
CONFIG_WEXT_CORE=y<br>
CONFIG_WEXT_PROC=y<br>
CONFIG_WEXT_SPY=y<br>
CONFIG_WEXT_PRIV=y<br>
CONFIG_CFG80211=y<br>
CONFIG_NL80211_TESTMODE=y<br>
# CONFIG_CFG80211_DEVELOPER_WARNINGS is not set<br>
# CONFIG_CFG80211_REG_DEBUG is not set<br>
# CONFIG_CFG80211_CERTIFICATION_ONUS is not set<br>
CONFIG_CFG80211_DEFAULT_PS=y<br>
# CONFIG_CFG80211_DEBUGFS is not set<br>
CONFIG_CFG80211_INTERNAL_REGDB=y<br>
# CONFIG_CFG80211_WEXT is not set<br>
# CONFIG_LIB80211 is not set<br>
# CONFIG_MAC80211 is not set<br>
# CONFIG_WIMAX is not set<br>
CONFIG_RFKILL=y<br>
CONFIG_RFKILL_PM=y<br>
CONFIG_RFKILL_LEDS=y<br>
# CONFIG_RFKILL_INPUT is not set<br>
# CONFIG_RFKILL_REGULATOR is not set<br>
# CONFIG_RFKILL_GPIO is not set<br>
# CONFIG_NET_9P is not set<br>
# CONFIG_CAIF is not set<br>
# CONFIG_CEPH_LIB is not set<br>
# CONFIG_NFC is not set<br>
# CONFIG_NFC_NQ is not set<br>
CONFIG_IPC_ROUTER=y<br>
CONFIG_IPC_ROUTER_SECURITY=y<br>
CONFIG_HAVE_BPF_JIT=y<br>
<br>
#<br>
# Device Drivers<br>
#<br>
<br>
#<br>
# Generic Driver Options<br>
#<br>
CONFIG_UEVENT_HELPER=y<br>
CONFIG_UEVENT_HELPER_PATH=""<br>
CONFIG_DEVTMPFS=y<br>
CONFIG_DEVTMPFS_MOUNT=y<br>
CONFIG_STANDALONE=y<br>
CONFIG_PREVENT_FIRMWARE_BUILD=y<br>
CONFIG_FW_LOADER=y<br>
CONFIG_FIRMWARE_IN_KERNEL=y<br>
CONFIG_EXTRA_FIRMWARE=""<br>
CONFIG_FW_LOADER_USER_HELPER=y<br>
CONFIG_FW_LOADER_USER_HELPER_FALLBACK=y<br>
CONFIG_WANT_DEV_COREDUMP=y<br>
CONFIG_ALLOW_DEV_COREDUMP=y<br>
CONFIG_DEV_COREDUMP=y<br>
# CONFIG_DEBUG_DRIVER is not set<br>
# CONFIG_DEBUG_DEVRES is not set<br>
# CONFIG_SYS_HYPERVISOR is not set<br>
# CONFIG_GENERIC_CPU_DEVICES is not set<br>
CONFIG_HAVE_CPU_AUTOPROBE=y<br>
CONFIG_GENERIC_CPU_AUTOPROBE=y<br>
CONFIG_SOC_BUS=y<br>
CONFIG_REGMAP=y<br>
CONFIG_REGMAP_I2C=y<br>
CONFIG_REGMAP_SPI=y<br>
CONFIG_REGMAP_SWR=y<br>
CONFIG_REGMAP_ALLOW_WRITE_DEBUGFS=y<br>
CONFIG_DMA_SHARED_BUFFER=y<br>
# CONFIG_FENCE_TRACE is not set<br>
CONFIG_DMA_CMA=y<br>
<br>
#<br>
# Default contiguous memory area size:<br>
#<br>
CONFIG_CMA_SIZE_MBYTES=16<br>
CONFIG_CMA_SIZE_SEL_MBYTES=y<br>
# CONFIG_CMA_SIZE_SEL_PERCENTAGE is not set<br>
# CONFIG_CMA_SIZE_SEL_MIN is not set<br>
# CONFIG_CMA_SIZE_SEL_MAX is not set<br>
CONFIG_CMA_ALIGNMENT=8<br>
<br>
#<br>
# Bus devices<br>
#<br>
# CONFIG_ARM_CCN is not set<br>
# CONFIG_VEXPRESS_CONFIG is not set<br>
# CONFIG_CONNECTOR is not set<br>
# CONFIG_MTD is not set<br>
CONFIG_DTC=y<br>
CONFIG_OF=y<br>
<br>
#<br>
# Device Tree and Open Firmware support<br>
#<br>
# CONFIG_OF_UNITTEST is not set<br>
CONFIG_OF_FLATTREE=y<br>
CONFIG_OF_EARLY_FLATTREE=y<br>
CONFIG_OF_ADDRESS=y<br>
CONFIG_OF_ADDRESS_PCI=y<br>
CONFIG_OF_IRQ=y<br>
CONFIG_OF_NET=y<br>
CONFIG_OF_MDIO=y<br>
CONFIG_OF_PCI=y<br>
CONFIG_OF_PCI_IRQ=y<br>
CONFIG_OF_SPMI=y<br>
CONFIG_OF_RESERVED_MEM=y<br>
CONFIG_OF_SLIMBUS=y<br>
CONFIG_OF_CORESIGHT=y<br>
CONFIG_OF_BATTERYDATA=y<br>
# CONFIG_OF_OVERLAY is not set<br>
# CONFIG_PARPORT is not set<br>
CONFIG_BLK_DEV=y<br>
# CONFIG_BLK_DEV_NULL_BLK is not set<br>
# CONFIG_BLK_DEV_PCIESSD_MTIP32XX is not set<br>
CONFIG_ZRAM=y<br>
# CONFIG_ZRAM_LZ4_COMPRESS is not set<br>
# CONFIG_ZRAM_DEBUG is not set<br>
# CONFIG_BLK_CPQ_CISS_DA is not set<br>
# CONFIG_BLK_DEV_DAC960 is not set<br>
# CONFIG_BLK_DEV_UMEM is not set<br>
# CONFIG_BLK_DEV_COW_COMMON is not set<br>
CONFIG_BLK_DEV_LOOP=y<br>
CONFIG_BLK_DEV_LOOP_MIN_COUNT=8<br>
# CONFIG_BLK_DEV_CRYPTOLOOP is not set<br>
# CONFIG_BLK_DEV_DRBD is not set<br>
CONFIG_BLK_DEV_NBD=y<br>
# CONFIG_BLK_DEV_NVME is not set<br>
# CONFIG_BLK_DEV_SKD is not set<br>
# CONFIG_BLK_DEV_SX8 is not set<br>
CONFIG_BLK_DEV_RAM=y<br>
CONFIG_BLK_DEV_RAM_COUNT=16<br>
CONFIG_BLK_DEV_RAM_SIZE=8192<br>
# CONFIG_BLK_DEV_XIP is not set<br>
# CONFIG_CDROM_PKTCDVD is not set<br>
# CONFIG_ATA_OVER_ETH is not set<br>
# CONFIG_BLK_DEV_RBD is not set<br>
# CONFIG_BLK_DEV_RSXX is not set<br>
<br>
#<br>
# Misc devices<br>
#<br>
# CONFIG_SENSORS_LIS3LV02D is not set<br>
# CONFIG_AD525X_DPOT is not set<br>
# CONFIG_DUMMY_IRQ is not set<br>
# CONFIG_PHANTOM is not set<br>
# CONFIG_SGI_IOC4 is not set<br>
# CONFIG_TIFM_CORE is not set<br>
# CONFIG_ICS932S401 is not set<br>
# CONFIG_ENCLOSURE_SERVICES is not set<br>
# CONFIG_HP_ILO is not set<br>
# CONFIG_APDS9802ALS is not set<br>
# CONFIG_ISL29003 is not set<br>
# CONFIG_ISL29020 is not set<br>
# CONFIG_SENSORS_TSL2550 is not set<br>
# CONFIG_SENSORS_BH1780 is not set<br>
# CONFIG_SENSORS_BH1770 is not set<br>
# CONFIG_SENSORS_APDS990X is not set<br>
# CONFIG_HMC6352 is not set<br>
# CONFIG_DS1682 is not set<br>
# CONFIG_TI_DAC7512 is not set<br>
CONFIG_UID_STAT=y<br>
# CONFIG_BMP085_I2C is not set<br>
# CONFIG_BMP085_SPI is not set<br>
# CONFIG_USB_SWITCH_FSA9480 is not set<br>
# CONFIG_LATTICE_ECP3_CONFIG is not set<br>
# CONFIG_SRAM is not set<br>
CONFIG_QSEECOM=y<br>
CONFIG_HDCP_QSEECOM=y<br>
CONFIG_UID_CPUTIME=y<br>
CONFIG_USB_EXT_TYPE_C_PERICOM=y<br>
# CONFIG_USB_EXT_TYPE_C_TI is not set<br>
# CONFIG_TI_DRV2667 is not set<br>
# CONFIG_QPNP_MISC is not set<br>
# CONFIG_C2PORT is not set<br>
<br>
#<br>
# EEPROM support<br>
#<br>
# CONFIG_EEPROM_AT24 is not set<br>
# CONFIG_EEPROM_AT25 is not set<br>
# CONFIG_EEPROM_LEGACY is not set<br>
# CONFIG_EEPROM_MAX6875 is not set<br>
# CONFIG_EEPROM_93CX6 is not set<br>
# CONFIG_EEPROM_93XX46 is not set<br>
# CONFIG_CB710_CORE is not set<br>
<br>
#<br>
# Texas Instruments shared transport line discipline<br>
#<br>
# CONFIG_TI_ST is not set<br>
# CONFIG_SENSORS_LIS3_SPI is not set<br>
# CONFIG_SENSORS_LIS3_I2C is not set<br>
<br>
#<br>
# Altera FPGA firmware download module<br>
#<br>
# CONFIG_ALTERA_STAPL is not set<br>
CONFIG_MSM_QDSP6V2_CODECS=y<br>
CONFIG_MSM_ULTRASOUND=y<br>
<br>
#<br>
# Intel MIC Bus Driver<br>
#<br>
<br>
#<br>
# Intel MIC Host Driver<br>
#<br>
<br>
#<br>
# Intel MIC Card Driver<br>
#<br>
# CONFIG_GENWQE is not set<br>
# CONFIG_ECHO is not set<br>
# CONFIG_CXL_BASE is not set<br>
<br>
#<br>
# SCSI device support<br>
#<br>
CONFIG_SCSI_MOD=y<br>
# CONFIG_RAID_ATTRS is not set<br>
CONFIG_SCSI=y<br>
CONFIG_SCSI_DMA=y<br>
# CONFIG_SCSI_NETLINK is not set<br>
# CONFIG_SCSI_MQ_DEFAULT is not set<br>
CONFIG_SCSI_PROC_FS=y<br>
<br>
#<br>
# SCSI support type (disk, tape, CD-ROM)<br>
#<br>
CONFIG_BLK_DEV_SD=y<br>
# CONFIG_CHR_DEV_ST is not set<br>
# CONFIG_CHR_DEV_OSST is not set<br>
# CONFIG_BLK_DEV_SR is not set<br>
CONFIG_CHR_DEV_SG=y<br>
CONFIG_CHR_DEV_SCH=y<br>
CONFIG_SCSI_CONSTANTS=y<br>
CONFIG_SCSI_LOGGING=y<br>
CONFIG_SCSI_SCAN_ASYNC=y<br>
<br>
#<br>
# SCSI Transports<br>
#<br>
# CONFIG_SCSI_SPI_ATTRS is not set<br>
# CONFIG_SCSI_FC_ATTRS is not set<br>
# CONFIG_SCSI_ISCSI_ATTRS is not set<br>
# CONFIG_SCSI_SAS_ATTRS is not set<br>
# CONFIG_SCSI_SAS_LIBSAS is not set<br>
# CONFIG_SCSI_SRP_ATTRS is not set<br>
CONFIG_SCSI_LOWLEVEL=y<br>
# CONFIG_ISCSI_TCP is not set<br>
# CONFIG_ISCSI_BOOT_SYSFS is not set<br>
# CONFIG_SCSI_CXGB3_ISCSI is not set<br>
# CONFIG_SCSI_CXGB4_ISCSI is not set<br>
# CONFIG_SCSI_BNX2_ISCSI is not set<br>
# CONFIG_BE2ISCSI is not set<br>
# CONFIG_BLK_DEV_3W_XXXX_RAID is not set<br>
# CONFIG_SCSI_HPSA is not set<br>
# CONFIG_SCSI_3W_9XXX is not set<br>
# CONFIG_SCSI_3W_SAS is not set<br>
# CONFIG_SCSI_ACARD is not set<br>
# CONFIG_SCSI_AACRAID is not set<br>
# CONFIG_SCSI_AIC7XXX is not set<br>
# CONFIG_SCSI_AIC79XX is not set<br>
# CONFIG_SCSI_AIC94XX is not set<br>
# CONFIG_SCSI_MVSAS is not set<br>
# CONFIG_SCSI_MVUMI is not set<br>
# CONFIG_SCSI_ARCMSR is not set<br>
# CONFIG_SCSI_ESAS2R is not set<br>
# CONFIG_MEGARAID_NEWGEN is not set<br>
# CONFIG_MEGARAID_LEGACY is not set<br>
# CONFIG_MEGARAID_SAS is not set<br>
# CONFIG_SCSI_MPT2SAS is not set<br>
# CONFIG_SCSI_MPT3SAS is not set<br>
CONFIG_SCSI_UFSHCD=y<br>
# CONFIG_SCSI_UFSHCD_PCI is not set<br>
CONFIG_SCSI_UFSHCD_PLATFORM=y<br>
CONFIG_SCSI_UFS_QCOM=y<br>
CONFIG_SCSI_UFS_QCOM_ICE=y<br>
# CONFIG_SCSI_HPTIOP is not set<br>
# CONFIG_SCSI_DMX3191D is not set<br>
# CONFIG_SCSI_EATA_PIO is not set<br>
# CONFIG_SCSI_FUTURE_DOMAIN is not set<br>
# CONFIG_SCSI_IPS is not set<br>
# CONFIG_SCSI_INITIO is not set<br>
# CONFIG_SCSI_INIA100 is not set<br>
# CONFIG_SCSI_STEX is not set<br>
# CONFIG_SCSI_SYM53C8XX_2 is not set<br>
# CONFIG_SCSI_QLOGIC_1280 is not set<br>
# CONFIG_SCSI_QLA_ISCSI is not set<br>
# CONFIG_SCSI_DC395x is not set<br>
# CONFIG_SCSI_DC390T is not set<br>
# CONFIG_SCSI_DEBUG is not set<br>
# CONFIG_SCSI_PMCRAID is not set<br>
# CONFIG_SCSI_PM8001 is not set<br>
# CONFIG_SCSI_LOWLEVEL_PCMCIA is not set<br>
# CONFIG_SCSI_DH is not set<br>
# CONFIG_SCSI_OSD_INITIATOR is not set<br>
CONFIG_HAVE_PATA_PLATFORM=y<br>
# CONFIG_ATA is not set<br>
CONFIG_MD=y<br>
# CONFIG_BLK_DEV_MD is not set<br>
# CONFIG_BCACHE is not set<br>
CONFIG_BLK_DEV_DM_BUILTIN=y<br>
CONFIG_BLK_DEV_DM=y<br>
# CONFIG_DM_DEBUG is not set<br>
CONFIG_DM_BUFIO=y<br>
CONFIG_DM_CRYPT=y<br>
CONFIG_DM_REQ_CRYPT=y<br>
# CONFIG_DM_SNAPSHOT is not set<br>
# CONFIG_DM_THIN_PROVISIONING is not set<br>
# CONFIG_DM_CACHE is not set<br>
# CONFIG_DM_ERA is not set<br>
# CONFIG_DM_MIRROR is not set<br>
# CONFIG_DM_RAID is not set<br>
# CONFIG_DM_ZERO is not set<br>
# CONFIG_DM_MULTIPATH is not set<br>
# CONFIG_DM_DELAY is not set<br>
CONFIG_DM_UEVENT=y<br>
# CONFIG_DM_FLAKEY is not set<br>
CONFIG_DM_VERITY=y<br>
# CONFIG_DM_ANDROID_VERITY is not set<br>
CONFIG_DM_VERITY_FEC=y<br>
# CONFIG_DM_SWITCH is not set<br>
# CONFIG_DM_LOG_WRITES is not set<br>
# CONFIG_TARGET_CORE is not set<br>
# CONFIG_FUSION is not set<br>
<br>
#<br>
# IEEE 1394 (FireWire) support<br>
#<br>
# CONFIG_FIREWIRE is not set<br>
# CONFIG_FIREWIRE_NOSY is not set<br>
# CONFIG_I2O is not set<br>
CONFIG_NETDEVICES=y<br>
CONFIG_MII=y<br>
CONFIG_NET_CORE=y<br>
# CONFIG_BONDING is not set<br>
# CONFIG_DUMMY is not set<br>
# CONFIG_EQUALIZER is not set<br>
# CONFIG_NET_FC is not set<br>
# CONFIG_IFB is not set<br>
# CONFIG_NET_TEAM is not set<br>
# CONFIG_MACVLAN is not set<br>
# CONFIG_VXLAN is not set<br>
# CONFIG_NETCONSOLE is not set<br>
# CONFIG_NETPOLL is not set<br>
# CONFIG_NET_POLL_CONTROLLER is not set<br>
CONFIG_TUN=y<br>
CONFIG_VETH=y<br>
# CONFIG_NLMON is not set<br>
# CONFIG_ARCNET is not set<br>
<br>
#<br>
# CAIF transport drivers<br>
#<br>
<br>
#<br>
# Distributed Switch Architecture drivers<br>
#<br>
# CONFIG_NET_DSA_MV88E6XXX is not set<br>
# CONFIG_NET_DSA_MV88E6060 is not set<br>
# CONFIG_NET_DSA_MV88E6XXX_NEED_PPU is not set<br>
# CONFIG_NET_DSA_MV88E6131 is not set<br>
# CONFIG_NET_DSA_MV88E6123_61_65 is not set<br>
# CONFIG_NET_DSA_MV88E6171 is not set<br>
# CONFIG_NET_DSA_BCM_SF2 is not set<br>
CONFIG_ETHERNET=y<br>
CONFIG_NET_VENDOR_3COM=y<br>
# CONFIG_VORTEX is not set<br>
# CONFIG_TYPHOON is not set<br>
CONFIG_NET_VENDOR_ADAPTEC=y<br>
# CONFIG_ADAPTEC_STARFIRE is not set<br>
CONFIG_NET_VENDOR_AGERE=y<br>
# CONFIG_ET131X is not set<br>
CONFIG_NET_VENDOR_ALTEON=y<br>
# CONFIG_ACENIC is not set<br>
# CONFIG_ALTERA_TSE is not set<br>
CONFIG_NET_VENDOR_AMD=y<br>
# CONFIG_AMD8111_ETH is not set<br>
# CONFIG_PCNET32 is not set<br>
# CONFIG_AMD_XGBE is not set<br>
# CONFIG_NET_XGENE is not set<br>
CONFIG_NET_VENDOR_ARC=y<br>
# CONFIG_ARC_EMAC is not set<br>
# CONFIG_EMAC_ROCKCHIP is not set<br>
CONFIG_NET_VENDOR_ATHEROS=y<br>
# CONFIG_ATL2 is not set<br>
# CONFIG_ATL1 is not set<br>
# CONFIG_ATL1E is not set<br>
# CONFIG_ATL1C is not set<br>
# CONFIG_ALX is not set<br>
CONFIG_NET_VENDOR_BROADCOM=y<br>
# CONFIG_B44 is not set<br>
# CONFIG_BCMGENET is not set<br>
# CONFIG_BNX2 is not set<br>
# CONFIG_CNIC is not set<br>
# CONFIG_TIGON3 is not set<br>
# CONFIG_BNX2X is not set<br>
# CONFIG_SYSTEMPORT is not set<br>
CONFIG_NET_VENDOR_BROCADE=y<br>
# CONFIG_BNA is not set<br>
CONFIG_NET_VENDOR_CHELSIO=y<br>
# CONFIG_CHELSIO_T1 is not set<br>
# CONFIG_CHELSIO_T3 is not set<br>
# CONFIG_CHELSIO_T4 is not set<br>
# CONFIG_CHELSIO_T4VF is not set<br>
CONFIG_NET_VENDOR_CISCO=y<br>
# CONFIG_ENIC is not set<br>
# CONFIG_DNET is not set<br>
CONFIG_NET_VENDOR_DEC=y<br>
# CONFIG_NET_TULIP is not set<br>
CONFIG_NET_VENDOR_DLINK=y<br>
# CONFIG_DL2K is not set<br>
# CONFIG_SUNDANCE is not set<br>
CONFIG_NET_VENDOR_EMULEX=y<br>
# CONFIG_BE2NET is not set<br>
CONFIG_NET_VENDOR_EXAR=y<br>
# CONFIG_S2IO is not set<br>
# CONFIG_VXGE is not set<br>
CONFIG_NET_VENDOR_HP=y<br>
# CONFIG_HP100 is not set<br>
CONFIG_NET_VENDOR_INTEL=y<br>
# CONFIG_E100 is not set<br>
# CONFIG_E1000 is not set<br>
# CONFIG_E1000E is not set<br>
# CONFIG_IGB is not set<br>
# CONFIG_IGBVF is not set<br>
# CONFIG_IXGB is not set<br>
# CONFIG_IXGBE is not set<br>
# CONFIG_IXGBEVF is not set<br>
# CONFIG_I40E is not set<br>
# CONFIG_I40EVF is not set<br>
# CONFIG_FM10K is not set<br>
CONFIG_NET_VENDOR_I825XX=y<br>
# CONFIG_IP1000 is not set<br>
# CONFIG_JME is not set<br>
CONFIG_NET_VENDOR_MARVELL=y<br>
# CONFIG_MVMDIO is not set<br>
# CONFIG_SKGE is not set<br>
# CONFIG_SKY2 is not set<br>
CONFIG_NET_VENDOR_MELLANOX=y<br>
# CONFIG_MLX4_EN is not set<br>
# CONFIG_MLX4_CORE is not set<br>
# CONFIG_MLX5_CORE is not set<br>
CONFIG_NET_VENDOR_MICREL=y<br>
# CONFIG_KS8842 is not set<br>
# CONFIG_KS8851 is not set<br>
# CONFIG_KS8851_MLL is not set<br>
# CONFIG_KSZ884X_PCI is not set<br>
CONFIG_NET_VENDOR_MICROCHIP=y<br>
# CONFIG_ENC28J60 is not set<br>
# CONFIG_ECM_IPA is not set<br>
CONFIG_RNDIS_IPA=y<br>
# CONFIG_MSM_RMNET_BAM is not set<br>
CONFIG_NET_VENDOR_MYRI=y<br>
# CONFIG_MYRI10GE is not set<br>
# CONFIG_FEALNX is not set<br>
CONFIG_NET_VENDOR_NATSEMI=y<br>
# CONFIG_NATSEMI is not set<br>
# CONFIG_NS83820 is not set<br>
CONFIG_NET_VENDOR_8390=y<br>
# CONFIG_NE2K_PCI is not set<br>
CONFIG_NET_VENDOR_NVIDIA=y<br>
# CONFIG_FORCEDETH is not set<br>
CONFIG_NET_VENDOR_OKI=y<br>
# CONFIG_ETHOC is not set<br>
CONFIG_NET_PACKET_ENGINE=y<br>
# CONFIG_HAMACHI is not set<br>
# CONFIG_YELLOWFIN is not set<br>
CONFIG_NET_VENDOR_QLOGIC=y<br>
# CONFIG_QLA3XXX is not set<br>
# CONFIG_QLCNIC is not set<br>
# CONFIG_QLGE is not set<br>
# CONFIG_NETXEN_NIC is not set<br>
CONFIG_NET_VENDOR_QUALCOMM=y<br>
# CONFIG_QCA7000 is not set<br>
# CONFIG_QCOM_EMAC is not set<br>
CONFIG_NET_VENDOR_REALTEK=y<br>
# CONFIG_8139CP is not set<br>
# CONFIG_8139TOO is not set<br>
# CONFIG_R8169 is not set<br>
CONFIG_NET_VENDOR_RDC=y<br>
# CONFIG_R6040 is not set<br>
CONFIG_NET_VENDOR_SAMSUNG=y<br>
# CONFIG_SXGBE_ETH is not set<br>
CONFIG_NET_VENDOR_SEEQ=y<br>
CONFIG_NET_VENDOR_SILAN=y<br>
# CONFIG_SC92031 is not set<br>
CONFIG_NET_VENDOR_SIS=y<br>
# CONFIG_SIS900 is not set<br>
# CONFIG_SIS190 is not set<br>
# CONFIG_SFC is not set<br>
CONFIG_NET_VENDOR_SMSC=y<br>
# CONFIG_SMC91X is not set<br>
# CONFIG_EPIC100 is not set<br>
# CONFIG_SMSC911X is not set<br>
# CONFIG_SMSC9420 is not set<br>
CONFIG_NET_VENDOR_STMICRO=y<br>
# CONFIG_STMMAC_ETH is not set<br>
CONFIG_NET_VENDOR_SUN=y<br>
# CONFIG_HAPPYMEAL is not set<br>
# CONFIG_SUNGEM is not set<br>
# CONFIG_CASSINI is not set<br>
# CONFIG_NIU is not set<br>
CONFIG_NET_VENDOR_TEHUTI=y<br>
# CONFIG_TEHUTI is not set<br>
CONFIG_NET_VENDOR_TI=y<br>
# CONFIG_TLAN is not set<br>
CONFIG_NET_VENDOR_VIA=y<br>
# CONFIG_VIA_RHINE is not set<br>
# CONFIG_VIA_VELOCITY is not set<br>
CONFIG_NET_VENDOR_WIZNET=y<br>
# CONFIG_WIZNET_W5100 is not set<br>
# CONFIG_WIZNET_W5300 is not set<br>
# CONFIG_FDDI is not set<br>
# CONFIG_HIPPI is not set<br>
CONFIG_PHYLIB=y<br>
<br>
#<br>
# MII PHY device drivers<br>
#<br>
# CONFIG_AT803X_PHY is not set<br>
# CONFIG_AMD_PHY is not set<br>
# CONFIG_AMD_XGBE_PHY is not set<br>
# CONFIG_MARVELL_PHY is not set<br>
# CONFIG_DAVICOM_PHY is not set<br>
# CONFIG_QSEMI_PHY is not set<br>
# CONFIG_LXT_PHY is not set<br>
# CONFIG_CICADA_PHY is not set<br>
# CONFIG_VITESSE_PHY is not set<br>
# CONFIG_SMSC_PHY is not set<br>
# CONFIG_BROADCOM_PHY is not set<br>
# CONFIG_BCM7XXX_PHY is not set<br>
# CONFIG_BCM87XX_PHY is not set<br>
# CONFIG_ICPLUS_PHY is not set<br>
# CONFIG_REALTEK_PHY is not set<br>
# CONFIG_NATIONAL_PHY is not set<br>
# CONFIG_STE10XP is not set<br>
# CONFIG_LSI_ET1011C_PHY is not set<br>
# CONFIG_MICREL_PHY is not set<br>
# CONFIG_FIXED_PHY is not set<br>
# CONFIG_MDIO_BITBANG is not set<br>
# CONFIG_MDIO_BUS_MUX_GPIO is not set<br>
# CONFIG_MDIO_BUS_MUX_MMIOREG is not set<br>
# CONFIG_MDIO_BCM_UNIMAC is not set<br>
# CONFIG_MICREL_KS8995MA is not set<br>
CONFIG_PPP=y<br>
CONFIG_PPP_BSDCOMP=y<br>
CONFIG_PPP_DEFLATE=y<br>
CONFIG_PPP_FILTER=y<br>
CONFIG_PPP_MPPE=y<br>
CONFIG_PPP_MULTILINK=y<br>
CONFIG_PPPOE=y<br>
CONFIG_PPPOL2TP=y<br>
CONFIG_PPPOLAC=y<br>
CONFIG_PPPOPNS=y<br>
CONFIG_PPP_ASYNC=y<br>
CONFIG_PPP_SYNC_TTY=y<br>
# CONFIG_SLIP is not set<br>
CONFIG_SLHC=y<br>
CONFIG_USB_NET_DRIVERS=y<br>
# CONFIG_USB_CATC is not set<br>
# CONFIG_USB_KAWETH is not set<br>
# CONFIG_USB_PEGASUS is not set<br>
# CONFIG_USB_RTL8150 is not set<br>
# CONFIG_USB_RTL8152 is not set<br>
CONFIG_USB_USBNET=y<br>
CONFIG_USB_NET_AX8817X=y<br>
CONFIG_USB_NET_AX88179_178A=y<br>
CONFIG_USB_NET_CDCETHER=y<br>
# CONFIG_USB_NET_CDC_EEM is not set<br>
CONFIG_USB_NET_CDC_NCM=y<br>
# CONFIG_USB_NET_HUAWEI_CDC_NCM is not set<br>
# CONFIG_USB_NET_CDC_MBIM is not set<br>
# CONFIG_USB_NET_DM9601 is not set<br>
# CONFIG_USB_NET_SR9700 is not set<br>
# CONFIG_USB_NET_SR9800 is not set<br>
# CONFIG_USB_NET_SMSC75XX is not set<br>
# CONFIG_USB_NET_SMSC95XX is not set<br>
# CONFIG_USB_NET_GL620A is not set<br>
CONFIG_USB_NET_NET1080=y<br>
# CONFIG_USB_NET_PLUSB is not set<br>
# CONFIG_USB_NET_MCS7830 is not set<br>
# CONFIG_USB_NET_RNDIS_HOST is not set<br>
CONFIG_USB_NET_CDC_SUBSET=y<br>
# CONFIG_USB_ALI_M5632 is not set<br>
# CONFIG_USB_AN2720 is not set<br>
CONFIG_USB_BELKIN=y<br>
CONFIG_USB_ARMLINUX=y<br>
# CONFIG_USB_EPSON2888 is not set<br>
# CONFIG_USB_KC2190 is not set<br>
CONFIG_USB_NET_ZAURUS=y<br>
# CONFIG_USB_NET_CX82310_ETH is not set<br>
# CONFIG_USB_NET_KALMIA is not set<br>
# CONFIG_USB_NET_QMI_WWAN is not set<br>
# CONFIG_USB_HSO is not set<br>
# CONFIG_USB_NET_INT51X1 is not set<br>
# CONFIG_USB_IPHETH is not set<br>
# CONFIG_USB_SIERRA_NET is not set<br>
# CONFIG_USB_VL600 is not set<br>
# CONFIG_USBNET_IPA_BRIDGE is not set<br>
CONFIG_WLAN=y<br>
# CONFIG_ATMEL is not set<br>
# CONFIG_PRISM54 is not set<br>
# CONFIG_USB_ZD1201 is not set<br>
# CONFIG_USB_NET_RNDIS_WLAN is not set<br>
# CONFIG_WIFI_CONTROL_FUNC is not set<br>
CONFIG_WCNSS_CORE=y<br>
CONFIG_WCNSS_CORE_PRONTO=y<br>
CONFIG_WCNSS_REGISTER_DUMP_ON_BITE=y<br>
CONFIG_WCNSS_MEM_PRE_ALLOC=y<br>
# CONFIG_WCNSS_SKB_PRE_ALLOC is not set<br>
CONFIG_CNSS_CRYPTO=y<br>
CONFIG_ATH_CARDS=y<br>
# CONFIG_ATH_DEBUG is not set<br>
# CONFIG_ATH5K_PCI is not set<br>
# CONFIG_ATH6KL is not set<br>
CONFIG_WIL6210=y<br>
CONFIG_WIL6210_ISR_COR=y<br>
CONFIG_WIL6210_TRACING=y<br>
# CONFIG_WIL6210_WRITE_IOCTL is not set<br>
CONFIG_WIL6210_PLATFORM_MSM=y<br>
# CONFIG_BRCMFMAC is not set<br>
# CONFIG_HOSTAP is not set<br>
# CONFIG_IPW2100 is not set<br>
# CONFIG_LIBERTAS is not set<br>
# CONFIG_WL_TI is not set<br>
# CONFIG_MWIFIEX is not set<br>
# CONFIG_CNSS is not set<br>
# CONFIG_CLD_DEBUG is not set<br>
# CONFIG_CLD_HL_SDIO_CORE is not set<br>
CONFIG_CLD_LL_CORE=y<br>
# CONFIG_CNSS_LOGGER is not set<br>
# CONFIG_WLAN_FEATURE_RX_WAKELOCK is not set<br>
<br>
#<br>
# Enable WiMAX (Networking options) to see the WiMAX drivers<br>
#<br>
# CONFIG_WAN is not set<br>
# CONFIG_VMXNET3 is not set<br>
# CONFIG_ISDN is not set<br>
<br>
#<br>
# Input device support<br>
#<br>
CONFIG_INPUT=y<br>
# CONFIG_INPUT_FF_MEMLESS is not set<br>
# CONFIG_INPUT_POLLDEV is not set<br>
# CONFIG_INPUT_SPARSEKMAP is not set<br>
# CONFIG_INPUT_MATRIXKMAP is not set<br>
<br>
#<br>
# Userland interfaces<br>
#<br>
CONFIG_INPUT_MOUSEDEV=y<br>
CONFIG_INPUT_MOUSEDEV_PSAUX=y<br>
CONFIG_INPUT_MOUSEDEV_SCREEN_X=1024<br>
CONFIG_INPUT_MOUSEDEV_SCREEN_Y=768<br>
# CONFIG_INPUT_JOYDEV is not set<br>
CONFIG_INPUT_EVDEV=y<br>
# CONFIG_INPUT_EVBUG is not set<br>
CONFIG_INPUT_KEYRESET=y<br>
CONFIG_INPUT_KEYCOMBO=y<br>
<br>
#<br>
# Input Device Drivers<br>
#<br>
CONFIG_INPUT_KEYBOARD=y<br>
# CONFIG_KEYBOARD_ADP5588 is not set<br>
# CONFIG_KEYBOARD_ADP5589 is not set<br>
CONFIG_KEYBOARD_ATKBD=y<br>
# CONFIG_KEYBOARD_QT1070 is not set<br>
# CONFIG_KEYBOARD_QT2160 is not set<br>
# CONFIG_KEYBOARD_LKKBD is not set<br>
CONFIG_KEYBOARD_GPIO=y<br>
# CONFIG_KEYBOARD_GPIO_POLLED is not set<br>
# CONFIG_KEYBOARD_TCA6416 is not set<br>
# CONFIG_KEYBOARD_TCA8418 is not set<br>
# CONFIG_KEYBOARD_MATRIX is not set<br>
# CONFIG_KEYBOARD_LM8323 is not set<br>
# CONFIG_KEYBOARD_LM8333 is not set<br>
# CONFIG_KEYBOARD_MAX7359 is not set<br>
# CONFIG_KEYBOARD_MCS is not set<br>
# CONFIG_KEYBOARD_MPR121 is not set<br>
# CONFIG_KEYBOARD_NEWTON is not set<br>
# CONFIG_KEYBOARD_OPENCORES is not set<br>
# CONFIG_KEYBOARD_SAMSUNG is not set<br>
# CONFIG_KEYBOARD_STOWAWAY is not set<br>
# CONFIG_KEYBOARD_SUNKBD is not set<br>
# CONFIG_KEYBOARD_OMAP4 is not set<br>
# CONFIG_KEYBOARD_XTKBD is not set<br>
# CONFIG_KEYBOARD_CAP1106 is not set<br>
# CONFIG_INPUT_MOUSE is not set<br>
CONFIG_INPUT_JOYSTICK=y<br>
# CONFIG_JOYSTICK_ANALOG is not set<br>
# CONFIG_JOYSTICK_A3D is not set<br>
# CONFIG_JOYSTICK_ADI is not set<br>
# CONFIG_JOYSTICK_COBRA is not set<br>
# CONFIG_JOYSTICK_GF2K is not set<br>
# CONFIG_JOYSTICK_GRIP is not set<br>
# CONFIG_JOYSTICK_GRIP_MP is not set<br>
# CONFIG_JOYSTICK_GUILLEMOT is not set<br>
# CONFIG_JOYSTICK_INTERACT is not set<br>
# CONFIG_JOYSTICK_SIDEWINDER is not set<br>
# CONFIG_JOYSTICK_TMDC is not set<br>
# CONFIG_JOYSTICK_IFORCE is not set<br>
# CONFIG_JOYSTICK_WARRIOR is not set<br>
# CONFIG_JOYSTICK_MAGELLAN is not set<br>
# CONFIG_JOYSTICK_SPACEORB is not set<br>
# CONFIG_JOYSTICK_SPACEBALL is not set<br>
# CONFIG_JOYSTICK_STINGER is not set<br>
# CONFIG_JOYSTICK_TWIDJOY is not set<br>
# CONFIG_JOYSTICK_ZHENHUA is not set<br>
# CONFIG_JOYSTICK_AS5011 is not set<br>
# CONFIG_JOYSTICK_JOYDUMP is not set<br>
CONFIG_JOYSTICK_XPAD=y<br>
# CONFIG_JOYSTICK_XPAD_FF is not set<br>
# CONFIG_JOYSTICK_XPAD_LEDS is not set<br>
CONFIG_INPUT_TABLET=y<br>
# CONFIG_TABLET_USB_ACECAD is not set<br>
# CONFIG_TABLET_USB_AIPTEK is not set<br>
# CONFIG_TABLET_USB_GTCO is not set<br>
# CONFIG_TABLET_USB_HANWANG is not set<br>
# CONFIG_TABLET_USB_KBTAB is not set<br>
# CONFIG_TABLET_SERIAL_WACOM4 is not set<br>
CONFIG_INPUT_TOUCHSCREEN=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_v21=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_I2C_v21=y<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_SPI_v21 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_CORE_v21 is not set<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_v26=y<br>
CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_I2C_v26=y<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_SPI_v26 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_RMI_HID_I2C_v26 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_DSX_CORE_v26 is not set<br>
# CONFIG_SECURE_TOUCH_SYNAPTICS_DSX_V26 is not set<br>
CONFIG_OF_TOUCHSCREEN=y<br>
# CONFIG_TOUCHSCREEN_ADS7846 is not set<br>
# CONFIG_TOUCHSCREEN_AD7877 is not set<br>
# CONFIG_TOUCHSCREEN_AD7879 is not set<br>
# CONFIG_TOUCHSCREEN_AR1021_I2C is not set<br>
# CONFIG_TOUCHSCREEN_ATMEL_MXT is not set<br>
# CONFIG_TOUCHSCREEN_ATMEL_MAXTOUCH_TS is not set<br>
# CONFIG_TOUCHSCREEN_AUO_PIXCIR is not set<br>
# CONFIG_TOUCHSCREEN_BU21013 is not set<br>
# CONFIG_TOUCHSCREEN_CY8CTMG110 is not set<br>
# CONFIG_TOUCHSCREEN_CYTTSP_CORE is not set<br>
# CONFIG_TOUCHSCREEN_CYTTSP4_CORE is not set<br>
# CONFIG_TOUCHSCREEN_DYNAPRO is not set<br>
# CONFIG_TOUCHSCREEN_HAMPSHIRE is not set<br>
# CONFIG_TOUCHSCREEN_EETI is not set<br>
# CONFIG_TOUCHSCREEN_EGALAX is not set<br>
# CONFIG_TOUCHSCREEN_FUJITSU is not set<br>
# CONFIG_TOUCHSCREEN_ILI210X is not set<br>
# CONFIG_TOUCHSCREEN_GUNZE is not set<br>
# CONFIG_TOUCHSCREEN_ELO is not set<br>
# CONFIG_TOUCHSCREEN_WACOM_W8001 is not set<br>
# CONFIG_TOUCHSCREEN_WACOM_I2C is not set<br>
# CONFIG_TOUCHSCREEN_MAX11801 is not set<br>
# CONFIG_TOUCHSCREEN_MCS5000 is not set<br>
# CONFIG_TOUCHSCREEN_MMS114 is not set<br>
# CONFIG_TOUCHSCREEN_MTOUCH is not set<br>
# CONFIG_TOUCHSCREEN_INEXIO is not set<br>
# CONFIG_TOUCHSCREEN_MK712 is not set<br>
# CONFIG_TOUCHSCREEN_PENMOUNT is not set<br>
# CONFIG_TOUCHSCREEN_EDT_FT5X06 is not set<br>
CONFIG_TOUCHSCREEN_FT5435=y<br>
CONFIG_TOUCHSCREEN_IST3038C=y<br>
# CONFIG_TOUCHSCREEN_TOUCHRIGHT is not set<br>
# CONFIG_TOUCHSCREEN_TOUCHWIN is not set<br>
# CONFIG_TOUCHSCREEN_PIXCIR is not set<br>
# CONFIG_TOUCHSCREEN_USB_COMPOSITE is not set<br>
# CONFIG_TOUCHSCREEN_TOUCHIT213 is not set<br>
# CONFIG_TOUCHSCREEN_TSC_SERIO is not set<br>
# CONFIG_TOUCHSCREEN_TSC2005 is not set<br>
# CONFIG_TOUCHSCREEN_TSC2007 is not set<br>
# CONFIG_TOUCHSCREEN_ST1232 is not set<br>
# CONFIG_TOUCHSCREEN_SUR40 is not set<br>
# CONFIG_TOUCHSCREEN_TPS6507X is not set<br>
# CONFIG_TOUCHSCREEN_ZFORCE is not set<br>
# CONFIG_SECURE_TOUCH is not set<br>
# CONFIG_TOUCHSCREEN_IT7260_I2C is not set<br>
# CONFIG_TOUCHSCREEN_GEN_VKEYS is not set<br>
# CONFIG_TOUCHSCREEN_FT5X06 is not set<br>
# CONFIG_TOUCHSCREEN_SYNAPTICS_I2C_RMI4 is not set<br>
# CONFIG_TOUCHSCREEN_GT9XX is not set<br>
CONFIG_TOUCHSCREEN_GT9XX_MIDO=y<br>
# CONFIG_TOUCHSCREEN_MAXIM_STI is not set<br>
CONFIG_INPUT_MISC=y<br>
# CONFIG_INPUT_AD714X is not set<br>
# CONFIG_INPUT_BMA150 is not set<br>
CONFIG_INPUT_HBTP_INPUT=y<br>
# CONFIG_HBTP_INPUT_SECURE_TOUCH is not set<br>
# CONFIG_INPUT_MMA8450 is not set<br>
# CONFIG_INPUT_MPU3050 is not set<br>
# CONFIG_INPUT_GP2A is not set<br>
# CONFIG_INPUT_GPIO_BEEPER is not set<br>
# CONFIG_INPUT_GPIO_TILT_POLLED is not set<br>
# CONFIG_INPUT_ATI_REMOTE2 is not set<br>
CONFIG_INPUT_KEYCHORD=y<br>
# CONFIG_INPUT_KEYSPAN_REMOTE is not set<br>
# CONFIG_INPUT_KXTJ9 is not set<br>
# CONFIG_INPUT_POWERMATE is not set<br>
# CONFIG_INPUT_YEALINK is not set<br>
# CONFIG_INPUT_CM109 is not set<br>
CONFIG_INPUT_UINPUT=y<br>
CONFIG_INPUT_GPIO=y<br>
# CONFIG_INPUT_PCF8574 is not set<br>
# CONFIG_INPUT_PWM_BEEPER is not set<br>
# CONFIG_INPUT_GPIO_ROTARY_ENCODER is not set<br>
# CONFIG_INPUT_ADXL34X is not set<br>
# CONFIG_INPUT_IMS_PCU is not set<br>
# CONFIG_INPUT_CMA3000 is not set<br>
# CONFIG_INPUT_SOC_BUTTON_ARRAY is not set<br>
# CONFIG_INPUT_DRV260X_HAPTICS is not set<br>
# CONFIG_INPUT_DRV2667_HAPTICS is not set<br>
CONFIG_INPUT_FINGERPRINT=y<br>
CONFIG_FINGERPRINT_GOODIX_GF3208=y<br>
CONFIG_FINGERPRINT_FPC1020=y<br>
<br>
#<br>
# Hardware I/O ports<br>
#<br>
CONFIG_SERIO=y<br>
CONFIG_SERIO_SERPORT=y<br>
# CONFIG_SERIO_AMBAKMI is not set<br>
# CONFIG_SERIO_PCIPS2 is not set<br>
CONFIG_SERIO_LIBPS2=y<br>
# CONFIG_SERIO_RAW is not set<br>
# CONFIG_SERIO_ALTERA_PS2 is not set<br>
# CONFIG_SERIO_PS2MULT is not set<br>
# CONFIG_SERIO_ARC_PS2 is not set<br>
# CONFIG_SERIO_APBPS2 is not set<br>
# CONFIG_GAMEPORT is not set<br>
<br>
#<br>
# Character devices<br>
#<br>
CONFIG_TTY=y<br>
CONFIG_VT=y<br>
CONFIG_CONSOLE_TRANSLATIONS=y<br>
CONFIG_VT_CONSOLE=y<br>
CONFIG_VT_CONSOLE_SLEEP=y<br>
CONFIG_HW_CONSOLE=y<br>
# CONFIG_VT_HW_CONSOLE_BINDING is not set<br>
CONFIG_UNIX98_PTYS=y<br>
# CONFIG_DEVPTS_MULTIPLE_INSTANCES is not set<br>
# CONFIG_LEGACY_PTYS is not set<br>
# CONFIG_SERIAL_NONSTANDARD is not set<br>
# CONFIG_NOZOMI is not set<br>
# CONFIG_N_GSM is not set<br>
# CONFIG_TRACE_SINK is not set<br>
# CONFIG_DEVMEM is not set<br>
# CONFIG_DEVKMEM is not set<br>
<br>
#<br>
# Serial drivers<br>
#<br>
# CONFIG_SERIAL_8250 is not set<br>
<br>
#<br>
# Non-8250 serial port support<br>
#<br>
# CONFIG_SERIAL_AMBA_PL010 is not set<br>
# CONFIG_SERIAL_AMBA_PL011 is not set<br>
# CONFIG_SERIAL_EARLYCON_ARM_SEMIHOST is not set<br>
# CONFIG_SERIAL_MAX3100 is not set<br>
# CONFIG_SERIAL_MAX310X is not set<br>
# CONFIG_SERIAL_MFD_HSU is not set<br>
CONFIG_SERIAL_CORE=y<br>
# CONFIG_SERIAL_JSM is not set<br>
CONFIG_SERIAL_MSM_HS=y<br>
# CONFIG_SERIAL_MSM_HSL is not set<br>
# CONFIG_SERIAL_SCCNXP is not set<br>
# CONFIG_SERIAL_SC16IS7XX is not set<br>
# CONFIG_SERIAL_ALTERA_JTAGUART is not set<br>
# CONFIG_SERIAL_ALTERA_UART is not set<br>
# CONFIG_SERIAL_IFX6X60 is not set<br>
CONFIG_SERIAL_MSM_SMD=y<br>
# CONFIG_SERIAL_XILINX_PS_UART is not set<br>
# CONFIG_SERIAL_ARC is not set<br>
# CONFIG_SERIAL_RP2 is not set<br>
# CONFIG_SERIAL_FSL_LPUART is not set<br>
<br>
#<br>
# Diag Support<br>
#<br>
CONFIG_DIAG_CHAR=y<br>
<br>
#<br>
# DIAG traffic over USB<br>
#<br>
CONFIG_DIAG_OVER_USB=y<br>
<br>
#<br>
# HSIC/SMUX support for DIAG<br>
#<br>
# CONFIG_TTY_PRINTK is not set<br>
# CONFIG_HVC_DCC is not set<br>
# CONFIG_IPMI_HANDLER is not set<br>
CONFIG_HW_RANDOM=y<br>
# CONFIG_HW_RANDOM_TIMERIOMEM is not set<br>
CONFIG_HW_RANDOM_MSM_LEGACY=y<br>
# CONFIG_R3964 is not set<br>
# CONFIG_APPLICOM is not set<br>
<br>
#<br>
# PCMCIA character devices<br>
#<br>
# CONFIG_RAW_DRIVER is not set<br>
# CONFIG_TCG_TPM is not set<br>
CONFIG_DEVPORT=y<br>
CONFIG_MSM_SMD_PKT=y<br>
# CONFIG_XILLYBUS is not set<br>
CONFIG_MSM_ADSPRPC=y<br>
# CONFIG_MSM_MDSP_TS is not set<br>
# CONFIG_MSM_RDBG is not set<br>
<br>
#<br>
# I2C support<br>
#<br>
CONFIG_I2C=y<br>
CONFIG_I2C_BOARDINFO=y<br>
CONFIG_I2C_COMPAT=y<br>
CONFIG_I2C_CHARDEV=y<br>
CONFIG_I2C_MUX=y<br>
<br>
#<br>
# Multiplexer I2C Chip support<br>
#<br>
# CONFIG_I2C_ARB_GPIO_CHALLENGE is not set<br>
# CONFIG_I2C_MUX_GPIO is not set<br>
# CONFIG_I2C_MUX_PCA9541 is not set<br>
# CONFIG_I2C_MUX_PCA954x is not set<br>
# CONFIG_I2C_MUX_PINCTRL is not set<br>
CONFIG_I2C_HELPER_AUTO=y<br>
<br>
#<br>
# I2C Hardware Bus support<br>
#<br>
<br>
#<br>
# PC SMBus host controller drivers<br>
#<br>
# CONFIG_I2C_ALI1535 is not set<br>
# CONFIG_I2C_ALI1563 is not set<br>
# CONFIG_I2C_ALI15X3 is not set<br>
# CONFIG_I2C_AMD756 is not set<br>
# CONFIG_I2C_AMD8111 is not set<br>
# CONFIG_I2C_I801 is not set<br>
# CONFIG_I2C_ISCH is not set<br>
# CONFIG_I2C_PIIX4 is not set<br>
# CONFIG_I2C_NFORCE2 is not set<br>
# CONFIG_I2C_SIS5595 is not set<br>
# CONFIG_I2C_SIS630 is not set<br>
# CONFIG_I2C_SIS96X is not set<br>
# CONFIG_I2C_VIA is not set<br>
# CONFIG_I2C_VIAPRO is not set<br>
<br>
#<br>
# I2C system bus drivers (mostly embedded / system-on-chip)<br>
#<br>
# CONFIG_I2C_CBUS_GPIO is not set<br>
# CONFIG_I2C_DESIGNWARE_PLATFORM is not set<br>
# CONFIG_I2C_DESIGNWARE_PCI is not set<br>
# CONFIG_I2C_GPIO is not set<br>
# CONFIG_I2C_NOMADIK is not set<br>
# CONFIG_I2C_OCORES is not set<br>
# CONFIG_I2C_PCA_PLATFORM is not set<br>
# CONFIG_I2C_PXA_PCI is not set<br>
# CONFIG_I2C_RK3X is not set<br>
# CONFIG_I2C_SIMTEC is not set<br>
# CONFIG_I2C_XILINX is not set<br>
# CONFIG_I2C_MSM_QUP is not set<br>
CONFIG_I2C_MSM_V2=y<br>
<br>
#<br>
# External I2C/SMBus adapter drivers<br>
#<br>
# CONFIG_I2C_DIOLAN_U2C is not set<br>
# CONFIG_I2C_PARPORT_LIGHT is not set<br>
# CONFIG_I2C_ROBOTFUZZ_OSIF is not set<br>
# CONFIG_I2C_TAOS_EVM is not set<br>
# CONFIG_I2C_TINY_USB is not set<br>
<br>
#<br>
# Other I2C/SMBus bus drivers<br>
#<br>
# CONFIG_I2C_DEBUG_CORE is not set<br>
# CONFIG_I2C_DEBUG_ALGO is not set<br>
# CONFIG_I2C_DEBUG_BUS is not set<br>
CONFIG_SLIMBUS=y<br>
# CONFIG_SLIMBUS_MSM_CTRL is not set<br>
CONFIG_SLIMBUS_MSM_NGD=y<br>
CONFIG_SOUNDWIRE=y<br>
CONFIG_SOUNDWIRE_WCD_CTRL=y<br>
CONFIG_SPI=y<br>
# CONFIG_SPI_DEBUG is not set<br>
CONFIG_SPI_MASTER=y<br>
<br>
#<br>
# SPI Master Controller Drivers<br>
#<br>
# CONFIG_SPI_ALTERA is not set<br>
# CONFIG_SPI_BITBANG is not set<br>
# CONFIG_SPI_GPIO is not set<br>
# CONFIG_SPI_FSL_SPI is not set<br>
# CONFIG_SPI_OC_TINY is not set<br>
# CONFIG_SPI_PL022 is not set<br>
# CONFIG_SPI_PXA2XX is not set<br>
# CONFIG_SPI_PXA2XX_PCI is not set<br>
# CONFIG_SPI_ROCKCHIP is not set<br>
CONFIG_SPI_QUP=y<br>
# CONFIG_SPI_SC18IS602 is not set<br>
# CONFIG_SPI_XCOMM is not set<br>
# CONFIG_SPI_XILINX is not set<br>
# CONFIG_SPI_DESIGNWARE is not set<br>
<br>
#<br>
# SPI Protocol Masters<br>
#<br>
CONFIG_SPI_SPIDEV=y<br>
# CONFIG_SPI_TLE62X0 is not set<br>
# CONFIG_SPMI is not set<br>
# CONFIG_HSI is not set<br>
<br>
#<br>
# PPS support<br>
#<br>
# CONFIG_PPS is not set<br>
<br>
#<br>
# PPS generators support<br>
#<br>
<br>
#<br>
# PTP clock support<br>
#<br>
# CONFIG_PTP_1588_CLOCK is not set<br>
<br>
#<br>
# Enable PHYLIB and NETWORK_PHY_TIMESTAMPING to see the additional clocks.<br>
#<br>
CONFIG_PINCTRL=y<br>
<br>
#<br>
# Pin controllers<br>
#<br>
CONFIG_PINMUX=y<br>
CONFIG_PINCONF=y<br>
CONFIG_GENERIC_PINCONF=y<br>
# CONFIG_DEBUG_PINCTRL is not set<br>
# CONFIG_PINCTRL_SINGLE is not set<br>
CONFIG_PINCTRL_MSM=y<br>
# CONFIG_PINCTRL_APQ8064 is not set<br>
# CONFIG_PINCTRL_MDM9607 is not set<br>
# CONFIG_PINCTRL_MDM9640 is not set<br>
# CONFIG_PINCTRL_MDMCALIFORNIUM is not set<br>
# CONFIG_PINCTRL_APQ8084 is not set<br>
# CONFIG_PINCTRL_IPQ8064 is not set<br>
# CONFIG_PINCTRL_MSM8960 is not set<br>
# CONFIG_PINCTRL_MSM8X74 is not set<br>
CONFIG_PINCTRL_MSM8953=y<br>
CONFIG_ARCH_HAVE_CUSTOM_GPIO_H=y<br>
CONFIG_ARCH_WANT_OPTIONAL_GPIOLIB=y<br>
CONFIG_ARCH_REQUIRE_GPIOLIB=y<br>
CONFIG_GPIOLIB=y<br>
CONFIG_GPIO_DEVRES=y<br>
CONFIG_OF_GPIO=y<br>
CONFIG_GPIOLIB_IRQCHIP=y<br>
# CONFIG_DEBUG_GPIO is not set<br>
CONFIG_GPIO_SYSFS=y<br>
<br>
#<br>
# Memory mapped GPIO drivers:<br>
#<br>
# CONFIG_GPIO_GENERIC_PLATFORM is not set<br>
# CONFIG_GPIO_DWAPB is not set<br>
# CONFIG_GPIO_PL061 is not set<br>
CONFIG_GPIO_QPNP_PIN=y<br>
# CONFIG_GPIO_QPNP_PIN_DEBUG is not set<br>
# CONFIG_GPIO_SCH311X is not set<br>
# CONFIG_GPIO_XGENE is not set<br>
# CONFIG_GPIO_VX855 is not set<br>
# CONFIG_GPIO_GRGPIO is not set<br>
<br>
#<br>
# I2C GPIO expanders:<br>
#<br>
# CONFIG_GPIO_MAX7300 is not set<br>
# CONFIG_GPIO_MAX732X is not set<br>
# CONFIG_GPIO_PCA953X is not set<br>
# CONFIG_GPIO_PCF857X is not set<br>
# CONFIG_GPIO_SX150X is not set<br>
# CONFIG_GPIO_ADP5588 is not set<br>
# CONFIG_GPIO_ADNP is not set<br>
<br>
#<br>
# PCI GPIO expanders:<br>
#<br>
# CONFIG_GPIO_BT8XX is not set<br>
# CONFIG_GPIO_AMD8111 is not set<br>
# CONFIG_GPIO_ML_IOH is not set<br>
# CONFIG_GPIO_RDC321X is not set<br>
<br>
#<br>
# SPI GPIO expanders:<br>
#<br>
# CONFIG_GPIO_MAX7301 is not set<br>
# CONFIG_GPIO_MCP23S08 is not set<br>
# CONFIG_GPIO_MC33880 is not set<br>
# CONFIG_GPIO_74X164 is not set<br>
<br>
#<br>
# AC97 GPIO expanders:<br>
#<br>
<br>
#<br>
# LPC GPIO expanders:<br>
#<br>
<br>
#<br>
# MODULbus GPIO expanders:<br>
#<br>
<br>
#<br>
# USB GPIO expanders:<br>
#<br>
# CONFIG_W1 is not set<br>
CONFIG_POWER_SUPPLY=y<br>
# CONFIG_POWER_SUPPLY_DEBUG is not set<br>
# CONFIG_PDA_POWER is not set<br>
# CONFIG_TEST_POWER is not set<br>
# CONFIG_BATTERY_DS2780 is not set<br>
# CONFIG_BATTERY_DS2781 is not set<br>
# CONFIG_BATTERY_DS2782 is not set<br>
# CONFIG_BATTERY_SBS is not set<br>
# CONFIG_BATTERY_BQ27x00 is not set<br>
# CONFIG_BATTERY_MAX17040 is not set<br>
# CONFIG_BATTERY_MAX17042 is not set<br>
# CONFIG_CHARGER_ISP1704 is not set<br>
# CONFIG_CHARGER_MAX8903 is not set<br>
# CONFIG_CHARGER_LP8727 is not set<br>
# CONFIG_CHARGER_GPIO is not set<br>
# CONFIG_CHARGER_MANAGER is not set<br>
# CONFIG_CHARGER_BQ2415X is not set<br>
# CONFIG_CHARGER_BQ24190 is not set<br>
# CONFIG_CHARGER_BQ24735 is not set<br>
# CONFIG_CHARGER_SMB347 is not set<br>
# CONFIG_SMB349_USB_CHARGER is not set<br>
# CONFIG_SMB349_DUAL_CHARGER is not set<br>
CONFIG_SMB1351_USB_CHARGER=y<br>
# CONFIG_SMB350_CHARGER is not set<br>
CONFIG_SMB135X_CHARGER=y<br>
# CONFIG_SMB1360_CHARGER_FG is not set<br>
# CONFIG_SMB358_CHARGER is not set<br>
# CONFIG_SMB23X_CHARGER is not set<br>
# CONFIG_BATTERY_BQ28400 is not set<br>
# CONFIG_QPNP_CHARGER is not set<br>
CONFIG_QPNP_SMBCHARGER=y<br>
# CONFIG_FUELGAUGE_STC3117 is not set<br>
CONFIG_QPNP_FG=y<br>
CONFIG_BATTERY_BCL=y<br>
# CONFIG_QPNP_VM_BMS is not set<br>
# CONFIG_QPNP_BMS is not set<br>
# CONFIG_QPNP_LINEAR_CHARGER is not set<br>
CONFIG_QPNP_TYPEC=y<br>
CONFIG_MSM_BCL_CTL=y<br>
CONFIG_MSM_BCL_PERIPHERAL_CTL=y<br>
CONFIG_POWER_RESET=y<br>
# CONFIG_POWER_RESET_GPIO is not set<br>
# CONFIG_POWER_RESET_GPIO_RESTART is not set<br>
# CONFIG_POWER_RESET_LTC2952 is not set<br>
CONFIG_POWER_RESET_MSM=y<br>
# CONFIG_MSM_DLOAD_MODE is not set<br>
CONFIG_MSM_PRESERVE_MEM=y<br>
# CONFIG_POWER_RESET_XGENE is not set<br>
# CONFIG_POWER_RESET_SYSCON is not set<br>
# CONFIG_POWER_AVS is not set<br>
CONFIG_MSM_PM=y<br>
CONFIG_APSS_CORE_EA=y<br>
CONFIG_MSM_APM=y<br>
CONFIG_MSM_IDLE_STATS=y<br>
CONFIG_MSM_IDLE_STATS_FIRST_BUCKET=62500<br>
CONFIG_MSM_IDLE_STATS_BUCKET_SHIFT=2<br>
CONFIG_MSM_IDLE_STATS_BUCKET_COUNT=10<br>
CONFIG_MSM_SUSPEND_STATS_FIRST_BUCKET=1000000000<br>
CONFIG_HWMON=y<br>
# CONFIG_HWMON_VID is not set<br>
# CONFIG_HWMON_DEBUG_CHIP is not set<br>
<br>
#<br>
# Native drivers<br>
#<br>
# CONFIG_SENSORS_AD7314 is not set<br>
# CONFIG_SENSORS_AD7414 is not set<br>
# CONFIG_SENSORS_AD7418 is not set<br>
# CONFIG_SENSORS_ADM1021 is not set<br>
# CONFIG_SENSORS_ADM1025 is not set<br>
# CONFIG_SENSORS_ADM1026 is not set<br>
# CONFIG_SENSORS_ADM1029 is not set<br>
# CONFIG_SENSORS_ADM1031 is not set<br>
# CONFIG_SENSORS_ADM9240 is not set<br>
# CONFIG_SENSORS_ADT7310 is not set<br>
# CONFIG_SENSORS_ADT7410 is not set<br>
# CONFIG_SENSORS_ADT7411 is not set<br>
# CONFIG_SENSORS_ADT7462 is not set<br>
# CONFIG_SENSORS_ADT7470 is not set<br>
# CONFIG_SENSORS_ADT7475 is not set<br>
# CONFIG_SENSORS_ASC7621 is not set<br>
# CONFIG_SENSORS_ATXP1 is not set<br>
# CONFIG_SENSORS_DS620 is not set<br>
# CONFIG_SENSORS_DS1621 is not set<br>
# CONFIG_SENSORS_I5K_AMB is not set<br>
# CONFIG_SENSORS_F71805F is not set<br>
# CONFIG_SENSORS_F71882FG is not set<br>
# CONFIG_SENSORS_F75375S is not set<br>
# CONFIG_SENSORS_GL518SM is not set<br>
# CONFIG_SENSORS_GL520SM is not set<br>
# CONFIG_SENSORS_G760A is not set<br>
# CONFIG_SENSORS_G762 is not set<br>
# CONFIG_SENSORS_GPIO_FAN is not set<br>
# CONFIG_SENSORS_HIH6130 is not set<br>
# CONFIG_SENSORS_IT87 is not set<br>
# CONFIG_SENSORS_JC42 is not set<br>
# CONFIG_SENSORS_POWR1220 is not set<br>
# CONFIG_SENSORS_LINEAGE is not set<br>
# CONFIG_SENSORS_LTC2945 is not set<br>
# CONFIG_SENSORS_LTC4151 is not set<br>
# CONFIG_SENSORS_LTC4215 is not set<br>
# CONFIG_SENSORS_LTC4222 is not set<br>
# CONFIG_SENSORS_LTC4245 is not set<br>
# CONFIG_SENSORS_LTC4260 is not set<br>
# CONFIG_SENSORS_LTC4261 is not set<br>
# CONFIG_SENSORS_MAX1111 is not set<br>
# CONFIG_SENSORS_MAX16065 is not set<br>
# CONFIG_SENSORS_MAX1619 is not set<br>
# CONFIG_SENSORS_MAX1668 is not set<br>
# CONFIG_SENSORS_MAX197 is not set<br>
# CONFIG_SENSORS_MAX6639 is not set<br>
# CONFIG_SENSORS_MAX6642 is not set<br>
# CONFIG_SENSORS_MAX6650 is not set<br>
# CONFIG_SENSORS_MAX6697 is not set<br>
# CONFIG_SENSORS_HTU21 is not set<br>
# CONFIG_SENSORS_MCP3021 is not set<br>
# CONFIG_SENSORS_ADCXX is not set<br>
# CONFIG_SENSORS_LM63 is not set<br>
# CONFIG_SENSORS_LM70 is not set<br>
# CONFIG_SENSORS_LM73 is not set<br>
# CONFIG_SENSORS_LM75 is not set<br>
# CONFIG_SENSORS_LM77 is not set<br>
# CONFIG_SENSORS_LM78 is not set<br>
# CONFIG_SENSORS_LM80 is not set<br>
# CONFIG_SENSORS_LM83 is not set<br>
# CONFIG_SENSORS_LM85 is not set<br>
# CONFIG_SENSORS_LM87 is not set<br>
# CONFIG_SENSORS_LM90 is not set<br>
# CONFIG_SENSORS_LM92 is not set<br>
# CONFIG_SENSORS_LM93 is not set<br>
# CONFIG_SENSORS_LM95234 is not set<br>
# CONFIG_SENSORS_LM95241 is not set<br>
# CONFIG_SENSORS_LM95245 is not set<br>
# CONFIG_SENSORS_PC87360 is not set<br>
# CONFIG_SENSORS_PC87427 is not set<br>
# CONFIG_SENSORS_NTC_THERMISTOR is not set<br>
# CONFIG_SENSORS_NCT6683 is not set<br>
# CONFIG_SENSORS_NCT6775 is not set<br>
# CONFIG_SENSORS_PCF8591 is not set<br>
CONFIG_SENSORS_EPM_ADC=y<br>
CONFIG_SENSORS_QPNP_ADC_VOLTAGE=y<br>
CONFIG_SENSORS_QPNP_ADC_CURRENT=y<br>
# CONFIG_PMBUS is not set<br>
# CONFIG_SENSORS_PWM_FAN is not set<br>
# CONFIG_SENSORS_SHT15 is not set<br>
# CONFIG_SENSORS_SHT21 is not set<br>
# CONFIG_SENSORS_SHTC1 is not set<br>
# CONFIG_SENSORS_SIS5595 is not set<br>
# CONFIG_SENSORS_DME1737 is not set<br>
# CONFIG_SENSORS_EMC1403 is not set<br>
# CONFIG_SENSORS_EMC2103 is not set<br>
# CONFIG_SENSORS_EMC6W201 is not set<br>
# CONFIG_SENSORS_SMSC47M1 is not set<br>
# CONFIG_SENSORS_SMSC47M192 is not set<br>
# CONFIG_SENSORS_SMSC47B397 is not set<br>
# CONFIG_SENSORS_SCH56XX_COMMON is not set<br>
# CONFIG_SENSORS_SCH5627 is not set<br>
# CONFIG_SENSORS_SCH5636 is not set<br>
# CONFIG_SENSORS_SMM665 is not set<br>
# CONFIG_SENSORS_ADC128D818 is not set<br>
# CONFIG_SENSORS_ADS1015 is not set<br>
# CONFIG_SENSORS_ADS7828 is not set<br>
# CONFIG_SENSORS_ADS7871 is not set<br>
# CONFIG_SENSORS_AMC6821 is not set<br>
# CONFIG_SENSORS_INA209 is not set<br>
# CONFIG_SENSORS_INA2XX is not set<br>
# CONFIG_SENSORS_THMC50 is not set<br>
# CONFIG_SENSORS_TMP102 is not set<br>
# CONFIG_SENSORS_TMP103 is not set<br>
# CONFIG_SENSORS_TMP401 is not set<br>
# CONFIG_SENSORS_TMP421 is not set<br>
# CONFIG_SENSORS_VIA686A is not set<br>
# CONFIG_SENSORS_VT1211 is not set<br>
# CONFIG_SENSORS_VT8231 is not set<br>
# CONFIG_SENSORS_W83781D is not set<br>
# CONFIG_SENSORS_W83791D is not set<br>
# CONFIG_SENSORS_W83792D is not set<br>
# CONFIG_SENSORS_W83793 is not set<br>
# CONFIG_SENSORS_W83795 is not set<br>
# CONFIG_SENSORS_W83L785TS is not set<br>
# CONFIG_SENSORS_W83L786NG is not set<br>
# CONFIG_SENSORS_W83627HF is not set<br>
# CONFIG_SENSORS_W83627EHF is not set<br>
CONFIG_THERMAL=y<br>
CONFIG_THERMAL_HWMON=y<br>
CONFIG_THERMAL_OF=y<br>
CONFIG_THERMAL_WRITABLE_TRIPS=y<br>
CONFIG_THERMAL_DEFAULT_GOV_STEP_WISE=y<br>
# CONFIG_THERMAL_DEFAULT_GOV_FAIR_SHARE is not set<br>
# CONFIG_THERMAL_DEFAULT_GOV_USER_SPACE is not set<br>
# CONFIG_THERMAL_DEFAULT_GOV_POWER_ALLOCATOR is not set<br>
# CONFIG_THERMAL_GOV_FAIR_SHARE is not set<br>
CONFIG_THERMAL_GOV_STEP_WISE=y<br>
# CONFIG_THERMAL_GOV_BANG_BANG is not set<br>
# CONFIG_THERMAL_GOV_USER_SPACE is not set<br>
# CONFIG_THERMAL_GOV_POWER_ALLOCATOR is not set<br>
# CONFIG_CPU_THERMAL is not set<br>
# CONFIG_THERMAL_EMULATION is not set<br>
CONFIG_THERMAL_TSENS8974=y<br>
CONFIG_LIMITS_MONITOR=y<br>
CONFIG_LIMITS_LITE_HW=y<br>
CONFIG_THERMAL_MONITOR=y<br>
CONFIG_THERMAL_QPNP=y<br>
CONFIG_THERMAL_QPNP_ADC_TM=y<br>
<br>
#<br>
# Texas Instruments thermal drivers<br>
#<br>
CONFIG_WATCHDOG=y<br>
# CONFIG_WATCHDOG_CORE is not set<br>
CONFIG_WATCHDOG_NOWAYOUT=y<br>
<br>
#<br>
# Watchdog Device Drivers<br>
#<br>
# CONFIG_SOFT_WATCHDOG is not set<br>
# CONFIG_GPIO_WATCHDOG is not set<br>
# CONFIG_XILINX_WATCHDOG is not set<br>
# CONFIG_ARM_SP805_WATCHDOG is not set<br>
# CONFIG_DW_WATCHDOG is not set<br>
# CONFIG_ALIM7101_WDT is not set<br>
# CONFIG_I6300ESB_WDT is not set<br>
# CONFIG_MEN_A21_WDT is not set<br>
<br>
#<br>
# PCI-based Watchdog Cards<br>
#<br>
# CONFIG_PCIPCWATCHDOG is not set<br>
# CONFIG_WDTPCI is not set<br>
<br>
#<br>
# USB-based Watchdog Cards<br>
#<br>
# CONFIG_USBPCWATCHDOG is not set<br>
CONFIG_SSB_POSSIBLE=y<br>
<br>
#<br>
# Sonics Silicon Backplane<br>
#<br>
# CONFIG_SSB is not set<br>
CONFIG_BCMA_POSSIBLE=y<br>
<br>
#<br>
# Broadcom specific AMBA<br>
#<br>
# CONFIG_BCMA is not set<br>
<br>
#<br>
# Multifunction device drivers<br>
#<br>
CONFIG_MFD_CORE=y<br>
# CONFIG_MFD_AS3711 is not set<br>
# CONFIG_MFD_AS3722 is not set<br>
# CONFIG_PMIC_ADP5520 is not set<br>
# CONFIG_MFD_AAT2870_CORE is not set<br>
# CONFIG_MFD_BCM590XX is not set<br>
# CONFIG_MFD_AXP20X is not set<br>
# CONFIG_MFD_CROS_EC is not set<br>
# CONFIG_PMIC_DA903X is not set<br>
# CONFIG_MFD_DA9052_SPI is not set<br>
# CONFIG_MFD_DA9052_I2C is not set<br>
# CONFIG_MFD_DA9055 is not set<br>
# CONFIG_MFD_DA9063 is not set<br>
# CONFIG_MFD_MC13XXX_SPI is not set<br>
# CONFIG_MFD_MC13XXX_I2C is not set<br>
# CONFIG_MFD_HI6421_PMIC is not set<br>
# CONFIG_HTC_PASIC3 is not set<br>
# CONFIG_HTC_I2CPLD is not set<br>
# CONFIG_LPC_ICH is not set<br>
# CONFIG_LPC_SCH is not set<br>
# CONFIG_INTEL_SOC_PMIC is not set<br>
# CONFIG_MFD_JANZ_CMODIO is not set<br>
# CONFIG_MFD_KEMPLD is not set<br>
# CONFIG_MFD_88PM800 is not set<br>
# CONFIG_MFD_88PM805 is not set<br>
# CONFIG_MFD_88PM860X is not set<br>
# CONFIG_MFD_MAX14577 is not set<br>
# CONFIG_MFD_MAX77686 is not set<br>
# CONFIG_MFD_MAX77693 is not set<br>
# CONFIG_MFD_MAX8907 is not set<br>
# CONFIG_MFD_MAX8925 is not set<br>
# CONFIG_MFD_MAX8997 is not set<br>
# CONFIG_MFD_MAX8998 is not set<br>
# CONFIG_MFD_MENF21BMC is not set<br>
# CONFIG_EZX_PCAP is not set<br>
# CONFIG_MFD_VIPERBOARD is not set<br>
# CONFIG_MFD_RETU is not set<br>
# CONFIG_MFD_PCF50633 is not set<br>
# CONFIG_MFD_I2C_PMIC is not set<br>
# CONFIG_MFD_RDC321X is not set<br>
# CONFIG_MFD_RTSX_PCI is not set<br>
# CONFIG_MFD_RTSX_USB is not set<br>
# CONFIG_MFD_RC5T583 is not set<br>
# CONFIG_MFD_RK808 is not set<br>
# CONFIG_MFD_RN5T618 is not set<br>
# CONFIG_MFD_SEC_CORE is not set<br>
# CONFIG_MFD_SI476X_CORE is not set<br>
# CONFIG_MFD_SM501 is not set<br>
# CONFIG_MFD_SMSC is not set<br>
# CONFIG_ABX500_CORE is not set<br>
# CONFIG_MFD_STMPE is not set<br>
# CONFIG_MFD_SYSCON is not set<br>
# CONFIG_MFD_TI_AM335X_TSCADC is not set<br>
# CONFIG_MFD_LP3943 is not set<br>
# CONFIG_MFD_LP8788 is not set<br>
# CONFIG_MFD_PALMAS is not set<br>
# CONFIG_TPS6105X is not set<br>
# CONFIG_TPS65010 is not set<br>
# CONFIG_TPS6507X is not set<br>
# CONFIG_MFD_TPS65090 is not set<br>
# CONFIG_MFD_TPS65217 is not set<br>
# CONFIG_MFD_TPS65218 is not set<br>
# CONFIG_MFD_TPS6586X is not set<br>
# CONFIG_MFD_TPS65910 is not set<br>
# CONFIG_MFD_TPS65912 is not set<br>
# CONFIG_MFD_TPS65912_I2C is not set<br>
# CONFIG_MFD_TPS65912_SPI is not set<br>
# CONFIG_MFD_TPS80031 is not set<br>
# CONFIG_TWL4030_CORE is not set<br>
# CONFIG_TWL6040_CORE is not set<br>
# CONFIG_MFD_WL1273_CORE is not set<br>
# CONFIG_MFD_LM3533 is not set<br>
# CONFIG_MFD_TC3589X is not set<br>
# CONFIG_MFD_TMIO is not set<br>
# CONFIG_MFD_VX855 is not set<br>
# CONFIG_MFD_ARIZONA_I2C is not set<br>
# CONFIG_MFD_ARIZONA_SPI is not set<br>
# CONFIG_MFD_WM8400 is not set<br>
# CONFIG_MFD_WM831X_I2C is not set<br>
# CONFIG_MFD_WM831X_SPI is not set<br>
# CONFIG_MFD_WM8350_I2C is not set<br>
# CONFIG_MFD_WM8994 is not set<br>
# CONFIG_WCD9306_CODEC is not set<br>
# CONFIG_WCD9320_CODEC is not set<br>
CONFIG_WCD9330_CODEC=y<br>
CONFIG_WCD9335_CODEC=y<br>
CONFIG_REGULATOR=y<br>
# CONFIG_REGULATOR_DEBUG is not set<br>
CONFIG_REGULATOR_FIXED_VOLTAGE=y<br>
# CONFIG_REGULATOR_VIRTUAL_CONSUMER is not set<br>
# CONFIG_REGULATOR_USERSPACE_CONSUMER is not set<br>
# CONFIG_REGULATOR_PROXY_CONSUMER is not set<br>
# CONFIG_REGULATOR_ACT8865 is not set<br>
# CONFIG_REGULATOR_AD5398 is not set<br>
CONFIG_REGULATOR_STUB=y<br>
# CONFIG_REGULATOR_DA9210 is not set<br>
# CONFIG_REGULATOR_DA9211 is not set<br>
CONFIG_REGULATOR_FAN53555=y<br>
CONFIG_REGULATOR_MSM_GFX_LDO=y<br>
# CONFIG_REGULATOR_GPIO is not set<br>
# CONFIG_REGULATOR_ISL9305 is not set<br>
CONFIG_REGULATOR_MEM_ACC=y<br>
# CONFIG_REGULATOR_ISL6271A is not set<br>
# CONFIG_REGULATOR_LP3971 is not set<br>
# CONFIG_REGULATOR_LP3972 is not set<br>
# CONFIG_REGULATOR_LP872X is not set<br>
# CONFIG_REGULATOR_LP8755 is not set<br>
# CONFIG_REGULATOR_LTC3589 is not set<br>
# CONFIG_REGULATOR_MAX1586 is not set<br>
# CONFIG_REGULATOR_MAX8649 is not set<br>
# CONFIG_REGULATOR_MAX8660 is not set<br>
# CONFIG_REGULATOR_MAX8952 is not set<br>
# CONFIG_REGULATOR_MAX8973 is not set<br>
# CONFIG_REGULATOR_PFUZE100 is not set<br>
# CONFIG_REGULATOR_PWM is not set<br>
# CONFIG_REGULATOR_TPS51632 is not set<br>
# CONFIG_REGULATOR_TPS62360 is not set<br>
# CONFIG_REGULATOR_TPS65023 is not set<br>
# CONFIG_REGULATOR_TPS6507X is not set<br>
# CONFIG_REGULATOR_TPS6524X is not set<br>
CONFIG_REGULATOR_RPM_SMD=y<br>
CONFIG_REGULATOR_QPNP=y<br>
CONFIG_REGULATOR_QPNP_LABIBB=y<br>
CONFIG_REGULATOR_SPM=y<br>
CONFIG_REGULATOR_CPR=y<br>
# CONFIG_REGULATOR_CPR2_GFX is not set<br>
CONFIG_REGULATOR_CPR3=y<br>
CONFIG_REGULATOR_CPR3_HMSS=y<br>
CONFIG_REGULATOR_CPR3_MMSS=y<br>
CONFIG_REGULATOR_CPR4_APSS=y<br>
CONFIG_REGULATOR_CPRH_KBSS=y<br>
CONFIG_REGULATOR_KRYO=y<br>
CONFIG_MEDIA_SUPPORT=y<br>
<br>
#<br>
# Multimedia core support<br>
#<br>
CONFIG_MEDIA_CAMERA_SUPPORT=y<br>
# CONFIG_MEDIA_ANALOG_TV_SUPPORT is not set<br>
# CONFIG_MEDIA_DIGITAL_TV_SUPPORT is not set<br>
CONFIG_MEDIA_RADIO_SUPPORT=y<br>
# CONFIG_MEDIA_SDR_SUPPORT is not set<br>
CONFIG_MEDIA_RC_SUPPORT=y<br>
CONFIG_MEDIA_CONTROLLER=y<br>
CONFIG_VIDEO_DEV=y<br>
CONFIG_VIDEO_V4L2_SUBDEV_API=y<br>
CONFIG_VIDEO_V4L2=y<br>
# CONFIG_VIDEO_ADV_DEBUG is not set<br>
# CONFIG_VIDEO_FIXED_MINOR_RANGES is not set<br>
CONFIG_V4L2_MEM2MEM_DEV=y<br>
CONFIG_VIDEOBUF_GEN=y<br>
CONFIG_VIDEOBUF2_CORE=y<br>
CONFIG_VIDEOBUF2_MEMOPS=y<br>
CONFIG_VIDEOBUF2_VMALLOC=y<br>
# CONFIG_TTPCI_EEPROM is not set<br>
<br>
#<br>
# Media drivers<br>
#<br>
CONFIG_RC_CORE=y<br>
CONFIG_RC_MAP=y<br>
CONFIG_RC_DECODERS=y<br>
CONFIG_LIRC=y<br>
CONFIG_IR_LIRC_CODEC=y<br>
CONFIG_IR_NEC_DECODER=y<br>
CONFIG_IR_RC5_DECODER=y<br>
CONFIG_IR_RC6_DECODER=y<br>
CONFIG_IR_JVC_DECODER=y<br>
CONFIG_IR_SONY_DECODER=y<br>
CONFIG_IR_SANYO_DECODER=y<br>
CONFIG_IR_SHARP_DECODER=y<br>
# CONFIG_IR_MCE_KBD_DECODER is not set<br>
CONFIG_IR_XMP_DECODER=y<br>
# CONFIG_IR_DUMP_DECODER is not set<br>
CONFIG_RC_DEVICES=y<br>
# CONFIG_RC_ATI_REMOTE is not set<br>
# CONFIG_IR_HIX5HD2 is not set<br>
# CONFIG_IR_IMON is not set<br>
# CONFIG_IR_MCEUSB is not set<br>
# CONFIG_IR_REDRAT3 is not set<br>
# CONFIG_IR_STREAMZAP is not set<br>
# CONFIG_IR_IGUANA is not set<br>
# CONFIG_IR_TTUSBIR is not set<br>
# CONFIG_IR_IMG is not set<br>
# CONFIG_RC_LOOPBACK is not set<br>
# CONFIG_IR_GPIO_CIR is not set<br>
# CONFIG_IR_GPIO is not set<br>
CONFIG_IR_PWM=y<br>
# CONFIG_MEDIA_USB_SUPPORT is not set<br>
# CONFIG_MEDIA_PCI_SUPPORT is not set<br>
CONFIG_V4L_PLATFORM_DRIVERS=y<br>
# CONFIG_VIDEO_CAFE_CCIC is not set<br>
CONFIG_SOC_CAMERA=y<br>
CONFIG_SOC_CAMERA_PLATFORM=y<br>
# CONFIG_V4L_MEM2MEM_DRIVERS is not set<br>
# CONFIG_V4L_TEST_DRIVERS is not set<br>
CONFIG_MSM_VIDC_V4L2=y<br>
CONFIG_MSM_VIDC_VMEM=y<br>
CONFIG_MSM_VIDC_GOVERNORS=y<br>
<br>
#<br>
# Qualcomm MSM Camera And Video<br>
#<br>
CONFIG_MSM_CAMERA=y<br>
# CONFIG_MSM_CAMERA_DEBUG is not set<br>
# CONFIG_MSM_CAMERA_AUTOMOTIVE is not set<br>
CONFIG_MSMB_CAMERA=y<br>
# CONFIG_MSMB_CAMERA_DEBUG is not set<br>
CONFIG_MSM_CAMERA_SENSOR=y<br>
CONFIG_MSM_CPP=y<br>
CONFIG_MSM_CCI=y<br>
CONFIG_MSM_CSI20_HEADER=y<br>
CONFIG_MSM_CSI22_HEADER=y<br>
CONFIG_MSM_CSI30_HEADER=y<br>
CONFIG_MSM_CSI31_HEADER=y<br>
CONFIG_MSM_CSIPHY=y<br>
CONFIG_MSM_CSID=y<br>
CONFIG_MSM_EEPROM=y<br>
CONFIG_MSM_ISPIF=y<br>
# CONFIG_MSM_ISPIF_V1 is not set<br>
# CONFIG_MSM_ISPIF_V2 is not set<br>
CONFIG_IMX134=y<br>
CONFIG_IMX132=y<br>
CONFIG_OV9724=y<br>
CONFIG_OV5648=y<br>
CONFIG_GC0339=y<br>
CONFIG_OV8825=y<br>
CONFIG_OV8865=y<br>
CONFIG_s5k4e1=y<br>
CONFIG_OV12830=y<br>
CONFIG_MSM_V4L2_VIDEO_OVERLAY_DEVICE=y<br>
CONFIG_MSMB_JPEG=y<br>
CONFIG_MSM_FD=y<br>
# CONFIG_MSM_JPEGDMA is not set<br>
# CONFIG_TSPP is not set<br>
CONFIG_MSM_SDE_ROTATOR=y<br>
<br>
#<br>
# Supported MMC/SDIO adapters<br>
#<br>
CONFIG_RADIO_ADAPTERS=y<br>
# CONFIG_RADIO_SI470X is not set<br>
# CONFIG_RADIO_SI4713 is not set<br>
# CONFIG_USB_MR800 is not set<br>
# CONFIG_USB_DSBR is not set<br>
# CONFIG_RADIO_MAXIRADIO is not set<br>
# CONFIG_RADIO_SHARK is not set<br>
# CONFIG_RADIO_SHARK2 is not set<br>
# CONFIG_USB_KEENE is not set<br>
# CONFIG_USB_RAREMONO is not set<br>
# CONFIG_USB_MA901 is not set<br>
# CONFIG_RADIO_TEA5764 is not set<br>
# CONFIG_RADIO_SAA7706H is not set<br>
# CONFIG_RADIO_TEF6862 is not set<br>
# CONFIG_RADIO_WL1273 is not set<br>
<br>
#<br>
# Texas Instruments WL128x FM driver (ST based)<br>
#<br>
# CONFIG_RADIO_WL128X is not set<br>
CONFIG_RADIO_IRIS=y<br>
CONFIG_RADIO_IRIS_TRANSPORT=y<br>
CONFIG_RADIO_SILABS=y<br>
# CONFIG_CYPRESS_FIRMWARE is not set<br>
<br>
#<br>
# Media ancillary drivers (tuners, sensors, i2c, frontends)<br>
#<br>
CONFIG_MEDIA_SUBDRV_AUTOSELECT=y<br>
CONFIG_VIDEO_IR_I2C=y<br>
<br>
#<br>
# Audio decoders, processors and mixers<br>
#<br>
<br>
#<br>
# RDS decoders<br>
#<br>
<br>
#<br>
# Video decoders<br>
#<br>
<br>
#<br>
# Video and audio decoders<br>
#<br>
<br>
#<br>
# Video encoders<br>
#<br>
<br>
#<br>
# Camera sensor devices<br>
#<br>
<br>
#<br>
# Flash devices<br>
#<br>
<br>
#<br>
# Video improvement chips<br>
#<br>
<br>
#<br>
# Audio/Video compression chips<br>
#<br>
<br>
#<br>
# Miscellaneous helper chips<br>
#<br>
<br>
#<br>
# Sensors used on soc_camera driver<br>
#<br>
<br>
#<br>
# soc_camera sensor drivers<br>
#<br>
# CONFIG_SOC_CAMERA_IMX074 is not set<br>
# CONFIG_SOC_CAMERA_MT9M001 is not set<br>
# CONFIG_SOC_CAMERA_MT9M111 is not set<br>
# CONFIG_SOC_CAMERA_MT9T031 is not set<br>
# CONFIG_SOC_CAMERA_MT9T112 is not set<br>
# CONFIG_SOC_CAMERA_MT9V022 is not set<br>
# CONFIG_SOC_CAMERA_OV2640 is not set<br>
# CONFIG_SOC_CAMERA_OV5642 is not set<br>
# CONFIG_SOC_CAMERA_OV6650 is not set<br>
# CONFIG_SOC_CAMERA_OV772X is not set<br>
# CONFIG_SOC_CAMERA_OV9640 is not set<br>
# CONFIG_SOC_CAMERA_OV9740 is not set<br>
# CONFIG_SOC_CAMERA_RJ54N1 is not set<br>
# CONFIG_SOC_CAMERA_TW9910 is not set<br>
CONFIG_MEDIA_TUNER=y<br>
CONFIG_MEDIA_TUNER_SIMPLE=y<br>
CONFIG_MEDIA_TUNER_TDA8290=y<br>
CONFIG_MEDIA_TUNER_TDA827X=y<br>
CONFIG_MEDIA_TUNER_TDA18271=y<br>
CONFIG_MEDIA_TUNER_TDA9887=y<br>
CONFIG_MEDIA_TUNER_TEA5761=y<br>
CONFIG_MEDIA_TUNER_TEA5767=y<br>
CONFIG_MEDIA_TUNER_MT20XX=y<br>
CONFIG_MEDIA_TUNER_XC2028=y<br>
CONFIG_MEDIA_TUNER_XC5000=y<br>
CONFIG_MEDIA_TUNER_XC4000=y<br>
CONFIG_MEDIA_TUNER_MC44S803=y<br>
<br>
#<br>
# Tools to develop new frontends<br>
#<br>
# CONFIG_DVB_DUMMY_FE is not set<br>
<br>
#<br>
# Graphics support<br>
#<br>
CONFIG_VGA_ARB=y<br>
CONFIG_VGA_ARB_MAX_GPUS=16<br>
CONFIG_MSM_KGSL=y<br>
# CONFIG_MSM_KGSL_CFF_DUMP is not set<br>
CONFIG_MSM_ADRENO_DEFAULT_GOVERNOR="msm-adreno-tz"<br>
CONFIG_MSM_KGSL_IOMMU=y<br>
<br>
#<br>
# Direct Rendering Manager<br>
#<br>
# CONFIG_DRM is not set<br>
<br>
#<br>
# Frame buffer Devices<br>
#<br>
CONFIG_FB=y<br>
# CONFIG_FIRMWARE_EDID is not set<br>
CONFIG_FB_CMDLINE=y<br>
# CONFIG_FB_DDC is not set<br>
# CONFIG_FB_BOOT_VESA_SUPPORT is not set<br>
CONFIG_FB_CFB_FILLRECT=y<br>
CONFIG_FB_CFB_COPYAREA=y<br>
CONFIG_FB_CFB_IMAGEBLIT=y<br>
# CONFIG_FB_CFB_REV_PIXELS_IN_BYTE is not set<br>
# CONFIG_FB_SYS_FILLRECT is not set<br>
# CONFIG_FB_SYS_COPYAREA is not set<br>
# CONFIG_FB_SYS_IMAGEBLIT is not set<br>
# CONFIG_FB_FOREIGN_ENDIAN is not set<br>
# CONFIG_FB_SYS_FOPS is not set<br>
# CONFIG_FB_SVGALIB is not set<br>
# CONFIG_FB_MACMODES is not set<br>
# CONFIG_FB_BACKLIGHT is not set<br>
# CONFIG_FB_MODE_HELPERS is not set<br>
# CONFIG_FB_TILEBLITTING is not set<br>
<br>
#<br>
# Frame buffer hardware drivers<br>
#<br>
# CONFIG_FB_CIRRUS is not set<br>
# CONFIG_FB_PM2 is not set<br>
# CONFIG_FB_ARMCLCD is not set<br>
# CONFIG_FB_CYBER2000 is not set<br>
# CONFIG_FB_ASILIANT is not set<br>
# CONFIG_FB_IMSTT is not set<br>
# CONFIG_FB_OPENCORES is not set<br>
# CONFIG_FB_S1D13XXX is not set<br>
# CONFIG_FB_NVIDIA is not set<br>
# CONFIG_FB_RIVA is not set<br>
# CONFIG_FB_I740 is not set<br>
# CONFIG_FB_MATROX is not set<br>
# CONFIG_FB_RADEON is not set<br>
# CONFIG_FB_ATY128 is not set<br>
# CONFIG_FB_ATY is not set<br>
# CONFIG_FB_S3 is not set<br>
# CONFIG_FB_SAVAGE is not set<br>
# CONFIG_FB_SIS is not set<br>
# CONFIG_FB_NEOMAGIC is not set<br>
# CONFIG_FB_KYRO is not set<br>
# CONFIG_FB_3DFX is not set<br>
# CONFIG_FB_VOODOO1 is not set<br>
# CONFIG_FB_VT8623 is not set<br>
# CONFIG_FB_TRIDENT is not set<br>
# CONFIG_FB_ARK is not set<br>
# CONFIG_FB_PM3 is not set<br>
# CONFIG_FB_CARMINE is not set<br>
# CONFIG_FB_SMSCUFX is not set<br>
# CONFIG_FB_UDL is not set<br>
# CONFIG_FB_VIRTUAL is not set<br>
# CONFIG_FB_METRONOME is not set<br>
# CONFIG_FB_MB862XX is not set<br>
CONFIG_FB_MSM=y<br>
# CONFIG_FB_BROADSHEET is not set<br>
# CONFIG_FB_AUO_K190X is not set<br>
# CONFIG_FB_SIMPLE is not set<br>
# CONFIG_MSM_BA_V4L2 is not set<br>
CONFIG_MSM_DBA=y<br>
CONFIG_MSM_DBA_ADV7533=y<br>
CONFIG_FB_MSM_MDSS_COMMON=y<br>
# CONFIG_FB_MSM_MDP is not set<br>
CONFIG_FB_MSM_MDSS=y<br>
# CONFIG_FB_MSM_MDP_NONE is not set<br>
# CONFIG_FB_MSM_QPIC_ILI_QVGA_PANEL is not set<br>
# CONFIG_FB_MSM_QPIC_PANEL_DETECT is not set<br>
CONFIG_FB_MSM_MDSS_WRITEBACK=y<br>
CONFIG_FB_MSM_MDSS_HDMI_PANEL=y<br>
# CONFIG_FB_MSM_MDSS_HDMI_MHL_SII8334 is not set<br>
# CONFIG_FB_MSM_MDSS_MHL3 is not set<br>
# CONFIG_FB_MSM_MDSS_DSI_CTRL_STATUS is not set<br>
# CONFIG_FB_MSM_MDSS_EDP_PANEL is not set<br>
# CONFIG_FB_MSM_MDSS_MDP3 is not set<br>
CONFIG_FB_MSM_MDSS_XLOG_DEBUG=y<br>
# CONFIG_FB_SSD1307 is not set<br>
# CONFIG_BACKLIGHT_LCD_SUPPORT is not set<br>
# CONFIG_ADF is not set<br>
# CONFIG_VGASTATE is not set<br>
<br>
#<br>
# Console display driver support<br>
#<br>
CONFIG_DUMMY_CONSOLE=y<br>
# CONFIG_FRAMEBUFFER_CONSOLE is not set<br>
# CONFIG_LOGO is not set<br>
CONFIG_SOUND=y<br>
# CONFIG_SOUND_OSS_CORE is not set<br>
CONFIG_SND=y<br>
CONFIG_SND_TIMER=y<br>
CONFIG_SND_PCM=y<br>
CONFIG_SND_HWDEP=y<br>
CONFIG_SND_RAWMIDI=y<br>
CONFIG_SND_COMPRESS_OFFLOAD=y<br>
CONFIG_SND_JACK=y<br>
# CONFIG_SND_SEQUENCER is not set<br>
# CONFIG_SND_MIXER_OSS is not set<br>
# CONFIG_SND_PCM_OSS is not set<br>
# CONFIG_SND_HRTIMER is not set<br>
CONFIG_SND_DYNAMIC_MINORS=y<br>
CONFIG_SND_MAX_CARDS=32<br>
CONFIG_SND_SUPPORT_OLD_API=y<br>
CONFIG_SND_VERBOSE_PROCFS=y<br>
# CONFIG_SND_VERBOSE_PRINTK is not set<br>
# CONFIG_SND_DEBUG is not set<br>
# CONFIG_SND_RAWMIDI_SEQ is not set<br>
# CONFIG_SND_OPL3_LIB_SEQ is not set<br>
# CONFIG_SND_OPL4_LIB_SEQ is not set<br>
# CONFIG_SND_SBAWE_SEQ is not set<br>
# CONFIG_SND_EMU10K1_SEQ is not set<br>
CONFIG_SND_DRIVERS=y<br>
# CONFIG_SND_DUMMY is not set<br>
# CONFIG_SND_ALOOP is not set<br>
# CONFIG_SND_MTPAV is not set<br>
# CONFIG_SND_SERIAL_U16550 is not set<br>
# CONFIG_SND_MPU401 is not set<br>
CONFIG_SND_PCI=y<br>
# CONFIG_SND_AD1889 is not set<br>
# CONFIG_SND_ALS300 is not set<br>
# CONFIG_SND_ALI5451 is not set<br>
# CONFIG_SND_ATIIXP is not set<br>
# CONFIG_SND_ATIIXP_MODEM is not set<br>
# CONFIG_SND_AU8810 is not set<br>
# CONFIG_SND_AU8820 is not set<br>
# CONFIG_SND_AU8830 is not set<br>
# CONFIG_SND_AW2 is not set<br>
# CONFIG_SND_AZT3328 is not set<br>
# CONFIG_SND_BT87X is not set<br>
# CONFIG_SND_CA0106 is not set<br>
# CONFIG_SND_CMIPCI is not set<br>
# CONFIG_SND_OXYGEN is not set<br>
# CONFIG_SND_CS4281 is not set<br>
# CONFIG_SND_CS46XX is not set<br>
# CONFIG_SND_CTXFI is not set<br>
# CONFIG_SND_DARLA20 is not set<br>
# CONFIG_SND_GINA20 is not set<br>
# CONFIG_SND_LAYLA20 is not set<br>
# CONFIG_SND_DARLA24 is not set<br>
# CONFIG_SND_GINA24 is not set<br>
# CONFIG_SND_LAYLA24 is not set<br>
# CONFIG_SND_MONA is not set<br>
# CONFIG_SND_MIA is not set<br>
# CONFIG_SND_ECHO3G is not set<br>
# CONFIG_SND_INDIGO is not set<br>
# CONFIG_SND_INDIGOIO is not set<br>
# CONFIG_SND_INDIGODJ is not set<br>
# CONFIG_SND_INDIGOIOX is not set<br>
# CONFIG_SND_INDIGODJX is not set<br>
# CONFIG_SND_EMU10K1 is not set<br>
# CONFIG_SND_EMU10K1X is not set<br>
# CONFIG_SND_ENS1370 is not set<br>
# CONFIG_SND_ENS1371 is not set<br>
# CONFIG_SND_ES1938 is not set<br>
# CONFIG_SND_ES1968 is not set<br>
# CONFIG_SND_FM801 is not set<br>
# CONFIG_SND_HDSP is not set<br>
# CONFIG_SND_HDSPM is not set<br>
# CONFIG_SND_ICE1712 is not set<br>
# CONFIG_SND_ICE1724 is not set<br>
# CONFIG_SND_INTEL8X0 is not set<br>
# CONFIG_SND_INTEL8X0M is not set<br>
# CONFIG_SND_KORG1212 is not set<br>
# CONFIG_SND_LOLA is not set<br>
# CONFIG_SND_LX6464ES is not set<br>
# CONFIG_SND_MAESTRO3 is not set<br>
# CONFIG_SND_MIXART is not set<br>
# CONFIG_SND_NM256 is not set<br>
# CONFIG_SND_PCXHR is not set<br>
# CONFIG_SND_RIPTIDE is not set<br>
# CONFIG_SND_RME32 is not set<br>
# CONFIG_SND_RME96 is not set<br>
# CONFIG_SND_RME9652 is not set<br>
# CONFIG_SND_SONICVIBES is not set<br>
# CONFIG_SND_TRIDENT is not set<br>
# CONFIG_SND_VIA82XX is not set<br>
# CONFIG_SND_VIA82XX_MODEM is not set<br>
# CONFIG_SND_VIRTUOSO is not set<br>
# CONFIG_SND_VX222 is not set<br>
# CONFIG_SND_YMFPCI is not set<br>
<br>
#<br>
# HD-Audio<br>
#<br>
# CONFIG_SND_HDA_INTEL is not set<br>
CONFIG_SND_SPI=y<br>
CONFIG_SND_USB=y<br>
CONFIG_SND_USB_AUDIO=y<br>
# CONFIG_SND_USB_UA101 is not set<br>
# CONFIG_SND_USB_CAIAQ is not set<br>
# CONFIG_SND_USB_6FIRE is not set<br>
# CONFIG_SND_USB_HIFACE is not set<br>
# CONFIG_SND_BCD2000 is not set<br>
# CONFIG_SND_USB_AUDIO_QMI is not set<br>
CONFIG_SND_SOC=y<br>
# CONFIG_SND_ATMEL_SOC is not set<br>
# CONFIG_SND_DESIGNWARE_I2S is not set<br>
<br>
#<br>
# SoC Audio for Freescale CPUs<br>
#<br>
<br>
#<br>
# Common SoC Audio options for Freescale CPUs:<br>
#<br>
# CONFIG_SND_SOC_FSL_ASRC is not set<br>
# CONFIG_SND_SOC_FSL_SAI is not set<br>
# CONFIG_SND_SOC_FSL_SSI is not set<br>
# CONFIG_SND_SOC_FSL_SPDIF is not set<br>
# CONFIG_SND_SOC_FSL_ESAI is not set<br>
# CONFIG_SND_SOC_IMX_AUDMUX is not set<br>
<br>
#<br>
# MSM SoC Audio support<br>
#<br>
CONFIG_SND_SOC_MSM_HOSTLESS_PCM=y<br>
CONFIG_SND_SOC_MSM_QDSP6V2_INTF=y<br>
CONFIG_SND_SOC_QDSP6V2=y<br>
# CONFIG_SND_SOC_QDSP_DEBUG is not set<br>
CONFIG_DOLBY_DAP=y<br>
CONFIG_DOLBY_DS2=y<br>
CONFIG_DTS_EAGLE=y<br>
CONFIG_DTS_SRS_TM=y<br>
CONFIG_QTI_PP=y<br>
CONFIG_SND_SOC_CPE=y<br>
CONFIG_SND_SOC_MSM8X16=y<br>
CONFIG_SND_SOC_I2C_AND_SPI=y<br>
<br>
#<br>
# CODEC drivers<br>
#<br>
# CONFIG_SND_SOC_ADAU1701 is not set<br>
# CONFIG_SND_SOC_AK4104 is not set<br>
# CONFIG_SND_SOC_AK4554 is not set<br>
# CONFIG_SND_SOC_AK4642 is not set<br>
# CONFIG_SND_SOC_AK5386 is not set<br>
# CONFIG_SND_SOC_ALC5623 is not set<br>
# CONFIG_SND_SOC_CS35L32 is not set<br>
# CONFIG_SND_SOC_CS42L52 is not set<br>
# CONFIG_SND_SOC_CS42L56 is not set<br>
# CONFIG_SND_SOC_CS42L73 is not set<br>
# CONFIG_SND_SOC_CS4265 is not set<br>
# CONFIG_SND_SOC_CS4270 is not set<br>
# CONFIG_SND_SOC_CS4271 is not set<br>
# CONFIG_SND_SOC_CS42XX8_I2C is not set<br>
# CONFIG_SND_SOC_HDMI_CODEC is not set<br>
# CONFIG_SND_SOC_ES8328 is not set<br>
# CONFIG_SND_SOC_PCM1681 is not set<br>
# CONFIG_SND_SOC_PCM1792A is not set<br>
# CONFIG_SND_SOC_PCM512x_I2C is not set<br>
# CONFIG_SND_SOC_PCM512x_SPI is not set<br>
# CONFIG_SND_SOC_SGTL5000 is not set<br>
# CONFIG_SND_SOC_SIRF_AUDIO_CODEC is not set<br>
# CONFIG_SND_SOC_SPDIF is not set<br>
# CONFIG_SND_SOC_SSM2602_SPI is not set<br>
# CONFIG_SND_SOC_SSM2602_I2C is not set<br>
# CONFIG_SND_SOC_SSM4567 is not set<br>
# CONFIG_SND_SOC_STA350 is not set<br>
# CONFIG_SND_SOC_TAS2552 is not set<br>
# CONFIG_SND_SOC_TAS5086 is not set<br>
# CONFIG_SND_SOC_TLV320AIC31XX is not set<br>
# CONFIG_SND_SOC_TLV320AIC3X is not set<br>
CONFIG_SND_SOC_WCD9330=y<br>
CONFIG_SND_SOC_WCD9335=y<br>
CONFIG_SND_SOC_WSA881X_SENSORS=y<br>
CONFIG_SND_SOC_WSA881X=y<br>
CONFIG_SND_SOC_WSA881X_ANALOG=y<br>
CONFIG_SND_SOC_MSM8X16_WCD=y<br>
CONFIG_SND_SOC_WCD9XXX=y<br>
CONFIG_SND_SOC_WCD9XXX_V2=y<br>
CONFIG_SND_SOC_WCD_CPE=y<br>
CONFIG_AUDIO_EXT_CLK=y<br>
CONFIG_SND_SOC_WCD_MBHC=y<br>
# CONFIG_SND_SOC_WM8510 is not set<br>
# CONFIG_SND_SOC_WM8523 is not set<br>
# CONFIG_SND_SOC_WM8580 is not set<br>
# CONFIG_SND_SOC_WM8711 is not set<br>
# CONFIG_SND_SOC_WM8728 is not set<br>
# CONFIG_SND_SOC_WM8731 is not set<br>
# CONFIG_SND_SOC_WM8737 is not set<br>
# CONFIG_SND_SOC_WM8741 is not set<br>
# CONFIG_SND_SOC_WM8750 is not set<br>
# CONFIG_SND_SOC_WM8753 is not set<br>
# CONFIG_SND_SOC_WM8770 is not set<br>
# CONFIG_SND_SOC_WM8776 is not set<br>
# CONFIG_SND_SOC_WM8804 is not set<br>
# CONFIG_SND_SOC_WM8903 is not set<br>
# CONFIG_SND_SOC_WM8962 is not set<br>
# CONFIG_SND_SOC_WM8978 is not set<br>
# CONFIG_SND_SOC_TPA6130A2 is not set<br>
CONFIG_SND_SOC_MSM_STUB=y<br>
CONFIG_SND_SOC_MSM_HDMI_DBA_CODEC_RX=y<br>
# CONFIG_SND_SIMPLE_CARD is not set<br>
# CONFIG_SOUND_PRIME is not set<br>
<br>
#<br>
# HID support<br>
#<br>
CONFIG_HID=y<br>
# CONFIG_HID_BATTERY_STRENGTH is not set<br>
CONFIG_HIDRAW=y<br>
CONFIG_UHID=y<br>
CONFIG_HID_GENERIC=y<br>
<br>
#<br>
# Special HID drivers<br>
#<br>
# CONFIG_HID_A4TECH is not set<br>
# CONFIG_HID_ACRUX is not set<br>
CONFIG_HID_APPLE=y<br>
# CONFIG_HID_APPLEIR is not set<br>
# CONFIG_HID_AUREAL is not set<br>
# CONFIG_HID_BELKIN is not set<br>
# CONFIG_HID_CHERRY is not set<br>
# CONFIG_HID_CHICONY is not set<br>
# CONFIG_HID_PRODIKEYS is not set<br>
# CONFIG_HID_CP2112 is not set<br>
# CONFIG_HID_CYPRESS is not set<br>
# CONFIG_HID_DRAGONRISE is not set<br>
# CONFIG_HID_EMS_FF is not set<br>
CONFIG_HID_ELECOM=y<br>
# CONFIG_HID_ELO is not set<br>
# CONFIG_HID_EZKEY is not set<br>
# CONFIG_HID_HOLTEK is not set<br>
# CONFIG_HID_GT683R is not set<br>
# CONFIG_HID_HUION is not set<br>
# CONFIG_HID_KEYTOUCH is not set<br>
# CONFIG_HID_KYE is not set<br>
# CONFIG_HID_UCLOGIC is not set<br>
# CONFIG_HID_WALTOP is not set<br>
# CONFIG_HID_GYRATION is not set<br>
# CONFIG_HID_ICADE is not set<br>
# CONFIG_HID_TWINHAN is not set<br>
# CONFIG_HID_KENSINGTON is not set<br>
# CONFIG_HID_LCPOWER is not set<br>
# CONFIG_HID_LENOVO is not set<br>
# CONFIG_HID_LOGITECH is not set<br>
CONFIG_HID_MAGICMOUSE=y<br>
CONFIG_HID_MICROSOFT=y<br>
# CONFIG_HID_MONTEREY is not set<br>
CONFIG_HID_MULTITOUCH=y<br>
# CONFIG_HID_NTRIG is not set<br>
# CONFIG_HID_ORTEK is not set<br>
# CONFIG_HID_PANTHERLORD is not set<br>
# CONFIG_HID_PENMOUNT is not set<br>
# CONFIG_HID_PETALYNX is not set<br>
# CONFIG_HID_PICOLCD is not set<br>
# CONFIG_HID_PRIMAX is not set<br>
# CONFIG_HID_ROCCAT is not set<br>
# CONFIG_HID_SAITEK is not set<br>
# CONFIG_HID_SAMSUNG is not set<br>
# CONFIG_HID_SONY is not set<br>
# CONFIG_HID_SPEEDLINK is not set<br>
# CONFIG_HID_STEELSERIES is not set<br>
# CONFIG_HID_SUNPLUS is not set<br>
# CONFIG_HID_RMI is not set<br>
# CONFIG_HID_GREENASIA is not set<br>
# CONFIG_HID_SMARTJOYPLUS is not set<br>
# CONFIG_HID_TIVO is not set<br>
# CONFIG_HID_TOPSEED is not set<br>
# CONFIG_HID_THINGM is not set<br>
# CONFIG_HID_THRUSTMASTER is not set<br>
# CONFIG_HID_WACOM is not set<br>
# CONFIG_HID_WIIMOTE is not set<br>
# CONFIG_HID_XINMO is not set<br>
# CONFIG_HID_ZEROPLUS is not set<br>
# CONFIG_HID_ZYDACRON is not set<br>
# CONFIG_HID_SENSOR_HUB is not set<br>
<br>
#<br>
# USB HID support<br>
#<br>
CONFIG_USB_HID=y<br>
# CONFIG_HID_PID is not set<br>
CONFIG_USB_HIDDEV=y<br>
<br>
#<br>
# I2C HID support<br>
#<br>
# CONFIG_I2C_HID is not set<br>
CONFIG_USB_OHCI_LITTLE_ENDIAN=y<br>
CONFIG_USB_SUPPORT=y<br>
CONFIG_USB_COMMON=y<br>
CONFIG_USB_ARCH_HAS_HCD=y<br>
CONFIG_USB=y<br>
CONFIG_USB_ANNOUNCE_NEW_DEVICES=y<br>
<br>
#<br>
# Miscellaneous USB options<br>
#<br>
CONFIG_USB_DEFAULT_PERSIST=y<br>
# CONFIG_USB_DYNAMIC_MINORS is not set<br>
# CONFIG_USB_OTG is not set<br>
# CONFIG_USB_OTG_WHITELIST is not set<br>
# CONFIG_USB_OTG_BLACKLIST_HUB is not set<br>
# CONFIG_USB_OTG_FSM is not set<br>
CONFIG_USB_MON=y<br>
# CONFIG_USB_WUSB_CBAF is not set<br>
<br>
#<br>
# USB Host Controller Drivers<br>
#<br>
# CONFIG_USB_C67X00_HCD is not set<br>
CONFIG_USB_XHCI_HCD=y<br>
CONFIG_USB_XHCI_PCI=y<br>
CONFIG_USB_XHCI_PLATFORM=y<br>
CONFIG_USB_EHCI_HCD=y<br>
CONFIG_USB_EHCI_ROOT_HUB_TT=y<br>
CONFIG_USB_EHCI_TT_NEWSCHED=y<br>
CONFIG_USB_EHCI_PCI=y<br>
CONFIG_USB_EHCI_MSM=y<br>
CONFIG_USB_EHCI_MSM_HSIC=y<br>
# CONFIG_USB_EHCI_HCD_PLATFORM is not set<br>
# CONFIG_USB_OXU210HP_HCD is not set<br>
# CONFIG_USB_ISP116X_HCD is not set<br>
# CONFIG_USB_ISP1760_HCD is not set<br>
# CONFIG_USB_ISP1362_HCD is not set<br>
# CONFIG_USB_FUSBH200_HCD is not set<br>
# CONFIG_USB_FOTG210_HCD is not set<br>
# CONFIG_USB_MAX3421_HCD is not set<br>
# CONFIG_USB_OHCI_HCD is not set<br>
# CONFIG_USB_UHCI_HCD is not set<br>
# CONFIG_USB_SL811_HCD is not set<br>
# CONFIG_USB_R8A66597_HCD is not set<br>
# CONFIG_USB_HCD_TEST_MODE is not set<br>
<br>
#<br>
# USB Device Class drivers<br>
#<br>
CONFIG_USB_ACM=y<br>
# CONFIG_USB_PRINTER is not set<br>
# CONFIG_USB_WDM is not set<br>
# CONFIG_USB_TMC is not set<br>
<br>
#<br>
# NOTE: USB_STORAGE depends on SCSI but BLK_DEV_SD may<br>
#<br>
<br>
#<br>
# also be needed; see USB_STORAGE Help for more info<br>
#<br>
CONFIG_USB_STORAGE=y<br>
# CONFIG_USB_STORAGE_DEBUG is not set<br>
# CONFIG_USB_STORAGE_REALTEK is not set<br>
CONFIG_USB_STORAGE_DATAFAB=y<br>
CONFIG_USB_STORAGE_FREECOM=y<br>
CONFIG_USB_STORAGE_ISD200=y<br>
CONFIG_USB_STORAGE_USBAT=y<br>
CONFIG_USB_STORAGE_SDDR09=y<br>
CONFIG_USB_STORAGE_SDDR55=y<br>
CONFIG_USB_STORAGE_JUMPSHOT=y<br>
CONFIG_USB_STORAGE_ALAUDA=y<br>
# CONFIG_USB_STORAGE_ONETOUCH is not set<br>
CONFIG_USB_STORAGE_KARMA=y<br>
CONFIG_USB_STORAGE_CYPRESS_ATACB=y<br>
# CONFIG_USB_STORAGE_ENE_UB6250 is not set<br>
# CONFIG_USB_UAS is not set<br>
<br>
#<br>
# USB Imaging devices<br>
#<br>
# CONFIG_USB_MDC800 is not set<br>
# CONFIG_USB_MICROTEK is not set<br>
# CONFIG_USBIP_CORE is not set<br>
# CONFIG_USB_MUSB_HDRC is not set<br>
CONFIG_USB_DWC3=y<br>
# CONFIG_USB_DWC3_HOST is not set<br>
# CONFIG_USB_DWC3_GADGET is not set<br>
CONFIG_USB_DWC3_DUAL_ROLE=y<br>
<br>
#<br>
# Platform Glue Driver Support<br>
#<br>
CONFIG_USB_DWC3_PCI=y<br>
CONFIG_USB_DWC3_MSM=y<br>
<br>
#<br>
# Debugging features<br>
#<br>
# CONFIG_USB_DWC3_DEBUG is not set<br>
# CONFIG_DWC3_HOST_USB3_LPM_ENABLE is not set<br>
# CONFIG_USB_DWC2 is not set<br>
# CONFIG_USB_CHIPIDEA is not set<br>
<br>
#<br>
# USB port drivers<br>
#<br>
CONFIG_USB_SERIAL=y<br>
# CONFIG_USB_SERIAL_CONSOLE is not set<br>
# CONFIG_USB_SERIAL_GENERIC is not set<br>
# CONFIG_USB_SERIAL_SIMPLE is not set<br>
# CONFIG_USB_SERIAL_AIRCABLE is not set<br>
# CONFIG_USB_SERIAL_ARK3116 is not set<br>
# CONFIG_USB_SERIAL_BELKIN is not set<br>
# CONFIG_USB_SERIAL_CH341 is not set<br>
# CONFIG_USB_SERIAL_WHITEHEAT is not set<br>
# CONFIG_USB_SERIAL_DIGI_ACCELEPORT is not set<br>
# CONFIG_USB_SERIAL_CP210X is not set<br>
# CONFIG_USB_SERIAL_CYPRESS_M8 is not set<br>
# CONFIG_USB_SERIAL_EMPEG is not set<br>
# CONFIG_USB_SERIAL_FTDI_SIO is not set<br>
# CONFIG_USB_SERIAL_VISOR is not set<br>
# CONFIG_USB_SERIAL_IPAQ is not set<br>
# CONFIG_USB_SERIAL_IR is not set<br>
# CONFIG_USB_SERIAL_EDGEPORT is not set<br>
# CONFIG_USB_SERIAL_EDGEPORT_TI is not set<br>
# CONFIG_USB_SERIAL_F81232 is not set<br>
# CONFIG_USB_SERIAL_GARMIN is not set<br>
# CONFIG_USB_SERIAL_IPW is not set<br>
# CONFIG_USB_SERIAL_IUU is not set<br>
# CONFIG_USB_SERIAL_KEYSPAN_PDA is not set<br>
# CONFIG_USB_SERIAL_KEYSPAN is not set<br>
# CONFIG_USB_SERIAL_KLSI is not set<br>
# CONFIG_USB_SERIAL_KOBIL_SCT is not set<br>
# CONFIG_USB_SERIAL_MCT_U232 is not set<br>
# CONFIG_USB_SERIAL_METRO is not set<br>
# CONFIG_USB_SERIAL_MOS7720 is not set<br>
# CONFIG_USB_SERIAL_MOS7840 is not set<br>
# CONFIG_USB_SERIAL_MXUPORT is not set<br>
# CONFIG_USB_SERIAL_NAVMAN is not set<br>
# CONFIG_USB_SERIAL_PL2303 is not set<br>
# CONFIG_USB_SERIAL_OTI6858 is not set<br>
# CONFIG_USB_SERIAL_QCAUX is not set<br>
# CONFIG_USB_SERIAL_QUALCOMM is not set<br>
# CONFIG_USB_SERIAL_SPCP8X5 is not set<br>
# CONFIG_USB_SERIAL_SAFE is not set<br>
# CONFIG_USB_SERIAL_SIERRAWIRELESS is not set<br>
# CONFIG_USB_SERIAL_SYMBOL is not set<br>
# CONFIG_USB_SERIAL_TI is not set<br>
# CONFIG_USB_SERIAL_CYBERJACK is not set<br>
# CONFIG_USB_SERIAL_XIRCOM is not set<br>
# CONFIG_USB_SERIAL_OPTION is not set<br>
# CONFIG_USB_SERIAL_OMNINET is not set<br>
# CONFIG_USB_SERIAL_OPTICON is not set<br>
# CONFIG_USB_SERIAL_XSENS_MT is not set<br>
# CONFIG_USB_SERIAL_WISHBONE is not set<br>
# CONFIG_USB_SERIAL_SSU100 is not set<br>
# CONFIG_USB_SERIAL_QT2 is not set<br>
# CONFIG_USB_SERIAL_DEBUG is not set<br>
<br>
#<br>
# USB Miscellaneous drivers<br>
#<br>
# CONFIG_USB_EMI62 is not set<br>
# CONFIG_USB_EMI26 is not set<br>
# CONFIG_USB_ADUTUX is not set<br>
# CONFIG_USB_SEVSEG is not set<br>
# CONFIG_USB_RIO500 is not set<br>
# CONFIG_USB_LEGOTOWER is not set<br>
# CONFIG_USB_LCD is not set<br>
# CONFIG_USB_LED is not set<br>
# CONFIG_USB_CYPRESS_CY7C63 is not set<br>
# CONFIG_USB_CYTHERM is not set<br>
# CONFIG_USB_IDMOUSE is not set<br>
# CONFIG_USB_FTDI_ELAN is not set<br>
# CONFIG_USB_APPLEDISPLAY is not set<br>
# CONFIG_USB_SISUSBVGA is not set<br>
# CONFIG_USB_LD is not set<br>
# CONFIG_USB_TRANCEVIBRATOR is not set<br>
# CONFIG_USB_IOWARRIOR is not set<br>
# CONFIG_USB_TEST is not set<br>
CONFIG_USB_EHSET_TEST_FIXTURE=y<br>
# CONFIG_USB_ISIGHTFW is not set<br>
# CONFIG_USB_YUREX is not set<br>
# CONFIG_USB_EZUSB_FX2 is not set<br>
# CONFIG_USB_HSIC_USB3503 is not set<br>
# CONFIG_USB_LINK_LAYER_TEST is not set<br>
<br>
#<br>
# USB Physical Layer drivers<br>
#<br>
CONFIG_USB_PHY=y<br>
# CONFIG_USB_OTG_WAKELOCK is not set<br>
CONFIG_NOP_USB_XCEIV=y<br>
# CONFIG_USB_GPIO_VBUS is not set<br>
# CONFIG_USB_ISP1301 is not set<br>
CONFIG_USB_MSM_OTG=y<br>
CONFIG_USB_MSM_HSPHY=y<br>
# CONFIG_USB_MSM_SSPHY is not set<br>
CONFIG_USB_MSM_SSPHY_QMP=y<br>
CONFIG_MSM_QUSB_PHY=y<br>
# CONFIG_USB_ULPI is not set<br>
CONFIG_DUAL_ROLE_USB_INTF=y<br>
CONFIG_USB_GADGET=y<br>
# CONFIG_USB_GADGET_DEBUG is not set<br>
CONFIG_USB_GADGET_DEBUG_FILES=y<br>
CONFIG_USB_GADGET_DEBUG_FS=y<br>
CONFIG_USB_GADGET_VBUS_DRAW=500<br>
CONFIG_USB_GADGET_STORAGE_NUM_BUFFERS=2<br>
<br>
#<br>
# USB Peripheral Controller<br>
#<br>
# CONFIG_USB_FOTG210_UDC is not set<br>
# CONFIG_USB_GR_UDC is not set<br>
# CONFIG_USB_R8A66597 is not set<br>
# CONFIG_USB_PXA27X is not set<br>
# CONFIG_USB_MV_UDC is not set<br>
# CONFIG_USB_MV_U3D is not set<br>
# CONFIG_USB_M66592 is not set<br>
# CONFIG_USB_AMD5536UDC is not set<br>
# CONFIG_USB_NET2272 is not set<br>
# CONFIG_USB_NET2280 is not set<br>
# CONFIG_USB_GOKU is not set<br>
# CONFIG_USB_EG20T is not set<br>
# CONFIG_USB_GADGET_XILINX is not set<br>
CONFIG_USB_CI13XXX_MSM=y<br>
# CONFIG_USB_CI13XXX_MSM_HSIC is not set<br>
# CONFIG_USB_DUMMY_HCD is not set<br>
CONFIG_USB_LIBCOMPOSITE=y<br>
CONFIG_USB_F_ACM=y<br>
CONFIG_USB_U_SERIAL=y<br>
CONFIG_USB_F_SERIAL=y<br>
CONFIG_USB_F_NCM=y<br>
CONFIG_USB_F_ECM=y<br>
CONFIG_USB_F_MASS_STORAGE=y<br>
CONFIG_USB_F_FS=y<br>
CONFIG_USB_F_UAC1=y<br>
CONFIG_USB_F_UAC2=y<br>
CONFIG_USB_F_UVC=y<br>
CONFIG_USB_F_AUDIO_SRC=y<br>
# CONFIG_USB_CONFIGFS is not set<br>
CONFIG_USB_G_ANDROID=y<br>
# CONFIG_USB_ANDROID_RNDIS_DWORD_ALIGNED is not set<br>
# CONFIG_USB_ZERO is not set<br>
# CONFIG_USB_AUDIO is not set<br>
# CONFIG_USB_ETH is not set<br>
# CONFIG_USB_G_NCM is not set<br>
# CONFIG_USB_GADGETFS is not set<br>
# CONFIG_USB_FUNCTIONFS is not set<br>
# CONFIG_USB_MASS_STORAGE is not set<br>
# CONFIG_USB_G_SERIAL is not set<br>
# CONFIG_USB_MIDI_GADGET is not set<br>
# CONFIG_USB_G_PRINTER is not set<br>
# CONFIG_USB_CDC_COMPOSITE is not set<br>
# CONFIG_USB_G_ACM_MS is not set<br>
# CONFIG_USB_G_MULTI is not set<br>
# CONFIG_USB_G_HID is not set<br>
# CONFIG_USB_G_DBGP is not set<br>
# CONFIG_USB_G_WEBCAM is not set<br>
# CONFIG_USB_LED_TRIG is not set<br>
# CONFIG_UWB is not set<br>
CONFIG_MMC=y<br>
# CONFIG_MMC_DEBUG is not set<br>
CONFIG_MMC_PERF_PROFILING=y<br>
CONFIG_MMC_CLKGATE=y<br>
# CONFIG_MMC_RING_BUFFER is not set<br>
# CONFIG_MMC_EMBEDDED_SDIO is not set<br>
CONFIG_MMC_PARANOID_SD_INIT=y<br>
<br>
#<br>
# MMC/SD/SDIO Card Drivers<br>
#<br>
CONFIG_MMC_BLOCK=y<br>
CONFIG_MMC_BLOCK_MINORS=32<br>
CONFIG_MMC_BLOCK_BOUNCE=y<br>
# CONFIG_MMC_BLOCK_DEFERRED_RESUME is not set<br>
# CONFIG_SDIO_UART is not set<br>
# CONFIG_MMC_TEST is not set<br>
<br>
#<br>
# MMC/SD/SDIO Host Controller Drivers<br>
#<br>
# CONFIG_MMC_ARMMMCI is not set<br>
CONFIG_MMC_SDHCI=y<br>
# CONFIG_MMC_SDHCI_PCI is not set<br>
CONFIG_MMC_SDHCI_PLTFM=y<br>
# CONFIG_MMC_SDHCI_OF_ARASAN is not set<br>
# CONFIG_MMC_SDHCI_PXAV3 is not set<br>
# CONFIG_MMC_SDHCI_PXAV2 is not set<br>
CONFIG_MMC_SDHCI_MSM=y<br>
CONFIG_MMC_SDHCI_MSM_ICE=y<br>
# CONFIG_MMC_TIFM_SD is not set<br>
# CONFIG_MMC_SPI is not set<br>
# CONFIG_MMC_CB710 is not set<br>
# CONFIG_MMC_VIA_SDMMC is not set<br>
# CONFIG_MMC_VUB300 is not set<br>
# CONFIG_MMC_USHC is not set<br>
# CONFIG_MMC_USDHI6ROL0 is not set<br>
CONFIG_MMC_CQ_HCI=y<br>
# CONFIG_MEMSTICK is not set<br>
CONFIG_NEW_LEDS=y<br>
CONFIG_LEDS_CLASS=y<br>
<br>
#<br>
# LED drivers<br>
#<br>
# CONFIG_LEDS_LM3530 is not set<br>
# CONFIG_LEDS_LM3642 is not set<br>
# CONFIG_LEDS_PCA9532 is not set<br>
# CONFIG_LEDS_GPIO is not set<br>
# CONFIG_LEDS_LP3944 is not set<br>
# CONFIG_LEDS_LP5521 is not set<br>
# CONFIG_LEDS_LP5523 is not set<br>
# CONFIG_LEDS_LP5562 is not set<br>
# CONFIG_LEDS_LP8501 is not set<br>
# CONFIG_LEDS_PCA955X is not set<br>
# CONFIG_LEDS_PCA963X is not set<br>
# CONFIG_LEDS_DAC124S085 is not set<br>
# CONFIG_LEDS_PWM is not set<br>
# CONFIG_LEDS_REGULATOR is not set<br>
# CONFIG_LEDS_BD2802 is not set<br>
# CONFIG_LEDS_INTEL_SS4200 is not set<br>
# CONFIG_LEDS_LT3593 is not set<br>
# CONFIG_LEDS_TCA6507 is not set<br>
# CONFIG_LEDS_LM355x is not set<br>
<br>
#<br>
# LED driver for blink(1) USB RGB LED is under Special HID drivers (HID_THINGM)<br>
#<br>
# CONFIG_LEDS_BLINKM is not set<br>
CONFIG_LEDS_QPNP=y<br>
CONFIG_LEDS_QPNP_FLASH=y<br>
CONFIG_LEDS_QPNP_WLED=y<br>
CONFIG_LEDS_AW2013=y<br>
<br>
#<br>
# LED Triggers<br>
#<br>
CONFIG_LEDS_TRIGGERS=y<br>
# CONFIG_LEDS_TRIGGER_TIMER is not set<br>
# CONFIG_LEDS_TRIGGER_ONESHOT is not set<br>
# CONFIG_LEDS_TRIGGER_HEARTBEAT is not set<br>
# CONFIG_LEDS_TRIGGER_BACKLIGHT is not set<br>
# CONFIG_LEDS_TRIGGER_CPU is not set<br>
# CONFIG_LEDS_TRIGGER_GPIO is not set<br>
# CONFIG_LEDS_TRIGGER_DEFAULT_ON is not set<br>
<br>
#<br>
# iptables trigger is under Netfilter config (LED target)<br>
#<br>
# CONFIG_LEDS_TRIGGER_TRANSIENT is not set<br>
# CONFIG_LEDS_TRIGGER_CAMERA is not set<br>
CONFIG_SWITCH=y<br>
# CONFIG_SWITCH_GPIO is not set<br>
# CONFIG_ACCESSIBILITY is not set<br>
# CONFIG_INFINIBAND is not set<br>
CONFIG_EDAC_SUPPORT=y<br>
CONFIG_EDAC=y<br>
CONFIG_EDAC_LEGACY_SYSFS=y<br>
# CONFIG_EDAC_DEBUG is not set<br>
CONFIG_EDAC_MM_EDAC=y<br>
CONFIG_EDAC_CORTEX_ARM64=y<br>
# CONFIG_EDAC_CORTEX_ARM64_PANIC_ON_CE is not set<br>
CONFIG_EDAC_CORTEX_ARM64_DBE_IRQ_ONLY=y<br>
CONFIG_EDAC_CORTEX_ARM64_PANIC_ON_UE=y<br>
CONFIG_RTC_LIB=y<br>
CONFIG_RTC_CLASS=y<br>
CONFIG_RTC_HCTOSYS=y<br>
CONFIG_RTC_SYSTOHC=y<br>
CONFIG_RTC_HCTOSYS_DEVICE="rtc0"<br>
# CONFIG_RTC_DEBUG is not set<br>
<br>
#<br>
# RTC interfaces<br>
#<br>
CONFIG_RTC_INTF_SYSFS=y<br>
CONFIG_RTC_INTF_PROC=y<br>
CONFIG_RTC_INTF_DEV=y<br>
# CONFIG_RTC_INTF_DEV_UIE_EMUL is not set<br>
# CONFIG_RTC_DRV_TEST is not set<br>
<br>
#<br>
# I2C RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_DS1307 is not set<br>
# CONFIG_RTC_DRV_DS1374 is not set<br>
# CONFIG_RTC_DRV_DS1672 is not set<br>
# CONFIG_RTC_DRV_DS3232 is not set<br>
# CONFIG_RTC_DRV_HYM8563 is not set<br>
# CONFIG_RTC_DRV_MAX6900 is not set<br>
# CONFIG_RTC_DRV_RS5C372 is not set<br>
# CONFIG_RTC_DRV_ISL1208 is not set<br>
# CONFIG_RTC_DRV_ISL12022 is not set<br>
# CONFIG_RTC_DRV_ISL12057 is not set<br>
# CONFIG_RTC_DRV_X1205 is not set<br>
# CONFIG_RTC_DRV_PCF2127 is not set<br>
# CONFIG_RTC_DRV_PCF8523 is not set<br>
# CONFIG_RTC_DRV_PCF8563 is not set<br>
# CONFIG_RTC_DRV_PCF85063 is not set<br>
# CONFIG_RTC_DRV_PCF8583 is not set<br>
# CONFIG_RTC_DRV_M41T80 is not set<br>
# CONFIG_RTC_DRV_BQ32K is not set<br>
# CONFIG_RTC_DRV_S35390A is not set<br>
# CONFIG_RTC_DRV_FM3130 is not set<br>
# CONFIG_RTC_DRV_RX8581 is not set<br>
# CONFIG_RTC_DRV_RX8025 is not set<br>
# CONFIG_RTC_DRV_EM3027 is not set<br>
# CONFIG_RTC_DRV_RV3029C2 is not set<br>
<br>
#<br>
# SPI RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_M41T93 is not set<br>
# CONFIG_RTC_DRV_M41T94 is not set<br>
# CONFIG_RTC_DRV_DS1305 is not set<br>
# CONFIG_RTC_DRV_DS1343 is not set<br>
# CONFIG_RTC_DRV_DS1347 is not set<br>
# CONFIG_RTC_DRV_DS1390 is not set<br>
# CONFIG_RTC_DRV_MAX6902 is not set<br>
# CONFIG_RTC_DRV_R9701 is not set<br>
# CONFIG_RTC_DRV_RS5C348 is not set<br>
# CONFIG_RTC_DRV_DS3234 is not set<br>
# CONFIG_RTC_DRV_PCF2123 is not set<br>
# CONFIG_RTC_DRV_RX4581 is not set<br>
# CONFIG_RTC_DRV_MCP795 is not set<br>
<br>
#<br>
# Platform RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_DS1286 is not set<br>
# CONFIG_RTC_DRV_DS1511 is not set<br>
# CONFIG_RTC_DRV_DS1553 is not set<br>
# CONFIG_RTC_DRV_DS1742 is not set<br>
# CONFIG_RTC_DRV_DS2404 is not set<br>
# CONFIG_RTC_DRV_EFI is not set<br>
# CONFIG_RTC_DRV_STK17TA8 is not set<br>
# CONFIG_RTC_DRV_M48T86 is not set<br>
# CONFIG_RTC_DRV_M48T35 is not set<br>
# CONFIG_RTC_DRV_M48T59 is not set<br>
# CONFIG_RTC_DRV_MSM6242 is not set<br>
# CONFIG_RTC_DRV_BQ4802 is not set<br>
# CONFIG_RTC_DRV_RP5C01 is not set<br>
# CONFIG_RTC_DRV_V3020 is not set<br>
<br>
#<br>
# on-CPU RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_PL030 is not set<br>
# CONFIG_RTC_DRV_PL031 is not set<br>
# CONFIG_RTC_DRV_SNVS is not set<br>
CONFIG_RTC_DRV_QPNP=y<br>
# CONFIG_RTC_DRV_XGENE is not set<br>
<br>
#<br>
# HID Sensor RTC drivers<br>
#<br>
# CONFIG_RTC_DRV_HID_SENSOR_TIME is not set<br>
# CONFIG_ESOC is not set<br>
CONFIG_DMADEVICES=y<br>
# CONFIG_DMADEVICES_DEBUG is not set<br>
<br>
#<br>
# DMA Devices<br>
#<br>
# CONFIG_AMBA_PL08X is not set<br>
# CONFIG_DW_DMAC_CORE is not set<br>
# CONFIG_DW_DMAC is not set<br>
# CONFIG_DW_DMAC_PCI is not set<br>
CONFIG_QCOM_SPS_DMA=y<br>
# CONFIG_PL330_DMA is not set<br>
# CONFIG_FSL_EDMA is not set<br>
CONFIG_DMA_ENGINE=y<br>
CONFIG_DMA_OF=y<br>
<br>
#<br>
# DMA Clients<br>
#<br>
# CONFIG_ASYNC_TX_DMA is not set<br>
# CONFIG_DMATEST is not set<br>
# CONFIG_AUXDISPLAY is not set<br>
CONFIG_UIO=y<br>
# CONFIG_UIO_CIF is not set<br>
# CONFIG_UIO_PDRV_GENIRQ is not set<br>
# CONFIG_UIO_DMEM_GENIRQ is not set<br>
# CONFIG_UIO_AEC is not set<br>
# CONFIG_UIO_SERCOS3 is not set<br>
# CONFIG_UIO_PCI_GENERIC is not set<br>
# CONFIG_UIO_NETX is not set<br>
# CONFIG_UIO_MF624 is not set<br>
CONFIG_UIO_MSM_SHAREDMEM=y<br>
# CONFIG_VFIO is not set<br>
# CONFIG_VIRT_DRIVERS is not set<br>
<br>
#<br>
# Virtio drivers<br>
#<br>
# CONFIG_VIRTIO_PCI is not set<br>
# CONFIG_VIRTIO_MMIO is not set<br>
<br>
#<br>
# Microsoft Hyper-V guest support<br>
#<br>
CONFIG_STAGING=y<br>
# CONFIG_PRISM2_USB is not set<br>
# CONFIG_R8712U is not set<br>
# CONFIG_R8188EU is not set<br>
# CONFIG_R8723AU is not set<br>
# CONFIG_RTS5208 is not set<br>
# CONFIG_LINE6_USB is not set<br>
# CONFIG_FB_XGI is not set<br>
# CONFIG_BCM_WIMAX is not set<br>
# CONFIG_FT1000 is not set<br>
<br>
#<br>
# Speakup console speech<br>
#<br>
# CONFIG_SPEAKUP is not set<br>
# CONFIG_TOUCHSCREEN_CLEARPAD_TM1217 is not set<br>
# CONFIG_STAGING_MEDIA is not set<br>
<br>
#<br>
# Android<br>
#<br>
CONFIG_ANDROID=y<br>
CONFIG_ANDROID_BINDER_IPC=y<br>
CONFIG_ASHMEM=y<br>
# CONFIG_ANDROID_LOGGER is not set<br>
CONFIG_ANDROID_TIMED_OUTPUT=y<br>
CONFIG_ANDROID_TIMED_GPIO=y<br>
CONFIG_ANDROID_LOW_MEMORY_KILLER=y<br>
CONFIG_ANDROID_LOW_MEMORY_KILLER_AUTODETECT_OOM_ADJ_VALUES=y<br>
CONFIG_SYNC=y<br>
CONFIG_SW_SYNC=y<br>
CONFIG_SW_SYNC_USER=y<br>
CONFIG_ONESHOT_SYNC=y<br>
# CONFIG_ONESHOT_SYNC_USER is not set<br>
CONFIG_ION=y<br>
# CONFIG_ION_TEST is not set<br>
# CONFIG_ION_DUMMY is not set<br>
CONFIG_ION_MSM=y<br>
# CONFIG_ALLOC_BUFFERS_IN_4K_CHUNKS is not set<br>
# CONFIG_FIQ_DEBUGGER is not set<br>
# CONFIG_FIQ_WATCHDOG is not set<br>
# CONFIG_STAGING_BOARD is not set<br>
# CONFIG_USB_WPAN_HCD is not set<br>
# CONFIG_WIMAX_GDM72XX is not set<br>
# CONFIG_DGNC is not set<br>
# CONFIG_DGAP is not set<br>
# CONFIG_GS_FPGABOOT is not set<br>
<br>
#<br>
# Qualcomm Atheros Prima WLAN module<br>
#<br>
# CONFIG_PRIMA_WLAN is not set<br>
CONFIG_PRONTO_WLAN=y<br>
# CONFIG_PRIMA_WLAN_BTAMP is not set<br>
CONFIG_PRIMA_WLAN_LFR=y<br>
CONFIG_PRIMA_WLAN_OKC=y<br>
CONFIG_PRIMA_WLAN_11AC_HIGH_TP=y<br>
CONFIG_WLAN_FEATURE_11W=y<br>
CONFIG_QCOM_VOWIFI_11R=y<br>
CONFIG_ENABLE_LINUX_REG=y<br>
CONFIG_WLAN_OFFLOAD_PACKETS=y<br>
CONFIG_QCOM_TDLS=y<br>
# CONFIG_GOLDFISH is not set<br>
<br>
#<br>
# Qualcomm MSM specific device drivers<br>
#<br>
CONFIG_MSM_AVTIMER=y<br>
CONFIG_MSM_BUS_SCALING=y<br>
CONFIG_BUS_TOPOLOGY_ADHOC=y<br>
# CONFIG_DEBUG_BUS_VOTER is not set<br>
CONFIG_QPNP_POWER_ON=y<br>
CONFIG_QPNP_REVID=y<br>
CONFIG_QPNP_COINCELL=y<br>
CONFIG_SPS=y<br>
# CONFIG_EP_PCIE is not set<br>
CONFIG_USB_BAM=y<br>
# CONFIG_SPS_SUPPORT_BAMDMA is not set<br>
CONFIG_SPS_SUPPORT_NDP_BAM=y<br>
# CONFIG_QPNP_VIBRATOR is not set<br>
CONFIG_IPA=y<br>
# CONFIG_IPA3 is not set<br>
# CONFIG_GSI is not set<br>
CONFIG_RMNET_IPA=y<br>
# CONFIG_SSM is not set<br>
# CONFIG_MSM_MHI is not set<br>
# CONFIG_PFT is not set<br>
# CONFIG_I2C_MSM_PROF_DBG is not set<br>
# CONFIG_SEEMP_CORE is not set<br>
CONFIG_QPNP_HAPTIC=y<br>
# CONFIG_GPIO_USB_DETECT is not set<br>
CONFIG_MSM_11AD=y<br>
# CONFIG_BW_MONITOR is not set<br>
CONFIG_MSM_SPMI=y<br>
CONFIG_MSM_SPMI_PMIC_ARB=y<br>
CONFIG_MSM_QPNP_INT=y<br>
# CONFIG_MSM_SPMI_DEBUGFS_RO is not set<br>
CONFIG_MACH_XIAOMI_MSM8953=y<br>
<br>
#<br>
# Xiaomi board selection<br>
#<br>
CONFIG_MACH_XIAOMI_MIDO=y<br>
<br>
#<br>
# SOC (System On Chip) specific Drivers<br>
#<br>
CONFIG_CP_ACCESS64=y<br>
# CONFIG_MSM_INRUSH_CURRENT_MITIGATION is not set<br>
CONFIG_MSM_QDSP6_APRV2=y<br>
# CONFIG_MSM_GLADIATOR_ERP is not set<br>
# CONFIG_MSM_GLADIATOR_ERP_V2 is not set<br>
# CONFIG_MSM_QDSP6_APRV3 is not set<br>
# CONFIG_MSM_QDSP6_APRV2_GLINK is not set<br>
# CONFIG_MSM_QDSP6_APRV3_GLINK is not set<br>
CONFIG_MSM_ADSP_LOADER=y<br>
# CONFIG_MSM_MEMORY_DUMP is not set<br>
CONFIG_MSM_MEMORY_DUMP_V2=y<br>
# CONFIG_MSM_DEBUG_LAR_UNLOCK is not set<br>
# CONFIG_MSM_JTAG is not set<br>
# CONFIG_MSM_JTAG_MM is not set<br>
# CONFIG_MSM_JTAGV8 is not set<br>
CONFIG_MSM_BOOT_STATS=y<br>
# CONFIG_MSM_BOOT_TIME_MARKER is not set<br>
CONFIG_MSM_CPUSS_DUMP=y<br>
CONFIG_MSM_COMMON_LOG=y<br>
CONFIG_MSM_DDR_HEALTH=y<br>
# CONFIG_MSM_HYP_DEBUG is not set<br>
CONFIG_MSM_WATCHDOG_V2=y<br>
CONFIG_MSM_FORCE_WDOG_BITE_ON_PANIC=y<br>
# CONFIG_MSM_CORE_HANG_DETECT is not set<br>
# CONFIG_MSM_GLADIATOR_HANG_DETECT is not set<br>
CONFIG_MSM_CPU_PWR_CTL=y<br>
# CONFIG_MSM_L2_IA_DEBUG is not set<br>
CONFIG_MSM_RPM_SMD=y<br>
CONFIG_MSM_RPM_RBCPR_STATS_V2_LOG=y<br>
CONFIG_MSM_RPM_LOG=y<br>
CONFIG_MSM_RPM_STATS_LOG=y<br>
CONFIG_MSM_RUN_QUEUE_STATS=y<br>
CONFIG_MSM_SCM=y<br>
CONFIG_MSM_SCM_XPU=y<br>
CONFIG_MSM_XPU_ERR_FATAL=y<br>
# CONFIG_MSM_XPU_ERR_NONFATAL is not set<br>
# CONFIG_MSM_SCM_ERRATA is not set<br>
# CONFIG_MSM_PFE_WA is not set<br>
CONFIG_MSM_MPM_OF=y<br>
CONFIG_MSM_SMEM=y<br>
CONFIG_MSM_SMD=y<br>
CONFIG_MSM_SMD_DEBUG=y<br>
CONFIG_MSM_GLINK=y<br>
CONFIG_MSM_GLINK_LOOPBACK_SERVER=y<br>
CONFIG_MSM_GLINK_SMD_XPRT=y<br>
CONFIG_MSM_GLINK_SMEM_NATIVE_XPRT=y<br>
CONFIG_MSM_SMEM_LOGGING=y<br>
CONFIG_MSM_SMP2P=y<br>
CONFIG_MSM_SMP2P_TEST=y<br>
CONFIG_MSM_SPM=y<br>
CONFIG_MSM_L2_SPM=y<br>
CONFIG_MSM_QMI_INTERFACE=y<br>
# CONFIG_MSM_DCC is not set<br>
# CONFIG_MSM_HVC is not set<br>
CONFIG_MSM_IPC_ROUTER_SMD_XPRT=y<br>
CONFIG_MSM_EVENT_TIMER=y<br>
# CONFIG_MSM_SYSMON_GLINK_COMM is not set<br>
# CONFIG_MSM_IPC_ROUTER_GLINK_XPRT is not set<br>
# CONFIG_MSM_SYSTEM_HEALTH_MONITOR is not set<br>
# CONFIG_MSM_GLINK_PKT is not set<br>
CONFIG_MSM_TZ_SMMU=y<br>
CONFIG_MSM_SUBSYSTEM_RESTART=y<br>
CONFIG_MSM_SYSMON_COMM=y<br>
CONFIG_MSM_PIL=y<br>
CONFIG_MSM_PIL_SSR_GENERIC=y<br>
CONFIG_MSM_PIL_MSS_QDSP6V5=y<br>
# CONFIG_MSM_SHARED_HEAP_ACCESS is not set<br>
# CONFIG_TRACER_PKT is not set<br>
CONFIG_MSM_SECURE_BUFFER=y<br>
CONFIG_ICNSS=y<br>
CONFIG_MSM_BAM_DMUX=y<br>
CONFIG_MSM_PERFORMANCE=y<br>
CONFIG_MSM_PERFORMANCE_HOTPLUG_ON=y<br>
# CONFIG_MSM_POWER is not set<br>
# CONFIG_MSM_SERVICE_LOCATOR is not set<br>
# CONFIG_MSM_QBT1000 is not set<br>
# CONFIG_MSM_PACMAN is not set<br>
# CONFIG_MSM_REMOTEQDSS is not set<br>
# CONFIG_QCOM_SMCINVOKE is not set<br>
CONFIG_SERIAL_NUM=y<br>
CONFIG_MEM_SHARE_QMI_SERVICE=y<br>
# CONFIG_SOC_TI is not set<br>
CONFIG_CLKDEV_LOOKUP=y<br>
CONFIG_HAVE_CLK_PREPARE=y<br>
CONFIG_MSM_CLK_CONTROLLER_V2=y<br>
CONFIG_MSM_MDSS_PLL=y<br>
CONFIG_HWSPINLOCK=y<br>
<br>
#<br>
# Hardware Spinlock drivers<br>
#<br>
CONFIG_REMOTE_SPINLOCK_MSM=y<br>
<br>
#<br>
# Clock Source drivers<br>
#<br>
CONFIG_CLKSRC_OF=y<br>
CONFIG_ARM_ARCH_TIMER=y<br>
CONFIG_ARM_ARCH_TIMER_EVTSTREAM=y<br>
# CONFIG_ATMEL_PIT is not set<br>
# CONFIG_SH_TIMER_CMT is not set<br>
# CONFIG_SH_TIMER_MTU2 is not set<br>
# CONFIG_SH_TIMER_TMU is not set<br>
# CONFIG_EM_TIMER_STI is not set<br>
# CONFIG_CLKSRC_VERSATILE is not set<br>
# CONFIG_MAILBOX is not set<br>
CONFIG_IOMMU_API=y<br>
CONFIG_IOMMU_SUPPORT=y<br>
<br>
#<br>
# Generic IOMMU Pagetable Support<br>
#<br>
CONFIG_IOMMU_IO_PGTABLE=y<br>
CONFIG_IOMMU_IO_PGTABLE_LPAE=y<br>
# CONFIG_IOMMU_IO_PGTABLE_LPAE_SELFTEST is not set<br>
# CONFIG_IOMMU_IO_PGTABLE_FAST is not set<br>
CONFIG_OF_IOMMU=y<br>
CONFIG_MSM_IOMMU=y<br>
CONFIG_MSM_IOMMU_V1=y<br>
# CONFIG_IOMMU_LPAE is not set<br>
# CONFIG_IOMMU_AARCH64 is not set<br>
# CONFIG_MSM_IOMMU_VBIF_CHECK is not set<br>
# CONFIG_IOMMU_NON_SECURE is not set<br>
# CONFIG_IOMMU_FORCE_4K_MAPPINGS is not set<br>
CONFIG_ARM_SMMU=y<br>
CONFIG_IOMMU_DEBUG=y<br>
# CONFIG_IOMMU_DEBUG_TRACKING is not set<br>
CONFIG_IOMMU_TESTS=y<br>
<br>
#<br>
# Remoteproc drivers<br>
#<br>
# CONFIG_STE_MODEM_RPROC is not set<br>
<br>
#<br>
# Rpmsg drivers<br>
#<br>
<br>
#<br>
# SOC (System On Chip) specific Drivers<br>
#<br>
CONFIG_PM_DEVFREQ=y<br>
<br>
#<br>
# DEVFREQ Governors<br>
#<br>
CONFIG_DEVFREQ_GOV_SIMPLE_ONDEMAND=y<br>
CONFIG_DEVFREQ_GOV_PERFORMANCE=y<br>
CONFIG_DEVFREQ_GOV_POWERSAVE=y<br>
CONFIG_DEVFREQ_GOV_USERSPACE=y<br>
CONFIG_DEVFREQ_GOV_CPUFREQ=y<br>
CONFIG_DEVFREQ_GOV_MSM_ADRENO_TZ=y<br>
CONFIG_MSM_BIMC_BWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_GPUBW_MON=y<br>
CONFIG_ARM_MEMLAT_MON=y<br>
CONFIG_MSMCCI_HWMON=y<br>
CONFIG_MSM_M4M_HWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_BW_HWMON=y<br>
CONFIG_DEVFREQ_GOV_MSM_CACHE_HWMON=y<br>
CONFIG_DEVFREQ_GOV_SPDM_HYP=y<br>
CONFIG_DEVFREQ_GOV_MEMLAT=y<br>
<br>
#<br>
# DEVFREQ Drivers<br>
#<br>
CONFIG_DEVFREQ_SIMPLE_DEV=y<br>
CONFIG_MSM_DEVFREQ_DEVBW=y<br>
CONFIG_SPDM_SCM=y<br>
CONFIG_DEVFREQ_SPDM=y<br>
# CONFIG_EXTCON is not set<br>
# CONFIG_MEMORY is not set<br>
# CONFIG_IIO is not set<br>
# CONFIG_VME_BUS is not set<br>
CONFIG_PWM=y<br>
CONFIG_PWM_SYSFS=y<br>
# CONFIG_PWM_FSL_FTM is not set<br>
# CONFIG_PWM_PCA9685 is not set<br>
CONFIG_PWM_QPNP=y<br>
CONFIG_IRQCHIP=y<br>
CONFIG_ARM_GIC=y<br>
CONFIG_ARM_GIC_V2M=y<br>
CONFIG_ARM_GIC_V3=y<br>
CONFIG_ARM_GIC_PANIC_HANDLER=y<br>
CONFIG_ARM_GIC_V3_ITS=y<br>
CONFIG_ARM_GIC_V3_ACL=y<br>
# CONFIG_ARM_GIC_V3_NO_ACCESS_CONTROL is not set<br>
CONFIG_MSM_SHOW_RESUME_IRQ=y<br>
CONFIG_MSM_IRQ=y<br>
# CONFIG_IPACK_BUS is not set<br>
# CONFIG_RESET_CONTROLLER is not set<br>
# CONFIG_FMC is not set<br>
CONFIG_CORESIGHT=y<br>
CONFIG_CORESIGHT_EVENT=y<br>
CONFIG_HAVE_CORESIGHT_SINK=y<br>
CONFIG_CORESIGHT_FUSE=y<br>
CONFIG_CORESIGHT_CTI=y<br>
CONFIG_CORESIGHT_CTI_SAVE_DISABLE=y<br>
CONFIG_CORESIGHT_CSR=y<br>
CONFIG_CORESIGHT_TMC=y<br>
CONFIG_CORESIGHT_TPIU=y<br>
CONFIG_CORESIGHT_FUNNEL=y<br>
CONFIG_CORESIGHT_REPLICATOR=y<br>
# CONFIG_CORESIGHT_TPDA is not set<br>
# CONFIG_CORESIGHT_TPDM is not set<br>
# CONFIG_CORESIGHT_DBGUI is not set<br>
CONFIG_CORESIGHT_STM=y<br>
# CONFIG_CORESIGHT_STM_DEFAULT_ENABLE is not set<br>
CONFIG_CORESIGHT_HWEVENT=y<br>
# CONFIG_CORESIGHT_ETM is not set<br>
# CONFIG_CORESIGHT_ETMV4 is not set<br>
# CONFIG_CORESIGHT_REMOTE_ETM is not set<br>
# CONFIG_CORESIGHT_QPDI is not set<br>
<br>
#<br>
# PHY Subsystem<br>
#<br>
CONFIG_GENERIC_PHY=y<br>
# CONFIG_BCM_KONA_USB2_PHY is not set<br>
# CONFIG_PHY_XGENE is not set<br>
CONFIG_PHY_QCOM_UFS=y<br>
# CONFIG_POWERCAP is not set<br>
# CONFIG_MCB is not set<br>
CONFIG_RAS=y<br>
# CONFIG_THUNDERBOLT is not set<br>
CONFIG_SENSORS_SSC=y<br>
<br>
#<br>
# Firmware Drivers<br>
#<br>
# CONFIG_FIRMWARE_MEMMAP is not set<br>
CONFIG_DMIID=y<br>
# CONFIG_DMI_SYSFS is not set<br>
<br>
#<br>
# EFI (Extensible Firmware Interface) Support<br>
#<br>
# CONFIG_EFI_VARS is not set<br>
CONFIG_EFI_PARAMS_FROM_FDT=y<br>
CONFIG_EFI_RUNTIME_WRAPPERS=y<br>
CONFIG_EFI_ARMSTUB=y<br>
CONFIG_MSM_TZ_LOG=y<br>
# CONFIG_BIF is not set<br>
<br>
#<br>
# Firmware Drivers<br>
#<br>
<br>
#<br>
# EFI (Extensible Firmware Interface) Support<br>
#<br>
<br>
#<br>
# File systems<br>
#<br>
CONFIG_DCACHE_WORD_ACCESS=y<br>
CONFIG_EXT2_FS=y<br>
CONFIG_EXT2_FS_XATTR=y<br>
# CONFIG_EXT2_FS_POSIX_ACL is not set<br>
# CONFIG_EXT2_FS_SECURITY is not set<br>
# CONFIG_EXT2_FS_XIP is not set<br>
CONFIG_EXT3_FS=y<br>
# CONFIG_EXT3_DEFAULTS_TO_ORDERED is not set<br>
CONFIG_EXT3_FS_XATTR=y<br>
# CONFIG_EXT3_FS_POSIX_ACL is not set<br>
# CONFIG_EXT3_FS_SECURITY is not set<br>
CONFIG_EXT4_FS=y<br>
# CONFIG_EXT4_FS_POSIX_ACL is not set<br>
CONFIG_EXT4_FS_SECURITY=y<br>
# CONFIG_EXT4_DEBUG is not set<br>
CONFIG_JBD=y<br>
# CONFIG_JBD_DEBUG is not set<br>
CONFIG_JBD2=y<br>
# CONFIG_JBD2_DEBUG is not set<br>
CONFIG_FS_MBCACHE=y<br>
# CONFIG_REISERFS_FS is not set<br>
# CONFIG_JFS_FS is not set<br>
# CONFIG_XFS_FS is not set<br>
# CONFIG_GFS2_FS is not set<br>
# CONFIG_OCFS2_FS is not set<br>
CONFIG_BTRFS_FS=y<br>
# CONFIG_BTRFS_FS_POSIX_ACL is not set<br>
# CONFIG_BTRFS_FS_CHECK_INTEGRITY is not set<br>
# CONFIG_BTRFS_FS_RUN_SANITY_TESTS is not set<br>
# CONFIG_BTRFS_DEBUG is not set<br>
# CONFIG_BTRFS_ASSERT is not set<br>
# CONFIG_NILFS2_FS is not set<br>
CONFIG_FS_POSIX_ACL=y<br>
CONFIG_EXPORTFS=y<br>
CONFIG_FILE_LOCKING=y<br>
CONFIG_FS_ENCRYPTION=y<br>
CONFIG_FSNOTIFY=y<br>
CONFIG_DNOTIFY=y<br>
CONFIG_INOTIFY_USER=y<br>
CONFIG_FANOTIFY=y<br>
# CONFIG_FANOTIFY_ACCESS_PERMISSIONS is not set<br>
CONFIG_QUOTA=y<br>
# CONFIG_QUOTA_NETLINK_INTERFACE is not set<br>
# CONFIG_PRINT_QUOTA_WARNING is not set<br>
# CONFIG_QUOTA_DEBUG is not set<br>
# CONFIG_QFMT_V1 is not set<br>
# CONFIG_QFMT_V2 is not set<br>
CONFIG_QUOTACTL=y<br>
CONFIG_AUTOFS4_FS=y<br>
CONFIG_FUSE_FS=y<br>
# CONFIG_CUSE is not set<br>
# CONFIG_OVERLAY_FS is not set<br>
<br>
#<br>
# Caches<br>
#<br>
# CONFIG_FSCACHE is not set<br>
<br>
#<br>
# CD-ROM/DVD Filesystems<br>
#<br>
CONFIG_ISO9660_FS=y<br>
# CONFIG_JOLIET is not set<br>
# CONFIG_ZISOFS is not set<br>
CONFIG_UDF_FS=y<br>
CONFIG_UDF_NLS=y<br>
<br>
#<br>
# DOS/FAT/NT Filesystems<br>
#<br>
CONFIG_FAT_FS=y<br>
CONFIG_MSDOS_FS=y<br>
CONFIG_VFAT_FS=y<br>
CONFIG_FAT_DEFAULT_CODEPAGE=437<br>
CONFIG_FAT_DEFAULT_IOCHARSET="iso8859-1"<br>
# CONFIG_NTFS_FS is not set<br>
<br>
#<br>
# Pseudo filesystems<br>
#<br>
CONFIG_PROC_FS=y<br>
# CONFIG_PROC_KCORE is not set<br>
CONFIG_PROC_SYSCTL=y<br>
CONFIG_PROC_PAGE_MONITOR=y<br>
CONFIG_KERNFS=y<br>
CONFIG_SYSFS=y<br>
CONFIG_TMPFS=y<br>
CONFIG_TMPFS_POSIX_ACL=y<br>
CONFIG_TMPFS_XATTR=y<br>
# CONFIG_HUGETLBFS is not set<br>
# CONFIG_HUGETLB_PAGE is not set<br>
CONFIG_CONFIGFS_FS=y<br>
CONFIG_MISC_FILESYSTEMS=y<br>
# CONFIG_ADFS_FS is not set<br>
# CONFIG_AFFS_FS is not set<br>
CONFIG_ECRYPT_FS=y<br>
# CONFIG_ECRYPT_FS_MESSAGING is not set<br>
CONFIG_SDCARD_FS=y<br>
# CONFIG_HFS_FS is not set<br>
# CONFIG_HFSPLUS_FS is not set<br>
# CONFIG_BEFS_FS is not set<br>
# CONFIG_BFS_FS is not set<br>
# CONFIG_EFS_FS is not set<br>
# CONFIG_LOGFS is not set<br>
# CONFIG_CRAMFS is not set<br>
# CONFIG_SQUASHFS is not set<br>
# CONFIG_VXFS_FS is not set<br>
# CONFIG_MINIX_FS is not set<br>
# CONFIG_OMFS_FS is not set<br>
# CONFIG_HPFS_FS is not set<br>
# CONFIG_QNX4FS_FS is not set<br>
# CONFIG_QNX6FS_FS is not set<br>
# CONFIG_ROMFS_FS is not set<br>
CONFIG_PSTORE=y<br>
CONFIG_PSTORE_CONSOLE=y<br>
# CONFIG_PSTORE_PMSG is not set<br>
CONFIG_PSTORE_FTRACE=y<br>
CONFIG_PSTORE_RAM=y<br>
# CONFIG_SYSV_FS is not set<br>
# CONFIG_UFS_FS is not set<br>
CONFIG_F2FS_FS=y<br>
CONFIG_F2FS_STAT_FS=y<br>
CONFIG_F2FS_FS_XATTR=y<br>
CONFIG_F2FS_FS_POSIX_ACL=y<br>
CONFIG_F2FS_FS_SECURITY=y<br>
# CONFIG_F2FS_CHECK_FS is not set<br>
CONFIG_F2FS_FS_ENCRYPTION=y<br>
# CONFIG_F2FS_IO_TRACE is not set<br>
# CONFIG_F2FS_FAULT_INJECTION is not set<br>
# CONFIG_EFIVAR_FS is not set<br>
CONFIG_NETWORK_FILESYSTEMS=y<br>
CONFIG_NFS_FS=y<br>
CONFIG_NFS_V2=y<br>
CONFIG_NFS_V3=y<br>
CONFIG_NFS_V3_ACL=y<br>
CONFIG_NFS_V4=y<br>
# CONFIG_NFS_SWAP is not set<br>
CONFIG_NFS_V4_1=y<br>
# CONFIG_NFS_V4_2 is not set<br>
CONFIG_PNFS_FILE_LAYOUT=y<br>
CONFIG_PNFS_BLOCK=y<br>
CONFIG_NFS_V4_1_IMPLEMENTATION_ID_DOMAIN="kernel.org"<br>
# CONFIG_NFS_V4_1_MIGRATION is not set<br>
# CONFIG_ROOT_NFS is not set<br>
# CONFIG_NFS_USE_LEGACY_DNS is not set<br>
CONFIG_NFS_USE_KERNEL_DNS=y<br>
# CONFIG_NFSD is not set<br>
CONFIG_GRACE_PERIOD=y<br>
CONFIG_LOCKD=y<br>
CONFIG_LOCKD_V4=y<br>
CONFIG_NFS_ACL_SUPPORT=y<br>
CONFIG_NFS_COMMON=y<br>
CONFIG_SUNRPC=y<br>
CONFIG_SUNRPC_GSS=y<br>
CONFIG_SUNRPC_BACKCHANNEL=y<br>
CONFIG_RPCSEC_GSS_KRB5=y<br>
# CONFIG_SUNRPC_DEBUG is not set<br>
# CONFIG_CEPH_FS is not set<br>
CONFIG_CIFS=y<br>
# CONFIG_CIFS_STATS is not set<br>
# CONFIG_CIFS_WEAK_PW_HASH is not set<br>
# CONFIG_CIFS_UPCALL is not set<br>
# CONFIG_CIFS_XATTR is not set<br>
CONFIG_CIFS_DEBUG=y<br>
# CONFIG_CIFS_DEBUG2 is not set<br>
# CONFIG_CIFS_DFS_UPCALL is not set<br>
# CONFIG_CIFS_NFSD_EXPORT is not set<br>
# CONFIG_CIFS_SMB2 is not set<br>
# CONFIG_NCP_FS is not set<br>
# CONFIG_CODA_FS is not set<br>
# CONFIG_AFS_FS is not set<br>
CONFIG_NLS=y<br>
CONFIG_NLS_DEFAULT="iso8859-1"<br>
CONFIG_NLS_CODEPAGE_437=y<br>
# CONFIG_NLS_CODEPAGE_737 is not set<br>
# CONFIG_NLS_CODEPAGE_775 is not set<br>
# CONFIG_NLS_CODEPAGE_850 is not set<br>
# CONFIG_NLS_CODEPAGE_852 is not set<br>
# CONFIG_NLS_CODEPAGE_855 is not set<br>
# CONFIG_NLS_CODEPAGE_857 is not set<br>
# CONFIG_NLS_CODEPAGE_860 is not set<br>
# CONFIG_NLS_CODEPAGE_861 is not set<br>
# CONFIG_NLS_CODEPAGE_862 is not set<br>
# CONFIG_NLS_CODEPAGE_863 is not set<br>
# CONFIG_NLS_CODEPAGE_864 is not set<br>
# CONFIG_NLS_CODEPAGE_865 is not set<br>
# CONFIG_NLS_CODEPAGE_866 is not set<br>
# CONFIG_NLS_CODEPAGE_869 is not set<br>
CONFIG_NLS_CODEPAGE_936=y<br>
CONFIG_NLS_CODEPAGE_950=y<br>
# CONFIG_NLS_CODEPAGE_932 is not set<br>
# CONFIG_NLS_CODEPAGE_949 is not set<br>
# CONFIG_NLS_CODEPAGE_874 is not set<br>
# CONFIG_NLS_ISO8859_8 is not set<br>
# CONFIG_NLS_CODEPAGE_1250 is not set<br>
# CONFIG_NLS_CODEPAGE_1251 is not set<br>
CONFIG_NLS_ASCII=y<br>
CONFIG_NLS_ISO8859_1=y<br>
# CONFIG_NLS_ISO8859_2 is not set<br>
# CONFIG_NLS_ISO8859_3 is not set<br>
# CONFIG_NLS_ISO8859_4 is not set<br>
# CONFIG_NLS_ISO8859_5 is not set<br>
# CONFIG_NLS_ISO8859_6 is not set<br>
# CONFIG_NLS_ISO8859_7 is not set<br>
# CONFIG_NLS_ISO8859_9 is not set<br>
# CONFIG_NLS_ISO8859_13 is not set<br>
# CONFIG_NLS_ISO8859_14 is not set<br>
# CONFIG_NLS_ISO8859_15 is not set<br>
# CONFIG_NLS_KOI8_R is not set<br>
# CONFIG_NLS_KOI8_U is not set<br>
# CONFIG_NLS_MAC_ROMAN is not set<br>
# CONFIG_NLS_MAC_CELTIC is not set<br>
# CONFIG_NLS_MAC_CENTEURO is not set<br>
# CONFIG_NLS_MAC_CROATIAN is not set<br>
# CONFIG_NLS_MAC_CYRILLIC is not set<br>
# CONFIG_NLS_MAC_GAELIC is not set<br>
# CONFIG_NLS_MAC_GREEK is not set<br>
# CONFIG_NLS_MAC_ICELAND is not set<br>
# CONFIG_NLS_MAC_INUIT is not set<br>
# CONFIG_NLS_MAC_ROMANIAN is not set<br>
# CONFIG_NLS_MAC_TURKISH is not set<br>
CONFIG_NLS_UTF8=y<br>
# CONFIG_DLM is not set<br>
# CONFIG_FILE_TABLE_DEBUG is not set<br>
# CONFIG_VIRTUALIZATION is not set<br>
<br>
#<br>
# Kernel hacking<br>
#<br>
<br>
#<br>
# printk and dmesg options<br>
#<br>
CONFIG_PRINTK_TIME=y<br>
CONFIG_MESSAGE_LOGLEVEL_DEFAULT=4<br>
# CONFIG_LOG_BUF_MAGIC is not set<br>
# CONFIG_BOOT_PRINTK_DELAY is not set<br>
# CONFIG_DYNAMIC_DEBUG is not set<br>
<br>
#<br>
# Compile-time checks and compiler options<br>
#<br>
CONFIG_DEBUG_INFO=y<br>
# CONFIG_DEBUG_INFO_REDUCED is not set<br>
# CONFIG_DEBUG_INFO_SPLIT is not set<br>
# CONFIG_DEBUG_INFO_DWARF4 is not set<br>
CONFIG_ENABLE_WARN_DEPRECATED=y<br>
CONFIG_ENABLE_MUST_CHECK=y<br>
CONFIG_FRAME_WARN=2048<br>
# CONFIG_STRIP_ASM_SYMS is not set<br>
# CONFIG_READABLE_ASM is not set<br>
# CONFIG_UNUSED_SYMBOLS is not set<br>
# CONFIG_PAGE_OWNER is not set<br>
CONFIG_DEBUG_FS=y<br>
# CONFIG_HEADERS_CHECK is not set<br>
# CONFIG_DEBUG_SECTION_MISMATCH is not set<br>
CONFIG_ARCH_WANT_FRAME_POINTERS=y<br>
CONFIG_FRAME_POINTER=y<br>
# CONFIG_DEBUG_FORCE_WEAK_PER_CPU is not set<br>
CONFIG_MAGIC_SYSRQ=y<br>
CONFIG_MAGIC_SYSRQ_DEFAULT_ENABLE=0x1<br>
CONFIG_DEBUG_KERNEL=y<br>
<br>
#<br>
# Memory Debugging<br>
#<br>
# CONFIG_DEBUG_PAGEALLOC is not set<br>
# CONFIG_DEBUG_OBJECTS is not set<br>
# CONFIG_SLUB_STATS is not set<br>
CONFIG_HAVE_DEBUG_KMEMLEAK=y<br>
# CONFIG_DEBUG_KMEMLEAK is not set<br>
# CONFIG_DEBUG_STACK_USAGE is not set<br>
# CONFIG_DEBUG_VM is not set<br>
# CONFIG_DEBUG_MEMORY_INIT is not set<br>
# CONFIG_DEBUG_PER_CPU_MAPS is not set<br>
CONFIG_HAVE_ARCH_KASAN=y<br>
# CONFIG_DEBUG_SHIRQ is not set<br>
<br>
#<br>
# Debug Lockups and Hangs<br>
#<br>
# CONFIG_LOCKUP_DETECTOR is not set<br>
# CONFIG_DETECT_HUNG_TASK is not set<br>
# CONFIG_PANIC_ON_OOPS is not set<br>
CONFIG_PANIC_ON_OOPS_VALUE=0<br>
CONFIG_PANIC_TIMEOUT=5<br>
CONFIG_PANIC_ON_RECURSIVE_FAULT=y<br>
CONFIG_SCHED_DEBUG=y<br>
# CONFIG_PANIC_ON_SCHED_BUG is not set<br>
# CONFIG_PANIC_ON_RT_THROTTLING is not set<br>
CONFIG_SYSRQ_SCHED_DEBUG=y<br>
CONFIG_SCHEDSTATS=y<br>
# CONFIG_SCHED_STACK_END_CHECK is not set<br>
CONFIG_TIMER_STATS=y<br>
CONFIG_DEBUG_PREEMPT=y<br>
<br>
#<br>
# Lock Debugging (spinlocks, mutexes, etc...)<br>
#<br>
# CONFIG_DEBUG_RT_MUTEXES is not set<br>
# CONFIG_RT_MUTEX_TESTER is not set<br>
# CONFIG_DEBUG_SPINLOCK is not set<br>
# CONFIG_DEBUG_MUTEXES is not set<br>
# CONFIG_DEBUG_WW_MUTEX_SLOWPATH is not set<br>
# CONFIG_DEBUG_LOCK_ALLOC is not set<br>
# CONFIG_PROVE_LOCKING is not set<br>
# CONFIG_LOCK_STAT is not set<br>
# CONFIG_DEBUG_ATOMIC_SLEEP is not set<br>
# CONFIG_DEBUG_LOCKING_API_SELFTESTS is not set<br>
# CONFIG_LOCK_TORTURE_TEST is not set<br>
CONFIG_STACKTRACE=y<br>
# CONFIG_DEBUG_KOBJECT is not set<br>
CONFIG_HAVE_DEBUG_BUGVERBOSE=y<br>
CONFIG_DEBUG_BUGVERBOSE=y<br>
# CONFIG_DEBUG_LIST is not set<br>
# CONFIG_DEBUG_PI_LIST is not set<br>
# CONFIG_DEBUG_SG is not set<br>
# CONFIG_DEBUG_NOTIFIERS is not set<br>
# CONFIG_DEBUG_CREDENTIALS is not set<br>
<br>
#<br>
# RCU Debugging<br>
#<br>
# CONFIG_SPARSE_RCU_POINTER is not set<br>
# CONFIG_TORTURE_TEST is not set<br>
# CONFIG_RCU_TORTURE_TEST is not set<br>
CONFIG_RCU_CPU_STALL_TIMEOUT=21<br>
CONFIG_RCU_CPU_STALL_VERBOSE=y<br>
# CONFIG_RCU_CPU_STALL_INFO is not set<br>
# CONFIG_RCU_TRACE is not set<br>
# CONFIG_DEBUG_BLOCK_EXT_DEVT is not set<br>
# CONFIG_NOTIFIER_ERROR_INJECTION is not set<br>
# CONFIG_FAULT_INJECTION is not set<br>
CONFIG_NOP_TRACER=y<br>
CONFIG_HAVE_FUNCTION_TRACER=y<br>
CONFIG_HAVE_FUNCTION_GRAPH_TRACER=y<br>
CONFIG_HAVE_DYNAMIC_FTRACE=y<br>
CONFIG_HAVE_FTRACE_MCOUNT_RECORD=y<br>
CONFIG_HAVE_SYSCALL_TRACEPOINTS=y<br>
CONFIG_HAVE_C_RECORDMCOUNT=y<br>
CONFIG_TRACE_CLOCK=y<br>
CONFIG_RING_BUFFER=y<br>
CONFIG_EVENT_TRACING=y<br>
CONFIG_CONTEXT_SWITCH_TRACER=y<br>
# CONFIG_MSM_RTB is not set<br>
CONFIG_IPC_LOGGING=y<br>
CONFIG_TRACING=y<br>
CONFIG_GENERIC_TRACER=y<br>
CONFIG_TRACING_SUPPORT=y<br>
CONFIG_FTRACE=y<br>
CONFIG_FUNCTION_TRACER=y<br>
CONFIG_FUNCTION_GRAPH_TRACER=y<br>
# CONFIG_IRQSOFF_TRACER is not set<br>
# CONFIG_PREEMPT_TRACER is not set<br>
# CONFIG_SCHED_TRACER is not set<br>
# CONFIG_FTRACE_SYSCALLS is not set<br>
# CONFIG_TRACER_SNAPSHOT is not set<br>
CONFIG_BRANCH_PROFILE_NONE=y<br>
# CONFIG_PROFILE_ANNOTATED_BRANCHES is not set<br>
# CONFIG_PROFILE_ALL_BRANCHES is not set<br>
# CONFIG_STACK_TRACER is not set<br>
# CONFIG_BLK_DEV_IO_TRACE is not set<br>
# CONFIG_PROBE_EVENTS is not set<br>
CONFIG_DYNAMIC_FTRACE=y<br>
# CONFIG_FUNCTION_PROFILER is not set<br>
CONFIG_CPU_FREQ_SWITCH_PROFILER=y<br>
CONFIG_FTRACE_MCOUNT_RECORD=y<br>
# CONFIG_FTRACE_STARTUP_TEST is not set<br>
# CONFIG_TRACEPOINT_BENCHMARK is not set<br>
# CONFIG_RING_BUFFER_BENCHMARK is not set<br>
# CONFIG_RING_BUFFER_STARTUP_TEST is not set<br>
<br>
#<br>
# Runtime Testing<br>
#<br>
# CONFIG_LKDTM is not set<br>
# CONFIG_TEST_LIST_SORT is not set<br>
# CONFIG_BACKTRACE_SELF_TEST is not set<br>
# CONFIG_RBTREE_TEST is not set<br>
# CONFIG_ATOMIC64_SELFTEST is not set<br>
# CONFIG_TEST_STRING_HELPERS is not set<br>
# CONFIG_TEST_KSTRTOX is not set<br>
# CONFIG_TEST_RHASHTABLE is not set<br>
# CONFIG_DMA_API_DEBUG is not set<br>
# CONFIG_TEST_FIRMWARE is not set<br>
# CONFIG_TEST_UDELAY is not set<br>
# CONFIG_PANIC_ON_DATA_CORRUPTION is not set<br>
# CONFIG_MEMTEST is not set<br>
# CONFIG_SAMPLES is not set<br>
CONFIG_HAVE_ARCH_KGDB=y<br>
# CONFIG_KGDB is not set<br>
CONFIG_ARCH_HAS_UBSAN_SANITIZE_ALL=y<br>
# CONFIG_UBSAN is not set<br>
# CONFIG_ARM64_PTDUMP is not set<br>
# CONFIG_STRICT_DEVMEM is not set<br>
# CONFIG_PID_IN_CONTEXTIDR is not set<br>
# CONFIG_ARM64_RANDOMIZE_TEXT_OFFSET is not set<br>
# CONFIG_FREE_PAGES_RDONLY is not set<br>
CONFIG_DEBUG_RODATA=y<br>
# CONFIG_DEBUG_ALIGN_RODATA is not set<br>
# CONFIG_CORESIGHT_LINKS_AND_SINKS is not set<br>
# CONFIG_CORESIGHT_SOURCE_ETM4X is not set<br>
<br>
#<br>
# Security options<br>
#<br>
CONFIG_KEYS=y<br>
# CONFIG_PERSISTENT_KEYRINGS is not set<br>
# CONFIG_BIG_KEYS is not set<br>
CONFIG_ENCRYPTED_KEYS=y<br>
# CONFIG_KEYS_DEBUG_PROC_KEYS is not set<br>
<br>
#<br>
# Qualcomm Technologies, Inc Per File Encryption security device drivers<br>
#<br>
# CONFIG_PFK is not set<br>
# CONFIG_SECURITY_DMESG_RESTRICT is not set<br>
# CONFIG_SECURITY_PERF_EVENTS_RESTRICT is not set<br>
CONFIG_SECURITY=y<br>
CONFIG_SECURITYFS=y<br>
CONFIG_SECURITY_NETWORK=y<br>
# CONFIG_SECURITY_NETWORK_XFRM is not set<br>
CONFIG_SECURITY_PATH=y<br>
CONFIG_LSM_MMAP_MIN_ADDR=4096<br>
CONFIG_SECURITY_SELINUX=y<br>
# CONFIG_SECURITY_SELINUX_BOOTPARAM is not set<br>
# CONFIG_SECURITY_SELINUX_DISABLE is not set<br>
CONFIG_SECURITY_SELINUX_DEVELOP=y<br>
CONFIG_SECURITY_SELINUX_AVC_STATS=y<br>
CONFIG_SECURITY_SELINUX_CHECKREQPROT_VALUE=1<br>
# CONFIG_SECURITY_SELINUX_POLICYDB_VERSION_MAX is not set<br>
# CONFIG_SECURITY_SMACK is not set<br>
# CONFIG_SECURITY_TOMOYO is not set<br>
# CONFIG_SECURITY_APPARMOR is not set<br>
CONFIG_SECURITY_YAMA=y<br>
CONFIG_SECURITY_YAMA_STACKED=y<br>
CONFIG_INTEGRITY=y<br>
# CONFIG_INTEGRITY_SIGNATURE is not set<br>
CONFIG_INTEGRITY_AUDIT=y<br>
# CONFIG_IMA is not set<br>
# CONFIG_EVM is not set<br>
CONFIG_DEFAULT_SECURITY_SELINUX=y<br>
# CONFIG_DEFAULT_SECURITY_YAMA is not set<br>
# CONFIG_DEFAULT_SECURITY_DAC is not set<br>
CONFIG_DEFAULT_SECURITY="selinux"<br>
CONFIG_XOR_BLOCKS=y<br>
CONFIG_CRYPTO=y<br>
<br>
#<br>
# Crypto core or helper<br>
#<br>
CONFIG_CRYPTO_ALGAPI=y<br>
CONFIG_CRYPTO_ALGAPI2=y<br>
CONFIG_CRYPTO_AEAD=y<br>
CONFIG_CRYPTO_AEAD2=y<br>
CONFIG_CRYPTO_BLKCIPHER=y<br>
CONFIG_CRYPTO_BLKCIPHER2=y<br>
CONFIG_CRYPTO_HASH=y<br>
CONFIG_CRYPTO_HASH2=y<br>
CONFIG_CRYPTO_RNG=y<br>
CONFIG_CRYPTO_RNG2=y<br>
CONFIG_CRYPTO_PCOMP2=y<br>
CONFIG_CRYPTO_MANAGER=y<br>
CONFIG_CRYPTO_MANAGER2=y<br>
# CONFIG_CRYPTO_USER is not set<br>
CONFIG_CRYPTO_MANAGER_DISABLE_TESTS=y<br>
CONFIG_CRYPTO_GF128MUL=y<br>
CONFIG_CRYPTO_NULL=y<br>
# CONFIG_CRYPTO_PCRYPT is not set<br>
CONFIG_CRYPTO_WORKQUEUE=y<br>
CONFIG_CRYPTO_CRYPTD=y<br>
# CONFIG_CRYPTO_MCRYPTD is not set<br>
CONFIG_CRYPTO_AUTHENC=y<br>
CONFIG_CRYPTO_ABLK_HELPER=y<br>
<br>
#<br>
# Authenticated Encryption with Associated Data<br>
#<br>
# CONFIG_CRYPTO_CCM is not set<br>
# CONFIG_CRYPTO_GCM is not set<br>
CONFIG_CRYPTO_SEQIV=y<br>
<br>
#<br>
# Block modes<br>
#<br>
CONFIG_CRYPTO_CBC=y<br>
CONFIG_CRYPTO_CTR=y<br>
CONFIG_CRYPTO_CTS=y<br>
CONFIG_CRYPTO_ECB=y<br>
# CONFIG_CRYPTO_LRW is not set<br>
# CONFIG_CRYPTO_PCBC is not set<br>
CONFIG_CRYPTO_XTS=y<br>
<br>
#<br>
# Hash modes<br>
#<br>
CONFIG_CRYPTO_CMAC=y<br>
CONFIG_CRYPTO_HMAC=y<br>
CONFIG_CRYPTO_XCBC=y<br>
# CONFIG_CRYPTO_VMAC is not set<br>
<br>
#<br>
# Digest<br>
#<br>
CONFIG_CRYPTO_CRC32C=y<br>
CONFIG_CRYPTO_CRC32=y<br>
# CONFIG_CRYPTO_CRCT10DIF is not set<br>
# CONFIG_CRYPTO_GHASH is not set<br>
CONFIG_CRYPTO_MD4=y<br>
CONFIG_CRYPTO_MD5=y<br>
# CONFIG_CRYPTO_MICHAEL_MIC is not set<br>
# CONFIG_CRYPTO_RMD128 is not set<br>
# CONFIG_CRYPTO_RMD160 is not set<br>
# CONFIG_CRYPTO_RMD256 is not set<br>
# CONFIG_CRYPTO_RMD320 is not set<br>
CONFIG_CRYPTO_SHA1=y<br>
CONFIG_CRYPTO_SHA256=y<br>
CONFIG_CRYPTO_SHA512=y<br>
# CONFIG_CRYPTO_TGR192 is not set<br>
# CONFIG_CRYPTO_WP512 is not set<br>
<br>
#<br>
# Ciphers<br>
#<br>
CONFIG_CRYPTO_AES=y<br>
# CONFIG_CRYPTO_ANUBIS is not set<br>
CONFIG_CRYPTO_ARC4=y<br>
# CONFIG_CRYPTO_BLOWFISH is not set<br>
# CONFIG_CRYPTO_CAMELLIA is not set<br>
# CONFIG_CRYPTO_CAST5 is not set<br>
# CONFIG_CRYPTO_CAST6 is not set<br>
CONFIG_CRYPTO_DES=y<br>
# CONFIG_CRYPTO_FCRYPT is not set<br>
# CONFIG_CRYPTO_KHAZAD is not set<br>
# CONFIG_CRYPTO_SALSA20 is not set<br>
# CONFIG_CRYPTO_SEED is not set<br>
# CONFIG_CRYPTO_SERPENT is not set<br>
# CONFIG_CRYPTO_TEA is not set<br>
CONFIG_CRYPTO_TWOFISH=y<br>
CONFIG_CRYPTO_TWOFISH_COMMON=y<br>
<br>
#<br>
# Compression<br>
#<br>
CONFIG_CRYPTO_DEFLATE=y<br>
# CONFIG_CRYPTO_ZLIB is not set<br>
CONFIG_CRYPTO_LZO=y<br>
# CONFIG_CRYPTO_LZ4 is not set<br>
# CONFIG_CRYPTO_LZ4HC is not set<br>
<br>
#<br>
# Random Number Generation<br>
#<br>
CONFIG_CRYPTO_ANSI_CPRNG=y<br>
# CONFIG_CRYPTO_DRBG_MENU is not set<br>
# CONFIG_CRYPTO_USER_API_HASH is not set<br>
# CONFIG_CRYPTO_USER_API_SKCIPHER is not set<br>
CONFIG_CRYPTO_HASH_INFO=y<br>
CONFIG_CRYPTO_HW=y<br>
CONFIG_CRYPTO_DEV_QCE50=y<br>
# CONFIG_FIPS_ENABLE is not set<br>
CONFIG_CRYPTO_DEV_QCRYPTO=y<br>
CONFIG_CRYPTO_DEV_QCOM_MSM_QCE=y<br>
CONFIG_CRYPTO_DEV_QCEDEV=y<br>
CONFIG_CRYPTO_DEV_OTA_CRYPTO=y<br>
CONFIG_CRYPTO_DEV_QCOM_ICE=y<br>
# CONFIG_CRYPTO_DEV_CCP is not set<br>
CONFIG_ASYMMETRIC_KEY_TYPE=y<br>
CONFIG_ASYMMETRIC_PUBLIC_KEY_SUBTYPE=y<br>
CONFIG_PUBLIC_KEY_ALGO_RSA=y<br>
CONFIG_X509_CERTIFICATE_PARSER=y<br>
# CONFIG_PKCS7_MESSAGE_PARSER is not set<br>
CONFIG_ARM64_CRYPTO=y<br>
CONFIG_CRYPTO_SHA1_ARM64_CE=y<br>
CONFIG_CRYPTO_SHA2_ARM64_CE=y<br>
CONFIG_CRYPTO_GHASH_ARM64_CE=y<br>
CONFIG_CRYPTO_AES_ARM64_CE=y<br>
CONFIG_CRYPTO_AES_ARM64_CE_CCM=y<br>
CONFIG_CRYPTO_AES_ARM64_CE_BLK=y<br>
CONFIG_CRYPTO_AES_ARM64_NEON_BLK=y<br>
CONFIG_BINARY_PRINTF=y<br>
<br>
#<br>
# Library routines<br>
#<br>
CONFIG_RAID6_PQ=y<br>
CONFIG_BITREVERSE=y<br>
CONFIG_GENERIC_STRNCPY_FROM_USER=y<br>
CONFIG_GENERIC_STRNLEN_USER=y<br>
CONFIG_GENERIC_NET_UTILS=y<br>
CONFIG_GENERIC_PCI_IOMAP=y<br>
CONFIG_GENERIC_IOMAP=y<br>
CONFIG_GENERIC_IO=y<br>
CONFIG_ARCH_USE_CMPXCHG_LOCKREF=y<br>
CONFIG_CRC_CCITT=y<br>
CONFIG_CRC16=y<br>
# CONFIG_CRC_T10DIF is not set<br>
CONFIG_CRC_ITU_T=y<br>
CONFIG_CRC32=y<br>
# CONFIG_CRC32_SELFTEST is not set<br>
CONFIG_CRC32_SLICEBY8=y<br>
# CONFIG_CRC32_SLICEBY4 is not set<br>
# CONFIG_CRC32_SARWATE is not set<br>
# CONFIG_CRC32_BIT is not set<br>
# CONFIG_CRC7 is not set<br>
CONFIG_LIBCRC32C=y<br>
# CONFIG_CRC8 is not set<br>
CONFIG_AUDIT_GENERIC=y<br>
CONFIG_AUDIT_ARCH_COMPAT_GENERIC=y<br>
CONFIG_AUDIT_COMPAT_GENERIC=y<br>
# CONFIG_RANDOM32_SELFTEST is not set<br>
CONFIG_ZLIB_INFLATE=y<br>
CONFIG_ZLIB_DEFLATE=y<br>
CONFIG_LZO_COMPRESS=y<br>
CONFIG_LZO_DECOMPRESS=y<br>
# CONFIG_XZ_DEC is not set<br>
# CONFIG_XZ_DEC_BCJ is not set<br>
CONFIG_DECOMPRESS_GZIP=y<br>
CONFIG_DECOMPRESS_BZIP2=y<br>
CONFIG_DECOMPRESS_LZMA=y<br>
CONFIG_GENERIC_ALLOCATOR=y<br>
CONFIG_REED_SOLOMON=y<br>
CONFIG_REED_SOLOMON_ENC8=y<br>
CONFIG_REED_SOLOMON_DEC8=y<br>
CONFIG_TEXTSEARCH=y<br>
CONFIG_TEXTSEARCH_KMP=y<br>
CONFIG_TEXTSEARCH_BM=y<br>
CONFIG_TEXTSEARCH_FSM=y<br>
CONFIG_ASSOCIATIVE_ARRAY=y<br>
CONFIG_HAS_IOMEM=y<br>
CONFIG_HAS_IOPORT_MAP=y<br>
CONFIG_HAS_DMA=y<br>
CONFIG_CPU_RMAP=y<br>
CONFIG_DQL=y<br>
CONFIG_NLATTR=y<br>
CONFIG_ARCH_HAS_ATOMIC64_DEC_IF_POSITIVE=y<br>
# CONFIG_AVERAGE is not set<br>
CONFIG_CLZ_TAB=y<br>
# CONFIG_CORDIC is not set<br>
# CONFIG_DDR is not set<br>
CONFIG_MPILIB=y<br>
CONFIG_LIBFDT=y<br>
CONFIG_OID_REGISTRY=y<br>
CONFIG_UCS2_STRING=y<br>
CONFIG_ARCH_HAS_SG_CHAIN=y<br>
CONFIG_QMI_ENCDEC=y<br>
# CONFIG_QMI_ENCDEC_DEBUG is not set<br>
# CONFIG_STRICT_MEMORY_RWX is not set<br>
</details><br>


Далее очистим все сгенерированные файлы, конфиги, и бекапы c помощью Мистера Пропера)))
```console
HABUILD_SDK [mido]:~/hadk/kernel/xiaomi/msm8953$ ARCH=arm64 make mrproper
```
<details>
  CLEAN   scripts/basic<br>
  CLEAN   scripts/kconfig<br>
  CLEAN   .config .config.old<br>
</details><br>

А также предыдущие скомпилированные файлы
```console
HABUILD_SDK [mido]:~/hadk$ make clean
```
<details>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190629-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
Running kati to generate build-lineage_mido-clean.ninja...<br>
/home/stalker/hadk/out/build-lineage_mido-clean.ninja is missing, regenerating...<br>
============================================<br>
PLATFORM_VERSION_CODENAME=REL<br>
PLATFORM_VERSION=7.1.2<br>
LINEAGE_VERSION=14.1-20190629-UNOFFICIAL-mido<br>
TARGET_PRODUCT=lineage_mido<br>
TARGET_BUILD_VARIANT=userdebug<br>
TARGET_BUILD_TYPE=release<br>
TARGET_BUILD_APPS=<br>
TARGET_ARCH=arm64<br>
TARGET_ARCH_VARIANT=armv8-a<br>
TARGET_CPU_VARIANT=generic<br>
TARGET_2ND_ARCH=arm<br>
TARGET_2ND_ARCH_VARIANT=armv7-a-neon<br>
TARGET_2ND_CPU_VARIANT=cortex-a53<br>
HOST_ARCH=x86_64<br>
HOST_2ND_ARCH=x86<br>
HOST_OS=linux<br>
HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty<br>
HOST_CROSS_OS=windows<br>
HOST_CROSS_ARCH=x86<br>
HOST_CROSS_2ND_ARCH=x86_64<br>
HOST_BUILD_TYPE=release<br>
BUILD_ID=NJH47F<br>
OUT_DIR=/home/stalker/hadk/out<br>
============================================<br>
Starting build with ninja<br>
ninja: Entering directory `.'<br>
[100% 1/1] Entire build directory removed.<br>
<br>
#### make completed successfully (2 seconds) ####<br>
</details><br>

Пересоберем ядро
```console
HABUILD_SDK [mido]:~/hadk$ make -j$(nproc --all) hybris-hal
```
<details>
...<br>
#### make completed successfully (02:40 (mm:ss)) ####<br>
</details><br>

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

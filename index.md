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
<details>
```console
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
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/log.o build/kati/log.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/func.o build/kati/func.cc<br>
prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/io.o build/kati/io.cc<br>
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
build/core/Makefile:34: warning: overriding commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'<br>
build/core/base_rules.mk:320: warning: ignoring old commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'<br>
Starting build with ninja<br>
ninja: Entering directory `.'<br>
[  4% 286/5859] Building Kernel Config<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
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
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
scripts/kconfig/conf --savedefconfig=defconfig Kconfig<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
[ 10% 622/5859] Building Kernel Headers<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
#<br>
# configuration written to .config<br>
#<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/generated/uapi/linux/version.h<br>
  UPD     include/generated/uapi/linux/version.h<br>
  WRAP    arch/arm64/include/generated/asm/bugs.h<br>
  WRAP    arch/arm64/include/generated/asm/checksum.h<br>
  WRAP    arch/arm64/include/generated/asm/clkdev.h<br>
  WRAP    arch/arm64/include/generated/asm/cputime.h<br>
  WRAP    arch/arm64/include/generated/asm/current.h<br>
  WRAP    arch/arm64/include/generated/asm/delay.h<br>
  WRAP    arch/arm64/include/generated/asm/div64.h<br>
  WRAP    arch/arm64/include/generated/asm/dma.h<br>
  WRAP    arch/arm64/include/generated/asm/dma-contiguous.h<br>
  WRAP    arch/arm64/include/generated/asm/early_ioremap.h<br>
  WRAP    arch/arm64/include/generated/asm/emergency-restart.h<br>
  WRAP    arch/arm64/include/generated/asm/errno.h<br>
  WRAP    arch/arm64/include/generated/asm/ftrace.h<br>
  WRAP    arch/arm64/include/generated/asm/hash.h<br>
  WRAP    arch/arm64/include/generated/asm/hw_irq.h<br>
  WRAP    arch/arm64/include/generated/asm/ioctl.h<br>
  WRAP    arch/arm64/include/generated/asm/ioctls.h<br>
  WRAP    arch/arm64/include/generated/asm/ipcbuf.h<br>
  WRAP    arch/arm64/include/generated/asm/irq_regs.h<br>
  WRAP    arch/arm64/include/generated/asm/kdebug.h<br>
  WRAP    arch/arm64/include/generated/asm/kvm_para.h<br>
  WRAP    arch/arm64/include/generated/asm/kmap_types.h<br>
  WRAP    arch/arm64/include/generated/asm/local.h<br>
  WRAP    arch/arm64/include/generated/asm/local64.h<br>
  WRAP    arch/arm64/include/generated/asm/mcs_spinlock.h<br>
  WRAP    arch/arm64/include/generated/asm/mman.h<br>
  WRAP    arch/arm64/include/generated/asm/msgbuf.h<br>
  WRAP    arch/arm64/include/generated/asm/msi.h<br>
  WRAP    arch/arm64/include/generated/asm/mutex.h<br>
  WRAP    arch/arm64/include/generated/asm/pci.h<br>
  WRAP    arch/arm64/include/generated/asm/pci-bridge.h<br>
  WRAP    arch/arm64/include/generated/asm/poll.h<br>
  WRAP    arch/arm64/include/generated/asm/preempt.h<br>
  WRAP    arch/arm64/include/generated/asm/rwsem.h<br>
  WRAP    arch/arm64/include/generated/asm/resource.h<br>
  WRAP    arch/arm64/include/generated/asm/scatterlist.h<br>
  WRAP    arch/arm64/include/generated/asm/sections.h<br>
  WRAP    arch/arm64/include/generated/asm/segment.h<br>
  WRAP    arch/arm64/include/generated/asm/sembuf.h<br>
  WRAP    arch/arm64/include/generated/asm/serial.h<br>
  WRAP    arch/arm64/include/generated/asm/shmbuf.h<br>
  WRAP    arch/arm64/include/generated/asm/sizes.h<br>
  WRAP    arch/arm64/include/generated/asm/simd.h<br>
  WRAP    arch/arm64/include/generated/asm/socket.h<br>
  WRAP    arch/arm64/include/generated/asm/sockios.h<br>
  WRAP    arch/arm64/include/generated/asm/swab.h<br>
  WRAP    arch/arm64/include/generated/asm/switch_to.h<br>
  WRAP    arch/arm64/include/generated/asm/termios.h<br>
  WRAP    arch/arm64/include/generated/asm/termbits.h<br>
  WRAP    arch/arm64/include/generated/asm/topology.h<br>
  WRAP    arch/arm64/include/generated/asm/trace_clock.h<br>
  WRAP    arch/arm64/include/generated/asm/types.h<br>
  WRAP    arch/arm64/include/generated/asm/unaligned.h<br>
  WRAP    arch/arm64/include/generated/asm/user.h<br>
  WRAP    arch/arm64/include/generated/asm/vga.h<br>
  WRAP    arch/arm64/include/generated/asm/xor.h<br>
  WRAP    arch/arm64/include/generated/uapi/asm/kvm_para.h<br>
  HOSTCC  scripts/unifdef<br>
  INSTALL usr/include/drm/ (18 files)<br>
  INSTALL usr/include/asm-generic/ (35 files)<br>
  INSTALL usr/include/media/ (21 files)<br>
  INSTALL usr/include/misc/ (1 file)<br>
  INSTALL usr/include/mtd/ (5 files)<br>
  INSTALL usr/include/linux/caif/ (2 files)<br>
  INSTALL usr/include/linux/hdlc/ (1 file)<br>
  INSTALL usr/include/linux/byteorder/ (2 files)<br>
  INSTALL usr/include/linux/../../../usr/include/linux/staging/android/uapi/ (2 files)<br>
  INSTALL usr/include/scsi/fc/ (4 files)<br>
  INSTALL usr/include/sound/ (19 files)<br>
  INSTALL usr/include/linux/can/ (5 files)<br>
  INSTALL usr/include/rdma/ (6 files)<br>
  INSTALL usr/include/linux/dvb/ (8 files)<br>
  INSTALL usr/include/linux/hsi/ (1 file)<br>
  INSTALL usr/include/scsi/ufs/ (2 files)<br>
  INSTALL usr/include/linux/isdn/ (1 file)<br>
  INSTALL usr/include/video/ (5 files)<br>
  INSTALL usr/include/linux/mmc/ (3 files)<br>
  INSTALL usr/include/linux/netfilter/ipset/ (4 files)<br>
  INSTALL usr/include/linux/mfd/wcd9xxx/ (2 files)<br>
  INSTALL usr/include/scsi/ (5 files)<br>
  INSTALL usr/include/linux/netfilter_arp/ (2 files)<br>
  INSTALL usr/include/linux/netfilter_bridge/ (17 files)<br>
  INSTALL usr/include/linux/netfilter_ipv4/ (10 files)<br>
  INSTALL usr/include/linux/netfilter_ipv6/ (12 files)<br>
  INSTALL usr/include/linux/nfc/ (1 file)<br>
  INSTALL usr/include/xen/ (4 files)<br>
  INSTALL usr/include/linux/netfilter/ (85 files)<br>
  INSTALL usr/include/linux/nfsd/ (5 files)<br>
  INSTALL usr/include/linux/mfd/ (1 file)<br>
  INSTALL usr/include/uapi/ (0 file)<br>
  INSTALL usr/include/linux/raid/ (2 files)<br>
  INSTALL usr/include/linux/sunrpc/ (1 file)<br>
  INSTALL usr/include/linux/spi/ (1 file)<br>
  INSTALL usr/include/linux/tc_act/ (8 files)<br>
  INSTALL usr/include/linux/tc_ematch/ (4 files)<br>
  INSTALL usr/include/linux/usb/ (11 files)<br>
  INSTALL usr/include/linux/ (471 files)<br>
  INSTALL usr/include/linux/wimax/ (1 file)<br>
  INSTALL usr/include/asm/ (35 files)<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
[ 54% 3170/5859] target thumb C: libunwind_32 <= external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c<br>
external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c:75:10: warning: Implement get_list_addr(), please. [-W#pragma-messages]<br>
# pragma message("Implement get_list_addr(), please.")<br>
         ^<br>
1 warning generated.<br>
[ 59% 3460/5859] target  C: libunwind <= external/libunwind/src/ptrace/_UPT_access_fpreg.c<br>
external/libunwind/src/ptrace/_UPT_access_fpreg.c:113:10: warning: _UPT_access_fpreg is not implemented and not currently used. [-W#pragma-messages]<br>
# pragma message("_UPT_access_fpreg is not implemented and not currently used.")<br>
         ^<br>
1 warning generated.<br>
[ 59% 3465/5859] target  C: libunwind <= external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c<br>
external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c:75:10: warning: Implement get_list_addr(), please. [-W#pragma-messages]<br>
# pragma message("Implement get_list_addr(), please.")<br>
         ^<br>
1 warning generated.<br>
[ 62% 3663/5859] target  C: libext2_uuid <= external/e2fsprogs/lib/uuid/gen_uuid.c<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:224:39: warning: unused parameter 'node_id' [-Wunused-parameter]<br>
static int get_node_id(unsigned char *node_id)<br>
                                      ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:36: warning: unused parameter 'op' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                   ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:47: warning: unused parameter 'out' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                              ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:57: warning: unused parameter 'num' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                                        ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:423:16: warning: unused function 'read_all' [-Wunused-function]<br>
static ssize_t read_all(int fd, char *buf, size_t count)<br>
               ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:450:13: warning: unused function 'close_all_fds' [-Wunused-function]<br>
static void close_all_fds(void)<br>
            ^<br>
6 warnings generated.<br>
[ 62% 3668/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio.c<br>
external/e2fsprogs/lib/quota/quotaio.c:100:48: warning: unused parameter 'fs' [-Wunused-parameter]<br>
static int compute_num_blocks_proc(ext2_filsys fs, blk64_t *blocknr,<br>
                                               ^<br>
external/e2fsprogs/lib/quota/quotaio.c:100:61: warning: unused parameter 'blocknr' [-Wunused-parameter]<br>
static int compute_num_blocks_proc(ext2_filsys fs, blk64_t *blocknr,<br>
                                                            ^<br>
2 warnings generated.<br>
[ 62% 3675/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/mkquota.c<br>
external/e2fsprogs/lib/quota/mkquota.c:53:54: warning: unused parameter 'fmt' [-Wunused-parameter]<br>
int quota_file_exists(ext2_filsys fs, int qtype, int fmt)<br>
                                                     ^<br>
external/e2fsprogs/lib/quota/mkquota.c:291:76: warning: unused parameter 'ino' [-Wunused-parameter]<br>
void quota_data_add(quota_ctx_t qctx, struct ext2_inode *inode, ext2_ino_t ino,<br>
                                                                           ^<br>
external/e2fsprogs/lib/quota/mkquota.c:317:76: warning: unused parameter 'ino' [-Wunused-parameter]<br>
void quota_data_sub(quota_ctx_t qctx, struct ext2_inode *inode, ext2_ino_t ino,<br>
                                                                           ^<br>
external/e2fsprogs/lib/quota/mkquota.c:343:21: warning: unused parameter 'ino' [-Wunused-parameter]<br>
                       ext2_ino_t ino, int adjust)<br>
                                  ^<br>
4 warnings generated.<br>
[ 62% 3681/5859] target  C: libext2_blkid <= external/e2fsprogs/lib/blkid/probe.c<br>
external/e2fsprogs/lib/blkid/probe.c:883:42: warning: unused parameter 'probe' [-Wunused-parameter]<br>
static int probe_zfs(struct blkid_probe *probe, struct blkid_magic *id,<br>
                                         ^<br>
external/e2fsprogs/lib/blkid/probe.c:883:69: warning: unused parameter 'id' [-Wunused-parameter]<br>
static int probe_zfs(struct blkid_probe *probe, struct blkid_magic *id,<br>
                                                                    ^<br>
external/e2fsprogs/lib/blkid/probe.c:884:23: warning: unused parameter 'buf' [-Wunused-parameter]<br>
                     unsigned char *buf)<br>
                                    ^<br>
external/e2fsprogs/lib/blkid/probe.c:1384:24: warning: unused parameter 'id' [-Wunused-parameter]<br>
                        struct blkid_magic *id,<br>
                                            ^<br>
external/e2fsprogs/lib/blkid/probe.c:1400:33: warning: unused parameter 'id' [-Wunused-parameter]<br>
            struct blkid_magic *id,<br>
                                ^<br>
5 warnings generated.<br>
[ 62% 3682/5859] target  C: libext2_blkid <= external/e2fsprogs/lib/blkid/probe_exfat.c<br>
external/e2fsprogs/lib/blkid/probe_exfat.c:177:24: warning: passing 'char [128]' to parameter of type 'unsigned char *' converts between pointers to integer types with different sign [-Wpointer-sign]<br>
                unicode_16le_to_utf8(utf8_label, sizeof(utf8_label), label->name, label->length * 2);<br>
                                     ^~~~~~~~~~<br>
external/e2fsprogs/lib/blkid/probe_exfat.c:128:49: note: passing argument to parameter 'str' here<br>
static void unicode_16le_to_utf8(unsigned char *str, int out_len,<br>
                                                ^<br>
1 warning generated.<br>
[ 62% 3690/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio_tree.c<br>
external/e2fsprogs/lib/quota/quotaio_tree.c:38:16: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
        for (i = 0; i < info->dqi_entry_size; i++)<br>
                    ~ ^ ~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/quota/quotaio_tree.c:626:16: warning: comparison of integers of different signs: 'uint' (aka 'unsigned int') and 'int' [-Wsign-compare]<br>
        for (i = 0; i < blocks; i++)<br>
                    ~ ^ ~~~~~~<br>
2 warnings generated.<br>
[ 63% 3694/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio_v2.c<br>
external/e2fsprogs/lib/quota/quotaio_v2.c:160:40: warning: comparison of integers of different signs: '__u32' (aka 'unsigned int') and 'int' [-Wsign-compare]<br>
        if (ext2fs_le32_to_cpu(dqh.dqh_magic) != file_magics[type]) {<br>
            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/quota/quotaio_v2.c:161:41: warning: comparison of integers of different signs: '__u32' (aka 'unsigned int') and 'int' [-Wsign-compare]<br>
                if (ext2fs_be32_to_cpu(dqh.dqh_magic) == file_magics[type])<br>
                    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/quota/quotaio_v2.c:278:43: warning: unused parameter 'h' [-Wunused-parameter]<br>
static int v2_report(struct quota_handle *h, int verbose)<br>
                                          ^<br>
external/e2fsprogs/lib/quota/quotaio_v2.c:278:50: warning: unused parameter 'verbose' [-Wunused-parameter]<br>
static int v2_report(struct quota_handle *h, int verbose)<br>
                                                 ^<br>
4 warnings generated.<br>
[ 63% 3695/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/../../e2fsck/dict.c<br>
external/e2fsprogs/lib/quota/../../e2fsck/dict.c:21:9: warning: 'NDEBUG' macro redefined [-Wmacro-redefined]<br>
#define NDEBUG<br>
        ^<br>
<command line>:7:9: note: previous definition is here<br>
#define NDEBUG 1<br>
        ^<br>
1 warning generated.<br>
[ 70% 4107/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/csum.c<br>
external/e2fsprogs/lib/ext2fs/csum.c:37:9: warning: unused variable 'offset' [-Wunused-variable]<br>
        size_t offset;<br>
               ^<br>
external/e2fsprogs/lib/ext2fs/csum.c:40:6: warning: variable 'crc' is used uninitialized whenever 'if' condition is false [-Wsometimes-uninitialized]<br>
        if (fs->super->s_feature_ro_compat & EXT4_FEATURE_RO_COMPAT_GDT_CSUM) {<br>
            ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/csum.c:82:9: note: uninitialized use occurs here<br>
        return crc;<br>
               ^~~<br>
external/e2fsprogs/lib/ext2fs/csum.c:40:2: note: remove the 'if' if its condition is always true<br>
        if (fs->super->s_feature_ro_compat & EXT4_FEATURE_RO_COMPAT_GDT_CSUM) {<br>
        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/csum.c:38:11: note: initialize the variable 'crc' to silence this warning<br>
        __u16 crc;<br>
                 ^<br>
                  = 0<br>
2 warnings generated.<br>
[ 70% 4116/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/ext_attr.c<br>
external/e2fsprogs/lib/ext2fs/ext_attr.c:154:1: warning: all paths through this function will call itself [-Winfinite-recursion]<br>
{<br>
^<br>
1 warning generated.<br>
[ 70% 4133/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/inline.c<br>
external/e2fsprogs/lib/ext2fs/inline.c:47:12: warning: unused variable 'retval' [-Wunused-variable]<br>
        errcode_t retval;<br>
                  ^<br>
1 warning generated.<br>
[ 70% 4134/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/ismounted.c<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:49:43: warning: unused parameter 'mnt_fsname' [-Wunused-parameter]<br>
static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,<br>
                                          ^<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:49:61: warning: unused parameter 'mnt_rdev' [-Wunused-parameter]<br>
static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,<br>
                                                            ^<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:50:11: warning: unused parameter 'file_dev' [-Wunused-parameter]<br>
                                dev_t file_dev, ino_t file_ino)<br>
                                      ^<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:50:27: warning: unused parameter 'file_ino' [-Wunused-parameter]<br>
                                dev_t file_dev, ino_t file_ino)<br>
                                                      ^<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:365:3: warning: "Can't use getmntent or getmntinfo to check for mounted filesystems!" [-W#warnings]<br>
 #warning "Can't use getmntent or getmntinfo to check for mounted filesystems!"<br>
  ^<br>
external/e2fsprogs/lib/ext2fs/ismounted.c:49:12: warning: unused function 'check_loop_mounted' [-Wunused-function]<br>
static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,<br>
           ^<br>
6 warnings generated.<br>
[ 70% 4154/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/tdb.c<br>
external/e2fsprogs/lib/ext2fs/tdb.c:411:13: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
            (ltype == tdb->global_lock.ltype || ltype == F_RDLCK)) {<br>
             ~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:506:13: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
            (ltype == tdb->global_lock.ltype || ltype == F_RDLCK)) {<br>
             ~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:627:55: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]<br>
        if (tdb->global_lock.count && tdb->global_lock.ltype == ltype) {<br>
                                      ~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:671:29: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]<br>
        if (tdb->global_lock.ltype != ltype || tdb->global_lock.count == 0) {<br>
            ~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:854:17: warning: comparison of integers of different signs: 'off_t' (aka 'long') and 'size_t' (aka 'unsigned long') [-Wsign-compare]<br>
        if (st.st_size < (size_t)len) {<br>
            ~~~~~~~~~~ ^ ~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1535:72: warning: unused parameter 'probe' [-Wunused-parameter]<br>
static int transaction_oob(struct tdb_context *tdb, tdb_off_t len, int probe)<br>
                                                                       ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1561:51: warning: unused parameter 'tdb' [-Wunused-parameter]<br>
static int transaction_brlock(struct tdb_context *tdb, tdb_off_t offset,<br>
                                                  ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1561:66: warning: unused parameter 'offset' [-Wunused-parameter]<br>
static int transaction_brlock(struct tdb_context *tdb, tdb_off_t offset,<br>
                                                                 ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1562:14: warning: unused parameter 'rw_type' [-Wunused-parameter]<br>
                              int rw_type, int lck_type, int probe, size_t len)<br>
                                  ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1562:27: warning: unused parameter 'lck_type' [-Wunused-parameter]<br>
                              int rw_type, int lck_type, int probe, size_t len)<br>
                                               ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1562:41: warning: unused parameter 'probe' [-Wunused-parameter]<br>
                              int rw_type, int lck_type, int probe, size_t len)<br>
                                                             ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1562:55: warning: unused parameter 'len' [-Wunused-parameter]<br>
                              int rw_type, int lck_type, int probe, size_t len)<br>
                                                                           ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1743:64: warning: unused parameter 'offset' [-Wunused-parameter]<br>
static int transaction_sync(struct tdb_context *tdb, tdb_off_t offset, tdb_len_t length)<br>
                                                               ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:1743:82: warning: unused parameter 'length' [-Wunused-parameter]<br>
static int transaction_sync(struct tdb_context *tdb, tdb_off_t offset, tdb_len_t length)<br>
                                                                                 ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:3004:12: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
        for (i=0;i<tdb->header.hash_size;i++) {<br>
                 ~^~~~~~~~~~~~~~~~~~~~~~<br>
external/e2fsprogs/lib/ext2fs/tdb.c:3097:63: warning: unused parameter 'private_data' [-Wunused-parameter]<br>
static int tdb_key_compare(TDB_DATA key, TDB_DATA data, void *private_data)<br>
                                                              ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:3801:45: warning: unused parameter 'tdb' [-Wunused-parameter]<br>
static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)<br>
                                            ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:3801:71: warning: unused parameter 'level' [-Wunused-parameter]<br>
static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)<br>
                                                                      ^<br>
external/e2fsprogs/lib/ext2fs/tdb.c:3801:90: warning: unused parameter 'fmt' [-Wunused-parameter]<br>
static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)<br>
                                                                                         ^<br>
19 warnings generated.<br>
[ 70% 4155/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/undo_io.c<br>
external/e2fsprogs/lib/ext2fs/undo_io.c:105:1: warning: missing field 'discard' initializer [-Wmissing-field-initializers]<br>
};<br>
^<br>
1 warning generated.<br>
[ 70% 4159/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/symlink.c<br>
external/e2fsprogs/lib/ext2fs/symlink.c:33:23: warning: unused variable 'handle' [-Wunused-variable]<br>
        ext2_extent_handle_t    handle;<br>
                                ^<br>
1 warning generated.<br>
[ 71% 4168/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/unix_io.c<br>
external/e2fsprogs/lib/ext2fs/unix_io.c:137:1: warning: missing field 'reserved' initializer [-Wmissing-field-initializers]<br>
};<br>
^<br>
1 warning generated.<br>
[ 71% 4169/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/test_io.c<br>
external/e2fsprogs/lib/ext2fs/test_io.c:100:1: warning: missing field 'reserved' initializer [-Wmissing-field-initializers]<br>
};<br>
^<br>
1 warning generated.<br>
[ 71% 4194/5859] target  C: libext4_utils_static <= system/extras/ext4_utils/make_ext4fs.c<br>
system/extras/ext4_utils/make_ext4fs.c:537:15: warning: comparison of integers of different signs: 'int' and 'u32' (aka 'unsigned int') [-Wsign-compare]<br>
        for(i = 0; i < aux_info.groups; i++) {<br>
                   ~ ^ ~~~~~~~~~~~~~~~<br>
system/extras/ext4_utils/make_ext4fs.c:631:22: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
                                if (min_bg_bound >= start_block - bg_first_block ||<br>
                                    ~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~<br>
system/extras/ext4_utils/make_ext4fs.c:632:19: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]<br>
                                        max_bg_bound <= end_block - bg_first_block) {<br>
                                        ~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~<br>
system/extras/ext4_utils/make_ext4fs.c:650:16: warning: comparison of integers of different signs: 'int' and 'u32' (aka 'unsigned int') [-Wsign-compare]<br>
        for (i = 0; i < aux_info.groups; i++) {<br>
                    ~ ^ ~~~~~~~~~~~~~~~<br>
4 warnings generated.<br>
[ 71% 4197/5859] target  C: libext4_utils_static <= system/extras/ext4_utils/allocate.c<br>
system/extras/ext4_utils/allocate.c:362:17: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]<br>
                for (j = 1; j < bgs[i].chunk_count; j++) {<br>
                            ~ ^ ~~~~~~~~~~~~~~~~~~<br>
1 warning generated.<br>
[ 72% 4231/5859] target  C: libdrm <= external/libdrm/xf86drm.c<br>
external/libdrm/xf86drm.c:895:36: warning: unused parameter 'fd' [-Wunused-parameter]<br>
drmVersionPtr drmGetLibVersion(int fd)<br>
                                   ^<br>
external/libdrm/xf86drm.c:1094:19: warning: implicit conversion from enumeration type 'drmMapType' to different enumeration type 'enum drm_map_type' [-Wenum-conversion]<br>
    map.type    = type;<br>
                ~ ^~~~<br>
external/libdrm/xf86drm.c:1095:19: warning: implicit conversion from enumeration type 'drmMapFlags' to different enumeration type 'enum drm_map_flags' [-Wenum-conversion]<br>
    map.flags   = flags;<br>
                ~ ^~~~~<br>
external/libdrm/xf86drm.c:1419:36: warning: implicit conversion from enumeration type 'drmDMAFlags' to different enumeration type 'enum drm_dma_flags' [-Wenum-conversion]<br>
    dma.flags           = request->flags;<br>
                        ~ ~~~~~~~~~^~~~~<br>
external/libdrm/xf86drm.c:2309:19: warning: implicit conversion from enumeration type 'enum drm_map_type' to different enumeration type 'drmMapType' [-Wenum-conversion]<br>
    *type   = map.type;<br>
            ~ ~~~~^~~~<br>
external/libdrm/xf86drm.c:2310:19: warning: implicit conversion from enumeration type 'enum drm_map_flags' to different enumeration type 'drmMapFlags' [-Wenum-conversion]<br>
    *flags  = map.flags;<br>
            ~ ~~~~^~~~~<br>
external/libdrm/xf86drm.c:2614:23: warning: unused parameter 'unused' [-Wunused-parameter]<br>
int drmOpenOnce(void *unused,<br>
                      ^<br>
7 warnings generated.<br>
[ 72% 4241/5859] target  C: libdrm <= external/libdrm/xf86drmMode.c<br>
external/libdrm/xf86drmMode.c:1342:38: warning: missing field 'count_objs' initializer [-Wmissing-field-initializers]<br>
        struct drm_mode_atomic atomic = { 0 };<br>
                                            ^<br>
1 warning generated.<br>
[ 74% 4344/5859] Target buildinfo: /home/stalker/hadk/out/target/product/mido/obj/ETC/system_build_prop_intermediates/build.prop<br>
Target buildinfo from: device/xiaomi/mido/system.prop<br>
[ 75% 4401/5859] host C: libsepol <= external/selinux/libsepol/cil/src/cil_mem.c<br>
external/selinux/libsepol/cil/src/cil_mem.c:109:7: warning: implicit declaration of function 'vasprintf' is invalid in C99 [-Wimplicit-function-declaration]<br>
        rc = vasprintf(strp, fmt, ap);<br>
             ^<br>
1 warning generated.<br>
[ 75% 4419/5859] host C: libsepol <= /home/stalker/hadk/out/host/l...86/obj/STATIC_LIBRARIES/libsepol_intermediates/cil/src/cil_lexer.c<br>
/home/stalker/hadk/out/host/linux-x86/obj/STATIC_LIBRARIES/libsepol_intermediates/cil/src/cil_lexer.c:1593:1: warning: function 'yy_fatal_error' could be declared with attribute 'noreturn' [-Wmissing-noreturn]<br>
{<br>
^<br>
1 warning generated.<br>
[ 76% 4477/5859] target  C: libext2_uuid_static <= external/e2fsprogs/lib/uuid/gen_uuid.c<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:224:39: warning: unused parameter 'node_id' [-Wunused-parameter]<br>
static int get_node_id(unsigned char *node_id)<br>
                                      ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:36: warning: unused parameter 'op' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                   ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:47: warning: unused parameter 'out' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                              ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:480:57: warning: unused parameter 'num' [-Wunused-parameter]<br>
static int get_uuid_via_daemon(int op, uuid_t out, int *num)<br>
                                                        ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:423:16: warning: unused function 'read_all' [-Wunused-function]<br>
static ssize_t read_all(int fd, char *buf, size_t count)<br>
               ^<br>
external/e2fsprogs/lib/uuid/gen_uuid.c:450:13: warning: unused function 'close_all_fds' [-Wunused-function]<br>
static void close_all_fds(void)<br>
            ^<br>
6 warnings generated.<br>
[ 81% 4752/5859] build /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/sepolicy<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  loading policy configuration from /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/policy.conf<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  policy configuration loaded<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  writing binary representation (version 30) to /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/sepolicy.tmp<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  loading policy configuration from /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/policy.conf.dontaudit<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  policy configuration loaded<br>
/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  writing binary representation (version 30) to /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates//sepolicy.dontaudit<br>
[ 91% 5360/5859] target  C: libclearsilverregex <= external/busybox/android/regex/bb_regex.c<br>
external/busybox/android/regex/bb_regex.c:5476:20: warning: unused parameter 'preg' [-Wunused-parameter]<br>
    const regex_t *preg;<br>
                   ^<br>
1 warning generated.<br>
[ 92% 5427/5859] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-boot_intermediates/init<br>
Fixing mount-points for device mido<br>
[ 92% 5427/5859] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-recovery_intermediates/init<br>
Fixing mount-points for device mido<br>
[ 92% 5435/5859] -e Prepare config for busybox binary<br>
find: `/home/stalker/hadk/out/target/product/mido/obj/EXECUTABLES/busybox_intermediates': No such file or directory<br>
make: Entering directory `/home/stalker/hadk/external/busybox'<br>
  Using /home/stalker/hadk/external/busybox as source for busybox<br>
  GEN     /home/stalker/hadk/out/target/product/mido/obj/busybox/full/Makefile<br>
  GEN     include/applets.h<br>
  GEN     include/usage.h<br>
  GEN     applets/Kbuild<br>
  GEN     e2fsprogs/Kbuild<br>
  GEN     e2fsprogs/Config.in<br>
  GEN     e2fsprogs/old_e2fsprogs/Kbuild<br>
  GEN     e2fsprogs/old_e2fsprogs/Config.in<br>
  GEN     e2fsprogs/old_e2fsprogs/e2p/Kbuild<br>
  GEN     e2fsprogs/old_e2fsprogs/uuid/Kbuild<br>
  GEN     e2fsprogs/old_e2fsprogs/ext2fs/Kbuild<br>
  GEN     e2fsprogs/old_e2fsprogs/blkid/Kbuild<br>
  GEN     scripts/Kbuild<br>
  GEN     debianutils/Kbuild<br>
  GEN     debianutils/Config.in<br>
  GEN     procps/Kbuild<br>
  GEN     procps/Config.in<br>
  GEN     sysklogd/Kbuild<br>
  GEN     sysklogd/Config.in<br>
  GEN     editors/Kbuild<br>
  GEN     editors/Config.in<br>
  GEN     mailutils/Kbuild<br>
  GEN     mailutils/Config.in<br>
  GEN     shell/Kbuild<br>
  GEN     shell/Config.in<br>
  GEN     libbb/Kbuild<br>
  GEN     libbb/Config.in<br>
  GEN     modutils/Kbuild<br>
  GEN     modutils/Config.in<br>
  GEN     runit/Kbuild<br>
  GEN     runit/Config.in<br>
  GEN     coreutils/Kbuild<br>
  GEN     coreutils/Config.in<br>
  GEN     coreutils/libcoreutils/Kbuild<br>
  GEN     miscutils/Kbuild<br>
  GEN     miscutils/Config.in<br>
  GEN     loginutils/Kbuild<br>
  GEN     loginutils/Config.in<br>
  GEN     printutils/Kbuild<br>
  GEN     printutils/Config.in<br>
  GEN     findutils/Kbuild<br>
  GEN     findutils/Config.in<br>
  GEN     util-linux/Kbuild<br>
  GEN     util-linux/Config.in<br>
  GEN     util-linux/volume_id/Kbuild<br>
  GEN     util-linux/volume_id/Config.in<br>
  GEN     console-tools/Kbuild<br>
  GEN     console-tools/Config.in<br>
  GEN     init/Kbuild<br>
  GEN     init/Config.in<br>
  GEN     libpwdgrp/Kbuild<br>
  GEN     selinux/Kbuild<br>
  GEN     selinux/Config.in<br>
  GEN     networking/Kbuild<br>
  GEN     networking/Config.in<br>
  GEN     networking/udhcp/Kbuild<br>
  GEN     networking/udhcp/Config.in<br>
  GEN     networking/libiproute/Kbuild<br>
  GEN     archival/Kbuild<br>
  GEN     archival/Config.in<br>
  GEN     archival/libarchive/Kbuild<br>
  HOSTCC  scripts/basic/fixdep<br>
  HOSTCC  scripts/basic/split-include<br>
/home/stalker/hadk/external/busybox/scripts/basic/split-include.c: In function 'main':<br>
/home/stalker/hadk/external/busybox/scripts/basic/split-include.c:134:11: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]<br>
      fgets(old_line, buffer_size, fp_target);<br>
           ^<br>
  HOSTCC  scripts/basic/docproc<br>
  GEN     /home/stalker/hadk/out/target/product/mido/obj/busybox/full/Makefile<br>
  HOSTCC  scripts/kconfig/conf.o<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c: In function 'conf_askvalue':<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c:106:8: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]<br>
   fgets(line, 128, stdin);<br>
        ^<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c: In function 'conf_choice':<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c:354:9: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]<br>
    fgets(line, 128, stdin);<br>
         ^<br>
  HOSTCC  scripts/kconfig/kxgettext.o<br>
  HOSTCC  scripts/kconfig/mconf.o<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c: In function 'show_textbox':<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c:847:7: warning: ignoring return value of 'write', declared with attribute warn_unused_result [-Wunused-result]<br>
  write(fd, text, strlen(text));<br>
       ^<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c: In function 'exec_conf':<br>
/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c:481:6: warning: ignoring return value of 'pipe', declared with attribute warn_unused_result [-Wunused-result]<br>
  pipe(pipefd);<br>
      ^<br>
  SHIPPED scripts/kconfig/zconf.tab.c<br>
  SHIPPED scripts/kconfig/lex.zconf.c<br>
  SHIPPED scripts/kconfig/zconf.hash.c<br>
  HOSTCC  scripts/kconfig/zconf.tab.o<br>
  HOSTLD  scripts/kconfig/conf<br>
scripts/kconfig/conf -s Config.in<br>
#<br>
# using defaults found in .config<br>
#<br>
  SPLIT   include/autoconf.h -> include/config/*<br>
  GEN     include/bbconfigopts.h<br>
  HOSTCC  applets/usage<br>
/home/stalker/hadk/external/busybox/applets/usage.c: In function 'main':<br>
/home/stalker/hadk/external/busybox/applets/usage.c:52:8: warning: ignoring return value of 'write', declared with attribute warn_unused_result [-Wunused-result]<br>
   write(STDOUT_FILENO, usage_array[i].usage, strlen(usage_array[i].usage) + 1);<br>
        ^<br>
  GEN     include/usage_compressed.h<br>
  HOSTCC  applets/applet_tables<br>
/home/stalker/hadk/external/busybox/applets/applet_tables.c: In function 'main':<br>
/home/stalker/hadk/external/busybox/applets/applet_tables.c:144:9: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]<br>
    fgets(line_old, sizeof(line_old), fp);<br>
         ^<br>
  GEN     include/applet_tables.h<br>
  CC      applets/applets.o<br>
  LD      applets/built-in.o<br>
  HOSTCC  applets/usage_pod<br>
make: Leaving directory `/home/stalker/hadk/external/busybox'<br>
[ 93% 5471/5859] target  C: static_busybox <= external/busybox/archival/unzip.c<br>
external/busybox/archival/unzip.c: In function 'unzip_extract':<br>
external/busybox/archival/unzip.c:295:35: warning: comparison of promoted ~unsigned with unsigned [-Wsign-compare]<br>
   if (zip_header->formatted.crc32 != (aux.crc32 ^ 0xffffffffL)) {<br>
                                   ^<br>
[ 95% 5598/5859] target  C: static_busybox <= external/busybox/libbb/change_identity.c<br>
external/busybox/libbb/change_identity.c: In function 'change_identity':<br>
external/busybox/libbb/change_identity.c:38:2: warning: 'endgrent' is deprecated (declared at bionic/libc/include/grp.h:59): endgrent is meaningless on Android [-Wdeprecated-declarations]<br>
  endgrent(); /* helps to close a fd used internally by libc */<br>
  ^<br>
[ 97% 5730/5859] target  C: static_busybox <= external/busybox/modutils/modutils.c<br>
external/busybox/modutils/modutils.c: In function 'try_to_mmap_module':<br>
external/busybox/modutils/modutils.c:116:17: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (st.st_size <= *image_size_p) {<br>
                 ^<br>
[ 98% 5757/5859] target  C: static_busybox <= external/busybox/networking/libiproute/libnetlink.c<br>
external/busybox/networking/libiproute/libnetlink.c: In function 'rta_addattr32':<br>
external/busybox/networking/libiproute/libnetlink.c:362:36: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (RTA_ALIGN(rta->rta_len) + len > maxlen) {<br>
                                    ^<br>
external/busybox/networking/libiproute/libnetlink.c: In function 'rta_addattr_l':<br>
external/busybox/networking/libiproute/libnetlink.c:378:36: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (RTA_ALIGN(rta->rta_len) + len > maxlen) {<br>
                                    ^<br>
[ 99% 5844/5859] target  C: static_busybox <= external/busybox/networking/udhcp/d6_dhcpc.c<br>
external/busybox/networking/udhcp/d6_dhcpc.c: In function 'd6_store_blob':<br>
external/busybox/networking/udhcp/d6_dhcpc.c:127:13: warning: pointer of type 'void *' used in arithmetic [-Wpointer-arith]<br>
  return dst + len;<br>
             ^<br>
external/busybox/networking/udhcp/d6_dhcpc.c: In function 'd6_recv_raw_packet':<br>
external/busybox/networking/udhcp/d6_dhcpc.c:605:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (bytes < sizeof(packet.ip6) + ntohs(packet.ip6.ip6_plen)) {<br>
            ^<br>
[ 99% 5845/5859] target  C: static_busybox <= external/busybox/networking/udhcp/dhcpd.c<br>
external/busybox/networking/udhcp/dhcpd.c: In function 'udhcpd_main':<br>
external/busybox/networking/udhcp/dhcpd.c:306:8: warning: 'str_I' is used uninitialized in this function [-Wuninitialized]<br>
  char *str_I = str_I;<br>
        ^<br>
external/busybox/networking/udhcp/dhcpd.c:599:14: warning: 'requested_nip' may be used uninitialized in this function [-Wmaybe-uninitialized]<br>
    if (lease && requested_nip == lease->lease_nip) {<br>
              ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/domain_codec.c<br>
external/busybox/networking/udhcp/domain_codec.c: In function 'dname_dec':<br>
external/busybox/networking/udhcp/domain_codec.c:50:17: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
   while (crtpos < clen) {<br>
                 ^<br>
external/busybox/networking/udhcp/domain_codec.c:55:20: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
     if (crtpos + 2 > clen) /* no offset to jump to? abort */<br>
                    ^<br>
external/busybox/networking/udhcp/domain_codec.c:63:25: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
     if (crtpos + *c + 1 > clen) /* label too long? abort */<br>
                         ^<br>
external/busybox/networking/udhcp/domain_codec.c:33:8: warning: 'ret' is used uninitialized in this function [-Wuninitialized]<br>
  char *ret = ret; /* for compiler */<br>
        ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/packet.c<br>
external/busybox/networking/udhcp/packet.c: In function 'udhcp_recv_kernel_packet':<br>
external/busybox/networking/udhcp/packet.c:92:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (bytes < offsetof(struct dhcp_packet, options)<br>
            ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/d6_packet.c<br>
external/busybox/networking/udhcp/d6_packet.c: In function 'd6_recv_kernel_packet':<br>
external/busybox/networking/udhcp/d6_packet.c:42:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
  if (bytes < offsetof(struct d6_packet, d6_options)) {<br>
            ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/dhcpc.c<br>
external/busybox/networking/udhcp/dhcpc.c: In function 'udhcpc_main':<br>
external/busybox/networking/udhcp/dhcpc.c:1244:11: warning: 'xid' is used uninitialized in this function [-Wuninitialized]<br>
  uint32_t xid = xid; /* for compiler */<br>
           ^<br>
external/busybox/networking/udhcp/dhcpc.c:1582:4: warning: 'server_addr' may be used uninitialized in this function [-Wmaybe-uninitialized]<br>
    perform_release(server_addr, requested_ip);<br>
    ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/signalpipe.c<br>
In file included from bionic/libc/include/unistd.h:35:0,<br>
                 from external/busybox/include/platform.h:299,<br>
                 from external/busybox/include/libbb.h:13,<br>
                 from external/busybox/networking/udhcp/common.h:11,<br>
                 from external/busybox/networking/udhcp/signalpipe.c:21:<br>
external/busybox/networking/udhcp/signalpipe.c: In function 'udhcp_sp_read':<br>
external/busybox/networking/udhcp/signalpipe.c:70:32: warning: passing argument 2 of '__FD_ISSET_chk' discards 'const' qualifier from pointer target type<br>
  if (!FD_ISSET(signal_pipe.rd, rfds))<br>
                                ^<br>
bionic/libc/include/sys/select.h:62:13: note: expected 'struct fd_set *' but argument is of type 'const struct fd_set *'<br>
 extern int  __FD_ISSET_chk(int, fd_set*, size_t);<br>
             ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/leases.c<br>
external/busybox/networking/udhcp/leases.c: In function 'add_lease':<br>
external/busybox/networking/udhcp/leases.c:65:21: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]<br>
    if (hostname_len > sizeof(oldest->hostname))<br>
                     ^<br>
[ 99% 5846/5859] target  C: static_busybox <= external/busybox/util-linux/mdev.c<br>
external/busybox/util-linux/mdev.c: In function 'make_device':<br>
external/busybox/util-linux/mdev.c:772:8: warning: 'aliaslink' may be used uninitialized in this function [-Wmaybe-uninitialized]<br>
     if (aliaslink == '>') {<br>
        ^<br>
[ 99% 5853/5859] Making initramfs : /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-boot_intermediates/boot-initramfs.gz<br>
2953 blocks<br>
[ 99% 5853/5859] Making initramfs : /home/stalker/hadk/out/target/.../mido/obj/ROOT/hybris-recovery_intermediates/recovery-initramfs.gz<br>
2953 blocks<br>
[ 99% 5853/5859] Building Kernel<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  GEN     ./Makefile<br>
scripts/kconfig/conf --silentoldconfig Kconfig<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/config/kernel.release<br>
  GEN     ./Makefile<br>
  CHK     include/generated/uapi/linux/version.h<br>
  HOSTCC  scripts/basic/bin2c<br>
  HOSTCC  scripts/kallsyms<br>
  HOSTCC  scripts/recordmcount<br>
  HOSTCC  scripts/sortextable<br>
  HOSTCC  scripts/dtc/dtc.o<br>
  CC      scripts/mod/empty.o<br>
  HOSTCC  scripts/asn1_compiler<br>
  HOSTCC  scripts/mod/mk_elfconfig<br>
  HOSTCC  scripts/dtc/flattree.o<br>
  CC      scripts/mod/devicetable-offsets.s<br>
  HOSTCC  scripts/dtc/fstree.o<br>
  HOSTCC  scripts/selinux/genheaders/genheaders<br>
  HOSTCC  scripts/selinux/mdp/mdp<br>
  HOSTCC  scripts/dtc/data.o<br>
  HOSTCC  scripts/dtc/livetree.o<br>
  HOSTCC  scripts/dtc/treesource.o<br>
  HOSTCC  scripts/dtc/srcpos.o<br>
  GEN     scripts/mod/devicetable-offsets.h<br>
  MKELF   scripts/mod/elfconfig.h<br>
  HOSTCC  scripts/mod/modpost.o<br>
  HOSTCC  scripts/dtc/checks.o<br>
  HOSTCC  scripts/dtc/util.o<br>
  HOSTCC  scripts/mod/file2alias.o<br>
  HOSTCC  scripts/mod/sumversion.o<br>
  SHIPPED scripts/dtc/dtc-lexer.lex.c<br>
  SHIPPED scripts/dtc/dtc-parser.tab.h<br>
  SHIPPED scripts/dtc/dtc-parser.tab.c<br>
  HOSTCC  scripts/dtc/dtc-lexer.lex.o<br>
  HOSTCC  scripts/dtc/dtc-parser.tab.o<br>
  HOSTLD  scripts/dtc/dtc<br>
  HOSTLD  scripts/mod/modpost<br>
  UPD     include/config/kernel.release<br>
  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel<br>
  CHK     include/generated/utsrelease.h<br>
  UPD     include/generated/utsrelease.h<br>
  CC      kernel/bounds.s<br>
  GEN     include/generated/bounds.h<br>
  CC      arch/arm64/kernel/asm-offsets.s<br>
  GEN     include/generated/asm-offsets.h<br>
  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh<br>
  CC      init/main.o<br>
  CHK     include/generated/compile.h<br>
  CC      init/do_mounts.o<br>
  CC      init/do_mounts_rd.o<br>
  CC      init/do_mounts_initrd.o<br>
  CC      init/do_mounts_dm.o<br>
  CC      init/noinitramfs.o<br>
  CC      init/initramfs.o<br>
  CC      init/calibrate.o<br>
  CC      init/init_task.o<br>
  HOSTCC  usr/gen_init_cpio<br>
  UPD     include/generated/compile.h<br>
  CC      init/version.o<br>
  GEN     usr/initramfs_data.cpio.gz<br>
  CC      arch/arm64/mm/dma-mapping.o<br>
  CC      arch/arm64/mm/extable.o<br>
  CC      arch/arm64/mm/fault.o<br>
  CC      arch/arm64/mm/init.o<br>
  AS      arch/arm64/mm/cache.o<br>
  CC      arch/arm64/mm/copypage.o<br>
  CC      arch/arm64/mm/flush.o<br>
  CC      arch/arm64/mm/ioremap.o<br>
  AS      usr/initramfs_data.o<br>
  LD      arch/arm64/net/built-in.o<br>
  CC      arch/arm64/mm/mmap.o<br>
  CC      arch/arm64/mm/pgd.o<br>
  CC      arch/arm64/mm/mmu.o<br>
  LD      usr/built-in.o<br>
  LDS     arch/arm64/kernel/vdso/vdso.lds<br>
  CC      arch/arm64/mm/context.o<br>
  AS      arch/arm64/kernel/head.o<br>
  LDS     arch/arm64/kernel/vmlinux.lds<br>
  AS      arch/arm64/mm/proc.o<br>
  VDSOA   arch/arm64/kernel/vdso/gettimeofday.o<br>
  CC      arch/arm64/mm/pageattr.o<br>
  VDSOA   arch/arm64/kernel/vdso/note.o<br>
  VDSOA   arch/arm64/kernel/vdso/sigreturn.o<br>
  VDSOL   arch/arm64/kernel/vdso/vdso.so.dbg<br>
  VDSOSYM arch/arm64/kernel/vdso/vdso-offsets.h<br>
  OBJCOPY arch/arm64/kernel/vdso/vdso.so<br>
  AS      arch/arm64/kernel/vdso/vdso.o<br>
  CC      arch/arm64/crypto/sha1-ce-glue.o<br>
  AS      arch/arm64/crypto/sha1-ce-core.o<br>
  CC      arch/arm64/crypto/sha2-ce-glue.o<br>
  AS      arch/arm64/crypto/sha2-ce-core.o<br>
  CC      arch/arm64/crypto/ghash-ce-glue.o<br>
  AS      arch/arm64/crypto/ghash-ce-core.o<br>
  CC      arch/arm64/crypto/aes-ce-cipher.o<br>
/home/stalker/hadk/kernel/xiaomi/msm8953/kernel/Makefile:133: *** No X.509 certificates found ***<br>
  CC      arch/arm64/crypto/aes-ce-ccm-glue.o<br>
  LD      arch/arm64/kernel/vdso/built-in.o<br>
  AS      arch/arm64/crypto/aes-ce-ccm-core.o<br>
  CC      arch/arm64/kernel/cputable.o<br>
  CC      arch/arm64/kernel/debug-monitors.o<br>
  AS      arch/arm64/kernel/entry.o<br>
  CC      arch/arm64/kernel/irq.o<br>
  CC      arch/arm64/crypto/aes-glue-ce.o<br>
  AS      arch/arm64/crypto/aes-ce.o<br>
  CC      arch/arm64/crypto/aes-glue-neon.o<br>
  AS      arch/arm64/crypto/aes-neon.o<br>
  CC      arch/arm64/kernel/fpsimd.o<br>
  AS      arch/arm64/kernel/entry-fpsimd.o<br>
  LD      init/mounts.o<br>
  CC      arch/arm64/kernel/process.o<br>
  LD      init/built-in.o<br>
  CC      arch/arm64/kernel/ptrace.o<br>
  CC      arch/arm64/kernel/setup.o<br>
  CC      arch/arm64/kernel/signal.o<br>
  CC      arch/arm64/kernel/sys.o<br>
  CC      arch/arm64/kernel/stacktrace.o<br>
  LD      arch/arm64/crypto/aes-ce-blk.o<br>
  CC      arch/arm64/kernel/time.o<br>
  CC      arch/arm64/kernel/traps.o<br>
  LD      ipc/built-in.o<br>
  CC      arch/arm64/kernel/io.o<br>
  CC      mm/filemap.o<br>
  LD      arch/arm64/crypto/sha1-ce.o<br>
  CC      arch/arm64/kernel/vdso.o<br>
  CC      kernel/fork.o<br>
  LD      arch/arm64/crypto/sha2-ce.o<br>
  AS      arch/arm64/kernel/hyp-stub.o<br>
  CC      kernel/exec_domain.o<br>
  CC      mm/mempool.o<br>
  CC      arch/arm64/kernel/psci.o<br>
  CC      kernel/panic.o<br>
  AS      arch/arm64/kernel/psci-call.o<br>
  CC      kernel/cpu.o<br>
  CC      arch/arm64/kernel/cpu_ops.o<br>
  CC      arch/arm64/kernel/insn.o<br>
  CC      arch/arm64/kernel/return_address.o<br>
  CC      mm/oom_kill.o<br>
  LD      arch/arm64/mm/built-in.o<br>
  CC      kernel/exit.o<br>
  CC      arch/arm64/kernel/cpuinfo.o<br>
  CC      mm/maccess.o<br>
  LD      arch/arm64/crypto/ghash-ce.o<br>
  CC      mm/page_alloc.o<br>
  LD      arch/arm64/crypto/aes-ce-ccm.o<br>
  LD      arch/arm64/crypto/aes-neon-blk.o<br>
  CC      arch/arm64/kernel/cpu_errata.o<br>
  LD      arch/arm64/crypto/built-in.o<br>
  CC      security/integrity/iint.o<br>
  LD      security/pfe/built-in.o<br>
  CC      kernel/softirq.o<br>
  CC      arch/arm64/kernel/cpufeature.o<br>
  CC      arch/arm64/kernel/alternative.o<br>
  CC      security/keys/gc.o<br>
  CC      kernel/resource.o<br>
  CC      fs/open.o<br>
  CC      kernel/sysctl.o<br>
  CC      kernel/sysctl_binary.o<br>
  GEN     security/selinux/flask.h security/selinux/av_permissions.h<br>
  CC      security/selinux/avc.o<br>
  AS      arch/arm64/kernel/sys32.o<br>
  CC      security/integrity/integrity_audit.o<br>
  CC      security/selinux/hooks.o<br>
  CC      kernel/capability.o<br>
  CC      security/keys/key.o<br>
  AS      arch/arm64/kernel/kuser32.o<br>
  CC      security/selinux/selinuxfs.o<br>
  CC      security/selinux/netlink.o<br>
  CC      security/selinux/nlmsgtab.o<br>
  CC      security/selinux/netif.o<br>
  CC      security/selinux/netnode.o<br>
  CC      security/keys/keyring.o<br>
  LD      security/integrity/integrity.o<br>
  CC      kernel/ptrace.o<br>
  LD      security/integrity/built-in.o<br>
  CC      security/selinux/netport.o<br>
  CC      security/commoncap.o<br>
  CC      crypto/api.o<br>
  CC      arch/arm64/kernel/signal32.o<br>
  CC      crypto/cipher.o<br>
  CC      crypto/compress.o<br>
  CC      security/selinux/exports.o<br>
  CC      security/keys/keyctl.o<br>
  CC      security/keys/permission.o<br>
  CC      arch/arm64/kernel/sys_compat.o<br>
  CC      security/selinux/ss/ebitmap.o<br>
  CC      security/min_addr.o<br>
  CC      arch/arm64/kernel/../../arm/kernel/opcodes.o<br>
  CC      security/selinux/ss/hashtab.o<br>
  CC      security/selinux/ss/symtab.o<br>
  CC      security/selinux/ss/sidtab.o<br>
  CC      security/selinux/ss/avtab.o<br>
  CC      security/security.o<br>
  CC      security/selinux/ss/policydb.o<br>
  CC      block/bio.o<br>
  CC      security/keys/process_keys.o<br>
  CC      fs/read_write.o<br>
  CC      arch/arm64/kernel/ftrace.o<br>
  CC      security/selinux/ss/services.o<br>
  CC      crypto/memneq.o<br>
  CC      security/selinux/ss/conditional.o<br>
  CC      security/selinux/ss/mls.o<br>
  AS      arch/arm64/kernel/entry-ftrace.o<br>
  CC      security/capability.o<br>
  CC      security/lsm_audit.o<br>
  CC      security/selinux/ss/status.o<br>
  CC      arch/arm64/kernel/smp.o<br>
  CC      crypto/crypto_wq.o<br>
  CC      crypto/algapi.o<br>
  CC      kernel/user.o<br>
  CC      kernel/signal.o<br>
  CC      kernel/sys.o<br>
  CC      kernel/kmod.o<br>
  CC      arch/arm64/kernel/smp_spin_table.o<br>
  CC      kernel/workqueue.o<br>
  CC      crypto/scatterwalk.o<br>
  CC      crypto/proc.o<br>
  CC      crypto/aead.o<br>
  CC      security/keys/request_key.o<br>
  CC      security/keys/request_key_auth.o<br>
  CC      crypto/ablkcipher.o<br>
  CC      arch/arm64/kernel/topology.o<br>
  CC      arch/arm64/kernel/perf_regs.o<br>
  CC      crypto/blkcipher.o<br>
  CC      crypto/chainiv.o<br>
  CC      security/keys/user_defined.o<br>
  CC      kernel/pid.o<br>
  CC      crypto/eseqiv.o<br>
  CC      arch/arm64/kernel/perf_event.o<br>
  CC      drivers/amba/bus.o<br>
  CC      security/keys/proc.o<br>
  CC      crypto/seqiv.o<br>
  CC      crypto/ahash.o<br>
  CC      mm/page-writeback.o<br>
  CC      arch/arm64/kernel/perf_debug.o<br>
  CC      block/elevator.o<br>
  CC      kernel/task_work.o<br>
  CC      security/keys/sysctl.o<br>
  CC      arch/arm64/kernel/perf_trace_counters.o<br>
  LD      drivers/amba/built-in.o<br>
  CC      mm/readahead.o<br>
  LD      drivers/auxdisplay/built-in.o<br>
  CC      crypto/shash.o<br>
  CC      security/keys/encrypted-keys/encrypted.o<br>
  CC      fs/file_table.o<br>
  CC      security/keys/encrypted-keys/ecryptfs_format.o<br>
  CC      mm/swap.o<br>
  CC      crypto/pcompress.o<br>
  CC      crypto/algboss.o<br>
  CC      mm/truncate.o<br>
  LD      security/keys/encrypted-keys/encrypted-keys.o<br>
  LD      security/keys/encrypted-keys/built-in.o<br>
  CC      crypto/testmgr.o<br>
  LD      security/keys/built-in.o<br>
  CC      crypto/cmac.o<br>
  CC      fs/super.o<br>
  CC      drivers/base/component.o<br>
  CC      arch/arm64/kernel/perf_trace_user.o<br>
  CC      crypto/hmac.o<br>
  CC      mm/vmscan.o<br>
  CC      kernel/extable.o<br>
  CC      crypto/xcbc.o<br>
  CC      kernel/params.o<br>
  CC      kernel/kthread.o<br>
  CC      kernel/sys_ni.o<br>
  LD      security/selinux/selinux.o<br>
  LD      security/selinux/built-in.o<br>
  CC      crypto/crypto_null.o<br>
  CC      crypto/md4.o<br>
  LD      security/built-in.o<br>
  CC      crypto/md5.o<br>
  CC      kernel/nsproxy.o<br>
  CC      crypto/sha1_generic.o<br>
  CC      crypto/sha256_generic.o<br>
  AS      arch/arm64/kernel/sleep.o<br>
  CC      fs/char_dev.o<br>
  CC      drivers/base/core.o<br>
  CC      fs/stat.o<br>
  CC      arch/arm64/kernel/suspend.o<br>
  CC      fs/exec.o<br>
  CC      fs/pipe.o<br>
  CC      kernel/notifier.o<br>
  CC      crypto/sha512_generic.o<br>
  CC      drivers/base/bus.o<br>
  CC      fs/namei.o<br>
  CC      block/blk-core.o<br>
  CC      drivers/base/dd.o<br>
  CC      kernel/ksysfs.o<br>
  CC      fs/fcntl.o<br>
  CC      crypto/gf128mul.o<br>
  CC      fs/ioctl.o<br>
  CC      drivers/base/syscore.o<br>
  CC      kernel/cred.o<br>
  CC      drivers/base/driver.o<br>
  CC      drivers/base/class.o<br>
  CC      kernel/reboot.o<br>
  CC      arch/arm64/kernel/cpuidle.o<br>
  CC      drivers/base/platform.o<br>
  CC      drivers/base/cpu.o<br>
  CC      arch/arm64/kernel/efi.o<br>
  CC      arch/arm64/kernel/efi-stub.o<br>
  CC      drivers/block/brd.o<br>
  CC      kernel/async.o<br>
  CC      drivers/block/loop.o<br>
  CC      drivers/base/firmware.o<br>
  CC      drivers/base/init.o<br>
  CC      drivers/base/map.o<br>
  CC      drivers/block/zram/zcomp_lzo.o<br>
  CC      crypto/ecb.o<br>
  CC      drivers/block/zram/zcomp.o<br>
  CC      block/blk-tag.o<br>
  CC      block/blk-sysfs.o<br>
  CC      drivers/base/devres.o<br>
  CC      drivers/block/zram/zram_drv.o<br>
  CC      kernel/range.o<br>
  AS      arch/arm64/kernel/efi-entry.o<br>
  CC      fs/readdir.o<br>
  CC      fs/select.o<br>
  CC      drivers/base/attribute_container.o<br>
  CC      block/blk-flush.o<br>
  CC      block/blk-settings.o<br>
  CC      drivers/base/transport_class.o<br>
  CC      arch/arm64/kernel/pci.o<br>
  CC      drivers/base/topology.o<br>
  CC      kernel/groups.o<br>
  CC      kernel/smpboot.o<br>
  CC      block/blk-ioc.o<br>
  CC      fs/dcache.o<br>
  LD      drivers/block/zram/zram.o<br>
  CC      crypto/cbc.o<br>
  CC      drivers/base/container.o<br>
  LD      drivers/block/zram/built-in.o<br>
  LD      drivers/block/built-in.o<br>
  CC      arch/arm64/kernel/armv8_deprecated.o<br>
  CC      block/blk-map.o<br>
  CC      fs/inode.o<br>
  CC      drivers/bluetooth/bluetooth-power.o<br>
  CC      block/blk-exec.o<br>
  CC      crypto/cts.o<br>
  CC      fs/attr.o<br>
  CC      drivers/base/property.o<br>
  CC      block/blk-merge.o<br>
  CC      block/blk-softirq.o<br>
  CC      drivers/base/dma-contiguous.o<br>
  CC      drivers/base/dma-mapping.o<br>
  CC      drivers/base/regmap/regmap.o<br>
  CC      kernel/bpf/core.o<br>
  CC      crypto/xts.o<br>
  CC      block/blk-timeout.o<br>
  CC      drivers/base/power/sysfs.o<br>
  CC      block/blk-iopoll.o<br>
  CC      drivers/base/regmap/regcache.o<br>
  CC      drivers/base/regmap/regcache-rbtree.o<br>
  LD      drivers/bluetooth/built-in.o<br>
  CC      mm/shmem.o<br>
  CC      drivers/base/regmap/regcache-lzo.o<br>
  CC      drivers/base/regmap/regcache-flat.o<br>
  LD      kernel/bpf/built-in.o<br>
  CC      drivers/base/dma-coherent.o<br>
  CC      drivers/base/regmap/regmap-debugfs.o<br>
  CC      block/blk-lib.o<br>
  CC      block/blk-mq.o<br>
  CC      kernel/events/core.o<br>
  CC      block/blk-mq-tag.o<br>
  CC      kernel/events/ring_buffer.o<br>
  CC      drivers/base/dma-removed.o<br>
  CC      fs/bad_inode.o<br>
  CC      drivers/base/regmap/regmap-i2c.o<br>
  LD      arch/arm64/kernel/built-in.o<br>
  CC      block/blk-mq-sysfs.o<br>
  CC      block/blk-mq-cpu.o<br>
  CC      crypto/ctr.o<br>
  CC      kernel/events/callchain.o<br>
  CC      block/blk-mq-cpumap.o<br>
  CC      block/ioctl.o<br>
  CC      crypto/cryptd.o<br>
  LD      kernel/events/built-in.o<br>
  CC      drivers/base/power/generic_ops.o<br>
  CC      drivers/base/power/common.o<br>
  CC      drivers/base/regmap/regmap-spi.o<br>
  CC      drivers/base/firmware_class.o<br>
  CC      fs/file.o<br>
  CC      kernel/irq/irqdesc.o<br>
  CC      drivers/base/power/qos.o<br>
  CC      crypto/des_generic.o<br>
  CC      block/genhd.o<br>
  CC      kernel/irq/handle.o<br>
  CC      block/scsi_ioctl.o<br>
  CC      drivers/base/power/runtime.o<br>
  CC      drivers/base/soc.o<br>
  CC      fs/filesystems.o<br>
  CC      drivers/base/regmap/regmap-swr.o<br>
  CC      block/partition-generic.o<br>
  CC      drivers/base/pinctrl.o<br>
  CC      drivers/base/devcoredump.o<br>
  CC      kernel/irq/manage.o<br>
  CC      kernel/irq/spurious.o<br>
  CC      block/ioprio.o<br>
  CC      fs/namespace.o<br>
  CC      drivers/base/power/main.o<br>
  CC      sound/sound_core.o<br>
  CC      block/bounce.o<br>
  CC      fs/seq_file.o<br>
  CC      block/partitions/check.o<br>
  CC      block/partitions/msdos.o<br>
  CC      drivers/base/power/wakeup.o<br>
  CC      block/bsg.o<br>
  LD      sound/arm/built-in.o<br>
  CC      fs/xattr.o<br>
  CC      kernel/irq/resend.o<br>
  LD      sound/atmel/built-in.o<br>
  CC      fs/libfs.o<br>
  CC      block/partitions/efi.o<br>
  LD      drivers/base/regmap/built-in.o<br>
  CC      crypto/twofish_generic.o<br>
  CC      block/noop-iosched.o<br>
  CC      block/deadline-iosched.o<br>
  CC      fs/fs-writeback.o<br>
  LD      sound/drivers/mpu401/built-in.o<br>
  CC      kernel/irq/chip.o<br>
  LD      sound/drivers/opl3/built-in.o<br>
  CC      block/cfq-iosched.o<br>
  CC      block/compat_ioctl.o<br>
  LD      drivers/bus/built-in.o<br>
  LD      sound/drivers/opl4/built-in.o<br>
  CC      crypto/twofish_common.o<br>
  LD      drivers/cdrom/built-in.o<br>
  CC      drivers/base/power/opp/core.o<br>
  LD      sound/drivers/pcsp/built-in.o<br>
  LD      sound/drivers/vx/built-in.o<br>
  CC      drivers/base/power/clock_ops.o<br>
  CC      sound/core/sound.o<br>
  LD      sound/drivers/built-in.o<br>
  CC      sound/core/init.o<br>
  CC      drivers/base/power/opp/cpu.o<br>
  CC      drivers/char/mem.o<br>
  CC      drivers/char/random.o<br>
  CC      fs/pnode.o<br>
  CC      mm/util.o<br>
  CC      crypto/aes_generic.o<br>
  CC      fs/splice.o<br>
  CC      drivers/char/misc.o<br>
  CC      crypto/arc4.o<br>
  CC      fs/sync.o<br>
  LD      block/partitions/built-in.o<br>
  CC      fs/utimes.o<br>
  CC      drivers/clk/clk-devres.o<br>
  CC      mm/mmzone.o<br>
  CC      mm/vmstat.o<br>
  LD      drivers/base/power/opp/built-in.o<br>
  LD      drivers/base/power/built-in.o<br>
  CC      fs/stack.o<br>
  CC      fs/fs_struct.o<br>
  LD      drivers/base/built-in.o<br>
  CC      drivers/clocksource/clksrc-of.o<br>
  CC      drivers/char/msm_smd_pkt.o<br>
  CC      crypto/deflate.o<br>
  CC      drivers/clocksource/arm_arch_timer.o<br>
  CC      kernel/locking/mutex.o<br>
  CC      mm/backing-dev.o<br>
  CC      kernel/irq/dummychip.o<br>
  CC      mm/mm_init.o<br>
  CC      sound/core/memory.o<br>
  CC      kernel/locking/semaphore.o<br>
  CC      drivers/clk/clkdev.o<br>
  CC      drivers/clocksource/dummy_timer.o<br>
  CC      crypto/crc32c_generic.o<br>
  CC      drivers/clk/clk.o<br>
  CC      sound/core/info.o<br>
  LD      block/built-in.o<br>
  CC      fs/statfs.o<br>
  CC      kernel/locking/rwsem.o<br>
  CC      crypto/crc32.o<br>
  LD      drivers/clocksource/built-in.o<br>
  CC      fs/fs_pin.o<br>
  CC      drivers/coresight/coresight.o<br>
  CC      drivers/coresight/coresight-event.o<br>
  CC      drivers/cpufreq/cpufreq.o<br>
  CC      kernel/irq/devres.o<br>
  CC      kernel/locking/mcs_spinlock.o<br>
  LD      firmware/built-in.o<br>
  CC      crypto/authenc.o<br>
  CC      fs/buffer.o<br>
  CC      drivers/clk/msm/clock.o<br>
  CC      drivers/clk/msm/clock-dummy.o<br>
  CC      kernel/locking/spinlock.o<br>
  CC      drivers/coresight/coresight-fuse.o<br>
  CC      drivers/coresight/coresight-cti.o<br>
  CC      kernel/locking/lglock.o<br>
  CC      drivers/cpufreq/freq_table.o<br>
  CC      kernel/locking/rtmutex.o<br>
  CC      drivers/clk/msm/clock-generic.o<br>
  CC      drivers/coresight/coresight-csr.o<br>
  CC      drivers/coresight/coresight-tmc.o<br>
  CC      fs/block_dev.o<br>
  CC      mm/mmu_context.o<br>
  CC      drivers/cpufreq/cpufreq_stats.o<br>
  CC      mm/percpu.o<br>
  CC      sound/core/control.o<br>
  CC      drivers/clk/msm/clock-local2.o<br>
  CC      fs/direct-io.o<br>
  CC      kernel/irq/autoprobe.o<br>
  CC      net/socket.o<br>
  CC      kernel/locking/rwsem-xadd.o<br>
  CC      kernel/irq/irqdomain.o<br>
  CC      kernel/irq/proc.o<br>
  CC      drivers/clk/msm/clock-pll.o<br>
  CC      net/802/p8022.o<br>
  LD      sound/firewire/built-in.o<br>
  CC      net/802/psnap.o<br>
  CC      fs/mpage.o<br>
  CC      mm/slab_common.o<br>
  CC      drivers/clk/msm/clock-alpha-pll.o<br>
  CC      sound/core/misc.o<br>
  CC      fs/proc_namespace.o<br>
  LD      kernel/locking/built-in.o<br>
  CC      net/802/stp.o<br>
  LD      arch/arm64/lib/built-in.o<br>
  AS      arch/arm64/lib/bitops.o<br>
  CC      kernel/irq/pm.o<br>
  CC      drivers/clk/msm/clock-rpm.o<br>
  CC      drivers/clk/msm/clock-voter.o<br>
  LD      drivers/char/agp/built-in.o<br>
  AS      arch/arm64/lib/clear_page.o<br>
  CC      drivers/coresight/coresight-tpiu.o<br>
  AS      arch/arm64/lib/clear_user.o<br>
  CC      kernel/irq/msi.o<br>
  CC      net/bluetooth/af_bluetooth.o<br>
  AS      arch/arm64/lib/copy_from_user.o<br>
  CC      mm/compaction.o<br>
  CC      net/bluetooth/hci_core.o<br>
  CC      mm/vmacache.o<br>
  CC      drivers/char/diag/diagchar_core.o<br>
  CC      crypto/authencesn.o<br>
  CC      drivers/clk/msm/clock-pm.o<br>
  AS      arch/arm64/lib/copy_in_user.o<br>
  CC      crypto/lzo.o<br>
  CC      crypto/rng.o<br>
  CC      net/bluetooth/hci_conn.o<br>
  CC      drivers/coresight/coresight-nidnt.o<br>
  CC      crypto/krng.o<br>
  CC      sound/core/device.o<br>
  AS      arch/arm64/lib/copy_page.o<br>
  LD      kernel/irq/built-in.o<br>
  AS      arch/arm64/lib/copy_to_user.o<br>
  CC      drivers/cpufreq/cpufreq_performance.o<br>
  CC      drivers/clk/msm/msm-clock-controller.o<br>
  CC      arch/arm64/lib/delay.o<br>
  CC      drivers/char/diag/diagchar_hdlc.o<br>
  CC      kernel/printk/printk.o<br>
  CC      drivers/clk/msm/clock-debug.o<br>
  CC      kernel/power/qos.o<br>
  CC      drivers/coresight/coresight-funnel.o<br>
  CC      drivers/cpufreq/cpufreq_powersave.o<br>
  CC      sound/core/jack.o<br>
  CC      kernel/power/main.o<br>
  LD      net/802/built-in.o<br>
  CC      net/bridge/br.o<br>
  CC      net/bridge/br_device.o<br>
  CC      net/bridge/br_fdb.o<br>
  CC      mm/interval_tree.o<br>
  CC      sound/core/hwdep.o<br>
  CC      drivers/cpufreq/cpufreq_userspace.o<br>
  AS      arch/arm64/lib/memchr.o<br>
  CC      kernel/power/process.o<br>
  CC      fs/cifs/cifsfs.o<br>
  CC      drivers/char/diag/diagfwd.o<br>
  CC      drivers/clk/msm/clock-cpu-8953.o<br>
  AS      arch/arm64/lib/memcmp.o<br>
  CC      mm/list_lru.o<br>
  CC      drivers/cpufreq/cpufreq_ondemand.o<br>
  CC      drivers/coresight/coresight-replicator.o<br>
  CC      kernel/power/suspend.o<br>
  CC      drivers/clk/msm/clock-gcc-8953.o<br>
  CC      mm/workingset.o<br>
  CC      drivers/coresight/coresight-stm.o<br>
  AS      arch/arm64/lib/memcpy.o<br>
  CC      sound/core/timer.o<br>
  CC      drivers/clk/msm/clock-rcgwr.o<br>
  AS      arch/arm64/lib/memmove.o<br>
  CC      drivers/char/diag/diagfwd_peripheral.o<br>
  CC      drivers/coresight/coresight-hwevent.o<br>
  CC      net/bridge/br_forward.o<br>
  CC      net/bridge/br_if.o<br>
  CC      drivers/cpufreq/cpufreq_conservative.o<br>
  AS      arch/arm64/lib/memset.o<br>
  CC      sound/core/pcm.o<br>
  AS      arch/arm64/lib/strchr.o<br>
  CC      drivers/cpufreq/cpufreq_interactive.o<br>
  LD      drivers/coresight/built-in.o<br>
  CC      drivers/char/diag/diagfwd_smd.o<br>
  CC      drivers/clk/msm/gdsc.o<br>
  CC      drivers/char/diag/diagfwd_socket.o<br>
  AS      arch/arm64/lib/strcmp.o<br>
  CC      mm/iov_iter.o<br>
  CC      drivers/cpufreq/cpufreq_governor.o<br>
  CC      drivers/cpufreq/qcom-cpufreq.o<br>
  AS      arch/arm64/lib/strlen.o<br>
  CC      sound/core/pcm_native.o<br>
  CC      crypto/ansi_cprng.o<br>
  CC      mm/debug.o<br>
  CC      sound/core/pcm_lib.o<br>
  AS      arch/arm64/lib/strncmp.o<br>
  AS      arch/arm64/lib/strnlen.o<br>
  CC      drivers/char/diag/diag_mux.o<br>
  LD      drivers/cpufreq/built-in.o<br>
  CC      drivers/char/diag/diag_memorydevice.o<br>
  CC      sound/core/pcm_timer.o<br>
  AS      arch/arm64/lib/strrchr.o<br>
  CC      mm/fremap.o<br>
  CC      net/bluetooth/hci_event.o<br>
  CC      crypto/asymmetric_keys/asymmetric_type.o<br>
  CC      sound/core/pcm_misc.o<br>
  AR      arch/arm64/lib/lib.a<br>
  CC      kernel/power/autosleep.o<br>
  CC      sound/core/pcm_memory.o<br>
  CC      drivers/char/diag/diag_usb.o<br>
  CC      net/bluetooth/mgmt.o<br>
  CC      drivers/char/diag/diagmem.o<br>
  CC      net/bridge/br_input.o<br>
  CC      kernel/power/wakelock.o<br>
  CC      net/core/sock.o<br>
  CC      drivers/char/diag/diagfwd_cntl.o<br>
  CC      crypto/asymmetric_keys/signature.o<br>
  CC      mm/gup.o<br>
  CC      sound/core/memalloc.o<br>
  CC      fs/cifs/cifssmb.o<br>
  CC      kernel/power/suspend_time.o<br>
  CC      kernel/power/poweroff.o<br>
  CC      mm/highmem.o<br>
  CC      drivers/char/diag/diag_dci.o<br>
  LD      kernel/printk/built-in.o<br>
  CC      kernel/power/wakeup_reason.o<br>
  CC      net/bridge/br_ioctl.o<br>
  CC      kernel/rcu/update.o<br>
  CC      kernel/rcu/srcu.o<br>
  CC      drivers/clk/msm/mdss/mdss-pll-util.o<br>
  CC      net/core/request_sock.o<br>
  CC      mm/memory.o<br>
  CC      mm/mincore.o<br>
  CC      mm/mlock.o<br>
  CC      kernel/rcu/tree.o<br>
  LD      kernel/power/built-in.o<br>
  CC      sound/core/rawmidi.o<br>
  CC      sound/core/compress_offload.o<br>
  CC      mm/mmap.o<br>
  CC      net/core/skbuff.o<br>
  CC      crypto/asymmetric_keys/public_key.o<br>
  CC      drivers/clk/msm/mdss/mdss-pll.o<br>
  CC      mm/mprotect.o<br>
  CC      kernel/sched/core.o<br>
  CC      mm/mremap.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-util.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-28lpm.o<br>
  LD      sound/core/snd.o<br>
  LD      sound/core/snd-hwdep.o<br>
  LD      sound/core/snd-timer.o<br>
  CC      drivers/char/diag/diag_masks.o<br>
  LD      sound/core/snd-pcm.o<br>
  LD      sound/core/snd-rawmidi.o<br>
  LD      sound/core/snd-compress.o<br>
  LD      sound/core/built-in.o<br>
  CC      net/core/iovec.o<br>
  CC      drivers/char/diag/diag_debugfs.o<br>
  CC      crypto/asymmetric_keys/rsa.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996.o<br>
  LD      sound/i2c/other/built-in.o<br>
  LD      sound/i2c/built-in.o<br>
  CC      mm/msync.o<br>
  ASN.1   crypto/asymmetric_keys/x509-asn1.c<br>
Extracted 203 tokens<br>
Extracted 14 types<br>
Extracted 12 actions<br>
Pass 1<br>
Pass 2<br>
  ASN.1   crypto/asymmetric_keys/x509_rsakey-asn1.c<br>
Extracted 16 tokens<br>
Extracted 1 types<br>
Extracted 1 actions<br>
Pass 1<br>
Pass 2<br>
  CC      crypto/asymmetric_keys/x509_public_key.o<br>
  LD      drivers/char/diag/diagchar.o<br>
  CC      drivers/cpuidle/cpuidle.o<br>
  LD      sound/mips/built-in.o<br>
  CC      net/core/datagram.o<br>
  LD      drivers/char/diag/built-in.o<br>
  LD      sound/parisc/built-in.o<br>
  CC      mm/rmap.o<br>
  CC      drivers/char/hw_random/core.o<br>
  CC      net/bridge/br_stp.o<br>
  LD      sound/isa/ad1816a/built-in.o<br>
  LD      sound/pcmcia/pdaudiocf/built-in.o<br>
  CC      drivers/char/hw_random/msm_rng.o<br>
  LD      sound/isa/ad1848/built-in.o<br>
  LD      sound/pcmcia/vx/built-in.o<br>
  CC      drivers/cpuidle/driver.o<br>
  LD      sound/isa/cs423x/built-in.o<br>
  LD      sound/pcmcia/built-in.o<br>
  LD      sound/isa/es1688/built-in.o<br>
  CC      net/bluetooth/hci_sock.o<br>
  LD      sound/isa/galaxy/built-in.o<br>
  CC      net/core/stream.o<br>
  LD      sound/isa/gus/built-in.o<br>
  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996-util.o<br>
  LD      sound/isa/msnd/built-in.o<br>
  CC      drivers/cpuidle/governor.o<br>
  LD      crypto/asymmetric_keys/asymmetric_keys.o<br>
  CC      drivers/cpuidle/sysfs.o<br>
  LD      sound/isa/opti9xx/built-in.o<br>
  CC      crypto/asymmetric_keys/x509-asn1.o<br>
  LD      sound/isa/sb/built-in.o<br>
  LD      sound/pci/ac97/built-in.o<br>
  LD      sound/isa/wavefront/built-in.o<br>
  CC      kernel/sched/fair.o<br>
  LD      sound/pci/ali5451/built-in.o<br>
  CC      crypto/asymmetric_keys/x509_rsakey-asn1.o<br>
  LD      sound/isa/wss/built-in.o<br>
  LD      sound/isa/built-in.o<br>
  LD      sound/pci/asihpi/built-in.o<br>
  CC      crypto/hash_info.o<br>
  CC      net/ethernet/eth.o<br>
  CC      crypto/ablk_helper.o<br>
  LD      sound/pci/au88x0/built-in.o<br>
  CC      crypto/asymmetric_keys/x509_cert_parser.o<br>
  LD      sound/pci/aw2/built-in.o<br>
  CC      fs/cifs/cifs_debug.o<br>
  LD      sound/pci/ca0106/built-in.o<br>
  LD      drivers/char/hw_random/rng-core.o<br>
  LD      sound/pci/cs46xx/built-in.o<br>
  LD      drivers/char/hw_random/built-in.o<br>
  LD      crypto/asymmetric_keys/x509_key_parser.o<br>
  CC      fs/cifs/connect.o<br>
  CC      drivers/char/adsprpc.o<br>
  LD      crypto/asymmetric_keys/built-in.o<br>
  LD      sound/pci/cs5535audio/built-in.o<br>
  CC      drivers/cpuidle/governors/ladder.o<br>
  LD      crypto/crypto.o<br>
  LD      sound/pci/ctxfi/built-in.o<br>
  LD      crypto/crypto_algapi.o<br>
  CC      mm/vmalloc.o<br>
  LD      crypto/crypto_blkcipher.o<br>
  LD      sound/pci/echoaudio/built-in.o<br>
  LD      crypto/crypto_hash.o<br>
  LD      crypto/cryptomgr.o<br>
  LD      net/ethernet/built-in.o<br>
  CC      net/bridge/br_stp_bpdu.o<br>
  LD      crypto/built-in.o<br>
  LD      sound/pci/emu10k1/built-in.o<br>
  LD      sound/pci/hda/built-in.o<br>
  LD      kernel/rcu/built-in.o<br>
  CC      fs/cifs/dir.o<br>
  CC      fs/cifs/file.o<br>
  LD      sound/pci/korg1212/built-in.o<br>
  CC      drivers/cpuidle/governors/menu.o<br>
  LD      sound/pci/ice1712/built-in.o<br>
  CC      net/core/scm.o<br>
  CC      drivers/clk/msm/mdss/mdss-hdmi-pll-8996.o<br>
  LD      sound/pci/lola/built-in.o<br>
  CC      fs/cifs/inode.o<br>
  CC      fs/cifs/link.o<br>
  CC      net/ipc_router/ipc_router_core.o<br>
  LD      sound/pci/lx6464es/built-in.o<br>
  LD      sound/pci/mixart/built-in.o<br>
  CC      drivers/char/adsprpc_compat.o<br>
  LD      sound/pci/nm256/built-in.o<br>
  LD      sound/pci/oxygen/built-in.o<br>
  CC      fs/cifs/misc.o<br>
  LD      drivers/cpuidle/governors/built-in.o<br>
  CC      drivers/cpuidle/lpm-levels.o<br>
  LD      sound/pci/pcxhr/built-in.o<br>
  LD      sound/pci/riptide/built-in.o<br>
  CC      drivers/crypto/msm/qcedev.o<br>
  LD      sound/pci/rme9652/built-in.o<br>
  CC      net/ipc_router/ipc_router_socket.o<br>
  LD      sound/pci/trident/built-in.o<br>
  LD      sound/pci/vx222/built-in.o<br>
  CC      net/bluetooth/hci_sysfs.o<br>
  LD      sound/pci/ymfpci/built-in.o<br>
  LD      sound/pci/built-in.o<br>
  LD      sound/ppc/built-in.o<br>
  CC      drivers/cpuidle/lpm-levels-of.o<br>
  LD      drivers/firmware/efi/libstub/built-in.o<br>
  CC      drivers/firmware/efi/libstub/arm-stub.o<br>
  CC      net/ipv4/route.o<br>
  LD      sound/sh/built-in.o<br>
  CC      drivers/crypto/msm/qce50.o<br>
  CC      net/bridge/br_stp_if.o<br>
  LD      drivers/char/built-in.o<br>
  CC      drivers/crypto/msm/compat_qcedev.o<br>
  CC      lib/lockref.o<br>
  CC      drivers/firmware/efi/libstub/efi-stub-helper.o<br>
  CC      drivers/crypto/msm/qcrypto.o<br>
  CC      net/core/gen_stats.o<br>
  CC      net/bluetooth/l2cap_core.o<br>
  LD      drivers/clk/msm/mdss/built-in.o<br>
  CC      drivers/crypto/msm/ota_crypto.o<br>
  CC      fs/cifs/netmisc.o<br>
  CC      net/ipv4/inetpeer.o<br>
  LD      drivers/clk/msm/built-in.o<br>
  LD      drivers/clk/built-in.o<br>
  CC      net/ipv4/protocol.o<br>
  CC      drivers/crypto/msm/ice.o<br>
  CC      fs/configfs/inode.o<br>
  CC      lib/bcd.o<br>
  CC      net/core/gen_estimator.o<br>
  LD      drivers/crypto/msm/built-in.o<br>
  CC      sound/soc/soc-core.o<br>
  CC      net/bluetooth/l2cap_sock.o<br>
  LD      drivers/crypto/built-in.o<br>
  CC      mm/pagewalk.o<br>
  CC      lib/div64.o<br>
  CC      fs/configfs/file.o<br>
  CC      fs/cifs/smbencrypt.o<br>
  CC      fs/configfs/dir.o<br>
  CC      net/ipc_router/ipc_router_security.o<br>
  CC      fs/configfs/symlink.o<br>
  CC      lib/sort.o<br>
  CC      mm/pgtable-generic.o<br>
  CC      mm/process_vm_access.o<br>
  CC      drivers/cpuidle/lpm-workarounds.o<br>
  CC      drivers/firmware/efi/libstub/fdt.o<br>
  LD      drivers/cpuidle/built-in.o<br>
  CC      kernel/sched/rt.o<br>
  CC      mm/showmem.o<br>
  LD      net/ipc_router/built-in.o<br>
  CC      sound/soc/soc-dapm.o<br>
  CC      net/bluetooth/smp.o<br>
  AR      drivers/firmware/efi/libstub/lib.a<br>
  CC      net/bridge/br_stp_timer.o<br>
  CC      lib/parser.o<br>
  CC      net/bluetooth/sco.o<br>
  CC      drivers/devfreq/devfreq.o<br>
  CC      fs/configfs/mount.o<br>
  CC      net/ipv4/ip_input.o<br>
  CC      drivers/devfreq/devfreq_trace.o<br>
  CC      net/ipv4/ip_fragment.o<br>
  CC      sound/soc/soc-jack.o<br>
  CC      net/core/net_namespace.o<br>
  CC      drivers/devfreq/governor_simpleondemand.o<br>
  CC      fs/cifs/transport.o<br>
  CC      mm/vmpressure.o<br>
  CC      net/ipv6/af_inet6.o<br>
  CC      net/bluetooth/lib.o<br>
  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3-mido.dtb<br>
  CC      fs/configfs/item.o<br>
  CC      mm/init-mm.o<br>
  CC      lib/halfmd4.o<br>
  CC      net/ipv4/ip_forward.o<br>
  CC      drivers/dma/dmaengine.o<br>
  CC      net/core/secure_seq.o<br>
  CC      mm/nobootmem.o<br>
  CC      fs/cifs/asn1.o<br>
  CC      lib/debug_locks.o<br>
  CC      net/bluetooth/a2mp.o<br>
  LD      fs/configfs/configfs.o<br>
  CC      net/ipv6/anycast.o<br>
  LD      fs/configfs/built-in.o<br>
  CC      fs/cifs/cifs_unicode.o<br>
  CC      lib/random32.o<br>
  CC      net/bluetooth/amp.o<br>
  CC      drivers/devfreq/governor_performance.o<br>
  CC      mm/fadvise.o<br>
  CC      fs/crypto/crypto.o<br>
  CC      drivers/dma/of-dma.o<br>
  CC      mm/madvise.o<br>
  CC      mm/memblock.o<br>
  CC      fs/cifs/nterr.o<br>
  CC      sound/soc/soc-cache.o<br>
  CC      net/bridge/br_netlink.o<br>
  CC      net/ipv4/ip_options.o<br>
  CC      net/bridge/br_sysfs_if.o<br>
  CC      drivers/devfreq/governor_powersave.o<br>
  CC      fs/crypto/fname.o<br>
  CC      net/core/flow_dissector.o<br>
  CC      drivers/dma/qcom-sps-dma.o<br>
  CC      net/ipv6/ip6_output.o<br>
  CC      mm/page_io.o<br>
  LD      drivers/dma/xilinx/built-in.o<br>
  CC      sound/soc/soc-utils.o<br>
  CC      net/bluetooth/bnep/core.o<br>
  CC      sound/soc/soc-pcm.o<br>
  CC      net/ipv4/ip_output.o<br>
  CC      fs/crypto/policy.o<br>
  CC      net/bridge/br_sysfs_br.o<br>
  CC      kernel/sched/proc.o<br>
  CC      lib/bust_spinlocks.o<br>
  CC      net/ipv4/ip_sockglue.o<br>
  LD      drivers/dma/built-in.o<br>
  CC      net/ipv4/inet_hashtables.o<br>
  CC      net/ipv6/ip6_input.o<br>
  CC      net/ipv6/addrconf.o<br>
  CC      drivers/devfreq/governor_userspace.o<br>
  CC      drivers/devfreq/governor_cpufreq.o<br>
  CC      net/bluetooth/bnep/sock.o<br>
  CC      net/ipv6/addrlabel.o<br>
  CC      fs/cifs/xattr.o<br>
  CC      drivers/devfreq/governor_msm_adreno_tz.o<br>
  CC      net/ipv4/inet_timewait_sock.o<br>
  CC      fs/crypto/keyinfo.o<br>
  CC      fs/crypto/bio.o<br>
  CC      net/ipv4/inet_connection_sock.o<br>
  CC      net/bluetooth/bnep/netdev.o<br>
  CC      lib/hexdump.o<br>
  CC      fs/cifs/cifsencrypt.o<br>
  CC      sound/soc/soc-compress.o<br>
  CC      net/ipv4/tcp.o<br>
  CC      lib/kasprintf.o<br>
  CC      net/core/sysctl_net_core.o<br>
  CC      fs/cifs/readdir.o<br>
  CC      mm/swap_state.o<br>
  CC      fs/cifs/ioctl.o<br>
  CC      sound/soc/soc-io.o<br>
  CC      lib/bitmap.o<br>
  CC      mm/swapfile.o<br>
  CC      lib/scatterlist.o<br>
  CC      fs/cifs/sess.o<br>
  CC      net/core/dev.o<br>
  CC      net/ipv4/tcp_input.o<br>
  CC      drivers/devfreq/bimc-bwmon.o<br>
  CC      sound/soc/soc-devres.o<br>
  CC      net/ipv4/tcp_output.o<br>
  CC      mm/swap_ratio.o<br>
  CC      net/ipv4/tcp_timer.o<br>
  CC      lib/gcd.o<br>
  LD      fs/crypto/fscrypto.o<br>
  CC      kernel/sched/clock.o<br>
  CC      net/bridge/br_nf_core.o<br>
  LD      fs/crypto/built-in.o<br>
  CC      mm/zcache.o<br>
  CC      mm/dmapool.o<br>
  CC      kernel/sched/cputime.o<br>
  CC      net/ipv6/route.o<br>
  CC      lib/lcm.o<br>
  CC      net/bridge/br_multicast.o<br>
  CC      kernel/sched/idle_task.o<br>
  CC      net/ipv4/tcp_ipv4.o<br>
  CC      net/ipv6/ip6_fib.o<br>
  LD      sound/soc/adi/built-in.o<br>
  CC      lib/list_sort.o<br>
  LD      sound/soc/atmel/built-in.o<br>
  CC      net/bridge/br_mdb.o<br>
  LD      sound/soc/au1x/built-in.o<br>
  CC      net/bridge/br_netfilter.o<br>
  LD      net/bluetooth/bnep/bnep.o<br>
  CC      drivers/dma-buf/dma-buf.o<br>
  LD      net/bluetooth/bnep/built-in.o<br>
  LD      sound/soc/bcm/built-in.o<br>
  CC      fs/cifs/export.o<br>
  LD      sound/soc/blackfin/built-in.o<br>
  LD      sound/soc/cirrus/built-in.o<br>
  CC      fs/cifs/smb1ops.o<br>
  CC      lib/uuid.o<br>
  CC      net/bluetooth/hidp/core.o<br>
  CC      net/bridge/netfilter/ebtables.o<br>
  CC      drivers/devfreq/arm-memlat-mon.o<br>
  CC      net/core/ethtool.o<br>
  CC      net/ipv6/ipv6_sockglue.o<br>
  CC      lib/flex_array.o<br>
  CC      kernel/sched/deadline.o<br>
  CC      lib/iovec.o<br>
  CC      net/bluetooth/hidp/sock.o<br>
  CC      mm/sparse.o<br>
  CC      sound/soc/codecs/msm_hdmi_dba_codec_rx.o<br>
  CC      sound/soc/codecs/wcd9330.o<br>
  CC      fs/cifs/winucase.o<br>
  CC      kernel/sched/stop_task.o<br>
  CC      lib/clz_ctz.o<br>
  CC      sound/soc/codecs/wcd9330-tables.o<br>
  CC      lib/bsearch.o<br>
  CC      lib/find_last_bit.o<br>
  LD      net/bluetooth/hidp/hidp.o<br>
  LD      net/bluetooth/hidp/built-in.o<br>
  CC      lib/find_next_bit.o<br>
  CC      kernel/sched/wait.o<br>
  CC      lib/llist.o<br>
  CC      net/bluetooth/rfcomm/core.o<br>
  CC      lib/memweight.o<br>
  CC      sound/soc/codecs/wcd9335.o<br>
  CC      net/ipv4/tcp_minisocks.o<br>
  CC      sound/soc/codecs/wcdcal-hwdep.o<br>
  CC      lib/kfifo.o<br>
  CC      sound/soc/codecs/audio-ext-clk.o<br>
  CC      kernel/sched/completion.o<br>
  CC      lib/percpu-refcount.o<br>
  CC      drivers/devfreq/msmcci-hwmon.o<br>
  LD      fs/cifs/cifs.o<br>
  CC      kernel/sched/idle.o<br>
  CC      drivers/dma-buf/fence.o<br>
  LD      fs/cifs/built-in.o<br>
  CC      mm/sparse-vmemmap.o<br>
  CC      lib/percpu_ida.o<br>
  CC      lib/hash.o<br>
  CC      sound/soc/codecs/wcd9xxx-resmgr.o<br>
  CC      fs/debugfs/inode.o<br>
  CC      net/ipv4/tcp_cong.o<br>
  CC      drivers/dma-buf/reservation.o<br>
  CC      sound/soc/codecs/wcd9xxx-mbhc.o<br>
  CC      mm/slub.o<br>
  CC      drivers/devfreq/m4m-hwmon.o<br>
  CC      lib/rhashtable.o<br>
  CC      drivers/devfreq/governor_bw_hwmon.o<br>
  CC      net/bluetooth/rfcomm/sock.o<br>
  CC      sound/soc/codecs/wcd9xxx-common.o<br>
  CC      kernel/sched/sched_avg.o<br>
  CC      fs/debugfs/file.o<br>
  CC      net/ipv4/tcp_metrics.o<br>
  CC      lib/reciprocal_div.o<br>
  CC      net/bluetooth/rfcomm/tty.o<br>
  CC      mm/migrate.o<br>
  LD      fs/debugfs/debugfs.o<br>
  LD      fs/debugfs/built-in.o<br>
  CC      sound/soc/codecs/wcd9xxx-common-v2.o<br>
  CC      fs/devpts/inode.o<br>
  CC      net/key/af_key.o<br>
  CC      lib/string_helpers.o<br>
  CC      drivers/devfreq/governor_cache_hwmon.o<br>
  CC      kernel/sched/cpupri.o<br>
  CC      kernel/sched/cpudeadline.o<br>
  CC      sound/soc/codecs/wcd9xxx-resmgr-v2.o<br>
  LD      net/key/built-in.o<br>
  CC      net/ipv6/ndisc.o<br>
  CC      lib/kstrtox.o<br>
  CC      lib/iomap.o<br>
  CC      net/ipv4/tcp_fastopen.o<br>
  CC      sound/soc/codecs/msm8x16-wcd.o<br>
  CC      lib/pci_iomap.o<br>
  CC      mm/memcontrol.o<br>
  LD      net/bluetooth/rfcomm/rfcomm.o<br>
  CC      lib/iomap_copy.o<br>
  LD      net/bluetooth/rfcomm/built-in.o<br>
  CC      net/ipv6/udp.o<br>
  LD      net/bluetooth/bluetooth.o<br>
  LD      net/bluetooth/built-in.o<br>
  CC      drivers/dma-buf/seqno-fence.o<br>
  CC      kernel/sched/stats.o<br>
  CC      drivers/devfreq/governor_gpubw_mon.o<br>
  CC      kernel/sched/cpuacct.o<br>
  CC      sound/soc/codecs/msm8x16-wcd-tables.o<br>
  CC      lib/devres.o<br>
  LD      sound/soc/davinci/built-in.o<br>
  CC      net/ipv4/tcp_offload.o<br>
  CC      lib/hweight.o<br>
  LD      sound/soc/fsl/built-in.o<br>
  LD      sound/soc/dwc/built-in.o<br>
  CC      lib/assoc_array.o<br>
  LD      drivers/dma-buf/built-in.o<br>
  CC      net/ipv4/datagram.o<br>
  LD      sound/soc/generic/built-in.o<br>
  CC      lib/smp_processor_id.o<br>
  CC      sound/soc/codecs/msm8916-wcd-irq.o<br>
  CC      mm/page_cgroup.o<br>
  LD      sound/soc/intel/built-in.o<br>
  LD      fs/devpts/devpts.o<br>
  CC      drivers/edac/edac_stub.o<br>
  CC      drivers/edac/edac_mc.o<br>
  LD      fs/devpts/built-in.o<br>
  LD      sound/soc/jz4740/built-in.o<br>
  CC      lib/bitrev.o<br>
  LD      sound/soc/kirkwood/built-in.o<br>
  LD      fs/exofs/built-in.o<br>
  CC      net/bridge/netfilter/ebtable_broute.o<br>
  CC      drivers/edac/edac_device.o<br>
  CC      net/core/dev_addr_lists.o<br>
  CC      fs/ecryptfs/dentry.o<br>
  CC      lib/crc-ccitt.o<br>
  CC      sound/soc/codecs/wcd_cpe_services.o<br>
  CC      net/ipv4/raw.o<br>
  CC      drivers/devfreq/governor_bw_vbif.o<br>
  CC      sound/soc/msm/msm-pcm-hostless.o<br>
  CC      fs/ext2/balloc.o<br>
  LD      net/bridge/netfilter/built-in.o<br>
  LD      kernel/sched/built-in.o<br>
  CC      lib/crc16.o<br>
  LD      net/bridge/bridge.o<br>
  HOSTCC  lib/gen_crc32table<br>
  LD      net/bridge/built-in.o<br>
  CC      net/ipv4/udp.o<br>
  CC      drivers/devfreq/governor_spdm_bw_hyp.o<br>
  CC      fs/ecryptfs/file.o<br>
  CC      mm/cleancache.o<br>
  CC      sound/soc/msm/msm-dai-fe.o<br>
  CC      lib/libcrc32c.o<br>
  CC      fs/ext2/dir.o<br>
  HZFILE  kernel/time/hz.bc<br>
  CC      kernel/time/timer.o<br>
  CC      net/core/dst.o<br>
  CC      kernel/trace/trace_clock.o<br>
  CC      lib/genalloc.o<br>
  CC      fs/ecryptfs/inode.o<br>
  CC      kernel/trace/ftrace.o<br>
  CC      sound/soc/msm/msm-cpe-lsm.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-slim.o<br>
  CC      drivers/edac/edac_mc_sysfs.o<br>
  CC      mm/page_isolation.o<br>
  CC      fs/ext2/file.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_slimslave.o<br>
  CC      fs/ecryptfs/main.o<br>
  CC      lib/lzo/lzo1x_compress.o<br>
  CC      fs/ext2/ialloc.o<br>
  CC      drivers/edac/edac_pci_sysfs.o<br>
  CC      fs/ext2/inode.o<br>
  CC      kernel/trace/ring_buffer.o<br>
  CC      fs/ext2/ioctl.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-v2.o<br>
  CC      drivers/devfreq/governor_memlat.o<br>
  CC      net/core/netevent.o<br>
  CC      lib/lzo/lzo1x_decompress_safe.o<br>
  CC      kernel/trace/trace.o<br>
  CC      net/core/neighbour.o<br>
  CC      drivers/edac/edac_module.o<br>
  CC      fs/ext2/namei.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-v2.o<br>
  CC      fs/ecryptfs/super.o<br>
  CC      kernel/trace/trace_output.o<br>
  LD      lib/lzo/lzo_compress.o<br>
  LD      lib/lzo/lzo_decompress.o<br>
  LD      lib/lzo/built-in.o<br>
  CC      mm/zbud.o<br>
  LD      sound/soc/mxs/built-in.o<br>
  CC      net/ipv6/udplite.o<br>
  CC      drivers/devfreq/devfreq_devbw.o<br>
  CC      drivers/devfreq/devfreq_simple_dev.o<br>
  LD      sound/soc/nuc900/built-in.o<br>
  CC      drivers/devfreq/devfreq_spdm.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-v2.o<br>
  CC      kernel/trace/trace_seq.o<br>
  CC      lib/mpi/generic_mpih-lshift.o<br>
  CC      kernel/trace/trace_stat.o<br>
  CC      drivers/devfreq/devfreq_spdm_debugfs.o<br>
  CC      kernel/trace/trace_printk.o<br>
  CC      sound/soc/codecs/wcd_cpe_core.o<br>
  CC      lib/mpi/generic_mpih-mul1.o<br>
  CC      fs/ecryptfs/mmap.o<br>
  CC      lib/mpi/generic_mpih-mul2.o<br>
  CC      net/ipv6/raw.o<br>
  CC      drivers/edac/edac_device_sysfs.o<br>
  CC      fs/ext2/super.o<br>
  CC      sound/soc/codecs/wcd-mbhc-v2.o<br>
  LD      drivers/devfreq/built-in.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-compress-q6-v2.o<br>
  CC      fs/ext2/symlink.o<br>
  CC      fs/ecryptfs/read_write.o<br>
  CC      lib/mpi/generic_mpih-mul3.o<br>
  CC      mm/zsmalloc.o<br>
  CC      drivers/edac/edac_pci.o<br>
  CC      fs/ecryptfs/events.o<br>
  CC      lib/mpi/generic_mpih-rshift.o<br>
  CC      fs/ecryptfs/crypto.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-lpa-v2.o<br>
  CC      kernel/time/hrtimer.o<br>
  CC      net/ipv6/icmp.o<br>
  CC      sound/soc/codecs/wsa881x.o<br>
  CC      net/ipv6/mcast.o<br>
  CC      sound/soc/codecs/wsa881x-tables.o<br>
  CC      lib/mpi/generic_mpih-sub1.o<br>
  CC      fs/ext2/xattr.o<br>
  CC      mm/early_ioremap.o<br>
  CC      mm/cma.o<br>
  CC      drivers/edac/cortex_arm64_edac.o<br>
  CC      fs/ecryptfs/keystore.o<br>
  CC      kernel/trace/trace_sched_switch.o<br>
  CC      lib/mpi/generic_mpih-add1.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-afe-v2.o<br>
  CC      kernel/time/itimer.o<br>
  CC      lib/mpi/mpicoder.o<br>
  CC      mm/process_reclaim.o<br>
  CC      sound/soc/codecs/wsa881x-regmap.o<br>
  CC      fs/ecryptfs/kthread.o<br>
  LD      drivers/edac/edac_core.o<br>
  LD      drivers/edac/built-in.o<br>
  CC      net/ipv6/reassembly.o<br>
  CC      net/ipv6/tcp_ipv6.o<br>
  CC      kernel/trace/trace_functions.o<br>
  LD      drivers/firewire/built-in.o<br>
  CC      lib/mpi/mpi-bit.o<br>
  CC      net/ipv4/udplite.o<br>
  CC      lib/mpi/mpi-cmp.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-voip-v2.o<br>
  CC      lib/mpi/mpih-cmp.o<br>
  CC      drivers/firmware/dmi_scan.o<br>
  CC      drivers/firmware/dmi-id.o<br>
  CC      kernel/trace/trace_cpu_freq_switch.o<br>
  CC      lib/mpi/mpih-div.o<br>
  CC      kernel/trace/trace_nop.o<br>
  CC      lib/mpi/mpih-mul.o<br>
  CC      lib/mpi/mpi-pow.o<br>
  CC      lib/mpi/mpiutil.o<br>
  CC      fs/ext2/xattr_user.o<br>
  LD      sound/sparc/built-in.o<br>
  CC      fs/ext2/xattr_trusted.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-voice-v2.o<br>
  LD      sound/spi/built-in.o<br>
  LD      sound/synth/built-in.o<br>
  CC      net/core/rtnetlink.o<br>
  CC      fs/ecryptfs/debug.o<br>
  CC      sound/soc/codecs/wsa881x-analog.o<br>
  CC      kernel/trace/trace_functions_graph.o<br>
  CC      drivers/firmware/efi/efi.o<br>
  CC      net/l2tp/l2tp_core.o<br>
  CC      kernel/trace/blktrace.o<br>
  CC      mm/cma_debug.o<br>
  CC      kernel/trace/trace_events.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-hdmi-v2.o<br>
  CC      kernel/time/posix-timers.o<br>
  LD      fs/ext2/ext2.o<br>
  LD      fs/ext2/built-in.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-lsm-client.o<br>
  CC      kernel/trace/trace_export.o<br>
  CC      drivers/firmware/efi/vars.o<br>
  LD      fs/ecryptfs/ecryptfs.o<br>
  CC      sound/soc/codecs/wsa881x-tables-analog.o<br>
  LD      lib/mpi/mpi.o<br>
  CC      kernel/trace/trace_event_perf.o<br>
  CC      kernel/trace/trace_events_filter.o<br>
  LD      lib/mpi/built-in.o<br>
  LD      fs/ecryptfs/built-in.o<br>
  CC      net/ipv4/udp_offload.o<br>
  CC      lib/reed_solomon/reed_solomon.o<br>
  CC      drivers/firmware/efi/reboot.o<br>
  CC      net/l2tp/l2tp_ppp.o<br>
  CC      kernel/time/posix-cpu-timers.o<br>
  CC      lib/zlib_deflate/deflate.o<br>
  CC      lib/zlib_deflate/deftree.o<br>
  CC      kernel/time/timekeeping.o<br>
  CC      fs/ext3/balloc.o<br>
  CC      net/ipv6/ping.o<br>
  LD      lib/reed_solomon/built-in.o<br>
  CC      lib/zlib_inflate/inffast.o<br>
  CC      net/ipv6/exthdrs.o<br>
  CC      net/ipv6/datagram.o<br>
  CC      lib/zlib_deflate/deflate_syms.o<br>
  CC      fs/ext3/bitmap.o<br>
  LD      mm/built-in.o<br>
  CC      net/ipv4/arp.o<br>
  CC      fs/ext3/dir.o<br>
  CC      sound/soc/codecs/wsa881x-regmap-analog.o<br>
  CC      net/l2tp/l2tp_ip.o<br>
  CC      sound/soc/codecs/wsa881x-irq.o<br>
  CC      fs/ext3/file.o<br>
  LD      lib/zlib_deflate/zlib_deflate.o<br>
  CC      fs/ext3/fsync.o<br>
  LD      lib/zlib_deflate/built-in.o<br>
  CC      lib/zlib_inflate/inflate.o<br>
  CC      net/ipv4/icmp.o<br>
  CC      kernel/time/ntp.o<br>
  CC      kernel/time/clocksource.o<br>
  CC      drivers/firmware/efi/runtime-wrappers.o<br>
  CC      sound/soc/codecs/wsa881x-temp-sensor.o<br>
  CC      lib/zlib_inflate/infutil.o<br>
  CC      net/ipv6/ip6_flowlabel.o<br>
  CC      net/l2tp/l2tp_netlink.o<br>
  CC      lib/zlib_inflate/inftrees.o<br>
  CC      lib/zlib_inflate/inflate_syms.o<br>
  CC      fs/ext3/ialloc.o<br>
  CC      kernel/time/jiffies.o<br>
  LD      drivers/firmware/efi/built-in.o<br>
  CC      kernel/time/timer_list.o<br>
  CC      sound/soc/msm/msm8952.o<br>
  CC      sound/soc/codecs/msm_stub.o<br>
  CC      kernel/freezer.o<br>
  CC      kernel/profile.o<br>
  CC      drivers/firmware/qcom/tz_log.o<br>
  CC      net/l2tp/l2tp_eth.o<br>
  CC      kernel/time/timeconv.o<br>
  LD      lib/zlib_inflate/zlib_inflate.o<br>
  CC      sound/soc/msm/msm-audio-pinctrl.o<br>
  LD      lib/zlib_inflate/built-in.o<br>
  CC      lib/textsearch.o<br>
  CC      kernel/stacktrace.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9330.o<br>
  CC      net/l2tp/l2tp_debugfs.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9335.o<br>
  LD      sound/soc/codecs/audio-ext-clock.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9xxx.o<br>
  LD      drivers/firmware/qcom/built-in.o<br>
  LD      sound/soc/codecs/snd-soc-wcd9xxx-v2.o<br>
  LD      drivers/firmware/built-in.o<br>
  CC      net/ipv6/inet6_connection_sock.o<br>
  LD      sound/soc/codecs/snd-soc-msm8952-wcd.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-host-voice-v2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-audio-effects-q6-v2.o<br>
  LD      sound/soc/codecs/snd-soc-wcd-cpe.o<br>
  CC      net/ipv6/sysctl_net_ipv6.o<br>
  CC      net/ipv6/xfrm6_policy.o<br>
  LD      sound/soc/codecs/snd-soc-wcd-mbhc.o<br>
  CC      drivers/gpio/devres.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x.o<br>
  CC      net/l2tp/l2tp_ip6.o<br>
  CC      lib/ts_kmp.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x-analog.o<br>
  CC      drivers/gpio/gpiolib.o<br>
  CC      net/ipv4/devinet.o<br>
  LD      sound/soc/codecs/snd-soc-wsa881x-sensor.o<br>
  CC      drivers/gpio/gpiolib-legacy.o<br>
  LD      sound/soc/codecs/snd-soc-msm-stub.o<br>
  CC      kernel/trace/trace_events_trigger.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-loopback-v2.o<br>
  CC      kernel/time/posix-clock.o<br>
  LD      sound/soc/codecs/built-in.o<br>
  CC      fs/ext4/balloc.o<br>
  CC      net/ipv6/xfrm6_state.o<br>
  CC      sound/soc/msm/msm8952-slimbus.o<br>
  LD      net/l2tp/built-in.o<br>
  CC      drivers/gpio/gpiolib-of.o<br>
  CC      sound/soc/msm/qdsp6v2/adsp_err.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-dtmf-v2.o<br>
  CC      net/core/utils.o<br>
  CC      kernel/time/alarmtimer.o<br>
  CC      kernel/time/clockevents.o<br>
  CC      lib/ts_bm.o<br>
  CC      drivers/gpio/gpiolib-sysfs.o<br>
  CC      net/ipv6/xfrm6_input.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dai-stub-v2.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-devdep.o<br>
  CC      sound/soc/msm/msm8952-dai-links.o<br>
  CC      kernel/trace/power-traces.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dolby-dap-config.o<br>
  CC      kernel/trace/rpm-traces.o<br>
  CC      fs/ext3/inode.o<br>
  CC      fs/ext4/bitmap.o<br>
  CC      fs/ext4/dir.o<br>
  CC      net/core/link_watch.o<br>
  CC      net/core/filter.o<br>
  CC      drivers/gpio/qpnp-pin.o<br>
  CC      net/ipv6/xfrm6_output.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-ds2-dap-config.o<br>
  CC      fs/ext4/file.o<br>
  CC      fs/ext4/fsync.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-dts-srs-tm-config.o<br>
  CC      net/core/sock_diag.o<br>
  CC      fs/ext4/ialloc.o<br>
  CC      kernel/trace/ipc_logging.o<br>
  CC      fs/ext4/inode.o<br>
  CC      fs/ext3/ioctl.o<br>
  CC      drivers/gpio/gpio-msm-smp2p.o<br>
  CC      lib/ts_fsm.o<br>
  CC      kernel/trace/ipc_logging_debug.o<br>
  CC      drivers/gpio/gpio-msm-smp2p-test.o<br>
  CC      net/ipv4/af_inet.o<br>
  CC      fs/ext4/page-io.o<br>
  CC      lib/percpu_counter.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-qti-pp-config.o<br>
  CC      fs/ext4/ioctl.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_calibration.o<br>
  CC      net/ipv6/xfrm6_protocol.o<br>
  CC      sound/soc/msm/qdsp6v2/audio_cal_utils.o<br>
  CC      sound/soc/msm/qdsp6v2/q6adm.o<br>
  CC      fs/ext3/namei.o<br>
  CC      kernel/time/tick-common.o<br>
  CC      lib/audit.o<br>
  LD      kernel/trace/libftrace.o<br>
  LD      kernel/trace/built-in.o<br>
  CC      kernel/time/tick-broadcast.o<br>
  CC      fs/ext3/super.o<br>
  CC      net/ipv4/igmp.o<br>
  LD      sound/soc/omap/built-in.o<br>
  CC      sound/soc/msm/qdsp6v2/q6afe.o<br>
  LD      sound/soc/pxa/built-in.o<br>
  CC      fs/ext4/namei.o<br>
  CC      sound/soc/msm/qdsp6v2/q6asm.o<br>
  CC      net/ipv4/fib_frontend.o<br>
  CC      sound/soc/msm/qdsp6v2/q6audio-v2.o<br>
  CC      kernel/time/tick-broadcast-hrtimer.o<br>
  CC      sound/soc/msm/qdsp6v2/q6voice.o<br>
  CC      lib/compat_audit.o<br>
  CC      net/ipv6/netfilter.o<br>
  CC      net/core/dev_ioctl.o<br>
  CC      fs/ext4/super.o<br>
  LD      drivers/gpu/drm/bridge/built-in.o<br>
  CC      kernel/time/sched_clock.o<br>
  CC      kernel/time/tick-oneshot.o<br>
  LD      drivers/gpu/drm/i2c/built-in.o<br>
  CC      lib/swiotlb.o<br>
  CC      kernel/time/tick-sched.o<br>
  LD      drivers/gpu/drm/panel/built-in.o<br>
  LD      drivers/gpu/drm/built-in.o<br>
  LD      drivers/gpio/built-in.o<br>
  CC      kernel/time/timer_stats.o<br>
  CC      drivers/gpu/vga/vgaarb.o<br>
  CC      net/ipv4/fib_semantics.o<br>
  CC      sound/soc/msm/qdsp6v2/q6core.o<br>
  CC      sound/soc/msm/qdsp6v2/rtac.o<br>
  CC      drivers/hid/hid-debug.o<br>
  CC      kernel/time/timekeeping_debug.o<br>
  CC      sound/soc/msm/qdsp6v2/q6lsm.o<br>
  CC      fs/ext4/symlink.o<br>
  LD      drivers/gpu/vga/built-in.o<br>
  CC      fs/ext4/hash.o<br>
  CC      fs/ext4/resize.o<br>
  CC      drivers/gpu/msm/kgsl.o<br>
  CC      drivers/hid/hid-core.o<br>
  CC      net/core/tso.o<br>
  CC      drivers/gpu/msm/kgsl_trace.o<br>
  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-noirq.o<br>
  LD      sound/soc/msm/qdsp6v2/snd-soc-qdsp6v2.o<br>
  BC      kernel/time/timeconst.h<br>
  CC      drivers/hid/hid-input.o<br>
  CC      fs/ext4/extents.o<br>
  CC      fs/ext4/ext4_jbd2.o<br>
  CC      drivers/gpu/msm/kgsl_cmdbatch.o<br>
  CC      kernel/time/time.o<br>
  CC      drivers/gpu/msm/kgsl_ioctl.o<br>
  CC      net/core/flow.o<br>
  CC      net/ipv6/fib6_rules.o<br>
  CC      drivers/hid/hidraw.o<br>
  CC      lib/iommu-helper.o<br>
  CC      lib/syscall.o<br>
  LD      sound/soc/msm/qdsp6v2/built-in.o<br>
  CC      net/ipv6/proc.o<br>
  LD      kernel/time/built-in.o<br>
  LD      sound/soc/msm/snd-soc-hostless-pcm.o<br>
  CC      kernel/futex.o<br>
  LD      sound/soc/msm/snd-soc-qdsp6v2.o<br>
  LD      sound/soc/msm/snd-soc-cpe.o<br>
  CC      kernel/futex_compat.o<br>
  LD      sound/soc/msm/snd-soc-msm8x16.o<br>
  LD      sound/soc/msm/built-in.o<br>
  CC      lib/nlattr.o<br>
  CC      lib/checksum.o<br>
  LD      sound/soc/rockchip/built-in.o<br>
  CC      net/core/net-sysfs.o<br>
  CC      net/ipv6/ah6.o<br>
  LD      sound/soc/samsung/built-in.o<br>
  CC      sound/usb/card.o<br>
  CC      net/ipv4/fib_trie.o<br>
  LD      sound/soc/sh/built-in.o<br>
  CC      kernel/smp.o<br>
  CC      net/core/net-procfs.o<br>
  CC      lib/cpu_rmap.o<br>
  LD      sound/soc/sirf/built-in.o<br>
  LD      sound/soc/spear/built-in.o<br>
  LD      sound/soc/tegra/built-in.o<br>
  LD      sound/soc/txx9/built-in.o<br>
  LD      sound/soc/ux500/built-in.o<br>
  LD      sound/soc/snd-soc-core.o<br>
  CC      drivers/hid/uhid.o<br>
  LD      sound/soc/built-in.o<br>
  CC      drivers/hid/hid-generic.o<br>
  CC      net/ipv4/inet_fragment.o<br>
  CC      net/ipv6/esp6.o<br>
  CC      kernel/uid16.o<br>
  CC      sound/usb/clock.o<br>
  CC      sound/last.o<br>
  CC      drivers/hid/hid-apple.o<br>
  CC      net/core/fib_rules.o<br>
  CC      net/core/net-traces.o<br>
  CC      net/core/sockev_nlmcast.o<br>
  CC      net/ipv4/ping.o<br>
  CC      net/ipv4/ip_tunnel_core.o<br>
  CC      kernel/system_keyring.o<br>
  CC      sound/usb/endpoint.o<br>
  CC      drivers/hid/hid-elecom.o<br>
  CC      lib/dynamic_queue_limits.o<br>
  CC      kernel/kallsyms.o<br>
  CC      drivers/hid/hid-magicmouse.o<br>
  CC      drivers/hid/hid-microsoft.o<br>
  CC      drivers/hid/hid-multitouch.o<br>
  CC      sound/usb/format.o<br>
  CC      kernel/compat.o<br>
  CC      kernel/cgroup.o<br>
  CC      net/ipv6/ipcomp6.o<br>
  CC      drivers/hid/usbhid/hid-core.o<br>
  CC      fs/ext3/symlink.o<br>
  CC      lib/strncpy_from_user.o<br>
  CC      fs/ext3/hash.o<br>
  CC      kernel/cgroup_freezer.o<br>
  CC      net/ipv4/gre_offload.o<br>
  CC      net/ipv4/ip_tunnel.o<br>
  CC      drivers/gpu/msm/kgsl_sharedmem.o<br>
  CC      fs/ext3/resize.o<br>
  CC      fs/ext3/ext3_jbd.o<br>
  CC      drivers/gpu/msm/kgsl_pwrctrl.o<br>
  LD      net/core/built-in.o<br>
  CC      drivers/gpu/msm/kgsl_pwrscale.o<br>
  CC      kernel/cpuset.o<br>
  CC      drivers/gpu/msm/kgsl_mmu.o<br>
  CC      fs/ext3/xattr.o<br>
  LD      drivers/hid/hid.o<br>
  CC      fs/ext4/migrate.o<br>
  CC      fs/ext3/xattr_user.o<br>
  CC      fs/ext3/xattr_trusted.o<br>
  CC      net/llc/llc_core.o<br>
  CC      net/ipv4/sysctl_net_ipv4.o<br>
  GZIP    kernel/config_data.gz<br>
  CC      lib/strnlen_user.o<br>
  CC      lib/net_utils.o<br>
  CC      lib/asn1_decoder.o<br>
  CC      net/llc/llc_input.o<br>
  CC      kernel/res_counter.o<br>
  CC      net/llc/llc_output.o<br>
  CC      net/ipv4/sysfs_net_ipv4.o<br>
  CC      sound/usb/helper.o<br>
  CC      sound/usb/mixer.o<br>
  CC      drivers/gpu/msm/kgsl_snapshot.o<br>
  CC      drivers/gpu/msm/kgsl_events.o<br>
  CC      fs/ext4/mballoc.o<br>
  CC      drivers/gpu/msm/kgsl_pool.o<br>
  GEN     lib/oid_registry_data.c<br>
  CC      fs/ext4/block_validity.o<br>
  CC      kernel/stop_machine.o<br>
  CC      kernel/audit.o<br>
perl: warning: Setting locale failed.<br>
perl: warning: Please check that your locale settings:<br>
	LANGUAGE = (unset),<br>
	LC_ALL = (unset),<br>
	LC_CTYPE = "en_US.UTF-8",<br>
	LC_COLLATE = "C",<br>
	LC_MESSAGES = "en_US.UTF-8",<br>
	LC_NUMERIC = "C",<br>
	LANG = (unset)<br>
    are supported and installed on your system.<br>
perl: warning: Falling back to the standard locale ("C").<br>
  CC      lib/ucs2_string.o<br>
  CC      lib/qmi_encdec.o<br>
  CC      lib/argv_split.o<br>
  CC      sound/usb/mixer_quirks.o<br>
  CC      fs/ext4/move_extent.o<br>
  LD      net/llc/llc.o<br>
  CC      net/ipv6/xfrm6_tunnel.o<br>
  CC      lib/bug.o<br>
  LD      net/llc/built-in.o<br>
  CC      net/netlink/af_netlink.o<br>
  CC      kernel/auditfilter.o<br>
  LD      fs/ext3/ext3.o<br>
  CC      lib/clz_tab.o<br>
  CC      fs/ext4/mmp.o<br>
  LD      fs/ext3/built-in.o<br>
  CC      fs/f2fs/dir.o<br>
  CC      lib/cmdline.o<br>
  CC      lib/cpumask.o<br>
  CC      net/ipv6/tunnel6.o<br>
  CC      net/netlink/genetlink.o<br>
  CC      sound/usb/pcm.o<br>
  CC      drivers/hid/usbhid/hid-quirks.o<br>
  CC      sound/usb/proc.o<br>
  CC      fs/fat/cache.o<br>
  CC      lib/ctype.o<br>
  CC      lib/dec_and_lock.o<br>
  CC      fs/fat/dir.o<br>
  CC      fs/ext4/indirect.o<br>
  CC      fs/ext4/extents_status.o<br>
  CC      kernel/auditsc.o<br>
  CC      drivers/hid/usbhid/hiddev.o<br>
  CC      net/netfilter/core.o<br>
  CC      kernel/audit_watch.o<br>
  CC      lib/decompress.o<br>
  LD      net/netlink/built-in.o<br>
  CC      lib/decompress_bunzip2.o<br>
  CC      fs/ext4/xattr.o<br>
  CC      sound/usb/quirks.o<br>
  LD      drivers/hid/usbhid/usbhid.o<br>
  CC      net/ipv6/xfrm6_mode_transport.o<br>
  CC      net/ipv4/proc.o<br>
  CC      net/ipv4/fib_rules.o<br>
  LD      drivers/hid/usbhid/built-in.o<br>
  CC      sound/usb/stream.o<br>
  CC      sound/usb/midi.o<br>
  LD      drivers/hid/built-in.o<br>
  CC      lib/decompress_inflate.o<br>
  CC      fs/fat/fatent.o<br>
  CC      drivers/gpu/msm/kgsl_iommu.o<br>
  CC      fs/ext4/xattr_user.o<br>
  LD      drivers/hsi/clients/built-in.o<br>
  LD      drivers/hsi/controllers/built-in.o<br>
  CC      fs/fuse/dev.o<br>
  CC      kernel/audit_tree.o<br>
  LD      sound/usb/6fire/built-in.o<br>
  LD      drivers/hsi/built-in.o<br>
  LD      sound/usb/bcd2000/built-in.o<br>
  LD      sound/usb/caiaq/built-in.o<br>
  CC      drivers/gpu/msm/kgsl_debugfs.o<br>
  LD      sound/usb/hiface/built-in.o<br>
  CC      net/ipv4/udp_tunnel.o<br>
  LD      sound/usb/misc/built-in.o<br>
  CC      fs/jbd/transaction.o<br>
  CC      drivers/gpu/msm/kgsl_sync.o<br>
  CC      fs/jbd/commit.o<br>
  CC      lib/decompress_unlzma.o<br>
  LD      sound/usb/usx2y/built-in.o<br>
  LD      sound/usb/snd-usb-audio.o<br>
  CC      fs/ext4/xattr_trusted.o<br>
  LD      sound/usb/snd-usbmidi-lib.o<br>
  LD      sound/usb/built-in.o<br>
  CC      fs/fuse/dir.o<br>
  CC      drivers/gpu/msm/kgsl_compat.o<br>
  LD      sound/soundcore.o<br>
  CC      kernel/seccomp.o<br>
  CC      fs/ext4/inline.o<br>
  CC      lib/dump_stack.o<br>
  LD      sound/built-in.o<br>
  CC      drivers/gpu/msm/adreno_ioctl.o<br>
  CC      fs/jbd/recovery.o<br>
  CC      fs/f2fs/file.o<br>
  CC      fs/f2fs/inode.o<br>
  CC      fs/ext4/xattr_security.o<br>
  CC      fs/fuse/file.o<br>
  CC      net/netfilter/nf_log.o<br>
  CC      net/ipv6/xfrm6_mode_tunnel.o<br>
  CC      net/ipv4/ah4.o<br>
  CC      lib/earlycpio.o<br>
  CC      net/ipv4/esp4.o<br>
  CC      fs/fuse/inode.o<br>
  CC      kernel/utsname_sysctl.o<br>
  CC      net/ipv4/ipcomp.o<br>
  CC      lib/extable.o<br>
  CC      fs/fat/file.o<br>
  CC      kernel/tracepoint.o<br>
  LD      fs/ext4/ext4.o<br>
  LD      fs/ext4/built-in.o<br>
  CC      net/ipv4/xfrm4_tunnel.o<br>
  CC      net/netfilter/nf_queue.o<br>
  CC      fs/fuse/control.o<br>
  CC      fs/fuse/shortcircuit.o<br>
  CC      kernel/elfcore.o<br>
  CC      lib/fdt.o<br>
  CC      lib/fdt_empty_tree.o<br>
  CC      lib/fdt_ro.o<br>
  CC      kernel/irq_work.o<br>
  CC      kernel/cpu_pm.o<br>
  CC      drivers/gpu/msm/adreno_ringbuffer.o<br>
  CC      lib/fdt_rw.o<br>
  CC      drivers/gpu/msm/adreno_drawctxt.o<br>
  CC      fs/fat/inode.o<br>
  CC      drivers/gpu/msm/adreno_dispatch.o<br>
  CERTS   kernel/x509_certificate_list<br>
  CC      fs/fat/misc.o<br>
  CC      net/netfilter/nf_sockopt.o<br>
  CC      fs/jbd/checkpoint.o<br>
  CC      fs/jbd/revoke.o<br>
  CC      drivers/gpu/msm/adreno_snapshot.o<br>
  CHK     kernel/config_data.h<br>
  CC      fs/fat/nfs.o<br>
  UPD     kernel/config_data.h<br>
  CC      fs/fat/namei_vfat.o<br>
  AS      kernel/system_certificates.o<br>
  CC      fs/fat/namei_msdos.o<br>
  CC      fs/f2fs/namei.o<br>
  CC      net/ipv6/xfrm6_mode_beet.o<br>
  CC      fs/jbd/journal.o<br>
  CC      lib/fdt_strerror.o<br>
  CC      kernel/configs.o<br>
  CC      net/ipv6/mip6.o<br>
  CC      drivers/gpu/msm/adreno_coresight.o<br>
  CC      net/netfilter/nfnetlink.o<br>
  CC      net/ipv6/netfilter/ip6_tables.o<br>
  CC      lib/fdt_sw.o<br>
  LD      fs/jbd/jbd.o<br>
  CC      net/ipv6/netfilter/ip6table_filter.o<br>
  LD      fs/jbd/built-in.o<br>
  LD      fs/fuse/fuse.o<br>
  CC      net/ipv6/sit.o<br>
  LD      fs/fuse/built-in.o<br>
  CC      net/ipv6/netfilter/ip6table_mangle.o<br>
  LD      kernel/built-in.o<br>
  CC      net/netfilter/nfnetlink_queue_core.o<br>
  CC      net/netfilter/nfnetlink_log.o<br>
  CC      net/netfilter/nf_conntrack_core.o<br>
  CC      net/netfilter/nf_conntrack_standalone.o<br>
  CC      drivers/gpu/msm/adreno_trace.o<br>
  CC      net/ipv4/tunnel4.o<br>
  CC      lib/fdt_wip.o<br>
  CC      net/ipv4/xfrm4_mode_transport.o<br>
  CC      net/ipv4/xfrm4_mode_tunnel.o<br>
  CC      drivers/gpu/msm/adreno_a3xx.o<br>
  CC      net/ipv6/netfilter/ip6table_raw.o<br>
  LD      fs/fat/fat.o<br>
  CC      net/ipv6/addrconf_core.o<br>
  CC      net/ipv6/exthdrs_core.o<br>
  CC      lib/flex_proportions.o<br>
  CC      net/netfilter/nf_conntrack_expect.o<br>
  CC      fs/jbd2/transaction.o<br>
  CC      net/netfilter/nf_conntrack_helper.o<br>
  CC      drivers/gpu/msm/adreno_a4xx.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_l3proto_ipv6.o<br>
  CC      net/ipv4/ipconfig.o<br>
  LD      fs/fat/msdos.o<br>
  LD      fs/fat/vfat.o<br>
  LD      fs/fat/built-in.o<br>
  CC      net/ipv4/netfilter.o<br>
  CC      fs/jbd2/commit.o<br>
  CC      drivers/hwmon/hwmon.o<br>
  CC      lib/idr.o<br>
  CC      drivers/gpu/msm/adreno_a5xx.o<br>
  CC      drivers/hwmon/epm_adc.o<br>
  CC      lib/int_sqrt.o<br>
  CC      fs/jbd2/recovery.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_proto_icmpv6.o<br>
  CC      net/ipv4/inet_diag.o<br>
  CC      drivers/hwmon/qpnp-adc-voltage.o<br>
  CC      lib/ioremap.o<br>
  CC      fs/f2fs/hash.o<br>
  CC      fs/f2fs/super.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4_compat.o<br>
  CC      net/ipv6/netfilter/nf_defrag_ipv6_hooks.o<br>
  CC      fs/jbd2/checkpoint.o<br>
  CC      fs/kernfs/mount.o<br>
  CC      fs/kernfs/inode.o<br>
  CC      drivers/gpu/msm/adreno_a3xx_snapshot.o<br>
  CC      fs/kernfs/dir.o<br>
  CC      fs/kernfs/file.o<br>
  CC      fs/jbd2/revoke.o<br>
  CC      drivers/hwmon/qpnp-adc-common.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4.o<br>
  CC      net/ipv6/netfilter/nf_conntrack_reasm.o<br>
  CC      fs/f2fs/inline.o<br>
  CC      lib/irq_regs.o<br>
  CC      lib/is_single_threaded.o<br>
  CC      drivers/gpu/msm/adreno_a4xx_snapshot.o<br>
  CC      lib/klist.o<br>
  CC      fs/f2fs/checkpoint.o<br>
  CC      drivers/gpu/msm/adreno_a5xx_snapshot.o<br>
  CC      drivers/gpu/msm/adreno_a4xx_preempt.o<br>
  CC      net/netfilter/nf_conntrack_proto.o<br>
  CC      fs/kernfs/symlink.o<br>
  CC      net/netfilter/nf_conntrack_l3proto_generic.o<br>
  CC      drivers/hwspinlock/hwspinlock_core.o<br>
  CC      net/ipv6/netfilter/nf_log_ipv6.o<br>
  CC      net/ipv4/netfilter/nf_conntrack_proto_icmp.o<br>
  LD      drivers/idle/built-in.o<br>
  CC      net/ipv4/netfilter/nf_nat_l3proto_ipv4.o<br>
  CC      drivers/i2c/i2c-boardinfo.o<br>
  CC      lib/kobject.o<br>
  CC      fs/f2fs/gc.o<br>
  CC      drivers/gpu/msm/adreno_a5xx_preempt.o<br>
  CC      net/netfilter/nf_conntrack_proto_generic.o<br>
  CC      net/netfilter/nf_conntrack_proto_tcp.o<br>
  CC      lib/kobject_uevent.o<br>
  CC      net/ipv4/netfilter/nf_nat_proto_icmp.o<br>
  CC      fs/jbd2/journal.o<br>
  CC      drivers/gpu/msm/adreno_sysfs.o<br>
  CC      fs/nls/nls_base.o<br>
  LD      fs/kernfs/built-in.o<br>
  CC      fs/nls/nls_cp437.o<br>
  CC      fs/nls/nls_cp936.o<br>
  CC      drivers/gpu/msm/adreno.o<br>
  CC      drivers/i2c/i2c-core.o<br>
  CC      lib/md5.o<br>
  CC      drivers/hwspinlock/msm_remote_spinlock.o<br>
  CC      fs/nls/nls_cp950.o<br>
  CC      lib/plist.o<br>
  LD      drivers/hwspinlock/built-in.o<br>
  CC      lib/proportions.o<br>
  CC      lib/radix-tree.o<br>
  CC      drivers/i2c/i2c-dev.o<br>
  CC      lib/ratelimit.o<br>
  CC      lib/rbtree.o<br>
  CC      drivers/gpu/msm/adreno_cp_parser.o<br>
  CC      net/ipv4/tcp_diag.o<br>
  CC      drivers/hwmon/qpnp-adc-current.o<br>
  CC      net/ipv4/tcp_cubic.o<br>
  CC      drivers/gpu/msm/adreno_perfcounter.o<br>
  CC      drivers/gpu/msm/adreno_iommu.o<br>
  CC      drivers/gpu/msm/adreno_debugfs.o<br>
  CC      net/netfilter/nf_conntrack_proto_udp.o<br>
  LD      drivers/hwmon/built-in.o<br>
  CC      fs/nls/nls_ascii.o<br>
  CC      net/netfilter/nf_conntrack_extend.o<br>
  CC      lib/sha1.o<br>
  CC      net/ipv4/xfrm4_policy.o<br>
  CC      net/ipv6/netfilter/nf_reject_ipv6.o<br>
  CC      net/ipv4/netfilter/nf_defrag_ipv4.o<br>
  CC      fs/nls/nls_iso8859-1.o<br>
  CC      lib/show_mem.o<br>
  CC      fs/nls/nls_utf8.o<br>
  CC      lib/string.o<br>
  CC      drivers/gpu/msm/adreno_profile.o<br>
  CC      drivers/gpu/msm/adreno_compat.o<br>
  CC      lib/timerqueue.o<br>
  CC      lib/vsprintf.o<br>
  LD      fs/nls/built-in.o<br>
  CC      net/ipv6/netfilter/ip6t_rpfilter.o<br>
  CC      net/ipv4/netfilter/nf_log_ipv4.o<br>
  CC      fs/notify/fsnotify.o<br>
  CC      fs/f2fs/data.o<br>
  CC      net/ipv4/netfilter/nf_reject_ipv4.o<br>
  CC      net/ipv4/netfilter/nf_nat_h323.o<br>
  CC      drivers/input/input.o<br>
  CC      net/ipv4/netfilter/nf_nat_pptp.o<br>
  LD      drivers/gpu/msm/msm_kgsl_core.o<br>
  CC      net/ipv4/xfrm4_state.o<br>
  CC      fs/notify/notification.o<br>
  GEN     lib/crc32table.h<br>
  CC      lib/oid_registry.o<br>
  CC      fs/f2fs/node.o<br>
  CC      drivers/input/input-compat.o<br>
  CC      drivers/input/input-mt.o<br>
  CC      fs/f2fs/segment.o<br>
  CC      fs/f2fs/recovery.o<br>
  CC      fs/notify/group.o<br>
  CC      net/netfilter/nf_conntrack_acct.o<br>
  CC      net/netfilter/nf_conntrack_seqadj.o<br>
  CC      fs/notify/inode_mark.o<br>
  CC      drivers/i2c/i2c-mux.o<br>
  CC      fs/notify/mark.o<br>
  CC      fs/f2fs/shrinker.o<br>
  LD      drivers/i2c/algos/built-in.o<br>
  AR      lib/lib.a<br>
  LD      drivers/gpu/msm/msm_adreno.o<br>
  CC      net/packet/af_packet.o<br>
  CC      lib/crc32.o<br>
  LD      drivers/gpu/msm/built-in.o<br>
  CC      fs/f2fs/extent_cache.o<br>
  CC      fs/notify/vfsmount_mark.o<br>
  LD      drivers/gpu/built-in.o<br>
  CC      net/ipv6/netfilter/ip6t_REJECT.o<br>
  CC      fs/notify/fdinfo.o<br>
  CC      drivers/input/ff-core.o<br>
  CC      drivers/i2c/busses/i2c-msm-v2.o<br>
  LD      net/ipv6/netfilter/nf_conntrack_ipv6.o<br>
  LD      net/ipv6/netfilter/nf_defrag_ipv6.o<br>
  LD      net/ipv6/netfilter/built-in.o<br>
  LD      fs/jbd2/jbd2.o<br>
  CC      net/ipv6/ip6_checksum.o<br>
  LD      fs/jbd2/built-in.o<br>
  LD      drivers/i2c/muxes/built-in.o<br>
  CC      net/ipv4/xfrm4_input.o<br>
  CC      drivers/input/mousedev.o<br>
  CC      net/ipv4/xfrm4_output.o<br>
  CC      fs/f2fs/debug.o<br>
  CC      fs/notify/dnotify/dnotify.o<br>
  LD      lib/built-in.o<br>
  CC      net/rfkill/core.o<br>
  LD      drivers/i2c/busses/built-in.o<br>
  CC      drivers/input/evdev.o<br>
  LD      drivers/i2c/built-in.o<br>
  CC      drivers/input/serio/serio.o<br>
  CC      net/ipv4/netfilter/nf_nat_masquerade_ipv4.o<br>
  CC      drivers/input/serio/serport.o<br>
  CC      fs/proc/task_mmu.o<br>
  CC      drivers/input/serio/libps2.o<br>
  CC      net/netfilter/nf_conntrack_ecache.o<br>
  CC      net/ipv6/ip6_icmp.o<br>
  LD      fs/notify/dnotify/built-in.o<br>
  CC      net/ipv6/output_core.o<br>
  LD      fs/notify/fanotify/built-in.o<br>
  LD      net/rfkill/rfkill.o<br>
  CC      fs/notify/inotify/inotify_fsnotify.o<br>
  LD      net/rfkill/built-in.o<br>
  CC      fs/notify/inotify/inotify_user.o<br>
  CC      fs/f2fs/xattr.o<br>
  CC      fs/proc/inode.o<br>
  CC      net/ipv4/netfilter/nf_nat_proto_gre.o<br>
  CC      drivers/input/fingerprint/fpc/fpc1020_tee.o<br>
  CC      net/ipv4/netfilter/ip_tables.o<br>
  CC      net/ipv4/netfilter/iptable_filter.o<br>
  CC      net/ipv4/netfilter/iptable_mangle.o<br>
  CC      net/netfilter/nf_conntrack_proto_dccp.o<br>
  CC      fs/proc/root.o<br>
  CC      drivers/input/joystick/xpad.o<br>
  LD      drivers/input/fingerprint/fpc/built-in.o<br>
  CC      net/ipv4/netfilter/iptable_nat.o<br>
  CC      drivers/input/fingerprint/goodix/gf_spi.o<br>
  CC      drivers/input/fingerprint/goodix/platform.o<br>
  CC      drivers/input/fingerprint/goodix/netlink.o<br>
  CC      fs/f2fs/acl.o<br>
  CC      fs/proc/base.o<br>
  CC      net/ipv4/xfrm4_protocol.o<br>
  LD      drivers/input/serio/built-in.o<br>
  LD      fs/notify/inotify/built-in.o<br>
  CC      fs/pstore/inode.o<br>
  LD      fs/notify/built-in.o<br>
  CC      fs/pstore/platform.o<br>
  CC      net/rmnet_data/rmnet_data_main.o<br>
  CC      net/ipv6/protocol.o<br>
  CC      net/ipv4/netfilter/iptable_raw.o<br>
  CC      fs/proc/generic.o<br>
  CC      fs/proc/array.o<br>
  CC      net/sched/sch_generic.o<br>
  LD      fs/f2fs/f2fs.o<br>
  LD      fs/f2fs/built-in.o<br>
  CC      net/sched/sch_mq.o<br>
  CC      fs/pstore/ftrace.o<br>
  CC      net/sched/sch_api.o<br>
  CC      fs/quota/dquot.o<br>
  CC      net/unix/af_unix.o<br>
  CC      net/unix/garbage.o<br>
  CC      net/rmnet_data/rmnet_data_config.o<br>
  CC      net/rmnet_data/rmnet_data_vnd.o<br>
  CC      net/unix/sysctl_net_unix.o<br>
  CC      fs/quota/quota.o<br>
  CC      net/wireless/core.o<br>
  CC      fs/quota/kqid.o<br>
  CC      net/ipv4/netfilter/iptable_security.o<br>
  CC      fs/pstore/ram.o<br>
  LD      drivers/input/joystick/built-in.o<br>
  CC      fs/pstore/ram_core.o<br>
  CC      net/wireless/sysfs.o<br>
  CC      net/wireless/radiotap.o<br>
  LD      drivers/input/fingerprint/goodix/built-in.o<br>
  CC      net/wireless/util.o<br>
  CC      net/wireless/reg.o<br>
  CC      net/netfilter/nf_conntrack_proto_gre.o<br>
  LD      drivers/input/fingerprint/built-in.o<br>
  CC      drivers/input/keyboard/atkbd.o<br>
  LD      fs/quota/built-in.o<br>
  CC      drivers/input/keyboard/gpio_keys.o<br>
  CC      net/ipv6/ip6_offload.o<br>
  CC      net/netfilter/nf_conntrack_proto_sctp.o<br>
  CC      fs/proc/fd.o<br>
  CC      net/netfilter/nf_conntrack_proto_udplite.o<br>
  CC      net/ipv4/netfilter/ipt_ah.o<br>
  CC      net/wireless/scan.o<br>
  CC      fs/proc/proc_tty.o<br>
  CC      net/ipv4/netfilter/ipt_MASQUERADE.o<br>
  LD      net/packet/built-in.o<br>
  CC      fs/proc/cmdline.o<br>
  CC      net/xfrm/xfrm_policy.o<br>
  CC      net/ipv6/tcpv6_offload.o<br>
  CC      net/ipv6/udp_offload.o<br>
  LD      fs/pstore/pstore.o<br>
  CC      fs/proc/consoles.o<br>
  CC      net/xfrm/xfrm_state.o<br>
  CC      net/netfilter/nf_conntrack_netlink.o<br>
  CC      net/ipv6/exthdrs_offload.o<br>
  LD      net/unix/unix.o<br>
  LD      net/unix/built-in.o<br>
  CC      net/netfilter/nf_conntrack_amanda.o<br>
  LD      fs/pstore/ramoops.o<br>
  LD      fs/pstore/built-in.o<br>
  CC      net/sched/sch_blackhole.o<br>
  CC      fs/ramfs/inode.o<br>
  CC      net/ipv4/netfilter/ipt_NATTYPE.o<br>
  CC      net/sched/cls_api.o<br>
  CC      net/rmnet_data/rmnet_data_handlers.o<br>
  CC      fs/proc/cpuinfo.o<br>
  CC      fs/ramfs/file-mmu.o<br>
  CC      fs/proc/devices.o<br>
  CC      net/rmnet_data/rmnet_map_data.o<br>
  CC      net/rmnet_data/rmnet_map_command.o<br>
  CC      fs/proc/interrupts.o<br>
  CC      fs/proc/loadavg.o<br>
  CC      fs/proc/meminfo.o<br>
  CC      net/sched/act_api.o<br>
  LD      drivers/input/keyboard/built-in.o<br>
  CC      net/wireless/nl80211.o<br>
  CC      drivers/input/misc/gpio_event.o<br>
  LD      fs/ramfs/ramfs.o<br>
  LD      fs/ramfs/built-in.o<br>
  CC      net/ipv6/inet6_hashtables.o<br>
  CC      fs/sdcardfs/dentry.o<br>
  CC      drivers/iommu/iommu.o<br>
  CC      net/ipv6/ip6_udp_tunnel.o<br>
  CC      drivers/iommu/iommu-traces.o<br>
  CC      net/netfilter/nf_conntrack_ftp.o<br>
  CC      drivers/input/misc/gpio_matrix.o<br>
  CC      fs/proc/stat.o<br>
  CC      net/netfilter/nf_conntrack_h323_main.o<br>
  CC      fs/proc/uptime.o<br>
  CC      fs/proc/version.o<br>
  CC      net/ipv4/netfilter/ipt_REJECT.o<br>
  CC      drivers/input/misc/gpio_input.o<br>
  CC      drivers/input/misc/gpio_output.o<br>
  LD      net/ipv6/ipv6.o<br>
  CC      drivers/input/misc/gpio_axis.o<br>
  LD      net/ipv6/built-in.o<br>
  CC      net/compat.o<br>
  CC      net/wireless/mlme.o<br>
  CC      net/sysctl_net.o<br>
  CC      fs/sdcardfs/file.o<br>
  CC      fs/sdcardfs/inode.o<br>
  CC      drivers/input/misc/hbtp_input.o<br>
  CC      fs/sdcardfs/main.o<br>
  CC      net/rmnet_data/rmnet_data_stats.o<br>
  CC      fs/sdcardfs/super.o<br>
  CC      net/ipv4/netfilter/arp_tables.o<br>
  CC      fs/proc/softirqs.o<br>
  CC      fs/sdcardfs/lookup.o<br>
  CC      net/ipv4/netfilter/arpt_mangle.o<br>
  CC      net/ipv4/netfilter/arptable_filter.o<br>
  CC      net/sched/sch_fifo.o<br>
  CC      net/activity_stats.o<br>
  CC      net/sched/sch_htb.o<br>
  CC      net/sched/sch_prio.o<br>
  CC      net/wireless/ibss.o<br>
  CC      drivers/iommu/iommu-sysfs.o<br>
  CC      fs/proc/namespaces.o<br>
  CC      net/wireless/sme.o<br>
  CC      net/wireless/chan.o<br>
  LD      net/ipv4/netfilter/nf_conntrack_ipv4.o<br>
  LD      net/ipv4/netfilter/nf_nat_ipv4.o<br>
  LD      drivers/input/tablet/built-in.o<br>
  CC      drivers/input/keyreset.o<br>
  LD      net/ipv4/netfilter/built-in.o<br>
  CC      drivers/iommu/msm_dma_iommu_mapping.o<br>
  CC      net/sched/cls_u32.o<br>
  CC      net/netfilter/nf_conntrack_h323_asn1.o<br>
  LD      net/ipv4/built-in.o<br>
  CC      fs/proc/self.o<br>
  CC      drivers/iommu/io-pgtable.o<br>
  CC      drivers/input/touchscreen/of_touchscreen.o<br>
  CC      drivers/iommu/io-pgtable-arm.o<br>
  CC      fs/sdcardfs/mmap.o<br>
  CC      net/wireless/ethtool.o<br>
  CC      net/wireless/mesh.o<br>
  CC      drivers/input/misc/hbtp_vm.o<br>
  LD      net/rmnet_data/rmnet_data.o<br>
  CC      drivers/input/touchscreen/ft5435/ft5435_ts.o<br>
  CC      net/xfrm/xfrm_hash.o<br>
  LD      net/rmnet_data/built-in.o<br>
  CC      net/wireless/ap.o<br>
  CC      fs/sdcardfs/packagelist.o<br>
  CC      drivers/input/touchscreen/gt9xx_mido/gt9xx.o<br>
  CC      drivers/input/touchscreen/gt9xx_mido/gt9xx_update.o<br>
  CC      drivers/input/touchscreen/gt9xx_mido/goodix_tool.o<br>
  CC      drivers/input/misc/keychord.o<br>
  CC      fs/proc/thread_self.o<br>
  CC      fs/sdcardfs/derived_perm.o<br>
  CC      fs/proc/proc_sysctl.o<br>
  CC      net/netfilter/nf_conntrack_irc.o<br>
  CC      fs/proc/proc_net.o<br>
  CC      net/netfilter/nf_conntrack_broadcast.o<br>
  LD      fs/sdcardfs/sdcardfs.o<br>
  CC      fs/proc/kmsg.o<br>
  CC      net/wireless/trace.o<br>
  CC      drivers/irqchip/irqchip.o<br>
  CC      drivers/input/misc/uinput.o<br>
  LD      fs/sdcardfs/built-in.o<br>
  CC      drivers/irqchip/irq-gic.o<br>
  CC      fs/proc/page.o<br>
  CC      drivers/leds/led-core.o<br>
  CC      net/netfilter/nf_conntrack_netbios_ns.o<br>
  CC      drivers/leds/led-class.o<br>
  LD      drivers/input/touchscreen/gt9xx_mido/built-in.o<br>
  CC      net/xfrm/xfrm_input.o<br>
  CC      drivers/input/touchscreen/synaptics_dsx/synaptics_dsx_i2c.o<br>
  CC      drivers/irqchip/irq-gic-common.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc.o<br>
  CC      net/sched/cls_fw.o<br>
  LD      drivers/input/touchscreen/synaptics_dsx/built-in.o<br>
  CC      drivers/iommu/io-pgtable-msm-secure.o<br>
  CC      drivers/input/touchscreen/synaptics_dsx_2.6/synaptics_dsx_i2c.o<br>
  CC      drivers/iommu/of_iommu.o<br>
  CC      net/sched/cls_flow.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_misc.o<br>
  CC      net/netfilter/nf_conntrack_pptp.o<br>
  LD      drivers/input/misc/built-in.o<br>
  CC      net/netfilter/nf_conntrack_sane.o<br>
  CC      net/netfilter/nf_conntrack_tftp.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_sys.o<br>
  CC      drivers/iommu/iommu-debug.o<br>
  LD      fs/proc/proc.o<br>
  CC      net/wireless/wext-core.o<br>
  LD      drivers/input/touchscreen/synaptics_dsx_2.6/built-in.o<br>
  CC      drivers/irqchip/irq-gic-v2m.o<br>
  CC      drivers/leds/led-triggers.o<br>
  CC      net/wireless/wext-proc.o<br>
  LD      fs/proc/built-in.o<br>
  CC      net/netfilter/nf_log_common.o<br>
  CC      drivers/iommu/msm_iommu.o<br>
  CC      net/sched/ematch.o<br>
  CC      fs/eventpoll.o<br>
  CC      fs/anon_inodes.o<br>
  CC      fs/sysfs/file.o<br>
  CC      fs/signalfd.o<br>
  CC      fs/timerfd.o<br>
  CC      net/netfilter/nf_nat_core.o<br>
  CC      drivers/leds/leds-qpnp.o<br>
  CC      drivers/leds/leds-qpnp-flash.o<br>
  CC      fs/eventfd.o<br>
  CC      drivers/iommu/msm_iommu_domains.o<br>
  CC      fs/aio.o<br>
  CC      drivers/iommu/msm_iommu_mapping.o<br>
  CC      drivers/leds/leds-qpnp-wled.o<br>
  CC      net/xfrm/xfrm_output.o<br>
  CC      drivers/irqchip/irq-gic-v3.o<br>
  CC      drivers/leds/leds-aw2013.o<br>
  LD      drivers/leds/trigger/built-in.o<br>
  CC      fs/locks.o<br>
  CC      drivers/irqchip/irq-gic-v3-its.o<br>
  CC      net/netfilter/nf_nat_proto_unknown.o<br>
  CC      fs/sysfs/dir.o<br>
  CC      drivers/irqchip/irq-gic-v3-its-pci-msi.o<br>
  CC      net/xfrm/xfrm_sysctl.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_update.o<br>
  CC      fs/sysfs/symlink.o<br>
  CC      net/sched/em_cmp.o<br>
  CC      net/sched/em_nbyte.o<br>
  LD      drivers/input/touchscreen/ft5435/built-in.o<br>
  CC      net/sched/em_u32.o<br>
  CC      net/sched/em_meta.o<br>
  CC      drivers/iommu/msm_iommu-v1.o<br>
  CC      net/wireless/wext-spy.o<br>
  CC      fs/sysfs/mount.o<br>
  CC      net/xfrm/xfrm_replay.o<br>
  CC      net/wireless/wext-priv.o<br>
  CC      net/netfilter/nf_nat_proto_common.o<br>
  CC      net/xfrm/xfrm_proc.o<br>
  CC      fs/sysfs/group.o<br>
  CC      drivers/iommu/msm_iommu_dev-v1.o<br>
  CC      net/sched/em_text.o<br>
  CC      drivers/irqchip/irq-msm.o<br>
  CC      drivers/irqchip/msm_show_resume_irq.o<br>
  CC      net/xfrm/xfrm_algo.o<br>
  CC      net/xfrm/xfrm_user.o<br>
  CC      net/xfrm/xfrm_ipcomp.o<br>
  CC      net/netfilter/nf_nat_proto_udp.o<br>
  LD      drivers/irqchip/built-in.o<br>
  CC      net/netfilter/nf_nat_proto_tcp.o<br>
  LD      drivers/lguest/built-in.o<br>
  LD      drivers/macintosh/built-in.o<br>
  LD      drivers/leds/built-in.o<br>
  LD      fs/sysfs/built-in.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_tracking.o<br>
  CC      net/wireless/regdb.o<br>
  CC      fs/compat.o<br>
  CC      drivers/mfd/mfd-core.o<br>
  CC      drivers/mfd/wcd9xxx-core.o<br>
  CC      drivers/md/dm-uevent.o<br>
  CC      net/netfilter/nf_nat_helper.o<br>
  CC      net/netfilter/nf_nat_proto_dccp.o<br>
  CC      drivers/iommu/msm_iommu_sec.o<br>
  LD      drivers/media/common/b2c2/built-in.o<br>
  LD      drivers/media/common/saa7146/built-in.o<br>
  CC      drivers/input/touchscreen/ist3038c/ist30xxc_cmcs.o<br>
  CC      fs/compat_ioctl.o<br>
  CC      drivers/md/dm.o<br>
  CC      fs/binfmt_script.o<br>
  LD      drivers/media/common/siano/built-in.o<br>
  CC      drivers/mfd/wcd9xxx-irq.o<br>
  LD      drivers/media/common/built-in.o<br>
  CC      drivers/md/dm-table.o<br>
  LD      drivers/media/firewire/built-in.o<br>
  CC      net/netfilter/nf_nat_proto_udplite.o<br>
  CC      net/netfilter/nf_nat_proto_sctp.o<br>
  CC      drivers/mfd/wcd9xxx-slimslave.o<br>
  LD      drivers/media/i2c/soc_camera/built-in.o<br>
  CC      drivers/media/i2c/ir-kbd-i2c.o<br>
  LD      net/sched/built-in.o<br>
  CC      fs/binfmt_elf.o<br>
  LD      drivers/input/touchscreen/ist3038c/built-in.o<br>
  CC      fs/compat_binfmt_elf.o<br>
  LD      drivers/media/mmc/siano/built-in.o<br>
  LD      drivers/input/touchscreen/built-in.o<br>
  CC      drivers/input/keycombo.o<br>
  LD      drivers/media/mmc/built-in.o<br>
  CC      fs/mbcache.o<br>
  CC      fs/posix_acl.o<br>
  CC      drivers/md/dm-target.o<br>
  LD      drivers/media/i2c/built-in.o<br>
  LD      drivers/media/parport/built-in.o<br>
  CC      drivers/misc/uid_stat.o<br>
  CC      drivers/mfd/wcd9xxx-core-resource.o<br>
  CC      drivers/mmc/card/block.o<br>
  LD      drivers/media/pci/b2c2/built-in.o<br>
  LD      drivers/input/input-core.o<br>
  LD      drivers/input/built-in.o<br>
  LD      drivers/media/platform/msm/broadcast/built-in.o<br>
  LD      drivers/media/pci/ddbridge/built-in.o<br>
  CC      drivers/mmc/core/core.o<br>
  LD      drivers/media/pci/dm1105/built-in.o<br>
  LD      drivers/media/pci/mantis/built-in.o<br>
  LD      drivers/media/pci/ngene/built-in.o<br>
  LD      net/wireless/cfg80211.o<br>
  LD      drivers/media/pci/pluto2/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/msm_v4l2_vidc.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_dev.o<br>
  LD      drivers/media/pci/pt1/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/camera/camera.o<br>
  LD      net/wireless/built-in.o<br>
  LD      drivers/media/pci/pt3/built-in.o<br>
  LD      drivers/media/pci/saa7146/built-in.o<br>
  LD      drivers/media/pci/ttpci/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/msm_camera_io_util.o<br>
  LD      drivers/media/pci/built-in.o<br>
  LD      drivers/media/platform/omap/built-in.o<br>
  CC      drivers/mmc/core/bus.o<br>
  CC      drivers/media/platform/soc_camera/soc_camera.o<br>
  CC      drivers/mmc/core/host.o<br>
  CC      drivers/media/radio/radio-iris.o<br>
  CC      drivers/media/platform/soc_camera/soc_mediabus.o<br>
  LD      drivers/misc/carma/built-in.o<br>
  CC      drivers/md/dm-linear.o<br>
  LD      drivers/misc/cb710/built-in.o<br>
  CC      drivers/mfd/wcd9330-regmap.o<br>
  CC      drivers/iommu/msm_iommu_pagetable.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_smmu_api.o<br>
  LD      drivers/misc/eeprom/built-in.o<br>
  LD      drivers/misc/lis3lv02d/built-in.o<br>
  CC      drivers/iommu/arm-smmu.o<br>
  LD      drivers/misc/mic/built-in.o<br>
  CC      net/netfilter/nf_nat_amanda.o<br>
  CC      drivers/mmc/core/mmc.o<br>
  CC      drivers/media/radio/radio-iris-transport.o<br>
  CC      drivers/misc/qcom/qdsp6v2/aac_in.o<br>
  CC      drivers/media/platform/soc_camera/soc_camera_platform.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_hw_ops.o<br>
  CC      fs/coredump.o<br>
  CC      fs/drop_caches.o<br>
  CC      drivers/mmc/core/mmc_ops.o<br>
  LD      net/xfrm/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/qcelp_in.o<br>
  CC      net/netfilter/nf_nat_ftp.o<br>
  LD      drivers/media/platform/soc_camera/built-in.o<br>
  CC      net/netfilter/nf_nat_irc.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_core.o<br>
  CC      fs/dcookies.o<br>
  CC      drivers/mfd/wcd9335-regmap.o<br>
  CC      drivers/mmc/core/sd.o<br>
  CC      drivers/mfd/wcd9335-tables.o<br>
  CC      drivers/misc/qcom/qdsp6v2/evrc_in.o<br>
  CC      drivers/mfd/wcd-gpio-ctrl.o<br>
  LD      fs/built-in.o<br>
  CC      drivers/md/dm-stripe.o<br>
  LD      drivers/media/platform/msm/camera_v2/camera/built-in.o<br>
  CC      drivers/mmc/core/sd_ops.o<br>
  CC      drivers/mmc/host/sdhci.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_common.o<br>
  CC      drivers/mmc/host/sdhci-pltfm.o<br>
  CC      drivers/media/radio/silabs/radio-silabs.o<br>
  CC      drivers/misc/qcom/qdsp6v2/amrnb_in.o<br>
  CC      drivers/media/platform/msm/camera_v2/common/cam_soc_api.o<br>
  LD      drivers/iommu/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/g711mlaw_in.o<br>
  CC      drivers/mmc/host/sdhci-msm.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc.o<br>
  CC      drivers/net/dummy.o<br>
  CC      drivers/mmc/core/sdio.o<br>
  CC      drivers/mmc/card/queue.o<br>
  LD      drivers/mfd/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/g711alaw_in.o<br>
  LD      drivers/nfc/built-in.o<br>
  CC      net/netfilter/nf_nat_tftp.o<br>
  CC      drivers/of/base.o<br>
  CC      net/netfilter/x_tables.o<br>
  CC      drivers/pci/access.o<br>
  CC      drivers/of/device.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_utils.o<br>
  CC      drivers/md/dm-ioctl.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_wma.o<br>
  CC      drivers/md/dm-io.o<br>
  CC      drivers/pci/bus.o<br>
  CC      drivers/of/platform.o<br>
  CC      drivers/phy/phy-core.o<br>
  CC      drivers/of/fdt.o<br>
  CC      drivers/phy/phy-qcom-ufs.o<br>
  CC      drivers/net/mii.o<br>
  LD      drivers/media/platform/msm/camera_v2/common/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_dev.o<br>
  CC      drivers/of/fdt_address.o<br>
  CC      drivers/mmc/core/sdio_ops.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-20nm.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_buf_mgr.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_wmapro.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_aac.o<br>
  LD      drivers/mmc/card/mmc_block.o<br>
  CC      drivers/of/address.o<br>
  LD      drivers/mmc/card/built-in.o<br>
  CC      drivers/of/irq.o<br>
  CC      drivers/net/Space.o<br>
  CC      drivers/pci/probe.o<br>
  CC      drivers/of/of_net.o<br>
  CC      drivers/net/loopback.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_base.o<br>
  CC      drivers/md/dm-kcopyd.o<br>
  CC      drivers/of/of_mdio.o<br>
  CC      drivers/of/of_pci.o<br>
  CC      drivers/pci/host-bridge.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vdec.o<br>
  CC      drivers/pci/remove.o<br>
  CC      drivers/md/dm-sysfs.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_formats.o<br>
  CC      drivers/mmc/core/sdio_bus.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-14nm.o<br>
  LD      drivers/media/radio/silabs/built-in.o<br>
  LD      drivers/media/radio/built-in.o<br>
  CC      drivers/pci/pci.o<br>
  CC      drivers/phy/phy-qcom-ufs-qmp-v3.o<br>
  LD      drivers/net/ethernet/3com/built-in.o<br>
  LD      drivers/net/ethernet/8390/built-in.o<br>
  LD      drivers/net/ethernet/adaptec/built-in.o<br>
  LD      drivers/net/ethernet/agere/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_util.o<br>
  LD      drivers/net/ethernet/alteon/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_io_util.o<br>
  LD      drivers/net/ethernet/amd/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_multi_aac.o<br>
  LD      drivers/net/ethernet/arc/built-in.o<br>
  CC      net/netfilter/xt_tcpudp.o<br>
  LD      drivers/net/ethernet/atheros/built-in.o<br>
  LD      drivers/net/ethernet/broadcom/built-in.o<br>
  CC      drivers/pci/pci-driver.o<br>
  LD      drivers/net/ethernet/brocade/built-in.o<br>
  LD      drivers/net/ethernet/chelsio/built-in.o<br>
  LD      drivers/net/ethernet/cisco/built-in.o<br>
  LD      drivers/net/ethernet/dec/built-in.o<br>
  LD      drivers/net/ethernet/dlink/built-in.o<br>
  LD      drivers/net/ethernet/emulex/built-in.o<br>
  LD      drivers/net/ethernet/hp/built-in.o<br>
  LD      drivers/net/ethernet/i825xx/built-in.o<br>
  LD      drivers/net/ethernet/intel/built-in.o<br>
  LD      drivers/net/ethernet/marvell/built-in.o<br>
  LD      drivers/net/ethernet/mellanox/built-in.o<br>
  LD      drivers/net/ethernet/micrel/built-in.o<br>
  LD      drivers/net/ethernet/microchip/built-in.o<br>
  CC      drivers/net/ethernet/msm/rndis_ipa.o<br>
  CC      drivers/media/tuners/tuner-xc2028.o<br>
  LD      drivers/net/ethernet/myricom/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_alac.o<br>
  CC      drivers/media/tuners/tuner-simple.o<br>
  CC      drivers/media/rc/keymaps/rc-adstech-dvb-t-pci.o<br>
  CC      drivers/mmc/host/sdhci-msm-ice.o<br>
  CC      drivers/md/dm-stats.o<br>
  CC      net/netfilter/xt_mark.o<br>
  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_hw.o<br>
  CC      drivers/of/of_pci_irq.o<br>
  CC      drivers/mmc/core/sdio_cis.o<br>
  CC      drivers/phy/phy-qcom-ufs-qrbtc-v2.o<br>
  CC      drivers/mmc/core/sdio_io.o<br>
  CC      drivers/media/rc/keymaps/rc-alink-dtu-m.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_util.o<br>
  CC      drivers/mmc/core/sdio_irq.o<br>
  LD      drivers/phy/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-anysee.o<br>
  LD      drivers/media/platform/msm/camera_v2/fd/built-in.o<br>
  LD      drivers/misc/ti-st/built-in.o<br>
  CC      drivers/misc/qseecom.o<br>
  CC      drivers/mmc/core/quirks.o<br>
  CC      drivers/pci/search.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_smmu.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_wb.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_axi_util.o<br>
  CC      drivers/pci/pci-sysfs.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_stats_util.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp48.o<br>
  CC      drivers/of/of_spmi.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_ape.o<br>
  CC      drivers/mmc/core/slot-gpio.o<br>
  CC      drivers/pci/rom.o<br>
  CC      drivers/mmc/host/cmdq_hci.o<br>
  CC      drivers/of/of_reserved_mem.o<br>
  CC      net/netfilter/xt_connmark.o<br>
  CC      drivers/media/rc/keymaps/rc-apac-viewcomp.o<br>
  CC      drivers/pci/setup-res.o<br>
  CC      drivers/pci/irq.o<br>
  CC      drivers/mmc/core/debugfs.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp47.o<br>
  CC      drivers/pci/vpd.o<br>
  CC      drivers/of/of_slimbus.o<br>
  CC      drivers/of/of_coresight.o<br>
  CC      drivers/md/dm-builtin.o<br>
  LD      drivers/mmc/host/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/msm_venc.o<br>
  CC      drivers/media/platform/msm/vidc/msm_smem.o<br>
  LD      drivers/mmc/core/mmc_core.o<br>
  LD      drivers/mmc/core/built-in.o<br>
  LD      drivers/mmc/built-in.o<br>
  CC      drivers/media/rc/rc-main.o<br>
  CC      drivers/media/tuners/tuner-types.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_pipe.o<br>
  CC      drivers/md/dm-bufio.o<br>
  LD      drivers/net/ethernet/natsemi/built-in.o<br>
  LD      drivers/net/ethernet/neterion/built-in.o<br>
  LD      drivers/net/ethernet/nvidia/built-in.o<br>
  CC      drivers/of/of_batterydata.o<br>
  LD      drivers/net/ethernet/oki-semi/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_debug.o<br>
  CC      drivers/media/rc/keymaps/rc-asus-pc39.o<br>
  LD      drivers/net/ethernet/packetengines/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_utils_aio.o<br>
  CC      drivers/md/dm-crypt.o<br>
  CC      drivers/md/dm-verity-fec.o<br>
  CC      drivers/pci/setup-bus.o<br>
  CC      drivers/media/tuners/mt20xx.o<br>
  LD      drivers/net/ethernet/msm/built-in.o<br>
  LD      drivers/net/ethernet/qlogic/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_ctl.o<br>
  LD      drivers/net/ethernet/qualcomm/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-asus-ps3-100.o<br>
  CC      drivers/pci/vc.o<br>
  LD      drivers/net/ethernet/rdc/built-in.o<br>
  LD      drivers/net/ethernet/realtek/built-in.o<br>
  LD      drivers/of/built-in.o<br>
  CC      drivers/pci/proc.o<br>
  CC      drivers/media/rc/rc-ir-raw.o<br>
  LD      drivers/net/ethernet/samsung/built-in.o<br>
  LD      drivers/net/ethernet/seeq/built-in.o<br>
  LD      drivers/net/ethernet/silan/built-in.o<br>
  LD      drivers/net/ethernet/sis/built-in.o<br>
  LD      drivers/net/ethernet/smsc/built-in.o<br>
  LD      drivers/net/ethernet/stmicro/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2.o<br>
  CC      net/netfilter/xt_nat.o<br>
  LD      drivers/net/ethernet/sun/built-in.o<br>
  CC      drivers/media/tuners/tda8290.o<br>
  LD      drivers/net/ethernet/tehuti/built-in.o<br>
  LD      drivers/net/ethernet/ti/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-ati-tv-wonder-hd-600.o<br>
  LD      drivers/net/ethernet/via/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-ati-x10.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1.o<br>
  LD      drivers/net/ethernet/wiznet/built-in.o<br>
  LD      drivers/net/ethernet/built-in.o<br>
  CC      drivers/net/phy/phy.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp46.o<br>
  CC      net/netfilter/xt_CLASSIFY.o<br>
  CC      drivers/net/phy/phy_device.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp44.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-a16d.o<br>
  CC      drivers/net/phy/mdio_bus.o<br>
  CC      drivers/pci/slot.o<br>
  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2_aio.o<br>
  CC      drivers/pci/quirks.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_mp3.o<br>
  CC      drivers/media/rc/lirc_dev.o<br>
  CC      drivers/md/dm-verity-target.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3.o<br>
  CC      drivers/pci/msi.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrnb.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrwb.o<br>
  CC      net/netfilter/xt_CONNSECMARK.o<br>
  CC      drivers/media/tuners/tea5767.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_res_parse.o<br>
  CC      drivers/media/tuners/tea5761.o<br>
  CC      drivers/media/tuners/tda9887.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-cardbus.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_sync.o<br>
  CC      drivers/pci/setup-irq.o<br>
  CC      drivers/net/ppp/ppp_generic.o<br>
  CC      drivers/net/ppp/ppp_async.o<br>
  CC      net/netfilter/xt_CT.o<br>
  CC      drivers/md/dm-req-crypt.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_debug.o<br>
  CC      drivers/net/slip/slhc.o<br>
  CC      drivers/media/platform/msm/vidc/venus_hfi.o<br>
  CC      drivers/pci/pci-label.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-dvbt.o<br>
  LD      drivers/md/dm-mod.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-m135a.o<br>
  LD      drivers/net/phy/libphy.o<br>
  LD      drivers/md/dm-verity.o<br>
  LD      drivers/md/built-in.o<br>
  LD      drivers/net/phy/built-in.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_debug.o<br>
  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3_debug.o<br>
  CC      drivers/net/usb/asix_devices.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_amrwbplus.o<br>
  CC      drivers/pci/syscall.o<br>
  CC      drivers/net/wireless/ath/wil6210/main.o<br>
  CC      drivers/media/platform/msm/vidc/hfi_response_handler.o<br>
  LD      drivers/net/slip/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_evrc.o<br>
  CC      drivers/net/wireless/ath/wil6210/netdev.o<br>
  CC      drivers/net/wireless/ath/wil6210/cfg80211.o<br>
  CC      drivers/pci/of.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_qcelp.o<br>
  CC      drivers/media/tuners/tda827x.o<br>
  CC      drivers/net/wireless/ath/wil6210/pcie_bus.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp40.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-m733a-rm-k6.o<br>
  CC      drivers/pci/host/pci-msm.o<br>
  CC      drivers/misc/qcom/qdsp6v2/amrwb_in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/audio_hwacc_effects.o<br>
  LD      drivers/media/platform/msm/sde/rotator/built-in.o<br>
  LD      drivers/media/platform/msm/sde/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-avermedia-rm-ks.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usf.o<br>
  CC      drivers/media/rc/keymaps/rc-avertv-303.o<br>
  CC      net/netfilter/xt_LOG.o<br>
  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp.o<br>
  CC      drivers/pinctrl/core.o<br>
  LD      drivers/pci/host/built-in.o<br>
  LD      drivers/pci/built-in.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usfcdev.o<br>
  CC      drivers/net/tun.o<br>
  CC      drivers/pinctrl/pinctrl-utils.o<br>
  LD      drivers/media/platform/msm/camera_v2/isp/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/ispif/msm_ispif.o<br>
  CC      net/netfilter/xt_NETMAP.o<br>
  CC      drivers/net/usb/asix_common.o<br>
  CC      net/netfilter/xt_NFLOG.o<br>
  CC      drivers/media/rc/keymaps/rc-azurewave-ad-tu700.o<br>
  CC      drivers/net/veth.o<br>
  CC      drivers/media/rc/keymaps/rc-behold.o<br>
  CC      drivers/media/tuners/tda18271-maps.o<br>
  CC      drivers/misc/qcom/qdsp6v2/ultrasound/q6usm.o<br>
  LD      drivers/media/platform/msm/camera_v2/ispif/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm_buf_mgr/msm_generic_buf_mgr.o<br>
  CC      drivers/media/rc/keymaps/rc-behold-columbus.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_dev.o<br>
  CC      drivers/net/usb/ax88172a.o<br>
  CC      drivers/media/tuners/tda18271-common.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm_vb2/msm_vb2.o<br>
  CC      drivers/media/rc/keymaps/rc-budget-ci-old.o<br>
  CC      drivers/pinctrl/pinmux.o<br>
  CC      drivers/media/tuners/tda18271-fe.o<br>
  CC      drivers/media/tuners/xc5000.o<br>
  LD      drivers/media/platform/msm/camera_v2/msm_buf_mgr/built-in.o<br>
  CC      drivers/net/ppp/bsd_comp.o<br>
  CC      drivers/net/ppp/ppp_deflate.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/odu_bridge.o<br>
  CC      drivers/media/platform/msm/vidc/hfi_packetization.o<br>
  CC      drivers/pinctrl/pinconf.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_sync.o<br>
  LD      drivers/media/platform/msm/camera_v2/msm_vb2/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_core.o<br>
  CC      drivers/media/rc/keymaps/rc-cinergy-1400.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_hw.o<br>
  CC      drivers/net/wireless/ath/wil6210/debugfs.o<br>
  CC      drivers/media/tuners/xc4000.o<br>
  CC      drivers/net/wireless/ath/wil6210/wmi.o<br>
  CC      drivers/media/tuners/mc44s803.o<br>
  CC      drivers/pinctrl/devicetree.o<br>
  CC      drivers/pinctrl/pinconf-generic.o<br>
  LD      drivers/pinctrl/freescale/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/ipa_mhi_client.o<br>
  CC      drivers/media/rc/keymaps/rc-cinergy.o<br>
  CC      drivers/media/rc/keymaps/rc-delock-61959.o<br>
  CC      net/netfilter/xt_NFQUEUE.o<br>
  CC      drivers/media/rc/keymaps/rc-dib0700-nec.o<br>
  CC      net/netfilter/xt_REDIRECT.o<br>
  CC      net/netfilter/xt_SECMARK.o<br>
  CC      drivers/media/rc/ir-nec-decoder.o<br>
  CC      drivers/power/power_supply_core.o<br>
  CC      drivers/media/rc/ir-rc5-decoder.o<br>
  CC      drivers/platform/msm/ipa/ipa_clients/ipa_uc_offload.o<br>
  CC      drivers/power/power_supply_sysfs.o<br>
  CC      drivers/media/platform/msm/vidc/vidc_hfi.o<br>
  CC      drivers/net/ppp/ppp_mppe.o<br>
  CC      drivers/net/ppp/ppp_synctty.o<br>
  LD      drivers/platform/msm/ipa/ipa_clients/built-in.o<br>
  CC      drivers/net/usb/ax88179_178a.o<br>
  LD      drivers/misc/qcom/qdsp6v2/ultrasound/built-in.o<br>
  CC      drivers/pinctrl/qcom/pinctrl-msm.o<br>
  LD      drivers/misc/qcom/qdsp6v2/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa.o<br>
  CC      drivers/media/rc/keymaps/rc-dib0700-rc5.o<br>
  LD      drivers/misc/qcom/built-in.o<br>
  LD      drivers/media/tuners/tda18271.o<br>
  CC      drivers/misc/hdcp.o<br>
  LD      drivers/media/tuners/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_debugfs.o<br>
  CC      drivers/net/usb/cdc_ether.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_hdr.o<br>
  CC      drivers/misc/compat_qseecom.o<br>
  CC      net/netfilter/xt_TPROXY.o<br>
  CC      drivers/power/power_supply_leds.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_flt.o<br>
  CC      net/netfilter/xt_TCPMSS.o<br>
  CC      drivers/net/wireless/cnss_crypto/cnss_secif.o<br>
  CC      drivers/media/rc/keymaps/rc-digitalnow-tinytwin.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_rt.o<br>
  CC      drivers/misc/uid_cputime.o<br>
  CC      drivers/net/ppp/pppox.o<br>
  CC      drivers/media/rc/keymaps/rc-digittrade.o<br>
  CC      drivers/net/usb/net1080.o<br>
  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_platform.o<br>
  CC      drivers/media/platform/msm/vidc/venus_boot.o<br>
  CC      drivers/misc/type-c-pericom.o<br>
  CC      drivers/media/rc/keymaps/rc-dm1105-nec.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dp.o<br>
  CC      drivers/power/smb1351-charger.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_client.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_utils.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_nat.o<br>
  CC      drivers/net/ppp/pppoe.o<br>
  CC      drivers/pinctrl/qcom/pinctrl-msm8953.o<br>
  CC      drivers/net/wireless/ath/wil6210/interrupt.o<br>
  CC      drivers/net/wireless/ath/wil6210/txrx.o<br>
  LD      drivers/net/wireless/cnss_crypto/built-in.o<br>
  LD      drivers/misc/built-in.o<br>
  CC      drivers/platform/msm/msm_11ad/msm_11ad.o<br>
  CC      drivers/net/usb/cdc_subset.o<br>
  CC      drivers/power/smb135x-charger.o<br>
  CC      drivers/pwm/core.o<br>
  CC      drivers/pwm/sysfs.o<br>
  CC      drivers/power/qpnp-fg.o<br>
  LD      drivers/media/usb/b2c2/built-in.o<br>
  LD      drivers/platform/msm/msm_11ad/msm_11ad_proxy.o<br>
  LD      drivers/media/usb/dvb-usb/built-in.o<br>
  LD      drivers/platform/msm/msm_11ad/built-in.o<br>
  LD      drivers/media/usb/dvb-usb-v2/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-dntv-live-dvb-t.o<br>
  LD      drivers/media/usb/s2255/built-in.o<br>
  CC      drivers/net/usb/zaurus.o<br>
  LD      drivers/media/usb/siano/built-in.o<br>
  CC      drivers/media/v4l2-core/v4l2-dev.o<br>
  LD      drivers/media/usb/stkwebcam/built-in.o<br>
  LD      drivers/media/usb/ttusb-budget/built-in.o<br>
  LD      drivers/media/usb/ttusb-dec/built-in.o<br>
  LD      drivers/media/usb/zr364xx/built-in.o<br>
  LD      drivers/media/usb/built-in.o<br>
  CC      drivers/media/rc/ir-rc6-decoder.o<br>
  LD      drivers/pinctrl/qcom/built-in.o<br>
  LD      drivers/pinctrl/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-dntv-live-dvbt-pro.o<br>
  CC      net/netfilter/xt_TEE.o<br>
  CC      net/netfilter/xt_TRACE.o<br>
  CC      drivers/media/v4l2-core/v4l2-ioctl.o<br>
  CC      net/netfilter/xt_IDLETIMER.o<br>
  LD      drivers/media/platform/msm/camera_v2/jpeg_10/built-in.o<br>
  CC      drivers/net/ppp/pppolac.o<br>
  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp_soc.o<br>
  CC      drivers/media/platform/msm/vidc/msm_vidc_dcvs.o<br>
  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp.o<br>
  CC      net/netfilter/xt_HARDIDLETIMER.o<br>
  CC      drivers/media/v4l2-core/v4l2-device.o<br>
  CC      drivers/pwm/pwm-qpnp.o<br>
  CC      net/netfilter/xt_comment.o<br>
  CC      drivers/media/rc/keymaps/rc-dvbsky.o<br>
  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_dyn_gov.o<br>
  CC      drivers/media/platform/msm/vidc/vmem/vmem.o<br>
  CC      drivers/net/usb/usbnet.o<br>
  CC      drivers/media/platform/msm/vidc/vmem/vmem_debugfs.o<br>
  CC      drivers/net/ppp/pppopns.o<br>
  CC      drivers/power/qpnp-smbcharger.o<br>
  CC      drivers/net/usb/cdc_ncm.o<br>
  CC      drivers/power/pmic-voter.o<br>
  CC      drivers/media/rc/keymaps/rc-em-terratec.o<br>
  CC      drivers/media/v4l2-core/v4l2-fh.o<br>
  CC      drivers/media/v4l2-core/v4l2-event.o<br>
  CC      net/netfilter/xt_connlimit.o<br>
  CC      net/netfilter/xt_conntrack.o<br>
  LD      drivers/media/platform/msm/vidc/vmem/built-in.o<br>
  CC      net/netfilter/xt_dscp.o<br>
  CC      drivers/net/wireless/ath/wil6210/debug.o<br>
  LD      drivers/pwm/built-in.o<br>
  CC      drivers/ras/ras.o<br>
  CC      drivers/power/qpnp-typec.o<br>
  CC      net/netfilter/xt_ecn.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv2.o<br>
  CC      drivers/ras/debugfs.o<br>
  LD      drivers/net/ppp/built-in.o<br>
  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_table_gov.o<br>
  CC      drivers/net/wireless/ath/wil6210/rx_reorder.o<br>
  CC      drivers/net/wireless/ath/wil6210/ioctl.o<br>
  CC      net/netfilter/xt_esp.o<br>
  CC      drivers/media/v4l2-core/v4l2-ctrls.o<br>
  CC      drivers/media/v4l2-core/v4l2-subdev.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv.o<br>
  CC      drivers/media/v4l2-core/v4l2-clk.o<br>
  CC      drivers/net/wireless/ath/wil6210/fw.o<br>
  LD      drivers/ras/built-in.o<br>
  CC      net/netfilter/xt_hashlimit.o<br>
  CC      drivers/media/rc/keymaps/rc-encore-enltv-fm53.o<br>
  CC      net/netfilter/xt_helper.o<br>
  CC      net/netfilter/xt_hl.o<br>
  CC      drivers/regulator/core.o<br>
  CC      drivers/rtc/rtc-lib.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_intf.o<br>
  LD      drivers/net/usb/asix.o<br>
  LD      drivers/net/usb/built-in.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_core.o<br>
  CC      drivers/media/rc/keymaps/rc-evga-indtube.o<br>
  CC      drivers/rtc/hctosys.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/teth_bridge.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_client_api.o<br>
  LD      drivers/media/platform/msm/vidc/governors/built-in.o<br>
  LD      drivers/media/platform/msm/vidc/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_interrupts.o<br>
  CC      net/netfilter/xt_iprange.o<br>
  CC      drivers/rtc/systohc.o<br>
  CC      drivers/rtc/class.o<br>
  LD      drivers/media/platform/msm/camera_v2/pproc/cpp/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/pproc/built-in.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc.o<br>
  CC      drivers/media/rc/keymaps/rc-eztv.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/actuator/msm_actuator.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_of.o<br>
  CC      drivers/power/battery_current_limit.o<br>
  CC      net/netfilter/xt_l2tp.o<br>
  CC      net/netfilter/xt_length.o<br>
  CC      drivers/media/rc/keymaps/rc-flydvb.o<br>
  CC      drivers/net/wireless/ath/wil6210/pm.o<br>
  CC      drivers/media/rc/keymaps/rc-flyvideo.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_rpm_smd.o<br>
  CC      net/netfilter/xt_limit.o<br>
  CC      drivers/net/wireless/ath/wil6210/pmc.o<br>
  CC      drivers/power/msm_bcl.o<br>
  CC      drivers/media/rc/keymaps/rc-fusionhdtv-mce.o<br>
  CC      drivers/regulator/dummy.o<br>
  CC      drivers/net/wireless/ath/wil6210/trace.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_fabric_adhoc.o<br>
  CC      net/netfilter/xt_mac.o<br>
  CC      drivers/rtc/interface.o<br>
  CC      net/netfilter/xt_multiport.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_wdi.o<br>
  CC      drivers/power/bcl_peripheral.o<br>
  CC      drivers/platform/msm/ipa/ipa_api.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_arb_adhoc.o<br>
  CC      net/netfilter/xt_pkttype.o<br>
  CC      drivers/media/platform/msm/camera_v2/msm.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_dependency_graph.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_rules.o<br>
  CC      drivers/media/rc/keymaps/rc-gadmei-rm008z.o<br>
  CC      drivers/media/v4l2-core/v4l2-async.o<br>
  CC      drivers/net/wireless/cnss_prealloc/cnss_prealloc.o<br>
  CC      drivers/media/rc/keymaps/rc-genius-tvgo-a11mce.o<br>
  CC      drivers/media/rc/keymaps/rc-gotview7135.o<br>
  CC      net/netfilter/xt_policy.o<br>
  CC      drivers/media/v4l2-core/v4l2-compat-ioctl32.o<br>
  CC      net/netfilter/xt_qtaguid_print.o<br>
  CC      drivers/rtc/rtc-dev.o<br>
  CC      net/netfilter/xt_qtaguid.o<br>
  CC      drivers/regulator/fixed-helper.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/actuator/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/cci/msm_cci.o<br>
  CC      drivers/media/v4l2-core/v4l2-of.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/csid/msm_csid.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_peers_list.o<br>
  CC      drivers/net/wireless/ath/wil6210/wil_platform.o<br>
  CC      drivers/media/rc/keymaps/rc-imon-mce.o<br>
  CC      drivers/media/v4l2-core/v4l2-common.o<br>
  CC      drivers/media/rc/keymaps/rc-imon-pad.o<br>
  LD      drivers/net/wireless/cnss_prealloc/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-iodata-bctv7e.o<br>
  CC      drivers/regulator/helpers.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dma.o<br>
  CC      drivers/net/wireless/wcnss/wcnss_wlan.o<br>
  CC      drivers/media/v4l2-core/v4l2-dv-timings.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_bimc_adhoc.o<br>
  CC      drivers/net/wireless/wcnss/wcnss_vreg.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/csiphy/msm_csiphy.o<br>
  CC      drivers/rtc/rtc-proc.o<br>
  CC      drivers/net/wireless/ath/wil6210/ethtool.o<br>
  CC      drivers/media/v4l2-core/v4l2-mem2mem.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_mhi.o<br>
  CC      net/netfilter/xt_quota.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_mhi.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_noc_adhoc.o<br>
  CC      drivers/power/qcom/msm-pm.o<br>
  CC      drivers/power/qcom/pm-data.o<br>
  CC      drivers/media/media-device.o<br>
  CC      drivers/media/rc/keymaps/rc-it913x-v1.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/csiphy/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-it913x-v2.o<br>
  CC      drivers/media/v4l2-core/videobuf-core.o<br>
  CC      drivers/regulator/devres.o<br>
  CC      drivers/net/wireless/ath/wil6210/wil_crash_dump.o<br>
  CC      net/netfilter/xt_quota2.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_ntn.o<br>
  CC      drivers/scsi/scsi.o<br>
  CC      drivers/regulator/of_regulator.o<br>
  CC      drivers/rtc/rtc-sysfs.o<br>
  CC      drivers/rtc/qpnp-rtc.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/csid/built-in.o<br>
  CC      net/netfilter/xt_socket.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/eeprom/msm_eeprom.o<br>
  CC      drivers/regulator/fixed.o<br>
  CC      drivers/regulator/mem-acc-regulator.o<br>
  CC      drivers/media/rc/keymaps/rc-kaiomy.o<br>
  CC      drivers/scsi/hosts.o<br>
  LD      drivers/rtc/rtc-core.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/flash/msm_flash.o<br>
  LD      drivers/rtc/built-in.o<br>
  CC      drivers/media/rc/ir-jvc-decoder.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_of_adhoc.o<br>
  CC      drivers/platform/msm/msm_bus/msm_buspm_coresight_adhoc.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/cci/built-in.o<br>
  CC      drivers/regulator/fan53555.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_cci_i2c.o<br>
  CC      drivers/scsi/scsi_ioctl.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/flash/built-in.o<br>
  CC      drivers/scsi/constants.o<br>
  CC      net/netfilter/xt_state.o<br>
  CC      drivers/scsi/scsicam.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa.o<br>
  CC      drivers/scsi/scsi_error.o<br>
  CC      drivers/power/qcom/lpm-stats.o<br>
  CC      drivers/net/wireless/ath/wil6210/p2p.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-315u.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-pc150u.o<br>
  CC      drivers/regulator/msm_gfx_ldo.o<br>
  CC      drivers/power/qcom/pm-boot.o<br>
  CC      drivers/power/qcom/msm-core.o<br>
  CC      drivers/media/v4l2-core/videobuf2-core.o<br>
  CC      drivers/media/v4l2-core/videobuf2-memops.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service_v01.o<br>
  CC      drivers/media/rc/keymaps/rc-kworld-plus-tv-analog.o<br>
  LD      drivers/net/wireless/ath/wil6210/wil6210.o<br>
  LD      drivers/net/wireless/ath/wil6210/built-in.o<br>
  LD      drivers/net/wireless/ath/built-in.o<br>
  CC      net/netfilter/xt_statistic.o<br>
  CC      drivers/scsi/scsi_lib.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service.o<br>
  CC      drivers/platform/msm/msm_bus/msm_bus_dbg.o<br>
  CC      drivers/media/rc/keymaps/rc-leadtek-y04g0051.o<br>
  LD      drivers/net/wireless/wcnss/wcnsscore.o<br>
  LD      drivers/net/wireless/wcnss/built-in.o<br>
  LD      drivers/net/wireless/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-lirc.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_qup_i2c.o<br>
  CC      drivers/sensors/sensors_ssc.o<br>
  CC      drivers/media/rc/keymaps/rc-lme2510.o<br>
  LD      drivers/net/built-in.o<br>
  CC      drivers/slimbus/slimbus.o<br>
  CC      drivers/power/qcom/debug_core.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/eeprom/built-in.o<br>
  CC      drivers/power/qcom/apm.o<br>
  CC      drivers/media/rc/ir-sony-decoder.o<br>
  CC      drivers/media/rc/ir-sanyo-decoder.o<br>
  LD      drivers/sensors/built-in.o<br>
  CC      drivers/media/media-devnode.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_spi.o<br>
  AS      drivers/soc/qcom/idle-v8.o<br>
  CC      drivers/slimbus/slim-msm.o<br>
  CC      drivers/soc/qcom/cpu_ops.o<br>
  CC      drivers/slimbus/slim-msm-ngd.o<br>
  CC      drivers/scsi/scsi_lib_dma.o<br>
  CC      drivers/media/media-entity.o<br>
  CC      drivers/media/rc/keymaps/rc-manli.o<br>
  CC      drivers/soc/qcom/msm_rq_stats.o<br>
  CC      drivers/regulator/rpm-smd-regulator.o<br>
  CC      drivers/regulator/qpnp-regulator.o<br>
  CC      net/netfilter/xt_string.o<br>
  CC      drivers/media/rc/ir-sharp-decoder.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_resource.o<br>
  CC      drivers/regulator/spm-regulator.o<br>
  LD      drivers/media/media.o<br>
  CC      drivers/platform/msm/ipa/ipa_rm_inactivity_timer.o<br>
  CC      drivers/scsi/scsi_scan.o<br>
  CC      drivers/scsi/scsi_sysfs.o<br>
  CC      drivers/scsi/scsi_devinfo.o<br>
  LD      drivers/power/qcom/built-in.o<br>
  CC      drivers/power/reset/msm-poweroff.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10.o<br>
  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa_fd_ioctl.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10-digitainer.o<br>
  LD      drivers/power/reset/built-in.o<br>
  CC      drivers/scsi/scsi_sysctl.o<br>
  LD      drivers/power/power_supply.o<br>
  LD      drivers/power/built-in.o<br>
  CC      drivers/soundwire/soundwire.o<br>
  CC      drivers/media/rc/ir-lirc-codec.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_dt_util.o<br>
  CC      drivers/soc/qcom/cpuss_dump.o<br>
  LD      drivers/platform/msm/ipa/ipa_v2/ipat.o<br>
  LD      drivers/platform/msm/ipa/ipa_v2/built-in.o<br>
  CC      drivers/soundwire/swr-wcd-ctrl.o<br>
  CC      drivers/scsi/scsi_proc.o<br>
  CC      drivers/media/v4l2-core/videobuf2-vmalloc.o<br>
  CC      net/netfilter/xt_time.o<br>
  CC      net/netfilter/xt_u32.o<br>
  LD      drivers/platform/msm/msm_bus/built-in.o<br>
  CC      drivers/regulator/cpr-regulator.o<br>
  CC      drivers/regulator/cpr3-regulator.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/io/built-in.o<br>
  LD      net/netfilter/netfilter.o<br>
  LD      drivers/platform/msm/ipa/ipa_common<br>
  LD      drivers/platform/msm/ipa/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ir_cut/msm_ir_cut.o<br>
  CC      drivers/spi/spi.o<br>
  CC      drivers/media/rc/keymaps/rc-medion-x10-or2x.o<br>
  LD      drivers/slimbus/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-digivox-ii.o<br>
  CC      drivers/platform/msm/spmi/spmi.o<br>
  LD      drivers/media/v4l2-core/videodev.o<br>
  LD      drivers/media/v4l2-core/built-in.o<br>
  LD      net/netfilter/nfnetlink_queue.o<br>
  CC      drivers/scsi/scsi_trace.o<br>
  CC      drivers/spi/spidev.o<br>
  CC      drivers/scsi/scsi_pm.o<br>
  CC      drivers/soc/qcom/memory_dump_v2.o<br>
  CC      drivers/platform/msm/spmi/spmi-resources.o<br>
  CC      drivers/spi/spi-qup.o<br>
  CC      drivers/media/rc/ir-xmp-decoder.o<br>
  CC      drivers/regulator/cpr3-util.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-digivox-iii.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere.o<br>
  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere-plus.o<br>
  LD      net/netfilter/nf_conntrack.o<br>
  LD      net/netfilter/nf_conntrack_h323.o<br>
  CC      drivers/scsi/sd.o<br>
  LD      net/netfilter/nf_nat.o<br>
  CC      drivers/scsi/sg.o<br>
  CC      drivers/scsi/ufs/ufs-qcom.o<br>
  CC      drivers/scsi/ch.o<br>
  LD      net/netfilter/built-in.o<br>
  CC      drivers/platform/msm/spmi/spmi-pmic-arb.o<br>
  CC      drivers/platform/msm/sps/bam.o<br>
  CC      drivers/soc/qcom/ddr-health.o<br>
  LD      net/built-in.o<br>
  CC      drivers/platform/msm/spmi/qpnp-int.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ir_cut/built-in.o<br>
  CC      drivers/platform/msm/spmi/spmi-dbgfs.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ir_led/msm_ir_led.o<br>
  CC      drivers/platform/msm/sps/sps_bam.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/ois/msm_ois.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_init.o<br>
  CC      drivers/spi/spi_qsd.o<br>
  CC      drivers/platform/msm/sps/sps.o<br>
  CC      drivers/media/rc/keymaps/rc-nebula.o<br>
  LD      drivers/soundwire/built-in.o<br>
  CC      drivers/staging/staging.o<br>
  CC      drivers/switch/switch_class.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ir_led/built-in.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_driver.o<br>
  CC      drivers/soc/qcom/watchdog_v2.o<br>
  CC      drivers/media/rc/keymaps/rc-nec-terratec-cinergy-xs.o<br>
  CC      drivers/soc/qcom/common_log.o<br>
  CC      drivers/soc/qcom/cpu_pwr_ctl.o<br>
  CC      drivers/staging/android/ion/ion.o<br>
  LD      drivers/staging/media/built-in.o<br>
  CC      drivers/staging/android/ion/ion_heap.o<br>
  LD      drivers/platform/msm/spmi/built-in.o<br>
  LD      drivers/switch/built-in.o<br>
  CC      drivers/platform/msm/qpnp-power-on.o<br>
  CC      drivers/thermal/thermal_core.o<br>
  CC      drivers/platform/msm/qpnp-revid.o<br>
  CC      drivers/media/rc/keymaps/rc-norwood.o<br>
  CC      drivers/thermal/thermal_hwmon.o<br>
  CC      drivers/staging/android/ion/ion_page_pool.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/ois/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-npgtech.o<br>
  CC      drivers/soc/qcom/socinfo.o<br>
  CC      drivers/staging/android/ion/ion_system_heap.o<br>
  CC      drivers/tty/tty_io.o<br>
  CC      drivers/thermal/of-thermal.o<br>
  CC      drivers/uio/uio.o<br>
  CC      drivers/media/rc/keymaps/rc-pctv-sedna.o<br>
  CC      drivers/usb/class/cdc-acm.o<br>
  CC      drivers/scsi/ufs/ufs-qcom-ice.o<br>
  CC      drivers/tty/n_tty.o<br>
  CC      drivers/thermal/step_wise.o<br>
  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor.o<br>
  CC      drivers/uio/msm_sharedmem/msm_sharedmem.o<br>
  CC      drivers/usb/common/common.o<br>
  LD      drivers/usb/class/built-in.o<br>
  CC      drivers/platform/msm/sps/sps_dma.o<br>
  LD      drivers/usb/common/usb-common.o<br>
  LD      drivers/usb/common/built-in.o<br>
  LD      drivers/video/backlight/built-in.o<br>
  CC      drivers/usb/core/usb.o<br>
  CC      drivers/video/fbdev/core/fb_notify.o<br>
  CC      drivers/video/fbdev/core/fb_cmdline.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-color.o<br>
  CC      drivers/platform/msm/sps/sps_map.o<br>
  LD      drivers/thermal/samsung/built-in.o<br>
  CC      drivers/thermal/msm-tsens.o<br>
  CC      drivers/video/fbdev/core/fbmem.o<br>
  CC      drivers/usb/core/hub.o<br>
  CC      drivers/soc/qcom/boot_stats.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-grey.o<br>
  LD      drivers/spi/built-in.o<br>
  CC      drivers/uio/msm_sharedmem/remote_filesystem_access_v01.o<br>
  CC      drivers/media/rc/keymaps/rc-pinnacle-pctv-hd.o<br>
  CC      drivers/regulator/cpr3-hmss-regulator.o<br>
  CC      drivers/video/fbdev/core/fbmon.o<br>
  CC      drivers/thermal/qpnp-temp-alarm.o<br>
  CC      drivers/uio/msm_sharedmem/sharedmem_qmi.o<br>
  CC      drivers/staging/android/ion/ion_carveout_heap.o<br>
  CC      drivers/scsi/ufs/ufshcd.o<br>
  CC      drivers/usb/core/hcd.o<br>
  CC      drivers/scsi/ufs/ufs_quirks.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview.o<br>
  CC      drivers/regulator/cpr3-mmss-regulator.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-mk12.o<br>
  CC      drivers/tty/tty_ioctl.o<br>
  CC      drivers/thermal/qpnp-adc-tm.o<br>
  LD      drivers/uio/msm_sharedmem/built-in.o<br>
  LD      drivers/uio/built-in.o<br>
  CC      drivers/thermal/msm_thermal.o<br>
  CC      drivers/staging/android/ion/ion_chunk_heap.o<br>
  CC      drivers/thermal/msm_thermal-dev.o<br>
  CC      drivers/staging/android/ion/ion_system_secure_heap.o<br>
  CC      drivers/soc/qcom/rpm-smd.o<br>
  CC      drivers/soc/qcom/event_timer.o<br>
  CC      drivers/usb/dwc3/core.o<br>
  CC      drivers/soc/qcom/rpm-smd-debug.o<br>
  CC      drivers/usb/dwc3/debug.o<br>
  LD      drivers/media/platform/msm/camera_v2/sensor/built-in.o<br>
  LD      drivers/media/platform/msm/camera_v2/built-in.o<br>
  CC      drivers/thermal/lmh_interface.o<br>
  LD      drivers/media/platform/msm/built-in.o<br>
  CC      drivers/video/fbdev/core/fbcmap.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-002t.o<br>
  CC      drivers/usb/dwc3/trace.o<br>
  CC      drivers/tty/tty_ldisc.o<br>
  CC      drivers/tty/tty_buffer.o<br>
  LD      drivers/media/platform/built-in.o<br>
  CC      drivers/platform/msm/qpnp-coincell.o<br>
  CC      drivers/thermal/lmh_lite.o<br>
  CC      drivers/video/fbdev/core/fbsysfs.o<br>
  CC      drivers/platform/msm/qpnp-haptic.o<br>
  CC      drivers/usb/gadget/usbstring.o<br>
  CC      drivers/soc/qcom/spm.o<br>
  CC      drivers/soc/qcom/spm_devices.o<br>
  CC      drivers/staging/android/ion/ion_cma_heap.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_ctl.o<br>
  LD      drivers/thermal/thermal_sys.o<br>
  CC      drivers/regulator/cpr4-apss-regulator.o<br>
  LD      drivers/thermal/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba.o<br>
  CC      drivers/media/rc/keymaps/rc-pixelview-new.o<br>
  CC      drivers/soc/qcom/scm.o<br>
  CC      drivers/staging/android/ion/ion_cma_secure_heap.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_init.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_helpers.o<br>
  CC      drivers/usb/gadget/config.o<br>
  CC      drivers/media/rc/keymaps/rc-powercolor-real-angel.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pipe.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_util.o<br>
  CC      drivers/usb/dwc3/host.o<br>
  CC      drivers/usb/gadget/epautoconf.o<br>
  CC      drivers/usb/core/urb.o<br>
  CC      drivers/tty/tty_port.o<br>
  CC      drivers/media/rc/keymaps/rc-proteus-2309.o<br>
  CC      drivers/video/fbdev/core/modedb.o<br>
  CC      drivers/video/fbdev/core/fbcvt.o<br>
  CC      drivers/usb/dwc3/gadget.o<br>
  CC      drivers/tty/tty_mutex.o<br>
  CC      drivers/media/rc/keymaps/rc-purpletv.o<br>
  CC      drivers/usb/core/message.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_debug.o<br>
  CC      drivers/usb/core/driver.o<br>
  CC      drivers/usb/gadget/composite.o<br>
  CC      drivers/soc/qcom/scm-boot.o<br>
  CC      drivers/soc/qcom/mpm-of.o<br>
  CC      drivers/soc/qcom/smem.o<br>
  CC      drivers/regulator/cprh-kbss-regulator.o<br>
  CC      drivers/media/rc/keymaps/rc-pv951.o<br>
  CC      drivers/tty/tty_ldsem.o<br>
  CC      drivers/video/fbdev/core/cfbfillrect.o<br>
  CC      drivers/staging/android/ion/compat_ion.o<br>
  CC      drivers/regulator/qpnp-labibb-regulator.o<br>
  CC      drivers/video/fbdev/core/cfbcopyarea.o<br>
  CC      drivers/video/fbdev/msm/../../msm/msm_dba/adv7533.o<br>
  CC      drivers/staging/android/ion/msm/msm_ion.o<br>
  CC      drivers/video/fbdev/core/cfbimgblt.o<br>
  CC      drivers/regulator/stub-regulator.o<br>
  CC      drivers/media/rc/keymaps/rc-hauppauge.o<br>
  LD      drivers/video/fbdev/core/fb.o<br>
  LD      drivers/video/fbdev/core/built-in.o<br>
  CC      drivers/staging/android/ion/msm/compat_msm_ion.o<br>
  CC      drivers/tty/pty.o<br>
  CC      drivers/tty/tty_audit.o<br>
  LD      drivers/video/fbdev/omap2/displays-new/built-in.o<br>
  LD      drivers/video/fbdev/omap2/dss/built-in.o<br>
  LD      drivers/video/fbdev/omap2/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-rc6-mce.o<br>
  CC      drivers/staging/android/binder.o<br>
  CC      drivers/media/rc/keymaps/rc-real-audio-220-32-keys.o<br>
  CC      drivers/soc/qcom/smem_debug.o<br>
  CC      drivers/usb/core/config.o<br>
  CC      drivers/usb/core/file.o<br>
  CC      drivers/soc/qcom/smd.o<br>
  CC      drivers/regulator/kryo-regulator.o<br>
  CC      drivers/usb/host/pci-quirks.o<br>
  CC      drivers/media/rc/pwm-ir.o<br>
  CC      drivers/usb/core/buffer.o<br>
  CC      drivers/usb/dwc3/ep0.o<br>
  CC      drivers/tty/sysrq.o<br>
  LD      drivers/staging/android/ion/msm/built-in.o<br>
  LD      drivers/staging/android/ion/built-in.o<br>
  CC      drivers/usb/dwc3/debugfs.o<br>
  LD      drivers/regulator/built-in.o<br>
  CC      drivers/usb/host/xhci-pci.o<br>
  CC      drivers/usb/host/xhci-plat.o<br>
  LD      drivers/video/fbdev/msm/../../msm/msm_dba/built-in.o<br>
  CC      drivers/usb/host/ehci-hcd.o<br>
  CC      drivers/media/rc/keymaps/rc-reddo.o<br>
  CC      drivers/media/rc/keymaps/rc-snapstream-firefly.o<br>
  CC      drivers/usb/core/sysfs.o<br>
  CC      drivers/usb/gadget/functions.o<br>
  CC      drivers/usb/core/endpoint.o<br>
  LD      drivers/tty/ipwireless/built-in.o<br>
  CC      drivers/tty/serial/serial_core.o<br>
  LD      drivers/tty/vt/built-in.o<br>
  CC      drivers/usb/dwc3/dwc3-pci.o<br>
  CC      drivers/media/rc/keymaps/rc-streamzap.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/dsi_status_6g.o<br>
  CC      drivers/media/rc/keymaps/rc-tbs-nec.o<br>
  CC      drivers/tty/serial/msm_serial_hs.o<br>
  CC      drivers/usb/core/devio.o<br>
  CC      drivers/usb/dwc3/dwc3-msm.o<br>
  CC      drivers/usb/host/ehci-pci.o<br>
  CC      drivers/tty/serial/msm_smd_tty.o<br>
  CC      drivers/usb/core/notify.o<br>
  CC      drivers/usb/gadget/u_f.o<br>
  CC      drivers/media/rc/keymaps/rc-technisat-usb2.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-cinergy-xs.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-slim.o<br>
  LD      drivers/tty/serial/built-in.o<br>
  CC      drivers/usb/dwc3/dbm.o<br>
  LD      drivers/tty/built-in.o<br>
  CC      drivers/usb/host/ehci-msm.o<br>
  CC      drivers/usb/core/generic.o<br>
  CC      drivers/platform/msm/sps/sps_mem.o<br>
  CC      drivers/platform/msm/sps/sps_rm.o<br>
  CC      drivers/staging/android/ashmem.o<br>
  CC      drivers/usb/gadget/debug.o<br>
  CC      drivers/usb/core/quirks.o<br>
  CC      drivers/usb/core/devices.o<br>
  CC      drivers/staging/android/timed_output.o<br>
  LD      drivers/scsi/scsi_mod.o<br>
  CC      drivers/scsi/ufs/ufshcd-pltfrm.o<br>
  CC      drivers/usb/misc/ehset.o<br>
  CC      drivers/scsi/ufs/ufs-debugfs.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp.o<br>
  CC      drivers/staging/android/timed_gpio.o<br>
  CC      drivers/usb/core/port.o<br>
  CC      drivers/usb/mon/mon_main.o<br>
  CC      drivers/usb/core/hcd-pci.o<br>
  CC      drivers/usb/gadget/function/f_acm.o<br>
  CC      drivers/media/rc/keymaps/rc-terratec-slim-2.o<br>
  CC      drivers/usb/host/xhci.o<br>
  LD      drivers/usb/misc/built-in.o<br>
  CC      drivers/usb/gadget/function/u_serial.o<br>
  LD      drivers/media/rc/rc-core.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_debug.o<br>
  CC      drivers/staging/android/lowmemorykiller.o<br>
  CC      drivers/usb/mon/mon_stat.o<br>
  CC      drivers/usb/mon/mon_text.o<br>
  LD      drivers/usb/dwc3/dwc3.o<br>
  LD      drivers/usb/dwc3/built-in.o<br>
  CC      drivers/usb/phy/phy.o<br>
  CC      drivers/staging/android/sync.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_cache_config.o<br>
  CC      drivers/usb/phy/of.o<br>
  CC      drivers/scsi/ufs/ufs-qcom-debugfs.o<br>
  LD      drivers/platform/msm/sps/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-tevii-nec.o<br>
  CC      drivers/platform/msm/usb_bam.o<br>
  CC      drivers/usb/phy/class-dual-role.o<br>
  CC      drivers/usb/mon/mon_bin.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_video.o<br>
  CC      drivers/platform/msm/avtimer.o<br>
  CC      drivers/soc/qcom/smd_debug.o<br>
  LD      drivers/usb/core/usbcore.o<br>
  CC      drivers/soc/qcom/smd_private.o<br>
  CC      drivers/soc/qcom/smd_init_dt.o<br>
  LD      drivers/usb/core/built-in.o<br>
  LD      drivers/usb/gadget/legacy/built-in.o<br>
  CC      drivers/soc/qcom/smsm_debug.o<br>
  CC      drivers/soc/qcom/glink.o<br>
  CC      drivers/usb/host/xhci-mem.o<br>
  LD      drivers/platform/msm/built-in.o<br>
  LD      drivers/platform/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_cmd.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_writeback.o<br>
  LD      drivers/scsi/sd_mod.o<br>
  LD      drivers/scsi/ufs/built-in.o<br>
  CC      drivers/usb/phy/phy-generic.o<br>
  CC      drivers/media/rc/keymaps/rc-tivo.o<br>
  LD      drivers/scsi/built-in.o<br>
  CC      drivers/usb/gadget/udc/udc-core.o<br>
  CC      drivers/usb/phy/phy-msm-usb.o<br>
  CC      drivers/usb/phy/phy-msm-hsusb.o<br>
  CC      drivers/usb/gadget/function/f_serial.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiData.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_rotator.o<br>
  CC      drivers/usb/phy/phy-msm-ssusb-qmp.o<br>
  LD      drivers/usb/gadget/udc/built-in.o<br>
  CC      drivers/usb/serial/usb-serial.o<br>
  CC      drivers/soc/qcom/glink_debugfs.o<br>
  CC      drivers/staging/android/sw_sync.o<br>
  CC      drivers/staging/android/oneshot_sync.o<br>
  CC      drivers/usb/storage/scsiglue.o<br>
  CC      drivers/usb/serial/generic.o<br>
  CC      drivers/media/rc/keymaps/rc-total-media-in-hand.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiDebug.o<br>
  CC      drivers/soc/qcom/glink_ssr.o<br>
  LD      drivers/usb/mon/usbmon.o<br>
  CC      drivers/soc/qcom/glink_loopback_server.o<br>
  LD      drivers/usb/mon/built-in.o<br>
  CC      drivers/usb/gadget/function/f_ncm.o<br>
  CC      drivers/media/rc/keymaps/rc-total-media-in-hand-02.o<br>
  CC      drivers/media/rc/keymaps/rc-trekstor.o<br>
  CC      drivers/usb/gadget/android.o<br>
  CC      drivers/soc/qcom/glink_smd_xprt.o<br>
  CC      drivers/usb/gadget/function/f_ecm.o<br>
  CC      drivers/usb/gadget/function/f_mass_storage.o<br>
  CC      drivers/usb/phy/phy-msm-qusb.o<br>
  CC      drivers/soc/qcom/glink_smem_native_xprt.o<br>
  LD      drivers/staging/android/built-in.o<br>
  CC      drivers/soc/qcom/smem_log.o<br>
  CC      drivers/soc/qcom/smp2p.o<br>
  CC      drivers/soc/qcom/smp2p_debug.o<br>
  CC      drivers/usb/storage/protocol.o<br>
  CC      drivers/media/rc/keymaps/rc-tt-1500.o<br>
  CC      drivers/usb/gadget/ci13xxx_msm.o<br>
  CC      drivers/usb/gadget/function/storage_common.o<br>
  LD      drivers/usb/phy/built-in.o<br>
  CC      drivers/media/rc/keymaps/rc-twinhan1027.o<br>
  CC      drivers/usb/storage/transport.o<br>
  CC      drivers/usb/serial/bus.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-m1f.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiExt.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-s350.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiHCBB.o<br>
  CC      drivers/media/rc/keymaps/rc-videomate-tv-pvr.o<br>
  CC      drivers/soc/qcom/smp2p_sleepstate.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiInfo.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiLinkCntl.o<br>
  CC      drivers/usb/host/xhci-ring.o<br>
  CC      drivers/media/rc/keymaps/rc-winfast.o<br>
  CC      drivers/soc/qcom/smp2p_loopback.o<br>
  CC      drivers/soc/qcom/smp2p_test.o<br>
  CC      drivers/media/rc/keymaps/rc-winfast-usbii-deluxe.o<br>
  CC      drivers/usb/storage/usb.o<br>
  CC      drivers/usb/host/xhci-hub.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiLinkSupervision.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiStatus.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_overlay.o<br>
  CC      drivers/usb/host/xhci-dbg.o<br>
  LD      drivers/usb/serial/usbserial.o<br>
  LD      drivers/usb/serial/built-in.o<br>
  CC      drivers/soc/qcom/smp2p_spinlock_test.o<br>
  CC      drivers/usb/host/xhci-trace.o<br>
  CC      drivers/usb/gadget/function/f_fs.o<br>
  CC      drivers/usb/storage/initializers.o<br>
  CC      drivers/media/rc/keymaps/rc-su3000.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapApiTimer.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_layer.o<br>
  CC      drivers/usb/storage/sierra_ms.o<br>
  LD      drivers/media/rc/keymaps/built-in.o<br>
  LD      drivers/usb/host/xhci-plat-hcd.o<br>
  CC      drivers/soc/qcom/qmi_interface.o<br>
  LD      drivers/media/rc/built-in.o<br>
  CC      drivers/usb/gadget/function/f_uac1.o<br>
  CC      drivers/usb/storage/option_ms.o<br>
  LD      drivers/media/built-in.o<br>
  CC      drivers/usb/gadget/function/u_uac1.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_splash_logo.o<br>
  CC      drivers/usb/gadget/function/f_uac2.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapModule.o<br>
  CC      drivers/usb/gadget/function/f_uvc.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xAuthFsm.o<br>
  CC      drivers/usb/storage/usual-tables.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xPrf.o<br>
  CC      drivers/usb/storage/alauda.o<br>
  CC      drivers/soc/qcom/ipc_router_smd_xprt.o<br>
  CC      drivers/usb/storage/cypress_atacb.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xSuppRsnFsm.o<br>
  CC      drivers/usb/storage/datafab.o<br>
  CC      drivers/usb/storage/freecom.o<br>
  LD      drivers/usb/host/xhci-hcd.o<br>
  LD      drivers/usb/host/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_cdm.o<br>
  CC      drivers/usb/storage/isd200.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsnAsfPacket.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_smmu.o<br>
  CC      drivers/usb/storage/jumpshot.o<br>
  CC      drivers/usb/storage/karma.o<br>
  CC      drivers/usb/gadget/function/uvc_queue.o<br>
  CC      drivers/usb/storage/sddr09.o<br>
  CC      drivers/usb/gadget/function/uvc_v4l2.o<br>
  CC      drivers/usb/storage/sddr55.o<br>
  CC      drivers/usb/gadget/function/uvc_video.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmAesKeyWrap.o<br>
  CC      drivers/usb/gadget/function/f_audio_source.o<br>
  LD      drivers/usb/gadget/function/usb_f_acm.o<br>
  LD      drivers/usb/gadget/function/usb_f_serial.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_wfd.o<br>
  LD      drivers/usb/gadget/function/usb_f_ncm.o<br>
  CC      drivers/soc/qcom/memshare/heap_mem_ext_v01.o<br>
  LD      drivers/usb/gadget/function/usb_f_ecm.o<br>
  LD      drivers/usb/gadget/function/usb_f_mass_storage.o<br>
  CC      drivers/usb/storage/shuttle_usbat.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr.o<br>
  LD      drivers/usb/gadget/libcomposite.o<br>
  CC      drivers/soc/qcom/memshare/msm_memshare.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v1_7.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmEapol.o<br>
  LD      drivers/usb/storage/usb-storage.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v3.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_common.o<br>
  LD      drivers/usb/storage/ums-alauda.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmReplayCtr.o<br>
  LD      drivers/usb/gadget/function/usb_f_fs.o<br>
  LD      drivers/usb/gadget/function/usb_f_uac1.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr_v2.o<br>
  LD      drivers/usb/storage/ums-cypress.o<br>
  CC      drivers/soc/qcom/rpm_rbcpr_stats_v2.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_debug.o<br>
  LD      drivers/usb/gadget/function/usb_f_uac2.o<br>
  LD      drivers/usb/gadget/function/usb_f_uvc.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug_xlog.o<br>
  LD      drivers/usb/storage/ums-datafab.o<br>
  LD      drivers/usb/storage/ums-freecom.o<br>
  CC      drivers/soc/qcom/cpaccess64.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/bapRsnTxRx.o<br>
  LD      drivers/usb/storage/ums-isd200.o<br>
  CC      drivers/soc/qcom/rpm_stats.o<br>
  LD      drivers/usb/storage/ums-jumpshot.o<br>
  LD      drivers/usb/storage/ums-karma.o<br>
  LD      drivers/usb/storage/ums-sddr09.o<br>
  LD      drivers/usb/storage/ums-sddr55.o<br>
  CC      drivers/soc/qcom/qdsp6v2/apr_tal.o<br>
  LD      drivers/usb/gadget/function/usb_f_audio_source.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/btampFsm.o<br>
  LD      drivers/usb/storage/ums-usbat.o<br>
  LD      drivers/usb/storage/built-in.o<br>
  LD      drivers/usb/gadget/function/built-in.o<br>
  CC      drivers/soc/qcom/rpm_master_stat.o<br>
  CC      drivers/soc/qcom/rpm_rail_stats.o<br>
  CC      drivers/staging/prima/CORE/BAP/src/btampHCI.o<br>
  LD      drivers/soc/qcom/memshare/built-in.o<br>
  CC      drivers/soc/qcom/system_stats.o<br>
  CC      drivers/soc/qcom/perf_event_l2.o<br>
  CC      drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe.o<br>
  CC      drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe_cfg_i.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/bap_hdd_main.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_assoc.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_debugfs.o<br>
  CC      drivers/soc/qcom/rpm_log.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_host.o<br>
  CC      drivers/soc/qcom/qdsp6v2/voice_svc.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_dev_pwr.o<br>
  CC      drivers/soc/qcom/msm_tz_smmu.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_dp_utils.o<br>
  CC      drivers/soc/qcom/peripheral-loader.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_cmd.o<br>
  CC      drivers/soc/qcom/qdsp6v2/msm_audio_ion.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_status.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_early_suspend.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_ftm.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_hostapd.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_main.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_mib.o<br>
  CC      drivers/soc/qcom/subsys-pil-tz.o<br>
  CC      drivers/soc/qcom/qdsp6v2/adsp-loader.o<br>
  CC      drivers/soc/qcom/pil-q6v5.o<br>
  CC      drivers/soc/qcom/pil-msa.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_panel.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_oemdata.o<br>
  CC      drivers/soc/qcom/pil-q6v5-mss.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_scan.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_softap_tx_rx.o<br>
  CC      drivers/soc/qcom/msm_performance.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_tx_rx.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/msm_mdss_io_8974.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_phy.o<br>
  LD      drivers/soc/qcom/qdsp6v2/built-in.o<br>
  CC      drivers/soc/qcom/subsystem_notif.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_trace.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_clk.o<br>
  CC      drivers/soc/qcom/subsystem_restart.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wext.o<br>
  CC      drivers/soc/qcom/ramdump.o<br>
  CC      drivers/soc/qcom/sysmon.o<br>
  CC      drivers/soc/qcom/sysmon-qmi.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wmm.o<br>
  CC      drivers/soc/qcom/secure_buffer.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wowl.o<br>
  CC      drivers/soc/qcom/icnss.o<br>
  CC      drivers/soc/qcom/wlan_firmware_service_v01.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_panel.o<br>
  CC      drivers/soc/qcom/bam_dmux.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_util.o<br>
  CC      drivers/soc/qcom/scm-xpu.o<br>
  CC      drivers/soc/qcom/serial_num.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg80211.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_edid.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_cec_core.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dba_utils.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_io_util.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_tx.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_p2p.o<br>
  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_tdls.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgApi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgDebug.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_panel.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgParamName.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgProcMsg.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgSendMsg.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/dph/dphHashTable.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp2p2.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_cec.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_audio.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_wb.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_fb.o<br>
  LD      drivers/soc/qcom/built-in.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAIDmgmt.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAdmitControl.o<br>
  LD      drivers/soc/built-in.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_util.o<br>
  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_compat_utils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limApi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAssocUtils.o<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-mdp.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limDebug.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limFT.o<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-dsi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limIbssPeerMgmt.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limLinkMonitoringAlgo.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limLogDump.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limP2P.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessActionFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocReqFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocRspFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAuthFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessBeaconFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessCfgUpdates.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDeauthFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDisassocFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessLmmMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMessageQueue.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmReqMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmRspMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeReqFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeRspFrame.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessSmeReqMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limPropExtsUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limRMC.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limRoamingAlgo.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limScanResultUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSecurityUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendManagementFrames.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendSmeRspMessages.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSerDesUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSession.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSessionUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSmeReqUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limStaHashApi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limTimerUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limTrace.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limUtils.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessTdls.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmAP.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmApi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmDebug.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schApi.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconGen.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconProcess.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schDebug.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schMessage.o<br>
  CC      drivers/staging/prima/CORE/MAC/src/pe/rrm/rrmApi.o<br>
  CC      drivers/staging/prima/CORE/SAP/src/sapApiLinkCntl.o<br>
  CC      drivers/staging/prima/CORE/SAP/src/sapChSelect.o<br>
  CC      drivers/staging/prima/CORE/SAP/src/sapFsm.o<br>
  CC      drivers/staging/prima/CORE/SAP/src/sapModule.o<br>
  LD      drivers/video/fbdev/msm/../../msm/mdss/built-in.o<br>
  CC      drivers/staging/prima/CORE/SME/src/btc/btcApi.o<br>
  CC      drivers/staging/prima/CORE/SME/src/ccm/ccmApi.o<br>
  CC      drivers/staging/prima/CORE/SME/src/ccm/ccmLogDump.o<br>
  LD      drivers/video/fbdev/msm/../../msm/built-in.o<br>
  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_Api.o<br>
  LD      drivers/video/fbdev/msm/built-in.o<br>
  LD      drivers/video/fbdev/built-in.o<br>
  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_FTApi.o<br>
  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_Trace.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrApiRoam.o<br>
  LD      drivers/video/built-in.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrApiScan.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrCmdProcess.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrLogDump.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrLinkList.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrNeighborRoam.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrUtil.o<br>
  CC      drivers/staging/prima/CORE/SME/src/csr/csrTdlsProcess.o<br>
  CC      drivers/staging/prima/CORE/SME/src/oemData/oemDataApi.o<br>
  CC      drivers/staging/prima/CORE/SME/src/p2p/p2p_Api.o<br>
  CC      drivers/staging/prima/CORE/SME/src/pmc/pmcApi.o<br>
  CC      drivers/staging/prima/CORE/SME/src/pmc/pmc.o<br>
  CC      drivers/staging/prima/CORE/SME/src/pmc/pmcLogDump.o<br>
  CC      drivers/staging/prima/CORE/SME/src/QoS/sme_Qos.o<br>
  CC      drivers/staging/prima/CORE/SME/src/rrm/sme_rrm.o<br>
  CC      drivers/staging/prima/CORE/SME/src/nan/nan_Api.o<br>
  CC      drivers/staging/prima/CORE/SVC/src/btc/wlan_btc_svc.o<br>
  CC      drivers/staging/prima/CORE/SVC/src/nlink/wlan_nlink_srv.o<br>
  CC      drivers/staging/prima/CORE/SVC/src/ptt/wlan_ptt_sock_svc.o<br>
  CC      drivers/staging/prima/CORE/SVC/src/logging/wlan_logging_sock_svc.o<br>
  CC      drivers/staging/prima/CORE/SYS/common/src/wlan_qct_sys.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palApiComm.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palTimer.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/platform/src/VossWrapper.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/system/src/macInitApi.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/system/src/sysEntryFunc.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/dot11f.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logApi.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logDump.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/macTrace.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/parserApi.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsApi.o<br>
  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsParser.o<br>
  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl.o<br>
  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_ba.o<br>
  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_trace.o<br>
  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_hosupport.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_api.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_event.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_getBin.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_list.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_lock.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_memory.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_mq.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_nvitem.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_packet.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_sched.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_threads.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_timer.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_trace.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_types.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_utils.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_parser.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_stream_read.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_template_builtin.o<br>
  CC      drivers/staging/prima/CORE/VOSS/src/vos_diag.o<br>
  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda.o<br>
  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_debug.o<br>
  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_ds.o<br>
  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_legacy.o<br>
  CC      drivers/staging/prima/CORE/WDA/src/wlan_nv.o<br>
  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi.o<br>
  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_dp.o<br>
  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_sta.o<br>
  CC      drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_bd.o<br>
  CC      drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_ds.o<br>
  CC      drivers/staging/prima/CORE/WDI/TRP/CTS/src/wlan_qct_wdi_cts.o<br>
  CC      drivers/staging/prima/CORE/WDI/TRP/DTS/src/wlan_qct_wdi_dts.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_api.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_device.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_msg.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_packet.o<br>
  LD      drivers/usb/gadget/g_android.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_sync.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_timer.o<br>
  LD      drivers/usb/gadget/built-in.o<br>
  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_trace.o<br>
  LD      drivers/usb/built-in.o<br>
  LD      drivers/staging/prima/wlan.o<br>
  LD      drivers/staging/prima/built-in.o<br>
  LD      drivers/staging/built-in.o<br>
  LD      drivers/built-in.o<br>
  LINK    vmlinux<br>
  LD      vmlinux.o<br>
  MODPOST vmlinux.o<br>
  GEN     .version<br>
  CHK     include/generated/compile.h<br>
  UPD     include/generated/compile.h<br>
  CC      init/version.o<br>
  LD      init/built-in.o<br>
  KSYM    .tmp_kallsyms1.o<br>
  KSYM    .tmp_kallsyms2.o<br>
  LD      vmlinux<br>
  SORTEX  vmlinux<br>
  SYSMAP  System.map<br>
  OBJCOPY arch/arm64/boot/Image<br>
  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3-mido.dtb<br>
  GZIP    arch/arm64/boot/Image.gz<br>
  CAT     arch/arm64/boot/Image.gz-dtb<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
Building DTBs<br>
make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
  CHK     include/config/kernel.release<br>
  GEN     ./Makefile<br>
  CHK     include/generated/uapi/linux/version.h<br>
  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel<br>
  CHK     include/generated/utsrelease.h<br>
  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh<br>
make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'<br>
make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'<br>
Kernel Modules not enabled<br>
[100% 5859/5859] Target boot image: /home/stalker/hadk/out/target/product/mido/boot.img<br>
/home/stalker/hadk/out/target/product/mido/boot.img maxsize=68395008 blocksize=135168 total=12419072 reserve=811008<br>
[100% 5859/5859] Install: /home/stalker/hadk/out/target/product/mido/hybris-recovery.img<br>
<br>
#### make completed successfully (02:30 (mm:ss)) ####<br>
```
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
`============================================`<br>
`PLATFORM_VERSION_CODENAME=REL`<br>
`PLATFORM_VERSION=7.1.2`<br>
`LINEAGE_VERSION=14.1-20190629-UNOFFICIAL-mido`<br>
`TARGET_PRODUCT=lineage_mido`<br>
`TARGET_BUILD_VARIANT=userdebug`<br>
`TARGET_BUILD_TYPE=release`<br>
`TARGET_BUILD_APPS=`<br>
`TARGET_ARCH=arm64`<br>
`TARGET_ARCH_VARIANT=armv8-a`<br>
`TARGET_CPU_VARIANT=generic`<br>
`TARGET_2ND_ARCH=arm`<br>
`TARGET_2ND_ARCH_VARIANT=armv7-a-neon`<br>
`TARGET_2ND_CPU_VARIANT=cortex-a53`<br>
`HOST_ARCH=x86_64`<br>
`HOST_2ND_ARCH=x86`<br>
`HOST_OS=linux`<br>
`HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty`<br>
`HOST_CROSS_OS=windows`<br>
`HOST_CROSS_ARCH=x86`<br>
`HOST_CROSS_2ND_ARCH=x86_64`<br>
`HOST_BUILD_TYPE=release`<br>
`BUILD_ID=NJH47F`<br>
`OUT_DIR=/home/stalker/hadk/out`<br>
`============================================`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/command.o build/kati/command.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/affinity.o build/kati/affinity.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/dep.o build/kati/dep.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/eval.o build/kati/eval.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file.o build/kati/file.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/exec.o build/kati/exec.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/expr.o build/kati/expr.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file_cache.o build/kati/file_cache.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/fileutil.o build/kati/fileutil.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/find.o build/kati/find.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/func.o build/kati/func.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/flags.o build/kati/flags.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/io.o build/kati/io.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/log.o build/kati/log.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/main.o build/kati/main.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/ninja.o build/kati/ninja.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/parser.o build/kati/parser.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/regen.o build/kati/regen.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/rule.o build/kati/rule.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stats.o build/kati/stats.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stmt.o build/kati/stmt.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/string_piece.o build/kati/string_piece.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stringprintf.o build/kati/stringprintf.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/strutil.o build/kati/strutil.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/symtab.o build/kati/symtab.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/thread_pool.o build/kati/thread_pool.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/timeutil.o build/kati/timeutil.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/var.o build/kati/var.cc`<br>
`echo '// +build ignore' > /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc`<br>
`echo >> /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc`<br>
`echo 'const char* kGitVersion = "bc43789a938c10cb00b81ddf08239c1b4cea48bb";' >> /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -Wall -Werror -MMD -MP -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/makeparallel_intermediates/makeparallel.o build/tools/makeparallel/makeparallel.cpp`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -fno-exceptions -Wno-multichar -m64 -Wa,--noexecstack -fPIC -no-canonical-prefixes -U_FORTIFY_SOURCE -D_FORTIFY_SOURCE=2 -fstack-protector -D__STDC_FORMAT_MACROS -D__STDC_CONSTANT_MACROS -DANDROID -fmessage-length=0 -W -Wall -Wno-unused -Winit-self -Wpointer-arith -O2 -g -fno-strict-aliasing -DNDEBUG -UDEBUG  -D__compiler_offsetof=__builtin_offsetof -Werror=int-conversion -Wno-reserved-id-macro -Wno-format-pedantic -Wno-unused-command-line-argument   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 -fstack-protector-strong    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -target x86_64-linux-gnu   -Wsign-promo  -Wno-inconsistent-missing-override   --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8 -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/x86_64-linux -isystem prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/include/c++/4.8/backward -target x86_64-linux-gnu -c -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.cc`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -m64 -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -Wl,--no-undefined-version    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/lib64/ -target x86_64-linux-gnu -static -std=c++11 -Wall -Werror -MMD -MP -o /home/stalker/hadk/out/host/linux-x86/bin/makeparallel /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/makeparallel_intermediates/makeparallel.o -lrt -lpthread`<br>
`prebuilts/clang/host/linux-x86/clang-2690385/bin/clang++   -m64 -Wl,-z,noexecstack -Wl,-z,relro -Wl,-z,now -Wl,--no-undefined-version    --gcc-toolchain=prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8 --sysroot prebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/sysroot -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/bin -Bprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/lib/gcc/x86_64-linux/4.8 -Lprebuilts/gcc/linux-x86/host/x86_64-linux-glibc2.15-4.8/x86_64-linux/lib64/ -target x86_64-linux-gnu -static -Wl,--whole-archive -lpthread -Wl,--no-whole-archive -ldl -std=c++11 -g -W -Wall -MMD -MP -O -DNOLOG -march=native -o /home/stalker/hadk/out/host/linux-x86/bin/ckati /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/affinity.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/command.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/dep.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/eval.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/exec.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/expr.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/file_cache.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/fileutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/find.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/flags.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/func.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/io.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/log.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/main.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/ninja.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/parser.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/regen.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/rule.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stats.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stmt.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/string_piece.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/stringprintf.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/strutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/symtab.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/thread_pool.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/timeutil.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/var.o /home/stalker/hadk/out/host/linux-x86/obj/EXECUTABLES/ckati_intermediates/version.o -lrt -lpthread`<br>
`sem_open.c:333: warning: the use of `mktemp' is dangerous, better use `mkstemp' or `mkdtemp'`<br>
`Running kati to generate build-lineage_mido.ninja...`<br>
`/home/stalker/hadk/out/build-lineage_mido.ninja is missing, regenerating...`<br>
`============================================`<br>
`PLATFORM_VERSION_CODENAME=REL`<br>
`PLATFORM_VERSION=7.1.2`<br>
`LINEAGE_VERSION=14.1-20190629-UNOFFICIAL-mido`<br>
`TARGET_PRODUCT=lineage_mido`<br>
`TARGET_BUILD_VARIANT=userdebug`<br>
`TARGET_BUILD_TYPE=release`<br>
`TARGET_BUILD_APPS=`<br>
`TARGET_ARCH=arm64`<br>
`TARGET_ARCH_VARIANT=armv8-a`<br>
`TARGET_CPU_VARIANT=generic`<br>
`TARGET_2ND_ARCH=arm`<br>
`TARGET_2ND_ARCH_VARIANT=armv7-a-neon`<br>
`TARGET_2ND_CPU_VARIANT=cortex-a53`<br>
`HOST_ARCH=x86_64`<br>
`HOST_2ND_ARCH=x86`<br>
`HOST_OS=linux`<br>
`HOST_OS_EXTRA=Linux-4.15.0-52-generic-x86_64-with-Ubuntu-14.04-trusty`<br>
`HOST_CROSS_OS=windows`<br>
`HOST_CROSS_ARCH=x86`<br>
`HOST_CROSS_2ND_ARCH=x86_64`<br>
`HOST_BUILD_TYPE=release`<br>
`BUILD_ID=NJH47F`<br>
`OUT_DIR=/home/stalker/hadk/out`<br>
`============================================`<br>
`Checking build tools versions...`<br>
`build/core/binary.mk:37: hal3-test-app uses kernel headers, but does not depend on them!`<br>
`external/speex/Android.mk:56: TODOArm64: enable neon in libspeex`<br>
`perl: warning: Setting locale failed.`<br>
`perl: warning: Please check that your locale settings:`<br>
`	LANGUAGE = (unset),`<br>
`	LC_ALL = (unset),`<br>
`	LC_MESSAGES = "en_US.UTF-8",`<br>
`	LC_CTYPE = "en_US.UTF-8",`<br>
`	LANG = (unset)`<br>
`    are supported and installed on your system.`<br>
`perl: warning: Falling back to the standard locale ("C").`<br>
`perl: warning: Setting locale failed.`<br>
`perl: warning: Please check that your locale settings:`<br>
`	LANGUAGE = (unset),`<br>
`	LC_ALL = (unset),`<br>
`	LC_MESSAGES = "en_US.UTF-8",`<br>
`	LC_CTYPE = "en_US.UTF-8",`<br>
`	LANG = (unset)`<br>
`    are supported and installed on your system.`<br>
`perl: warning: Falling back to the standard locale ("C").`<br>
`hybris/hybris-boot/Android.mk:71: ********************* /boot appears to live on /dev/block/bootdevice/by-name/boot`<br>
`hybris/hybris-boot/Android.mk:72: ********************* /data appears to live on /dev/block/bootdevice/by-name/userdata`<br>
`build/core/Makefile:34: warning: overriding commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'`<br>
`build/core/base_rules.mk:320: warning: ignoring old commands for target `/home/stalker/hadk/out/target/product/mido/system/bin/wcnss_service'`<br>
`Starting build with ninja`<br>
`ninja: Entering directory `.'`<br>
`[  5% 313/5859] Building Kernel Config`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  GEN     ./Makefile`<br>
`  HOSTCC  scripts/basic/fixdep`<br>
`  HOSTCC  scripts/kconfig/conf.o`<br>
`  SHIPPED scripts/kconfig/zconf.tab.c`<br>
`  SHIPPED scripts/kconfig/zconf.lex.c`<br>
`  SHIPPED scripts/kconfig/zconf.hash.c`<br>
`  HOSTCC  scripts/kconfig/zconf.tab.o`<br>
`  HOSTLD  scripts/kconfig/conf`<br>
`#`<br>
`# configuration written to .config`<br>
`#`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  GEN     ./Makefile`<br>
`scripts/kconfig/conf --savedefconfig=defconfig Kconfig`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`[ 11% 702/5859] Building Kernel Headers`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  GEN     ./Makefile`<br>
`#`<br>
`# configuration written to .config`<br>
`#`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  CHK     include/generated/uapi/linux/version.h`<br>
`  UPD     include/generated/uapi/linux/version.h`<br>
`  WRAP    arch/arm64/include/generated/asm/bugs.h`<br>
`  WRAP    arch/arm64/include/generated/asm/checksum.h`<br>
`  WRAP    arch/arm64/include/generated/asm/clkdev.h`<br>
`  WRAP    arch/arm64/include/generated/asm/cputime.h`<br>
`  WRAP    arch/arm64/include/generated/asm/current.h`<br>
`  WRAP    arch/arm64/include/generated/asm/delay.h`<br>
`  WRAP    arch/arm64/include/generated/asm/div64.h`<br>
`  WRAP    arch/arm64/include/generated/asm/dma.h`<br>
`  WRAP    arch/arm64/include/generated/asm/dma-contiguous.h`<br>
`  WRAP    arch/arm64/include/generated/asm/early_ioremap.h`<br>
`  WRAP    arch/arm64/include/generated/asm/emergency-restart.h`<br>
`  WRAP    arch/arm64/include/generated/asm/errno.h`<br>
`  WRAP    arch/arm64/include/generated/asm/ftrace.h`<br>
`  WRAP    arch/arm64/include/generated/asm/hash.h`<br>
`  WRAP    arch/arm64/include/generated/asm/hw_irq.h`<br>
`  WRAP    arch/arm64/include/generated/asm/ioctl.h`<br>
`  WRAP    arch/arm64/include/generated/asm/ioctls.h`<br>
`  WRAP    arch/arm64/include/generated/asm/ipcbuf.h`<br>
`  WRAP    arch/arm64/include/generated/asm/irq_regs.h`<br>
`  WRAP    arch/arm64/include/generated/asm/kdebug.h`<br>
`  WRAP    arch/arm64/include/generated/asm/kmap_types.h`<br>
`  WRAP    arch/arm64/include/generated/asm/kvm_para.h`<br>
`  WRAP    arch/arm64/include/generated/asm/local.h`<br>
`  WRAP    arch/arm64/include/generated/asm/local64.h`<br>
`  WRAP    arch/arm64/include/generated/asm/mcs_spinlock.h`<br>
`  WRAP    arch/arm64/include/generated/asm/mman.h`<br>
`  WRAP    arch/arm64/include/generated/asm/msi.h`<br>
`  WRAP    arch/arm64/include/generated/asm/msgbuf.h`<br>
`  WRAP    arch/arm64/include/generated/asm/mutex.h`<br>
`  WRAP    arch/arm64/include/generated/asm/pci.h`<br>
`  WRAP    arch/arm64/include/generated/asm/pci-bridge.h`<br>
`  WRAP    arch/arm64/include/generated/asm/poll.h`<br>
`  WRAP    arch/arm64/include/generated/asm/preempt.h`<br>
`  WRAP    arch/arm64/include/generated/asm/resource.h`<br>
`  WRAP    arch/arm64/include/generated/asm/rwsem.h`<br>
`  WRAP    arch/arm64/include/generated/asm/scatterlist.h`<br>
`  WRAP    arch/arm64/include/generated/asm/sections.h`<br>
`  WRAP    arch/arm64/include/generated/asm/segment.h`<br>
`  WRAP    arch/arm64/include/generated/asm/sembuf.h`<br>
`  WRAP    arch/arm64/include/generated/asm/serial.h`<br>
`  WRAP    arch/arm64/include/generated/asm/shmbuf.h`<br>
`  WRAP    arch/arm64/include/generated/asm/simd.h`<br>
`  WRAP    arch/arm64/include/generated/asm/sizes.h`<br>
`  WRAP    arch/arm64/include/generated/asm/socket.h`<br>
`  WRAP    arch/arm64/include/generated/asm/sockios.h`<br>
`  WRAP    arch/arm64/include/generated/asm/swab.h`<br>
`  WRAP    arch/arm64/include/generated/asm/switch_to.h`<br>
`  WRAP    arch/arm64/include/generated/asm/termbits.h`<br>
`  WRAP    arch/arm64/include/generated/asm/termios.h`<br>
`  WRAP    arch/arm64/include/generated/asm/topology.h`<br>
`  WRAP    arch/arm64/include/generated/asm/trace_clock.h`<br>
`  WRAP    arch/arm64/include/generated/asm/types.h`<br>
`  WRAP    arch/arm64/include/generated/asm/user.h`<br>
`  WRAP    arch/arm64/include/generated/asm/unaligned.h`<br>
`  WRAP    arch/arm64/include/generated/asm/vga.h`<br>
`  WRAP    arch/arm64/include/generated/asm/xor.h`<br>
`  WRAP    arch/arm64/include/generated/uapi/asm/kvm_para.h`<br>
`  HOSTCC  scripts/unifdef`<br>
`  INSTALL usr/include/asm-generic/ (35 files)`<br>
`  INSTALL usr/include/drm/ (18 files)`<br>
`  INSTALL usr/include/media/ (21 files)`<br>
`  INSTALL usr/include/mtd/ (5 files)`<br>
`  INSTALL usr/include/misc/ (1 file)`<br>
`  INSTALL usr/include/linux/caif/ (2 files)`<br>
`  INSTALL usr/include/scsi/fc/ (4 files)`<br>
`  INSTALL usr/include/rdma/ (6 files)`<br>
`  INSTALL usr/include/scsi/ufs/ (2 files)`<br>
`  INSTALL usr/include/linux/can/ (5 files)`<br>
`  INSTALL usr/include/linux/dvb/ (8 files)`<br>
`  INSTALL usr/include/linux/byteorder/ (2 files)`<br>
`  INSTALL usr/include/linux/../../../usr/include/linux/staging/android/uapi/ (2 files)`<br>
`  INSTALL usr/include/sound/ (19 files)`<br>
`  INSTALL usr/include/linux/hdlc/ (1 file)`<br>
`  INSTALL usr/include/video/ (5 files)`<br>
`  INSTALL usr/include/linux/hsi/ (1 file)`<br>
`  INSTALL usr/include/uapi/ (0 file)`<br>
`  INSTALL usr/include/xen/ (4 files)`<br>
`  INSTALL usr/include/linux/mfd/wcd9xxx/ (2 files)`<br>
`  INSTALL usr/include/linux/isdn/ (1 file)`<br>
`  INSTALL usr/include/linux/mmc/ (3 files)`<br>
`  INSTALL usr/include/linux/netfilter/ipset/ (4 files)`<br>
`  INSTALL usr/include/scsi/ (5 files)`<br>
`  INSTALL usr/include/linux/netfilter_arp/ (2 files)`<br>
`  INSTALL usr/include/linux/netfilter_bridge/ (17 files)`<br>
`  INSTALL usr/include/linux/netfilter_ipv4/ (10 files)`<br>
`  INSTALL usr/include/linux/netfilter_ipv6/ (12 files)`<br>
`  INSTALL usr/include/linux/mfd/ (1 file)`<br>
`  INSTALL usr/include/linux/nfc/ (1 file)`<br>
`  INSTALL usr/include/linux/netfilter/ (85 files)`<br>
`  INSTALL usr/include/linux/nfsd/ (5 files)`<br>
`  INSTALL usr/include/linux/raid/ (2 files)`<br>
`  INSTALL usr/include/linux/spi/ (1 file)`<br>
`  INSTALL usr/include/linux/tc_ematch/ (4 files)`<br>
`  INSTALL usr/include/linux/sunrpc/ (1 file)`<br>
`  INSTALL usr/include/linux/tc_act/ (8 files)`<br>
`  INSTALL usr/include/linux/wimax/ (1 file)`<br>
`  INSTALL usr/include/linux/usb/ (11 files)`<br>
`  INSTALL usr/include/linux/ (471 files)`<br>
`  INSTALL usr/include/asm/ (35 files)`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`[ 53% 3153/5859] target thumb C: libunwind_32 <= external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c`<br>
`external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c:75:10: warning: Implement get_list_addr(), please. [-W#pragma-messages]`<br>
`# pragma message("Implement get_list_addr(), please.")`<br>
`         ^`<br>
`1 warning generated.`<br>
`[ 58% 3445/5859] target  C: libunwind <= external/libunwind/src/ptrace/_UPT_access_fpreg.c`<br>
`external/libunwind/src/ptrace/_UPT_access_fpreg.c:113:10: warning: _UPT_access_fpreg is not implemented and not currently used. [-W#pragma-messages]`<br>
`# pragma message("_UPT_access_fpreg is not implemented and not currently used.")`<br>
`         ^`<br>
`1 warning generated.`<br>
`[ 58% 3448/5859] target  C: libunwind <= external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c`<br>
`external/libunwind/src/ptrace/_UPT_get_dyn_info_list_addr.c:75:10: warning: Implement get_list_addr(), please. [-W#pragma-messages]`<br>
`# pragma message("Implement get_list_addr(), please.")`<br>
`         ^`<br>
`1 warning generated.`<br>
`[ 62% 3635/5859] target  C: libext2_uuid <= external/e2fsprogs/lib/uuid/gen_uuid.c`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:224:39: warning: unused parameter 'node_id' [-Wunused-parameter]`<br>
`static int get_node_id(unsigned char *node_id)`<br>
`                                      ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:36: warning: unused parameter 'op' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                   ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:47: warning: unused parameter 'out' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                              ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:57: warning: unused parameter 'num' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                                        ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:423:16: warning: unused function 'read_all' [-Wunused-function]`<br>
`static ssize_t read_all(int fd, char *buf, size_t count)`<br>
`               ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:450:13: warning: unused function 'close_all_fds' [-Wunused-function]`<br>
`static void close_all_fds(void)`<br>
`            ^`<br>
`6 warnings generated.`<br>
`[ 62% 3641/5859] target  C: libext2_blkid <= external/e2fsprogs/lib/blkid/probe.c`<br>
`external/e2fsprogs/lib/blkid/probe.c:883:42: warning: unused parameter 'probe' [-Wunused-parameter]`<br>
`static int probe_zfs(struct blkid_probe *probe, struct blkid_magic *id,`<br>
`                                         ^`<br>
`external/e2fsprogs/lib/blkid/probe.c:883:69: warning: unused parameter 'id' [-Wunused-parameter]`<br>
`static int probe_zfs(struct blkid_probe *probe, struct blkid_magic *id,`<br>
`                                                                    ^`<br>
`external/e2fsprogs/lib/blkid/probe.c:884:23: warning: unused parameter 'buf' [-Wunused-parameter]`<br>
`                     unsigned char *buf)`<br>
`                                    ^`<br>
`external/e2fsprogs/lib/blkid/probe.c:1384:24: warning: unused parameter 'id' [-Wunused-parameter]`<br>
`                        struct blkid_magic *id,`<br>
`                                            ^`<br>
`external/e2fsprogs/lib/blkid/probe.c:1400:33: warning: unused parameter 'id' [-Wunused-parameter]`<br>
`            struct blkid_magic *id,`<br>
`                                ^`<br>
`5 warnings generated.`<br>
`[ 62% 3645/5859] target  C: libext2_blkid <= external/e2fsprogs/lib/blkid/probe_exfat.c`<br>
`external/e2fsprogs/lib/blkid/probe_exfat.c:177:24: warning: passing 'char [128]' to parameter of type 'unsigned char *' converts between pointers to integer types with different sign [-Wpointer-sign]`<br>
`                unicode_16le_to_utf8(utf8_label, sizeof(utf8_label), label->name, label->length * 2);`<br>
`                                     ^~~~~~~~~~`<br>
`external/e2fsprogs/lib/blkid/probe_exfat.c:128:49: note: passing argument to parameter 'str' here`<br>
`static void unicode_16le_to_utf8(unsigned char *str, int out_len,`<br>
`                                                ^`<br>
`1 warning generated.`<br>
`[ 62% 3648/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/mkquota.c`<br>
`external/e2fsprogs/lib/quota/mkquota.c:53:54: warning: unused parameter 'fmt' [-Wunused-parameter]`<br>
`int quota_file_exists(ext2_filsys fs, int qtype, int fmt)`<br>
`                                                     ^`<br>
`external/e2fsprogs/lib/quota/mkquota.c:291:76: warning: unused parameter 'ino' [-Wunused-parameter]`<br>
`void quota_data_add(quota_ctx_t qctx, struct ext2_inode *inode, ext2_ino_t ino,`<br>
`                                                                           ^`<br>
`external/e2fsprogs/lib/quota/mkquota.c:317:76: warning: unused parameter 'ino' [-Wunused-parameter]`<br>
`void quota_data_sub(quota_ctx_t qctx, struct ext2_inode *inode, ext2_ino_t ino,`<br>
`                                                                           ^`<br>
`external/e2fsprogs/lib/quota/mkquota.c:343:21: warning: unused parameter 'ino' [-Wunused-parameter]`<br>
`                       ext2_ino_t ino, int adjust)`<br>
`                                  ^`<br>
`4 warnings generated.`<br>
`[ 62% 3650/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio_tree.c`<br>
`external/e2fsprogs/lib/quota/quotaio_tree.c:38:16: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`        for (i = 0; i < info->dqi_entry_size; i++)`<br>
`                    ~ ^ ~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/quota/quotaio_tree.c:626:16: warning: comparison of integers of different signs: 'uint' (aka 'unsigned int') and 'int' [-Wsign-compare]`<br>
`        for (i = 0; i < blocks; i++)`<br>
`                    ~ ^ ~~~~~~`<br>
`2 warnings generated.`<br>
`[ 62% 3651/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio.c`<br>
`external/e2fsprogs/lib/quota/quotaio.c:100:48: warning: unused parameter 'fs' [-Wunused-parameter]`<br>
`static int compute_num_blocks_proc(ext2_filsys fs, blk64_t *blocknr,`<br>
`                                               ^`<br>
`external/e2fsprogs/lib/quota/quotaio.c:100:61: warning: unused parameter 'blocknr' [-Wunused-parameter]`<br>
`static int compute_num_blocks_proc(ext2_filsys fs, blk64_t *blocknr,`<br>
`                                                            ^`<br>
`2 warnings generated.`<br>
`[ 62% 3652/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/quotaio_v2.c`<br>
`external/e2fsprogs/lib/quota/quotaio_v2.c:160:40: warning: comparison of integers of different signs: '__u32' (aka 'unsigned int') and 'int' [-Wsign-compare]`<br>
`        if (ext2fs_le32_to_cpu(dqh.dqh_magic) != file_magics[type]) {`<br>
`            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/quota/quotaio_v2.c:161:41: warning: comparison of integers of different signs: '__u32' (aka 'unsigned int') and 'int' [-Wsign-compare]`<br>
`                if (ext2fs_be32_to_cpu(dqh.dqh_magic) == file_magics[type])`<br>
`                    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/quota/quotaio_v2.c:278:43: warning: unused parameter 'h' [-Wunused-parameter]`<br>
`static int v2_report(struct quota_handle *h, int verbose)`<br>
`                                          ^`<br>
`external/e2fsprogs/lib/quota/quotaio_v2.c:278:50: warning: unused parameter 'verbose' [-Wunused-parameter]`<br>
`static int v2_report(struct quota_handle *h, int verbose)`<br>
`                                                 ^`<br>
`4 warnings generated.`<br>
`[ 62% 3657/5859] target  C: libext2_quota <= external/e2fsprogs/lib/quota/../../e2fsck/dict.c`<br>
`external/e2fsprogs/lib/quota/../../e2fsck/dict.c:21:9: warning: 'NDEBUG' macro redefined [-Wmacro-redefined]`<br>
`#define NDEBUG`<br>
`        ^`<br>
`<command line>:7:9: note: previous definition is here`<br>
`#define NDEBUG 1`<br>
`        ^`<br>
`1 warning generated.`<br>
`[ 69% 4093/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/csum.c`<br>
`external/e2fsprogs/lib/ext2fs/csum.c:37:9: warning: unused variable 'offset' [-Wunused-variable]`<br>
`        size_t offset;`<br>
`               ^`<br>
`external/e2fsprogs/lib/ext2fs/csum.c:40:6: warning: variable 'crc' is used uninitialized whenever 'if' condition is false [-Wsometimes-uninitialized]`<br>
`        if (fs->super->s_feature_ro_compat & EXT4_FEATURE_RO_COMPAT_GDT_CSUM) {`<br>
`            ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/csum.c:82:9: note: uninitialized use occurs here`<br>
`        return crc;`<br>
`               ^~~`<br>
`external/e2fsprogs/lib/ext2fs/csum.c:40:2: note: remove the 'if' if its condition is always true`<br>
`        if (fs->super->s_feature_ro_compat & EXT4_FEATURE_RO_COMPAT_GDT_CSUM) {`<br>
`        ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/csum.c:38:11: note: initialize the variable 'crc' to silence this warning`<br>
`        __u16 crc;`<br>
`                 ^`<br>
`                  = 0`<br>
`2 warnings generated.`<br>
`[ 69% 4097/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/ext_attr.c`<br>
`external/e2fsprogs/lib/ext2fs/ext_attr.c:154:1: warning: all paths through this function will call itself [-Winfinite-recursion]`<br>
`{`<br>
`^`<br>
`1 warning generated.`<br>
`[ 70% 4116/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/inline.c`<br>
`external/e2fsprogs/lib/ext2fs/inline.c:47:12: warning: unused variable 'retval' [-Wunused-variable]`<br>
`        errcode_t retval;`<br>
`                  ^`<br>
`1 warning generated.`<br>
`[ 70% 4128/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/ismounted.c`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:49:43: warning: unused parameter 'mnt_fsname' [-Wunused-parameter]`<br>
`static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,`<br>
`                                          ^`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:49:61: warning: unused parameter 'mnt_rdev' [-Wunused-parameter]`<br>
`static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,`<br>
`                                                            ^`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:50:11: warning: unused parameter 'file_dev' [-Wunused-parameter]`<br>
`                                dev_t file_dev, ino_t file_ino)`<br>
`                                      ^`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:50:27: warning: unused parameter 'file_ino' [-Wunused-parameter]`<br>
`                                dev_t file_dev, ino_t file_ino)`<br>
`                                                      ^`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:365:3: warning: "Can't use getmntent or getmntinfo to check for mounted filesystems!" [-W#warnings]`<br>
` #warning "Can't use getmntent or getmntinfo to check for mounted filesystems!"`<br>
`  ^`<br>
`external/e2fsprogs/lib/ext2fs/ismounted.c:49:12: warning: unused function 'check_loop_mounted' [-Wunused-function]`<br>
`static int check_loop_mounted(const char *mnt_fsname, dev_t mnt_rdev,`<br>
`           ^`<br>
`6 warnings generated.`<br>
`[ 70% 4139/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/symlink.c`<br>
`external/e2fsprogs/lib/ext2fs/symlink.c:33:23: warning: unused variable 'handle' [-Wunused-variable]`<br>
`        ext2_extent_handle_t    handle;`<br>
`                                ^`<br>
`1 warning generated.`<br>
`[ 70% 4144/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/tdb.c`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:411:13: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`            (ltype == tdb->global_lock.ltype || ltype == F_RDLCK)) {`<br>
`             ~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:506:13: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`            (ltype == tdb->global_lock.ltype || ltype == F_RDLCK)) {`<br>
`             ~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:627:55: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]`<br>
`        if (tdb->global_lock.count && tdb->global_lock.ltype == ltype) {`<br>
`                                      ~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:671:29: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]`<br>
`        if (tdb->global_lock.ltype != ltype || tdb->global_lock.count == 0) {`<br>
`            ~~~~~~~~~~~~~~~~~~~~~~ ^  ~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:854:17: warning: comparison of integers of different signs: 'off_t' (aka 'long') and 'size_t' (aka 'unsigned long') [-Wsign-compare]`<br>
`        if (st.st_size < (size_t)len) {`<br>
`            ~~~~~~~~~~ ^ ~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1535:72: warning: unused parameter 'probe' [-Wunused-parameter]`<br>
`static int transaction_oob(struct tdb_context *tdb, tdb_off_t len, int probe)`<br>
`                                                                       ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1561:51: warning: unused parameter 'tdb' [-Wunused-parameter]`<br>
`static int transaction_brlock(struct tdb_context *tdb, tdb_off_t offset,`<br>
`                                                  ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1561:66: warning: unused parameter 'offset' [-Wunused-parameter]`<br>
`static int transaction_brlock(struct tdb_context *tdb, tdb_off_t offset,`<br>
`                                                                 ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1562:14: warning: unused parameter 'rw_type' [-Wunused-parameter]`<br>
`                              int rw_type, int lck_type, int probe, size_t len)`<br>
`                                  ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1562:27: warning: unused parameter 'lck_type' [-Wunused-parameter]`<br>
`                              int rw_type, int lck_type, int probe, size_t len)`<br>
`                                               ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1562:41: warning: unused parameter 'probe' [-Wunused-parameter]`<br>
`                              int rw_type, int lck_type, int probe, size_t len)`<br>
`                                                             ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1562:55: warning: unused parameter 'len' [-Wunused-parameter]`<br>
`                              int rw_type, int lck_type, int probe, size_t len)`<br>
`                                                                           ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1743:64: warning: unused parameter 'offset' [-Wunused-parameter]`<br>
`static int transaction_sync(struct tdb_context *tdb, tdb_off_t offset, tdb_len_t length)`<br>
`                                                               ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:1743:82: warning: unused parameter 'length' [-Wunused-parameter]`<br>
`static int transaction_sync(struct tdb_context *tdb, tdb_off_t offset, tdb_len_t length)`<br>
`                                                                                 ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:3004:12: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`        for (i=0;i<tdb->header.hash_size;i++) {`<br>
`                 ~^~~~~~~~~~~~~~~~~~~~~~`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:3097:63: warning: unused parameter 'private_data' [-Wunused-parameter]`<br>
`static int tdb_key_compare(TDB_DATA key, TDB_DATA data, void *private_data)`<br>
`                                                              ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:3801:45: warning: unused parameter 'tdb' [-Wunused-parameter]`<br>
`static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)`<br>
`                                            ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:3801:71: warning: unused parameter 'level' [-Wunused-parameter]`<br>
`static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)`<br>
`                                                                      ^`<br>
`external/e2fsprogs/lib/ext2fs/tdb.c:3801:90: warning: unused parameter 'fmt' [-Wunused-parameter]`<br>
`static void null_log_fn(struct tdb_context *tdb, enum tdb_debug_level level, const char *fmt, ...)`<br>
`                                                                                         ^`<br>
`19 warnings generated.`<br>
`[ 70% 4145/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/undo_io.c`<br>
`external/e2fsprogs/lib/ext2fs/undo_io.c:105:1: warning: missing field 'discard' initializer [-Wmissing-field-initializers]`<br>
`};`<br>
`^`<br>
`1 warning generated.`<br>
`[ 70% 4147/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/unix_io.c`<br>
`external/e2fsprogs/lib/ext2fs/unix_io.c:137:1: warning: missing field 'reserved' initializer [-Wmissing-field-initializers]`<br>
`};`<br>
`^`<br>
`1 warning generated.`<br>
`[ 70% 4154/5859] target  C: libext2fs <= external/e2fsprogs/lib/ext2fs/test_io.c`<br>
`external/e2fsprogs/lib/ext2fs/test_io.c:100:1: warning: missing field 'reserved' initializer [-Wmissing-field-initializers]`<br>
`};`<br>
`^`<br>
`1 warning generated.`<br>
`[ 71% 4187/5859] target  C: libext4_utils_static <= system/extras/ext4_utils/make_ext4fs.c`<br>
`system/extras/ext4_utils/make_ext4fs.c:537:15: warning: comparison of integers of different signs: 'int' and 'u32' (aka 'unsigned int') [-Wsign-compare]`<br>
`        for(i = 0; i < aux_info.groups; i++) {`<br>
`                   ~ ^ ~~~~~~~~~~~~~~~`<br>
`system/extras/ext4_utils/make_ext4fs.c:631:22: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`                                if (min_bg_bound >= start_block - bg_first_block ||`<br>
`                                    ~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~`<br>
`system/extras/ext4_utils/make_ext4fs.c:632:19: warning: comparison of integers of different signs: 'int' and 'unsigned int' [-Wsign-compare]`<br>
`                                        max_bg_bound <= end_block - bg_first_block) {`<br>
`                                        ~~~~~~~~~~~~ ^  ~~~~~~~~~~~~~~~~~~~~~~~~~~`<br>
`system/extras/ext4_utils/make_ext4fs.c:650:16: warning: comparison of integers of different signs: 'int' and 'u32' (aka 'unsigned int') [-Wsign-compare]`<br>
`        for (i = 0; i < aux_info.groups; i++) {`<br>
`                    ~ ^ ~~~~~~~~~~~~~~~`<br>
`4 warnings generated.`<br>
`[ 71% 4197/5859] target  C: libext4_utils_static <= system/extras/ext4_utils/allocate.c`<br>
`system/extras/ext4_utils/allocate.c:362:17: warning: comparison of integers of different signs: 'unsigned int' and 'int' [-Wsign-compare]`<br>
`                for (j = 1; j < bgs[i].chunk_count; j++) {`<br>
`                            ~ ^ ~~~~~~~~~~~~~~~~~~`<br>
`1 warning generated.`<br>
`[ 72% 4223/5859] target  C: libdrm <= external/libdrm/xf86drm.c`<br>
`external/libdrm/xf86drm.c:895:36: warning: unused parameter 'fd' [-Wunused-parameter]`<br>
`drmVersionPtr drmGetLibVersion(int fd)`<br>
`                                   ^`<br>
`external/libdrm/xf86drm.c:1094:19: warning: implicit conversion from enumeration type 'drmMapType' to different enumeration type 'enum drm_map_type' [-Wenum-conversion]`<br>
`    map.type    = type;`<br>
`                ~ ^~~~`<br>
`external/libdrm/xf86drm.c:1095:19: warning: implicit conversion from enumeration type 'drmMapFlags' to different enumeration type 'enum drm_map_flags' [-Wenum-conversion]`<br>
`    map.flags   = flags;`<br>
`                ~ ^~~~~`<br>
`external/libdrm/xf86drm.c:1419:36: warning: implicit conversion from enumeration type 'drmDMAFlags' to different enumeration type 'enum drm_dma_flags' [-Wenum-conversion]`<br>
`    dma.flags           = request->flags;`<br>
`                        ~ ~~~~~~~~~^~~~~`<br>
`external/libdrm/xf86drm.c:2309:19: warning: implicit conversion from enumeration type 'enum drm_map_type' to different enumeration type 'drmMapType' [-Wenum-conversion]`<br>
`    *type   = map.type;`<br>
`            ~ ~~~~^~~~`<br>
`external/libdrm/xf86drm.c:2310:19: warning: implicit conversion from enumeration type 'enum drm_map_flags' to different enumeration type 'drmMapFlags' [-Wenum-conversion]`<br>
`    *flags  = map.flags;`<br>
`            ~ ~~~~^~~~~`<br>
`external/libdrm/xf86drm.c:2614:23: warning: unused parameter 'unused' [-Wunused-parameter]`<br>
`int drmOpenOnce(void *unused,`<br>
`                      ^`<br>
`7 warnings generated.`<br>
`[ 72% 4236/5859] target  C: libdrm <= external/libdrm/xf86drmMode.c`<br>
`external/libdrm/xf86drmMode.c:1342:38: warning: missing field 'count_objs' initializer [-Wmissing-field-initializers]`<br>
`        struct drm_mode_atomic atomic = { 0 };`<br>
`                                            ^`<br>
`1 warning generated.`<br>
`[ 74% 4337/5859] Target buildinfo: /home/stalker/hadk/out/tar...oduct/mido/obj/ETC/system_build_prop_intermediates/build.prop`<br>
`Target buildinfo from: device/xiaomi/mido/system.prop`<br>
`[ 74% 4391/5859] host C: libsepol <= external/selinux/libsepol/cil/src/cil_mem.c`<br>
`external/selinux/libsepol/cil/src/cil_mem.c:109:7: warning: implicit declaration of function 'vasprintf' is invalid in C99 [-Wimplicit-function-declaration]`<br>
`        rc = vasprintf(strp, fmt, ap);`<br>
`             ^`<br>
`1 warning generated.`<br>
`[ 75% 4409/5859] host C: libsepol <= /home/stalker/hadk/out/h...j/STATIC_LIBRARIES/libsepol_intermediates/cil/src/cil_lexer.c`<br>
`/home/stalker/hadk/out/host/linux-x86/obj/STATIC_LIBRARIES/libsepol_intermediates/cil/src/cil_lexer.c:1593:1: warning: function 'yy_fatal_error' could be declared with attribute 'noreturn' [-Wmissing-noreturn]`<br>
`{`<br>
`^`<br>
`1 warning generated.`<br>
`[ 75% 4452/5859] target  C: libext2_uuid_static <= external/e2fsprogs/lib/uuid/gen_uuid.c`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:224:39: warning: unused parameter 'node_id' [-Wunused-parameter]`<br>
`static int get_node_id(unsigned char *node_id)`<br>
`                                      ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:36: warning: unused parameter 'op' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                   ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:47: warning: unused parameter 'out' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                              ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:480:57: warning: unused parameter 'num' [-Wunused-parameter]`<br>
`static int get_uuid_via_daemon(int op, uuid_t out, int *num)`<br>
`                                                        ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:423:16: warning: unused function 'read_all' [-Wunused-function]`<br>
`static ssize_t read_all(int fd, char *buf, size_t count)`<br>
`               ^`<br>
`external/e2fsprogs/lib/uuid/gen_uuid.c:450:13: warning: unused function 'close_all_fds' [-Wunused-function]`<br>
`static void close_all_fds(void)`<br>
`            ^`<br>
`6 warnings generated.`<br>
`[ 80% 4696/5859] build /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/sepolicy`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  loading policy configuration from /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/policy.conf`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  policy configuration loaded`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  writing binary representation (version 30) to /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/sepolicy.tmp`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  loading policy configuration from /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates/policy.conf.dontaudit`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  policy configuration loaded`<br>
`/home/stalker/hadk/out/host/linux-x86/bin/checkpolicy:  writing binary representation (version 30) to /home/stalker/hadk/out/target/product/mido/obj/ETC/sepolicy_intermediates//sepolicy.dontaudit`<br>
`[ 91% 5345/5859] target  C: libclearsilverregex <= external/busybox/android/regex/bb_regex.c`<br>
`external/busybox/android/regex/bb_regex.c:5476:20: warning: unused parameter 'preg' [-Wunused-parameter]`<br>
`    const regex_t *preg;`<br>
`                   ^`<br>
`1 warning generated.`<br>
`[ 92% 5417/5859] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-boot_intermediates/init`<br>
`Fixing mount-points for device mido`<br>
`[ 92% 5417/5859] build /home/stalker/hadk/out/target/product/mido/obj/ROOT/hybris-recovery_intermediates/init`<br>
`Fixing mount-points for device mido`<br>
`[ 92% 5435/5859] -e Prepare config for busybox binary`<br>
`find: `/home/stalker/hadk/out/target/product/mido/obj/EXECUTABLES/busybox_intermediates': No such file or directory`<br>
`make: Entering directory `/home/stalker/hadk/external/busybox'`<br>
`  Using /home/stalker/hadk/external/busybox as source for busybox`<br>
`  GEN     /home/stalker/hadk/out/target/product/mido/obj/busybox/full/Makefile`<br>
`  GEN     include/applets.h`<br>
`  GEN     include/usage.h`<br>
`  GEN     applets/Kbuild`<br>
`  GEN     e2fsprogs/Kbuild`<br>
`  GEN     e2fsprogs/Config.in`<br>
`  GEN     e2fsprogs/old_e2fsprogs/Kbuild`<br>
`  GEN     e2fsprogs/old_e2fsprogs/Config.in`<br>
`  GEN     e2fsprogs/old_e2fsprogs/e2p/Kbuild`<br>
`  GEN     e2fsprogs/old_e2fsprogs/uuid/Kbuild`<br>
`  GEN     e2fsprogs/old_e2fsprogs/ext2fs/Kbuild`<br>
`  GEN     e2fsprogs/old_e2fsprogs/blkid/Kbuild`<br>
`  GEN     scripts/Kbuild`<br>
`  GEN     debianutils/Kbuild`<br>
`  GEN     debianutils/Config.in`<br>
`  GEN     procps/Kbuild`<br>
`  GEN     procps/Config.in`<br>
`  GEN     sysklogd/Kbuild`<br>
`  GEN     sysklogd/Config.in`<br>
`  GEN     editors/Kbuild`<br>
`  GEN     editors/Config.in`<br>
`  GEN     mailutils/Kbuild`<br>
`  GEN     mailutils/Config.in`<br>
`  GEN     shell/Kbuild`<br>
`  GEN     shell/Config.in`<br>
`  GEN     libbb/Kbuild`<br>
`  GEN     libbb/Config.in`<br>
`  GEN     modutils/Kbuild`<br>
`  GEN     modutils/Config.in`<br>
`  GEN     runit/Kbuild`<br>
`  GEN     runit/Config.in`<br>
`  GEN     coreutils/Kbuild`<br>
`  GEN     coreutils/Config.in`<br>
`  GEN     coreutils/libcoreutils/Kbuild`<br>
`  GEN     miscutils/Kbuild`<br>
`  GEN     miscutils/Config.in`<br>
`  GEN     loginutils/Kbuild`<br>
`  GEN     loginutils/Config.in`<br>
`  GEN     printutils/Kbuild`<br>
`  GEN     printutils/Config.in`<br>
`  GEN     findutils/Kbuild`<br>
`  GEN     findutils/Config.in`<br>
`  GEN     util-linux/Kbuild`<br>
`  GEN     util-linux/Config.in`<br>
`  GEN     util-linux/volume_id/Kbuild`<br>
`  GEN     util-linux/volume_id/Config.in`<br>
`  GEN     console-tools/Kbuild`<br>
`  GEN     console-tools/Config.in`<br>
`  GEN     init/Kbuild`<br>
`  GEN     init/Config.in`<br>
`  GEN     libpwdgrp/Kbuild`<br>
`  GEN     selinux/Kbuild`<br>
`  GEN     selinux/Config.in`<br>
`  GEN     networking/Kbuild`<br>
`  GEN     networking/Config.in`<br>
`  GEN     networking/udhcp/Kbuild`<br>
`  GEN     networking/udhcp/Config.in`<br>
`  GEN     networking/libiproute/Kbuild`<br>
`  GEN     archival/Kbuild`<br>
`  GEN     archival/Config.in`<br>
`  GEN     archival/libarchive/Kbuild`<br>
`  HOSTCC  scripts/basic/fixdep`<br>
`  HOSTCC  scripts/basic/split-include`<br>
`/home/stalker/hadk/external/busybox/scripts/basic/split-include.c: In function 'main':`<br>
`/home/stalker/hadk/external/busybox/scripts/basic/split-include.c:134:11: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]`<br>
`      fgets(old_line, buffer_size, fp_target);`<br>
`           ^`<br>
`  HOSTCC  scripts/basic/docproc`<br>
`  GEN     /home/stalker/hadk/out/target/product/mido/obj/busybox/full/Makefile`<br>
`  HOSTCC  scripts/kconfig/conf.o`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c: In function 'conf_askvalue':`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c:106:8: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]`<br>
`   fgets(line, 128, stdin);`<br>
`        ^`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c: In function 'conf_choice':`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/conf.c:354:9: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]`<br>
`    fgets(line, 128, stdin);`<br>
`         ^`<br>
`  HOSTCC  scripts/kconfig/kxgettext.o`<br>
`  HOSTCC  scripts/kconfig/mconf.o`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c: In function 'show_textbox':`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c:847:7: warning: ignoring return value of 'write', declared with attribute warn_unused_result [-Wunused-result]`<br>
`  write(fd, text, strlen(text));`<br>
`       ^`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c: In function 'exec_conf':`<br>
`/home/stalker/hadk/external/busybox/scripts/kconfig/mconf.c:481:6: warning: ignoring return value of 'pipe', declared with attribute warn_unused_result [-Wunused-result]`<br>
`  pipe(pipefd);`<br>
`      ^`<br>
`  SHIPPED scripts/kconfig/zconf.tab.c`<br>
`  SHIPPED scripts/kconfig/lex.zconf.c`<br>
`  SHIPPED scripts/kconfig/zconf.hash.c`<br>
`  HOSTCC  scripts/kconfig/zconf.tab.o`<br>
`  HOSTLD  scripts/kconfig/conf`<br>
`scripts/kconfig/conf -s Config.in`<br>
`#`<br>
`# using defaults found in .config`<br>
`#`<br>
`  SPLIT   include/autoconf.h -> include/config/*`<br>
`  GEN     include/bbconfigopts.h`<br>
`  HOSTCC  applets/usage`<br>
`/home/stalker/hadk/external/busybox/applets/usage.c: In function 'main':`<br>
`/home/stalker/hadk/external/busybox/applets/usage.c:52:8: warning: ignoring return value of 'write', declared with attribute warn_unused_result [-Wunused-result]`<br>
`   write(STDOUT_FILENO, usage_array[i].usage, strlen(usage_array[i].usage) + 1);`<br>
`        ^`<br>
`  GEN     include/usage_compressed.h`<br>
`  HOSTCC  applets/applet_tables`<br>
`/home/stalker/hadk/external/busybox/applets/applet_tables.c: In function 'main':`<br>
`/home/stalker/hadk/external/busybox/applets/applet_tables.c:144:9: warning: ignoring return value of 'fgets', declared with attribute warn_unused_result [-Wunused-result]`<br>
`    fgets(line_old, sizeof(line_old), fp);`<br>
`         ^`<br>
`  GEN     include/applet_tables.h`<br>
`  CC      applets/applets.o`<br>
`  LD      applets/built-in.o`<br>
`  HOSTCC  applets/usage_pod`<br>
`make: Leaving directory `/home/stalker/hadk/external/busybox'`<br>
`[ 93% 5473/5859] target  C: static_busybox <= external/busybox/archival/unzip.c`<br>
`external/busybox/archival/unzip.c: In function 'unzip_extract':`<br>
`external/busybox/archival/unzip.c:295:35: warning: comparison of promoted ~unsigned with unsigned [-Wsign-compare]`<br>
`   if (zip_header->formatted.crc32 != (aux.crc32 ^ 0xffffffffL)) {`<br>
`                                   ^`<br>
`[ 95% 5591/5859] target  C: static_busybox <= external/busybox/libbb/change_identity.c`<br>
`external/busybox/libbb/change_identity.c: In function 'change_identity':`<br>
`external/busybox/libbb/change_identity.c:38:2: warning: 'endgrent' is deprecated (declared at bionic/libc/include/grp.h:59): endgrent is meaningless on Android [-Wdeprecated-declarations]`<br>
`  endgrent(); /* helps to close a fd used internally by libc */`<br>
`  ^`<br>
`[ 97% 5737/5859] target  C: static_busybox <= external/busybox/modutils/modutils.c`<br>
`external/busybox/modutils/modutils.c: In function 'try_to_mmap_module':`<br>
`external/busybox/modutils/modutils.c:116:17: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (st.st_size <= *image_size_p) {`<br>
`                 ^`<br>
`[ 98% 5765/5859] target  C: static_busybox <= external/busybox/networking/libiproute/libnetlink.c`<br>
`external/busybox/networking/libiproute/libnetlink.c: In function 'rta_addattr32':`<br>
`external/busybox/networking/libiproute/libnetlink.c:362:36: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (RTA_ALIGN(rta->rta_len) + len > maxlen) {`<br>
`                                    ^`<br>
`external/busybox/networking/libiproute/libnetlink.c: In function 'rta_addattr_l':`<br>
`external/busybox/networking/libiproute/libnetlink.c:378:36: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (RTA_ALIGN(rta->rta_len) + len > maxlen) {`<br>
`                                    ^`<br>
`[ 99% 5840/5859] target  C: static_busybox <= external/busybox/networking/udhcp/d6_dhcpc.c`<br>
`external/busybox/networking/udhcp/d6_dhcpc.c: In function 'd6_store_blob':`<br>
`external/busybox/networking/udhcp/d6_dhcpc.c:127:13: warning: pointer of type 'void *' used in arithmetic [-Wpointer-arith]`<br>
`  return dst + len;`<br>
`             ^`<br>
`external/busybox/networking/udhcp/d6_dhcpc.c: In function 'd6_recv_raw_packet':`<br>
`external/busybox/networking/udhcp/d6_dhcpc.c:605:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (bytes < sizeof(packet.ip6) + ntohs(packet.ip6.ip6_plen)) {`<br>
`            ^`<br>
`[ 99% 5841/5859] target  C: static_busybox <= external/busybox/networking/udhcp/domain_codec.c`<br>
`external/busybox/networking/udhcp/domain_codec.c: In function 'dname_dec':`<br>
`external/busybox/networking/udhcp/domain_codec.c:50:17: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`   while (crtpos < clen) {`<br>
`                 ^`<br>
`external/busybox/networking/udhcp/domain_codec.c:55:20: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`     if (crtpos + 2 > clen) /* no offset to jump to? abort */`<br>
`                    ^`<br>
`external/busybox/networking/udhcp/domain_codec.c:63:25: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`     if (crtpos + *c + 1 > clen) /* label too long? abort */`<br>
`                         ^`<br>
`external/busybox/networking/udhcp/domain_codec.c:33:8: warning: 'ret' is used uninitialized in this function [-Wuninitialized]`<br>
`  char *ret = ret; /* for compiler */`<br>
`        ^`<br>
`[ 99% 5845/5859] target  C: static_busybox <= external/busybox/networking/udhcp/leases.c`<br>
`external/busybox/networking/udhcp/leases.c: In function 'add_lease':`<br>
`external/busybox/networking/udhcp/leases.c:65:21: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`    if (hostname_len > sizeof(oldest->hostname))`<br>
`                     ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/signalpipe.c`<br>
`In file included from bionic/libc/include/unistd.h:35:0,`<br>
`                 from external/busybox/include/platform.h:299,`<br>
`                 from external/busybox/include/libbb.h:13,`<br>
`                 from external/busybox/networking/udhcp/common.h:11,`<br>
`                 from external/busybox/networking/udhcp/signalpipe.c:21:`<br>
`external/busybox/networking/udhcp/signalpipe.c: In function 'udhcp_sp_read':`<br>
`external/busybox/networking/udhcp/signalpipe.c:70:32: warning: passing argument 2 of '__FD_ISSET_chk' discards 'const' qualifier from pointer target type`<br>
`  if (!FD_ISSET(signal_pipe.rd, rfds))`<br>
`                                ^`<br>
`bionic/libc/include/sys/select.h:62:13: note: expected 'struct fd_set *' but argument is of type 'const struct fd_set *'`<br>
` extern int  __FD_ISSET_chk(int, fd_set*, size_t);`<br>
`             ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/util-linux/mdev.c`<br>
`external/busybox/util-linux/mdev.c: In function 'make_device':`<br>
`external/busybox/util-linux/mdev.c:772:8: warning: 'aliaslink' may be used uninitialized in this function [-Wmaybe-uninitialized]`<br>
`     if (aliaslink == '>') {`<br>
`        ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/packet.c`<br>
`external/busybox/networking/udhcp/packet.c: In function 'udhcp_recv_kernel_packet':`<br>
`external/busybox/networking/udhcp/packet.c:92:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (bytes < offsetof(struct dhcp_packet, options)`<br>
`            ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/d6_packet.c`<br>
`external/busybox/networking/udhcp/d6_packet.c: In function 'd6_recv_kernel_packet':`<br>
`external/busybox/networking/udhcp/d6_packet.c:42:12: warning: comparison between signed and unsigned integer expressions [-Wsign-compare]`<br>
`  if (bytes < offsetof(struct d6_packet, d6_options)) {`<br>
`            ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/dhcpd.c`<br>
`external/busybox/networking/udhcp/dhcpd.c: In function 'udhcpd_main':`<br>
`external/busybox/networking/udhcp/dhcpd.c:306:8: warning: 'str_I' is used uninitialized in this function [-Wuninitialized]`<br>
`  char *str_I = str_I;`<br>
`        ^`<br>
`external/busybox/networking/udhcp/dhcpd.c:599:14: warning: 'requested_nip' may be used uninitialized in this function [-Wmaybe-uninitialized]`<br>
`    if (lease && requested_nip == lease->lease_nip) {`<br>
`              ^`<br>
`[ 99% 5846/5859] target  C: static_busybox <= external/busybox/networking/udhcp/dhcpc.c`<br>
`external/busybox/networking/udhcp/dhcpc.c: In function 'udhcpc_main':`<br>
`external/busybox/networking/udhcp/dhcpc.c:1244:11: warning: 'xid' is used uninitialized in this function [-Wuninitialized]`<br>
`  uint32_t xid = xid; /* for compiler */`<br>
`           ^`<br>
`external/busybox/networking/udhcp/dhcpc.c:1582:4: warning: 'server_addr' may be used uninitialized in this function [-Wmaybe-uninitialized]`<br>
`    perform_release(server_addr, requested_ip);`<br>
`    ^`<br>
`[ 99% 5853/5859] Making initramfs : /home/stalker/hadk/out/ta...uct/mido/obj/ROOT/hybris-boot_intermediates/boot-initramfs.gz`<br>
`2953 blocks`<br>
`[ 99% 5853/5859] Making initramfs : /home/stalker/hadk/out/ta.../obj/ROOT/hybris-recovery_intermediates/recovery-initramfs.gz`<br>
`2953 blocks`<br>
`[ 99% 5853/5859] Building Kernel`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  GEN     ./Makefile`<br>
`scripts/kconfig/conf --silentoldconfig Kconfig`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  CHK     include/config/kernel.release`<br>
`  GEN     ./Makefile`<br>
`  CHK     include/generated/uapi/linux/version.h`<br>
`  HOSTCC  scripts/basic/bin2c`<br>
`  HOSTCC  scripts/kallsyms`<br>
`  HOSTCC  scripts/conmakehash`<br>
`  HOSTCC  scripts/recordmcount`<br>
`  CC      scripts/mod/empty.o`<br>
`  HOSTCC  scripts/dtc/dtc.o`<br>
`  HOSTCC  scripts/mod/mk_elfconfig`<br>
`  HOSTCC  scripts/sortextable`<br>
`  CC      scripts/mod/devicetable-offsets.s`<br>
`  HOSTCC  scripts/dtc/flattree.o`<br>
`  HOSTCC  scripts/dtc/fstree.o`<br>
`  HOSTCC  scripts/dtc/data.o`<br>
`  HOSTCC  scripts/dtc/livetree.o`<br>
`  HOSTCC  scripts/asn1_compiler`<br>
`  HOSTCC  scripts/selinux/genheaders/genheaders`<br>
`  HOSTCC  scripts/selinux/mdp/mdp`<br>
`  HOSTCC  scripts/dtc/treesource.o`<br>
`  GEN     scripts/mod/devicetable-offsets.h`<br>
`  HOSTCC  scripts/dtc/srcpos.o`<br>
`  HOSTCC  scripts/dtc/checks.o`<br>
`  HOSTCC  scripts/dtc/util.o`<br>
`  SHIPPED scripts/dtc/dtc-lexer.lex.c`<br>
`  SHIPPED scripts/dtc/dtc-parser.tab.h`<br>
`  SHIPPED scripts/dtc/dtc-parser.tab.c`<br>
`  MKELF   scripts/mod/elfconfig.h`<br>
`  HOSTCC  scripts/dtc/dtc-lexer.lex.o`<br>
`  HOSTCC  scripts/mod/modpost.o`<br>
`  HOSTCC  scripts/mod/file2alias.o`<br>
`  HOSTCC  scripts/mod/sumversion.o`<br>
`  HOSTCC  scripts/dtc/dtc-parser.tab.o`<br>
`  UPD     include/config/kernel.release`<br>
`  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel`<br>
`  CHK     include/generated/utsrelease.h`<br>
`  UPD     include/generated/utsrelease.h`<br>
`  HOSTLD  scripts/dtc/dtc`<br>
`  CC      kernel/bounds.s`<br>
`  GEN     include/generated/bounds.h`<br>
`  CC      arch/arm64/kernel/asm-offsets.s`<br>
`  HOSTLD  scripts/mod/modpost`<br>
`  GEN     include/generated/asm-offsets.h`<br>
`  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh`<br>
`  CC      init/main.o`<br>
`  CHK     include/generated/compile.h`<br>
`  CC      init/do_mounts.o`<br>
`  CC      init/do_mounts_rd.o`<br>
`  CC      init/do_mounts_initrd.o`<br>
`  CC      init/do_mounts_dm.o`<br>
`  CC      init/noinitramfs.o`<br>
`  CC      init/initramfs.o`<br>
`  CC      init/calibrate.o`<br>
`  CC      init/init_task.o`<br>
`  HOSTCC  usr/gen_init_cpio`<br>
`  UPD     include/generated/compile.h`<br>
`  CC      init/version.o`<br>
`  AS      arch/arm64/kernel/head.o`<br>
`  LDS     arch/arm64/kernel/vmlinux.lds`<br>
`  CC      arch/arm64/mm/dma-mapping.o`<br>
`  CC      arch/arm64/mm/extable.o`<br>
`  CC      arch/arm64/mm/fault.o`<br>
`  CC      arch/arm64/mm/init.o`<br>
`  AS      arch/arm64/mm/cache.o`<br>
`  GEN     usr/initramfs_data.cpio.gz`<br>
`  CC      arch/arm64/mm/copypage.o`<br>
`  CC      arch/arm64/mm/flush.o`<br>
`  LDS     arch/arm64/kernel/vdso/vdso.lds`<br>
`  CC      arch/arm64/mm/mmap.o`<br>
`  CC      arch/arm64/mm/ioremap.o`<br>
`  LD      arch/arm64/net/built-in.o`<br>
`  CC      arch/arm64/mm/pgd.o`<br>
`  AS      usr/initramfs_data.o`<br>
`  CC      arch/arm64/mm/mmu.o`<br>
`  CC      arch/arm64/mm/context.o`<br>
`  AS      arch/arm64/mm/proc.o`<br>
`  CC      arch/arm64/mm/pageattr.o`<br>
`  LD      usr/built-in.o`<br>
`  VDSOA   arch/arm64/kernel/vdso/gettimeofday.o`<br>
`  VDSOA   arch/arm64/kernel/vdso/note.o`<br>
`  VDSOA   arch/arm64/kernel/vdso/sigreturn.o`<br>
`  LD      init/mounts.o`<br>
`/home/stalker/hadk/kernel/xiaomi/msm8953/kernel/Makefile:133: *** No X.509 certificates found ***`<br>
`  VDSOL   arch/arm64/kernel/vdso/vdso.so.dbg`<br>
`  OBJCOPY arch/arm64/kernel/vdso/vdso.so`<br>
`  VDSOSYM arch/arm64/kernel/vdso/vdso-offsets.h`<br>
`  CC      arch/arm64/crypto/sha1-ce-glue.o`<br>
`  AS      arch/arm64/kernel/vdso/vdso.o`<br>
`  AS      arch/arm64/crypto/sha1-ce-core.o`<br>
`  CC      arch/arm64/crypto/sha2-ce-glue.o`<br>
`  CC      arch/arm64/crypto/ghash-ce-glue.o`<br>
`  AS      arch/arm64/crypto/ghash-ce-core.o`<br>
`  AS      arch/arm64/crypto/sha2-ce-core.o`<br>
`  CC      arch/arm64/crypto/aes-ce-ccm-glue.o`<br>
`  CC      arch/arm64/crypto/aes-ce-cipher.o`<br>
`  AS      arch/arm64/crypto/aes-ce-ccm-core.o`<br>
`  CC      arch/arm64/crypto/aes-glue-ce.o`<br>
`  LD      arch/arm64/kernel/vdso/built-in.o`<br>
`  AS      arch/arm64/crypto/aes-ce.o`<br>
`  CC      arch/arm64/crypto/aes-glue-neon.o`<br>
`  LD      init/built-in.o`<br>
`  CC      arch/arm64/kernel/cputable.o`<br>
`  CC      arch/arm64/kernel/debug-monitors.o`<br>
`  AS      arch/arm64/crypto/aes-neon.o`<br>
`  AS      arch/arm64/kernel/entry.o`<br>
`  CC      arch/arm64/kernel/irq.o`<br>
`  LD      arch/arm64/crypto/sha1-ce.o`<br>
`  LD      arch/arm64/crypto/ghash-ce.o`<br>
`  LD      arch/arm64/crypto/sha2-ce.o`<br>
`  CC      arch/arm64/kernel/fpsimd.o`<br>
`  AS      arch/arm64/kernel/entry-fpsimd.o`<br>
`  CC      arch/arm64/kernel/process.o`<br>
`  CC      arch/arm64/kernel/ptrace.o`<br>
`  CC      arch/arm64/kernel/setup.o`<br>
`  CC      arch/arm64/kernel/signal.o`<br>
`  LD      arch/arm64/crypto/aes-neon-blk.o`<br>
`  LD      arch/arm64/crypto/aes-ce-ccm.o`<br>
`  CC      arch/arm64/kernel/sys.o`<br>
`  CC      arch/arm64/kernel/stacktrace.o`<br>
`  LD      arch/arm64/mm/built-in.o`<br>
`  CC      arch/arm64/kernel/time.o`<br>
`  CC      arch/arm64/kernel/traps.o`<br>
`  CC      arch/arm64/kernel/io.o`<br>
`  CC      arch/arm64/kernel/vdso.o`<br>
`  AS      arch/arm64/kernel/hyp-stub.o`<br>
`  CC      arch/arm64/kernel/psci.o`<br>
`  CC      kernel/fork.o`<br>
`  AS      arch/arm64/kernel/psci-call.o`<br>
`  CC      ipc/compat.o`<br>
`  CC      ipc/util.o`<br>
`  CC      arch/arm64/kernel/cpu_ops.o`<br>
`  CC      arch/arm64/kernel/insn.o`<br>
`  CC      arch/arm64/kernel/return_address.o`<br>
`  CC      arch/arm64/kernel/cpuinfo.o`<br>
`  CC      arch/arm64/kernel/cpu_errata.o`<br>
`  CC      kernel/exec_domain.o`<br>
`  CC      mm/filemap.o`<br>
`  CC      arch/arm64/kernel/cpufeature.o`<br>
`  CC      kernel/panic.o`<br>
`  CC      arch/arm64/kernel/alternative.o`<br>
`  CC      mm/mempool.o`<br>
`  CC      ipc/msgutil.o`<br>
`  AS      arch/arm64/kernel/sys32.o`<br>
`  AS      arch/arm64/kernel/kuser32.o`<br>
`  CC      arch/arm64/kernel/signal32.o`<br>
`  CC      arch/arm64/kernel/sys_compat.o`<br>
`  CC      ipc/msg.o`<br>
`  CC      kernel/cpu.o`<br>
`  CC      arch/arm64/kernel/../../arm/kernel/opcodes.o`<br>
`  LD      arch/arm64/crypto/aes-ce-blk.o`<br>
`  CC      arch/arm64/kernel/ftrace.o`<br>
`  CC      kernel/exit.o`<br>
`  LD      arch/arm64/crypto/built-in.o`<br>
`  CC      ipc/sem.o`<br>
`  AS      arch/arm64/kernel/entry-ftrace.o`<br>
`  CC      ipc/shm.o`<br>
`  CC      security/integrity/iint.o`<br>
`  CC      ipc/ipcns_notifier.o`<br>
`  CC      arch/arm64/kernel/smp.o`<br>
`  CC      security/integrity/integrity_audit.o`<br>
`  CC      arch/arm64/kernel/smp_spin_table.o`<br>
`  CC      security/keys/gc.o`<br>
`  CC      arch/arm64/kernel/topology.o`<br>
`  CC      arch/arm64/kernel/perf_regs.o`<br>
`  CC      arch/arm64/kernel/perf_event.o`<br>
`  CC      arch/arm64/kernel/perf_debug.o`<br>
`  CC      kernel/softirq.o`<br>
`  CC      kernel/resource.o`<br>
`  CC      kernel/sysctl.o`<br>
`  CC      security/keys/key.o`<br>
`  CC      fs/open.o`<br>
`  CC      arch/arm64/kernel/perf_trace_counters.o`<br>
`  CC      fs/read_write.o`<br>
`  CC      fs/file_table.o`<br>
`  CC      security/keys/keyring.o`<br>
`  CC      kernel/sysctl_binary.o`<br>
`  CC      arch/arm64/kernel/perf_trace_user.o`<br>
`  CC      kernel/capability.o`<br>
`  AS      arch/arm64/kernel/sleep.o`<br>
`  LD      security/pfe/built-in.o`<br>
`  CC      arch/arm64/kernel/suspend.o`<br>
`  CC      security/keys/keyctl.o`<br>
`  CC      mm/oom_kill.o`<br>
`  CC      mm/maccess.o`<br>
`  CC      mm/page_alloc.o`<br>
`  CC      mm/page-writeback.o`<br>
`  CC      arch/arm64/kernel/cpuidle.o`<br>
`  CC      mm/readahead.o`<br>
`  CC      ipc/syscall.o`<br>
`  CC      mm/swap.o`<br>
`  CC      mm/truncate.o`<br>
`  CC      crypto/api.o`<br>
`  CC      ipc/ipc_sysctl.o`<br>
`  CC      kernel/ptrace.o`<br>
`  CC      mm/vmscan.o`<br>
`  CC      kernel/user.o`<br>
`  LD      security/integrity/integrity.o`<br>
`  CC      arch/arm64/kernel/efi.o`<br>
`  CC      kernel/signal.o`<br>
`  LD      security/integrity/built-in.o`<br>
`  CC      block/bio.o`<br>
`  CC      crypto/cipher.o`<br>
`  CC      arch/arm64/kernel/efi-stub.o`<br>
`  CC      ipc/namespace.o`<br>
`  CC      mm/shmem.o`<br>
`  AS      arch/arm64/kernel/efi-entry.o`<br>
`  CC      mm/util.o`<br>
`  CC      kernel/sys.o`<br>
`  CC      crypto/compress.o`<br>
`  CC      block/elevator.o`<br>
`  CC      mm/mmzone.o`<br>
`  CC      arch/arm64/kernel/pci.o`<br>
`  CC      arch/arm64/kernel/armv8_deprecated.o`<br>
`  CC      crypto/memneq.o`<br>
`  CC      mm/vmstat.o`<br>
`  CC      crypto/crypto_wq.o`<br>
`  CC      block/blk-core.o`<br>
`  CC      kernel/kmod.o`<br>
`  GEN     security/selinux/flask.h security/selinux/av_permissions.h`<br>
`  CC      security/selinux/avc.o`<br>
`  CC      security/selinux/hooks.o`<br>
`  CC      security/selinux/selinuxfs.o`<br>
`  CC      crypto/algapi.o`<br>
`  CC      security/selinux/netlink.o`<br>
`  CC      security/selinux/nlmsgtab.o`<br>
`  CC      security/selinux/netif.o`<br>
`  CC      crypto/scatterwalk.o`<br>
`  CC      security/selinux/netnode.o`<br>
`  CC      crypto/proc.o`<br>
`  LD      ipc/built-in.o`<br>
`  CC      crypto/aead.o`<br>
`  CC      mm/backing-dev.o`<br>
`  CC      block/blk-tag.o`<br>
`  CC      sound/sound_core.o`<br>
`  CC      security/selinux/netport.o`<br>
`  CC      block/blk-sysfs.o`<br>
`  CC      crypto/ablkcipher.o`<br>
`  LD      sound/arm/built-in.o`<br>
`  LD      sound/atmel/built-in.o`<br>
`  CC      crypto/blkcipher.o`<br>
`  CC      crypto/chainiv.o`<br>
`  CC      crypto/eseqiv.o`<br>
`  CC      security/selinux/exports.o`<br>
`  CC      security/selinux/ss/ebitmap.o`<br>
`  LD      firmware/built-in.o`<br>
`  CC      block/blk-flush.o`<br>
`  CC      block/blk-settings.o`<br>
`  CC      block/blk-ioc.o`<br>
`  CC      crypto/seqiv.o`<br>
`  CC      block/blk-map.o`<br>
`  CC      security/keys/permission.o`<br>
`  CC      crypto/ahash.o`<br>
`  CC      security/selinux/ss/hashtab.o`<br>
`  CC      drivers/amba/bus.o`<br>
`  CC      security/selinux/ss/symtab.o`<br>
`  CC      crypto/shash.o`<br>
`  CC      sound/core/sound.o`<br>
`  CC      block/blk-exec.o`<br>
`  CC      mm/mm_init.o`<br>
`  CC      sound/core/init.o`<br>
`  CC      sound/core/memory.o`<br>
`  CC      security/selinux/ss/sidtab.o`<br>
`  CC      security/keys/process_keys.o`<br>
`  CC      crypto/pcompress.o`<br>
`  CC      security/selinux/ss/avtab.o`<br>
`  CC      fs/super.o`<br>
`  CC      block/blk-merge.o`<br>
`  CC      sound/core/info.o`<br>
`  CC      sound/core/control.o`<br>
`  CC      sound/core/misc.o`<br>
`  CC      security/keys/request_key.o`<br>
`  CC      fs/char_dev.o`<br>
`  CC      block/blk-softirq.o`<br>
`  CC      crypto/algboss.o`<br>
`  CC      security/selinux/ss/policydb.o`<br>
`  CC      kernel/workqueue.o`<br>
`  CC      security/keys/request_key_auth.o`<br>
`  CC      security/selinux/ss/services.o`<br>
`  LD      arch/arm64/kernel/built-in.o`<br>
`  CC      sound/core/device.o`<br>
`  LD      drivers/auxdisplay/built-in.o`<br>
`  CC      kernel/pid.o`<br>
`  CC      security/commoncap.o`<br>
`  CC      fs/stat.o`<br>
`  CC      security/yama/yama_lsm.o`<br>
`  CC      kernel/task_work.o`<br>
`  CC      block/blk-timeout.o`<br>
`  CC      security/selinux/ss/conditional.o`<br>
`  CC      block/blk-iopoll.o`<br>
`  CC      mm/mmu_context.o`<br>
`  CC      security/selinux/ss/mls.o`<br>
`  CC      fs/exec.o`<br>
`  CC      block/blk-lib.o`<br>
`  CC      security/selinux/ss/status.o`<br>
`  CC      security/min_addr.o`<br>
`  CC      security/keys/user_defined.o`<br>
`  CC      kernel/extable.o`<br>
`  LD      security/yama/yama.o`<br>
`  CC      block/blk-mq.o`<br>
`  LD      security/yama/built-in.o`<br>
`  CC      drivers/base/component.o`<br>
`  CC      drivers/base/core.o`<br>
`  CC      sound/core/jack.o`<br>
`  CC      fs/pipe.o`<br>
`  CC      security/keys/proc.o`<br>
`  CC      sound/core/hwdep.o`<br>
`  CC      kernel/params.o`<br>
`  CC      block/blk-mq-tag.o`<br>
`  LD      security/selinux/selinux.o`<br>
`  CC      block/blk-mq-sysfs.o`<br>
`  CC      sound/core/timer.o`<br>
`  LD      security/selinux/built-in.o`<br>
`  CC      drivers/base/bus.o`<br>
`  CC      fs/namei.o`<br>
`  CC      security/keys/sysctl.o`<br>
`  CC      drivers/base/dd.o`<br>
`  CC      sound/core/pcm.o`<br>
`  CC      block/blk-mq-cpu.o`<br>
`  CC      kernel/kthread.o`<br>
`  CC      security/keys/encrypted-keys/encrypted.o`<br>
`  CC      sound/core/pcm_native.o`<br>
`  CC      sound/core/pcm_lib.o`<br>
`  CC      crypto/testmgr.o`<br>
`  CC      crypto/cmac.o`<br>
`  CC      drivers/base/syscore.o`<br>
`  CC      sound/core/pcm_timer.o`<br>
`  CC      block/blk-mq-cpumap.o`<br>
`  CC      drivers/base/driver.o`<br>
`  CC      fs/fcntl.o`<br>
`  CC      fs/ioctl.o`<br>
`  CC      security/keys/encrypted-keys/ecryptfs_format.o`<br>
`  CC      sound/core/pcm_misc.o`<br>
`  CC      kernel/sys_ni.o`<br>
`  CC      crypto/hmac.o`<br>
`  CC      crypto/xcbc.o`<br>
`  CC      drivers/base/class.o`<br>
`  CC      net/socket.o`<br>
`  LD      drivers/amba/built-in.o`<br>
`  CC      block/ioctl.o`<br>
`  CC      security/security.o`<br>
`  CC      fs/readdir.o`<br>
`  CC      block/genhd.o`<br>
`  CC      net/802/p8022.o`<br>
`  CC      kernel/nsproxy.o`<br>
`  CC      mm/percpu.o`<br>
`  CC      kernel/notifier.o`<br>
`  LD      security/keys/encrypted-keys/encrypted-keys.o`<br>
`  CC      security/capability.o`<br>
`  LD      security/keys/encrypted-keys/built-in.o`<br>
`  CC      crypto/crypto_null.o`<br>
`  LD      security/keys/built-in.o`<br>
`  CC      crypto/md4.o`<br>
`  CC      fs/select.o`<br>
`  CC      sound/core/pcm_memory.o`<br>
`  CC      net/802/psnap.o`<br>
`  CC      block/scsi_ioctl.o`<br>
`  CC      security/inode.o`<br>
`  CC      security/lsm_audit.o`<br>
`  CC      mm/slab_common.o`<br>
`  CC      security/device_cgroup.o`<br>
`  CC      block/partition-generic.o`<br>
`  CC      kernel/ksysfs.o`<br>
`  CC      block/ioprio.o`<br>
`  CC      net/802/stp.o`<br>
`  CC      sound/core/memalloc.o`<br>
`  CC      crypto/md5.o`<br>
`  CC      crypto/sha1_generic.o`<br>
`  CC      net/bluetooth/af_bluetooth.o`<br>
`  CC      drivers/base/platform.o`<br>
`  CC      crypto/sha256_generic.o`<br>
`  LD      security/built-in.o`<br>
`  CC      kernel/cred.o`<br>
`  CC      mm/compaction.o`<br>
`  CC      block/partitions/check.o`<br>
`  CC      block/bounce.o`<br>
`  CC      sound/core/rawmidi.o`<br>
`  CC      crypto/sha512_generic.o`<br>
`  LD      net/802/built-in.o`<br>
`  CC      mm/vmacache.o`<br>
`  CC      mm/interval_tree.o`<br>
`  LD      sound/firewire/built-in.o`<br>
`  CC      mm/list_lru.o`<br>
`  CC      kernel/reboot.o`<br>
`  CC      kernel/async.o`<br>
`  LD      sound/drivers/mpu401/built-in.o`<br>
`  LD      arch/arm64/lib/built-in.o`<br>
`  AS      arch/arm64/lib/bitops.o`<br>
`  CC      block/partitions/msdos.o`<br>
`  CC      crypto/gf128mul.o`<br>
`  CC      block/partitions/efi.o`<br>
`  CC      sound/core/compress_offload.o`<br>
`  AS      arch/arm64/lib/clear_page.o`<br>
`  LD      sound/drivers/opl3/built-in.o`<br>
`  AS      arch/arm64/lib/clear_user.o`<br>
`  CC      mm/workingset.o`<br>
`  LD      sound/drivers/opl4/built-in.o`<br>
`  CC      kernel/range.o`<br>
`  AS      arch/arm64/lib/copy_from_user.o`<br>
`  CC      kernel/groups.o`<br>
`  CC      mm/iov_iter.o`<br>
`  LD      sound/drivers/pcsp/built-in.o`<br>
`  CC      crypto/ecb.o`<br>
`  LD      sound/core/snd.o`<br>
`  CC      crypto/cbc.o`<br>
`  LD      block/partitions/built-in.o`<br>
`  LD      sound/core/snd-hwdep.o`<br>
`  LD      sound/drivers/vx/built-in.o`<br>
`  CC      drivers/base/cpu.o`<br>
`  LD      sound/core/snd-timer.o`<br>
`  CC      block/bsg.o`<br>
`  LD      sound/core/snd-pcm.o`<br>
`  LD      sound/drivers/built-in.o`<br>
`  CC      drivers/base/firmware.o`<br>
`  CC      fs/dcache.o`<br>
`  LD      sound/core/snd-rawmidi.o`<br>
`  CC      drivers/base/init.o`<br>
`  CC      kernel/smpboot.o`<br>
`  CC      mm/debug.o`<br>
`  CC      net/bluetooth/hci_core.o`<br>
`  AS      arch/arm64/lib/copy_in_user.o`<br>
`  LD      sound/core/snd-compress.o`<br>
`  CC      mm/fremap.o`<br>
`  LD      sound/core/built-in.o`<br>
`  CC      crypto/cts.o`<br>
`  CC      drivers/base/map.o`<br>
`  CC      block/blk-cgroup.o`<br>
`  CC      kernel/bpf/core.o`<br>
`  CC      mm/gup.o`<br>
`  CC      net/bluetooth/hci_conn.o`<br>
`  CC      fs/inode.o`<br>
`  CC      mm/highmem.o`<br>
`  AS      arch/arm64/lib/copy_page.o`<br>
`  CC      fs/attr.o`<br>
`  CC      crypto/xts.o`<br>
`  LD      sound/i2c/other/built-in.o`<br>
`  CC      crypto/ctr.o`<br>
`  CC      block/noop-iosched.o`<br>
`  LD      sound/i2c/built-in.o`<br>
`  CC      block/deadline-iosched.o`<br>
`  AS      arch/arm64/lib/copy_to_user.o`<br>
`  LD      sound/mips/built-in.o`<br>
`  CC      block/cfq-iosched.o`<br>
`  CC      crypto/cryptd.o`<br>
`  LD      sound/parisc/built-in.o`<br>
`  CC      mm/memory.o`<br>
`  CC      arch/arm64/lib/delay.o`<br>
`  CC      crypto/des_generic.o`<br>
`  CC      block/compat_ioctl.o`<br>
`  LD      sound/isa/ad1816a/built-in.o`<br>
`  LD      sound/isa/ad1848/built-in.o`<br>
`  LD      sound/isa/cs423x/built-in.o`<br>
`  LD      block/built-in.o`<br>
`  CC      crypto/twofish_generic.o`<br>
`  LD      sound/isa/es1688/built-in.o`<br>
`  CC      net/bridge/br.o`<br>
`  CC      mm/mincore.o`<br>
`  LD      sound/isa/galaxy/built-in.o`<br>
`  CC      net/core/sock.o`<br>
`  LD      sound/isa/gus/built-in.o`<br>
`  CC      drivers/base/devres.o`<br>
`  CC      net/core/request_sock.o`<br>
`  LD      sound/isa/msnd/built-in.o`<br>
`  LD      sound/pci/ac97/built-in.o`<br>
`  CC      crypto/twofish_common.o`<br>
`  CC      crypto/aes_generic.o`<br>
`  LD      sound/isa/opti9xx/built-in.o`<br>
`  CC      mm/mlock.o`<br>
`  CC      drivers/base/attribute_container.o`<br>
`  LD      sound/pci/ali5451/built-in.o`<br>
`  LD      sound/isa/sb/built-in.o`<br>
`  CC      crypto/arc4.o`<br>
`  LD      sound/isa/wavefront/built-in.o`<br>
`  LD      sound/pci/asihpi/built-in.o`<br>
`  LD      sound/isa/wss/built-in.o`<br>
`  CC      crypto/deflate.o`<br>
`  LD      sound/isa/built-in.o`<br>
`  LD      sound/pci/au88x0/built-in.o`<br>
`  CC      drivers/base/transport_class.o`<br>
`  CC      crypto/crc32c_generic.o`<br>
`  CC      crypto/crc32.o`<br>
`  CC      mm/mmap.o`<br>
`  CC      fs/bad_inode.o`<br>
`  CC      crypto/authenc.o`<br>
`  LD      sound/pci/aw2/built-in.o`<br>
`  CC      crypto/authencesn.o`<br>
`  LD      sound/pci/ca0106/built-in.o`<br>
`  CC      crypto/lzo.o`<br>
`  CC      drivers/base/topology.o`<br>
`  CC      fs/file.o`<br>
`  CC      crypto/rng.o`<br>
`  LD      kernel/bpf/built-in.o`<br>
`  LD      sound/pci/cs46xx/built-in.o`<br>
`  LD      sound/pci/cs5535audio/built-in.o`<br>
`  CC      mm/mprotect.o`<br>
`  AS      arch/arm64/lib/memchr.o`<br>
`  AS      arch/arm64/lib/memcmp.o`<br>
`  LD      sound/pci/ctxfi/built-in.o`<br>
`  CC      crypto/krng.o`<br>
`  CC      crypto/ansi_cprng.o`<br>
`  CC      fs/filesystems.o`<br>
`  CC      crypto/xor.o`<br>
`  LD      sound/pci/echoaudio/built-in.o`<br>
`  CC      drivers/base/container.o`<br>
`  CC      kernel/events/core.o`<br>
`  CC      kernel/events/ring_buffer.o`<br>
`  LD      sound/pci/emu10k1/built-in.o`<br>
`  CC      mm/mremap.o`<br>
`  AS      arch/arm64/lib/memcpy.o`<br>
`  LD      sound/pci/hda/built-in.o`<br>
`  CC      crypto/hash_info.o`<br>
`  CC      fs/namespace.o`<br>
`  CC      crypto/ablk_helper.o`<br>
`  CC      drivers/base/property.o`<br>
`  CC      crypto/asymmetric_keys/asymmetric_type.o`<br>
`  LD      sound/pci/ice1712/built-in.o`<br>
`  CC      net/core/skbuff.o`<br>
`  CC      kernel/events/callchain.o`<br>
`  CC      mm/msync.o`<br>
`  AS      arch/arm64/lib/memmove.o`<br>
`  LD      sound/pci/korg1212/built-in.o`<br>
`  CC      crypto/asymmetric_keys/signature.o`<br>
`  LD      crypto/crypto.o`<br>
`  LD      crypto/crypto_algapi.o`<br>
`  CC      fs/seq_file.o`<br>
`  CC      crypto/asymmetric_keys/public_key.o`<br>
`  LD      crypto/crypto_blkcipher.o`<br>
`  LD      sound/pci/lola/built-in.o`<br>
`  CC      net/bluetooth/hci_event.o`<br>
`  CC      mm/rmap.o`<br>
`  LD      kernel/events/built-in.o`<br>
`  CC      drivers/base/devtmpfs.o`<br>
`  CC      fs/xattr.o`<br>
`  LD      sound/pci/lx6464es/built-in.o`<br>
`  AS      arch/arm64/lib/memset.o`<br>
`  CC      crypto/asymmetric_keys/rsa.o`<br>
`  LD      sound/pci/mixart/built-in.o`<br>
`  LD      sound/pcmcia/pdaudiocf/built-in.o`<br>
`  LD      sound/pci/nm256/built-in.o`<br>
`  CC      drivers/base/dma-contiguous.o`<br>
`  LD      sound/pcmcia/vx/built-in.o`<br>
`  CC      kernel/irq/irqdesc.o`<br>
`  LD      sound/pci/oxygen/built-in.o`<br>
`  CC      mm/vmalloc.o`<br>
`  CC      fs/libfs.o`<br>
`  AS      arch/arm64/lib/strchr.o`<br>
`  LD      sound/pcmcia/built-in.o`<br>
`  ASN.1   crypto/asymmetric_keys/x509-asn1.c`<br>
`Extracted 203 tokens`<br>
`Extracted 14 types`<br>
`Extracted 12 actions`<br>
`Pass 1`<br>
`Pass 2`<br>
`  ASN.1   crypto/asymmetric_keys/x509_rsakey-asn1.c`<br>
`Extracted 16 tokens`<br>
`Extracted 1 types`<br>
`Extracted 1 actions`<br>
`Pass 1`<br>
`Pass 2`<br>
`  LD      crypto/crypto_hash.o`<br>
`  CC      crypto/asymmetric_keys/x509_public_key.o`<br>
`  LD      sound/pci/pcxhr/built-in.o`<br>
`  CC      net/bridge/br_device.o`<br>
`  CC      net/bluetooth/mgmt.o`<br>
`  AS      arch/arm64/lib/strcmp.o`<br>
`  CC      lib/lockref.o`<br>
`  CC      drivers/block/brd.o`<br>
`  CC      fs/fs-writeback.o`<br>
`  LD      sound/pci/riptide/built-in.o`<br>
`  CC      drivers/base/regmap/regmap.o`<br>
`  CC      lib/bcd.o`<br>
`  CC      mm/pagewalk.o`<br>
`  CC      kernel/irq/handle.o`<br>
`  LD      sound/pci/rme9652/built-in.o`<br>
`  CC      drivers/base/power/sysfs.o`<br>
`  CC      lib/div64.o`<br>
`  LD      sound/pci/trident/built-in.o`<br>
`  CC      drivers/block/loop.o`<br>
`  LD      crypto/asymmetric_keys/asymmetric_keys.o`<br>
`  AS      arch/arm64/lib/strlen.o`<br>
`  CC      kernel/irq/manage.o`<br>
`  CC      mm/pgtable-generic.o`<br>
`  CC      crypto/asymmetric_keys/x509-asn1.o`<br>
`  LD      sound/pci/vx222/built-in.o`<br>
`  CC      fs/pnode.o`<br>
`  CC      drivers/base/regmap/regcache.o`<br>
`  CC      lib/sort.o`<br>
`  CC      drivers/base/power/generic_ops.o`<br>
`  CC      kernel/irq/spurious.o`<br>
`  LD      sound/pci/ymfpci/built-in.o`<br>
`  CC      crypto/asymmetric_keys/x509_rsakey-asn1.o`<br>
`  CC      mm/process_vm_access.o`<br>
`  AS      arch/arm64/lib/strncmp.o`<br>
`  CC      drivers/block/nbd.o`<br>
`  LD      sound/pci/built-in.o`<br>
`  CC      drivers/base/power/common.o`<br>
`  CC      fs/splice.o`<br>
`  LD      sound/ppc/built-in.o`<br>
`  CC      drivers/base/regmap/regcache-rbtree.o`<br>
`  CC      crypto/asymmetric_keys/x509_cert_parser.o`<br>
`  CC      kernel/irq/resend.o`<br>
`  CC      drivers/base/power/qos.o`<br>
`  LD      sound/sh/built-in.o`<br>
`  LD      crypto/asymmetric_keys/x509_key_parser.o`<br>
`  CC      fs/sync.o`<br>
`  LD      crypto/asymmetric_keys/built-in.o`<br>
`  AS      arch/arm64/lib/strnlen.o`<br>
`  CC      lib/parser.o`<br>
`  LD      crypto/cryptomgr.o`<br>
`  CC      drivers/block/zram/zcomp_lzo.o`<br>
`  CC      drivers/base/regmap/regcache-lzo.o`<br>
`  LD      crypto/built-in.o`<br>
`  CC      kernel/irq/chip.o`<br>
`  CC      drivers/base/power/runtime.o`<br>
`  CC      lib/halfmd4.o`<br>
`  CC      drivers/base/regmap/regcache-flat.o`<br>
`  CC      fs/utimes.o`<br>
`  AS      arch/arm64/lib/strrchr.o`<br>
`  CC      drivers/block/zram/zcomp.o`<br>
`  CC      lib/debug_locks.o`<br>
`  CC      kernel/irq/dummychip.o`<br>
`  CC      drivers/block/zram/zram_drv.o`<br>
`  CC      drivers/base/power/main.o`<br>
`  CC      drivers/base/regmap/regmap-debugfs.o`<br>
`  AR      arch/arm64/lib/lib.a`<br>
`  CC      lib/random32.o`<br>
`  CC      fs/stack.o`<br>
`  CC      lib/bust_spinlocks.o`<br>
`  CC      kernel/irq/devres.o`<br>
`  CC      drivers/base/power/wakeup.o`<br>
`  LD      drivers/block/zram/zram.o`<br>
`  LD      drivers/block/zram/built-in.o`<br>
`  CC      drivers/base/regmap/regmap-i2c.o`<br>
`  CC      fs/fs_struct.o`<br>
`  CC      lib/hexdump.o`<br>
`  LD      drivers/block/built-in.o`<br>
`  CC      kernel/irq/autoprobe.o`<br>
`  CC      drivers/base/power/opp/core.o`<br>
`  LD      drivers/firmware/efi/libstub/built-in.o`<br>
`  CC      drivers/base/power/opp/cpu.o`<br>
`  CC      drivers/firmware/efi/libstub/arm-stub.o`<br>
`  CC      drivers/bluetooth/hci_ldisc.o`<br>
`  CC      lib/kasprintf.o`<br>
`  CC      fs/statfs.o`<br>
`  CC      kernel/irq/irqdomain.o`<br>
`  CC      drivers/base/regmap/regmap-spi.o`<br>
`  LD      drivers/base/power/opp/built-in.o`<br>
`  CC      sound/soc/soc-core.o`<br>
`  CC      mm/showmem.o`<br>
`  CC      kernel/irq/proc.o`<br>
`  CC      drivers/base/power/clock_ops.o`<br>
`  CC      lib/bitmap.o`<br>
`  CC      mm/vmpressure.o`<br>
`  CC      drivers/bluetooth/hci_h4.o`<br>
`  CC      net/bridge/br_fdb.o`<br>
`  CC      drivers/base/regmap/regmap-swr.o`<br>
`  CC      kernel/irq/pm.o`<br>
`  CC      fs/fs_pin.o`<br>
`  LD      drivers/base/power/built-in.o`<br>
`  CC      lib/scatterlist.o`<br>
`  CC      drivers/base/dma-mapping.o`<br>
`  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3-mido.dtb`<br>
`  CC      fs/buffer.o`<br>
`  LD      drivers/base/regmap/built-in.o`<br>
`  CC      drivers/bluetooth/bluetooth-power.o`<br>
`  CC      net/core/iovec.o`<br>
`  CC      mm/init-mm.o`<br>
`  CC      lib/gcd.o`<br>
`  CC      drivers/base/dma-coherent.o`<br>
`  CC      kernel/irq/msi.o`<br>
`  CC      sound/soc/soc-dapm.o`<br>
`  CC      lib/lcm.o`<br>
`  CC      mm/nobootmem.o`<br>
`  CC      fs/block_dev.o`<br>
`  LD      drivers/bluetooth/hci_uart.o`<br>
`  CC      drivers/base/dma-removed.o`<br>
`  LD      drivers/bluetooth/built-in.o`<br>
`  CC      kernel/locking/mutex.o`<br>
`  CC      mm/fadvise.o`<br>
`  LD      kernel/irq/built-in.o`<br>
`  CC      drivers/firmware/efi/libstub/efi-stub-helper.o`<br>
`  CC      sound/soc/soc-jack.o`<br>
`  CC      lib/list_sort.o`<br>
`  CC      sound/soc/soc-cache.o`<br>
`  CC      fs/direct-io.o`<br>
`  CC      drivers/base/firmware_class.o`<br>
`  CC      kernel/locking/semaphore.o`<br>
`  CC      kernel/locking/rwsem.o`<br>
`  CC      sound/soc/soc-utils.o`<br>
`  CC      lib/uuid.o`<br>
`  CC      mm/madvise.o`<br>
`  CC      lib/flex_array.o`<br>
`  CC      fs/mpage.o`<br>
`  CC      kernel/power/qos.o`<br>
`  CC      drivers/base/soc.o`<br>
`  CC      drivers/base/pinctrl.o`<br>
`  CC      kernel/locking/mcs_spinlock.o`<br>
`  CC      mm/memblock.o`<br>
`  CC      fs/proc_namespace.o`<br>
`  CC      sound/soc/soc-pcm.o`<br>
`  CC      kernel/power/main.o`<br>
`  CC      lib/iovec.o`<br>
`  CC      kernel/locking/spinlock.o`<br>
`  CC      drivers/base/devcoredump.o`<br>
`  CC      kernel/locking/lglock.o`<br>
`  CC      net/bluetooth/hci_sock.o`<br>
`  CC      mm/page_io.o`<br>
`  CC      sound/soc/soc-compress.o`<br>
`  CC      fs/autofs4/init.o`<br>
`  CC      mm/swap_state.o`<br>
`  LD      drivers/base/built-in.o`<br>
`  CC      net/bridge/br_forward.o`<br>
`  CC      drivers/firmware/efi/libstub/fdt.o`<br>
`  CC      kernel/power/console.o`<br>
`  CC      lib/clz_ctz.o`<br>
`  CC      kernel/locking/rtmutex.o`<br>
`  CC      mm/swapfile.o`<br>
`  LD      drivers/bus/built-in.o`<br>
`  CC      fs/autofs4/inode.o`<br>
`  CC      sound/soc/soc-io.o`<br>
`  CC      lib/bsearch.o`<br>
`  CC      kernel/power/process.o`<br>
`  LD      drivers/cdrom/built-in.o`<br>
`  CC      kernel/locking/rwsem-xadd.o`<br>
`  CC      mm/swap_ratio.o`<br>
`  CC      lib/find_last_bit.o`<br>
`  CC      sound/soc/soc-devres.o`<br>
`  CC      fs/autofs4/root.o`<br>
`  LD      kernel/locking/built-in.o`<br>
`  CC      lib/find_next_bit.o`<br>
`  CC      kernel/printk/printk.o`<br>
`  CC      kernel/power/suspend.o`<br>
`  CC      mm/zcache.o`<br>
`  CC      fs/autofs4/symlink.o`<br>
`  CC      drivers/char/mem.o`<br>
`  CC      lib/llist.o`<br>
`  LD      sound/soc/adi/built-in.o`<br>
`  CC      kernel/power/autosleep.o`<br>
`  LD      kernel/printk/built-in.o`<br>
`  CC      fs/autofs4/waitq.o`<br>
`  LD      sound/soc/atmel/built-in.o`<br>
`  CC      net/core/datagram.o`<br>
`  CC      mm/dmapool.o`<br>
`  CC      drivers/char/random.o`<br>
`  CC      drivers/char/misc.o`<br>
`  CC      lib/memweight.o`<br>
`  CC      fs/btrfs/super.o`<br>
`  LD      sound/soc/au1x/built-in.o`<br>
`  CC      fs/cifs/cifsfs.o`<br>
`  AR      drivers/firmware/efi/libstub/lib.a`<br>
`  CC      mm/sparse.o`<br>
`  LD      sound/soc/bcm/built-in.o`<br>
`  CC      lib/kfifo.o`<br>
`  CC      drivers/char/msm_smd_pkt.o`<br>
`  CC      net/core/stream.o`<br>
`  CC      kernel/power/wakelock.o`<br>
`  LD      sound/soc/cirrus/built-in.o`<br>
`  LD      sound/soc/blackfin/built-in.o`<br>
`  CC      fs/autofs4/expire.o`<br>
`  CC      fs/autofs4/dev-ioctl.o`<br>
`  CC      net/bridge/br_if.o`<br>
`  CC      lib/percpu-refcount.o`<br>
`  CC      fs/cifs/cifssmb.o`<br>
`  CC      mm/sparse-vmemmap.o`<br>
`  CC      fs/btrfs/ctree.o`<br>
`  LD      drivers/char/agp/built-in.o`<br>
`  CC      lib/percpu_ida.o`<br>
`  CC      kernel/power/suspend_time.o`<br>
`  CC      mm/slub.o`<br>
`  LD      sound/soc/davinci/built-in.o`<br>
`  LD      fs/autofs4/autofs4.o`<br>
`  CC      fs/cifs/cifs_debug.o`<br>
`  CC      fs/cifs/connect.o`<br>
`  LD      fs/autofs4/built-in.o`<br>
`  CC      fs/cifs/dir.o`<br>
`  CC      fs/btrfs/extent-tree.o`<br>
`  CC      kernel/power/poweroff.o`<br>
`  CC      lib/hash.o`<br>
`  CC      lib/rhashtable.o`<br>
`  CC      drivers/char/diag/diagchar_core.o`<br>
`  CC      kernel/power/wakeup_reason.o`<br>
`  CC      net/bluetooth/hci_sysfs.o`<br>
`  CC      net/bluetooth/l2cap_core.o`<br>
`  CC      mm/migrate.o`<br>
`  CC      mm/memcontrol.o`<br>
`  CC      fs/btrfs/print-tree.o`<br>
`  CC      sound/soc/codecs/msm_hdmi_dba_codec_rx.o`<br>
`  CC      drivers/char/diag/diagchar_hdlc.o`<br>
`  CC      lib/reciprocal_div.o`<br>
`  CC      fs/cifs/file.o`<br>
`  CC      lib/string_helpers.o`<br>
`  CC      mm/page_cgroup.o`<br>
`  CC      mm/cleancache.o`<br>
`  LD      kernel/power/built-in.o`<br>
`  CC      fs/cifs/inode.o`<br>
`  CC      fs/btrfs/root-tree.o`<br>
`  CC      sound/soc/codecs/wcd9330.o`<br>
`  CC      lib/kstrtox.o`<br>
`  CC      fs/cifs/link.o`<br>
`  CC      kernel/rcu/update.o`<br>
`  CC      drivers/char/diag/diagfwd.o`<br>
`  CC      net/bridge/br_input.o`<br>
`  CC      drivers/char/diag/diagfwd_peripheral.o`<br>
`  CC      lib/iomap.o`<br>
`  CC      mm/page_isolation.o`<br>
`  CC      kernel/rcu/srcu.o`<br>
`  CC      sound/soc/codecs/wcd9330-tables.o`<br>
`  CC      fs/btrfs/dir-item.o`<br>
`  CC      fs/cifs/misc.o`<br>
`  CC      sound/soc/codecs/wcd9335.o`<br>
`  CC      fs/cifs/netmisc.o`<br>
`  CC      lib/pci_iomap.o`<br>
`  CC      drivers/char/diag/diagfwd_smd.o`<br>
`  CC      fs/btrfs/file-item.o`<br>
`  CC      kernel/rcu/tree.o`<br>
`  CC      sound/soc/codecs/wcdcal-hwdep.o`<br>
`  CC      mm/zbud.o`<br>
`  CC      fs/cifs/smbencrypt.o`<br>
`  CC      sound/soc/codecs/audio-ext-clk.o`<br>
`  CC      fs/btrfs/inode-item.o`<br>
`  CC      net/bluetooth/l2cap_sock.o`<br>
`  CC      lib/iomap_copy.o`<br>
`  CC      drivers/char/diag/diagfwd_socket.o`<br>
`  CC      mm/zsmalloc.o`<br>
`  CC      fs/cifs/transport.o`<br>
`  CC      sound/soc/codecs/wcd9xxx-resmgr.o`<br>
`  CC      sound/soc/codecs/wcd9xxx-mbhc.o`<br>
`  LD      kernel/rcu/built-in.o`<br>
`  CC      lib/devres.o`<br>
`  CC      net/core/scm.o`<br>
`  CC      lib/hweight.o`<br>
`  CC      fs/btrfs/inode-map.o`<br>
`  CC      mm/early_ioremap.o`<br>
`  CC      kernel/sched/core.o`<br>
`  CC      fs/btrfs/disk-io.o`<br>
`  CC      drivers/char/diag/diag_mux.o`<br>
`  CC      lib/assoc_array.o`<br>
`  CC      net/bridge/br_ioctl.o`<br>
`  CC      fs/cifs/asn1.o`<br>
`  CC      lib/smp_processor_id.o`<br>
`  CC      sound/soc/codecs/wcd9xxx-common.o`<br>
`  CC      drivers/char/diag/diag_memorydevice.o`<br>
`  CC      kernel/sched/fair.o`<br>
`  CC      mm/cma.o`<br>
`  CC      sound/soc/codecs/wcd9xxx-common-v2.o`<br>
`  CC      fs/btrfs/transaction.o`<br>
`  CC      net/bluetooth/smp.o`<br>
`  CC      lib/bitrev.o`<br>
`  CC      fs/btrfs/inode.o`<br>
`  CC      drivers/char/diag/diag_usb.o`<br>
`  CC      kernel/sched/rt.o`<br>
`  CC      mm/process_reclaim.o`<br>
`  CC      fs/cifs/cifs_unicode.o`<br>
`  CC      sound/soc/codecs/wcd9xxx-resmgr-v2.o`<br>
`  CC      drivers/char/diag/diagmem.o`<br>
`  CC      lib/crc-ccitt.o`<br>
`  CC      lib/crc16.o`<br>
`  CC      fs/btrfs/file.o`<br>
`  CC      net/core/gen_stats.o`<br>
`  CC      sound/soc/codecs/msm8x16-wcd.o`<br>
`  CC      drivers/char/diag/diagfwd_cntl.o`<br>
`  CC      kernel/sched/proc.o`<br>
`  CC      lib/crc-itu-t.o`<br>
`  CC      mm/cma_debug.o`<br>
`  CC      fs/btrfs/tree-defrag.o`<br>
`  HOSTCC  lib/gen_crc32table`<br>
`  CC      fs/cifs/nterr.o`<br>
`  CC      sound/soc/codecs/msm8x16-wcd-tables.o`<br>
`  LD      mm/built-in.o`<br>
`  CC      drivers/char/diag/diag_dci.o`<br>
`  CC      sound/soc/codecs/msm8916-wcd-irq.o`<br>
`  CC      lib/libcrc32c.o`<br>
`  CC      fs/btrfs/extent_map.o`<br>
`  CC      net/bridge/br_stp.o`<br>
`  CC      kernel/sched/clock.o`<br>
`  CC      lib/genalloc.o`<br>
`  CC      fs/cifs/xattr.o`<br>
`  CC      fs/cifs/cifsencrypt.o`<br>
`  CC      sound/soc/codecs/wcd_cpe_services.o`<br>
`  CC      fs/cifs/readdir.o`<br>
`  CC      fs/btrfs/sysfs.o`<br>
`  CC      kernel/sched/cputime.o`<br>
`  CC      drivers/char/diag/diag_masks.o`<br>
`  CC      drivers/char/diag/diag_debugfs.o`<br>
`  CC      lib/lzo/lzo1x_compress.o`<br>
`  CC      fs/cifs/ioctl.o`<br>
`  CC      fs/btrfs/struct-funcs.o`<br>
`  CC      kernel/sched/idle_task.o`<br>
`  CC      net/bluetooth/sco.o`<br>
`  CC      sound/soc/codecs/wcd_cpe_core.o`<br>
`  CC      lib/lzo/lzo1x_decompress_safe.o`<br>
`  LD      drivers/char/diag/diagchar.o`<br>
`  CC      fs/cifs/sess.o`<br>
`  LD      drivers/char/diag/built-in.o`<br>
`  CC      fs/btrfs/xattr.o`<br>
`  CC      fs/cifs/export.o`<br>
`  CC      kernel/sched/deadline.o`<br>
`  CC      lib/mpi/generic_mpih-lshift.o`<br>
`  CC      sound/soc/codecs/wcd-mbhc-v2.o`<br>
`  CC      drivers/char/hw_random/core.o`<br>
`  CC      fs/cifs/smb1ops.o`<br>
`  LD      lib/lzo/lzo_compress.o`<br>
`  CC      lib/mpi/generic_mpih-mul1.o`<br>
`  LD      lib/lzo/lzo_decompress.o`<br>
`  LD      lib/lzo/built-in.o`<br>
`  CC      net/bridge/br_stp_bpdu.o`<br>
`  CC      fs/btrfs/ordered-data.o`<br>
`  CC      fs/cifs/winucase.o`<br>
`  CC      kernel/sched/stop_task.o`<br>
`  CC      drivers/char/hw_random/msm_rng.o`<br>
`  CC      fs/btrfs/extent_io.o`<br>
`  CC      sound/soc/codecs/wsa881x.o`<br>
`  CC      net/bridge/br_stp_if.o`<br>
`  CC      kernel/sched/wait.o`<br>
`  CC      lib/mpi/generic_mpih-mul2.o`<br>
`  LD      fs/cifs/cifs.o`<br>
`  CC      kernel/sched/completion.o`<br>
`  LD      drivers/char/hw_random/rng-core.o`<br>
`  CC      kernel/sched/idle.o`<br>
`  CC      fs/btrfs/volumes.o`<br>
`  CC      fs/btrfs/async-thread.o`<br>
`  LD      fs/cifs/built-in.o`<br>
`  LD      drivers/char/hw_random/built-in.o`<br>
`  CC      lib/mpi/generic_mpih-mul3.o`<br>
`  CC      sound/soc/codecs/wsa881x-tables.o`<br>
`  CC      drivers/char/adsprpc.o`<br>
`  CC      net/bluetooth/lib.o`<br>
`  CC      net/core/gen_estimator.o`<br>
`  CC      lib/mpi/generic_mpih-rshift.o`<br>
`  CC      sound/soc/codecs/wsa881x-regmap.o`<br>
`  CC      drivers/char/adsprpc_compat.o`<br>
`  CC      fs/configfs/inode.o`<br>
`  CC      kernel/sched/sched_avg.o`<br>
`  CC      lib/mpi/generic_mpih-sub1.o`<br>
`  CC      fs/crypto/crypto.o`<br>
`  CC      lib/mpi/generic_mpih-add1.o`<br>
`  CC      fs/btrfs/ioctl.o`<br>
`  LD      drivers/char/built-in.o`<br>
`  CC      kernel/sched/cpupri.o`<br>
`  CC      fs/crypto/fname.o`<br>
`  CC      sound/soc/codecs/wsa881x-analog.o`<br>
`  CC      fs/btrfs/locking.o`<br>
`  CC      lib/mpi/mpicoder.o`<br>
`  CC      drivers/clk/clk-devres.o`<br>
`  CC      fs/configfs/file.o`<br>
`  CC      net/bridge/br_stp_timer.o`<br>
`  CC      kernel/sched/cpudeadline.o`<br>
`  CC      fs/debugfs/inode.o`<br>
`  CC      lib/mpi/mpi-bit.o`<br>
`  CC      fs/crypto/policy.o`<br>
`  CC      sound/soc/codecs/wsa881x-tables-analog.o`<br>
`  CC      fs/btrfs/orphan.o`<br>
`  CC      drivers/clk/clkdev.o`<br>
`  CC      lib/mpi/mpi-cmp.o`<br>
`  CC      fs/debugfs/file.o`<br>
`  CC      sound/soc/codecs/wsa881x-regmap-analog.o`<br>
`  CC      fs/btrfs/export.o`<br>
`  CC      kernel/sched/stats.o`<br>
`  CC      fs/crypto/keyinfo.o`<br>
`  CC      fs/configfs/dir.o`<br>
`  CC      drivers/clk/clk.o`<br>
`  CC      lib/mpi/mpih-cmp.o`<br>
`  LD      fs/debugfs/debugfs.o`<br>
`  CC      fs/crypto/bio.o`<br>
`  CC      sound/soc/codecs/wsa881x-irq.o`<br>
`  LD      fs/debugfs/built-in.o`<br>
`  CC      fs/btrfs/tree-log.o`<br>
`  CC      kernel/sched/debug.o`<br>
`  CC      fs/devpts/inode.o`<br>
`  CC      fs/configfs/symlink.o`<br>
`  CC      lib/mpi/mpih-div.o`<br>
`  CC      sound/soc/codecs/wsa881x-temp-sensor.o`<br>
`  CC      fs/ecryptfs/dentry.o`<br>
`  LD      fs/crypto/fscrypto.o`<br>
`  LD      fs/crypto/built-in.o`<br>
`  CC      fs/btrfs/free-space-cache.o`<br>
`  CC      fs/configfs/mount.o`<br>
`  CC      kernel/sched/cpuacct.o`<br>
`  CC      fs/btrfs/zlib.o`<br>
`  LD      fs/devpts/devpts.o`<br>
`  LD      fs/devpts/built-in.o`<br>
`  CC      fs/ecryptfs/file.o`<br>
`  CC      sound/soc/codecs/msm_stub.o`<br>
`  LD      fs/exofs/built-in.o`<br>
`  CC      drivers/clk/msm/clock.o`<br>
`  CC      lib/mpi/mpih-mul.o`<br>
`  CC      fs/btrfs/lzo.o`<br>
`  CC      fs/btrfs/compression.o`<br>
`  CC      net/bridge/br_netlink.o`<br>
`  CC      fs/configfs/item.o`<br>
`  LD      kernel/sched/built-in.o`<br>
`  CC      fs/ecryptfs/inode.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd9330.o`<br>
`  CC      fs/exportfs/expfs.o`<br>
`  CC      net/bluetooth/a2mp.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd9335.o`<br>
`  CC      fs/btrfs/delayed-ref.o`<br>
`  CC      drivers/clk/msm/clock-dummy.o`<br>
`  LD      sound/soc/codecs/audio-ext-clock.o`<br>
`  LD      fs/configfs/configfs.o`<br>
`  LD      fs/exportfs/exportfs.o`<br>
`  CC      net/core/net_namespace.o`<br>
`  LD      fs/exportfs/built-in.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd9xxx.o`<br>
`  CC      lib/mpi/mpi-pow.o`<br>
`  CC      net/bridge/br_sysfs_if.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd9xxx-v2.o`<br>
`  CC      net/bluetooth/amp.o`<br>
`  LD      sound/soc/codecs/snd-soc-msm8952-wcd.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd-cpe.o`<br>
`  LD      fs/configfs/built-in.o`<br>
`  CC      fs/ecryptfs/main.o`<br>
`  CC      fs/btrfs/relocation.o`<br>
`  CC      fs/ecryptfs/super.o`<br>
`  LD      sound/soc/codecs/snd-soc-wcd-mbhc.o`<br>
`  LD      sound/soc/codecs/snd-soc-wsa881x.o`<br>
`  CC      drivers/clk/msm/clock-generic.o`<br>
`  CC      lib/mpi/mpiutil.o`<br>
`  CC      net/bluetooth/bnep/core.o`<br>
`  CC      net/bluetooth/hidp/core.o`<br>
`  LD      sound/soc/codecs/snd-soc-wsa881x-analog.o`<br>
`  LD      sound/soc/codecs/snd-soc-wsa881x-sensor.o`<br>
`  LD      sound/soc/codecs/snd-soc-msm-stub.o`<br>
`  CC      drivers/clk/msm/clock-local2.o`<br>
`  LD      sound/soc/codecs/built-in.o`<br>
`  HZFILE  kernel/time/hz.bc`<br>
`  CC      fs/btrfs/delayed-inode.o`<br>
`  CC      kernel/time/timer.o`<br>
`  CC      fs/ecryptfs/mmap.o`<br>
`  CC      kernel/time/hrtimer.o`<br>
`  LD      lib/mpi/mpi.o`<br>
`  LD      lib/mpi/built-in.o`<br>
`  LD      sound/soc/dwc/built-in.o`<br>
`  CC      drivers/clk/msm/clock-pll.o`<br>
`  LD      sound/soc/fsl/built-in.o`<br>
`  CC      fs/btrfs/scrub.o`<br>
`  LD      sound/soc/generic/built-in.o`<br>
`  CC      kernel/time/itimer.o`<br>
`  CC      kernel/time/posix-timers.o`<br>
`  CC      fs/ecryptfs/read_write.o`<br>
`  LD      sound/soc/intel/built-in.o`<br>
`  CC      lib/raid6/algos.o`<br>
`  CC      fs/btrfs/reada.o`<br>
`  LD      sound/soc/jz4740/built-in.o`<br>
`  CC      kernel/time/posix-cpu-timers.o`<br>
`  CC      drivers/clk/msm/clock-alpha-pll.o`<br>
`  CC      kernel/time/timekeeping.o`<br>
`  CC      fs/ecryptfs/events.o`<br>
`  LD      sound/soc/kirkwood/built-in.o`<br>
`  CC      lib/raid6/recov.o`<br>
`  CC      fs/ecryptfs/crypto.o`<br>
`  CC      fs/ext2/balloc.o`<br>
`  CC      drivers/clk/msm/clock-rpm.o`<br>
`  CC      kernel/time/ntp.o`<br>
`  CC      fs/btrfs/backref.o`<br>
`  HOSTCC  lib/raid6/mktables`<br>
`  CC      net/bluetooth/rfcomm/core.o`<br>
`  CC      fs/ext2/dir.o`<br>
`  CC      fs/ecryptfs/keystore.o`<br>
`  CC      sound/soc/msm/msm-pcm-hostless.o`<br>
`  CC      fs/ecryptfs/kthread.o`<br>
`  CC      kernel/time/clocksource.o`<br>
`  CC      drivers/clk/msm/clock-voter.o`<br>
`  UNROLL  lib/raid6/int1.c`<br>
`  CC      net/bluetooth/rfcomm/sock.o`<br>
`  CC      fs/btrfs/ulist.o`<br>
`  UNROLL  lib/raid6/int2.c`<br>
`  CC      fs/ecryptfs/debug.o`<br>
`  CC      fs/ext2/file.o`<br>
`  UNROLL  lib/raid6/int4.c`<br>
`  UNROLL  lib/raid6/int8.c`<br>
`  UNROLL  lib/raid6/int16.c`<br>
`  CC      kernel/time/jiffies.o`<br>
`  UNROLL  lib/raid6/int32.c`<br>
`  CC      drivers/clk/msm/clock-pm.o`<br>
`  CC      lib/raid6/neon.o`<br>
`  LD      fs/ecryptfs/ecryptfs.o`<br>
`  CC      fs/ext2/ialloc.o`<br>
`  CC      fs/btrfs/qgroup.o`<br>
`  LD      fs/ecryptfs/built-in.o`<br>
`  CC      kernel/time/timer_list.o`<br>
`  CC      kernel/time/timeconv.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dai-slim.o`<br>
`  UNROLL  lib/raid6/neon1.c`<br>
`  UNROLL  lib/raid6/neon2.c`<br>
`  UNROLL  lib/raid6/neon4.c`<br>
`  CC      fs/ext2/inode.o`<br>
`  UNROLL  lib/raid6/neon8.c`<br>
`  CC      fs/btrfs/send.o`<br>
`  CC      net/bluetooth/bnep/sock.o`<br>
`  CC      drivers/clk/msm/msm-clock-controller.o`<br>
`  TABLE   lib/raid6/tables.c`<br>
`  CC      net/bridge/br_sysfs_br.o`<br>
`  CC      lib/raid6/int1.o`<br>
`  CC      kernel/time/posix-clock.o`<br>
`  CC      sound/soc/msm/qdsp6v2/audio_slimslave.o`<br>
`  CC      lib/raid6/int2.o`<br>
`  CC      drivers/clk/msm/clock-debug.o`<br>
`  CC      fs/btrfs/dev-replace.o`<br>
`  CC      fs/ext2/ioctl.o`<br>
`  CC      kernel/time/alarmtimer.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-v2.o`<br>
`  CC      fs/ext2/namei.o`<br>
`  CC      drivers/clk/msm/clock-cpu-8953.o`<br>
`  CC      lib/raid6/int4.o`<br>
`  CC      fs/btrfs/raid56.o`<br>
`  CC      fs/ext2/super.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-v2.o`<br>
`  CC      kernel/time/clockevents.o`<br>
`  CC      net/core/secure_seq.o`<br>
`  CC      lib/raid6/int8.o`<br>
`  CC      kernel/time/tick-common.o`<br>
`  CC      drivers/clk/msm/clock-gcc-8953.o`<br>
`  CC      drivers/clk/msm/clock-rcgwr.o`<br>
`  CC      fs/btrfs/uuid-tree.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-v2.o`<br>
`  CC      fs/btrfs/props.o`<br>
`  CC      fs/ext2/symlink.o`<br>
`  CC      lib/raid6/int16.o`<br>
`  CC      kernel/time/tick-broadcast.o`<br>
`  CC      drivers/clk/msm/gdsc.o`<br>
`  LD      sound/sparc/built-in.o`<br>
`  LD      sound/soc/mxs/built-in.o`<br>
`  CC      net/bluetooth/bnep/netdev.o`<br>
`  CC      fs/btrfs/hash.o`<br>
`  CC      lib/raid6/int32.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-compress-q6-v2.o`<br>
`  CC      fs/ext2/xattr.o`<br>
`  LD      sound/soc/nuc900/built-in.o`<br>
`  LD      sound/spi/built-in.o`<br>
`  CC      kernel/time/tick-broadcast-hrtimer.o`<br>
`  CC      fs/ext2/xattr_user.o`<br>
`  CC      kernel/trace/trace_clock.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-pll-util.o`<br>
`  LD      sound/soc/omap/built-in.o`<br>
`  CC      lib/raid6/neon1.o`<br>
`  CC      kernel/trace/ftrace.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-lpa-v2.o`<br>
`  CC      kernel/time/sched_clock.o`<br>
`  LD      fs/btrfs/btrfs.o`<br>
`  CC      kernel/time/tick-oneshot.o`<br>
`  CC      fs/ext2/xattr_trusted.o`<br>
`  CC      kernel/time/tick-sched.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-pll.o`<br>
`  CC      kernel/trace/ring_buffer.o`<br>
`  CC      kernel/trace/trace.o`<br>
`  CC      lib/raid6/neon2.o`<br>
`  CC      net/bluetooth/hidp/sock.o`<br>
`  LD      fs/btrfs/built-in.o`<br>
`  LD      fs/ext2/ext2.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-afe-v2.o`<br>
`  CC      kernel/trace/trace_output.o`<br>
`  CC      kernel/time/timer_stats.o`<br>
`  LD      fs/ext2/built-in.o`<br>
`  CC      lib/raid6/neon4.o`<br>
`  CC      kernel/trace/trace_seq.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-dsi-pll-util.o`<br>
`  LD      sound/synth/built-in.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-voip-v2.o`<br>
`  CC      net/dns_resolver/dns_key.o`<br>
`  CC      kernel/time/timekeeping_debug.o`<br>
`  CC      net/core/flow_dissector.o`<br>
`  CC      lib/raid6/neon8.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-dsi-pll-28lpm.o`<br>
`  CC      kernel/trace/trace_stat.o`<br>
`  CC      fs/ext3/balloc.o`<br>
`  BC      kernel/time/timeconst.h`<br>
`  CC      fs/f2fs/dir.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-voice-v2.o`<br>
`  CC      lib/raid6/tables.o`<br>
`  CC      kernel/time/time.o`<br>
`  CC      kernel/trace/trace_printk.o`<br>
`  CC      fs/ext3/bitmap.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996.o`<br>
`  CC      fs/ext4/balloc.o`<br>
`  LD      lib/raid6/raid6_pq.o`<br>
`  CC      fs/f2fs/file.o`<br>
`  CC      kernel/trace/trace_sched_switch.o`<br>
`  CC      fs/ext3/dir.o`<br>
`  LD      lib/raid6/built-in.o`<br>
`  LD      kernel/time/built-in.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dai-q6-hdmi-v2.o`<br>
`  CC      kernel/kcmp.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-dsi-pll-8996-util.o`<br>
`  CC      fs/ext3/file.o`<br>
`  CC      lib/zlib_deflate/deflate.o`<br>
`  CC      lib/reed_solomon/reed_solomon.o`<br>
`  CC      kernel/trace/trace_functions.o`<br>
`  CC      fs/f2fs/inode.o`<br>
`  CC      fs/ext4/bitmap.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-lsm-client.o`<br>
`  CC      kernel/freezer.o`<br>
`  CC      drivers/clk/msm/mdss/mdss-hdmi-pll-8996.o`<br>
`  LD      lib/reed_solomon/built-in.o`<br>
`  CC      lib/zlib_deflate/deftree.o`<br>
`  CC      fs/ext3/fsync.o`<br>
`  CC      net/dns_resolver/dns_query.o`<br>
`  CC      fs/ext4/dir.o`<br>
`  CC      kernel/trace/trace_cpu_freq_switch.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-host-voice-v2.o`<br>
`  CC      lib/zlib_deflate/deflate_syms.o`<br>
`  CC      kernel/profile.o`<br>
`  CC      fs/f2fs/namei.o`<br>
`  LD      drivers/clk/msm/mdss/built-in.o`<br>
`  CC      lib/zlib_inflate/inffast.o`<br>
`  CC      fs/ext3/ialloc.o`<br>
`  LD      drivers/clk/msm/built-in.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-audio-effects-q6-v2.o`<br>
`  CC      kernel/trace/trace_nop.o`<br>
`  LD      lib/zlib_deflate/zlib_deflate.o`<br>
`  CC      fs/ext4/file.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-loopback-v2.o`<br>
`  CC      net/bridge/br_nf_core.o`<br>
`  CC      fs/f2fs/hash.o`<br>
`  LD      lib/zlib_deflate/built-in.o`<br>
`  LD      drivers/clk/built-in.o`<br>
`  CC      fs/ext3/inode.o`<br>
`  LD      sound/soc/pxa/built-in.o`<br>
`  CC      lib/zlib_inflate/inflate.o`<br>
`  CC      lib/zlib_inflate/infutil.o`<br>
`  CC      lib/zlib_inflate/inftrees.o`<br>
`  CC      kernel/trace/trace_functions_graph.o`<br>
`  CC      net/bluetooth/rfcomm/tty.o`<br>
`  CC      fs/ext4/fsync.o`<br>
`  CC      drivers/clocksource/clksrc-of.o`<br>
`  CC      sound/soc/msm/qdsp6v2/adsp_err.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-dtmf-v2.o`<br>
`  CC      fs/f2fs/super.o`<br>
`  CC      lib/zlib_inflate/inflate_syms.o`<br>
`  CC      fs/f2fs/inline.o`<br>
`  CC      fs/ext3/ioctl.o`<br>
`  LD      net/bluetooth/bnep/bnep.o`<br>
`  CC      fs/f2fs/checkpoint.o`<br>
`  CC      kernel/trace/blktrace.o`<br>
`  LD      net/bluetooth/bnep/built-in.o`<br>
`  CC      drivers/clocksource/arm_arch_timer.o`<br>
`  CC      fs/ext4/ialloc.o`<br>
`  CC      net/ethernet/eth.o`<br>
`  CC      fs/ext4/inode.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dai-stub-v2.o`<br>
`  CC      drivers/coresight/coresight.o`<br>
`  CC      fs/ext4/page-io.o`<br>
`  LD      lib/zlib_inflate/zlib_inflate.o`<br>
`  LD      lib/zlib_inflate/built-in.o`<br>
`  CC      lib/textsearch.o`<br>
`  CC      fs/ext3/namei.o`<br>
`  CC      kernel/trace/trace_events.o`<br>
`  CC      fs/f2fs/gc.o`<br>
`  CC      fs/f2fs/data.o`<br>
`  CC      fs/ext4/ioctl.o`<br>
`  CC      drivers/clocksource/dummy_timer.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-routing-devdep.o`<br>
`  CC      fs/ext4/namei.o`<br>
`  CC      drivers/coresight/coresight-event.o`<br>
`  CC      fs/ext3/super.o`<br>
`  CC      lib/ts_kmp.o`<br>
`  CC      fs/f2fs/node.o`<br>
`  CC      kernel/trace/trace_export.o`<br>
`  CC      drivers/coresight/coresight-fuse.o`<br>
`  LD      net/dns_resolver/dns_resolver.o`<br>
`  LD      drivers/clocksource/built-in.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dolby-dap-config.o`<br>
`  CC      lib/ts_bm.o`<br>
`  CC      fs/f2fs/segment.o`<br>
`  CC      fs/ext4/super.o`<br>
`  LD      net/dns_resolver/built-in.o`<br>
`  CC      drivers/coresight/coresight-cti.o`<br>
`  CC      drivers/coresight/coresight-csr.o`<br>
`  LD      net/bluetooth/bluetooth.o`<br>
`  CC      fs/ext3/symlink.o`<br>
`  CC      lib/ts_fsm.o`<br>
`  CC      fs/ext3/hash.o`<br>
`  CC      kernel/stacktrace.o`<br>
`  LD      net/bluetooth/hidp/hidp.o`<br>
`  CC      kernel/trace/trace_event_perf.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-ds2-dap-config.o`<br>
`  CC      kernel/futex.o`<br>
`  CC      fs/f2fs/recovery.o`<br>
`  LD      net/bluetooth/hidp/built-in.o`<br>
`  CC      fs/ext4/symlink.o`<br>
`  CC      drivers/coresight/coresight-tmc.o`<br>
`  CC      fs/f2fs/shrinker.o`<br>
`  CC      fs/f2fs/extent_cache.o`<br>
`  CC      fs/f2fs/debug.o`<br>
`  CC      fs/ext3/resize.o`<br>
`  CC      lib/percpu_counter.o`<br>
`  CC      kernel/futex_compat.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-dts-srs-tm-config.o`<br>
`  CC      kernel/trace/trace_events_filter.o`<br>
`  CC      kernel/trace/trace_events_trigger.o`<br>
`  CC      fs/f2fs/xattr.o`<br>
`  CC      kernel/smp.o`<br>
`  CC      drivers/coresight/coresight-tpiu.o`<br>
`  CC      fs/ext3/ext3_jbd.o`<br>
`  CC      fs/ext4/hash.o`<br>
`  CC      kernel/trace/power-traces.o`<br>
`  CC      lib/audit.o`<br>
`  CC      net/core/sysctl_net_core.o`<br>
`  CC      fs/f2fs/acl.o`<br>
`  CC      net/core/dev.o`<br>
`  CC      net/core/ethtool.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-qti-pp-config.o`<br>
`  CC      fs/ext3/xattr.o`<br>
`  CC      sound/soc/msm/qdsp6v2/audio_calibration.o`<br>
`  CC      net/ipc_router/ipc_router_core.o`<br>
`  CC      kernel/trace/rpm-traces.o`<br>
`  CC      lib/compat_audit.o`<br>
`  CC      kernel/trace/ipc_logging.o`<br>
`  CC      drivers/coresight/coresight-nidnt.o`<br>
`  LD      fs/f2fs/f2fs.o`<br>
`  CC      fs/ext4/resize.o`<br>
`  CC      fs/ext4/extents.o`<br>
`  LD      fs/f2fs/built-in.o`<br>
`  CC      sound/soc/msm/qdsp6v2/audio_cal_utils.o`<br>
`  CC      fs/ext3/xattr_user.o`<br>
`  CC      lib/swiotlb.o`<br>
`  CC      kernel/trace/ipc_logging_debug.o`<br>
`  CC      lib/iommu-helper.o`<br>
`  CC      drivers/coresight/coresight-funnel.o`<br>
`  CC      drivers/coresight/coresight-replicator.o`<br>
`  CC      lib/syscall.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6adm.o`<br>
`  CC      fs/ext4/ext4_jbd2.o`<br>
`  CC      fs/ext4/migrate.o`<br>
`  CC      fs/ext3/xattr_trusted.o`<br>
`  LD      kernel/trace/libftrace.o`<br>
`  CC      drivers/coresight/coresight-stm.o`<br>
`  LD      kernel/trace/built-in.o`<br>
`  CC      fs/fuse/dev.o`<br>
`  CC      fs/fat/cache.o`<br>
`  CC      net/bridge/br_multicast.o`<br>
`  LD      fs/ext3/ext3.o`<br>
`  CC      lib/nlattr.o`<br>
`  CC      kernel/uid16.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6afe.o`<br>
`  CC      fs/fat/dir.o`<br>
`  CC      fs/ext4/mballoc.o`<br>
`  LD      fs/ext3/built-in.o`<br>
`  CC      fs/fuse/dir.o`<br>
`  CC      drivers/coresight/coresight-hwevent.o`<br>
`  CC      net/ipc_router/ipc_router_socket.o`<br>
`  CC      sound/soc/msm/msm-dai-fe.o`<br>
`  CC      kernel/system_keyring.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6asm.o`<br>
`  CC      lib/checksum.o`<br>
`  CC      fs/fat/fatent.o`<br>
`  LD      drivers/coresight/built-in.o`<br>
`  CC      fs/ext4/block_validity.o`<br>
`  CC      fs/fuse/file.o`<br>
`  CC      lib/cpu_rmap.o`<br>
`  CC      lib/dynamic_queue_limits.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6audio-v2.o`<br>
`  CC      kernel/kallsyms.o`<br>
`  CC      fs/fat/file.o`<br>
`  CC      drivers/cpufreq/cpufreq.o`<br>
`  CC      fs/ext4/move_extent.o`<br>
`  CC      fs/fuse/inode.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6voice.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6core.o`<br>
`  CC      lib/strncpy_from_user.o`<br>
`  CC      kernel/compat.o`<br>
`  CC      fs/fuse/control.o`<br>
`  LD      net/ethernet/built-in.o`<br>
`  CC      drivers/cpufreq/freq_table.o`<br>
`  CC      sound/soc/msm/qdsp6v2/rtac.o`<br>
`  CC      sound/soc/msm/qdsp6v2/q6lsm.o`<br>
`  CC      fs/fat/inode.o`<br>
`  CC      fs/ext4/mmp.o`<br>
`  CC      lib/strnlen_user.o`<br>
`  CC      kernel/cgroup.o`<br>
`  CC      fs/fuse/shortcircuit.o`<br>
`  CC      lib/net_utils.o`<br>
`  CC      sound/soc/msm/qdsp6v2/msm-pcm-q6-noirq.o`<br>
`  CC      drivers/cpufreq/cpufreq_stats.o`<br>
`  CC      fs/ext4/indirect.o`<br>
`  CC      fs/fat/misc.o`<br>
`  LD      net/bluetooth/rfcomm/rfcomm.o`<br>
`  LD      fs/fuse/fuse.o`<br>
`  CC      lib/asn1_decoder.o`<br>
`  CC      kernel/cgroup_freezer.o`<br>
`  CC      drivers/cpufreq/cpufreq_performance.o`<br>
`  LD      net/bluetooth/rfcomm/built-in.o`<br>
`  LD      fs/fuse/built-in.o`<br>
`  LD      sound/soc/msm/qdsp6v2/snd-soc-qdsp6v2.o`<br>
`  CC      drivers/cpufreq/cpufreq_powersave.o`<br>
`  LD      net/bluetooth/built-in.o`<br>
`  LD      sound/soc/msm/qdsp6v2/built-in.o`<br>
`  CC      fs/ext4/extents_status.o`<br>
`  CC      net/core/dev_addr_lists.o`<br>
`  CC      kernel/cpuset.o`<br>
`  CC      sound/usb/card.o`<br>
`  CC      sound/usb/clock.o`<br>
`  GEN     lib/oid_registry_data.c`<br>
`  CC      fs/fat/nfs.o`<br>
`perl: warning: Setting locale failed.`<br>
`perl: warning: Please check that your locale settings:`<br>
`	LANGUAGE = (unset),`<br>
`	LC_ALL = (unset),`<br>
`	LC_CTYPE = "en_US.UTF-8",`<br>
`	LC_COLLATE = "C",`<br>
`	LC_MESSAGES = "en_US.UTF-8",`<br>
`	LC_NUMERIC = "C",`<br>
`	LANG = (unset)`<br>
`    are supported and installed on your system.`<br>
`perl: warning: Falling back to the standard locale ("C").`<br>
`  CC      drivers/cpufreq/cpufreq_userspace.o`<br>
`  CC      lib/ucs2_string.o`<br>
`  CC      sound/soc/msm/msm-cpe-lsm.o`<br>
`  CC      fs/ext4/xattr.o`<br>
`  CC      sound/soc/msm/msm8952.o`<br>
`  CC      sound/usb/endpoint.o`<br>
`  CC      lib/qmi_encdec.o`<br>
`  CC      kernel/utsname.o`<br>
`  CC      lib/argv_split.o`<br>
`  CC      sound/soc/msm/msm-audio-pinctrl.o`<br>
`  CC      fs/fat/namei_vfat.o`<br>
`  CC      drivers/cpufreq/cpufreq_ondemand.o`<br>
`  CC      sound/soc/msm/msm8952-slimbus.o`<br>
`  CC      fs/ext4/xattr_user.o`<br>
`  CC      net/ipv4/route.o`<br>
`  CC      sound/usb/format.o`<br>
`  CC      kernel/pid_namespace.o`<br>
`  CC      fs/ext4/xattr_trusted.o`<br>
`  CC      lib/bug.o`<br>
`  CC      drivers/cpufreq/cpufreq_conservative.o`<br>
`  CC      sound/soc/msm/msm8952-dai-links.o`<br>
`  CC      sound/usb/helper.o`<br>
`  CC      fs/fat/namei_msdos.o`<br>
`  CC      lib/clz_tab.o`<br>
`  CC      fs/ext4/inline.o`<br>
`  CC      drivers/cpuidle/cpuidle.o`<br>
`  CC      drivers/cpufreq/cpufreq_interactive.o`<br>
`  CC      drivers/crypto/msm/qcedev.o`<br>
`  GZIP    kernel/config_data.gz`<br>
`  LD      fs/fat/fat.o`<br>
`  CC      lib/cmdline.o`<br>
`  LD      sound/soc/msm/snd-soc-hostless-pcm.o`<br>
`  LD      fs/fat/vfat.o`<br>
`  LD      sound/soc/msm/snd-soc-qdsp6v2.o`<br>
`  CC      kernel/res_counter.o`<br>
`  LD      fs/fat/msdos.o`<br>
`  LD      sound/soc/msm/snd-soc-cpe.o`<br>
`  LD      fs/fat/built-in.o`<br>
`  CC      drivers/cpufreq/cpufreq_governor.o`<br>
`  LD      sound/soc/msm/snd-soc-msm8x16.o`<br>
`  CC      fs/ext4/xattr_security.o`<br>
`  CC      lib/cpumask.o`<br>
`  CC      kernel/stop_machine.o`<br>
`  CC      drivers/cpuidle/driver.o`<br>
`  CC      drivers/crypto/msm/qce50.o`<br>
`  LD      sound/soc/msm/built-in.o`<br>
`  CC      sound/usb/mixer.o`<br>
`  CC      kernel/audit.o`<br>
`  CC      drivers/cpufreq/qcom-cpufreq.o`<br>
`  CC      net/ipc_router/ipc_router_security.o`<br>
`  CC      kernel/auditfilter.o`<br>
`  CC      drivers/crypto/msm/compat_qcedev.o`<br>
`  CC      lib/ctype.o`<br>
`  CC      sound/usb/mixer_quirks.o`<br>
`  LD      fs/ext4/ext4.o`<br>
`  LD      sound/soc/rockchip/built-in.o`<br>
`  CC      drivers/cpuidle/governor.o`<br>
`  LD      sound/soc/samsung/built-in.o`<br>
`  LD      sound/soc/sh/built-in.o`<br>
`  CC      lib/dec_and_lock.o`<br>
`  LD      fs/ext4/built-in.o`<br>
`  CC      kernel/auditsc.o`<br>
`  LD      sound/soc/sirf/built-in.o`<br>
`  CC      kernel/audit_watch.o`<br>
`  CC      lib/decompress.o`<br>
`  CC      drivers/cpuidle/sysfs.o`<br>
`  LD      drivers/cpufreq/built-in.o`<br>
`  CC      sound/usb/pcm.o`<br>
`  LD      sound/soc/spear/built-in.o`<br>
`  CC      drivers/crypto/msm/qcrypto.o`<br>
`  CC      lib/decompress_bunzip2.o`<br>
`  CC      sound/usb/proc.o`<br>
`  CC      kernel/audit_tree.o`<br>
`  LD      sound/soc/tegra/built-in.o`<br>
`  CC      fs/isofs/namei.o`<br>
`  CC      drivers/cpuidle/governors/ladder.o`<br>
`  CC      fs/isofs/inode.o`<br>
`  LD      sound/soc/txx9/built-in.o`<br>
`  CC      kernel/seccomp.o`<br>
`  CC      kernel/utsname_sysctl.o`<br>
`  CC      net/core/dst.o`<br>
`  CC      drivers/crypto/msm/ota_crypto.o`<br>
`  LD      sound/soc/ux500/built-in.o`<br>
`  CC      sound/usb/quirks.o`<br>
`  CC      lib/decompress_inflate.o`<br>
`  LD      sound/soc/snd-soc-core.o`<br>
`  CC      drivers/cpuidle/governors/menu.o`<br>
`  CC      kernel/tracepoint.o`<br>
`  LD      sound/soc/built-in.o`<br>
`  CC      sound/usb/stream.o`<br>
`  CC      fs/isofs/dir.o`<br>
`  CC      fs/isofs/util.o`<br>
`  CC      drivers/crypto/msm/ice.o`<br>
`  CC      lib/decompress_unlzma.o`<br>
`  CC      sound/last.o`<br>
`  CC      sound/usb/midi.o`<br>
`  CC      fs/isofs/rock.o`<br>
`  LD      drivers/cpuidle/governors/built-in.o`<br>
`  CC      drivers/cpuidle/lpm-levels.o`<br>
`  CC      kernel/elfcore.o`<br>
`  LD      sound/usb/6fire/built-in.o`<br>
`  CC      fs/jbd/transaction.o`<br>
`  CC      kernel/irq_work.o`<br>
`  LD      drivers/crypto/msm/built-in.o`<br>
`  CC      drivers/devfreq/devfreq.o`<br>
`  LD      drivers/crypto/built-in.o`<br>
`  CC      lib/dump_stack.o`<br>
`  LD      sound/usb/bcd2000/built-in.o`<br>
`  CC      net/bridge/br_mdb.o`<br>
`  CC      kernel/cpu_pm.o`<br>
`  LD      sound/usb/caiaq/built-in.o`<br>
`  CC      fs/jbd/commit.o`<br>
`  CC      drivers/cpuidle/lpm-levels-of.o`<br>
`  CERTS   kernel/x509_certificate_list`<br>
`  CC      fs/jbd2/transaction.o`<br>
`  CHK     kernel/config_data.h`<br>
`  CC      drivers/devfreq/devfreq_trace.o`<br>
`  CC      fs/isofs/export.o`<br>
`  LD      sound/usb/hiface/built-in.o`<br>
`  CC      lib/earlycpio.o`<br>
`  UPD     kernel/config_data.h`<br>
`  CC      drivers/dma/dmaengine.o`<br>
`  LD      sound/soundcore.o`<br>
`  AS      kernel/system_certificates.o`<br>
`  CC      fs/jbd/recovery.o`<br>
`  CC      kernel/configs.o`<br>
`  LD      sound/usb/misc/built-in.o`<br>
`  LD      sound/usb/usx2y/built-in.o`<br>
`  CC      drivers/cpuidle/lpm-workarounds.o`<br>
`  LD      sound/usb/snd-usb-audio.o`<br>
`  CC      fs/jbd2/commit.o`<br>
`  CC      drivers/devfreq/governor_simpleondemand.o`<br>
`  LD      fs/isofs/isofs.o`<br>
`  CC      drivers/devfreq/governor_performance.o`<br>
`  LD      sound/usb/snd-usbmidi-lib.o`<br>
`  LD      fs/isofs/built-in.o`<br>
`  CC      drivers/devfreq/governor_powersave.o`<br>
`  CC      fs/jbd/checkpoint.o`<br>
`  CC      lib/extable.o`<br>
`  CC      drivers/devfreq/governor_userspace.o`<br>
`  CC      drivers/dma/of-dma.o`<br>
`  LD      sound/usb/built-in.o`<br>
`  CC      fs/jbd2/recovery.o`<br>
`  LD      kernel/built-in.o`<br>
`  LD      sound/built-in.o`<br>
`  CC      drivers/dma/qcom-sps-dma.o`<br>
`  CC      lib/fdt.o`<br>
`  LD      drivers/cpuidle/built-in.o`<br>
`  CC      drivers/devfreq/governor_cpufreq.o`<br>
`  LD      drivers/dma/xilinx/built-in.o`<br>
`  CC      lib/fdt_empty_tree.o`<br>
`  CC      fs/jbd2/checkpoint.o`<br>
`  CC      net/core/netevent.o`<br>
`  CC      lib/fdt_ro.o`<br>
`  CC      fs/jbd/revoke.o`<br>
`  CC      lib/fdt_rw.o`<br>
`  LD      drivers/dma/built-in.o`<br>
`  CC      fs/jbd2/revoke.o`<br>
`  CC      drivers/dma-buf/dma-buf.o`<br>
`  CC      fs/jbd/journal.o`<br>
`  CC      lib/fdt_strerror.o`<br>
`  CC      lib/fdt_sw.o`<br>
`  CC      net/ipv4/inetpeer.o`<br>
`  CC      drivers/devfreq/governor_msm_adreno_tz.o`<br>
`  CC      lib/fdt_wip.o`<br>
`  CC      lib/flex_proportions.o`<br>
`  CC      fs/jbd2/journal.o`<br>
`  CC      drivers/dma-buf/fence.o`<br>
`  CC      lib/idr.o`<br>
`  CC      drivers/devfreq/bimc-bwmon.o`<br>
`  LD      fs/jbd/jbd.o`<br>
`  CC      lib/int_sqrt.o`<br>
`  LD      fs/jbd/built-in.o`<br>
`  CC      fs/kernfs/mount.o`<br>
`  CC      lib/ioremap.o`<br>
`  LD      net/ipc_router/built-in.o`<br>
`  CC      drivers/dma-buf/reservation.o`<br>
`  CC      lib/irq_regs.o`<br>
`  CC      net/core/neighbour.o`<br>
`  CC      drivers/edac/edac_stub.o`<br>
`  LD      fs/jbd2/jbd2.o`<br>
`  CC      fs/kernfs/inode.o`<br>
`  LD      fs/jbd2/built-in.o`<br>
`  CC      lib/is_single_threaded.o`<br>
`  LD      drivers/firewire/built-in.o`<br>
`  CC      drivers/devfreq/arm-memlat-mon.o`<br>
`  CC      lib/klist.o`<br>
`  CC      drivers/dma-buf/seqno-fence.o`<br>
`  CC      fs/kernfs/dir.o`<br>
`  CC      drivers/edac/edac_mc.o`<br>
`  CC      fs/lockd/clntlock.o`<br>
`  LD      drivers/dma-buf/built-in.o`<br>
`  CC      drivers/edac/edac_device.o`<br>
`  CC      lib/kobject.o`<br>
`  CC      drivers/firmware/dmi_scan.o`<br>
`  CC      drivers/devfreq/msmcci-hwmon.o`<br>
`  CC      fs/nfs_common/nfsacl.o`<br>
`  CC      fs/lockd/clntproc.o`<br>
`  CC      drivers/devfreq/m4m-hwmon.o`<br>
`  CC      fs/kernfs/file.o`<br>
`  CC      drivers/firmware/dmi-id.o`<br>
`  CC      fs/kernfs/symlink.o`<br>
`  CC      drivers/edac/edac_mc_sysfs.o`<br>
`  CC      fs/nfs_common/grace.o`<br>
`  CC      lib/kobject_uevent.o`<br>
`  CC      drivers/devfreq/governor_bw_hwmon.o`<br>
`  CC      fs/lockd/clntxdr.o`<br>
`  CC      lib/md5.o`<br>
`  LD      fs/kernfs/built-in.o`<br>
`  CC      drivers/edac/edac_pci_sysfs.o`<br>
`  CC      lib/plist.o`<br>
`  CC      net/bridge/br_netfilter.o`<br>
`  LD      fs/nfs_common/nfs_acl.o`<br>
`  CC      lib/proportions.o`<br>
`  CC      drivers/devfreq/governor_cache_hwmon.o`<br>
`  CC      net/core/rtnetlink.o`<br>
`  LD      fs/nfs_common/built-in.o`<br>
`  CC      net/ipv6/af_inet6.o`<br>
`  CC      lib/radix-tree.o`<br>
`  CC      drivers/firmware/efi/efi.o`<br>
`  CC      net/ipv4/protocol.o`<br>
`  CC      fs/lockd/host.o`<br>
`  CC      fs/nfs/client.o`<br>
`  CC      net/ipv4/ip_input.o`<br>
`  CC      net/ipv6/anycast.o`<br>
`  CC      fs/nls/nls_base.o`<br>
`  CC      drivers/firmware/efi/vars.o`<br>
`  CC      drivers/edac/edac_module.o`<br>
`  CC      drivers/devfreq/governor_gpubw_mon.o`<br>
`  CC      lib/ratelimit.o`<br>
`  CC      fs/lockd/svc.o`<br>
`  CC      fs/nls/nls_cp437.o`<br>
`  CC      drivers/firmware/efi/reboot.o`<br>
`  CC      fs/nfs/dir.o`<br>
`  CC      lib/rbtree.o`<br>
`  CC      drivers/edac/edac_device_sysfs.o`<br>
`  CC      drivers/devfreq/governor_bw_vbif.o`<br>
`  CC      drivers/firmware/efi/runtime-wrappers.o`<br>
`  CC      fs/nls/nls_cp936.o`<br>
`  CC      fs/lockd/svclock.o`<br>
`  CC      lib/sha1.o`<br>
`  CC      drivers/devfreq/governor_spdm_bw_hyp.o`<br>
`  CC      drivers/edac/edac_pci.o`<br>
`  CC      fs/nls/nls_cp950.o`<br>
`  CC      fs/nfs/file.o`<br>
`  CC      net/ipv4/ip_fragment.o`<br>
`  LD      drivers/firmware/efi/built-in.o`<br>
`  CC      drivers/edac/cortex_arm64_edac.o`<br>
`  CC      lib/show_mem.o`<br>
`  CC      fs/lockd/svcshare.o`<br>
`  CC      drivers/firmware/qcom/tz_log.o`<br>
`  CC      fs/nfs/getroot.o`<br>
`  CC      fs/nls/nls_ascii.o`<br>
`  CC      drivers/devfreq/governor_memlat.o`<br>
`  LD      drivers/firmware/qcom/built-in.o`<br>
`  LD      drivers/edac/edac_core.o`<br>
`  LD      drivers/firmware/built-in.o`<br>
`  CC      fs/lockd/svcproc.o`<br>
`  CC      lib/string.o`<br>
`  LD      drivers/edac/built-in.o`<br>
`  CC      lib/timerqueue.o`<br>
`  CC      fs/nls/nls_iso8859-1.o`<br>
`  CC      drivers/devfreq/devfreq_devbw.o`<br>
`  CC      fs/nfs/inode.o`<br>
`  CC      fs/lockd/svcsubs.o`<br>
`  CC      drivers/gpio/devres.o`<br>
`  CC      lib/vsprintf.o`<br>
`  CC      fs/nls/nls_utf8.o`<br>
`  CC      fs/nfs/super.o`<br>
`  LD      drivers/gpu/drm/bridge/built-in.o`<br>
`  CC      drivers/devfreq/devfreq_simple_dev.o`<br>
`  LD      drivers/gpu/drm/i2c/built-in.o`<br>
`  CC      drivers/gpio/gpiolib.o`<br>
`  GEN     lib/crc32table.h`<br>
`  CC      lib/oid_registry.o`<br>
`  AR      lib/lib.a`<br>
`  LD      drivers/gpu/drm/panel/built-in.o`<br>
`  CC      fs/lockd/mon.o`<br>
`  LD      fs/nls/built-in.o`<br>
`  CC      fs/nfs/direct.o`<br>
`  LD      drivers/gpu/drm/built-in.o`<br>
`  CC      lib/crc32.o`<br>
`  CC      drivers/devfreq/devfreq_spdm.o`<br>
`  CC      drivers/gpio/gpiolib-legacy.o`<br>
`  CC      net/core/utils.o`<br>
`  CC      fs/nfs/pagelist.o`<br>
`  CC      drivers/gpio/gpiolib-of.o`<br>
`  CC      fs/lockd/xdr.o`<br>
`  CC      drivers/devfreq/devfreq_spdm_debugfs.o`<br>
`  LD      lib/built-in.o`<br>
`  CC      drivers/gpio/gpiolib-sysfs.o`<br>
`  CC      drivers/hid/hid-debug.o`<br>
`  CC      drivers/gpio/qpnp-pin.o`<br>
`  CC      net/ipv6/ip6_output.o`<br>
`  CC      fs/nfs/read.o`<br>
`  CC      fs/nfs/symlink.o`<br>
`  LD      drivers/devfreq/built-in.o`<br>
`  CC      drivers/gpio/gpio-msm-smp2p.o`<br>
`  CC      drivers/hid/hid-core.o`<br>
`  CC      fs/nfs/unlink.o`<br>
`  CC      net/ipv4/ip_forward.o`<br>
`  CC      drivers/gpio/gpio-msm-smp2p-test.o`<br>
`  CC      net/ipv6/ip6_input.o`<br>
`  CC      net/key/af_key.o`<br>
`  CC      fs/lockd/clnt4xdr.o`<br>
`  CC      fs/lockd/xdr4.o`<br>
`  CC      fs/nfs/write.o`<br>
`  CC      drivers/hid/hid-input.o`<br>
`  CC      drivers/gpu/vga/vgaarb.o`<br>
`  LD      drivers/gpio/built-in.o`<br>
`  CC      net/core/link_watch.o`<br>
`  CC      drivers/hid/hidraw.o`<br>
`  CC      fs/lockd/svc4proc.o`<br>
`  LD      drivers/gpu/vga/built-in.o`<br>
`  LD      drivers/hsi/clients/built-in.o`<br>
`  CC      fs/lockd/procfs.o`<br>
`  CC      fs/nfs/namespace.o`<br>
`  LD      drivers/hsi/controllers/built-in.o`<br>
`  LD      drivers/hsi/built-in.o`<br>
`  CC      drivers/hid/uhid.o`<br>
`  CC      drivers/gpu/msm/kgsl.o`<br>
`  CC      drivers/hid/hid-generic.o`<br>
`  CC      net/ipv4/ip_options.o`<br>
`  CC      net/ipv4/ip_output.o`<br>
`  CC      drivers/hid/hid-apple.o`<br>
`  CC      fs/nfs/mount_clnt.o`<br>
`  LD      fs/lockd/lockd.o`<br>
`  CC      drivers/gpu/msm/kgsl_trace.o`<br>
`  CC      net/bridge/netfilter/ebtables.o`<br>
`  LD      fs/lockd/built-in.o`<br>
`  CC      net/bridge/netfilter/ebtable_broute.o`<br>
`  CC      fs/nfs/nfstrace.o`<br>
`  CC      drivers/hid/hid-elecom.o`<br>
`  CC      net/ipv4/ip_sockglue.o`<br>
`  CC      drivers/gpu/msm/kgsl_cmdbatch.o`<br>
`  CC      drivers/hid/hid-magicmouse.o`<br>
`  CC      fs/nfs/sysctl.o`<br>
`  CC      drivers/gpu/msm/kgsl_ioctl.o`<br>
`  CC      drivers/gpu/msm/kgsl_sharedmem.o`<br>
`  CC      drivers/hid/hid-microsoft.o`<br>
`  CC      drivers/gpu/msm/kgsl_pwrctrl.o`<br>
`  CC      fs/nfs/nfs2super.o`<br>
`  CC      drivers/hid/hid-multitouch.o`<br>
`  CC      fs/nfs/proc.o`<br>
`  CC      net/core/filter.o`<br>
`  CC      drivers/gpu/msm/kgsl_pwrscale.o`<br>
`  CC      drivers/gpu/msm/kgsl_mmu.o`<br>
`  CC      drivers/gpu/msm/kgsl_snapshot.o`<br>
`  CC      net/core/sock_diag.o`<br>
`  CC      drivers/gpu/msm/kgsl_events.o`<br>
`  CC      fs/nfs/nfs2xdr.o`<br>
`  CC      drivers/hid/usbhid/hid-core.o`<br>
`  CC      drivers/gpu/msm/kgsl_pool.o`<br>
`  CC      drivers/gpu/msm/kgsl_iommu.o`<br>
`  CC      drivers/gpu/msm/kgsl_debugfs.o`<br>
`  CC      net/ipv4/inet_hashtables.o`<br>
`  CC      drivers/hid/usbhid/hid-quirks.o`<br>
`  CC      fs/nfs/nfs3super.o`<br>
`  CC      drivers/gpu/msm/kgsl_sync.o`<br>
`  CC      drivers/gpu/msm/kgsl_compat.o`<br>
`  CC      drivers/gpu/msm/adreno_ioctl.o`<br>
`  CC      drivers/gpu/msm/adreno_ringbuffer.o`<br>
`  CC      drivers/hid/usbhid/hiddev.o`<br>
`  CC      drivers/gpu/msm/adreno_drawctxt.o`<br>
`  CC      fs/nfs/nfs3client.o`<br>
`  CC      drivers/gpu/msm/adreno_dispatch.o`<br>
`  CC      drivers/gpu/msm/adreno_snapshot.o`<br>
`  LD      drivers/hid/usbhid/usbhid.o`<br>
`  CC      drivers/gpu/msm/adreno_coresight.o`<br>
`  CC      drivers/gpu/msm/adreno_trace.o`<br>
`  LD      drivers/hid/usbhid/built-in.o`<br>
`  CC      drivers/gpu/msm/adreno_a3xx.o`<br>
`  LD      drivers/hid/hid.o`<br>
`  CC      fs/nfs/nfs3proc.o`<br>
`  LD      drivers/hid/built-in.o`<br>
`  CC      net/ipv6/addrconf.o`<br>
`  CC      fs/nfs/nfs3xdr.o`<br>
`  CC      drivers/gpu/msm/adreno_a4xx.o`<br>
`  CC      fs/nfs/nfs3acl.o`<br>
`  CC      fs/nfs/nfs4proc.o`<br>
`  CC      drivers/gpu/msm/adreno_a5xx.o`<br>
`  CC      fs/nfs/nfs4xdr.o`<br>
`  CC      drivers/hwmon/hwmon.o`<br>
`  CC      fs/nfs/nfs4state.o`<br>
`  CC      drivers/gpu/msm/adreno_a3xx_snapshot.o`<br>
`  CC      fs/nfs/nfs4renewd.o`<br>
`  CC      drivers/hwmon/epm_adc.o`<br>
`  CC      fs/nfs/nfs4super.o`<br>
`  CC      fs/nfs/nfs4file.o`<br>
`  CC      drivers/gpu/msm/adreno_a4xx_snapshot.o`<br>
`  CC      fs/nfs/delegation.o`<br>
`  CC      fs/nfs/idmap.o`<br>
`  CC      fs/nfs/callback.o`<br>
`  CC      drivers/hwmon/qpnp-adc-voltage.o`<br>
`  CC      fs/nfs/callback_xdr.o`<br>
`  CC      drivers/gpu/msm/adreno_a5xx_snapshot.o`<br>
`  CC      fs/nfs/callback_proc.o`<br>
`  CC      net/core/dev_ioctl.o`<br>
`  LD      net/key/built-in.o`<br>
`  CC      drivers/gpu/msm/adreno_a4xx_preempt.o`<br>
`  CC      fs/nfs/nfs4namespace.o`<br>
`  CC      net/ipv4/inet_timewait_sock.o`<br>
`  CC      drivers/hwmon/qpnp-adc-common.o`<br>
`  CC      net/llc/llc_core.o`<br>
`  CC      net/l2tp/l2tp_core.o`<br>
`  CC      fs/nfs/nfs4getroot.o`<br>
`  CC      fs/nfs/nfs4client.o`<br>
`  CC      net/ipv4/inet_connection_sock.o`<br>
`  CC      drivers/hwmon/qpnp-adc-current.o`<br>
`  CC      drivers/gpu/msm/adreno_a5xx_preempt.o`<br>
`  CC      drivers/gpu/msm/adreno_sysfs.o`<br>
`  CC      drivers/gpu/msm/adreno.o`<br>
`  CC      net/core/tso.o`<br>
`  CC      fs/nfs/nfs4session.o`<br>
`  LD      drivers/hwmon/built-in.o`<br>
`  CC      net/ipv4/tcp.o`<br>
`  CC      drivers/hwspinlock/hwspinlock_core.o`<br>
`  CC      drivers/gpu/msm/adreno_cp_parser.o`<br>
`  CC      drivers/gpu/msm/adreno_perfcounter.o`<br>
`  CC      fs/nfs/dns_resolve.o`<br>
`  CC      fs/nfs/nfs4trace.o`<br>
`  CC      net/ipv4/tcp_input.o`<br>
`  CC      drivers/hwspinlock/msm_remote_spinlock.o`<br>
`  CC      fs/nfs/nfs4sysctl.o`<br>
`  LD      net/bridge/netfilter/built-in.o`<br>
`  LD      net/bridge/bridge.o`<br>
`  CC      drivers/gpu/msm/adreno_iommu.o`<br>
`  CC      drivers/gpu/msm/adreno_debugfs.o`<br>
`  CC      fs/nfs/pnfs.o`<br>
`  LD      drivers/hwspinlock/built-in.o`<br>
`  LD      net/bridge/built-in.o`<br>
`  CC      net/ipv6/addrlabel.o`<br>
`  CC      fs/nfs/pnfs_dev.o`<br>
`  CC      drivers/i2c/i2c-boardinfo.o`<br>
`  CC      drivers/gpu/msm/adreno_profile.o`<br>
`  CC      drivers/gpu/msm/adreno_compat.o`<br>
`  CC      fs/nfs/blocklayout/blocklayout.o`<br>
`  LD      drivers/gpu/msm/msm_kgsl_core.o`<br>
`  CC      fs/nfs/filelayout/filelayout.o`<br>
`  CC      drivers/i2c/i2c-core.o`<br>
`  CC      drivers/i2c/i2c-dev.o`<br>
`  LD      drivers/gpu/msm/msm_adreno.o`<br>
`  CC      fs/nfs/blocklayout/dev.o`<br>
`  LD      drivers/gpu/msm/built-in.o`<br>
`  CC      fs/nfs/blocklayout/extent_tree.o`<br>
`  CC      fs/nfs/filelayout/filelayoutdev.o`<br>
`  CC      drivers/i2c/i2c-mux.o`<br>
`  LD      drivers/gpu/built-in.o`<br>
`  LD      drivers/i2c/algos/built-in.o`<br>
`  CC      fs/nfs/blocklayout/rpc_pipefs.o`<br>
`  CC      net/ipv4/tcp_output.o`<br>
`  LD      fs/nfs/filelayout/nfs_layout_nfsv41_files.o`<br>
`  CC      drivers/i2c/busses/i2c-msm-v2.o`<br>
`  LD      fs/nfs/filelayout/built-in.o`<br>
`  LD      drivers/idle/built-in.o`<br>
`  CC      net/core/flow.o`<br>
`  CC      net/ipv4/tcp_timer.o`<br>
`  LD      drivers/i2c/muxes/built-in.o`<br>
`  CC      net/ipv4/tcp_ipv4.o`<br>
`  CC      net/llc/llc_input.o`<br>
`  CC      net/ipv4/tcp_minisocks.o`<br>
`  LD      drivers/i2c/busses/built-in.o`<br>
`  LD      fs/nfs/blocklayout/blocklayoutdriver.o`<br>
`  LD      drivers/i2c/built-in.o`<br>
`  LD      fs/nfs/blocklayout/built-in.o`<br>
`  CC      net/core/net-sysfs.o`<br>
`  LD      fs/nfs/nfs.o`<br>
`  LD      fs/nfs/nfsv2.o`<br>
`  LD      fs/nfs/nfsv3.o`<br>
`  LD      fs/nfs/nfsv4.o`<br>
`  CC      net/ipv6/route.o`<br>
`  CC      drivers/input/input.o`<br>
`  LD      fs/nfs/built-in.o`<br>
`  CC      drivers/input/input-compat.o`<br>
`  CC      drivers/input/input-mt.o`<br>
`  CC      net/netfilter/core.o`<br>
`  CC      drivers/input/ff-core.o`<br>
`  CC      drivers/input/mousedev.o`<br>
`  CC      fs/notify/fsnotify.o`<br>
`  CC      net/l2tp/l2tp_ppp.o`<br>
`  CC      net/core/net-procfs.o`<br>
`  CC      drivers/input/evdev.o`<br>
`  CC      fs/notify/notification.o`<br>
`  CC      drivers/input/fingerprint/fpc/fpc1020_tee.o`<br>
`  CC      drivers/input/joystick/xpad.o`<br>
`  CC      net/llc/llc_output.o`<br>
`  CC      fs/notify/group.o`<br>
`  CC      net/ipv4/tcp_cong.o`<br>
`  LD      drivers/input/fingerprint/fpc/built-in.o`<br>
`  CC      drivers/input/fingerprint/goodix/gf_spi.o`<br>
`  LD      drivers/input/joystick/built-in.o`<br>
`  CC      drivers/input/fingerprint/goodix/platform.o`<br>
`  CC      fs/notify/inode_mark.o`<br>
`  CC      drivers/input/fingerprint/goodix/netlink.o`<br>
`  CC      fs/notify/mark.o`<br>
`  CC      drivers/input/keyboard/atkbd.o`<br>
`  CC      net/ipv6/ip6_fib.o`<br>
`  LD      drivers/input/fingerprint/goodix/built-in.o`<br>
`  CC      fs/notify/vfsmount_mark.o`<br>
`  CC      net/ipv6/ipv6_sockglue.o`<br>
`  LD      drivers/input/fingerprint/built-in.o`<br>
`  CC      drivers/input/keyboard/gpio_keys.o`<br>
`  LD      drivers/input/tablet/built-in.o`<br>
`  CC      fs/notify/fdinfo.o`<br>
`  LD      net/llc/llc.o`<br>
`  CC      fs/notify/dnotify/dnotify.o`<br>
`  CC      drivers/input/misc/gpio_event.o`<br>
`  CC      net/netfilter/nf_log.o`<br>
`  LD      net/llc/built-in.o`<br>
`  LD      drivers/input/keyboard/built-in.o`<br>
`  CC      drivers/input/misc/gpio_matrix.o`<br>
`  CC      drivers/input/misc/gpio_input.o`<br>
`  CC      net/netlink/af_netlink.o`<br>
`  LD      fs/notify/dnotify/built-in.o`<br>
`  CC      net/ipv6/ndisc.o`<br>
`  CC      fs/notify/fanotify/fanotify.o`<br>
`  CC      net/core/fib_rules.o`<br>
`  CC      net/netlink/genetlink.o`<br>
`  CC      drivers/input/touchscreen/of_touchscreen.o`<br>
`  CC      drivers/input/misc/gpio_output.o`<br>
`  CC      fs/notify/fanotify/fanotify_user.o`<br>
`  CC      drivers/input/touchscreen/ft5435/ft5435_ts.o`<br>
`  CC      net/ipv4/tcp_metrics.o`<br>
`  CC      drivers/input/misc/gpio_axis.o`<br>
`  LD      fs/notify/fanotify/built-in.o`<br>
`  CC      drivers/input/touchscreen/gt9xx_mido/gt9xx.o`<br>
`  CC      net/ipv4/tcp_fastopen.o`<br>
`  CC      fs/notify/inotify/inotify_fsnotify.o`<br>
`  CC      drivers/input/misc/hbtp_input.o`<br>
`  CC      net/l2tp/l2tp_ip.o`<br>
`  LD      drivers/input/touchscreen/ft5435/built-in.o`<br>
`  CC      net/ipv4/tcp_offload.o`<br>
`  CC      fs/notify/inotify/inotify_user.o`<br>
`  CC      drivers/input/misc/hbtp_vm.o`<br>
`  CC      drivers/input/touchscreen/gt9xx_mido/gt9xx_update.o`<br>
`  LD      fs/notify/inotify/built-in.o`<br>
`  LD      fs/notify/built-in.o`<br>
`  CC      drivers/input/misc/keychord.o`<br>
`  CC      drivers/input/touchscreen/gt9xx_mido/goodix_tool.o`<br>
`  LD      drivers/input/touchscreen/gt9xx_mido/built-in.o`<br>
`  CC      net/ipv6/udp.o`<br>
`  CC      drivers/input/misc/uinput.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc.o`<br>
`  CC      net/ipv4/datagram.o`<br>
`  CC      net/netfilter/nf_queue.o`<br>
`  CC      fs/proc/task_mmu.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc_misc.o`<br>
`  LD      drivers/input/misc/built-in.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc_sys.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc_update.o`<br>
`  CC      fs/proc/inode.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc_tracking.o`<br>
`  CC      drivers/input/touchscreen/ist3038c/ist30xxc_cmcs.o`<br>
`  CC      fs/proc/root.o`<br>
`  CC      net/core/net-traces.o`<br>
`  CC      fs/proc/base.o`<br>
`  LD      drivers/input/touchscreen/ist3038c/built-in.o`<br>
`  CC      net/core/netclassid_cgroup.o`<br>
`  CC      fs/proc/generic.o`<br>
`  CC      drivers/input/touchscreen/synaptics_dsx_2.6/synaptics_dsx_i2c.o`<br>
`  CC      drivers/input/touchscreen/synaptics_dsx/synaptics_dsx_i2c.o`<br>
`  CC      net/ipv6/udplite.o`<br>
`  CC      net/ipv6/raw.o`<br>
`  LD      drivers/input/touchscreen/synaptics_dsx/built-in.o`<br>
`  LD      drivers/input/touchscreen/synaptics_dsx_2.6/built-in.o`<br>
`  CC      fs/proc/array.o`<br>
`  LD      drivers/input/touchscreen/built-in.o`<br>
`  CC      net/packet/af_packet.o`<br>
`  CC      drivers/input/keyreset.o`<br>
`  CC      net/ipv6/icmp.o`<br>
`  CC      fs/proc/fd.o`<br>
`  CC      net/ipv4/udp.o`<br>
`  CC      net/ipv4/raw.o`<br>
`  CC      net/l2tp/l2tp_netlink.o`<br>
`  CC      drivers/input/keycombo.o`<br>
`  CC      fs/proc/proc_tty.o`<br>
`  CC      fs/proc/cmdline.o`<br>
`  LD      net/netlink/built-in.o`<br>
`  CC      fs/proc/consoles.o`<br>
`  CC      fs/proc/cpuinfo.o`<br>
`  CC      net/netfilter/nf_sockopt.o`<br>
`  LD      drivers/input/input-core.o`<br>
`  CC      drivers/input/serio/serio.o`<br>
`  LD      drivers/input/built-in.o`<br>
`  CC      net/netfilter/nfnetlink.o`<br>
`  CC      fs/proc/devices.o`<br>
`  CC      net/netfilter/nfnetlink_acct.o`<br>
`  CC      net/netfilter/nfnetlink_queue_core.o`<br>
`  CC      drivers/input/serio/serport.o`<br>
`  CC      fs/proc/interrupts.o`<br>
`  CC      drivers/input/serio/libps2.o`<br>
`  CC      net/core/sockev_nlmcast.o`<br>
`  LD      drivers/input/serio/built-in.o`<br>
`  CC      fs/proc/loadavg.o`<br>
`  CC      fs/proc/meminfo.o`<br>
`  CC      drivers/iommu/iommu.o`<br>
`  CC      fs/proc/stat.o`<br>
`  CC      net/l2tp/l2tp_eth.o`<br>
`  CC      drivers/iommu/iommu-traces.o`<br>
`  CC      fs/proc/uptime.o`<br>
`  CC      fs/proc/version.o`<br>
`  CC      drivers/iommu/iommu-sysfs.o`<br>
`  CC      net/ipv6/mcast.o`<br>
`  CC      fs/proc/softirqs.o`<br>
`  CC      drivers/iommu/msm_dma_iommu_mapping.o`<br>
`  CC      fs/proc/namespaces.o`<br>
`  CC      fs/proc/self.o`<br>
`  CC      drivers/iommu/io-pgtable.o`<br>
`  CC      drivers/iommu/io-pgtable-arm.o`<br>
`  CC      fs/proc/thread_self.o`<br>
`  CC      fs/proc/proc_sysctl.o`<br>
`  CC      drivers/iommu/io-pgtable-msm-secure.o`<br>
`  CC      drivers/irqchip/irqchip.o`<br>
`  CC      drivers/iommu/of_iommu.o`<br>
`  CC      drivers/iommu/iommu-debug.o`<br>
`  CC      fs/proc/proc_net.o`<br>
`  CC      drivers/iommu/msm_iommu.o`<br>
`  CC      net/netfilter/nfnetlink_log.o`<br>
`  CC      drivers/iommu/msm_iommu_domains.o`<br>
`  CC      drivers/iommu/msm_iommu_mapping.o`<br>
`  LD      net/core/built-in.o`<br>
`  CC      drivers/iommu/msm_iommu-v1.o`<br>
`  CC      fs/proc/kmsg.o`<br>
`  CC      drivers/irqchip/irq-gic.o`<br>
`  CC      fs/proc/page.o`<br>
`  CC      drivers/iommu/msm_iommu_dev-v1.o`<br>
`  CC      drivers/iommu/msm_iommu_sec.o`<br>
`  CC      net/netfilter/nf_conntrack_core.o`<br>
`  CC      drivers/irqchip/irq-gic-common.o`<br>
`  CC      drivers/iommu/msm_iommu_pagetable.o`<br>
`  CC      drivers/irqchip/irq-gic-v2m.o`<br>
`  CC      net/l2tp/l2tp_debugfs.o`<br>
`  LD      fs/proc/proc.o`<br>
`  CC      drivers/leds/led-core.o`<br>
`  CC      drivers/leds/led-class.o`<br>
`  CC      drivers/irqchip/irq-gic-v3.o`<br>
`  CC      fs/quota/dquot.o`<br>
`  LD      fs/proc/built-in.o`<br>
`  CC      fs/pstore/inode.o`<br>
`  CC      net/netfilter/nf_conntrack_standalone.o`<br>
`  CC      drivers/iommu/arm-smmu.o`<br>
`  CC      net/netfilter/nf_conntrack_expect.o`<br>
`  CC      net/l2tp/l2tp_ip6.o`<br>
`  CC      drivers/irqchip/irq-gic-v3-its.o`<br>
`  CC      fs/quota/quota.o`<br>
`  CC      fs/pstore/platform.o`<br>
`  CC      drivers/leds/led-triggers.o`<br>
`  LD      drivers/iommu/built-in.o`<br>
`  LD      drivers/lguest/built-in.o`<br>
`  CC      drivers/leds/leds-qpnp.o`<br>
`  CC      net/netfilter/nf_conntrack_helper.o`<br>
`  CC      net/netfilter/nf_conntrack_proto.o`<br>
`  CC      fs/pstore/ftrace.o`<br>
`  CC      drivers/irqchip/irq-gic-v3-its-pci-msi.o`<br>
`  CC      fs/quota/kqid.o`<br>
`  LD      drivers/macintosh/built-in.o`<br>
`  CC      drivers/irqchip/irq-msm.o`<br>
`  CC      drivers/leds/leds-qpnp-flash.o`<br>
`  CC      fs/pstore/ram.o`<br>
`  CC      drivers/irqchip/msm_show_resume_irq.o`<br>
`  LD      fs/quota/built-in.o`<br>
`  CC      drivers/leds/leds-qpnp-wled.o`<br>
`  LD      drivers/irqchip/built-in.o`<br>
`  CC      fs/pstore/ram_core.o`<br>
`  CC      fs/ramfs/inode.o`<br>
`  CC      drivers/leds/leds-aw2013.o`<br>
`  LD      fs/pstore/pstore.o`<br>
`  CC      fs/ramfs/file-mmu.o`<br>
`  CC      net/ipv4/udplite.o`<br>
`  LD      fs/pstore/ramoops.o`<br>
`  LD      fs/pstore/built-in.o`<br>
`  CC      drivers/md/dm-uevent.o`<br>
`  CC      drivers/md/dm.o`<br>
`  LD      drivers/leds/trigger/built-in.o`<br>
`  LD      fs/ramfs/ramfs.o`<br>
`  LD      fs/ramfs/built-in.o`<br>
`  LD      drivers/leds/built-in.o`<br>
`  LD      net/packet/built-in.o`<br>
`  CC      drivers/md/dm-table.o`<br>
`  CC      drivers/md/dm-target.o`<br>
`  CC      drivers/md/dm-linear.o`<br>
`  CC      fs/sdcardfs/dentry.o`<br>
`  CC      drivers/mfd/mfd-core.o`<br>
`  LD      drivers/media/firewire/built-in.o`<br>
`  CC      drivers/md/dm-stripe.o`<br>
`  CC      drivers/md/dm-ioctl.o`<br>
`  LD      drivers/media/common/b2c2/built-in.o`<br>
`  CC      drivers/md/dm-io.o`<br>
`  CC      net/netfilter/nf_conntrack_l3proto_generic.o`<br>
`  CC      fs/sdcardfs/file.o`<br>
`  LD      drivers/media/common/saa7146/built-in.o`<br>
`  CC      drivers/mfd/wcd9xxx-core.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_generic.o`<br>
`  LD      drivers/media/common/siano/built-in.o`<br>
`  LD      drivers/media/i2c/soc_camera/built-in.o`<br>
`  CC      drivers/media/i2c/ir-kbd-i2c.o`<br>
`  CC      drivers/md/dm-kcopyd.o`<br>
`  CC      fs/sdcardfs/inode.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_tcp.o`<br>
`  CC      drivers/md/dm-sysfs.o`<br>
`  LD      drivers/media/common/built-in.o`<br>
`  CC      drivers/md/dm-stats.o`<br>
`  LD      net/l2tp/built-in.o`<br>
`  LD      drivers/media/mmc/siano/built-in.o`<br>
`  CC      drivers/mfd/wcd9xxx-irq.o`<br>
`  CC      fs/sdcardfs/main.o`<br>
`  LD      drivers/media/mmc/built-in.o`<br>
`  CC      drivers/md/dm-builtin.o`<br>
`  CC      drivers/md/dm-bufio.o`<br>
`  CC      net/rfkill/core.o`<br>
`  CC      drivers/md/dm-crypt.o`<br>
`  CC      drivers/md/dm-verity-fec.o`<br>
`  LD      drivers/media/i2c/built-in.o`<br>
`  CC      net/ipv6/reassembly.o`<br>
`  CC      fs/sdcardfs/super.o`<br>
`  LD      drivers/media/parport/built-in.o`<br>
`  CC      drivers/mfd/wcd9xxx-slimslave.o`<br>
`  CC      drivers/mfd/wcd9xxx-core-resource.o`<br>
`  CC      drivers/md/dm-verity-target.o`<br>
`  CC      net/ipv6/tcp_ipv6.o`<br>
`  CC      net/ipv6/ping.o`<br>
`  CC      fs/sdcardfs/lookup.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_udp.o`<br>
`  CC      drivers/mfd/wcd9330-regmap.o`<br>
`  CC      drivers/mfd/wcd9335-regmap.o`<br>
`  CC      drivers/md/dm-req-crypt.o`<br>
`  CC      fs/sdcardfs/mmap.o`<br>
`  CC      net/ipv6/exthdrs.o`<br>
`  CC      net/rmnet_data/rmnet_data_main.o`<br>
`  CC      drivers/mfd/wcd9335-tables.o`<br>
`  CC      fs/sdcardfs/packagelist.o`<br>
`  LD      drivers/md/dm-mod.o`<br>
`  LD      drivers/md/dm-verity.o`<br>
`  LD      drivers/media/pci/b2c2/built-in.o`<br>
`  CC      net/ipv4/udp_offload.o`<br>
`  CC      drivers/mfd/wcd-gpio-ctrl.o`<br>
`  LD      drivers/md/built-in.o`<br>
`  CC      net/ipv4/arp.o`<br>
`  CC      fs/sdcardfs/derived_perm.o`<br>
`  LD      drivers/media/pci/ddbridge/built-in.o`<br>
`  LD      drivers/media/pci/dm1105/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_extend.o`<br>
`  LD      fs/sdcardfs/sdcardfs.o`<br>
`  LD      fs/sdcardfs/built-in.o`<br>
`  LD      drivers/media/pci/mantis/built-in.o`<br>
`  LD      drivers/mfd/built-in.o`<br>
`  LD      drivers/media/pci/ngene/built-in.o`<br>
`  CC      fs/sysfs/file.o`<br>
`  CC      drivers/misc/uid_stat.o`<br>
`  LD      drivers/media/pci/pluto2/built-in.o`<br>
`  LD      drivers/media/pci/pt1/built-in.o`<br>
`  CC      fs/sysfs/dir.o`<br>
`  CC      drivers/mmc/card/block.o`<br>
`  LD      drivers/media/pci/pt3/built-in.o`<br>
`  LD      drivers/misc/carma/built-in.o`<br>
`  LD      drivers/media/pci/saa7146/built-in.o`<br>
`  LD      drivers/misc/eeprom/built-in.o`<br>
`  LD      drivers/misc/cb710/built-in.o`<br>
`  CC      drivers/mmc/card/queue.o`<br>
`  CC      fs/sysfs/symlink.o`<br>
`  LD      drivers/media/pci/ttpci/built-in.o`<br>
`  CC      net/rmnet_data/rmnet_data_config.o`<br>
`  LD      drivers/media/pci/built-in.o`<br>
`  LD      drivers/misc/lis3lv02d/built-in.o`<br>
`  LD      net/rfkill/rfkill.o`<br>
`  LD      drivers/mmc/card/mmc_block.o`<br>
`  LD      drivers/misc/mic/built-in.o`<br>
`  LD      net/rfkill/built-in.o`<br>
`  CC      fs/sysfs/mount.o`<br>
`  LD      drivers/mmc/card/built-in.o`<br>
`  LD      drivers/misc/ti-st/built-in.o`<br>
`  CC      drivers/misc/qseecom.o`<br>
`  LD      drivers/media/platform/msm/broadcast/built-in.o`<br>
`  CC      fs/sysfs/group.o`<br>
`  CC      drivers/misc/hdcp.o`<br>
`  CC      drivers/mmc/core/core.o`<br>
`  CC      net/netfilter/nf_conntrack_acct.o`<br>
`  CC      drivers/mmc/core/bus.o`<br>
`  LD      fs/sysfs/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_seqadj.o`<br>
`  CC      net/rmnet_data/rmnet_data_vnd.o`<br>
`  LD      drivers/media/platform/omap/built-in.o`<br>
`  CC      net/rmnet_data/rmnet_data_handlers.o`<br>
`  CC      drivers/media/platform/soc_camera/soc_camera.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/aac_in.o`<br>
`  CC      drivers/mmc/core/host.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/camera/camera.o`<br>
`  CC      fs/udf/balloc.o`<br>
`  CC      drivers/media/platform/soc_camera/soc_mediabus.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/qcelp_in.o`<br>
`  CC      drivers/mmc/core/mmc.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/camera/built-in.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/evrc_in.o`<br>
`  CC      fs/udf/dir.o`<br>
`  CC      drivers/media/platform/soc_camera/soc_camera_platform.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/amrnb_in.o`<br>
`  CC      fs/udf/file.o`<br>
`  CC      drivers/mmc/core/mmc_ops.o`<br>
`  CC      net/netfilter/nf_conntrack_ecache.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/common/msm_camera_io_util.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/common/cam_smmu_api.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/g711mlaw_in.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/g711alaw_in.o`<br>
`  LD      drivers/media/platform/soc_camera/built-in.o`<br>
`  CC      fs/udf/ialloc.o`<br>
`  CC      fs/udf/inode.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/common/cam_hw_ops.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_utils.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/common/cam_soc_api.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_wma.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_wmapro.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_aac.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_multi_aac.o`<br>
`  CC      fs/udf/lowlevel.o`<br>
`  CC      net/ipv6/datagram.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_alac.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_dccp.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_ape.o`<br>
`  CC      fs/udf/namei.o`<br>
`  CC      fs/udf/partition.o`<br>
`  CC      drivers/misc/compat_qseecom.o`<br>
`  CC      net/ipv4/icmp.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_utils_aio.o`<br>
`  CC      fs/udf/super.o`<br>
`  CC      fs/udf/truncate.o`<br>
`  CC      net/ipv6/ip6_flowlabel.o`<br>
`  CC      net/ipv6/inet6_connection_sock.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2.o`<br>
`  CC      fs/udf/symlink.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/common/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_gre.o`<br>
`  CC      fs/udf/directory.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_dev.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_core.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/q6audio_v2_aio.o`<br>
`  CC      drivers/media/radio/radio-iris.o`<br>
`  CC      fs/udf/misc.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_buf_mgr.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_dev.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_base.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_sctp.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_mp3.o`<br>
`  CC      drivers/mmc/core/sd.o`<br>
`  CC      net/netfilter/nf_conntrack_proto_udplite.o`<br>
`  CC      drivers/media/radio/radio-iris-transport.o`<br>
`  CC      fs/udf/udftime.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/fd/msm_fd_hw.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_amrnb.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_formats.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_util.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/fd/built-in.o`<br>
`  CC      drivers/mmc/core/sd_ops.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/ispif/msm_ispif.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_util.o`<br>
`  CC      fs/udf/unicode.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_axi_util.o`<br>
`  CC      drivers/media/radio/silabs/radio-silabs.o`<br>
`  CC      drivers/mmc/core/sdio.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/ispif/built-in.o`<br>
`  CC      net/rmnet_data/rmnet_map_data.o`<br>
`  LD      fs/udf/udf.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_io_util.o`<br>
`  LD      fs/udf/built-in.o`<br>
`  LD      drivers/media/radio/silabs/built-in.o`<br>
`  CC      drivers/mmc/core/sdio_ops.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_amrwb.o`<br>
`  CC      fs/eventpoll.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp_stats_util.o`<br>
`  LD      drivers/media/radio/built-in.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_smmu.o`<br>
`  CC      drivers/mmc/core/sdio_bus.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_dev.o`<br>
`  CC      fs/anon_inodes.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp48.o`<br>
`  CC      drivers/mmc/core/sdio_cis.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_wb.o`<br>
`  CC      net/netfilter/nf_conntrack_netlink.o`<br>
`  CC      net/ipv6/sysctl_net_ipv6.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_sync.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_pipe.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_amrwbplus.o`<br>
`  CC      drivers/mmc/core/sdio_io.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp47.o`<br>
`  CC      fs/signalfd.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_core.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_hw.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_evrc.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_ctl.o`<br>
`  CC      drivers/mmc/core/sdio_irq.o`<br>
`  CC      net/netfilter/nf_conntrack_amanda.o`<br>
`  CC      fs/timerfd.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp46.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/jpeg_10/msm_jpeg_platform.o`<br>
`  CC      drivers/mmc/core/quirks.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_qcelp.o`<br>
`  CC      net/ipv6/xfrm6_policy.o`<br>
`  CC      fs/eventfd.o`<br>
`  CC      drivers/mmc/core/slot-gpio.o`<br>
`  CC      drivers/mmc/core/debugfs.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp44.o`<br>
`  CC      net/ipv6/xfrm6_state.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/jpeg_10/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_ftp.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/amrwb_in.o`<br>
`  CC      fs/aio.o`<br>
`  CC      net/ipv4/devinet.o`<br>
`  CC      net/netfilter/nf_conntrack_h323_main.o`<br>
`  LD      drivers/mmc/core/mmc_core.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/msm_buf_mgr/msm_generic_buf_mgr.o`<br>
`  LD      drivers/mmc/core/built-in.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/audio_hwacc_effects.o`<br>
`  CC      fs/locks.o`<br>
`  CC      drivers/mmc/host/sdhci.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_sync.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/msm_buf_mgr/built-in.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usf.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_debug.o`<br>
`  CC      fs/compat.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/msm_vb2/msm_vb2.o`<br>
`  CC      net/ipv6/xfrm6_input.o`<br>
`  CC      drivers/mmc/host/sdhci-pltfm.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r1_debug.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/ultrasound/usfcdev.o`<br>
`  CC      drivers/media/rc/keymaps/rc-adstech-dvb-t-pci.o`<br>
`  CC      fs/compat_ioctl.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/msm_vb2/built-in.o`<br>
`  CC      drivers/mmc/host/sdhci-msm.o`<br>
`  CC      drivers/mmc/host/sdhci-msm-ice.o`<br>
`  CC      drivers/media/rc/keymaps/rc-alink-dtu-m.o`<br>
`  CC      drivers/misc/qcom/qdsp6v2/ultrasound/q6usm.o`<br>
`  CC      drivers/media/platform/msm/sde/rotator/sde_rotator_r3_debug.o`<br>
`  CC      fs/binfmt_script.o`<br>
`  CC      net/rmnet_data/rmnet_map_command.o`<br>
`  CC      drivers/mmc/host/cmdq_hci.o`<br>
`  CC      net/rmnet_data/rmnet_data_stats.o`<br>
`  LD      drivers/media/platform/msm/sde/rotator/built-in.o`<br>
`  LD      drivers/media/platform/msm/sde/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-anysee.o`<br>
`  CC      fs/binfmt_elf.o`<br>
`  LD      drivers/mmc/host/built-in.o`<br>
`  LD      drivers/misc/qcom/qdsp6v2/ultrasound/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-apac-viewcomp.o`<br>
`  LD      drivers/misc/qcom/qdsp6v2/built-in.o`<br>
`  LD      drivers/mmc/built-in.o`<br>
`  CC      fs/compat_binfmt_elf.o`<br>
`  CC      drivers/media/rc/keymaps/rc-asus-pc39.o`<br>
`  LD      drivers/misc/qcom/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-asus-ps3-100.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_v4l2_vidc.o`<br>
`  CC      drivers/misc/uid_cputime.o`<br>
`  CC      fs/mbcache.o`<br>
`  CC      drivers/media/rc/keymaps/rc-ati-tv-wonder-hd-600.o`<br>
`  CC      drivers/net/mii.o`<br>
`  CC      drivers/media/rc/keymaps/rc-ati-x10.o`<br>
`  CC      drivers/misc/type-c-pericom.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-a16d.o`<br>
`  CC      net/netfilter/nf_conntrack_h323_asn1.o`<br>
`  CC      fs/posix_acl.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia.o`<br>
`  CC      drivers/media/rc/rc-main.o`<br>
`  CC      drivers/net/Space.o`<br>
`  LD      drivers/misc/built-in.o`<br>
`  CC      fs/coredump.o`<br>
`  CC      drivers/media/rc/rc-ir-raw.o`<br>
`  CC      fs/drop_caches.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-cardbus.o`<br>
`  CC      net/ipv6/xfrm6_output.o`<br>
`  CC      drivers/net/loopback.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-dvbt.o`<br>
`  CC      fs/fhandle.o`<br>
`  CC      drivers/media/rc/lirc_dev.o`<br>
`  CC      drivers/media/rc/ir-nec-decoder.o`<br>
`  CC      drivers/net/phy/phy.o`<br>
`  LD      net/rmnet_data/rmnet_data.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp40.o`<br>
`  LD      net/rmnet_data/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_irc.o`<br>
`  CC      fs/dcookies.o`<br>
`  CC      net/netfilter/nf_conntrack_broadcast.o`<br>
`  CC      drivers/media/rc/ir-rc5-decoder.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-m135a.o`<br>
`  CC      drivers/net/phy/phy_device.o`<br>
`  CC      drivers/media/rc/ir-rc6-decoder.o`<br>
`  CC      drivers/media/rc/ir-jvc-decoder.o`<br>
`  LD      fs/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/isp/msm_isp.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-m733a-rm-k6.o`<br>
`  CC      net/sched/sch_generic.o`<br>
`  CC      drivers/media/rc/ir-sony-decoder.o`<br>
`  CC      drivers/net/phy/mdio_bus.o`<br>
`  CC      drivers/net/ppp/ppp_generic.o`<br>
`  CC      drivers/media/rc/ir-sanyo-decoder.o`<br>
`  CC      net/sched/sch_mq.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/isp/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avermedia-rm-ks.o`<br>
`  CC      net/ipv4/af_inet.o`<br>
`  LD      drivers/net/phy/libphy.o`<br>
`  CC      net/sched/sch_api.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp_soc.o`<br>
`  CC      drivers/net/ppp/ppp_async.o`<br>
`  LD      drivers/net/phy/built-in.o`<br>
`  CC      drivers/net/ppp/bsd_comp.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/pproc/cpp/msm_cpp.o`<br>
`  CC      drivers/net/ppp/ppp_deflate.o`<br>
`  CC      drivers/net/ppp/ppp_mppe.o`<br>
`  CC      drivers/net/ppp/ppp_synctty.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/pproc/cpp/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vidc_common.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/pproc/built-in.o`<br>
`  CC      drivers/net/ppp/pppox.o`<br>
`  LD      drivers/net/ethernet/3com/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-avertv-303.o`<br>
`  CC      drivers/net/ppp/pppoe.o`<br>
`  LD      drivers/net/ethernet/8390/built-in.o`<br>
`  CC      drivers/net/ppp/pppolac.o`<br>
`  CC      net/ipv6/xfrm6_protocol.o`<br>
`  LD      drivers/net/ethernet/adaptec/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vidc.o`<br>
`  LD      drivers/net/ethernet/agere/built-in.o`<br>
`  CC      drivers/media/rc/ir-sharp-decoder.o`<br>
`  CC      drivers/net/ppp/pppopns.o`<br>
`  LD      drivers/net/ethernet/alteon/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vdec.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_venc.o`<br>
`  LD      drivers/net/ethernet/amd/built-in.o`<br>
`  CC      drivers/media/rc/ir-lirc-codec.o`<br>
`  CC      drivers/media/rc/ir-xmp-decoder.o`<br>
`  CC      net/netfilter/nf_conntrack_netbios_ns.o`<br>
`  LD      drivers/net/ppp/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/actuator/msm_actuator.o`<br>
`  LD      drivers/net/ethernet/arc/built-in.o`<br>
`  CC      net/sched/sch_blackhole.o`<br>
`  CC      drivers/net/slip/slhc.o`<br>
`  CC      drivers/media/rc/pwm-ir.o`<br>
`  LD      drivers/net/ethernet/atheros/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_smem.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vidc_debug.o`<br>
`  LD      drivers/net/ethernet/broadcom/built-in.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/actuator/built-in.o`<br>
`  LD      drivers/media/rc/rc-core.o`<br>
`  CC      net/netfilter/nf_conntrack_pptp.o`<br>
`  LD      drivers/net/ethernet/brocade/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/csid/msm_csid.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/cci/msm_cci.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/csiphy/msm_csiphy.o`<br>
`  LD      drivers/net/slip/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vidc_res_parse.o`<br>
`  CC      drivers/media/rc/keymaps/rc-azurewave-ad-tu700.o`<br>
`  CC      drivers/media/platform/msm/vidc/venus_hfi.o`<br>
`  LD      drivers/net/ethernet/chelsio/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/hfi_response_handler.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/cci/built-in.o`<br>
`  CC      net/sched/cls_api.o`<br>
`  CC      net/sched/act_api.o`<br>
`  LD      drivers/net/ethernet/cisco/built-in.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/csiphy/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-behold.o`<br>
`  CC      drivers/media/platform/msm/vidc/hfi_packetization.o`<br>
`  CC      drivers/net/usb/asix_devices.o`<br>
`  CC      drivers/media/platform/msm/vidc/vidc_hfi.o`<br>
`  LD      drivers/net/ethernet/dec/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/eeprom/msm_eeprom.o`<br>
`  CC      drivers/media/rc/keymaps/rc-behold-columbus.o`<br>
`  LD      drivers/net/ethernet/dlink/built-in.o`<br>
`  CC      drivers/net/usb/asix_common.o`<br>
`  LD      drivers/net/ethernet/emulex/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/venus_boot.o`<br>
`  LD      drivers/net/ethernet/hp/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-budget-ci-old.o`<br>
`  LD      drivers/net/ethernet/i825xx/built-in.o`<br>
`  CC      drivers/net/usb/ax88172a.o`<br>
`  LD      drivers/net/ethernet/intel/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-cinergy-1400.o`<br>
`  CC      net/ipv6/netfilter.o`<br>
`  CC      net/ipv6/fib6_rules.o`<br>
`  LD      drivers/net/ethernet/marvell/built-in.o`<br>
`  CC      drivers/net/usb/ax88179_178a.o`<br>
`  LD      drivers/net/ethernet/mellanox/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-cinergy.o`<br>
`  CC      net/ipv4/igmp.o`<br>
`  CC      net/netfilter/nf_conntrack_sane.o`<br>
`  CC      net/ipv4/fib_frontend.o`<br>
`  LD      drivers/net/ethernet/micrel/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-delock-61959.o`<br>
`  LD      drivers/net/ethernet/microchip/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dib0700-nec.o`<br>
`  CC      drivers/net/ethernet/msm/rndis_ipa.o`<br>
`  LD      drivers/net/ethernet/myricom/built-in.o`<br>
`  CC      net/netfilter/nf_conntrack_tftp.o`<br>
`  LD      drivers/net/ethernet/msm/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dib0700-rc5.o`<br>
`  LD      drivers/net/ethernet/natsemi/built-in.o`<br>
`  LD      drivers/nfc/built-in.o`<br>
`  LD      drivers/net/ethernet/neterion/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-digitalnow-tinytwin.o`<br>
`  CC      drivers/media/rc/keymaps/rc-digittrade.o`<br>
`  LD      drivers/net/ethernet/nvidia/built-in.o`<br>
`  LD      drivers/net/ethernet/oki-semi/built-in.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/csid/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dm1105-nec.o`<br>
`  CC      drivers/of/base.o`<br>
`  LD      drivers/net/ethernet/packetengines/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/flash/msm_flash.o`<br>
`  CC      drivers/of/device.o`<br>
`  LD      drivers/net/ethernet/qlogic/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dntv-live-dvb-t.o`<br>
`  LD      drivers/net/ethernet/qualcomm/built-in.o`<br>
`  CC      net/sched/sch_fifo.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/flash/built-in.o`<br>
`  LD      drivers/net/ethernet/rdc/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dntv-live-dvbt-pro.o`<br>
`  CC      drivers/pci/access.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_cci_i2c.o`<br>
`  LD      drivers/net/ethernet/realtek/built-in.o`<br>
`  LD      drivers/net/ethernet/samsung/built-in.o`<br>
`  CC      net/ipv6/proc.o`<br>
`  LD      drivers/net/ethernet/seeq/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-dvbsky.o`<br>
`  CC      net/sched/sch_htb.o`<br>
`  LD      drivers/net/ethernet/silan/built-in.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/eeprom/built-in.o`<br>
`  CC      drivers/pci/bus.o`<br>
`  CC      drivers/media/platform/msm/vidc/msm_vidc_dcvs.o`<br>
`  LD      drivers/net/ethernet/sis/built-in.o`<br>
`  CC      drivers/pci/probe.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/ir_cut/msm_ir_cut.o`<br>
`  LD      drivers/net/ethernet/smsc/built-in.o`<br>
`  LD      drivers/net/ethernet/stmicro/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_qup_i2c.o`<br>
`  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_dyn_gov.o`<br>
`  LD      drivers/net/ethernet/sun/built-in.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/ir_cut/built-in.o`<br>
`  CC      drivers/pci/host-bridge.o`<br>
`  CC      net/netfilter/nf_log_common.o`<br>
`  CC      drivers/media/platform/msm/vidc/governors/msm_vidc_table_gov.o`<br>
`  LD      drivers/net/ethernet/tehuti/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/vmem/vmem.o`<br>
`  LD      drivers/net/ethernet/ti/built-in.o`<br>
`  CC      drivers/pci/remove.o`<br>
`  CC      net/netfilter/nf_nat_core.o`<br>
`  LD      drivers/net/ethernet/via/built-in.o`<br>
`  LD      drivers/net/ethernet/wiznet/built-in.o`<br>
`  CC      drivers/media/platform/msm/vidc/vmem/vmem_debugfs.o`<br>
`  CC      net/netfilter/nf_nat_proto_unknown.o`<br>
`  LD      drivers/net/ethernet/built-in.o`<br>
`  LD      drivers/media/platform/msm/vidc/governors/built-in.o`<br>
`  CC      drivers/of/platform.o`<br>
`  CC      drivers/pci/pci.o`<br>
`  CC      drivers/pci/pci-driver.o`<br>
`  LD      drivers/media/platform/msm/vidc/vmem/built-in.o`<br>
`  LD      drivers/media/platform/msm/vidc/built-in.o`<br>
`  CC      net/sched/sch_prio.o`<br>
`  CC      drivers/of/fdt.o`<br>
`  CC      drivers/net/usb/cdc_ether.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_spi.o`<br>
`  CC      drivers/media/rc/keymaps/rc-em-terratec.o`<br>
`  CC      drivers/media/rc/keymaps/rc-encore-enltv2.o`<br>
`  CC      drivers/net/usb/net1080.o`<br>
`  CC      drivers/net/usb/cdc_subset.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/io/msm_camera_dt_util.o`<br>
`  CC      drivers/net/usb/zaurus.o`<br>
`  CC      drivers/media/rc/keymaps/rc-encore-enltv.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/main.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/netdev.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/cfg80211.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/pcie_bus.o`<br>
`  CC      net/ipv6/ah6.o`<br>
`  CC      net/ipv6/esp6.o`<br>
`  CC      drivers/media/rc/keymaps/rc-encore-enltv-fm53.o`<br>
`  CC      net/ipv6/ipcomp6.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/io/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-evga-indtube.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/debugfs.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/ir_led/msm_ir_led.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/wmi.o`<br>
`  CC      net/ipv4/fib_semantics.o`<br>
`  CC      drivers/media/rc/keymaps/rc-eztv.o`<br>
`  CC      drivers/media/rc/keymaps/rc-flydvb.o`<br>
`  CC      drivers/of/fdt_address.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/interrupt.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/ir_led/built-in.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/txrx.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/ois/msm_ois.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_init.o`<br>
`  CC      drivers/media/rc/keymaps/rc-flyvideo.o`<br>
`  CC      drivers/media/rc/keymaps/rc-fusionhdtv-mce.o`<br>
`  CC      drivers/media/rc/keymaps/rc-gadmei-rm008z.o`<br>
`  CC      drivers/of/address.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor_driver.o`<br>
`  CC      net/sched/cls_u32.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/ois/built-in.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/msm.o`<br>
`  CC      drivers/media/rc/keymaps/rc-genius-tvgo-a11mce.o`<br>
`  CC      drivers/net/usb/usbnet.o`<br>
`  CC      drivers/media/platform/msm/camera_v2/sensor/msm_sensor.o`<br>
`  CC      drivers/of/irq.o`<br>
`  CC      drivers/media/rc/keymaps/rc-gotview7135.o`<br>
`  CC      drivers/of/of_net.o`<br>
`  CC      drivers/net/usb/cdc_ncm.o`<br>
`  CC      net/netfilter/nf_nat_proto_common.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/debug.o`<br>
`  CC      net/netfilter/nf_nat_proto_udp.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/rx_reorder.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/sensor/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-imon-mce.o`<br>
`  CC      drivers/media/rc/keymaps/rc-imon-pad.o`<br>
`  LD      drivers/media/platform/msm/camera_v2/built-in.o`<br>
`  CC      drivers/of/of_mdio.o`<br>
`  CC      drivers/of/of_pci.o`<br>
`  CC      drivers/of/of_pci_irq.o`<br>
`  LD      drivers/net/usb/asix.o`<br>
`  LD      drivers/net/usb/built-in.o`<br>
`  LD      drivers/media/platform/msm/built-in.o`<br>
`  CC      drivers/net/tun.o`<br>
`  CC      drivers/of/of_spmi.o`<br>
`  CC      drivers/media/rc/keymaps/rc-iodata-bctv7e.o`<br>
`  CC      drivers/net/wireless/cnss_crypto/cnss_secif.o`<br>
`  CC      drivers/net/veth.o`<br>
`  CC      drivers/of/of_reserved_mem.o`<br>
`  CC      drivers/of/of_slimbus.o`<br>
`  CC      drivers/media/rc/keymaps/rc-it913x-v1.o`<br>
`  LD      drivers/net/wireless/cnss_crypto/built-in.o`<br>
`  LD      drivers/media/platform/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-it913x-v2.o`<br>
`  CC      drivers/pci/search.o`<br>
`  CC      drivers/pci/pci-sysfs.o`<br>
`  CC      drivers/pci/rom.o`<br>
`  CC      drivers/of/of_coresight.o`<br>
`  CC      drivers/of/of_batterydata.o`<br>
`  CC      drivers/media/rc/keymaps/rc-kaiomy.o`<br>
`  CC      net/ipv6/xfrm6_tunnel.o`<br>
`  CC      drivers/pci/setup-res.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/ioctl.o`<br>
`  CC      drivers/pci/irq.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/fw.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/pm.o`<br>
`  CC      drivers/media/rc/keymaps/rc-kworld-315u.o`<br>
`  CC      drivers/media/rc/keymaps/rc-kworld-pc150u.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/pmc.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/trace.o`<br>
`  CC      drivers/media/rc/keymaps/rc-kworld-plus-tv-analog.o`<br>
`  LD      drivers/of/built-in.o`<br>
`  CC      net/ipv6/tunnel6.o`<br>
`  CC      drivers/net/wireless/cnss_prealloc/cnss_prealloc.o`<br>
`  CC      drivers/pci/vpd.o`<br>
`  CC      drivers/media/rc/keymaps/rc-leadtek-y04g0051.o`<br>
`  LD      drivers/net/wireless/cnss_prealloc/built-in.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/wil_platform.o`<br>
`  CC      drivers/pci/setup-bus.o`<br>
`  CC      net/unix/af_unix.o`<br>
`  CC      drivers/net/wireless/wcnss/wcnss_wlan.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/ethtool.o`<br>
`  CC      net/netfilter/nf_nat_proto_tcp.o`<br>
`  CC      net/sched/cls_fw.o`<br>
`  CC      net/unix/garbage.o`<br>
`  CC      net/ipv4/fib_trie.o`<br>
`  CC      net/unix/sysctl_net_unix.o`<br>
`  CC      drivers/net/wireless/wcnss/wcnss_vreg.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/wil_crash_dump.o`<br>
`  CC      drivers/media/rc/keymaps/rc-lirc.o`<br>
`  LD      drivers/net/wireless/wcnss/wcnsscore.o`<br>
`  LD      drivers/net/wireless/wcnss/built-in.o`<br>
`  CC      net/sunrpc/clnt.o`<br>
`  CC      drivers/media/rc/keymaps/rc-lme2510.o`<br>
`  CC      drivers/net/wireless/ath/wil6210/p2p.o`<br>
`  CC      net/sunrpc/xprt.o`<br>
`  CC      net/netfilter/nf_nat_helper.o`<br>
`  CC      drivers/media/rc/keymaps/rc-manli.o`<br>
`  CC      drivers/pci/vc.o`<br>
`  CC      drivers/media/rc/keymaps/rc-medion-x10.o`<br>
`  CC      net/sunrpc/socklib.o`<br>
`  CC      net/ipv6/xfrm6_mode_transport.o`<br>
`  LD      drivers/net/wireless/ath/wil6210/wil6210.o`<br>
`  CC      net/ipv4/inet_fragment.o`<br>
`  CC      net/ipv4/ping.o`<br>
`  LD      drivers/net/wireless/ath/wil6210/built-in.o`<br>
`  LD      drivers/net/wireless/ath/built-in.o`<br>
`  CC      net/wireless/core.o`<br>
`  LD      drivers/net/wireless/built-in.o`<br>
`  CC      net/ipv4/ip_tunnel_core.o`<br>
`  CC      net/ipv4/gre_offload.o`<br>
`  CC      net/sched/cls_flow.o`<br>
`  LD      drivers/net/built-in.o`<br>
`  CC      net/sched/cls_cgroup.o`<br>
`  CC      drivers/media/rc/keymaps/rc-medion-x10-digitainer.o`<br>
`  CC      drivers/pci/proc.o`<br>
`  CC      drivers/media/rc/keymaps/rc-medion-x10-or2x.o`<br>
`  CC      drivers/pci/slot.o`<br>
`  CC      net/netfilter/nf_nat_proto_dccp.o`<br>
`  CC      drivers/media/rc/keymaps/rc-msi-digivox-ii.o`<br>
`  CC      drivers/pci/quirks.o`<br>
`  CC      drivers/pci/msi.o`<br>
`  CC      net/netfilter/nf_nat_proto_udplite.o`<br>
`  CC      net/ipv6/xfrm6_mode_tunnel.o`<br>
`  CC      drivers/media/rc/keymaps/rc-msi-digivox-iii.o`<br>
`  CC      net/sched/ematch.o`<br>
`  CC      net/sunrpc/xprtsock.o`<br>
`  CC      drivers/pci/setup-irq.o`<br>
`  LD      net/unix/unix.o`<br>
`  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere.o`<br>
`  CC      net/ipv6/xfrm6_mode_beet.o`<br>
`  LD      net/unix/built-in.o`<br>
`  CC      net/ipv4/ip_tunnel.o`<br>
`  CC      net/ipv4/sysctl_net_ipv4.o`<br>
`  CC      net/xfrm/xfrm_policy.o`<br>
`  CC      net/xfrm/xfrm_state.o`<br>
`  CC      drivers/media/rc/keymaps/rc-msi-tvanywhere-plus.o`<br>
`  CC      drivers/media/rc/keymaps/rc-nebula.o`<br>
`  CC      net/netfilter/nf_nat_proto_sctp.o`<br>
`  CC      net/netfilter/nf_nat_amanda.o`<br>
`  CC      net/netfilter/nf_nat_ftp.o`<br>
`  CC      drivers/pci/pci-label.o`<br>
`  CC      net/compat.o`<br>
`  CC      drivers/media/rc/keymaps/rc-nec-terratec-cinergy-xs.o`<br>
`  CC      net/wireless/sysfs.o`<br>
`  CC      net/sysctl_net.o`<br>
`  CC      net/sched/em_cmp.o`<br>
`  CC      net/sched/em_nbyte.o`<br>
`  CC      drivers/media/rc/keymaps/rc-norwood.o`<br>
`  CC      net/ipv6/mip6.o`<br>
`  CC      net/ipv4/sysfs_net_ipv4.o`<br>
`  CC      drivers/pci/syscall.o`<br>
`  CC      drivers/media/rc/keymaps/rc-npgtech.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pctv-sedna.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pinnacle-color.o`<br>
`  CC      net/netfilter/nf_nat_irc.o`<br>
`  CC      net/ipv4/proc.o`<br>
`  CC      net/activity_stats.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pinnacle-grey.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pinnacle-pctv-hd.o`<br>
`  CC      net/netfilter/nf_nat_tftp.o`<br>
`  CC      net/sched/em_u32.o`<br>
`  CC      net/ipv4/fib_rules.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pixelview.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pixelview-mk12.o`<br>
`  CC      net/ipv4/udp_tunnel.o`<br>
`  CC      net/sunrpc/sched.o`<br>
`  CC      drivers/pci/of.o`<br>
`  CC      drivers/pci/host/pci-msm.o`<br>
`  CC      net/wireless/radiotap.o`<br>
`  CC      net/wireless/util.o`<br>
`  LD      drivers/pci/host/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pixelview-002t.o`<br>
`  CC      net/sched/em_meta.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pixelview-new.o`<br>
`  CC      drivers/media/rc/keymaps/rc-powercolor-real-angel.o`<br>
`  CC      net/ipv6/netfilter/ip6_tables.o`<br>
`  CC      net/sunrpc/auth.o`<br>
`  CC      net/ipv6/netfilter/ip6table_filter.o`<br>
`  CC      net/sunrpc/auth_null.o`<br>
`  CC      drivers/media/rc/keymaps/rc-proteus-2309.o`<br>
`  CC      net/ipv4/ah4.o`<br>
`  CC      drivers/media/rc/keymaps/rc-purpletv.o`<br>
`  CC      net/netfilter/x_tables.o`<br>
`  CC      drivers/media/rc/keymaps/rc-pv951.o`<br>
`  LD      drivers/pci/built-in.o`<br>
`  CC      drivers/media/rc/keymaps/rc-hauppauge.o`<br>
`  CC      drivers/media/rc/keymaps/rc-rc6-mce.o`<br>
`  CC      drivers/media/rc/keymaps/rc-real-audio-220-32-keys.o`<br>
`  CC      net/ipv6/netfilter/ip6table_mangle.o`<br>
`  CC      net/ipv4/esp4.o`<br>
`  CC      drivers/media/rc/keymaps/rc-reddo.o`<br>
`  CC      net/xfrm/xfrm_hash.o`<br>
`  CC      net/ipv6/netfilter/ip6table_raw.o`<br>
`  CC      drivers/media/rc/keymaps/rc-snapstream-firefly.o`<br>
`  CC      net/wireless/reg.o`<br>
`  CC      net/netfilter/xt_tcpudp.o`<br>
`  CC      net/ipv6/netfilter/nf_conntrack_l3proto_ipv6.o`<br>
`  CC      net/xfrm/xfrm_input.o`<br>
`  CC      drivers/media/rc/keymaps/rc-streamzap.o`<br>
`  CC      net/sunrpc/auth_unix.o`<br>
`  CC      drivers/media/rc/keymaps/rc-tbs-nec.o`<br>
`  CC      net/sched/em_text.o`<br>
`  CC      drivers/media/rc/keymaps/rc-technisat-usb2.o`<br>
`  CC      drivers/media/rc/keymaps/rc-terratec-cinergy-xs.o`<br>
`  CC      net/wireless/scan.o`<br>
`  CC      net/sunrpc/auth_generic.o`<br>
`  CC      net/ipv6/netfilter/nf_conntrack_proto_icmpv6.o`<br>
`  CC      net/ipv6/netfilter/nf_defrag_ipv6_hooks.o`<br>
`  CC      drivers/media/rc/keymaps/rc-terratec-slim.o`<br>
`  CC      drivers/media/rc/keymaps/rc-terratec-slim-2.o`<br>
`  CC      net/netfilter/xt_mark.o`<br>
`  CC      net/ipv4/ipcomp.o`<br>
`  CC      drivers/media/rc/keymaps/rc-tevii-nec.o`<br>
`  CC      net/ipv6/sit.o`<br>
`  CC      drivers/media/rc/keymaps/rc-tivo.o`<br>
`  CC      drivers/media/rc/keymaps/rc-total-media-in-hand.o`<br>
`  CC      net/netfilter/xt_connmark.o`<br>
`  CC      net/ipv6/netfilter/nf_conntrack_reasm.o`<br>
`  CC      net/sunrpc/svc.o`<br>
`  CC      net/sunrpc/svcsock.o`<br>
`  CC      net/sunrpc/svcauth.o`<br>
`  CC      drivers/media/rc/keymaps/rc-total-media-in-hand-02.o`<br>
`  LD      net/sched/built-in.o`<br>
`  CC      net/ipv6/addrconf_core.o`<br>
`  CC      net/xfrm/xfrm_output.o`<br>
`  CC      drivers/media/rc/keymaps/rc-trekstor.o`<br>
`  CC      net/sunrpc/svcauth_unix.o`<br>
`  CC      drivers/media/rc/keymaps/rc-tt-1500.o`<br>
`  CC      drivers/media/rc/keymaps/rc-twinhan1027.o`<br>
`  CC      net/sunrpc/addr.o`<br>
`  CC      net/ipv6/netfilter/nf_log_ipv6.o`<br>
`  CC      drivers/media/rc/keymaps/rc-videomate-m1f.o`<br>
`  CC      net/netfilter/xt_nat.o`<br>
`  CC      drivers/media/rc/keymaps/rc-videomate-s350.o`<br>
`  CC      drivers/media/rc/keymaps/rc-videomate-tv-pvr.o`<br>
`  CC      net/ipv4/xfrm4_tunnel.o`<br>
`  CC      net/netfilter/xt_CLASSIFY.o`<br>
`  CC      net/xfrm/xfrm_sysctl.o`<br>
`  CC      drivers/media/rc/keymaps/rc-winfast.o`<br>
`  CC      drivers/media/rc/keymaps/rc-winfast-usbii-deluxe.o`<br>
`  CC      net/wireless/nl80211.o`<br>
`  CC      net/wireless/mlme.o`<br>
`  CC      drivers/media/rc/keymaps/rc-su3000.o`<br>
`  CC      net/ipv6/exthdrs_core.o`<br>
`  CC      net/ipv6/ip6_checksum.o`<br>
`  CC      net/netfilter/xt_CONNSECMARK.o`<br>
`  CC      net/xfrm/xfrm_replay.o`<br>
`  CC      net/ipv6/ip6_icmp.o`<br>
`  CC      net/sunrpc/rpcb_clnt.o`<br>
`  LD      drivers/media/rc/keymaps/built-in.o`<br>
`  CC      net/xfrm/xfrm_proc.o`<br>
`  CC      net/netfilter/xt_CT.o`<br>
`  CC      net/ipv6/netfilter/nf_reject_ipv6.o`<br>
`  CC      net/ipv6/output_core.o`<br>
`  LD      drivers/media/rc/built-in.o`<br>
`  CC      net/sunrpc/timer.o`<br>
`  CC      net/ipv4/tunnel4.o`<br>
`  CC      net/sunrpc/xdr.o`<br>
`  CC      drivers/media/tuners/tuner-xc2028.o`<br>
`  CC      net/netfilter/xt_LOG.o`<br>
`  CC      drivers/media/tuners/tuner-simple.o`<br>
`  CC      net/ipv6/protocol.o`<br>
`  CC      drivers/media/tuners/tuner-types.o`<br>
`  CC      net/ipv6/ip6_offload.o`<br>
`  CC      drivers/media/tuners/mt20xx.o`<br>
`  CC      net/xfrm/xfrm_algo.o`<br>
`  CC      net/sunrpc/sunrpc_syms.o`<br>
`  CC      net/xfrm/xfrm_user.o`<br>
`  CC      net/ipv6/netfilter/ip6t_rpfilter.o`<br>
`  CC      net/ipv6/tcpv6_offload.o`<br>
`  CC      drivers/media/tuners/tda8290.o`<br>
`  CC      net/ipv4/xfrm4_mode_transport.o`<br>
`  CC      net/wireless/ibss.o`<br>
`  CC      net/sunrpc/cache.o`<br>
`  CC      net/netfilter/xt_NETMAP.o`<br>
`  CC      drivers/media/tuners/tea5767.o`<br>
`  CC      net/netfilter/xt_NFLOG.o`<br>
`  CC      net/ipv6/udp_offload.o`<br>
`  CC      net/sunrpc/rpc_pipe.o`<br>
`  CC      net/netfilter/xt_NFQUEUE.o`<br>
`  CC      net/ipv6/exthdrs_offload.o`<br>
`  CC      net/ipv6/netfilter/ip6t_REJECT.o`<br>
`  CC      net/ipv6/inet6_hashtables.o`<br>
`  CC      drivers/media/tuners/tea5761.o`<br>
`  CC      net/wireless/sme.o`<br>
`  CC      net/netfilter/xt_REDIRECT.o`<br>
`  CC      net/ipv4/xfrm4_mode_tunnel.o`<br>
`  CC      net/ipv6/ip6_udp_tunnel.o`<br>
`  CC      net/netfilter/xt_SECMARK.o`<br>
`  LD      net/ipv6/ipv6.o`<br>
`  CC      net/sunrpc/svc_xprt.o`<br>
`  CC      net/sunrpc/backchannel_rqst.o`<br>
`  CC      net/ipv4/ipconfig.o`<br>
`  CC      net/sunrpc/bc_svc.o`<br>
`  LD      net/ipv6/netfilter/nf_conntrack_ipv6.o`<br>
`  LD      net/ipv6/netfilter/nf_defrag_ipv6.o`<br>
`  CC      drivers/media/tuners/tda9887.o`<br>
`  LD      net/ipv6/netfilter/built-in.o`<br>
`  CC      net/netfilter/xt_TPROXY.o`<br>
`  CC      net/netfilter/xt_TCPMSS.o`<br>
`  CC      drivers/media/tuners/tda827x.o`<br>
`  CC      net/wireless/chan.o`<br>
`  CC      drivers/media/tuners/tda18271-maps.o`<br>
`  CC      net/wireless/ethtool.o`<br>
`  CC      net/ipv4/netfilter.o`<br>
`  CC      net/xfrm/xfrm_ipcomp.o`<br>
`  LD      net/ipv6/built-in.o`<br>
`  LD      drivers/media/usb/b2c2/built-in.o`<br>
`  CC      net/sunrpc/stats.o`<br>
`  LD      drivers/media/usb/dvb-usb/built-in.o`<br>
`  LD      drivers/media/usb/dvb-usb-v2/built-in.o`<br>
`  CC      net/sunrpc/sysctl.o`<br>
`  LD      drivers/media/usb/s2255/built-in.o`<br>
`  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4_compat.o`<br>
`  LD      drivers/media/usb/siano/built-in.o`<br>
`  LD      drivers/media/usb/stkwebcam/built-in.o`<br>
`  LD      drivers/media/usb/ttusb-budget/built-in.o`<br>
`  CC      net/ipv4/netfilter/nf_conntrack_l3proto_ipv4.o`<br>
`  LD      drivers/media/usb/ttusb-dec/built-in.o`<br>
`  CC      net/ipv4/netfilter/nf_conntrack_proto_icmp.o`<br>
`  LD      drivers/media/usb/zr364xx/built-in.o`<br>
`  CC      drivers/media/tuners/tda18271-common.o`<br>
`  LD      drivers/media/usb/built-in.o`<br>
`  CC      drivers/media/tuners/tda18271-fe.o`<br>
`  CC      drivers/media/tuners/xc5000.o`<br>
`  CC      drivers/media/media-device.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-dev.o`<br>
`  CC      net/ipv4/inet_diag.o`<br>
`  CC      net/netfilter/xt_TEE.o`<br>
`  LD      net/sunrpc/xprtrdma/built-in.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-ioctl.o`<br>
`  CC      net/sunrpc/auth_gss/auth_gss.o`<br>
`  CC      net/netfilter/xt_TRACE.o`<br>
`  CC      net/sunrpc/auth_gss/gss_generic_token.o`<br>
`  CC      net/ipv4/tcp_diag.o`<br>
`  CC      drivers/media/media-devnode.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_l3proto_ipv4.o`<br>
`  LD      net/xfrm/built-in.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-device.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-fh.o`<br>
`  CC      drivers/media/tuners/xc4000.o`<br>
`  CC      net/sunrpc/auth_gss/gss_mech_switch.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_proto_icmp.o`<br>
`  CC      net/ipv4/netfilter/nf_defrag_ipv4.o`<br>
`  CC      net/sunrpc/auth_gss/svcauth_gss.o`<br>
`  CC      net/netfilter/xt_IDLETIMER.o`<br>
`  CC      drivers/media/media-entity.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-event.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-ctrls.o`<br>
`  CC      net/netfilter/xt_HARDIDLETIMER.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-subdev.o`<br>
`  CC      net/ipv4/tcp_cubic.o`<br>
`  CC      net/ipv4/netfilter/nf_log_ipv4.o`<br>
`  CC      net/ipv4/netfilter/nf_reject_ipv4.o`<br>
`  CC      net/ipv4/tcp_memcontrol.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_h323.o`<br>
`  LD      drivers/media/media.o`<br>
`  CC      net/ipv4/xfrm4_policy.o`<br>
`  CC      net/sunrpc/auth_gss/gss_rpc_upcall.o`<br>
`  CC      drivers/media/tuners/mc44s803.o`<br>
`  LD      drivers/media/tuners/tda18271.o`<br>
`  LD      drivers/media/tuners/built-in.o`<br>
`  CC      net/sunrpc/auth_gss/gss_rpc_xdr.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_mech.o`<br>
`  CC      net/netfilter/xt_comment.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-clk.o`<br>
`  CC      net/netfilter/xt_connlimit.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-async.o`<br>
`  CC      net/ipv4/xfrm4_state.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_pptp.o`<br>
`  CC      net/ipv4/xfrm4_input.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_masquerade_ipv4.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_seal.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_unseal.o`<br>
`  CC      net/netfilter/xt_conntrack.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-compat-ioctl32.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_seqnum.o`<br>
`  LD      net/sunrpc/sunrpc.o`<br>
`  CC      drivers/phy/phy-core.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-of.o`<br>
`  CC      net/netfilter/xt_dscp.o`<br>
`  CC      drivers/phy/phy-qcom-ufs.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_wrap.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_crypto.o`<br>
`  CC      net/netfilter/xt_ecn.o`<br>
`  CC      drivers/phy/phy-qcom-ufs-qmp-20nm.o`<br>
`  CC      net/sunrpc/auth_gss/gss_krb5_keys.o`<br>
`  CC      net/ipv4/xfrm4_output.o`<br>
`  CC      drivers/phy/phy-qcom-ufs-qmp-14nm.o`<br>
`  CC      net/ipv4/xfrm4_protocol.o`<br>
`  CC      net/ipv4/netfilter/nf_nat_proto_gre.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-common.o`<br>
`  CC      net/ipv4/netfilter/ip_tables.o`<br>
`  CC      drivers/phy/phy-qcom-ufs-qmp-v3.o`<br>
`  CC      net/ipv4/netfilter/iptable_filter.o`<br>
`  CC      net/ipv4/netfilter/iptable_mangle.o`<br>
`  CC      net/ipv4/netfilter/iptable_nat.o`<br>
`  CC      net/netfilter/xt_esp.o`<br>
`  LD      net/sunrpc/auth_gss/auth_rpcgss.o`<br>
`  CC      net/ipv4/netfilter/iptable_raw.o`<br>
`  CC      net/netfilter/xt_hashlimit.o`<br>
`  CC      net/wireless/mesh.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-dv-timings.o`<br>
`  CC      drivers/phy/phy-qcom-ufs-qrbtc-v2.o`<br>
`  LD      net/sunrpc/auth_gss/rpcsec_gss_krb5.o`<br>
`  LD      net/sunrpc/auth_gss/built-in.o`<br>
`  CC      drivers/media/v4l2-core/v4l2-mem2mem.o`<br>
`  LD      net/sunrpc/built-in.o`<br>
`  LD      drivers/phy/built-in.o`<br>
`  CC      drivers/media/v4l2-core/videobuf-core.o`<br>
`  CC      drivers/media/v4l2-core/videobuf2-core.o`<br>
`  CC      net/ipv4/netfilter/iptable_security.o`<br>
`  CC      net/wireless/ap.o`<br>
`  CC      drivers/media/v4l2-core/videobuf2-memops.o`<br>
`  CC      net/netfilter/xt_helper.o`<br>
`  CC      drivers/pinctrl/core.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_clients/odu_bridge.o`<br>
`  CC      net/ipv4/netfilter/ipt_ah.o`<br>
`  CC      drivers/pinctrl/pinctrl-utils.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa.o`<br>
`  CC      drivers/pinctrl/pinmux.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_debugfs.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_clients/ipa_mhi_client.o`<br>
`  CC      net/ipv4/netfilter/ipt_MASQUERADE.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_api.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_hdr.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_rm.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_rm_dependency_graph.o`<br>
`  CC      net/netfilter/xt_hl.o`<br>
`  CC      net/ipv4/netfilter/ipt_NATTYPE.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_flt.o`<br>
`  CC      net/netfilter/xt_iprange.o`<br>
`  CC      net/wireless/trace.o`<br>
`  CC      drivers/media/v4l2-core/videobuf2-vmalloc.o`<br>
`  CC      drivers/pinctrl/pinconf.o`<br>
`  CC      drivers/pinctrl/devicetree.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_rm_peers_list.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_clients/ipa_uc_offload.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_rt.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dp.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_client.o`<br>
`  CC      net/ipv4/netfilter/ipt_REJECT.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_utils.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_nat.o`<br>
`  CC      drivers/pinctrl/pinconf-generic.o`<br>
`  LD      drivers/media/v4l2-core/videodev.o`<br>
`  LD      drivers/media/v4l2-core/built-in.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_rm_resource.o`<br>
`  LD      drivers/media/built-in.o`<br>
`  CC      net/netfilter/xt_l2tp.o`<br>
`  CC      net/netfilter/xt_length.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_rm_inactivity_timer.o`<br>
`  LD      drivers/pinctrl/freescale/built-in.o`<br>
`  CC      drivers/pinctrl/qcom/pinctrl-msm.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_intf.o`<br>
`  CC      drivers/power/power_supply_core.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/teth_bridge.o`<br>
`  CC      drivers/pinctrl/qcom/pinctrl-msm8953.o`<br>
`  CC      drivers/power/power_supply_sysfs.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_interrupts.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc.o`<br>
`  CC      drivers/pwm/core.o`<br>
`  LD      drivers/platform/msm/ipa/ipa_clients/built-in.o`<br>
`  CC      drivers/power/power_supply_leds.o`<br>
`  CC      drivers/pwm/sysfs.o`<br>
`  CC      drivers/pwm/pwm-qpnp.o`<br>
`  LD      drivers/platform/msm/ipa/ipa_common`<br>
`  CC      drivers/power/smb1351-charger.o`<br>
`  CC      drivers/power/smb135x-charger.o`<br>
`  CC      net/wireless/wext-core.o`<br>
`  CC      drivers/power/qpnp-fg.o`<br>
`  CC      drivers/power/qpnp-smbcharger.o`<br>
`  CC      net/wireless/wext-proc.o`<br>
`  CC      net/ipv4/netfilter/arp_tables.o`<br>
`  LD      drivers/pinctrl/qcom/built-in.o`<br>
`  LD      drivers/pinctrl/built-in.o`<br>
`  CC      drivers/ras/ras.o`<br>
`  CC      drivers/ras/debugfs.o`<br>
`  CC      drivers/power/pmic-voter.o`<br>
`  CC      net/netfilter/xt_limit.o`<br>
`  CC      drivers/power/qpnp-typec.o`<br>
`  CC      net/netfilter/xt_mac.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_wdi.o`<br>
`  CC      net/wireless/wext-spy.o`<br>
`  CC      drivers/power/battery_current_limit.o`<br>
`  CC      drivers/power/msm_bcl.o`<br>
`  LD      drivers/pwm/built-in.o`<br>
`  CC      net/wireless/wext-priv.o`<br>
`  CC      net/wireless/regdb.o`<br>
`  CC      drivers/power/bcl_peripheral.o`<br>
`  LD      drivers/ras/built-in.o`<br>
`  CC      drivers/platform/msm/msm_11ad/msm_11ad.o`<br>
`  CC      net/netfilter/xt_multiport.o`<br>
`  CC      net/netfilter/xt_nfacct.o`<br>
`  CC      drivers/power/qcom/msm-pm.o`<br>
`  CC      drivers/power/qcom/pm-data.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_core.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_client_api.o`<br>
`  CC      net/ipv4/netfilter/arpt_mangle.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_dma.o`<br>
`  CC      drivers/power/reset/msm-poweroff.o`<br>
`  CC      net/netfilter/xt_pkttype.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_mhi.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_mhi.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_uc_ntn.o`<br>
`  CC      drivers/power/qcom/lpm-stats.o`<br>
`  CC      drivers/power/qcom/pm-boot.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_of.o`<br>
`  LD      drivers/power/reset/built-in.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_rpm_smd.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_fabric_adhoc.o`<br>
`  CC      drivers/power/qcom/msm-core.o`<br>
`  CC      net/netfilter/xt_policy.o`<br>
`  LD      drivers/platform/msm/msm_11ad/msm_11ad_proxy.o`<br>
`  LD      drivers/platform/msm/msm_11ad/built-in.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_arb_adhoc.o`<br>
`  CC      drivers/platform/msm/spmi/spmi.o`<br>
`  CC      drivers/platform/msm/spmi/spmi-resources.o`<br>
`  CC      drivers/platform/msm/spmi/spmi-pmic-arb.o`<br>
`  CC      net/netfilter/xt_qtaguid_print.o`<br>
`  LD      drivers/power/power_supply.o`<br>
`  CC      drivers/power/qcom/debug_core.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_rules.o`<br>
`  CC      net/ipv4/netfilter/arptable_filter.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa.o`<br>
`  CC      drivers/power/qcom/apm.o`<br>
`  CC      net/netfilter/xt_qtaguid.o`<br>
`  CC      drivers/platform/msm/spmi/qpnp-int.o`<br>
`  CC      drivers/platform/msm/sps/bam.o`<br>
`  CC      drivers/platform/msm/qpnp-power-on.o`<br>
`  CC      drivers/platform/msm/sps/sps_bam.o`<br>
`  CC      net/netfilter/xt_quota.o`<br>
`  CC      net/netfilter/xt_quota2.o`<br>
`  CC      drivers/platform/msm/sps/sps.o`<br>
`  CC      net/netfilter/xt_socket.o`<br>
`  CC      net/netfilter/xt_state.o`<br>
`  CC      drivers/platform/msm/qpnp-revid.o`<br>
`  CC      drivers/platform/msm/qpnp-coincell.o`<br>
`  CC      drivers/platform/msm/spmi/spmi-dbgfs.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_bimc_adhoc.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_noc_adhoc.o`<br>
`  LD      net/ipv4/netfilter/nf_conntrack_ipv4.o`<br>
`  LD      net/ipv4/netfilter/nf_nat_ipv4.o`<br>
`  LD      net/ipv4/netfilter/built-in.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_of_adhoc.o`<br>
`  LD      net/ipv4/built-in.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_buspm_coresight_adhoc.o`<br>
`  LD      drivers/power/qcom/built-in.o`<br>
`  CC      drivers/platform/msm/msm_bus/msm_bus_dbg.o`<br>
`  CC      drivers/platform/msm/qpnp-haptic.o`<br>
`  CC      drivers/platform/msm/usb_bam.o`<br>
`  CC      drivers/platform/msm/avtimer.o`<br>
`  CC      net/netfilter/xt_statistic.o`<br>
`  CC      net/netfilter/xt_string.o`<br>
`  CC      net/netfilter/xt_time.o`<br>
`  CC      net/netfilter/xt_u32.o`<br>
`  LD      drivers/power/built-in.o`<br>
`  LD      drivers/platform/msm/spmi/built-in.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service_v01.o`<br>
`  CC      drivers/platform/msm/sps/sps_dma.o`<br>
`  CC      drivers/regulator/core.o`<br>
`  CC      drivers/platform/msm/sps/sps_map.o`<br>
`  CC      drivers/platform/msm/sps/sps_mem.o`<br>
`  CC      drivers/regulator/dummy.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/ipa_qmi_service.o`<br>
`  LD      net/netfilter/netfilter.o`<br>
`  LD      net/netfilter/nfnetlink_queue.o`<br>
`  LD      net/netfilter/nf_conntrack.o`<br>
`  LD      net/netfilter/nf_conntrack_h323.o`<br>
`  CC      drivers/platform/msm/sps/sps_rm.o`<br>
`  CC      drivers/platform/msm/ipa/ipa_v2/rmnet_ipa_fd_ioctl.o`<br>
`  CC      drivers/regulator/fixed-helper.o`<br>
`  LD      drivers/platform/msm/ipa/ipa_v2/ipat.o`<br>
`  LD      net/netfilter/nf_nat.o`<br>
`  CC      drivers/regulator/helpers.o`<br>
`  CC      drivers/regulator/devres.o`<br>
`  CC      drivers/regulator/of_regulator.o`<br>
`  CC      drivers/regulator/fixed.o`<br>
`  CC      drivers/rtc/rtc-lib.o`<br>
`  CC      drivers/rtc/hctosys.o`<br>
`  CC      drivers/rtc/systohc.o`<br>
`  CC      drivers/regulator/mem-acc-regulator.o`<br>
`  CC      drivers/rtc/class.o`<br>
`  LD      net/netfilter/built-in.o`<br>
`  CC      drivers/rtc/interface.o`<br>
`  LD      drivers/platform/msm/msm_bus/built-in.o`<br>
`  CC      drivers/scsi/scsi.o`<br>
`  CC      drivers/sensors/sensors_ssc.o`<br>
`  CC      drivers/slimbus/slimbus.o`<br>
`  CC      drivers/scsi/hosts.o`<br>
`  CC      drivers/slimbus/slim-msm.o`<br>
`  CC      drivers/rtc/rtc-dev.o`<br>
`  CC      drivers/regulator/fan53555.o`<br>
`  CC      drivers/slimbus/slim-msm-ngd.o`<br>
`  CC      drivers/regulator/rpm-smd-regulator.o`<br>
`  CC      drivers/regulator/msm_gfx_ldo.o`<br>
`  CC      drivers/rtc/rtc-proc.o`<br>
`  CC      drivers/regulator/qpnp-regulator.o`<br>
`  CC      drivers/regulator/spm-regulator.o`<br>
`  LD      drivers/platform/msm/ipa/ipa_v2/built-in.o`<br>
`  LD      drivers/platform/msm/ipa/built-in.o`<br>
`  CC      drivers/regulator/cpr-regulator.o`<br>
`  AS      drivers/soc/qcom/idle-v8.o`<br>
`  LD      drivers/sensors/built-in.o`<br>
`  CC      drivers/regulator/cpr3-regulator.o`<br>
`  LD      drivers/platform/msm/sps/built-in.o`<br>
`  CC      drivers/soundwire/soundwire.o`<br>
`  CC      drivers/soc/qcom/cpu_ops.o`<br>
`  CC      drivers/soundwire/swr-wcd-ctrl.o`<br>
`  CC      drivers/rtc/rtc-sysfs.o`<br>
`  CC      drivers/rtc/qpnp-rtc.o`<br>
`  LD      net/wireless/cfg80211.o`<br>
`  CC      drivers/regulator/cpr3-util.o`<br>
`  LD      net/wireless/built-in.o`<br>
`  LD      net/built-in.o`<br>
`  CC      drivers/soc/qcom/msm_rq_stats.o`<br>
`  CC      drivers/regulator/cpr3-hmss-regulator.o`<br>
`  CC      drivers/scsi/scsi_ioctl.o`<br>
`  CC      drivers/regulator/cpr3-mmss-regulator.o`<br>
`  CC      drivers/spi/spi.o`<br>
`  CC      drivers/scsi/constants.o`<br>
`  CC      drivers/regulator/cpr4-apss-regulator.o`<br>
`  LD      drivers/soundwire/built-in.o`<br>
`  CC      drivers/scsi/scsicam.o`<br>
`  CC      drivers/staging/staging.o`<br>
`  CC      drivers/staging/android/ion/ion.o`<br>
`  LD      drivers/rtc/rtc-core.o`<br>
`  LD      drivers/rtc/built-in.o`<br>
`  CC      drivers/staging/android/binder.o`<br>
`  LD      drivers/staging/media/built-in.o`<br>
`  CC      drivers/switch/switch_class.o`<br>
`  CC      drivers/staging/android/ion/ion_heap.o`<br>
`  LD      drivers/slimbus/built-in.o`<br>
`  CC      drivers/thermal/thermal_core.o`<br>
`  CC      drivers/staging/android/ashmem.o`<br>
`  CC      drivers/staging/android/ion/ion_page_pool.o`<br>
`  CC      drivers/soc/qcom/cpuss_dump.o`<br>
`  CC      drivers/soc/qcom/memory_dump_v2.o`<br>
`  CC      drivers/staging/android/timed_output.o`<br>
`  CC      drivers/spi/spidev.o`<br>
`  CC      drivers/spi/spi-qup.o`<br>
`  CC      drivers/scsi/scsi_error.o`<br>
`  LD      drivers/platform/msm/built-in.o`<br>
`  CC      drivers/thermal/thermal_hwmon.o`<br>
`  CC      drivers/thermal/of-thermal.o`<br>
`  LD      drivers/platform/built-in.o`<br>
`  CC      drivers/staging/android/timed_gpio.o`<br>
`  CC      drivers/tty/tty_io.o`<br>
`  CC      drivers/tty/n_tty.o`<br>
`  CC      drivers/tty/tty_ioctl.o`<br>
`  CC      drivers/uio/uio.o`<br>
`  LD      drivers/switch/built-in.o`<br>
`  CC      drivers/thermal/step_wise.o`<br>
`  CC      drivers/scsi/scsi_lib.o`<br>
`  CC      drivers/scsi/scsi_lib_dma.o`<br>
`  CC      drivers/tty/tty_ldisc.o`<br>
`  CC      drivers/tty/tty_buffer.o`<br>
`  CC      drivers/soc/qcom/ddr-health.o`<br>
`  CC      drivers/tty/tty_port.o`<br>
`  CC      drivers/soc/qcom/watchdog_v2.o`<br>
`  CC      drivers/tty/tty_mutex.o`<br>
`  CC      drivers/tty/tty_ldsem.o`<br>
`  LD      drivers/thermal/samsung/built-in.o`<br>
`  CC      drivers/thermal/msm-tsens.o`<br>
`  CC      drivers/staging/android/lowmemorykiller.o`<br>
`  CC      drivers/tty/pty.o`<br>
`  CC      drivers/thermal/qpnp-temp-alarm.o`<br>
`  CC      drivers/thermal/qpnp-adc-tm.o`<br>
`  CC      drivers/scsi/scsi_scan.o`<br>
`  CC      drivers/spi/spi_qsd.o`<br>
`  CC      drivers/tty/tty_audit.o`<br>
`  CC      drivers/tty/sysrq.o`<br>
`  CC      drivers/uio/msm_sharedmem/msm_sharedmem.o`<br>
`  CC      drivers/uio/msm_sharedmem/remote_filesystem_access_v01.o`<br>
`  CC      drivers/staging/android/ion/ion_system_heap.o`<br>
`  CC      drivers/scsi/scsi_sysfs.o`<br>
`  LD      drivers/spi/built-in.o`<br>
`  CC      drivers/scsi/scsi_devinfo.o`<br>
`  CC      drivers/soc/qcom/common_log.o`<br>
`  CC      drivers/soc/qcom/cpu_pwr_ctl.o`<br>
`  CC      drivers/staging/android/ion/ion_carveout_heap.o`<br>
`  CC      drivers/soc/qcom/socinfo.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiData.o`<br>
`  CC      drivers/uio/msm_sharedmem/sharedmem_qmi.o`<br>
`  CC      drivers/soc/qcom/boot_stats.o`<br>
`  LD      drivers/tty/ipwireless/built-in.o`<br>
`  CC      drivers/tty/serial/serial_core.o`<br>
`  CC      drivers/regulator/cprh-kbss-regulator.o`<br>
`  CC      drivers/thermal/msm_thermal.o`<br>
`  CC      drivers/tty/vt/vt_ioctl.o`<br>
`  LD      drivers/uio/msm_sharedmem/built-in.o`<br>
`  LD      drivers/uio/built-in.o`<br>
`  CC      drivers/thermal/msm_thermal-dev.o`<br>
`  CC      drivers/tty/vt/vc_screen.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiDebug.o`<br>
`  CC      drivers/staging/android/ion/ion_chunk_heap.o`<br>
`  CC      drivers/thermal/lmh_interface.o`<br>
`  CC      drivers/soc/qcom/rpm-smd.o`<br>
`  CC      drivers/regulator/qpnp-labibb-regulator.o`<br>
`  CC      drivers/scsi/scsi_sysctl.o`<br>
`  CC      drivers/scsi/scsi_proc.o`<br>
`  CC      drivers/thermal/lmh_lite.o`<br>
`  CC      drivers/regulator/stub-regulator.o`<br>
`  CC      drivers/staging/android/ion/ion_system_secure_heap.o`<br>
`  LD      drivers/thermal/thermal_sys.o`<br>
`  CC      drivers/regulator/kryo-regulator.o`<br>
`  CC      drivers/staging/android/ion/ion_cma_heap.o`<br>
`  CC      drivers/soc/qcom/event_timer.o`<br>
`  CC      drivers/soc/qcom/rpm-smd-debug.o`<br>
`  CC      drivers/staging/android/sync.o`<br>
`  CC      drivers/soc/qcom/spm.o`<br>
`  CC      drivers/tty/vt/selection.o`<br>
`  CC      drivers/staging/android/ion/ion_cma_secure_heap.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiExt.o`<br>
`  CC      drivers/scsi/scsi_trace.o`<br>
`  CC      drivers/scsi/scsi_pm.o`<br>
`  CC      drivers/tty/vt/keyboard.o`<br>
`  CC      drivers/tty/vt/consolemap.o`<br>
`  CC      drivers/staging/android/sw_sync.o`<br>
`  CC      drivers/tty/serial/msm_serial_hs.o`<br>
`  LD      drivers/regulator/built-in.o`<br>
`  CC      drivers/tty/serial/msm_smd_tty.o`<br>
`  CC      drivers/usb/class/cdc-acm.o`<br>
`  CONMK   drivers/tty/vt/consolemap_deftbl.c`<br>
`  CC      drivers/staging/android/ion/compat_ion.o`<br>
`  LD      drivers/thermal/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiHCBB.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiInfo.o`<br>
`  LD      drivers/video/backlight/built-in.o`<br>
`  CC      drivers/scsi/ufs/ufs-qcom.o`<br>
`  CC      drivers/video/console/dummycon.o`<br>
`  CC      drivers/scsi/ufs/ufs-qcom-ice.o`<br>
`  CC      drivers/video/fbdev/core/fb_notify.o`<br>
`  CC      drivers/staging/android/oneshot_sync.o`<br>
`  LD      drivers/usb/class/built-in.o`<br>
`  CC      drivers/usb/common/common.o`<br>
`  LD      drivers/video/console/built-in.o`<br>
`  CC      drivers/video/fbdev/core/fb_cmdline.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_ctl.o`<br>
`  CC      drivers/video/fbdev/core/fbmem.o`<br>
`  CC      drivers/scsi/ufs/ufshcd.o`<br>
`  CC      drivers/staging/android/ion/msm/msm_ion.o`<br>
`  CC      drivers/soc/qcom/spm_devices.o`<br>
`  CC      drivers/staging/android/ion/msm/compat_msm_ion.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pipe.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_util.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/dsi_status_6g.o`<br>
`  LD      drivers/usb/common/usb-common.o`<br>
`  LD      drivers/usb/common/built-in.o`<br>
`  CC      drivers/usb/core/usb.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiLinkCntl.o`<br>
`  LD      drivers/tty/serial/built-in.o`<br>
`  LD      drivers/video/fbdev/omap2/displays-new/built-in.o`<br>
`  LD      drivers/video/fbdev/omap2/dss/built-in.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba.o`<br>
`  LD      drivers/video/fbdev/omap2/built-in.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_init.o`<br>
`  CC      drivers/tty/vt/vt.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_helpers.o`<br>
`  CC      drivers/scsi/sd.o`<br>
`  SHIPPED drivers/tty/vt/defkeymap.c`<br>
`  CC      drivers/tty/vt/consolemap_deftbl.o`<br>
`  CC      drivers/tty/vt/defkeymap.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/msm_dba/msm_dba_debug.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_debug.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/msm_dba/adv7533.o`<br>
`  LD      drivers/tty/vt/built-in.o`<br>
`  CC      drivers/soc/qcom/scm.o`<br>
`  LD      drivers/tty/built-in.o`<br>
`  LD      drivers/watchdog/built-in.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_cache_config.o`<br>
`  LD      drivers/staging/android/ion/msm/built-in.o`<br>
`  LD      drivers/staging/android/ion/built-in.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_video.o`<br>
`  CC      drivers/scsi/ufs/ufs_quirks.o`<br>
`  LD      drivers/staging/android/built-in.o`<br>
`  CC      drivers/video/fbdev/core/fbmon.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_cmd.o`<br>
`  CC      drivers/scsi/ufs/ufshcd-pltfrm.o`<br>
`  CC      drivers/usb/core/hub.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_intf_writeback.o`<br>
`  CC      drivers/video/fbdev/core/fbcmap.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiLinkSupervision.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiStatus.o`<br>
`  CC      drivers/scsi/ufs/ufs-debugfs.o`<br>
`  CC      drivers/usb/core/hcd.o`<br>
`  CC      drivers/usb/core/urb.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapApiTimer.o`<br>
`  CC      drivers/usb/core/message.o`<br>
`  CC      drivers/scsi/ufs/ufs-qcom-debugfs.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapModule.o`<br>
`  CC      drivers/usb/core/driver.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_rotator.o`<br>
`  CC      drivers/video/fbdev/core/fbsysfs.o`<br>
`  CC      drivers/soc/qcom/scm-boot.o`<br>
`  CC      drivers/soc/qcom/mpm-of.o`<br>
`  LD      drivers/video/fbdev/msm/../../msm/msm_dba/built-in.o`<br>
`  CC      drivers/soc/qcom/smem.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_overlay.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xAuthFsm.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xPrf.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_layer.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_splash_logo.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_cdm.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_smmu.o`<br>
`  CC      drivers/scsi/sg.o`<br>
`  CC      drivers/video/fbdev/core/modedb.o`<br>
`  CC      drivers/video/fbdev/core/fbcvt.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_wfd.o`<br>
`  CC      drivers/usb/core/config.o`<br>
`  CC      drivers/usb/core/file.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v1_7.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_v3.o`<br>
`  CC      drivers/soc/qcom/smem_debug.o`<br>
`  CC      drivers/soc/qcom/smd.o`<br>
`  CC      drivers/usb/dwc3/core.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_pp_common.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_mdp_debug.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsn8021xSuppRsnFsm.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsnAsfPacket.o`<br>
`  CC      drivers/soc/qcom/smd_debug.o`<br>
`  CC      drivers/usb/dwc3/debug.o`<br>
`  CC      drivers/video/fbdev/core/cfbfillrect.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmAesKeyWrap.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmEapol.o`<br>
`  CC      drivers/video/fbdev/core/cfbcopyarea.o`<br>
`  CC      drivers/video/fbdev/core/cfbimgblt.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug.o`<br>
`  CC      drivers/usb/core/buffer.o`<br>
`  CC      drivers/scsi/ch.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_debug_xlog.o`<br>
`  CC      drivers/usb/dwc3/trace.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi.o`<br>
`  CC      drivers/soc/qcom/smd_private.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_host.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsnSsmReplayCtr.o`<br>
`  LD      drivers/scsi/scsi_mod.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_cmd.o`<br>
`  LD      drivers/scsi/sd_mod.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_status.o`<br>
`  LD      drivers/video/fbdev/core/fb.o`<br>
`  LD      drivers/video/fbdev/core/built-in.o`<br>
`  CC      drivers/usb/dwc3/host.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/bapRsnTxRx.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/btampFsm.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_panel.o`<br>
`  CC      drivers/soc/qcom/smd_init_dt.o`<br>
`  LD      drivers/scsi/ufs/built-in.o`<br>
`  CC      drivers/usb/core/sysfs.o`<br>
`  CC      drivers/usb/gadget/usbstring.o`<br>
`  CC      drivers/usb/core/endpoint.o`<br>
`  CC      drivers/usb/gadget/config.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/msm_mdss_io_8974.o`<br>
`  CC      drivers/usb/core/devio.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_phy.o`<br>
`  LD      drivers/scsi/built-in.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dsi_clk.o`<br>
`  CC      drivers/usb/core/notify.o`<br>
`  CC      drivers/soc/qcom/smsm_debug.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_panel.o`<br>
`  CC      drivers/usb/core/generic.o`<br>
`  CC      drivers/usb/dwc3/gadget.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_util.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_edid.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_cec_core.o`<br>
`  CC      drivers/usb/dwc3/ep0.o`<br>
`  CC      drivers/usb/dwc3/debugfs.o`<br>
`  CC      drivers/usb/core/quirks.o`<br>
`  CC      drivers/usb/gadget/epautoconf.o`<br>
`  CC      drivers/soc/qcom/glink.o`<br>
`  CC      drivers/usb/gadget/composite.o`<br>
`  CC      drivers/usb/gadget/functions.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_dba_utils.o`<br>
`  CC      drivers/staging/prima/CORE/BAP/src/btampHCI.o`<br>
`  CC      drivers/usb/core/devices.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_io_util.o`<br>
`  CC      drivers/usb/host/pci-quirks.o`<br>
`  CC      drivers/soc/qcom/glink_debugfs.o`<br>
`  CC      drivers/soc/qcom/glink_ssr.o`<br>
`  CC      drivers/usb/dwc3/dwc3-pci.o`<br>
`  CC      drivers/usb/core/port.o`<br>
`  CC      drivers/usb/gadget/u_f.o`<br>
`  CC      drivers/usb/core/hcd-pci.o`<br>
`  CC      drivers/usb/gadget/debug.o`<br>
`  CC      drivers/usb/host/xhci-pci.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_tx.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_panel.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_hdcp2p2.o`<br>
`  CC      drivers/usb/host/xhci-plat.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_cec.o`<br>
`  CC      drivers/usb/dwc3/dwc3-msm.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_hdmi_audio.o`<br>
`  CC      drivers/usb/misc/ehset.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_wb.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_fb.o`<br>
`  CC      drivers/soc/qcom/glink_loopback_server.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_util.o`<br>
`  CC      drivers/video/fbdev/msm/../../msm/mdss/mdss_compat_utils.o`<br>
`  CC      drivers/usb/host/ehci-hcd.o`<br>
`  CC      drivers/usb/host/ehci-pci.o`<br>
`  CC      drivers/usb/gadget/function/f_acm.o`<br>
`  LD      drivers/usb/gadget/legacy/built-in.o`<br>
`  CC      drivers/usb/gadget/udc/udc-core.o`<br>
`  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-mdp.o`<br>
`  LD      drivers/video/fbdev/msm/../../msm/mdss/mdss-dsi.o`<br>
`  CC      drivers/usb/host/ehci-msm.o`<br>
`  CC      drivers/usb/host/xhci.o`<br>
`  CC      drivers/usb/host/xhci-mem.o`<br>
`  CC      drivers/usb/gadget/function/u_serial.o`<br>
`  CC      drivers/usb/host/xhci-ring.o`<br>
`  CC      drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe.o`<br>
`  LD      drivers/usb/core/usbcore.o`<br>
`  LD      drivers/usb/core/built-in.o`<br>
`  LD      drivers/usb/misc/built-in.o`<br>
`  CC      drivers/usb/mon/mon_main.o`<br>
`  CC      drivers/usb/mon/mon_stat.o`<br>
`  CC      drivers/usb/phy/phy.o`<br>
`  CC      drivers/usb/mon/mon_text.o`<br>
`  CC      drivers/usb/gadget/function/f_serial.o`<br>
`  LD      drivers/video/fbdev/msm/../../msm/mdss/built-in.o`<br>
`  CC      drivers/usb/serial/usb-serial.o`<br>
`  LD      drivers/video/fbdev/msm/../../msm/built-in.o`<br>
`  CC      drivers/usb/dwc3/dbm.o`<br>
`  LD      drivers/video/fbdev/msm/built-in.o`<br>
`  LD      drivers/usb/dwc3/dwc3.o`<br>
`  CC      drivers/usb/mon/mon_bin.o`<br>
`  LD      drivers/video/fbdev/built-in.o`<br>
`  LD      drivers/usb/dwc3/built-in.o`<br>
`  CC      drivers/usb/serial/generic.o`<br>
`  CC      drivers/usb/host/xhci-hub.o`<br>
`  CC      drivers/usb/host/xhci-dbg.o`<br>
`  CC      drivers/soc/qcom/glink_smd_xprt.o`<br>
`  LD      drivers/video/built-in.o`<br>
`  LD      drivers/usb/gadget/udc/built-in.o`<br>
`  CC      drivers/usb/gadget/android.o`<br>
`  CC      drivers/usb/gadget/ci13xxx_msm.o`<br>
`  CC      drivers/soc/qcom/glink_smem_native_xprt.o`<br>
`  CC      drivers/usb/phy/of.o`<br>
`  LD      drivers/usb/mon/usbmon.o`<br>
`  LD      drivers/usb/mon/built-in.o`<br>
`  CC      drivers/usb/phy/class-dual-role.o`<br>
`  CC      drivers/usb/storage/scsiglue.o`<br>
`  CC      drivers/usb/gadget/function/f_ncm.o`<br>
`  CC      drivers/usb/phy/phy-generic.o`<br>
`  CC      drivers/usb/storage/protocol.o`<br>
`  CC      drivers/usb/serial/bus.o`<br>
`  CC      drivers/staging/prima/CORE/DXE/src/wlan_qct_dxe_cfg_i.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/bap_hdd_main.o`<br>
`  CC      drivers/usb/phy/phy-msm-usb.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_assoc.o`<br>
`  CC      drivers/usb/gadget/function/f_ecm.o`<br>
`  CC      drivers/usb/host/xhci-trace.o`<br>
`  CC      drivers/usb/storage/transport.o`<br>
`  CC      drivers/usb/phy/phy-msm-hsusb.o`<br>
`  LD      drivers/usb/host/xhci-plat-hcd.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg.o`<br>
`  CC      drivers/usb/phy/phy-msm-ssusb-qmp.o`<br>
`  LD      drivers/usb/serial/usbserial.o`<br>
`  CC      drivers/usb/storage/usb.o`<br>
`  LD      drivers/usb/serial/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_debugfs.o`<br>
`  CC      drivers/usb/storage/initializers.o`<br>
`  CC      drivers/usb/storage/sierra_ms.o`<br>
`  CC      drivers/usb/storage/option_ms.o`<br>
`  CC      drivers/soc/qcom/smem_log.o`<br>
`  CC      drivers/usb/gadget/function/f_mass_storage.o`<br>
`  LD      drivers/usb/host/xhci-hcd.o`<br>
`  CC      drivers/usb/gadget/function/storage_common.o`<br>
`  CC      drivers/soc/qcom/smp2p.o`<br>
`  CC      drivers/soc/qcom/smp2p_debug.o`<br>
`  CC      drivers/usb/phy/phy-msm-qusb.o`<br>
`  CC      drivers/usb/gadget/function/f_fs.o`<br>
`  CC      drivers/soc/qcom/smp2p_sleepstate.o`<br>
`  CC      drivers/usb/storage/usual-tables.o`<br>
`  CC      drivers/usb/storage/alauda.o`<br>
`  CC      drivers/soc/qcom/smp2p_loopback.o`<br>
`  LD      drivers/usb/gadget/libcomposite.o`<br>
`  CC      drivers/usb/storage/cypress_atacb.o`<br>
`  CC      drivers/usb/storage/datafab.o`<br>
`  CC      drivers/usb/gadget/function/f_uac1.o`<br>
`  CC      drivers/soc/qcom/smp2p_test.o`<br>
`  CC      drivers/soc/qcom/smp2p_spinlock_test.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_dev_pwr.o`<br>
`  CC      drivers/usb/storage/freecom.o`<br>
`  CC      drivers/soc/qcom/qmi_interface.o`<br>
`  CC      drivers/usb/storage/isd200.o`<br>
`  CC      drivers/soc/qcom/ipc_router_smd_xprt.o`<br>
`  CC      drivers/soc/qcom/memshare/heap_mem_ext_v01.o`<br>
`  CC      drivers/usb/storage/jumpshot.o`<br>
`  CC      drivers/soc/qcom/memshare/msm_memshare.o`<br>
`  LD      drivers/usb/host/built-in.o`<br>
`  CC      drivers/usb/gadget/function/u_uac1.o`<br>
`  CC      drivers/usb/storage/karma.o`<br>
`  CC      drivers/usb/storage/sddr09.o`<br>
`  CC      drivers/usb/gadget/function/f_uac2.o`<br>
`  CC      drivers/usb/gadget/function/f_uvc.o`<br>
`  LD      drivers/soc/qcom/memshare/built-in.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/apr.o`<br>
`  LD      drivers/usb/phy/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_dp_utils.o`<br>
`  CC      drivers/soc/qcom/rpm_rbcpr_stats_v2.o`<br>
`  CC      drivers/soc/qcom/cpaccess64.o`<br>
`  CC      drivers/usb/storage/sddr55.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/apr_v2.o`<br>
`  CC      drivers/usb/gadget/function/uvc_queue.o`<br>
`  CC      drivers/soc/qcom/rpm_stats.o`<br>
`  CC      drivers/usb/gadget/function/uvc_v4l2.o`<br>
`  CC      drivers/soc/qcom/rpm_master_stat.o`<br>
`  CC      drivers/usb/storage/shuttle_usbat.o`<br>
`  CC      drivers/usb/gadget/function/uvc_video.o`<br>
`  CC      drivers/usb/gadget/function/f_audio_source.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_acm.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/apr_tal.o`<br>
`  CC      drivers/soc/qcom/rpm_rail_stats.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_serial.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_ncm.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_ecm.o`<br>
`  CC      drivers/soc/qcom/system_stats.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_mass_storage.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_fs.o`<br>
`  CC      drivers/soc/qcom/perf_event_l2.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_uac1.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/voice_svc.o`<br>
`  CC      drivers/soc/qcom/rpm_log.o`<br>
`  CC      drivers/soc/qcom/msm_tz_smmu.o`<br>
`  CC      drivers/soc/qcom/peripheral-loader.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_early_suspend.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_ftm.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_hostapd.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/msm_audio_ion.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_audio_source.o`<br>
`  CC      drivers/soc/qcom/qdsp6v2/adsp-loader.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_main.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_mib.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_uac2.o`<br>
`  CC      drivers/soc/qcom/subsys-pil-tz.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_oemdata.o`<br>
`  CC      drivers/soc/qcom/pil-q6v5.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_scan.o`<br>
`  LD      drivers/usb/storage/usb-storage.o`<br>
`  LD      drivers/usb/storage/ums-alauda.o`<br>
`  LD      drivers/usb/storage/ums-cypress.o`<br>
`  LD      drivers/usb/storage/ums-datafab.o`<br>
`  LD      drivers/usb/storage/ums-freecom.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_softap_tx_rx.o`<br>
`  LD      drivers/usb/storage/ums-isd200.o`<br>
`  LD      drivers/usb/storage/ums-jumpshot.o`<br>
`  CC      drivers/soc/qcom/pil-msa.o`<br>
`  LD      drivers/usb/storage/ums-karma.o`<br>
`  CC      drivers/soc/qcom/pil-q6v5-mss.o`<br>
`  LD      drivers/usb/gadget/function/usb_f_uvc.o`<br>
`  LD      drivers/usb/gadget/function/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_tx_rx.o`<br>
`  LD      drivers/usb/storage/ums-sddr09.o`<br>
`  LD      drivers/usb/storage/ums-sddr55.o`<br>
`  CC      drivers/soc/qcom/msm_performance.o`<br>
`  CC      drivers/soc/qcom/subsystem_notif.o`<br>
`  CC      drivers/soc/qcom/subsystem_restart.o`<br>
`  CC      drivers/soc/qcom/ramdump.o`<br>
`  CC      drivers/soc/qcom/sysmon.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_trace.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wext.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wmm.o`<br>
`  CC      drivers/soc/qcom/sysmon-qmi.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_wowl.o`<br>
`  CC      drivers/soc/qcom/secure_buffer.o`<br>
`  LD      drivers/usb/storage/ums-usbat.o`<br>
`  LD      drivers/usb/storage/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_cfg80211.o`<br>
`  CC      drivers/soc/qcom/icnss.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_p2p.o`<br>
`  CC      drivers/soc/qcom/wlan_firmware_service_v01.o`<br>
`  LD      drivers/soc/qcom/qdsp6v2/built-in.o`<br>
`  CC      drivers/soc/qcom/bam_dmux.o`<br>
`  CC      drivers/soc/qcom/scm-xpu.o`<br>
`  CC      drivers/soc/qcom/serial_num.o`<br>
`  CC      drivers/staging/prima/CORE/HDD/src/wlan_hdd_tdls.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgApi.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgDebug.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgParamName.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgProcMsg.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/cfg/cfgSendMsg.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/dph/dphHashTable.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAIDmgmt.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAdmitControl.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limApi.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limAssocUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limDebug.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limFT.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limIbssPeerMgmt.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limLinkMonitoringAlgo.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limLogDump.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limP2P.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessActionFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocReqFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAssocRspFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessAuthFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessBeaconFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessCfgUpdates.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDeauthFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessDisassocFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessLmmMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMessageQueue.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmReqMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessMlmRspMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeReqFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessProbeRspFrame.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessSmeReqMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limPropExtsUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limRMC.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limRoamingAlgo.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limScanResultUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSecurityUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendManagementFrames.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSendSmeRspMessages.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSerDesUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSession.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSessionUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limSmeReqUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limStaHashApi.o`<br>
`  LD      drivers/soc/qcom/built-in.o`<br>
`  LD      drivers/soc/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limTimerUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limTrace.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limUtils.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/lim/limProcessTdls.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmAP.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmApi.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/pmm/pmmDebug.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schApi.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconGen.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schBeaconProcess.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schDebug.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/sch/schMessage.o`<br>
`  CC      drivers/staging/prima/CORE/MAC/src/pe/rrm/rrmApi.o`<br>
`  CC      drivers/staging/prima/CORE/SAP/src/sapApiLinkCntl.o`<br>
`  CC      drivers/staging/prima/CORE/SAP/src/sapChSelect.o`<br>
`  CC      drivers/staging/prima/CORE/SAP/src/sapFsm.o`<br>
`  CC      drivers/staging/prima/CORE/SAP/src/sapModule.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/btc/btcApi.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/ccm/ccmApi.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/ccm/ccmLogDump.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_Api.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_FTApi.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/sme_common/sme_Trace.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrApiRoam.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrApiScan.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrCmdProcess.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrLinkList.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrLogDump.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrNeighborRoam.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrUtil.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/csr/csrTdlsProcess.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/oemData/oemDataApi.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/p2p/p2p_Api.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/pmc/pmcApi.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/pmc/pmc.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/pmc/pmcLogDump.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/QoS/sme_Qos.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/rrm/sme_rrm.o`<br>
`  CC      drivers/staging/prima/CORE/SME/src/nan/nan_Api.o`<br>
`  CC      drivers/staging/prima/CORE/SVC/src/btc/wlan_btc_svc.o`<br>
`  CC      drivers/staging/prima/CORE/SVC/src/nlink/wlan_nlink_srv.o`<br>
`  CC      drivers/staging/prima/CORE/SVC/src/ptt/wlan_ptt_sock_svc.o`<br>
`  CC      drivers/staging/prima/CORE/SVC/src/logging/wlan_logging_sock_svc.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/common/src/wlan_qct_sys.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palApiComm.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/pal/src/palTimer.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/platform/src/VossWrapper.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/system/src/macInitApi.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/system/src/sysEntryFunc.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/dot11f.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logApi.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/logDump.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/macTrace.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/parserApi.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsApi.o`<br>
`  CC      drivers/staging/prima/CORE/SYS/legacy/src/utils/src/utilsParser.o`<br>
`  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl.o`<br>
`  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_ba.o`<br>
`  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_hosupport.o`<br>
`  CC      drivers/staging/prima/CORE/TL/src/wlan_qct_tl_trace.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_api.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_event.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_getBin.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_list.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_lock.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_memory.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_mq.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_nvitem.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_packet.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_sched.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_threads.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_timer.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_trace.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_types.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_utils.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_parser.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_stream_read.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/wlan_nv_template_builtin.o`<br>
`  CC      drivers/staging/prima/CORE/VOSS/src/vos_diag.o`<br>
`  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda.o`<br>
`  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_debug.o`<br>
`  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_ds.o`<br>
`  CC      drivers/staging/prima/CORE/WDA/src/wlan_qct_wda_legacy.o`<br>
`  CC      drivers/staging/prima/CORE/WDA/src/wlan_nv.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_dp.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/CP/src/wlan_qct_wdi_sta.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_bd.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/DP/src/wlan_qct_wdi_ds.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/TRP/CTS/src/wlan_qct_wdi_cts.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/TRP/DTS/src/wlan_qct_wdi_dts.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_api.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_device.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_msg.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_packet.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_sync.o`<br>
`  LD      drivers/usb/gadget/g_android.o`<br>
`  LD      drivers/usb/gadget/built-in.o`<br>
`  LD      drivers/usb/built-in.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_timer.o`<br>
`  CC      drivers/staging/prima/CORE/WDI/WPAL/src/wlan_qct_pal_trace.o`<br>
`  LD      drivers/staging/prima/wlan.o`<br>
`  LD      drivers/staging/prima/built-in.o`<br>
`  LD      drivers/staging/built-in.o`<br>
`  LD      drivers/built-in.o`<br>
`  LINK    vmlinux`<br>
`  LD      vmlinux.o`<br>
`  MODPOST vmlinux.o`<br>
`  GEN     .version`<br>
`  CHK     include/generated/compile.h`<br>
`  UPD     include/generated/compile.h`<br>
`  CC      init/version.o`<br>
`  LD      init/built-in.o`<br>
`  KSYM    .tmp_kallsyms1.o`<br>
`  KSYM    .tmp_kallsyms2.o`<br>
`  LD      vmlinux`<br>
`  SORTEX  vmlinux`<br>
`  SYSMAP  System.map`<br>
`  OBJCOPY arch/arm64/boot/Image`<br>
`  DTC     arch/arm64/boot/dts/qcom/msm8953-qrd-sku3-mido.dtb`<br>
`  GZIP    arch/arm64/boot/Image.gz`<br>
`  CAT     arch/arm64/boot/Image.gz-dtb`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`Building DTBs`<br>
`make: Entering directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`make[1]: Entering directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`  CHK     include/config/kernel.release`<br>
`  GEN     ./Makefile`<br>
`  CHK     include/generated/uapi/linux/version.h`<br>
`  Using /home/stalker/hadk/kernel/xiaomi/msm8953 as source for kernel`<br>
`  CHK     include/generated/utsrelease.h`<br>
`  CALL    /home/stalker/hadk/kernel/xiaomi/msm8953/scripts/checksyscalls.sh`<br>
`make[1]: Leaving directory `/home/stalker/hadk/out/target/product/mido/obj/KERNEL_OBJ'`<br>
`make: Leaving directory `/home/stalker/hadk/kernel/xiaomi/msm8953'`<br>
`Kernel Modules not enabled`<br>
`[100% 5859/5859] Target boot image: /home/stalker/hadk/out/target/product/mido/boot.img`<br>
`/home/stalker/hadk/out/target/product/mido/boot.img maxsize=68395008 blocksize=135168 total=13373440 reserve=811008`<br>
`[100% 5859/5859] Install: /home/stalker/hadk/out/target/product/mido/hybris-recovery.img`<br>
``<br>
`#### make completed successfully (02:40 (mm:ss)) ####`<br>
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


Обновим манифест и добавим наши репозитории. В файл должен быть следующим
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

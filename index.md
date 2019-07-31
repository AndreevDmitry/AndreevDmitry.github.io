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
PlatformSDK:~$ ubu-chroot -r $PLATFORM_SDK_ROOT/sdks/ubuntu
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


Как видим, присутствует несколько ошибок и много предупреждений, желательно все это исправить. Как это сделать описано в блоке neochapay (https://neochapay.ru/blogs/zapiski-utkonosa-programmista/konfiguracija-jadra-dlja-sailfish-os.html)
Воспользуемся этой инструкцией

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
```console
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
```console
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

Производить поиск параметра можно с помощью символа "/", в поле ввода вносим имя конфига, но без префикса "CONFIG_". Для того чтобы перейти на требуемый пункт меню из режима поиска можно воспользовать соответствующими цифровыми клавишами (1) (2) и т.д.  и ставим "Y". Помните, что удовлетворить нужно все зависимости, например
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
Т.е. нужно также включить WATCHDOG
Видим, что не хватает WATCHDOG, а также видим что если нажать 1, то перейдем собственно к WATCHDOG и активируем его, ну а затем нужно будет войти в подменю и соответственно включить "Disable watchdog shutdown on close"

Или другой пример
```console
│ Symbol: NFS_ACL_SUPPORT [=n]
│ Type  : tristate
│   Defined at fs/Kconfig:254
│   Depends on: NETWORK_FILESYSTEMS [=y]
│   Selects: FS_POSIX_ACL [=y]
│   Selected by: NFS_FS [=y] && NETWORK_FILESYSTEMS [=y] && INET [=y] && FILE_LOCKING [=y] && NFS_V3_ACL [=n] || NFSD [=n] && NETWORK_FILESYSTEMS [=y] && INET [=y] && FILE_LOCKING [=y] && NFSD_V2_ACL [=n]
NFS_ACL_SUPPORT активируестя если выполнены условия из Selected by. Соответственно активируем NFS_V3_ACL (т.к он тоже фигурирует в нашем списке)
```

И так в том же духе...

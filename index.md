# Вольный перевод Sailfish OS HADK версии 3.0.1.0 (https://sailfishos.org/content/uploads/2019/03/SailfishOS-HardwareAdaptationDevelopmentKit-3.0.1.0.pdf) от 15 марта 2019 с примером портирования на Xiaomi Mido (а также проблемами, решениями и выводами вводимых команд)

**Любые действия Вы делаете на свой страх и риск, помните, что при неумелом обращении с некоторыми командами (например `sudo rm -rf / srv`) вы можете удалить корень системы, лучше делайте копипаст и проверяйте, а не перепечатывайте.**

**Пожалуйста, прежде чем делать что-либо дочитайте руководство до конца, чтобы не повторять описанных ошибок.**

Портирование производилось на Ubuntu 18.04 x64 cо следующими предустановленными программами:
- adb
- fastboot

# Установка переменных окружения
```bash
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
```bash
HOST:~$ cat <<'EOF' >> $HOME/.mersdkubu.profile
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }
export PS1="HABUILD_SDK [\${DEVICE}] $PS1"
hadk
EOF
```

На выходе получаем скрытый файл .mersdkubu.profile в вашей домашней директории

# Устанавливаем Platform SDK 2.1.1
Скачиваем архив песочницы крайней на момент написания перевода версии 2.1.1 в которой в дальшейшем будем собирать Sailfish
```bash
HOST:~$ curl -k -O http://releases.sailfishos.org/sdk/installers/latest/Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 ;
```
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
 100  137M  100  137M    0     0  14.4M      0  0:00:09  0:00:09 --:--:-- 17.9M<br>
</details><br>

Распаковываем скачанный архив
```bash
HOST:~$ sudo tar --numeric-owner -p -xjf Jolla-latest-SailfishOS_Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk;
```
Проверяем, что все распаковалось
```bash
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
```bash
HOST:~$ echo "export PLATFORM_SDK_ROOT=$PLATFORM_SDK_ROOT" >> ~/.bashrc
HOST:~$ echo 'alias sfossdk=$PLATFORM_SDK_ROOT/sdks/sfossdk/mer-sdk-chroot' >> ~/.bashrc ; exec bash ;
```

Соответственно в конец файла .bashrc должны были добавиться 2 строки, проверяем
```bash
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
```bash
HOST:~$ cat .mersdk.profile
```

<details>
```bash
PS1="PlatformSDK $PS1"
[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done
```
</details><br>


Заходим в песочницу,
```bash
HOST:~$ sfossdk
```

<details>
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
</details><br>

Соответственно видим, что все вводимые команды выполнятся из окружения Platform SDK (и как следствие имеем набор команд и параметров который предустановлен в данной песочнице), но находимся мы в домашнем каталоге.

Выходим из песочницы
```bash
PlatformSDK:~$ exit
```
Добавляем идентификацию устройства на которое будем портировать Sailfish
```bash
HOST:~$ cat <<'EOF' >> $HOME/.mersdk.profile
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }
hadk
EOF
```


Соответственно в конце файла .mersdk.profile добавились 2 строки, проверяем
```bash
HOST:~$ tail -n2 .mersdk.profile
```

<details>
function hadk() { source $HOME/.hadk.env; echo "Env setup for $DEVICE"; }<br>
hadk<br>
</details><br>

Заходим снова в PlatformSDK
```bash
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
```bash
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
```bash
PlatformSDK:~$ exit
```

Удаляем ее
```bash
HOST:~$ sudo rm -rf /srv
```

Скачиваем версию 2.0 для сборки Sailfish 3.0.2.8

```bash
HOST:~$ curl -k -O  http://releases.sailfishos.org/sdk/installers/2.0/Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2
```

<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100  135M  100  135M    0     0  10.2M      0  0:00:13  0:00:13 --:--:-- 11.6M
</details><br>

Создаем папку для установки песочницы
```bash
HOST:~$ sudo mkdir -p $PLATFORM_SDK_ROOT/sdks/sfossdk ;
```

Распаковываем скачанный архив
```bash
HOST:~$ sudo tar --numeric-owner -p -xjf Sailfish_OS-3.0.2.8-Platform_SDK_Chroot-i486.tar.bz2 -C $PLATFORM_SDK_ROOT/sdks/sfossdk  ;
```

Проверяем что все распаковалось, должно быть примерно так
```bash
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
```bash
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
```bash
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
```bash
PlatformSDK:~$ sudo zypper up
```
<details>
Loading repository data...<br>
Reading installed packages...<br>
<br>
Nothing to do.
</details><br>

Проверяем, что используем версию 3.0.3.10 Platform SDK
```bash
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
```bash
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
```bash
PlatformSDK:~$ TARBALL=ubuntu-trusty-20180613-android-rootfs.tar.bz2
PlatformSDK:~$ curl -O https://releases.sailfishos.org/ubu/$TARBALL
```
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100  440M  100  440M    0     0  16.7M      0  0:00:26  0:00:26 --:--:-- 17.2M
</details><br>

```bash
PlatformSDK:~$ UBUNTU_CHROOT=$PLATFORM_SDK_ROOT/sdks/ubuntu
PlatformSDK:~$ sudo mkdir -p $UBUNTU_CHROOT
```

Проверяем, комплектацию среды
```bash
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
```bash
HABUILD_SDK [mido]:~$ git config --global user.name "Your Name"
HABUILD_SDK [mido]:~$ git config --global user.email "you@example.com"
```

Далее установим программу для синхронизации локального хранилища с глобальным
```bash
HABUILD_SDK [mido]:~$ mkdir ~/bin
HABUILD_SDK [mido]:~$ PATH=~/bin:$PATH
HABUILD_SDK [mido]:~$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
```
<details>
% Total    % Received % Xferd  Average Speed   Time    Time     Time  Current<br>
                               Dload  Upload   Total   Spent    Left  Speed<br>
100 29142  100 29142    0     0  47424      0 --:--:-- --:--:-- --:--:-- 47462
</details><br>
```bash
HABUILD_SDK [mido]:~$ chmod a+x ~/bin/repo
```

Теперь нужно определиться с какой версией hybris будем синхронизироваться, т.к. на Mido есть две версии Android: (6 и 7).
Смотрим какая установлена у вас в Настройки-О телефоне, если 6 то нужно использовать hybris-13.0 (либо лучше обновиться до 7), если 7 то hybris-14.1.
Дальнейшее описание будет выполнено для версии 14.1

Создаем папку, где будет располагаться все, что нам нужно для сборки
```bash
HABUILD_SDK [mido]:~$ sudo mkdir -p $ANDROID_ROOT
HABUILD_SDK [mido]:~$ sudo chown -R $USER $ANDROID_ROOT
HABUILD_SDK [mido]:~$ cd $ANDROID_ROOT
```
Синхронизируемся с  репозиторием
```bash
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
```bash
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
```bash
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
```bash
HABUILD_SDK [mido] stalker@stalkerPC:~/hadk$ breakfast $DEVICE
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

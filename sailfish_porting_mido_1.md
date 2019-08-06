# Портируем Sailfish OS на Xiaomi Redmi Note 4x (mido)
## Приведенный ниже текст - это вольный перевод Sailfish OS [HADK версии 3.0.1.0](https://sailfishos.org/content/uploads/2019/03/SailfishOS-HardwareAdaptationDevelopmentKit-3.0.1.0.pdf) от 15 марта 2019 с примером портирования на Xiaomi Mido (а также проблемами, решениями и выводами команд)

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
PS1="PlatformSDK $PS1"<br>
[ -d /etc/bash_completion.d ] && for i in /etc/bash_completion.d/*;do . $i;done<br>
</details><br>

Заходим в песочницу,
```console
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

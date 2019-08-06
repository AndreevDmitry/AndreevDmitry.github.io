# Портируем Sailfish OS на Xiaomi Redmi Note 4x (mido)
## Приведенный ниже текст - это вольный перевод Sailfish OS [HADK версии 3.0.1.0](https://sailfishos.org/content/uploads/2019/03/SailfishOS-HardwareAdaptationDevelopmentKit-3.0.1.0.pdf) от 15 марта 2019 с примером портирования на Xiaomi Mido (а также проблемами, решениями и выводами команд)

**Любые действия Вы делаете на свой страх и риск, помните, что при неумелом обращении с некоторыми командами (например `sudo rm -rf / srv`) вы можете удалить корень системы, лучше делайте копипаст и проверяйте, а не перепечатывайте.**

**Пожалуйста, прежде чем делать что-либо дочитайте руководство до конца, чтобы не повторять описанных ошибок.**

Портирование производилось на Ubuntu 18.04 x64 cо следующими предустановленными программами:
- adb
- fastboot

1. [Установка среды для сборки](sailfish_porting/sailfish_porting_mido_1.md)
2. [Сборка ядра](sailfish_porting/sailfish_porting_mido_2.md)
3. [Сборка пакетов Sailfish](sailfish_porting/sailfish_porting_mido_3.md)
4. [Создание образа и установка Sailfish](sailfish_porting/sailfish_porting_mido_4.md)
5. [Попытка запуска системы с ядром ElectraBlue (TBD)](sailfish_porting/sailfish_porting_mido_5.md)
6. [Попытка запуска системы c модифицированными конфигурациями пакетов Sailfish](sailfish_porting/sailfish_porting_mido_6.md)

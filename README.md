# BI.ZONE Triage
Репозиторий содержит BI.ZONE Triage — бесплатный инструмент для сбора данных, необходимых для проведения расследования инцидентов, оценки компрометации и проактивного поиска угроз. Инструмент создан на базе решения [BI.ZONE EDR](https://bi.zone/catalog/products/edr/).

Последний релиз: [BI.ZONE Triage v1.7.0.1-beta](https://github.com/bi-zone/triage/releases/tag/Latest)

# Использование BI.ZONE Triage
Запуск BI.ZONE Triage возможен только с привилегиями root.

## Profiles и Presets
Для сбора данных используются Profiles и Presets.
Каждый Profile включают в себя набор данных операционной системы, сгруппированных по смыслу. Каждый Preset включает набор определенных Profiles.
Посмотреть список Profiles и Presets:
```bash
sudo ./bz_triage_nix -p=list
```
Необходимые для сбора Profiles перечисляются через запятую. Для исключения определенного Profile из Preset он указывается с префиксом **-**. Presets с  префиксом **-** не используются.

Пример:
```bash
sudo ./bz_triage_nix -p=all,-packages
```
Подробнее о Profiles и Presets [см. в документации на странице Wiki](https://github.com/bi-zone/triage/wiki/Инструкция-по-использованию#profiles-и-presets).

## Инвентаризация файлов в указанной директории
Инвентаризация файлов в указанной директории производится с помощью команды:
```bash
sudo ./bz_triage_nix --customdir=/usr/* --customdirdepth=5
```
Где:
- ```--customdir```: путь или список путей для инвентаризации с указанием через wildcard файлов;
- ```--customdirdepth```: глубина рекурсии при инвентаризации (по-умолчанию составляет 3).

### Использование Wildcard
При указании wildcard допускается использовать следующие символы:
- **\*** — для замены нескольких символов (в том числе 0);
- **\*\*** — в конце пути для рекурсивного поиска при инвентаризации;
- **?** — для замены одного символа.

### Примеры
Инвентаризация файлов без рекурсии:
```bash
sudo ./bz_triage_nix --customdir=/home/testuser*
```
Инвентаризация файлов с глубиной рекурсии по-умолчанию:
```bash
sudo ./bz_triage_nix --customdir=/home/testuser**
```
Инвентаризация файлов по списку директорий c глубиной рекурсии 4:
```bash
sudo ./bz_triage_nix --customdir=/home/somuser/dirs.txt --customdirdepth=4
```
Пример dirs.txt:
```
/home/*/Downloads/**
/etc/**/*.conf
/home/**/samp??.pdf
```

## Использование сканера YARA
Комбинации ключей YARA для сканирования:
- ```--yararules``` и ```--yaradir``` для сканирования указанной директории или списка директорий (**используется без wildcard**);
- ```--yararules``` и ```--yarapid``` для сканирования памяти и файлов процессов по PID. PIDs перечисляются через запятую, либо используется значение all для сканирования всех процессов;
- ```--yararules``` и ```--yarapname``` для сканирования памяти и файлов процессов по имени или его вхождению. Для поска по вхождению используйте символ **\*** в начале или конце указанной строки. Имена или строки для поиска вхождения можно перечислить через запятую;
- ```--yararules``` и ```-p yarascan``` для сканирования важных областей операционной системы (подробнее в  разделе [Сканирование важных областей операционной системы сканером YARA](#сканирование-важных-областей-операционной-системы-сканером-yara)).

В ```--yararules``` можно передать список фидов YARA для сканирования, для этого укажите путь к директории с фидами. Кроме фидов в указанной директории не должно быть других файлов, иначе сканер не сможет корректно вычитать наборы правил. При ошибках в синтаксисе любого из правил, все передаваемые правила будут проигнорированы.

Используйте ключ ```--yaradirdepth``` для указания глубины рекурсии. По-умолчанию глубина рекурсии 3.

### Сканирование важных областей операционной системы сканером YARA
Для сканирования важных областей операционной системы используется Preset ```yarascan``` совместно с ключом ```--yararules```. Этот Preset включает в себя набор Profiles, потенциально полезных для сканирования YARA: ```processes```, ```openfiles```, ```autoruns```, ```keydirsinfo```,```webdirinfo```. Команда позволяет просканировать их по вашему набору YARA-правил.

> При сканировании сканером YARA важных областей операционной системы в результатах работы отображается только информация о файлах и процессах, по которым сработало хотя бы одно YARA-правило.

## Примеры использования YARA
**Сканирование директории:**
```bash
sudo ./bz_triage_nix --yararules=./myrules.yar --yaradir=/tmp --yaradirdepth=5
```
**Сканирование списка путей по набору фидов:**
```bash
sudo ./bz_triage_nix --yararules=./yararules/ --yaradir=/home/testuser/dirspaths.txt
```
Файл dirspaths.txt:
```
/home/testuser/Downloads
/var/spool
/home/testuser/somefolder/sample.pdf
```
**Сканирование памяти и файлов процессов по PID:**
```bash
sudo ./bz_triage_nix --yararules=./myrules.yar --yarapid=5514,5617
```
**Сканирование памяти и файлов процессов по имени или его вхождению:**
```bash
sudo ./bz_triage_nix --yararules=./myrules.yar --yarapname=*systemd*,dbus*
```
**Сканирование важных областей операционной системы ([см. описание важных областей в Wiki](https://github.com/bi-zone/triage/wiki/Инструкция-по-использованию#сканирование-yara-важных-областей-операционной-системы)):**
```bash
sudo ./bz_triage_nix -p=yarascan --yararules=./myrules.yar
```

## Сохранение результатов работы
Если способ сохранения не указан, результаты работы BI.ZONE Triage сохраняются в текущую директорию в виде набора файлов JSON сгруппированных по Profiles.

Варианты сохранения результатов работы:
1. Локальное сохранение в архив:
```bash
sudo ./bz_triage_nix -p=all -o=./outdir/ -z --zipname=results.zip --zippwd=somepassword
```
Атрибуты ```--zipname``` и ```--zippwd``` опциональны. Если имя архива не указано, оно генерируется автоматически.

2. Вывод в STDOUT:
```bash
sudo ./bz_triage_nix -p=users --stdout 2>/dev/null
```
3. Отправка по сети в систему управления событиями кибербезопасности:
```bash
sudo ./bz_triage_nix -p=investigation,containers --dsthost=10.10.10.10 --dstport=5000 --netproto=tcp
```
4. Локальное сохранение в файлы JSON:
```bash
sudo ./bz_triage_nix -p=investigation,containers -o=/home/testuser
```
Если указать несколько вариантов сохранения, используется наиболее приоритетный способ согласно указанному выше порядку.

## Ограничения BI.ZONE Triage
### Ограничения при файловой инвентаризации
#### Ограничение при инвентаризации важных областей операционной системы
- в инвентаризацию попадают только файлы размером до 50Мб (файлы процессов без ограничений по размеру);
- в директориях **Downloads** и **tmp** инвентаризация ограничена по 2500 файлами;
- используемые методы хеширования: md5 и sha256;
- глубина рекурсии по умолчанию 3.

#### Ограничения при инвентаризации указанной директории
- в инвентаризацию попадают только файлы размером до 50Мб;
- используемые методы хеширования: md5 и sha256;
- глубина рекурсии по умолчанию 3.

### Ограничения сканера YARA
- сканируются только файлы размером до 100Мб;
- сканирование файла прерывается, если оно занимает более 20 секунд;
- относительные пути для аргумента ```--yaradir``` не поддерживаются.

# Cсылки
[BI.ZONE EDR](https://bi.zone/catalog/products/edr/)

[Примеры собираемых данных](event-examples)

[Wiki с описанием структуры собираемых данных](https://github.com/bi-zone/triage/wiki)

[Сообщить об ошибках и пожеланиях к функционалу BI.ZONE Triage](https://github.com/bi-zone/triage/issues)

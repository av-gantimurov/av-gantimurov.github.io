---
title: Blackout Ransomware
date: 2025-10-12 12:00:00 +0300
categories: [ "Malware analysis", Ransomware ]
tags: [ 4bid, monkey_bid, ransomware ]
media_subpath: /assets/2025_10_12/
---

|Критерий|Значение|
|:---|:---|
|Имя файла|1.exe|
|Размер|610332 байт (596.03 kB)|
|MD5|627ff0c45c9d62edb8e03a16c2ad87ce|
|SHA1|27a271c54b75a73dfec7b2aadd6617272c9a7ff2|
|SHA256|ba1c0b9d54ef9edc1ae8d3e2a1a21b7965f0c7339405667428c4549a77eba3bd|

**Ransomware.Blackout**

Шифровальщик семейства Blackout группировки 4BID.

Реализует следующие функциональные возможности:

1. Полное шифрование файлов алгоритмом AES128-GCM по списку расширений (T1486).
1. Шифрование файлов на сетевых дисках.
1. Отключение средств защиты (T1562.001).
1. Завершение процессов (T1057).
1. Остановка сервисов (T1490).
1. Выявление контролируемого окружение (песочниц).
1. Удаление теневых копий (T1490).
1. Отключение утилит настройки системы.
1. Очистка журналов Windows (T1070.001).

### Алгоритм работы

1. Создает мьютекс `BlackoutMutex` (T1480.002).
1. Если указано в настройках, то пытается отключить Defender путем завершения процессов `MsMpEng.exe`, `smartscreen.exe`.
1. Пытается получить токен `winlogon.exe` (T1134.001). Если не удалось получить токен `winlogon`, то создает сервис `Blackout Temporary Service` с текущим исполняемым файлом и его запускает в целях получить токен (T1569).
1. Если указано в настройках, то отключает перенаправление файловой системы для x86 процессов.
1. Если указано в настройках, то в случае выявления факта запуска в контролируемом окружении, завершает процесс (T1497.001).
1. Если указано в настройках, то завершает процессы по списку.
1. Если указано в настройках, то останавливает и удаляет сервисы по списку.
1. Если указано в настройках, то удаляет теневые копии.
1. Рекурсивно шифрует файлы с исключениями по списку.
1. Если указано в настройках, то создает иконку для зашифрованных файлов (c расширением `.blackout`).
1. Если указано в настройках, то меняет обои на рабочем столе.
1. Если указано сообщение о выкупе, то сохраняет его в файл `README.txt` в каждую директорию.
1. Если указано в настройках, то очищает журнал событий.
1. Открывает сообщение о выкупе.
1. Отключает пользователей `Administrator`, `Guest` (T1531).
1. Отключает через реестр менеджер задач и утилиты реестра, возможность запуска скриптов Powershell .
1. Если указано в настройках, то шифрует файлы на доступных сетевых дисках.
1. Если указано в настройках, то удаляет себя из системы через примерно 3000 секунд (T1070.004).

### Defense Evasion

Для усложнения выявления распространялся в упакованном при помощи UPX виде (T1027.002).

#### Sandbox Evasion

В настройках может быть указана опция `ANTI_ANALYSIS`. В таком случае ВПО пытается обнаружить факт запуска в контролируемом окружении и завершить работу.

1. Проверяется имя файла. Если имя файла содержит строки `virus`, `sample`, `sandbox`, то завершает работу.
2. Проверяются запущенные процессы. Если выявлены процессы с именами `wireshark`, `fiddler`, `ollydbg`, `x32dbg`, то завершает работу.

#### Очистка журналов

Если указана опция `DELETE_EVENTLOGS`, то очищает журналы `Application`, `Security`, `System` через вызов WinAPI `ClearEventLogA` (T1070.001).

### Persistence

Если указана опция `PERSISTENCE`, то устанавливает себя в автозагрузку через ключ реестра `[HKCU]\\Software\\Microsoft\\Windows\\CurrentVersion\\Run:Blackout` (T1547.001)

### Impact

#### Остановка процессов

Если указан опция `KILL_PROCESSES_FLAG`, то заверщает процессы по следующему списку:

- sql;
- oracle;
- ocssd;
- dbsnmp;
- synctime;
- agntsvc;
- isqlplussvc;
- xfssvccon;
- mydesktopservice;
- ocautoupds;
- encsvc;
- firefox;
- tbirdconfig;
- mydesktopqos;
- ocomm;
- dbeng50;
- sqbcoreservice;
- excel;
- infopath;
- msaccess;
- mspub;
- onenote;
- outlook;
- powerpnt;
- steam;
- thebat;
- thunderbird;
- visio;
- winword;
- wordpad;
- notepad;
- calc;
- wuauclt;
- onedrive;
- veeam;
- backup;
- sqlserver;
- backup;
- restore;
- recovery.

#### Остановка сервисов

Если указана опция `KILL_SERVICES_FLAG`, то останавливает и удаляет следующие сервисы:

- WinDefend;
- Sense;
- WdNisSvc;
- MsMpSvc;
- vss;
- sql;
- svc$;
- memtas;
- mepocs;
- msexchange;
- sophos;
- veeam;
- backup;
- GxVss;
- GxBlr;
- GxFWD;
- GxCVD;
- GxCIMgr;
- Veeam;
- Backup;
- Restore;
- Recovery;
- ShadowCopy;
- VolumeShadowCopy;

#### Удаление теневых копий

Если указана опция `DELETE_SHADOWS`, то удаляет теневые копии через команду `vssadmin delete shadows /all /quiet`

#### Шифрование файлов

Рекурсивно шифрует файлы при помощи алгоритма AES128-GCM. Ключ шифрования вшит в исполняемый файл и не меняется.
Для каждого файла вырабатывается IV, который сохраняется в конце зашифрованного файла.
В название файла добавляется расширение `.blackout`.

При шифровании пропускает следующие директории:

- $recycle.bin;
- config.msi;
- $windows.~bt;
- $windows.~ws;
- windows;
- boot;
- program files;
- program files (x86);
- programdata;
- system volume information;
- tor browser;
- windows.old;
- intel;
- msocache;
- perflogs;
- x64dbg;
- public;
- all users;
- default;
- microsoft;

Пропускает файлы со следующими названиями:

- README.txt;
- autorun.inf;
- boot.ini;
- bootfont.bin;
- bootsect.bak;
- desktop.ini;
- iconcache.db;
- ntldr;
- ntuser.dat;
- ntuser.dat.log;
- ntuser.ini;
- thumbs.db;
- GDIPFONTCACHEV1.DAT;
- d3d9caps.dat;

И файлы со следующими расширениями:

- 386;
- adv;
- ani;
- bat;
- bin;
- cab;
- cmd;
- com;
- cpl;
- cur;
- deskthemepack;
- diagcab;
- diagcfg;
- diagpkg;
- dll;
- drv;
- exe;
- hlp;
- icl;
- icns;
- ico;
- ics;
- idx;
- ldf;
- lnk;
- mod;
- mpa;
- msc;
- msp;
- msstyles;
- msu;
- nls;
- nomedia;
- ocx;
- prf;
- ps1;
- rom;
- rtp;
- scr;
- shs;
- spl;
- sys;
- theme;
- themepack;
- wpx;
- lock;
- key;
- hta;
- msi;
- pdb;
- search-ms;

### Конфигурация

Хранится в памяти в виде глобальных переменных. Возможные значения - `true`, `false`.

Конфигурация:

- `SKIP_HIDDEN_FOLDERS` - пропускать скрытые директории;
- `KILL_DEFENDER` - останавливать процессы Defender;
- `KILL_PROCESSES_FLAG` - останавливать процессы по списку;
- `KILL_SERVICES_FLAG` - отключать сервисы по списку;
- `DELETE_EVENTLOGS` - удаление событий из журналов по списку;
- `DELETE_SHADOWS` - удаление теневых копий;
- `ANTI_ANALYSIS` - завершение работы в случае выявления контролируемого окружения;
- `PERSISTENCE` - запись себя в автозагрузку;
- `NETWORK_SPREAD` - шифрование сетевых дисков;
- `DISABLE_FS_REDIRECTION` - отключения перенапраление файловой системы для процессов x86;
- `USE_RESTART_MANAGER` - не используется;
- `SET_WALLPAPER` - заменять обои на рабочем столе;
- `SET_ICONS` - устанавливать иконку для файлов с расширением `.blackout`;
- `SELF_DESTRUCT` - самоудаление;

### Особенности

Ранее не встречался, новый на момент сентября 2025 года.

В конфигурационном файле присутствует опция `USE_RESTART_MANAGER`, которая никак не используется.

Использовался группировкой [4BID](https://securelist.ru/sovmestnye-ataki-4bid-bo-team-red-likho/114124/).

#### Сообщение о выкупе

```text
==============================

!!! ATTENTION !!!

All your data has been encrypted with a unique key.

This key is additionally encrypted with our private key.

It is impossible to restore files without it.

We also downloaded some of the information.

If you do not get in touch, the data will be published and transferred to your competitors,

And the responsibility will fall on you.

As proof, we can decrypt one file for free.

You have 24 hours.

After that, the private key will be destroyed, and no one will be able to restore the data.

Monkey_BID@outlook.com

==============================
```

Конфигурация расположена по адресу 0x14ee4

```yaml
SKIP_HIDDEN_FOLDERS: false
KILL_DEFENDER: true
KILL_PROCESSES_FLAG: true
KILL_SERVICES_FLAG: true
DELETE_EVENTLOGS: true
DELETE_SHADOWS: true
ANTI_ANALYSIS: false
PERSISTENCE: true
NETWORK_SPREAD: true
DISABLE_FS_REDIRECTION: true
USE_RESTART_MANAGER: true
SET_WALLPAPER: false
SET_ICONS: true
SELF_DESTRUCT: true
```


```python
def extract_fld(ea: int):
    flag= idaapi.get_strlit_contents(ea, -1, idaapi.STRTYPE_C).decode()
    descr = idaapi.get_strlit_contents(ea +8, -1, idaapi.STRTYPE_C).decode()
    return(flag, descr)

def get_config(ea: int):
    while True:
        try:
            f, d = extract_fld(ea)
            print(f"{d}: {f}")
            ea += 50
        except AttributeError:
            break

get_config(0x7ff647df5ee4)
```

### Индикаторы компрометации

| Значение | Описание |
|:---|:---|
|`BlackoutMutex`|мьютекс для исключения запуска нескольких экземпляров шифровальщика|

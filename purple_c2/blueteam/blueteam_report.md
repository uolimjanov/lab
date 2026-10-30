# 🔵 Blue Team Hisoboti: Incident Response & Threat Hunting

**Sana:** 2026-yil 30-oktabr  
**Voqea Xulosasi:** Ruxsatsiz C2 aloqasi, zararli yuklovchining bajarilishi, huquqlarni oshirishga urinish va log boshqaruv tizimining (Splunk) kompromatga uchrashi.  
**Tahlil Metodologiyasi:** MITRE ATT&CK, NIST SP 800-61 (Incident Handling), DFIR (Digital Forensics and Incident Response).

---

## 1. Aniqlash va Tahlil (Detection & Analysis)

Bizning SIEM (Splunk) va EDR tizimlarimiz quyidagi IOC (Indicators of Compromise) larni aniqladi:

### Tarmoq (Network - T1071, T1105)
*   **C2 Aloqasi:** `172.20.10.5` IP manziliga va `80`-portga (HTTP) doimiy "heartbeat" so'rovlari. Cobalt Strike'ning "jitter" va `round-robin` xususiyati tufayli trafik notekis bo'lgan, lekin baribir anomaliya sifatida aniqlangan.
*   **Ma'lumot O'g'irlash (Exfiltration):** Splunk serveriga (`192.168.1.146`) ruxsatsiz kirish va `Forwarding and Receiving` sozlamalarining o'zgartirilishi. Bu loglarni tashqi muhitga yuborish urinishi sifatida baholandi.

### Jarayonlar (Process Execution - T1059, T1204)
*   **Bajarilish:** `syurpriz.exe` (imzosiz, noma'lum EXE) ning Telegram `Downloads` papkasidan ishga tushishi. Ota-ona jarayon: `telegram.exe`.
*   **Huquqlarni Oshirish:** `svc-exe` orqali xizmat yaratishga urinish (Event ID 7045 - New Service Installed). Shuningdek, `uac-token-duplication` orqali UAC Bypass urinishi (Event ID 4688).
*   **Credential Access:** `mimikatz` vositasining LSASS xotirasiga kirishga urinishi (Event ID 4656 va 4663 - Handle to LSASS Process).

### Fayl Tizimi (File System - T1083, T1105)
*   `C:\Users\zayd\Downloads\Telegram Desktop\` papkasida `syurpriz.exe` va `artifact.exe` fayllarining paydo bo'lishi.
*   `C:\Users\zayd\Desktop\macro_pack\` papkasidagi shubhali fayllar (`.docm` makrolar, `.exe` yuklovchilar).
*   `C:\Users\zayd\AppData\Roaming\` papkasiga `syurpriz.exe` nusxasining yaratilishi (Persistence uchun).

### Ro'yxatga Olish Tizimi (Registry - T1547)
*   `HKLM\Software\Microsoft\Windows\CurrentVersion\Run\WindowsUpdate` kalitining yaratilishi. Bu doimiylikni (Persistence) ta'minlash uchun qilingan.

---

## 2. Ta'sir Doirasi (Impact)
*   **Windows Host (`USER1234`):** To'liq kompromatga uchradi. Hujumchi buyruqlar bajarish, fayl tizimiga kirish va tizim huquqlarini oshirish imkoniyatiga ega bo'ldi.
*   **Splunk Infratuzilmasi:** Administrator huquqlari egallandi. Bu loglarni o'chirish, soxta loglar yaratish yoki boshqa tizimlarga hujum qilish uchun "sakrash taxtasi" (pivot) sifatida ishlatilishi mumkin.

---

## 3. Qarshi Choralar (Containment, Eradication, Recovery)
1.  **Izolyatsiya (Containment):** `USER1234` kompyuterini darhol tarmoqdan uzish (Network Isolation). Splunk serverining internetga va ichki tarmoqqa kirishini cheklash.
2.  **Bloklash (Eradication):** `172.20.10.5` IP manzilini va C2 aloqalarini tarmoq darajasida bloklash. `syurpriz.exe` faylining SHA256 hash'ini EDR/AV tizimlariga "qora ro'yxat"ga qo'shish.
3.  **Tozalash (Recovery):** Zararlangan Windows mashinasini qayta o'rnatish va barcha parollarni almashtirish. Splunk'dagi `admin` parolini va API tokenlarini zudlik bilan yangilash.
4.  **Tahlil (Lessons Learned):** `syurpriz.exe` faylini xavfsiz muhitda (sandbox) tahlil qilib, uning to'liq imkoniyatlarini aniqlash. Splunk'ga qanday qilib kirilganini (zaif parol, ochiq API yoki xavfsizlik devori xatosi) aniqlash.

---

## 4. Kelajakdagi Profilaktika (Prevention & Hardening)
*   **Ijtimoiy injiniringga qarshi (T1566):** Xodimlarni Telegram yoki boshqa messenjerlar orqali noma'lum `.exe` fayllarni yuklab olish va ishga tushirish xavfi bo'yicha o'qitish (Security Awareness Training).
*   **Tizim Xavfsizligi (T1548):** Windows tizimlarida AppLocker yoki WDAC orqali faqat ruxsat etilgan dasturlarning ishlashiga ruxsat berish. LSASS himoyasini (`RunAsPPL`) yoqish.
*   **Loglash (T1078):** Splunk kabi muhim tizimlarga kirishda ko'p faktorli autentifikatsiya (MFA) qo'llash va faqat ishonchli IP manzillardan kirishga ruxsat berish (IP Whitelisting).

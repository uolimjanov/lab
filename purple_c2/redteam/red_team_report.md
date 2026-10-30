# 🔴 Red Team Hisoboti: C2 Infratuzilmasi va Hujum Zanjiri

**Sana:** 2026-yil 30-oktabr  
**Operatsiya:**  C2 Lab  
**Hujum Vektori:** Ijtimoiy injiniring (Telegram) -> Malware Execution -> C2 Beacon -> Lateral Movement  
**Maqsad:** Tizimga kirish, huquqlarni oshirish va log infratuzilmasini (Splunk) egallash.

---

## 1. Razvedka va Infratuzilma (Reconnaissance & Setup)
*   **C2 Server:** Kali Linux muhitida `teamserver` ishga tushirildi.
    *   **C2 IP:** `172.20.10.5`
    *   **Teamserver Port:** `50050`
*   **Listener (Malleable C2):** `windows/beacon_http/reverse_http` protokoli asosida `win` nomli listener yaratildi.
    *   **Host Rotation Strategy:** `round-robin` (trafikni tahlil qilishni qiyinlashtirish uchun).
    *   **HTTP Port:** `80`
*   **Infratuzilma Xatolari:** `BOF_OperatorsKit.cna` va `RdpThief.cna` skriptlari topilmadi. Bu keyinchalik ba'zi imkoniyatlarni cheklab qo'ydi. Shuningdek, 80-portda `Web Server error: Socket closed` kuzatildi.

## 2. Payload Yetkazib Berish (Delivery & Execution)
*   **Payload Generatsiyasi:** Cobalt Strike orqali x64 arxitektura uchun `Windows Executable (EXE)` yaratildi.
*   **Maskirovka (T1036):** Fayl ijtimoiy injiniring usuli bilan **`syurpriz.exe`** deb nomlandi.
*   **Vektor (T1566):** Fayl Telegram Desktop orqali qurbonga yuborildi (Fayl hajmi: 17.5 Kb).
*   **Execution (T1204):** Qurbon (`zayd`) faylni ishga tushirdi. Beacon `USER1234` kompyuterida, PID `6196` ostida muvaffaqiyatli Check-in qildi.

## 3. Post-Exploitation va Privilege Escalation
*   **Huquqlarni Oshirish (T1548):** 
    *   `elevate svc-exe` buyrug'i bajarildi (UAC bypass urinishi).
    *   *Natija:* `could not upload file` va `could not open service control manager` xatoliklari sababli muvaffaqiyatsizlikka uchradi.
*   **Credential Access (T1003):** 
    *   `mimikatz` yordamida LSASS xotirasidan parollarni o'g'irlashga urinildi.
    *   *Natija:* FQDN xatosi va LSASS xotirasiga kirishda muammo (`Handle on memory 0x00000005`) kuzatildi.
*   **Discovery (T1083):** 
    *   Qurbonning `C:\Users\zayd\Desktop\` papkasi o'rganildi. `macro_pack` papkasi va ichidagi fayllar (`.git`, `alive.docm`, `macro_pack.exe`) aniqlangan. Bu keyingi hujumlar uchun potentsial nishon bo'lishi mumkin.
    *   `C:\Users\zayd\Downloads\Telegram Desktop\` papkasida `artifact.exe` va `syurpriz.exe` aniqlandi.
*   **Lateral Movement va Pivoting:** 
    *   Hujumchi tarmoqdagi **Splunk Enterprise** (`192.168.1.146:8000`) tizimiga `admin` huquqlari bilan kirishga erishdi. 
    *   Splunk'ning "Yuborish va qabul qilish" (Forwarding and receiving) panelida sozlamalar o'zgartirildi. Bu loglarni manipulyatsiya qilish imkoniyatini beradi.

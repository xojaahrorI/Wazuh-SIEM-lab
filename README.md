# Wazuh SIEM — Security Monitoring Lab

**Wazuh 4.14.10 | Ubuntu Server | Windows Server | VMware Workstation**

## 1. Loyiha haqida

Ushbu loyiha Wazuh SIEM platformasi yordamida Windows Server tizimining xavfsizlik hodisalarini markazlashgan holda kuzatish va tahlil qilish uchun tashkil etilgan amaliy laboratoriya hisoblanadi.

Laboratoriyada Ubuntu Server Wazuh Manager sifatida sozlangan. Windows Server'ga Wazuh Agent o‘rnatilib, markaziy monitoring serveriga ulangan.

Wazuh Dashboard orqali Agent holati va xavfsizlik alertlarini kuzatish amalga oshiriladi.

## 2. Loyiha maqsadlari

- Wazuh SIEM platformasini o‘rnatish va sozlash.
- Ubuntu Server'da markaziy monitoring muhitini yaratish.
- Windows Server'ga Wazuh Agent o‘rnatish.
- Agent va Manager o‘rtasidagi aloqani sozlash.
- Windows xavfsizlik hodisalarini monitoring qilish.
- Security Event ID'larni o‘rganish va tahlil qilish.
- Xavfsizlik alertlarini Wazuh Dashboard orqali kuzatish.
- Tarmoq va ulanish muammolarini diagnostika qilish.

## 3. Laboratoriya arxitekturasi

| Komponent | Operatsion tizim | IP manzil |
|---|---|---|
| Wazuh Manager | Ubuntu Server | `192.168.90.130` |
| Wazuh Agent | Windows Server | `192.168.90.4` |

**Ishlash sxemasi:**

```text
        Windows Server
         192.168.90.4
               |
          Wazuh Agent
               |
          TCP 1514
          TCP 1515
               |
               v
        Ubuntu Server
        192.168.90.130
               |
         Wazuh Manager
               |
         Wazuh Indexer
               |
        Wazuh Dashboard
               |
          Web Browser
```

## 4. Ishlatilgan texnologiyalar

- **Wazuh 4.14.10** — SIEM va xavfsizlik monitoringi.
- **Ubuntu Server** — Wazuh server muhiti.
- **Windows Server** — monitoring qilinadigan endpoint.
- **VMware Workstation** — virtual laboratoriya muhiti.
- **PowerShell** — Windows konfiguratsiyasi va diagnostikasi.
- **Linux Terminal** — Ubuntu server boshqaruvi.

## 5. Wazuh komponentlari

| Komponent | Vazifasi |
|---|---|
| Wazuh Manager | Agentlardan kelgan ma'lumotlarni qabul qiladi va tahlil qiladi. |
| Wazuh Indexer | Monitoring ma'lumotlarini saqlaydi va qidirish imkonini beradi. |
| Wazuh Dashboard | Ma'lumotlarni grafik va jadvallar orqali ko‘rsatadi. |
| Wazuh Agent | Windows tizimidagi hodisalarni yig‘ib, Manager'ga yuboradi. |

## 6. Muhim tarmoq portlari

| Port | Protokol | Vazifasi |
|---|---|---|
| 1514 | TCP | Agent va Manager o‘rtasidagi aloqa |
| 1515 | TCP | Agent ro‘yxatdan o‘tishi (Enrollment) |
| 55000 | TCP | Wazuh API |
| 443 | TCP | Dashboard HTTPS |

## 7. O‘rnatish va sozlash

### 7.1. Ubuntu Server'da Wazuh o‘rnatish

O‘rnatish skriptini yuklab olish:

```bash
cd ~
curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh
```

All-in-one o‘rnatishni boshlash:

```bash
sudo bash ./wazuh-install.sh -a
```

Xizmatlarni tekshirish:

```bash
sudo systemctl status wazuh-manager
sudo systemctl status wazuh-indexer
sudo systemctl status wazuh-dashboard
```

### 7.2. Windows Server'ga Wazuh Agent o‘rnatish

PowerShell'ni Administrator sifatida ishga tushirish.

Agent MSI paketini yuklab olish:

```powershell
$msi = "$env:TEMP\wazuh-agent-4.14.10-1.msi"

Invoke-WebRequest `
-Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.10-1.msi" `
-OutFile $msi
```

Agent'ni Manager manzili bilan o‘rnatish:

```powershell
msiexec.exe /i $msi /q WAZUH_MANAGER="192.168.90.130" WAZUH_AGENT_NAME="Windows-Server"
```

Agent xizmatini ishga tushirish:

```powershell
Start-Service WazuhSvc
```

Xizmat holatini tekshirish:

```powershell
Get-Service WazuhSvc
```

### 7.3. Tarmoq aloqasini tekshirish

Windows Server'dan Ubuntu Wazuh Manager portlarini tekshirish:

```powershell
Test-NetConnection 192.168.90.130 -Port 1514

Test-NetConnection 192.168.90.130 -Port 1515
```

`TcpTestSucceeded : True` natijasi TCP ulanishi muvaffaqiyatli ekanini bildiradi.

## 8. Amalda uchragan muammolar va yechimlar

### 1. Wazuh xizmatini ishga tushirishdagi xato

**Xato:**

```text
The service name is invalid.
NET HELPMSG 2185
```

**Sabab:** Windows xizmatining nomi noto‘g‘ri ko‘rsatilgan.

**Yechim:**

```powershell
Get-Service *wazuh*
Start-Service WazuhSvc
```

### 2. Agent Manager'ga ulana olmasligi

**Xato:**

```text
Unable to connect to enrollment service
at '[192.168.90.4]:1515'
```

**Sabab:** Agent sozlamalarida Manager manzili sifatida Windows Server IP manzili ko‘rsatilgan.

**Yechim:** Manager manzili Ubuntu Server IP manziliga almashtirildi:

```text
192.168.90.130
```

### 3. TCP portga ulanish muvaffaqiyatsizligi

**Xato:**

```text
TcpTestSucceeded : False
```

**Tekshirish:**

- Manager IP manzilining to‘g‘riligini tekshirish.
- Ubuntu Firewall sozlamalarini tekshirish.
- Wazuh Manager xizmatlari ishlayotganini tekshirish.
- 1514 va 1515 portlarini tekshirish.

### 4. Netplan konfiguratsiyasi xatosi

**Xato:**

```text
Error in network definition:
unknown key 'dhcp4'
```

**Sabab:** YAML faylidagi indentatsiya yoki parametr joylashuvi noto‘g‘ri.

**Yechim:** Netplan konfiguratsiyasidagi `dhcp4` parametrini tegishli Ethernet interfeysi ichiga joylashtirish.

## 9. Windows Security Event ID tahlili

Laboratoriyada quyidagi Windows Security Event ID'lar o‘rganish uchun muhim hisoblanadi:

| Event ID | Hodisa |
|---|---|
| 4624 | Muvaffaqiyatli tizimga kirish |
| 4625 | Muvaffaqiyatsiz tizimga kirish |
| 4634 | Tizimdan chiqish |
| 4648 | Boshqa credential bilan tizimga kirish |
| 4672 | Maxsus imtiyozlar berilgan login |
| 4720 | Yangi foydalanuvchi yaratish |
| 4722 | Foydalanuvchi hisobini yoqish |
| 4725 | Foydalanuvchi hisobini o‘chirib qo‘yish |

Wazuh Dashboard'da muvaffaqiyatsiz login hodisalarini qidirish uchun:

```text
data.win.system.eventID:4625
```

Muvaffaqiyatli va muvaffaqiyatsiz loginlarni birgalikda qidirish:

```text
data.win.system.eventID:(4624 OR 4625)
```

## 10. Amaliy natijalar

Wazuh Dashboard orqali laboratoriyaning ishlash holati tekshirildi.

Dashboard'da kuzatilgan natijalar:

| Ko‘rsatkich | Natija |
|---|---:|
| Active Agents | 2 |
| Disconnected Agents | 0 |
| Critical Alerts | 0 |
| High Alerts | 0 |
| Medium Alerts | 34 |
| Low Alerts | 75 |

*Ushbu ko‘rsatkichlar Dashboard skrinshoti olingan vaqtdagi oxirgi 24 soatlik statistikani aks ettiradi.*

## 11. Dashboard

Wazuh Dashboard orqali Agent holati, xavfsizlik alertlari va monitoring ma'lumotlari kuzatildi.

![Wazuh Dashboard](screenshots/wazuh-dashboard.png)

## 12. Loyiha davomida o‘rganilgan ko‘nikmalar

- Linux server boshqaruvi.
- Windows Server bilan ishlash.
- SIEM platformasini o‘rnatish va sozlash.
- Wazuh Agent va Manager konfiguratsiyasi.
- TCP portlar va tarmoq diagnostikasi.
- Windows Security Event Log tahlili.
- Xavfsizlik alertlarini kuzatish.
- Muammolarni aniqlash va bartaraf etish.

## 13. Repository tuzilishi

```text
Wazuh-SIEM-Lab/
│
├── README.md
│
├── screenshots/
│   └── wazuh-dashboard.png
│
└── documentation/
    └── Wazuh_Laboratoriya.pdf
```

## 14. Xulosa

Ushbu laboratoriya orqali Wazuh SIEM platformasi asosida markaziy xavfsizlik monitoring muhiti tashkil etildi. Ubuntu Server'da Wazuh komponentlari sozlandi va Windows Server Agent monitoring tizimiga ulandi.

Dashboard orqali Agent holati va xavfsizlik alertlarini kuzatish amaliyoti bajarildi.

**Project:** Wazuh SIEM Security Monitoring Lab  
**Version:** Wazuh 4.14.10  
**Environment:** VMware Workstation  
**Status:** Dashboard'da 2 ta Active Agent ko‘rsatilgan.

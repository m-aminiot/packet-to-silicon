# راهنمای جامع ابزار nmap (استاندارد اکتشاف شبکه و ممیزی امنیتی پورت‌ها)

## معرفی ابزار و نقش آن در اکتشاف شبکه

ابزار [`nmap`](https://nmap.org/book/man.html) (مخفف Network Mapper، توسعه‌یافته توسط Gordon "Fyodor" Lyon) شناخته‌شده‌ترین ابزار متن‌باز برای اکتشاف شبکه (Network Discovery)، شناسایی میزبان‌های فعال، اسکن پورت‌ها، تشخیص نسخه دقیق سرویس‌ها، تشخیص سیستم‌عامل و اجرای اسکریپت‌های ممیزی امنیتی (NSE) است ([Linux Manpage: nmap(1)](https://linux.die.net/man/1/nmap)).

این ابزار با ارسال بسته‌های خام IP به میزبان‌های هدف و تحلیل دقیق پاسخ‌های برگشتی (و حتی نحوه عدم پاسخگویی)، وضعیت پورت‌ها را در دسته‌هایی مانند `open` (باز)، `closed` (بسته) یا `filtered` (فیلترشده توسط فایروال) طبقه‌بندی می‌کند ([Nmap Reference Guide](https://nmap.org/book/man.html)).

## نصب ابزار در لینوکس

برای نصب [`nmap`](https://nmap.org/book/man.html) در توزیع‌های مبتنی بر دبیان و اوبونتو:

```bash
sudo apt install nmap
```

## جدول کامل فلگ‌ها و سوئیچ‌های پرکاربرد

بر اساس مرجع رسمی [`Nmap Options Summary`](https://nmap.org/book/man-briefoptions.html)، مهم‌ترین فلگ‌های اسکن و شناسایی به شرح زیر هستند:

| فلگ / سوئیچ | دسته عملکردی | توضیح دقیق عملکرد و کاربرد فنی | مرجع رسمی |
| :--- | :--- | :--- | :--- |
| `-sS` | تکنیک اسکن | اسکن نیمه‌باز TCP SYN (Stealth Scan)؛ ارسال بسته `SYN` و عدم تکمیل دست‌تکان سه‌مرحله‌ای پس از دریافت `SYN/ACK` (نیازمند دسترسی `sudo`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sT` | تکنیک اسکن | اسکن اتصال کامل TCP Connect؛ استفاده از فراخوان سیستمی `connect()` سیستم‌عامل زمانی که دسترسی به سوکت خام وجود ندارد. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sU` | تکنیک اسکن | اسکن پورت‌های پروتکل UDP (مانند DNS روی پورت ۵۳، SNMP روی ۱۶۱ و DHCP روی ۶۷/۶۸). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sA` / `-sN` / `-sF` / `-sX` | تکنیک اسکن | اسکن‌های تخصصی TCP ACK (`-sA` برای نقشه‌برداری قوانین فایروال)، Null (`-sN`)، FIN (`-sF`) و Xmas (`-sX`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sn` | کشف میزبان | اسکن پینگ (Ping Scan)؛ تنها شناسایی میزبان‌های روشن در شبکه بدون اسکن کردن پورت‌های آن‌ها. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-Pn` | کشف میزبان | فرض کردن روشن بودن تمامی اهداف و پرش از مرحله کشف اولیه میزبان (مفید برای عبور از فایروال‌هایی که Ping را مسدود کرده‌اند). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-p <range>` | انتخاب پورت | تعیین پورت یا بازه پورت‌ها برای اسکن (مانند `-p 22,80,443` یا `-p 1-1000`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-p-` | انتخاب پورت | اسکن پورت‌های ۱ تا ۶۵۵۳۵ برای پروتکل‌هایی که نوع اسکنشان فعال شده است؛ به‌تنهایی اسکن هم‌زمان TCP و UDP نیست. برای UDP باید `-sU` و برای TCP یک روش TCP مثل `-sS` یا `-sT` نیز مشخص شود. | [Nmap Reference](https://nmap.org/book/man-port-specification.html) |
| `-F` / `--top-ports <n>` | انتخاب پورت | حالت سریع (`-F` برای اسکن ۱۰۰ پورت پرکاربرد) یا اسکن `<n>` پورت پرتکرار با `--top-ports`. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sV` | تشخیص سرویس | ارسال کاوشگرهای تخصصی به پورت‌های باز جهت شناسایی نام سرویس و نسخه دقیق نرم‌افزار در حال اجرا. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-O` | تشخیص سیستم‌عامل | شناسایی نوع و نسخه سیستم‌عامل هدف (OS Detection) از طریق تحلیل اثرانگشت پشته TCP/IP. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-sC` / `--script=<name>` | موتور اسکریپت (NSE) | اجرای اسکریپت‌های پیش‌فرض Nmap (`-sC`) یا اسکریپت‌های مشخص‌شده برای کشف آسیب‌پذیری و اطلاعات تکمیلی. | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-A` | اسکن تهاجمی جامع | فعال‌سازی هم‌زمان تشخیص سیستم‌عامل (`-O`)، تشخیص نسخه سرویس (`-sV`)، اسکریپت‌های پیش‌فرض (`-sC`) و مسیریابی (`--traceroute`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-T<0-5>` | زمان‌بندی و سرعت | تنظیم قالب سرعت اسکن از `T0` (بسیار آهسته/Paranoid) تا `T4` (سریع/Aggressive) و `T5` (فوق‌سریع/Insane). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-oN` / `-oX` / `-oG` / `-oA` | خروجی و گزارش | ذخیره نتایج اسکن در فایل به فرمت متنی عادی (`-oN`)، XML (`-oX`)، قابل جستجو با Grep (`-oG`) یا هر سه فرمت با هم (`-oA`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |
| `-v` / `--open` | نمایش جزئیات | افزایش سطح جزئیات خروجی (`-v` یا `-vv`) و نمایش تنها پورت‌های باز (`--open`). | [Nmap Options](https://nmap.org/book/man-briefoptions.html) |

## سناریوهای کاربردی و تحلیل نمونه دستورات

### ۱. اسکن کامل تمامی پورت‌های زیرشبکه همراه با تشخیص نسخه سرویس (نمونه دستور تصویر)

این دستور میزبان‌های پاسخ‌گو در محدودهٔ `192.168.1.0/24` را هدف می‌گیرد و روی هرکدام تمام پورت‌های TCP را با روش SYN و تشخیص نسخهٔ سرویس بررسی می‌کند. این اسکن می‌تواند زمان‌بر و پرحجم باشد؛ ابتدا فقط روی سامانه‌های خودتان یا با مجوز صریح اجرا کنید و برای شروع از یک میزبان و بازهٔ محدود پورت استفاده کنید ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)):

```bash
sudo nmap -sS -sV -p- 192.168.1.0/24
```

* **`sudo`**: تامین دسترسی ریشه برای ساخت بسته‌های خام در اسکن `-sS` ([Nmap Reference Guide](https://nmap.org/book/man.html)).
* **`-sS`**: اجرای اسکن TCP SYN که سریع‌تر است و اتصال کامل در لاگ برخی برنامه‌های کاربردی ثبت نمی‌کند ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)).
* **`-sV`**: بررسی بنر و پاسخ پروتکل‌ها برای تشخیص دقیق نرم‌افزار و نسخه سرویس روی هر پورت باز (مثلاً تشخیص نسخه OpenSSH یا Apache) ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)).
* **`-p-`**: اسکن کامل پورت‌های ۱ تا ۶۵۵۳۵ تا سرویس‌های اجراشده روی پورت‌های غیراستاندارد نیز کشف شوند ([Nmap Reference Guide](https://nmap.org/book/man.html)).
* **`192.168.1.0/24`**: هدف قرار دادن کل رنج شبکه محلی با پیشوند CIDR `/24` ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)).

### ۲. کشف سریع میزبان‌های زنده در شبکه بدون اسکن پورت

اگر فقط می‌خواهید بدانید چه دستگاه‌هایی در حال حاضر به شبکه متصل هستند ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)):

```bash
sudo nmap -sn 192.168.1.0/24
```

### ۳. اسکن جامع یک سرور خاص و ذخیره گزارش در سه فرمت

برای انجام اسکن کامل (سرویس، سیستم‌عامل و اسکریپت‌های پیش‌فرض) با سرعت بالا و ذخیره خروجی ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)):

```bash
sudo nmap -A -T4 -oA scan_report 192.168.1.10
```

## کاربردهای عملی در مدیریت و امنیت شبکه

* **شناسایی سرویس‌های مجاز و غیرمجاز (Shadow IT)**: کشف پورت‌های باز فراموش‌شده یا سرویس‌هایی که روی پورت‌های غیرمتعارف اجرا می‌شوند ([Linux Manpage: nmap(1)](https://linux.die.net/man/1/nmap)).
* **ممیزی نسخه‌های آسیب‌پذیر**: تطبیق نسخه دقیق سرویس‌های شناسایی‌شده با `-sV` با پایگاه‌های داده آسیب‌پذیری (CVE) ([Nmap Reference Guide](https://nmap.org/book/man.html)).
* **پایش دوره‌ای تغییرات شبکه**: ذخیره خروجی با فلگ `-oX` و مقایسه اسکن‌های دوره‌ای برای شناسایی دستگاه‌ها یا پورت‌های جدید در زیرساخت ([Nmap Options Summary](https://nmap.org/book/man-briefoptions.html)).

# راهنمای جامع ابزار ettercap (چارچوب حملات مرد میانی و دستکاری لایه کاربردی)

## معرفی ابزار و معماری شنود یکپارچه

ابزار [`ettercap`](https://linux.die.net/man/8/ettercap) (طراحی‌شده توسط Alberto Ornaghi و Marco Valleri) یک چارچوب جامع برای اجرای حملات مرد میانی (MitM)، شنود فعال و غیرفعال پروتکل‌ها، فیلترینگ محتوای بسته‌ها در لحظه (On-the-fly Content Filtering) و اجرای پلاگین‌های لایه کاربردی در شبکه‌های محلی است ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).

این ابزار از دو روش اصلی شنود پشتیبانی می‌کند ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)):
* **شنود یکپارچه (Unified Sniffing)**: دریافت تمام بسته‌های عبوری از یک کارت شبکه و هدایت خودکار بسته‌ها در لایه ۳ توسط موتور داخلی [`ettercap`](https://linux.die.net/man/8/ettercap) (در این حالت، ابزار به صورت خودکار فورواردینگ هسته لینوکس را غیرفعال می‌کند تا از ارسال دوباره بسته‌ها جلوگیری شود).
* **شنود پلی (Bridged Sniffing)**: قرارگیری نامحسوس در لایه ۱ بین دو کارت شبکه فیزیکی و عبور دادن ترافیک میان آن‌ها همراه با قابلیت تغییر یا حذف بسته‌ها ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).

## نصب و فایل‌های پیکربندی مهم

برای نصب نسخه خط فرمان (سبک و بدون پیش‌نیازهای گرافیکی) در اوبونتو و دبیان:

```bash
sudo apt install ettercap-text-only
```

فایل‌های پیکربندی کلیدی ابزار عبارت‌اند از:
* [`/etc/ettercap/etter.conf`](https://manpages.ubuntu.com/manpages/focal/man5/etter.conf.5.html): تنظیمات پایه، شناسه کاربری (`EC_UID`) و دستورات تغییر مسیر پورت برای رهگیری SSL.
* `/etc/ettercap/etter.dns`: فایل تعریف رکوردهای DNS جعلی که توسط پلاگین [`dns_spoof`](https://manpages.ubuntu.com/manpages/focal/man8/ettercap_plugins.8.html) برای هدایت دامنه‌ها به آدرس IP دلخواه استفاده می‌شود.

## ساختار تعریف اهداف (Target Specification)

در [`ettercap`](https://linux.die.net/man/8/ettercap)، اهداف به صورت دوطرفه و با فرمت استاندارد `MAC/IPs/IPv6/PORTs` تعریف می‌شوند (یا در شبکه‌های IPv4 کلاسیک به شکل `MAC/IPs/PORTs`). خالی گذاشتن هر بخش به معنای `ANY` (تمامی مقادیر) است ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)):
* `//`: تمامی آدرس‌های MAC، تمامی آدرس‌های IP و تمامی پورت‌ها در زیرشبکه ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* `/192.168.1.16//`: هر آدرس MAC، تنها آدرس IP برابر با `192.168.1.16` و هر پورتی ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).

## جدول کامل فلگ‌ها و سوئیچ‌های خط فرمان

بر اساس مستندات رسمی [`ettercap(8)`](https://linux.die.net/man/8/ettercap)، سوئیچ‌های اصلی رابط کاربری، انتخاب حمله MitM و مدیریت پلاگین‌ها در جدول زیر آورده شده‌اند:

| فلگ (کوتاه / بلند) | دسته عملکردی | توضیح دقیق عملکرد و کاربرد فنی | مرجع رسمی |
| :--- | :--- | :--- | :--- |
| `-T` / `--text` | رابط کاربری | اجرا در محیط متنی (Text-only Console)؛ با فشردن کلید `h` در حین اجرا راهنمای تعاملی نمایش داده می‌شود. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-C` / `--curses` | رابط کاربری | اجرا در محیط شبه‌گرافیکی ترمینال مبتنی بر Ncurses ([`ettercap_curses(8)`](https://manpages.ubuntu.com/manpages/focal/man8/ettercap_curses.8.html)). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-G` / `--gtk` | رابط کاربری | اجرا با رابط گرافیکی GTK2 (در صورت نصب بسته گرافیکی). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-q` / `--quiet` | نمایش خروجی | حالت سکوت (Quiet Mode) در محیط متنی؛ عدم چاپ محتوای خام تک‌تک بسته‌ها در ترمینال و نمایش تنها رویدادهای مهم و اطلاعات احراز هویت. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-Q` / `--superquiet` | نمایش خروجی | حالت سکوت کامل؛ ذخیره اطلاعات جمع‌آوری‌شده در پروفایل‌ها بدون چاپ نام‌های کاربری و رمزها در صفحه. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-i` / `--iface <iface>` | تنظیمات شبکه | مشخص کردن کارت شبکه مورد استفاده برای شنود و تزریق بسته (مانند `enp0s1` یا `eth0`). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-M` / `--mitm <method:args>` | موتور حمله MitM | فعال‌سازی ماژول حمله مرد میانی شامل `arp`، `icmp`، `dhcp` و `port` همراه با پارامترهای مربوطه. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-M arp:remote` | موتور حمله MitM | اجرای مسموم‌سازی دوطرفه ARP همراه با پارامتر `remote` جهت رهگیری ترافیک خارج از شبکه محلی که از گیت‌وی عبور می‌کند. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-M arp:oneway` | موتور حمله MitM | مسموم کردن یک‌طرفه (تنها مسموم کردن جدول ARP کلاینت در `TARGET1` بدون ارسال بسته جعلی به روتر). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-P` / `--plugin <plugin>` | مدیریت پلاگین | اجرای پلاگین مشخص‌شده (مانند `dns_spoof`، `chk_poison` یا `list` برای مشاهده فهرست تمام پلاگین‌ها). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-F` / `--filter <file>` | فیلترینگ محتوا | بارگذاری فایل فیلتر باینری کامپایل‌شده با ابزار [`etterfilter(8)`](https://manpages.ubuntu.com/manpages/focal/man8/etterfilter.8.html) برای تغییر یا جایگزینی Payload بسته‌ها. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-z` / `--silent` | اسکن اولیه | غیرفعال کردن اسکن اولیه ARP کل شبکه هنگام راه‌اندازی برنامه. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-w` / `--write <file>` | ذخیره‌سازی | ذخیره تمام بسته‌های رهگیری‌شده در یک فایل با فرمت استاندارد `pcap` برای تحلیل در Wireshark. | [ettercap(8)](https://linux.die.net/man/8/ettercap) |
| `-L` / `--log <logfile>` | گزارش‌گیری | ثبت کامل بسته‌ها (`.ecp`) و اطلاعات استخراج‌شده میزبان‌ها (`.eci`) قابل خواندن با [`etterlog(8)`](https://manpages.ubuntu.com/manpages/jammy/man8/etterlog.8.html). | [ettercap(8)](https://linux.die.net/man/8/ettercap) |

## سناریوهای کاربردی و تحلیل نمونه دستورات

### ۱. حمله مرد میانی همراه با جعل DNS (نمونه دستور تصویر)

پس از ثبت رکوردهای دامنه مورد نظر در فایل `/etc/ettercap/etter.dns`، دستور زیر ترافیک بین کلاینت (`192.168.1.16`) و گیت‌وی (`192.168.1.2`) را روی کارت شبکه `enp0s1` رهگیری کرده و درخواست‌های DNS را با پلاگین [`dns_spoof`](https://manpages.ubuntu.com/manpages/focal/man8/ettercap_plugins.8.html) پاسخ می‌دهد ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)):

```bash
sudo ettercap -T -q -i enp0s1 -M arp:remote /192.168.1.16// /192.168.1.2// -P dns_spoof
```

* **`-T -q`**: اجرای برنامه در محیط متنی (`-T`) و در حالت سکوت (`-q`) تا ترمینال با محتوای خام بسته‌ها پر نشود ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* **`-i enp0s1`**: انتخاب کارت شبکه `enp0s1` ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* **`-M arp:remote`**: فعال‌سازی مسموم‌سازی جدول ARP با قابلیت رهگیری ارتباطات اینترنتی عبوری از گیت‌وی (`remote`) ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* **`/192.168.1.16//` و `/192.168.1.2//`**: تعیین دو هدف ارتباطی (کلاینت و گیت‌وی) ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* **`-P dns_spoof`**: فعال‌سازی پلاگین جعل DNS برای هدایت دامنه‌های تعریف‌شده در `etter.dns` به آدرس IP مشخص‌شده ([Ubuntu Manpage: ettercap_plugins(8)](https://manpages.ubuntu.com/manpages/focal/man8/ettercap_plugins.8.html)).

### ۲. مشاهده فهرست تمام پلاگین‌های نصب‌شده

برای دیدن فهرست کامل پلاگین‌های قابل استفاده در سیستم خود ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)):

```bash
ettercap -P list
```

## کاربردهای عملی در تست نفوذ و تحلیل امنیت شبکه

* **ارزیابی آسیب‌پذیری در برابر DNS Spoofing**: بررسی تاثیر جعل پاسخ‌های DNS محلی و بررسی عملکرد مکانیزم‌های امنیتی مانند DNSSEC و HSTS ([Ubuntu Manpage: ettercap_plugins(8)](https://manpages.ubuntu.com/manpages/focal/man8/ettercap_plugins.8.html)).
* **تشخیص سیستم‌عامل غیرفعال (Passive OS Fingerprinting)**: شناسایی سیستم‌عامل‌ها، سرویس‌های فعال، پورت‌های باز و سازنده کارت شبکه میزبان‌های LAN تنها با شنود غیرفعال بسته‌ها بدون ارسال هیچ بسته اسکن ([Linux Manpage: ettercap(8)](https://linux.die.net/man/8/ettercap)).
* **تست تزریق و ویرایش بسته در لحظه**: استفاده از اسکریپت‌های کامپایل‌شده [`etterfilter`](https://manpages.ubuntu.com/manpages/focal/man8/etterfilter.8.html) برای بررسی یکپارچگی داده‌ها در پروتکل‌های فاقد رمزنگاری یا امضای دیجیتال.

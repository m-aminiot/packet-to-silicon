# راهنمای جامع ابزار hping3 (ساخت و ارسال بسته‌های سفارشی TCP/IP)

## معرفی ابزار و قابلیت‌های کلیدی در مهندسی بسته

ابزار [`hping3`](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) (طراحی‌شده توسط Salvatore Sanfilippo) یک مولد و تحلیل‌گر بسته خط فرمان برای پروتکل‌های TCP/IP است که فراتر از دستور استاندارد `ping` عمل می‌کند ([Linux Manpage: hping3(8)](https://linux.die.net/man/8/hping3)). در حالی که `ping` معمولی تنها بسته‌های ICMP Echo Request ارسال می‌کند، [`hping3`](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) امکان ساخت دستی بسته‌های TCP، UDP، ICMP و RAW-IP را با کنترل کامل بر تمام فیلدهای هدر، فلگ‌های کنترلی، تکهتکه کردن بسته‌ها (Fragmentation) و جعل آدرس مبدا (IP Spoofing) فراهم می‌سازد ([Kali Linux Tools: hping3](https://www.kali.org/tools/hping3/)).

## نصب ابزار در لینوکس

برای نصب [`hping3`](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) در توزیع‌های مبتنی بر دبیان، اوبونتو و کالی لینوکس از دستور زیر استفاده کنید:

```bash
sudo apt install hping3
```

## جدول کامل فلگ‌ها و سوئیچ‌های پرکاربرد

بر اساس مستندات رسمی [`hping3(8)`](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)، مهم‌ترین سوئیچ‌های انتخاب پروتکل، تنظیم فلگ‌های TCP، دستکاری آدرس IP و زمان‌بندی ارسال در جدول زیر دسته‌بندی شده‌اند:

| فلگ (کوتاه / بلند) | دسته عملکردی | توضیح دقیق عملکرد و کاربرد فنی | مرجع رسمی |
| :--- | :--- | :--- | :--- |
| `-S` / `--syn` | فلگ TCP | فعال کردن فلگ `SYN` در هدر TCP برای شروع دست‌تکان سه مرحله‌ای یا اسکن نیمه‌باز. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-A` / `--ack` | فلگ TCP | فعال کردن فلگ `ACK` برای بررسی قوانین فایروال (State Table) و تشخیص فیلترینگ پورت. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-F` / `-R` / `-P` / `-U` | فلگ TCP | تنظیم به ترتیب فلگ‌های `FIN`، `RST`، `PUSH` و `URG` در هدر بسته TCP. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-X` / `-Y` | فلگ TCP | تنظیم فلگ‌های ترکیبی Xmas (`-X`) و Ymas (`-Y`) برای بررسی واکنش پشتهٔ شبکه. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-a` / `--spoof <host>` | هدر IP | جعل آدرس IP مبدا (IP Spoofing)؛ پاسخ‌های سرور هدف به آدرس جعل‌شده ارسال می‌شوند. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `--rand-source` | هدر IP | تولید و درج آدرس‌های IP مبدا تصادفی برای هر بسته ارسالی جهت تست جداول وضعیت فایروال. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-p` / `--destport <port>` | پورت TCP/UDP | تعیین پورت مقصد؛ با افزودن `+` یا `++` قبل از شماره پورت، شماره پورت با هر بسته افزایش می‌یابد. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-s` / `--baseport <port>` | پورت TCP/UDP | تعیین پورت مبدا پایه (که به صورت پیش‌فرض با ارسال هر بسته یکی افزایش می‌یابد؛ با `-k` ثابت می‌ماند). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-c` / `--count <count>` | کنترل ارسال | توقف برنامه پس از ارسال و دریافت تعداد مشخصی بسته (`count`). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-i` / `--interval <wait>` | زمان‌بندی | تعیین فاصله زمانی بین بسته‌ها به ثانیه (مثلاً `-i 2`) یا میکروثانیه (مثلاً `-i u1000`). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `--fast` / `--faster` | زمان‌بندی | ارسال سریع بسته‌ها (`--fast` معادل ۱۰ بسته در ثانیه و `--faster` معادل `-i u1`). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `--flood` | زمان‌بندی | ارسال بسته‌ها با حداکثر سرعت ممکن بدون انتظار یا نمایش پاسخ‌های دریافتی (تست فشار). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-0` / `-1` / `-2` | انتخاب پروتکل | تغییر حالت پیش‌فرض (TCP) به ترتیب به حالت RAW-IP (`-0`)، ICMP (`-1`) و UDP (`-2`). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-8` / `--scan <ports>` | حالت اسکن | اسکن بازه‌ای از پورت‌ها (مانند `--scan 1-1000,8080` یا `--scan known`). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-d` / `--data <size>` | بدنه بسته | تعیین اندازه Payload (بدنه داده بسته) بر حسب بایت. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-f` / `--frag` | هدر IP | تکهتکه کردن (Fragment) بسته‌ها برای ارزیابی عملکرد فایروال و سیستم‌های تشخیص نفوذ (IDS). | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |
| `-T` / `--traceroute` | مسیریابی | اجرای Traceroute مبتنی بر پروتکل انتخابی (TCP/UDP/ICMP) با افزایش تدریجی فیلد `TTL`. | [hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html) |

## سناریوهای کاربردی و تحلیل نمونه دستورات

### ۱. ارسال بسته TCP SYN با جعل آدرس مبدا (نمونه دستور تصویر)

این دستور ۵ بستهٔ TCP با فلگ `SYN` به پورت ۸۰ میزبان مجاز `192.168.1.2` می‌فرستد و آدرس مبدأ را به `192.168.1.100` تغییر می‌دهد. در این حالت پاسخ‌ها به آدرس مبدأ اعلام‌شده فرستاده می‌شوند و لزوماً به فرستندهٔ واقعی برنمی‌گردند ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)):

```bash
sudo hping3 -S 192.168.1.2 -a 192.168.1.100 -p 80 -c 5
```

* **`-S`**: فعال‌سازی فلگ `SYN` در هدر TCP ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).
* **`192.168.1.2`**: آدرس IP میزبان مقصد ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).
* **`-a 192.168.1.100`**: جعل آدرس مبدا (Spoofed Source IP) به طوری که سرور مقصد پاسخ `SYN-ACK` یا `RST` را به `192.168.1.100` می‌فرستد ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).
* **`-p 80`**: هدف قرار دادن پورت ۸۰ (سرویس وب HTTP) ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).
* **`-c 5`**: توقف عملیات پس از ارسال ۵ بسته ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).

### ۲. اسکن پورت‌های شناخته شده با فلگ SYN

برای اسکن سریع پورت‌های ۱ تا ۱۰۲۴ روی سرور هدف و مشاهده مستقیم فلگ‌های پاسخ (`flags=SA` به معنای پورت باز و `flags=RA` به معنای پورت بسته) ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)):

```bash
sudo hping3 --scan 1-1024 -S 192.168.1.2
```

### ۳. اجرای Traceroute روی پورت TCP مشخص (عبور از فیلتر ICMP)

زمانی که فایروال مقصد بسته‌های ICMP را مسدود می‌کند، می‌توان مسیر شبکه را با ارسال بسته‌های TCP SYN به پورت ۴۴۳ ردیابی کرد ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)):

```bash
sudo hping3 -S -p 443 -T 192.168.1.2
```

## کاربردهای عملی در تست نفوذ و ممیزی فایروال

* **ممیزی قوانین فایروال (Firewall Rule Testing)**: بررسی واکنش فایروال‌های Stateful و Stateless در برابر بسته‌های دارای فلگ‌های `ACK`، `FIN` یا بسته‌های تکهتکه‌شده (`-f`) ([Linux Manpage: hping3(8)](https://linux.die.net/man/8/hping3)).
* **تحلیل پیش‌بینی‌پذیری شماره توالی TCP**: استفاده از فلگ `-Q` (`--seqnum`) برای بررسی تصادفی بودن اعداد Sequence تولیدشده توسط سیستم‌عامل هدف ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).
* **کشف حداکثر واحد انتقال مسیر (Path MTU Discovery)**: استفاده از فلگ `-y` (`--dontfrag`) همراه با اندازه‌های مختلف داده (`-d`) برای یافتن دقیق MTU شبکه ([Ubuntu Manpage: hping3(8)](https://manpages.ubuntu.com/manpages/jammy/man8/hping3.8.html)).

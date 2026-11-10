# راهنمای جامع ابزار netcat / nc (چاقوی سوئیسی شبکه و ارتباطات سوکتی)

## محدودهٔ استفاده

نمونه‌ها را روی میزبان‌های خودتان یا سامانه‌هایی اجرا کنید که اجازهٔ آزمونشان را دارید. شنونده و انتقال سادهٔ netcat رمزنگاری و احراز هویت ندارند؛ اطلاعات حساس را با آن منتقل نکنید.

## معرفی ابزار و جایگاه آن در مهندسی شبکه

ابزار [`netcat`](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) (که با دستور کوتاه [`nc`](https://linux.die.net/man/1/nc) فراخوانی می‌شود) یک ابزار خط فرمان یونیکس و لینوکس برای خواندن و نوشتن مستقیم داده‌ها در اتصالات شبکه بر بستر پروتکل‌های TCP و UDP است ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)). به دلیل انعطاف‌پذیری بالا در ایجاد تقریباً هر نوع اتصال کلاینت یا سرور، در ادبیات شبکه به آن لقب «چاقوی سوئیسی TCP/IP» داده شده است ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).

این ابزار می‌تواند به‌عنوان کلاینت متصل شود، در حالت شنونده منتظر اتصال بماند، اتصال و تبادل داده را عیب‌یابی کند و در شبکهٔ مورداعتماد فایل منتقل کند ([Linux Manpage: nc(1)](https://linux.die.net/man/1/nc)).

## نسخه‌های مختلف و نحوه نصب در لینوکس

در توزیع‌های مدرن لینوکس سه پیاده‌سازی اصلی از این ابزار وجود دارد:
1. **نسخه سنتی ([`netcat-traditional`](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html))**: دارای سوئیچ‌های اجرای دستور `-e` و `-c` پس از برقراری اتصال.
2. **نسخه OpenBSD ([`netcat-openbsd`](https://manpages.ubuntu.com/manpages/jammy/man1/nc_openbsd.1.html))**: نسخه پیش‌فرض در بسیاری از توزیع‌های اوبونتو با پشتیبانی از سوکت‌های دامنه یونیکس و IPv6 (فاقد فلگ `-e` به دلایل امنیتی).
3. **نسخه پروژه Nmap ([`ncat`](https://man7.org/linux/man-pages/man1/ncat.1.html))**: نسخه پیشرفته همراه با پشتیبانی از رمزنگاری SSL/TLS و کنترل دسترسی.

نصب نسخه سنتی در اوبونتو و دبیان:

```bash
sudo apt install netcat-traditional
```

## جدول کامل فلگ‌ها و سوئیچ‌های خط فرمان

بر اساس مستندات رسمی [`nc.traditional(1)`](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) و [`nc(1)`](https://linux.die.net/man/1/nc)، مهم‌ترین فلگ‌های این ابزار در جدول زیر گردآوری شده‌اند:

| فلگ / سوئیچ | شکل کامل و سینتکس | توضیح دقیق عملکرد و کاربرد فنی | مرجع رسمی |
| :--- | :--- | :--- | :--- |
| `-l` | `-l` | فعال‌سازی حالت گوش دادن (Listen Mode) برای پذیرش اتصالات ورودی به عنوان سرور. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-p` | `-p <port>` | تعیین شماره پورت محلی (در حالت سرور در `netcat-traditional` به صورت `nc -lvp 4444` استفاده می‌شود). | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-z` | `-z` | حالت Zero-I/O؛ بررسی باز بودن پورت و قطع فوری اتصال بدون ارسال هیچ داده‌ای (ویژه اسکن پورت). | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-v` | `-v` / `-vv` | نمایش جزئیات اتصال (Verbose Mode)؛ استفاده دوبار (`-vv`) اطلاعات آماری بیشتری چاپ می‌کند. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-n` | `-n` | عدم انجام پرس‌وجوی DNS و کار صرفاً با آدرس‌های عددی IP برای افزایش سرعت. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-u` | `-u` | استفاده از پروتکل UDP به جای پروتکل پیش‌فرض TCP. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-w` | `-w <secs>` | تعیین مهلت زمانی (Timeout) بر حسب ثانیه برای برقراری اتصال یا پایان خواندن داده از شبکه. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-e` | `-e <filename>` | اجرای یک فایل اجرایی (مانند `/bin/bash`) پس از برقراری اتصال و اتصال ورودی/خروجی آن به سوکت شبکه (پشتیبانی به نسخه و پیاده‌سازی وابسته است و معمولاً در OpenBSD netcat وجود ندارد). اجرای فرمان از راه دور خطرناک است؛ این گزینه را در مثال‌های عمومی استفاده نکنید. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-c` | `-c <string>` | اجرای یک رشته دستور شل از طریق `/bin/sh -c` پس از برقراری اتصال (در `netcat-traditional`). | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-q` | `-q <secs>` | انتظار به مدت مشخص‌شده (بر حسب ثانیه) پس از دریافت پایان فایل (`EOF`) در ورودی استاندارد و سپس بستن اتصال. | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |
| `-s` | `-s <addr>` | تعیین آدرس IP مبدا محلی برای ارسال بسته‌ها (در سیستم‌های دارای چند آدرس IP). | [nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html) |

## سناریوهای کاربردی و تحلیل نمونه دستورات

### ۱. تست باز بودن یک پورت خاص یا بازه پورت‌ها (نمونه اول تصویر)

برای بررسی سریع اینکه آیا پورت ۴۴۵ (سرویس SMB) روی میزبان `192.168.1.1` باز است یا خیر، بدون ارسال داده اضافی ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)):

```bash
nc -zv 192.168.1.1 445
```

* **`-z`**: فعال کردن حالت اسکن بدون تبادل داده (Zero-I/O) ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).
* **`-v`**: نمایش پیام وضعیت اتصال (موفقیت یا رد شدن اتصال) در ترمینال ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).
* **`192.168.1.1 445`**: آدرس IP هدف و شماره پورت مقصد (می‌توان بازه‌ای مانند `20-80` نیز وارد کرد) ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).

### ۲. گوش دادن روی یک پورت به عنوان سرور / شنونده (نمونه دوم تصویر)

برای باز کردن پورت `4444` روی سیستم خودتان و انتظار برای اتصال کلاینت یا دریافت دادهٔ آزمایشی ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)):

```bash
nc -lvp 4444
```

* **`-l`**: قرار دادن برنامه در حالت Listen (سرور) ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).
* **`-v`**: نمایش اعلان به محض متصل شدن یک کلاینت به پورت ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).
* **`-p 4444`**: مشخص کردن شماره پورت محلی `4444` برای گوش دادن ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).

### ۳. انتقال مستقیم فایل بین دو سیستم در شبکه

در سیستم گیرنده (گوش دادن روی پورت ۹۰۰۰ و ذخیره خروجی در فایل) ([Linux Manpage: nc(1)](https://linux.die.net/man/1/nc)):

```bash
nc -lvp 9000 > backup.tar.gz
```

در سیستم فرستنده (ارسال فایل به آدرس سیستم گیرنده) ([Linux Manpage: nc(1)](https://linux.die.net/man/1/nc)):

```bash
nc -w 3 192.168.1.10 9000 < backup.tar.gz
```

## کاربردهای عملی در عیب‌یابی شبکه و تست نفوذ

* **دریافت بنر سرویس‌ها (Banner Grabbing)**: اتصال مستقیم به پورت‌های باز (مانند پورت ۸۰، ۲۱ یا ۲۵) برای مشاهده پیام خوش‌آمدگویی و نسخه دقیق سرور ([Linux Manpage: nc(1)](https://linux.die.net/man/1/nc)).
* **عیب‌یابی اتصال و فایروال**: راه‌اندازی یک شنونده موقت با `nc -lvp` روی سرور مقصد و تست اتصال از کلاینت برای اطمینان از باز بودن مسیر شبکه و قوانین فایروال ([Ubuntu Manpage: nc.traditional(1)](https://manpages.ubuntu.com/manpages/xenial/man1/nc.traditional.1.html)).
* **محدودیت امنیتی**: شنوندهٔ `nc` رمزنگاری یا احراز هویت ارائه نمی‌کند؛ آن را فقط موقتاً در شبکهٔ آزمایشگاهی اجرا کنید و پورت را پس از پایان کار ببندید.

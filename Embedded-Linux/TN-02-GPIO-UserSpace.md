# Technical Notes #2 — GPIO pada Linux

---

## 1. Pendahuluan

**GPIO (General Purpose Input/Output)** adalah pin digital yang dapat dikendalikan atau dibaca oleh software. Di embedded Linux, GPIO bukan sekadar "pin yang dapat di-`1` atau `0`". Alurnya adalah:

![](/home/arif/Documents/markdown/Technical-Notes/Embedded-Linux/resource/GPIObasic-1.png)

Linux memberikan kontrol GPIO dari userspace melalui **GPIO character device**, dengan device seperti

```text
/dev/gpiochip0
/dev/gpiochip1
...
```

---

## 2. GPIO pada Linux

### 2.1 GPIO Chip VS Line

Pada Linux modern, GPIO controller direpresentasikan sebagai **GPIO chip**.

Contoh:

```text
/dev/gpiochip0
/dev/gpiochip1
/dev/gpiochip2
```

Satu GPIO chip dapat memiliki banyak **GPIO lines**, yang mana biasanya merepresentasikan sebuah pin dari Chip atau peripheral GPIO tersebut.

```text
gpiochip0
│
├── line 0
├── line 1
├── line 2
├── ...
└── line N
```

Line diidentifikasi dengan **offset** di dalam chip tersebut.

```text
/dev/gpiochip0
        │
        └── offset 17
```

| Istilah   | Arti                                         |
| --------- | -------------------------------------------- |
| GPIO chip | Satu controller yang mengelola sejumlah GPIO |
| GPIO line | Satu pin/logical GPIO yang dimiliki chip     |
| Offset    | Nomor line di dalam sebuah chip              |
| Line name | Nama yang diberikan untuk line               |
| Consumer  | Software/device yang sedang menggunakan line |

Contoh:

```text
gpiochip2
├── offset 0  → "GPIO_A"
├── offset 1  → "GPIO_B"
├── offset 2  → "LED_STATUS"
└── offset 3  → "BUTTON"
```

---

### 2.2 Konfigurasi GPIO Line

Sebuah line dapat memiliki konfigurasi berupa

- direction

- active-low

- bias

- drive

- edge detection

- debounce

- output value

- event clock

Direction:

- INPUT

- OUTPUT

Bias:

- PULL_UP

- PULL_DOWN

- DISABLED

Drive:

- PUSH_PULL

- OPEN_DRAIN

- OPEN_SOURCE

---

## 3. Tools libgpiod: Tes GPIO Tanpa Menulis Program

`libgpiod` menyediakan tools berikut untuk membantu mengecek GPIO:

- gpiodetect

- gpioinfo

- gpioget

- gpioset

- gpiomon

- gpionotify

### 3.1 Mendeteksi GPIO Chip `gpiodetect`

```text
gpiochip0 [pinctrl-controller] (32 lines)
gpiochip1 [gpio-expander] (16 lines)
```

`gpiodetect` menampilkan chip, label, dan jumlah line.

---

### 3.2 Melihat Detail Line `gpioinfo`

```text
line  5: "LED_STATUS"  output
line 17: "USER_BUTTON" input active-low
```

Informasi yang berguna antara lain:

- offset

- name

- direction

- consumer

- active-low

- bias

- drive

- edge detection

- debounce

---

### 3.3 Membaca GPIO `gpioget`

```bash
gpioget --chip gpiochip0 17
```

Tool akan melakukan request terhadap line dan membaca nilai logisnya.

---

### 3.4 Menulis GPIO `gpioset`

```bash
gpioset --chip gpiochip0 17=1
```

atau:

```bash
gpioset --chip gpiochip0 17=0
```

`gpioset` mempertahankan line dalam request selama process tersebut masih berjalan. Karena itu, **jangan mengasumsikan nilai output akan tetap sama setelah `gpioset` keluar**.

---

### 3.5 Monitor Edge `gpiomon`

```bash
gpiomon --chip gpiochip0 17
```

Rising:

```bash
gpiomon --rising-edge --chip gpiochip0 17
```

Falling:

```bash
gpiomon --falling-edge --chip gpiochip0 17
```

Both:

```bash
gpiomon --edges=both --chip gpiochip0 17
```

`gpiomon` menggunakan edge-event mechanism GPIO character device. Event membawa informasi seperti tipe edge dan timestamp.

---

## 4. Python

Python cocok digunakan untuk prototype dan eksperimen karena tidak membutuhkan banyak boilerplate. Ada dua library yang umum digunakan, yang mana masing-masing menawarkan level abstraction yang berbeda:

```text
High-level:
    GPIO Zero

Linux GPIO API:
    gpiod
```

---

### 4.1 GPIO Zero

GPIO Zero menyediakan abstraction berbasis device.

```python
from gpiozero import LED
from time import sleep

led = LED(17)

while True:
    led.on()
    sleep(1)

    led.off()
    sleep(1)
```

Kelebihannya:

```text
API sederhana
device abstraction
event callbacks
cepat untuk prototype
```

GPIO Zero menggunakan **pin factory**, sehingga backend yang digunakan dapat berbeda. Dokumentasi saat ini menunjukkan dukungan `lgpio`, `RPi.GPIO`, `pigpio`, dan native implementation; prioritas backend dapat bergantung pada environment.

Gunakan GPIO Zero terutama ketika tujuan utamanya adalah **membangun aplikasi sederhana dengan cepat**, bukan mempelajari interface GPIO Linux secara langsung.

---

### 4.2. Python dengan libgpiod

```bash
python -m pip install gpiod
```

API Python libgpiod 2.x menggunakan konsep:

```text
LineSettings
LineRequest
Value
Direction
Edge
Bias
Drive
```

---

#### 4.2.1 Output

```python
import time
import gpiod
from gpiod.line import Direction, Value

LINE = 17

with gpiod.request_lines(
    "/dev/gpiochip0",
    consumer="gpio-example",
    config={
        LINE: gpiod.LineSettings(
            direction=Direction.OUTPUT,
            output_value=Value.INACTIVE,
        )
    },
) as request:

    while True:
        request.set_value(LINE, Value.ACTIVE)
        time.sleep(1)

        request.set_value(LINE, Value.INACTIVE)
        time.sleep(1)
```

`request_lines()` melakukan request terhadap line secara eksklusif, lalu object request digunakan untuk operasi selanjutnya. Line dapat diidentifikasi dengan offset ataupun name.

---

#### 4.2.2 Input

```python
import time
import gpiod
from gpiod.line import Direction

LINE = 17

with gpiod.request_lines(
    "/dev/gpiochip0",
    consumer="button-reader",
    config={
        LINE: gpiod.LineSettings(
            direction=Direction.INPUT,
        )
    },
) as request:

    while True:
        print(request.get_value(LINE))
        time.sleep(0.1)
```

Object `LineRequest` menyediakan `get_value()` dan `set_value()` untuk line yang sedang dimiliki oleh request.

---

#### 4.2.3 Input dengan Edge Event

```python
import gpiod
from gpiod.line import Direction, Edge

LINE = 17

with gpiod.request_lines(
    "/dev/gpiochip0",
    consumer="button-monitor",
    config={
        LINE: gpiod.LineSettings(
            direction=Direction.INPUT,
            edge_detection=Edge.BOTH,
        )
    },
) as request:

    while True:
        if request.wait_edge_events():
            for event in request.read_edge_events():
                print(
                    event.event_type,
                    event.timestamp_ns,
                )
```

`wait_edge_events()` menunggu event, sedangkan `read_edge_events()` mengambil event dari buffer. Event menyediakan tipe edge, offset line, timestamp, dan sequence number.

---

#### 4.2.4 Konfigurasi Input

Contoh button dengan pull-up dan debounce:

```python
from datetime import timedelta

import gpiod
from gpiod.line import Direction, Edge, Bias

settings = gpiod.LineSettings(
    direction=Direction.INPUT,
    edge_detection=Edge.BOTH,
    bias=Bias.PULL_UP,
    debounce_period=timedelta(milliseconds=20),
)
```

Konfigurasi semacam ini hanya akan berhasil sejauh kernel dan GPIO controller mendukung fitur terkait.

---

# 5. Daftar Library GPIO Python

Selain GPIO Zero dan `gpiod`, beberapa library lain relevan:

| Library          | Abstraction   | Karakteristik                                 |
| ---------------- | ------------- | --------------------------------------------- |
| `gpiod`          | Rendah        | Binding resmi libgpiod                        |
| GPIO Zero        | Tinggi        | Device-oriented, mudah digunakan              |
| `lgpio`          | Rendah        | Berkomunikasi dengan `/dev/gpiochip*`         |
| python-periphery | Rendah–sedang | GPIO dan peripheral Linux lainnya             |
| `RPi.GPIO`       | Rendah        | Raspberry Pi-oriented                         |
| `pigpio`         | Tinggi        | Raspberry Pi dengan daemon-based architecture |

`lgpio` sendiri berkomunikasi dengan Linux `gpiochip` devices dan bukan secara fundamental API khusus Raspberry Pi.

Untuk **generic embedded Linux**, `gpiod` adalah pilihan yang paling langsung terhadap model GPIO Linux modern.

---

# 6. C dengan libgpiod

Header:

```c
#include <gpiod.h>
```

Alur penggunaannya:

1. gpiod_chip

2. gpiod_line_settings

3. gpiod_line_config

4. gpiod_request_config

5. gpiod_line_request

## 6.1 Contoh Sederhana (Output blinky)

```c
#include <gpiod.h>
#include <stdio.h>
#include <unistd.h>

int main(void)
{
    const char *chip_path = "/dev/gpiochip0";
    unsigned int offset = 17;

    struct gpiod_chip *chip = NULL;
    struct gpiod_line_settings *settings = NULL;
    struct gpiod_line_config *line_config = NULL;
    struct gpiod_request_config *req_config = NULL;
    struct gpiod_line_request *request = NULL;

    chip = gpiod_chip_open(chip_path);
    if (!chip)
        return 1;

    settings = gpiod_line_settings_new();
    line_config = gpiod_line_config_new();
    req_config = gpiod_request_config_new();

    if (!settings || !line_config || !req_config)
        return 1;

    gpiod_request_config_set_consumer(
        req_config,
        "gpio-example"
    );

    gpiod_line_settings_set_direction(
        settings,
        GPIOD_LINE_DIRECTION_OUTPUT
    );

    gpiod_line_settings_set_output_value(
        settings,
        GPIOD_LINE_VALUE_INACTIVE
    );

    if (gpiod_line_config_add_line_settings(
            line_config,
            &offset,
            1,
            settings) < 0)
        return 1;

    request = gpiod_chip_request_lines(
        chip,
        req_config,
        line_config
    );

    if (!request)
        return 1;

    while (1) {
        gpiod_line_request_set_value(
            request,
            offset,
            GPIOD_LINE_VALUE_ACTIVE
        );

        sleep(1);

        gpiod_line_request_set_value(
            request,
            offset,
            GPIOD_LINE_VALUE_INACTIVE
        );

        sleep(1);
    }

    gpiod_line_request_release(request);
    gpiod_request_config_free(req_config);
    gpiod_line_config_free(line_config);
    gpiod_line_settings_free(settings);
    gpiod_chip_close(chip);

    return 0;
}
```

---

## 6.2 Compiling

```bash
gcc gpio.c -o gpio \
    $(pkg-config --cflags --libs libgpiod)
```

Periksa library:

```bash
pkg-config --cflags libgpiod
pkg-config --libs libgpiod
```

---

## 6.3 Input

Set direction:

```c
gpiod_line_settings_set_direction(
    settings,
    GPIOD_LINE_DIRECTION_INPUT
);
```

Kemudian membaca:

```c
enum gpiod_line_value value;

value = gpiod_line_request_get_value(
    request,
    offset
);
```

Nilai yang digunakan libgpiod bersifat logis:

```text
GPIOD_LINE_VALUE_ACTIVE
GPIOD_LINE_VALUE_INACTIVE
```

sehingga active-low dapat ditangani di konfigurasi line.

---

# 12. Polling vs Event

Untuk input, terdapat dua pola utama.

### Polling

```text
read
 ↓
check
 ↓
sleep
 ↓
read
 ↓
...
```

Kelebihan:

```text
sederhana
```

Kekurangan:

```text
CPU overhead
latency bergantung interval
event singkat dapat terlewat
```

### Event-driven

```text
wait
 ↓
GPIO edge
 ↓
kernel event
 ↓
userspace terbangun
 ↓
handle event
```

Untuk signal yang memang berbentuk event, pendekatan event-driven biasanya lebih tepat.

libgpiod menyediakan API khusus untuk menunggu dan membaca edge event.

---

# 13. GPIO Event Bukan Hard Real-Time

Walaupun GPIO edge dapat berasal dari hardware interrupt, application userspace **tidak berjalan langsung di interrupt context**.

Alurnya:

```text
GPIO edge
    ↓
Hardware IRQ
    ↓
Kernel
    ↓
GPIO driver
    ↓
Event buffer
    ↓
Scheduler
    ↓
Userspace process
```

Akibatnya terdapat:

```text
scheduling latency
context switch
interrupt latency
system load
```

Jadi userspace GPIO cocok untuk event handling biasa, tetapi tidak boleh dianggap sebagai hard real-time interrupt handler.

Untuk timing yang ketat gunakan mekanisme yang memang dirancang untuk itu:

```text
hardware timer
PWM
DMA
dedicated peripheral
kernel driver
real-time mechanism
```

---

# 14. Kapan GPIO Seharusnya Tidak Dipakai Langsung?

Tidak semua hardware yang dikontrol melalui GPIO seharusnya dibuatkan aplikasi GPIO sendiri.

Linux sudah mempunyai subsystem untuk banyak fungsi umum.

Contoh:

```text
GPIO + LED
     ↓
LED subsystem

GPIO + button
     ↓
Input / gpio-keys

GPIO + PWM requirement
     ↓
PWM subsystem
```

Kernel documentation secara eksplisit menyarankan menggunakan driver/subsystem yang sudah tersedia jika hardware memang memiliki subsystem yang sesuai, daripada mengendalikan hardware langsung dari userspace.

Contoh lain:

```text
I2C device → I2C subsystem
SPI device → SPI subsystem
UART       → TTY/UART subsystem
PWM        → PWM subsystem
1-Wire     → 1-Wire subsystem
```

Karena itu prinsipnya:

> **Gunakan GPIO userspace untuk kebutuhan GPIO yang memang membutuhkan kontrol dari userspace; gunakan subsystem/driver Linux ketika fungsi hardware tersebut sudah mempunyai interface kernel yang tepat.**

---

# 15. Device Tree dan GPIO

Pada embedded system, Device Tree dapat mendeskripsikan hubungan antara device dan GPIO.

Secara sederhana:

```text
Device Tree
     │
     ▼
device uses GPIO X
     │
     ▼
Kernel
     │
     ▼
Driver / GPIO subsystem
```

Contoh:

```dts
leds {
    compatible = "gpio-leds";

    status_led {
        gpios = <&gpio 17 0>;
    };
};
```

Jika device digunakan melalui driver kernel, application biasanya tidak perlu mengetahui GPIO line yang sebenarnya.

Ini berbeda dengan program userspace yang secara langsung membuka:

```text
/dev/gpiochipN
```

Jadi Device Tree membantu **menghubungkan hardware description dengan driver**, sedangkan libgpiod memberikan userspace interface untuk kasus yang memang dikelola dari userspace. Linux GPIO documentation menyediakan mapping GPIO melalui Device Tree dan ACPI.

---

# 16. Legacy GPIO Sysfs

Linux sebelumnya menyediakan interface:

```text
/sys/class/gpio/
```

Contoh lama:

```bash
echo 17 > /sys/class/gpio/export
echo out > /sys/class/gpio/gpio17/direction
echo 1 > /sys/class/gpio/gpio17/value
```

Interface tersebut sekarang **obsolete/deprecated**.

Dokumentasi kernel menyatakan bahwa GPIO sysfs API telah digantikan oleh GPIO character-device API, dan development baru seharusnya menggunakan API character device.

```text
Legacy
/sys/class/gpio/

Modern
/dev/gpiochipN
libgpiod
```

Sysfs tetap perlu dipahami ketika menangani sistem lama, tetapi bukan pilihan untuk project baru.

---

# 17. Debugging GPIO

Ketika GPIO tidak bekerja, ikuti urutan berikut.

### 1. Hardware

Periksa:

```text
wiring
GND
voltage level
current
resistor
pull-up/down
```

### 2. Pinmux

Pastikan pin benar-benar berada pada mode GPIO.

### 3. GPIO chip

```bash
gpiodetect
```

### 4. Line

```bash
gpioinfo
```

Periksa:

```text
offset
name
direction
consumer
active-low
bias
drive
```

### 5. Uji langsung

```bash
gpioget ...
gpioset ...
gpiomon ...
```

### 6. Kernel

```bash
dmesg | grep -i gpio
dmesg | grep -i pinctrl
```

atau:

```bash
journalctl -k
```

### 7. Permission

```bash
ls -l /dev/gpiochip*
id
```

Jika `/dev/gpiochipN` tidak dapat diakses, masalah dapat berada pada permission device, group, udev rule, atau environment service/container.

---

# 18. Best Practices

## Hardware

```text
• Selalu gunakan datasheet sebagai sumber utama.
• Jangan mengasumsikan voltage level.
• Jangan melebihi current rating.
• Hindari floating input.
• Gunakan pull-up/down yang sesuai.
• Gunakan resistor pembatas arus untuk LED.
• Perhatikan active-low.
```

## Linux

```text
• Gunakan GPIO character device untuk development baru.
• Gunakan libgpiod untuk API GPIO Linux.
• Gunakan line name jika tersedia.
• Jangan mengandalkan nomor GPIO yang ambigu.
• Periksa pinmux dan Device Tree.
• Perhatikan ownership/consumer.
• Gunakan subsystem kernel jika fungsi hardware sudah didukung.
```

## Software

```text
• Request line secara eksplisit.
• Lepaskan request dengan benar.
• Gunakan event untuk event-driven input.
• Jangan mengandalkan userspace GPIO untuk hard real-time.
• Pisahkan GPIO abstraction dari business logic.
• Handle error dan device disappearance.
• Jangan mengandalkan state output setelah process release.
```

---

# 19. GPIO Application Architecture

Untuk aplikasi yang lebih besar, jangan menyebarkan detail libgpiod ke seluruh program.

```text
Application
    │
    ▼
GPIO Abstraction
    │
    ├── LED
    ├── Button
    ├── Reset
    └── Enable
    │
    ▼
libgpiod
    │
    ▼
/dev/gpiochipN
```

Contoh interface aplikasi:

```c
gpio_init();
led_set(true);
button_wait();
gpio_cleanup();
```

Detail seperti:

```text
gpiochip
offset
line settings
consumer
active-low
```

tetap berada di GPIO layer.

Dengan demikian, perubahan hardware tidak harus mengubah seluruh application.

---

# 20. Workflow Praktis

Urutan kerja yang dapat digunakan pada hampir semua board:

```text
Hardware schematic
        ↓
Datasheet
        ↓
Pinmux / Device Tree
        ↓
gpiodetect
        ↓
gpioinfo
        ↓
gpioget / gpioset
        ↓
gpiomon
        ↓
Python
        ↓
Python + gpiod
        ↓
C + libgpiod
        ↓
C++ + libgpiodcxx
        ↓
Kernel driver / subsystem
```

Prinsipnya:

> **Pastikan Linux mengenali GPIO dengan benar sebelum debugging application.**

---

# 21. Cheat Sheet

## Inspect

```bash
gpiodetect
gpioinfo
```

## Read

```bash
gpioget --chip gpiochip0 <line>
```

## Write

```bash
gpioset --chip gpiochip0 <line>=1
gpioset --chip gpiochip0 <line>=0
```

## Monitor

```bash
gpiomon --chip gpiochip0 <line>
gpiomon --rising-edge --chip gpiochip0 <line>
gpiomon --falling-edge --chip gpiochip0 <line>
```

## Kernel

```bash
dmesg | grep -i gpio
dmesg | grep -i pinctrl
journalctl -k
```

## Device

```bash
ls -l /dev/gpiochip*
```

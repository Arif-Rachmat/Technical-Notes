# NOTE 0: Konsep Dasar dan Peristilahan Rangkaian Elektronika & Desain PCB

## **1. Alur Desain PCB**
Proses desain PCB terdiri dari empat tahapan utama:

![Diagram Workflow PCB](/Elektro/Resources/PCBWorkflow.jpg)


* **Circuit Idea & Schematic**: Menggambar diagram skematik untuk menentukan komponen elektronik dan hubungan jalur kelistrikannya secara logis.
* **Linked Board Layout**: Menghubungkan data skematik ke editor PCB dan menata posisi bentuk fisik (*footprint*) komponen di atas papan.
* **Trace Routing & Rule Check**: Membuat jalur koneksi tembaga (*trace*) antar-pin komponen, lalu menjalankan *Design Rule Check* (DRC) untuk memastikan tidak ada eror kelistrikan atau pelanggaran ukuran.
* **Gerber & Drill Files for Fabrication**: Mengekspor file akhir manufaktur berupa format Gerber (`.GBR`) untuk lapisan papan dan file Drill (`.DRL`) untuk pemetaan titik bor.

## **2. Symbol vs Footprint**

![Symbol vs footprint vs real](/Elektro/Resources/schematic-symbol-to-pcb-footprint.jpg)

### Symbol (Simbol Skematik)

Representasi grafis komponen dalam skematik/diagram rangkaian, yang menunjukkan **koneksi elektrik logis** — bukan bentuk fisiknya.

- Digunakan di software schematic capture (KiCad Eeschema, Altium Schematic Editor, Eagle, dll.)
- Menampilkan: jumlah pin, nama/nomor pin, fungsi pin, dan reference designator (R1, C1, U1, dst.)
- Tujuannya: memastikan hubungan antar komponen dalam rangkaian benar secara elektris
- Contoh: simbol resistor (kotak/zigzag), simbol IC (kotak dengan pin berlabel sesuai datasheet)

### Footprint (Land Pattern / PCB Footprint)

Representasi fisik komponen berupa pola pad tembaga di PCB — tempat kaki komponen akan disolder.

- Digunakan di software PCB layout
- Menentukan: ukuran pad, jarak antar pin (pitch), bentuk pad, ukuran lubang bor (untuk THT), courtyard, dan outline silkscreen
- Harus sesuai persis dengan dimensi fisik komponen sesuai datasheet (package/case, misal SOIC-8, 0805, TO-220)
- Tujuannya: memastikan komponen fisik benar-benar pas dan bisa disolder dengan baik di PCB

<div align='center'>

## Perbedaan

| Aspek | Symbol | Footprint |
|:---:|:---:|:---:|
| Fungsi | Representasi logis/listrik | Representasi fisik jejak kaki pin |
| Dipakai di | Schematic diagram | PCB layout |
| Fokus | Konektivitas pin | Dimensi & posisi pad komponen |
| Bentuk visual | Simbolik (tidak realistis) | Sesuai bentuk nyata komponen |

</div>

Satu komponen (part) dalam library biasanya memiliki **satu symbol** tapi bisa punya **beberapa pilihan footprint** (misalnya resistor bisa versi THT atau SMD 0603/0805, atau satu model op-amp yang sama bisa DIP-8 atau SOP-8).

## **3. Karakeristik Komponen Elektronik**
### **3.1 Jenis Mounting (Pemasangan)**

![SMD vs THT Visualization](/Elektro/Resources/SMDvsTHT.jpg)

#### A. Through-Hole Technology (THT)
Metode di mana kawat/pin komponen dimasukkan ke dalam lubang bor yang telah dilapisi tembaga (*Plated Through-Hole*) pada PCB, kemudian disolder dari sisi berlawanan.

| | |
|---|---|
| **Ciri fisik** | Punya kaki panjang (lead), badan komponen relatif besar |
| **Contoh komponen** | Resistor axial, capacitor elektrolit besar, konektor, IC DIP, transformer |
| **Kekuatan mekanik** | ✅ Sangat kuat, tahan tarikan/getaran (lead menembus & terkunci di PCB) |
| **Kerapatan PCB** | ❌ Boros ruang, butuh lubang bor per pin |
| **Proses solder** | Wave soldering / solder manual |
| **Biaya produksi** | Lebih mahal (perlu proses drilling) |
| **Cocok untuk** | Prototyping, komponen daya tinggi, konektor, edukasi/hobi |

#### B. Surface-Mount Technology/Device (SMT/SMD)
Metode di mana pin/terminal komponen diletakkan langsung di atas permukaan tembaga (*Pad*) tanpa melubangi substrat PCB.

| | |
|---|---|
| **Ciri fisik** | Kaki pendek/tanpa kaki, badan komponen kecil |
| **Contoh komponen** | Resistor/capacitor chip (0402, 0603, 0805), IC SOIC/QFN/BGA |
| **Kekuatan mekanik** | ⚠️ Lebih rentan lepas dari getaran/stress dibanding THT |
| **Kerapatan PCB** | ✅ Sangat hemat ruang, mendukung desain kompak & double-sided |
| **Proses solder** | Reflow soldering (pakai solder paste + stencil) |
| **Biaya produksi** | Lebih murah untuk produksi massal (bisa full otomatis/pick & place) |
| **Cocok untuk** | Produk elektronik modern, perangkat portabel, mass production |

### **3.2 Packaging**

**Packaging** merupakan wadah fisik yang membungkus *die*/*chip* semikonduktor (atau elemen komponen pasif) dan menyediakan jalur koneksi listrik ke dunia luar (PCB). 

Package adalah rumah fisik yang membungkus *die* semikonduktor dan menghubungkan koneksi listriknya ke PCB — *die* itu sendiri berukuran mikroskopis, dan package-lah yang membuatnya bisa ditangani, disolder, dan terlindungi dari stres mekanik, kelembapan, dan panas.

![Packaging Visualization](/Elektro/Resources/packaging.jpg)

Beberapa fungsi utama dari packaging:
- **Proteksi fisik:** Melindungi die dari debu, kelembapan, benturan
- **Interkoneksi listrik:** Menghubungkan pad die (bonding wire/bump) ke pin/pad eksternal
- **Manajemen panas:** Menyalurkan panas dari die ke lingkungan/PCB (thermal dissipation)
- **Kemudahan handling:** Membuat komponen bisa dipegang, disolder, dan diproses otomatis (pick & place)
- **Identifikasi:** Marking: part number, logo, pin-1 indicator

#### **Klasifikasi Packaging**

##### 1. Berdasarkan Metode Mounting
- **Through-Hole (PTH/THT)** — kaki masuk lubang PCB
- **Surface-Mount (SMT/SMD)** — dipasang di permukaan pad PCB

##### 2. Berdasarkan Bentuk Kaki/Terminal
![Perbandingan packaging berdasarkan bentuk kaki](/Elektro/Resources/klasifikasiBentuk.png)
| Tipe | Ciri | Contoh |
|---|---|---|
| **Leaded (gull-wing)** | Kaki melengkung keluar, terlihat jelas | SOIC, QFP, SOT |
| **Leadless (no-lead)** | Pad datar di bawah badan, tanpa kaki menonjol | QFN, DFN |
| **Ball array** | Bola solder di bawah badan | BGA, CSP |
| **J-lead** | Kaki melengkung ke bawah badan (bentuk J) | SOJ, PLCC |

##### 3. Berdasarkan Material Badan
- **Plastic** — paling umum, murah (mayoritas produk konsumer)
- **Ceramic** — tahan panas ekstrem & radiasi, untuk aerospace/militer (CDIP, CBGA)

#### **Jenis Packaging Standar / Umum**
![List of IC Packages](/Elektro/Resources/List-of-IC-Package-Types.png)
##### ---Through-Hole---
| Package | Deskripsi |
|---|---|
| **DIP** (Dual In-line Package) | Dua baris pin sejajar keluar tegak lurus dari badan plastik persegi, pitch 2.54 mm, cocok untuk breadboard |
| **TO-92** | Package transistor murah untuk hobi, daya rendah |
| **TO-220 / TO-247** | Package daya untuk regulator tegangan (LM317, L7805), bisa dipasang heatsink |
| **SIP** (Single In-line) | Satu baris pin (mis. modul resistor array) |

##### ---Surface-Mount — Small Outline Family---
| Package | Deskripsi |
|---|---|
| **SOIC/SOP** | Versi SMD dari DIP, luas 30–50% lebih kecil dan tinggi 70% lebih rendah, pitch 1.27mm, hand-solderable |
| **TSOP** | Lebih tipis dari SOP, pitch 1.27 mm |
| **SSOP/TSSOP** | Pitch 0.65 mm, lebih kompak |

##### ---Surface-Mount — Discrete/Transistor---
| Package | Deskripsi |
|---|---|
| **SOT-23** | 3 pin, pitch 0.95 mm, badan 2.9×1.3 mm — sangat kompak, dipakai untuk op-amp kecil & IC supervisor |
| **SOT-223** | 4 pin, badan lebih besar dengan tab exposed untuk disipasi termal LDO hingga ~1W |
| **SOD** | Package dioda dua terminal (small outline diode) |
| **D2PAK/TO-263** | Versi SMD dari TO-220, mampu disipasi daya lebih tinggi, pin dibengkokkan agar rata di permukaan PCB |

##### ---Surface-Mount — Quad/High Pin Count---
| Package | Deskripsi |
|---|---|
| **QFP/LQFP/TQFP** | Pin di 4 sisi, pitch 0.4–1.0 mm, hingga 300+ pin |
| **QFN** | Leadless, pad exposed di bawah untuk termal bagus |
| **DFN** | Mirip QFN, pin count lebih sedikit, lebih kecil dari SOIC |

##### ---Ball/Chip-Scale---
| Package | Deskripsi |
|---|---|
| **BGA** | Bola solder di bawah badan, kepadatan pin tertinggi |
| **CSP** (Chip Scale Package) | Ukuran hampir sama dengan die itu sendiri |
| **WLCSP** | CSP tanpa substrat tambahan, langsung di wafer — paling tipis |

##### ---Advanced / Multi-Die---
| Package | Deskripsi |
|---|---|
| **SiP** (System-in-Package) | Menggabungkan beberapa die, komponen pasif, kadang package lain dalam satu modul — umum di modul wireless & IoT |
| **MCM** (Multi-Chip Module) | Beberapa die telanjang di satu substrat, terhubung wire-bond atau flip-chip |

#### **Pemilihan Package**

| Faktor | Pertimbangan |
|---|---|
| **Kemudahan solder** | THT/SOIC/SOT mudah manual; QFN/BGA butuh reflow + peralatan khusus |
| **Kerapatan PCB** | BGA/QFN paling hemat ruang; DIP paling boros |
| **Disipasi panas** | QFN dengan pad termal mencapai 15–40°C/W, BGA 10–30°C/W — jauh lebih baik dari DIP (50–100°C/W) atau SOIC standar (80–120°C/W) |
| **Rework/repair** | THT & SOIC mudah dicabut/diganti; QFN & BGA sulit (butuh hot air/reballing) |
| **Kekuatan mekanik** | THT lebih tahan guncangan/getaran |
| **Volume produksi** | SMD (terutama QFN/BGA) lebih efisien untuk mass production otomatis |

### **3.3 Konsep Pitch**

**Pitch** merupakan jarak dari titik pusat satu pin/pad ke titik pusat pin/pad terdekat berikutnya, diukur dalam **mm** atau **inch**. Parameter ini menentukan kecocokan fisik antara komponen dan footprint di PCB.

![Contoh Pitch SOT-8](/Elektro/Resources/SOTpitch-.png)

#### Jenis Pitch

| Jenis | Konteks | Keterangan |
|---|---|---|
| **Pin pitch** | Konektor, header, IC | Jarak antar kaki komponen |
| **Pad pitch** | Footprint PCB | Jarak antar pad tembaga (harus match dengan pin pitch komponen) |
| **Lead pitch** | IC (SOIC, QFP, dll.) | Jarak antar kaki IC — biasanya sangat kecil pada IC modern |
| **Trace/track pitch** | Routing PCB | Jarak antar jalur tembaga (terkait design rules) |
| **BGA ball pitch** | IC BGA | Jarak antar bola solder di bawah chip |

#### Standar Pitch Umum

| Komponen | Pitch Umum |
|---|---|
| Header/pin THT standar | **2.54 mm** (0.1 inch) — paling umum di breadboard/Arduino |
| Header pitch kecil | 2.0 mm, 1.27 mm (0.05 inch) |
| Connector JST-XH | 2.5 mm |
| Connector JST-PH | 2.0 mm |
| IC SOIC | 1.27 mm |
| IC TSSOP | 0.65 mm |
| IC QFP | 0.5 mm / 0.65 mm / 0.8 mm |
| IC QFN | 0.4 mm / 0.5 mm |
| BGA (umum) | 0.5 mm – 1.0 mm (bisa lebih kecil di chip modern) |

## **4. Elemen Konektivitas Skematik**
1. **Net:** Jalur konduksi logis yang menghubungkan pin-pin komponen secara elektrik.
2. **Local Label:** Nama yang diberikan pada Net. Semua jalur yang diberi nama label sama dalam satu sheet akan terhubung secara otomatis tanpa perlu menarik garis kawat (*wire*).
3. **Global Label:** Label yang berlaku lintas sheet pada skematik hirarki berhalaman banyak.
4. **Power Port (`VCC, GND, +5V, +3V3`):** Symbol khusus merepresentasikan jalur daya. Di KiCad, Power Port berfungsi sebagai Global Label tersembunyi.
5. **Bus:** Kumpulan jalur net paralel yang dikelompokkan menjadi satu garis tebal untuk menyederhanakan tampilan skematik (misal: `DATA[0..7]`, `ADDR[0..15]`).
6. **Bus Entry:** Garis miring kecil penghubung antara net individual ke dalam sebuah Bus.

## **5. Lapisan PCB**
```
+-------------------------------------------------------------------+
| [ F.SilkS ] Teks Nama Komponen, Outline, Logo                     |
+-------------------------------------------------------------------+
| [ F.Mask  ] Menutupi Tembaga, Mengisolasi Area Non-Pad            |
+-------------------------------------------------------------------+
| [ F.Cu    ] Traces, Pads, dan Ground Plane Sisi Atas              |
+-------------------------------------------------------------------+
|                   SUBSTRAT DIELEKTRIK (FR-4)                      |
+-------------------------------------------------------------------+
| [ B.Cu    ] Traces, Pads, dan Ground Plane Sisi Bawah             |
+-------------------------------------------------------------------+
| [ B.Mask  ] Menutupi Tembaga Sisi Bawah                           |
+-------------------------------------------------------------------+
| [ B.SilkS ] Teks dan Identifikasi Sisi Bawah                      |
+-------------------------------------------------------------------+
```
| Nama Lapisan | Kode Singkat | Fungsi Utama |
| :---: | :---: | :--- |
| **Front Copper** | `F.Cu` | Lapisan pola tembaga konduktif bagian depan/atas (Sinyal, Power, Ground). |
| **Back Copper** | `B.Cu` | Lapisan pola tembaga konduktif bagian belakang/bawah. |
| **Front Solder Mask** | `F.Mask` | Lapisan cat pelindung epoksi (biasanya hijau, biru, atau hitam) yang menutupi tembaga atas *kecuali* area Pad solder agar tembaga tidak teroksidasi dan tidak konslet saat disolder. |
| **Back Solder Mask** | `B.Mask` | Lapisan pelindung tembaga untuk sisi bawah PCB. |
| **Front Silkscreen** | `F.SilkS` | Cetakan tinta sablon (biasanya putih) untuk *Designator* (R1, C1, U1), polaritas, garis tepi komponen, dan teks instruksi. |
| **Back Silkscreen** | `B.SilkS` | Cetakan tinta sablon bagian bawah. |
| **Solder Paste** | `F.Paste` / `B.Paste` | Layer referensi untuk pembuatan lembaran *Stencil* baja saat pengaplikasian pasta solder SMT otomatis. |
| **Edge Cuts** | `Edge.Cuts` | Garis batas pemotongan mekanis yang menentukan bentuk fisik luar papan PCB. |


## 6. Elemen Geometri Fisik PCB: Pad, Via, Trace, dan Zone

```
             STRUKTUR ELEMEN GEOMETRI PCB (TAMPAKAN ATAS & SAMPING)

     [ TOP COPPER TRACE ]         [ PLATED VIA ]              [ THERMAL RELIEF PAD ]
         Width (W)                  Drill Size                    Spoke / Connection
         |<----->|                    |<-->|                         +--+    +--+
      ==============               +--+    +--+                      |  |    |  |
      |  Trace     |               |  |Via |  |                    +-+  +----+  +-+
      ==============               |  |Hole|  |                    |  COPPER ZONE |
                                   +--+    +--+                    +-+  +----+  +-+
                                  Pad Diameter                       |  |    |  |
                                  |<-------->|                       +--+    +--+

```

### 6.1. Jalur Tembaga (*Traces*)

* **Trace Width (Lebar Jalur):** Ditentukan berdasarkan kuat arus listrik yang mengalir (dihitung menggunakan standar **IPC-2221**).
* *Signal Lines:* $0.25\text{ mm} - 0.38\text{ mm}$ ($10\text{ mil} - 15\text{ mil}$).
* *Power Lines (1A–2A):* $1.0\text{ mm} - 1.5\text{ mm}$ ($40\text{ mil} - 60\text{ mil}$).


* **Clearance (Jarak Bebas):** Jarak minimal antar-konduktor yang berdekatan untuk mencegah terjadinya percikan listrik (*arcing*) atau *short circuit*.

---

### 6.2. Via (Laluan Antar Lapisan)

Via adalah lubang kecil yang dinding dalamnya dilapisi tembaga (*plated hole*) untuk menyambungkan koneksi listrik dari lapisan `F.Cu` ke `B.Cu`.

```
                         ANATOMI VIA THROUGH-HOLE

          Pad Diameter (Annular Ring)
        |<------------------------>|
        +-------+            +-------+
        | F.Cu  |            | F.Cu  |
   =====+       +============+       +===== Top Copper
        | Dinding Tembaga    |             
        | Pelapis (Plated)   |             Substrat FR-4
   =====+       +============+       +===== Bottom Copper
        | B.Cu  |            | B.Cu  |
        +-------+            +-------+
                |<---------->|
                  Drill Size

```

* **Standard Via:** Lubang bor tembus dari layer teratas ke terbawah.
* **Microvia / Blind / Buried Via:** Digunakan pada PCB multi-layer (*High Density Interconnect* / HDI).

---

### 6.3. Copper Zone / Copper Pour (Area Tembaga Luas)

* **Pengertian:** Mengisi area kosong PCB dengan tuangan tembaga luas yang dihubungkan ke Net tertentu (paling umum adalah **GND**).
* **Fungsi Utama:**
1. **Ground Plane:** Meredam interferensi gelombang elektromagnetik (EMI Shielding).
2. **Heat Sink:** Membantu membuang panas dari komponen daya tinggi (seperti MOSFET/Regulator).
3. **Impedansi Rendah:** Menyediakan jalur kembali arus (*return path*) yang sangat efisien.



#### Thermal Relief vs. Solid Connection

```
       THERMAL RELIEF CONNECTION                   SOLID CONNECTION
       (Mencegah Solder Heat Sink)                (Konduksi Arus Maksimal)

            +---+       +---+                          +-----------+
            |   |       |   |                          |           |
            |   +-------+   |                          |  COPPER   |
         +--+               +--+                       |   POUR    |
         |     +---------+     |                       |  (SOLID)  |
         |     |   PAD   |     |                       |  +-----+  |
         |     +---------+     |                       |  | PAD |  |
         +--+               +--+                       |  +-----+  |
            |   +-------+   |                          |           |
            |   |       |   |                          |           |
            +---+       +---+                          +-----------+

```

* **Thermal Relief:** Digunakan pada Pad komponen yang terhubung ke Ground Zone agar panas solder tidak cepat tersedot ke area tembaga raksasa saat soldering tangan (mencegah *cold joint*).
* **Solid Connection:** Digunakan jika Pad membutuhkan konduksi arus atau transfer panas maksimal (misal: pad pembuang panas MOSFET SMD).

---

## 7. Aturan Desain & Integritas: ERC dan DRC

Untuk memastikan desain bebas dari kesalahan logika maupun manufaktur, KiCad menyediakan dua modul verifikasi otomatis:

```
                            PROSES VERIFIKASI DESAIN

      [ SKEMATIK ] ───>  ERC (Electrical Rules Check)
                         ├── Memeriksa Pin Melayang (Unconnected)
                         ├── Memeriksa Konflik Output Power
                         └── Memeriksa Driver Tanpa Power Supply
                         
      [ PCB LAYOUT ] ───> DRC (Design Rules Check)
                         ├── Memeriksa Short Circuit (Tabrakan Net)
                         ├── Memeriksa Jarak Bebas Minimal (Clearance Violation)
                         └── Memeriksa Jalur Belum Terhubung (Unrouted Net)

```

1. **ERC (Electrical Rules Check):**
* Berada di modul **Eeschema**.
* Memeriksa kesalahan logika listrik. Contoh: Dua pin berkategori *Power Output* yang dihubungkan secara langsung, pin input yang dibiarkan menggantung tanpa koneksi (*floating pin*).


2. **DRC (Design Rules Check):**
* Berada di modul **Pcbnew**.
* Memeriksa kesesuaian fisik PCB dengan kemampuan pabrikasi (*Fab Limits*). Contoh: Jarak dua trace terlalu dekat ($<\text{Clearance Limit}$), ada net yang belum terhubung (*unrouted net*), atau lubang bor overlap.



---

## 8. Ringkasan Istilah & Glosarium Teknis PCB

| Istilah / Singkatan | Kepanjangan / Kategori | Deskripsi Singkat |
| --- | --- | --- |
| **PCB** | *Printed Circuit Board* | Papan fisik tempat menempel dan menghubungkan komponen elektronika. |
| **EDA** | *Electronic Design Automation* | Perangkat lunak komputer untuk merancang sirkuit elektronik (contoh: KiCad, Altium, EasyEDA). |
| **THT** | *Through-Hole Technology* | Metode mounting komponen dengan memasukkan kaki/pin ke dalam lubang PCB. |
| **SMT / SMD** | *Surface-Mount Technology / Device* | Metode mounting komponen langsung di atas permukaan pad tembaga PCB. |
| **Pitch** | Geometri Interkoneksi | Jarak pusat-ke-pusat antara dua pin/pad yang berdekatan. |
| **Pad** | PCB Geometry | Area tembaga terbuka tanpa solder mask tempat menyolder pin komponen. |
| **Via** | PCB Interconnect | Lubang berinti tembaga yang menghubungkan trace antar lapisan (layer) PCB. |
| **Solder Mask** | PCB Layer | Lapisan pelindung cat isolator penutup tembaga PCB (mencegah korosi & short circuit). |
| **Silkscreen** | PCB Layer | Tinta sablon informasi simbol, nama komponen (R1, C1), logo, dan garis tepi. |
| **Courtyard** | PCB Footprint Bounds | Area batas aman komponen pada PCB untuk mencegah tumbukan fisik antar komponen. |
| **RefDes** | *Reference Designator* | Kode penanda unik komponen pada skematik dan PCB (misal: R101, C202, U1). |
| **ERC / DRC** | Uji Integritas Desain | *Electrical Rules Check* (Uji Listrik Skematik) dan *Design Rules Check* (Uji Layout PCB). |
| **Gerber File** | Manufacturing Output | Format file standar industri berisi peta layer PCB untuk dibaca oleh mesin pabrikasi PCB. |
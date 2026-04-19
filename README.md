# Watch_TSC_Project

Un smartwatch construit de la zero — schematic, PCB 4 straturi, carcasă 3D — centrat pe nRF52840 și un display E-Paper care nu consumă energie când nu se schimbă nimic pe ecran.

---

## Ce este acest proiect

Scopul a fost să proiectez un ceas inteligent complet: de la primul simbol în schematic până la modelul 3D gata de imprimat. Nu un modul dev board montat într-o carcasă, ci un PCB custom cu toate componentele alese și integrate manual.

Tehnologia cheie este display-ul **E-Paper (e-ink)** — odată ce afișează ceva, menține imaginea fără niciun consum de curent. Combinat cu modurile deep sleep ale nRF52840 (1.5μA), rezultatul este un dispozitiv care poate rula zile întregi dintr-o celulă LiPo mică.

### Ce integrează placa

| Subsistem | Componentă | Rol |
|-----------|-----------|-----|
| Microcontroller | nRF52840 | Cortex-M4 64MHz, BT 5.0, USB nativ |
| Display | E-Paper via SPI | Afișaj zero-consum în standby |
| Mișcare | BMA423 | Accelerometru 3 axe, step counter, wake-on-wrist |
| Încărcare | BQ25180 | LiPo charger programabil I2C, 5mA–1A |
| Alimentare | RT6160 | Buck-Boost 3.3V stabil indiferent de tensiunea bateriei |
| Baterie | MAX17048 | Fuel gauge ModelGauge, fără shunt rezistor |
| Feedback | DRV2605 | Driver haptic, 123 efecte ERM/LRA |
| USB | KH-TYPE-C-16P + USBLC6 | Conector C16P cu protecție ESD ±15kV |
| Input | EVP-AKE31A ×3 | Butoane tactile SMD ultra-subțiri |
| RF | 2450AT18B100E | Antenă SMD 2.45GHz pentru Bluetooth |

---

## Diagrama bloc a sistemului

```
  USB-C (5V)
      │
      ▼
 ┌─────────┐    VBUS      ┌──────────────┐
 │  ESD    │─────────────▶│  BQ25180     │ LiPo Charger
 │USBLC6   │              │  (I2C 0x6A)  │◀─── Baterie LiPo
 └────┬────┘              └──────┬───────┘
      │ D+/D−                    │ VBAT
      │                          ▼
      │                   ┌──────────────┐
      │                   │  MAX17048    │ Fuel Gauge
      │                   │  (I2C 0x36)  │
      │                   └──────┬───────┘
      │                          │ VBAT
      │                          ▼
      │                   ┌──────────────┐
      │                   │   RT6160     │ Buck-Boost DC/DC
      │                   │  (I2C 0x62)  │─── 3V3 ───────────┐
      │                   └──────────────┘                    │
      │                                                       │
      └───────────────────────────────────────────────────────▼
                                                     ┌─────────────────┐
                    ┌── SPI ──▶ E-Paper Display       │                 │
                    │                                 │   nRF52840      │
                    ├── I2C ──▶ BMA423  (0x18)        │   Cortex-M4     │
                    │       ──▶ DRV2605 (0x5A)        │   64MHz / BT5.0 │
                    │       ──▶ BQ25180 (0x6A)        │                 │
                    │       ──▶ RT6160  (0x62)        └────────┬────────┘
                    │       ──▶ MAX17048(0x36)                 │
                    │                                          │ GPIO
                    ├── SWD ──▶ TC2030-IDC                     │
                    │                                          ▼
                    └── USB ──▶ (intern nRF52840)      Butoane ×3 / IMU INT / HAPTIC_EN
```

---

## Componente — BOM complet

| Ref | Part Number | Descriere | Qty |
|-----|------------|-----------|:---:|
| U1 | nRF52840-QIAA | MCU ARM Cortex-M4, BT 5.0, USB 2.0, 1MB Flash | 1 |
| IC1 | BQ25180YBGR | LiPo charger liniar 1A I2C, 8-DSBGA | 1 |
| IC2 | DRV2605YZFR | Haptic driver ERM/LRA I2C 123 efecte, 9-BGA | 1 |
| IC3 | BMA423 | Accelerometru MEMS triaxial 12-bit, LGA-12 | 1 |
| U3 | MAX17048G+T10 | Fuel gauge ModelGauge 1-cell, SOT-23-6 | 1 |
| IC9 | RT6160AWSC | Buck-Boost DC/DC 800mA I2C, 15-WL-CSP | 1 |
| D3 | USBLC6-2SC6Y | TVS ESD ±15kV bidirecțional USB, SOT-23-6 | 1 |
| J4 | KH-TYPE-C-16P | Conector USB-C 16 pini SMD | 1 |
| J1 | 503480-2400 | Conector FPC/FFC 0.5mm pas 24 circuite | 1 |
| J2 | TC2030-IDC | Tag-Connect SWD 6 pini footprint-only | 1 |
| Q2 | DMG2305UX-7 | P-MOSFET 20V/4.2A, SOT-23 | 1 |
| Q3 | SI1308EDL-T1-GE3 | N-MOSFET 30V/1.5A, SC-70 | 1 |
| D1,D2,D5 | MBR0530 | Schottky 30V/500mA, SOD-123 | 3 |
| SW_* | EVP-AKE31A | Buton tactil SMD ultra-subțire Panasonic | 3 |
| ANT1 | 2450AT18B100E | Antenă SMD 2.45GHz Johanson | 1 |
| L7 | FTC252012SR47MBCA | Inductor 0.47μH SMD 2016 | 1 |
| L5 | 744043680 | Inductor 68μH WE-TPC SMD | 1 |
| X1 | Crystal 32MHz | Oscilator SMD 32MHz 2016 | 1 |
| X2 | Crystal 32.768kHz | Oscilator RTC 32.768kHz 3215 SMD | 1 |
| Cx | — | Condensatoare SMD 0201/0402, 1pF–22μF | ~50 |
| Rx | — | Rezistoare SMD 0201, 2.2Ω–10kΩ | ~15 |
| Lx | — | Inductoare SMD 0402, 3.9nH–10μH | ~4 |
| TP_* | TP20R | Test pad SMD 2mm | 14 |
| SJ1 | — | Solder jumper SMD | 1 |

---

## Hardware în detaliu

### Cum circulă energia

Bateria LiPo (3.7–4.2V) alimentează regulatorul **RT6160** care produce un 3V3 stabil pentru toată placa. Când e conectat USB-C, VBUS (5V) merge direct la **BQ25180** care preia încărcarea bateriei independent de restul sistemului. nRF52840 monitorizează ambele stări: știe când USB e prezent (CHARGER_PG) și câtă baterie a mai rămas (MAX17048 via I2C).

Componentele cu consum ridicat — display-ul și haptic driver-ul — sunt controlate prin MOSFET-uri (Q2/Q3), astfel că nRF52840 le poate tăia complet alimentarea când nu sunt necesare.

### Magistrala I2C — 5 periferice pe aceleași fire

| Componentă | Adresă I2C | Ce face |
|------------|:----------:|---------|
| BQ25180 | 0x6A | Controlează curentul și tensiunea de încărcare |
| RT6160 | 0x62 | Ajustează tensiunea de ieșire dacă e necesar |
| BMA423 | 0x18 | Livrează date accelerometru, numără pași |
| MAX17048 | 0x36 | Raportează SOC% și tensiunea celulei |
| DRV2605 | 0x5A | Selectează și redă efecte tactile |

Clock pe **P0.06**, date pe **P0.07**. Rezistoare de pull-up pe ambele linii.

### SPI — display E-Paper

Comunicație write-only spre display (nu există linie MISO — displayul nu trimite date înapoi). Frecvență maximă 4MHz. Pe lângă SCK și MOSI, display-ul are nevoie de 3 semnale GPIO suplimentare: DC (Data/Command), RST (reset hardware) și BUSY (ceasul trebuie să aștepte când displayul procesează un refresh).

### SWD — programare și debug

Conectorul **TC2030-IDC** (Tag-Connect, fără pini fizici pe PCB) expune SWDIO, SWDCLK, SWO și RESET. Nu ocupă spațiu vertical în ansamblu — cablul de programare se atașează temporar prin spring pins direct pe pad-urile de pe PCB.

### Bluetooth și antena

Antena **2450AT18B100E** este o antenă SMD ceramică pentru 2.4GHz, plasată în colțul plăcii cu un cutout în substrat sub ea. Cutout-ul elimină FR4-ul (material dielelectric cu pierderi la 2.4GHz) din zona de radiație, îmbunătățind câștigul și SWR-ul antenei. Cristalul de 32MHz asigură referința de frecvență pentru radio.

### Cristalul de 32.768kHz — RTC în sleep

Al doilea oscilator, la 32.768kHz, alimentează timer-ul RTC al nRF52840 și rămâne activ chiar și în deep sleep. Permite ceasului să știe ora exactă și să se trezească la intervale precise fără să țină procesorul pornit.

### BMA423 — mai mult decât un accelerometru

BMA423 are un procesor intern dedicat care rulează algoritmii de step counting și wake-on-wrist fără să implice nRF52840. Când detectează mișcarea caracteristică ridicării mâinii, trimite o întrerupere pe **P1.08 (IMU_INT1)** care trezește MCU-ul din deep sleep în microsecunde. Consum în această stare: **0.9μA**.

### Estimare consum și autonomie

| Scenariu | Curent estimat |
|----------|:-------------:|
| nRF52840 TX Bluetooth activ | 6.26 mA |
| nRF52840 RX Bluetooth activ | 4.82 mA |
| Refresh display E-Paper | ~26 mA (câteva secunde) |
| BMA423 monitorizare continuă | 130 μA |
| MAX17048 fuel gauge activ | 23 μA |
| DRV2605 vibrație activă | 5 mA |
| nRF52840 deep sleep + RTC + BMA low-power | ~50–100 μA |

**Consum mediu în utilizare normală** (BT conectat periodic, display refresh la fiecare minut, IMU activ): **6–8 mA** la 3.3V.

Cu o baterie LiPo de **300mAh**: autonomie estimată **35–50 ore**.

---

## Alocarea pinilor nRF52840

### Display E-Paper (SPI)

| Pin MCU | Semnal | Direcție | Descriere |
|---------|--------|:--------:|-----------|
| P0.02 / AIN0 | SCK | OUT | Clock SPI pentru display |
| P0.03 / AIN1 | MOSI | OUT | Date seriale spre display |
| P0.05 / AIN3 | EPD_CS | OUT | Chip select, activ LOW |
| P0.15 | EPD_RST | OUT | Reset hardware display, activ LOW |
| P0.16 | EPD_DC | OUT | LOW = comandă, HIGH = date |
| P0.17 | EPD_BUSY | IN | HIGH când displayul procesează |

### Periferice I2C

| Pin MCU | Semnal | Descriere |
|---------|--------|-----------|
| P0.06 | SCL | Clock comun pentru toate perifericele I2C |
| P0.07 | SDA | Date bidirecționale I2C |
| P0.08 | IMU_INT2 | Întrerupere secundară BMA423 (step counter) |
| P1.08 | IMU_INT1 | Întrerupere primară BMA423 (wake-on-wrist) |
| P0.10 / NFC2 | FUELGAUGE_ALT | Alarmă MAX17048 la baterie scăzută |
| P0.11 | CHARGER_PG | Power Good BQ25180 — 1 când USB e valid |
| P0.12 | HAPTIC_EN | Enable DRV2605, activ HIGH |

### Butoane și control

| Pin MCU | Semnal | Descriere |
|---------|--------|-----------|
| P0.13 | BTN_UP | Buton sus (navigare meniu) |
| P0.14 | BTN_ENT | Buton enter / confirmare |
| P1.02 | BTN_DN | Buton jos (navigare meniu) |

### USB, SWD și ceasuri

| Pin MCU | Semnal | Interfață | Descriere |
|---------|--------|:---------:|-----------|
| D+ | USB_DP | USB | Linie diferențială pozitivă, prin ESD |
| D− | USB_DM | USB | Linie diferențială negativă, prin ESD |
| VBUS | 5V_USB | PWR | Detecție USB prezent → BQ25180 |
| SWDIO | SWDIO | SWD | Date programare/debug |
| SWDCLK | SWDCLK | SWD | Clock programare/debug |
| SWO | SWO | SWD | Serial Wire Output (trace) |
| P0.18 / RESET | RESET | GPIO | Reset hardware MCU |
| ANT | RF | RF | Antenă Bluetooth 2.4GHz |
| XC1, XC2 | XTAL_32M | CLK | Cristal 32MHz — clock principal |
| XL1, XL2 | XTAL_32K | CLK | Cristal 32.768kHz — RTC sleep |

---

## PCB — decizii de proiectare

### 4 straturi: de ce și cum

| Strat | Conținut |
|-------|---------|
| **L1 — Top** | Toate componentele SMD + rutare semnale principale |
| **L2** | Plan de masă GND continuu |
| **L3** | Plan de alimentare 3V3 |
| **L4 — Bottom** | Rutare secundară + via connections |

Planul de masă pe L2 direct sub stratul de semnal L1 minimizează inductanța buclei de return pentru fiecare traseu. Fără acest plan, semnalele SPI și I2C ar genera mai mult EMI și ar fi mai sensibile la zgomot. Planul de 3V3 pe L3 reduce impedanța de alimentare, esențial pentru vârfurile de curent ale nRF52840 în TX.

### Via-in-pad

BQ25180, RT6160, DRV2605 și MAX17048 vin în packaging BGA/CSP cu pitch sub 0.5mm — imposibil de rutat cu via-uri laterale în spațiul disponibil. Soluția: via-uri plasate direct în centrul pad-urilor BGA. Acestea trebuie **umplute cu rășină epoxidică și planarize** (filled & capped) înainte de HASL/ENIG, altfel pasta de lipit curge în via și jointul devine defectuos.

### Cutout sub antenă

Substatul FR4 este un material cu pierderi dielectrice la 2.4GHz. Lăsat sub zona de radiație a antenei 2450AT18B100E, absoarbe o parte din energia RF și detune antena față de 50Ω. Soluția: un decupaj fizic al PCB-ului sub jumătatea neconectată a antenei, astfel că aceasta radiază în aer.

### 14 test pad-uri TP20R

Distribuite pe semnalele: 3V3, VBAT, GND, SDA, SCL, SWDCLK, SWDIO, SWO, RESET, VREG. Permit conectarea unui multimetru sau logicanalyzer după asamblare fără a fi nevoie de probe pe QFN/BGA.

---

## Ansamblu 3D

Modelat complet în **Autodesk Fusion 360**. Ansamblul verifică că totul se potrivește fizic înainte de fabricație.

Stratificarea de jos în sus:

```
  ┌─────────────────────────┐  ← Cadru superior + geam display
  │     Display E-Paper     │  ← Panou e-ink + cablu FPC
  │         PCB             │  ← Placa cu toate componentele
  │      Baterie LiPo       │  ← Celulă cu fire sudate pe test pads
  └─────────────────────────┘  ← Carcasă inferioară
```

Butoanele (×3) sunt accesibile prin locașuri în carcasa inferioară, pe laterale. Conectorul USB-C iese prin fanta frontală a carcasei. Shaker-ul haptic este montat cilindric în spațiul rămas lângă baterie.

---

## Galerie

### PCB — Layout 2D
![PCB 2D](Images/2D_PCB.png)

### PCB — Randare 3D
![PCB 3D](Images/3D_PCB.png)

### Ceas asamblat
![Ceas asamblat](Images/Ceas%203D%20asamblat.png)

### Exploded View
![Exploded View](Images/Exploded%20view.png)

### Baterie LiPo
![Baterie](Images/Baterie3D.png)

### Display E-Paper
![Display](Images/Display3D.png)

### Motor Haptic
![Shaker](Images/Shaker3D.png)

---

## Structura repository-ului

```
Watch_TSC_Project/
│
├── Hardware/
│   ├── PCB_2D.brd              # Fișier board KiCad/Eagle
│   ├── Schematic_final.sch     # Schematic complet
│   └── Schematic_final.pdf     # Export PDF schematic
│
├── Manufacturing/
│   ├── gerbers.zip             # Fișiere Gerber pentru fabricație PCB
│   ├── Watch_TSC_bom.bom       # Bill of Materials pentru comandă componente
│   ├── PnP_Watch_TSC_front.cpl # Pick & Place față
│   └── PnP_Watch_TSC_back.cpl  # Pick & Place spate
│
├── Mechanical/
│   ├── Exploded_Watch_3D.zip   # Export STEP ansamblu exploded
│   └── Watch_3D.f3z            # Proiect Autodesk Fusion 360
│
├── Images/                     # Randări și capturi PCB
│
├── LICENSE
└── README.md
```

---

MIT License — Copyright 2026 Stoenescu Constantin-Andrei

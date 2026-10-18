# Detekce překážek a vyhýbání se jim pomocí 3D dToF LiDAR senzoru na palubě bezpilotního letounu

**Autor:** Kryštof Kuča

**Vedoucí práce: Ing. Václav Riss**

Tento repozitář obsahuje veškerá hardwarová a softwarová data k bakalářské práci, která se zabývá integrací moderního 3D dToF (Direct Time-of-Flight) LiDAR senzoru VL53L9CX na palubu bezpilotního letounu (UAV) za účelem detekce překážek.

## 📂 Struktura repozitáře

Repozitář je logicky rozdělen do následujících sekcí:

```text
📁 Bakalarka/
├── 📁 lidar_pcb/                 # Kompletní hardwarový návrh plošného spoje (KiCad 8)
│   ├── 📁 libs/                  # Lokální knihovny součástek, symbolů a 3D modelů (SnapEDA)
│   ├── 📄 lidar_pcb.kicad_pro    # Hlavní konfigurační soubor KiCad projektu
│   ├── 📄 lidar_pcb.kicad_sch    # Zdrojový soubor schématu
│   ├── 📄 lidar_pcb.kicad_pcb    # Zdrojový soubor návrhu plošného spoje
│   └── 📄 lidar_pcb.pdf          # Rychlý náhled zapojení v PDF (nevyžaduje instalaci KiCadu)
├── 📁 software/                  # (Bude doplněno) Zdrojové kódy pro mikrokontrolér, I3C drivery
├── 📁 mechanics/                 # (Bude doplněno) 3D modely (STL/STEP) pro uchycení na rám dronu
├── 📁 docs/                      # (Bude doplněno) Datasheety, doplňující dokumentace a poznámky
├── 📄 .gitignore                 # Definice ignorovaných souborů pro Git (zálohy KiCadu atd.)
└── 📄 README.md                  # Tento soubor s popisem projektu
```

## ⚡ Návrh hardwaru (Custom PCB)

Jelikož senzor **VL53L9CX** vyžaduje specifické napájení a komunikační rozhraní, byl pro něj navržen dedikovaný plošný spoj (Custom PCB). Návrh je optimalizován pro nasazení na dronu – využívá výhradně SMD součástky pro minimalizaci hmotnosti a rozměrů a konektor JST-GH s mechanickou západkou odolnou proti vibracím.

### Hlavní komponenty a princip fungování

1. **Senzor (U1 - VL53L9CX):**
   * Hlavní měřicí prvek. Komunikuje přes vysokorychlostní sběrnici **I3C** (až 12.5 MHz).
   * Vyžaduje velmi přesný externí hodinový signál `AP_CLK` (až 27 MHz) a přesné řízení piny `XSHUT` (zapnutí), `SYNC_IN` (synchronizace snímků) a `INTR` (přerušení pro mastera).

2. **Napájení (Step-Down měniče U2, U3 - TLV62568P):**
   * Senzor se vyznačuje vysokými a velmi rychlými špičkovými odběry (až stovky mA) během záblesků VCSEL laseru.
   * Z palubních 5 V dronu jsou lokálně generovány dvě nezávislé napěťové úrovně:
     * **3.3 V:** Pro analogové obvody (`AVDD`) a napájení laseru (`VBAT_LDD`, `VBAT_RX`).
     * **1.2 V:** Pro digitální jádro senzoru (`DVDD`).
   * Použití vysoce účinných buck měničů místo lineárních LDO stabilizátorů bylo zvoleno pro minimalizaci generovaného tepla, což je pro přesnost dToF senzorů kritické.

3. **Převodníky logických úrovní (Level Shifters U4, U5):**
   * Piny senzoru pracují primárně na 1.2V logice (pin `IOVDD`), zatímco hlavní řídicí jednotka dronu obvykle využívá 3.3V logiku.
   * **TXB0102 (U5):** Zajišťuje obousměrný rychlý převod pro komunikační linky I3C (`SDA`, `SCL`).
   * **TXB0104 (U4):** Zajišťuje převod pro hardwarové řídicí signály a hodiny (`AP_CLK`, `XSHUT`, `SYNC_IN`, `INTR`).

### Klíčová pravidla návrhu (PCB Layout Rules)

Při návrhu plošného spoje byla striktně dodržována následující aplikační pravidla předepsaná výrobcem:

* **Sjednocená referenční zem (Solid GND Plane):** Deska využívá jedinou společnou GND rovinu (litou měď) na spodní vrstvě. Nedochází k žádnému řezání nebo izolování země pod měniči. To zajišťuje bezpečný odvod tepla a eliminuje vznik parazitních smyček (antén) pro 27MHz signály.
* **Termální management:** Senzor VL53L9CX generuje teplo (zejména např. v režimu "Gaming"). Středové pady (B4-B9, D4-D9) jsou připojeny přímo na velkou plochu GND pomocí sítě termálních prokovených otvorů (vias), které desku využívají jako chladič.
* **Striktní blokování napájení (Decoupling):** Kvůli 50% proudovým špičkám laseru jsou předepsané blokovací keramické SMD kondenzátory (10 µF, 4.7 µF, 1 µF s dielektrikem X5R/X7R) umístěny v řádu milimetrů přímo u napájecích pinů senzoru.
* **Minimalizace "Hot Loops":** Tlumivky (stíněné výkonové SMD verze) a 4.7 µF vstupní kondenzátory jsou umístěny bezprostředně u spínacích pinů měničů TLV62568P, aby se vysokofrekvenční spínací šum nešířil do citlivých částí LiDARu.

## 🛠 Jak otevřít projekt

1. Nainstalujte si [KiCad EDA](https://www.kicad.org/) (verze 8.0.x nebo novější).
2. Naklonujte si tento repozitář:
   ```bash
   git clone https://github.com/TheKrystof007/Bakalarka.git
   ```
3. Otevřete soubor `lidar_pcb/lidar_pcb.kicad_pro` v hlavní aplikaci KiCad.
4. Schéma najdete pod `lidar_pcb.kicad_sch` a fyzický návrh desky pod `lidar_pcb.kicad_pcb`. Lokální knihovny jsou relativně nalinkované, projekt by se měl otevřít bez chybějících závislostí.

## 🚀 Budoucí cíle (Roadmap)

**Fáze 1: Hardware a PCB**
- [x] Návrh elektrického schématu a kontrola pravidel (ERC)
- [x] Výběr vhodných SMD komponent a přiřazení pouzder (Footprints)
- [ ] Návrh plošného spoje (Routing & Layout)
  - [ ] *Návrh souvislé GND plochy pro odvod tepla a minimalizaci rušení*
  - [ ] *Optimalizace "hot loops" pro step-down měniče*
- [ ] Výroba PCB a osazení prototypu součástkami v přetavovací peci

**Fáze 2: Software a Komunikace (STM32)**
- [ ] Integrace I3C driveru na řídicí jednotce (STM32)
- [ ] Zajištění spolehlivého a rychlého posílání dat ze senzoru do mikrokontroléru
- [ ] Implementace různých profilů skenování (např. *Precision mode* vs. *Ambient mode*)
- [ ] Návrh a nasazení komprese dat (zajištění propustnosti pro budoucí zapojení více LiDARů současně)

**Fáze 3: Zpracování dat a Algoritmy**
- [ ] Získávání, filtrace a vizualizace 3D hloubkových map
- [ ] Otestování reálných vlastností LiDARu (vliv vnějšího osvětlení, teploty a dosah)
- [ ] Vývoj a implementace antikolizního algoritmu pro bezpilotní letoun
- [ ] Letové zkoušky a ladění systému v reálném prostředí
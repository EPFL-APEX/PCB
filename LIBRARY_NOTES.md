# Bibliothèque « apex » — contrôleur de chambre à brouillard (projet EPFL MAKE)

## Emplacements
- Symboles : `lib/apex.kicad_sym` — dans le repo, versionné git
- Empreintes : `lib/apex.pretty/` — dans le repo, versionné git
- Tables globales KiCad : `~/.config/kicad/10.0/sym-lib-table` et `fp-lib-table` — la lib `apex` y est enregistrée **en global** par chemin absolu vers le repo : visible dans tous les projets KiCad, source de vérité unique dans `lib/` (convention du README du repo).

Note : le repo contient également `lib/APEX_lib` (expérimentation D4184 + ébauche Pico), laissée intacte et volontairement non enregistrée dans les tables globales.

## Conventions
- Un symbole par composant dans `apex` : broches avec noms et numéros exacts, types électriques corrects, champs Reference / Value / Footprint / Datasheet / Description renseignés.
- Fonctions alternatives de broches : syntaxe KiCad 10 `(alternate "NOM" type_électrique style_graphique)` — visibles dans l'onglet « Alternate pin assignments » des propriétés du symbole.
- Empreinte standard KiCad quand elle existe, sinon création dans `apex.pretty`.
- GPIO RP2040 strictement 3,3 V, non tolérants au 5 V — aucune ligne de signal ne doit voir du 5 V.
- Section 230 V : creepage/clearance ≥ 2,5 mm, séparation nette secteur / logique.
- Sortie HV 3-5 kV : ne passe pas sur le PCB ; uniquement un bornier de départ de câble largement espacé.
- Validation de chaque fichier : `kicad-cli sym upgrade` / `fp upgrade` (rc 0 = S-expression valide) + export SVG de contrôle visuel.
- Générateur : `/tmp/gen_apex_lib.py` — régénère toute la lib (Pico conservé tel quel depuis le fichier existant, symboles 2-19 reconstruits, empreintes custom réécrites). Idempotent.

## Récapitulatif composants

| # | Composant | Symbole apex | Empreinte | Statut |
|---|---|---|---|---|
| 1 | Raspberry Pi Pico (variante) | `Raspberry_Pi_Pico` | `apex:RPi_Pico_SMD` (castellated) | créé, validé utilisateur |
| 2 | DS18B20 ×8 (bus 1-Wire) | `DS18B20` | `Package_TO_SOT_THT:TO-92` | créé, validé (parse + SVG) |
| 3 | BME280 (breakout I2C) | `BME280_BoB` | `Connector_PinHeader_2.54mm:PinHeader_1x06_P2.54mm_Vertical` | créé, validé ; ordre broches carte à vérifier |
| 4 | TFT 2,8" ILI9341 + tactile + SD (MSP2807) | `TFT_MSP2807` | `Connector_PinHeader_2.54mm:PinHeader_1x14_P2.54mm_Vertical` | créé, validé ; ordre broches module à vérifier |
| 5 | Encodeur rotatif (KY-040) | `Encoder_Rotatif` | `Connector_PinHeader_2.54mm:PinHeader_1x05_P2.54mm_Vertical` | créé, validé ; SW non assigné |
| 6 | Carte driver Nixie (boîte noire, 4 connecteurs) | `Nixie_Driver_Board` | `apex:Nixie_Board_Headers` (custom) | créé, validé ; SHDN/C1/C2 à confirmer |
| 7 | Tubes Nixie | *(annulé)* | — | carte d'affichage existante : couvert par le composant 6 |
| 8 | Alim HV 3-5 kV (module externe) | `HV_Module` | `apex:HV_Module_Conn` (custom) | créé, validé ; polarité EN à confirmer |
| 9 | SSR-25 DA | `SSR_25DA` | *(vide — montage hors PCB)* | créé, validé ; on_board = no |
| 10 | Module relais 12 V ×2 | `Relay_Module_12V` | `apex:Relay_Module_Conn` (custom) | créé, validé |
| 11 | Relais 5 V (coupe alim HV) | `Relay_Module_5V` | `apex:Relay_Module_Conn` (custom) | créé, validé ; nu/module non décidé |
| 12 | BT137-700E ×2 | `BT137-700E` | `Package_TO_SOT_THT:TO-220-3_Vertical` | créé, validé |
| 13 | MOC3043 ×2 | `MOC3043` | `Package_DIP:DIP-6_W7.62mm` | créé, validé |
| 14 | S9014 ×6 | `S9014` | `Package_TO_SOT_THT:TO-92` | créé, validé (E-B-C confirmé datasheet) |
| 15 | Résistances (1 k, 330-390, 180-360, 39) | `R` | `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal` | créé, validé |
| 16 | Condensateurs (100 nF, 100 µF, 10 nF) | `C` + `CP` | `Capacitor_THT:C_Rect_L7.0mm_W2.5mm_P5.00mm` + `Capacitor_THT:CP_Radial_D8.0mm_P3.50mm` | créé, validé ; X2 à confirmer |
| 17 | Diodes (1N4007, 1N4148) | `D` | `Diode_THT:D_DO-41_SOD81_P10.16mm_Horizontal` (DO-35 pour 1N4148) | créé, validé |
| 18 | Connecteurs (borniers 5,08 + headers 2,54) | `Conn_Bornier_2P`, `Conn_Bornier_3P`, `Conn_Header_1x02`, `Conn_Header_1x03` | TerminalBlock_RND 205-00232/00233 + PinHeader 2,54 | créé, validé |
| 19 | Fusible + varistance 230 V | `Fuse` + `Varistor` | `Fuse:Fuse_Blade_ATO_directSolder` + `Varistor:RV_Disc_D12mm_W4.6mm_P7.5mm` | créé ; **dimensionnement à valider** |

Empreintes custom dans `apex.pretty` : `RPi_Pico_SMD` (composant 1), `Nixie_Board_Headers` (composant 6), `HV_Module_Conn` (composant 8), `Relay_Module_Conn` (composants 10-11).

## Détail par composant

### 1. Raspberry Pi Pico (variante) — `Raspberry_Pi_Pico` / `apex:RPi_Pico_SMD`
- Carte réelle = **variante du Pico** (brochage validé par l'utilisateur, 2026-11-15). Colonne gauche 1-20 = standard. Colonne droite : 21=GP16, 22=GP17, 23=GND, 24=GP18, 25=GP19, 26=GP20, 27=GP21, 28=GND, 29=GP22, 30=RUN, 31=GP26, 32=GP27, **33=GND (pas d'AGND)**, 34=GP28, **35=GP29**, **36=3V3**, **37=GP23 (pas de 3V3_EN)**, 38=GND, **39=VIN**, **40=VOUT**.
- Types électriques : GP0-GP23, GP26-GP29 = bidirectional ; GND ×8, VIN = power_in ; 3V3, VOUT = power_out ; RUN = input.
- 27 fonctions alternatives renseignées (assignation projet) : UART0_TX/RX (GP0/1, debug), I2C1_SDA/SCL (GP2/3 → BME280), REL_5V (GP4), REL_12V_A/B (GP5/6), TRIAC_A/B (GP7/8 → MOC3043), SSR_MOTOR (GP9), SPI1_SCK/MOSI (GP10/11) + NIXIE_RCLK (GP13, latch GPIO), OWIRE (GP12, 1-Wire PIO → DS18B20), ENC_A/B (GP14/15), SPI0_MISO/CS_TFT/SCK/MOSI (GP16-19 → écran), TFT_DC/RST (GP20/21), TOUCH_CS (GP22), SD_CS (GP26), NIXIE_SHDN (GP27), ADC2 (GP28), ADC3 (GP29). GP23 libre (aucune fonction projet).
- Alimentation : VIN (39) = entrée 5 V depuis le rail 5 V du PCB ; 3V3 (36) = sortie du régulateur embarqué → bus 1-Wire + BME280 ; VOUT (40) = sémantique inconnue (clone générique) — type power_out, sera laissée NC dans notre design.
- Remarque : la spec disait « GND ×8 » — la carte variante a effectivement 8 × GND (le Pico officiel a 7 GND + AGND).
- Carte = clone générique USB-C (pas de doc constructeur) : le champ Datasheet pointe vers le datasheet Pico officiel (référence mécanique + RP2040), le brochage variante est documenté dans le champ Description du symbole.
- Empreinte `RPi_Pico_SMD` : montage **castellated** (décision utilisateur), adaptée de la lib KiCad-RP-Pico (ncarandini, licence MIT), convertie au format KiCad 10. 40 pads SMD 3,5×1,7 mm avec demi-perçages (drill offset ±0,9), lignes espacées de 17,78 mm, pas 2,54 mm ; + 3 pads SWD en bord bas : 41=SWCLK, 42=GND, 43=RUN (sans broche côté symbole — laissés non connectés, usage debug optionnel) ; + 4 pads de masse boîtier USB (pads A-D du datasheet §2.2, pour l'écoulement de la soudure du connecteur USB traversant). Modèle 3D retiré (chemin d'origine inexistant).
- Format mécanique : **confirmé identique au Pico officiel par l'utilisateur** (21×51 mm, pads castellated sur grille 0,1") — l'empreinte est valide telle quelle.
- Contrôles : `sym upgrade` rc 0, `fp upgrade` rc 0, export SVG OK.

### 2. DS18B20 — `DS18B20` / `Package_TO_SOT_THT:TO-92`
- TO-92 : **1=GND, 2=DQ, 3=VDD** (alimentation parasite possible en reliant VDD à GND côté sonde).
- Types : GND, VDD = power_in ; DQ = bidirectional.
- 8 sondes sur le **même bus 1-Wire** (GP12 via PIO) : un seul symbole suffit, à placer ×8 dans le schéma. Pull-up **1 kΩ** vers 3V3 (limite le courant de sink < 4 mA).
- Les sondes arrivent par câble : prévoir `Conn_Header_1x03` (3V3, DQ, GND) côté PCB.
- Datasheet : analog.com (ex-Maxim), champ Datasheet renseigné.

### 3. BME280 (breakout I2C) — `BME280_BoB` / `PinHeader_1x06_P2.54mm_Vertical`
- Breakout 6 broches : **1=VCC, 2=GND, 3=SCL, 4=SDA, 5=CSB, 6=SDO**.
- Types : VCC/GND power_in ; SCL input ; SDA bidirectional ; CSB, SDO input.
- Adresse I2C **0x76** (SDO=GND) ou **0x77** (SDO=VDD). Alimentation **3,3 V uniquement** (bus I2C1 du Pico : GP2/GP3).
- **À vérifier au câblage** : l'ordre réel des broches de la carte (variable selon le fabricant du breakout).

### 4. Écran TFT 2,8" ILI9341 + tactile + SD (MSP2807) — `TFT_MSP2807` / `PinHeader_1x14_P2.54mm_Vertical`
- 14 broches. Gauche 1-7 : **VCC, GND, TFT_CS, RST, DC, MOSI, SCK**. Droite 8-14 : **LED, MISO, SD_CS, T_CLK, T_CS, T_DIN, T_DO**.
- Types : VCC/GND power_in ; TFT_CS, RST, DC, MOSI, SCK, LED, SD_CS, T_CLK, T_CS, T_DIN = input ; MISO, T_DO = output.
- **VCC = 5 V** (rétroéclairage ~100 mA, jamais depuis 3V3(OUT) du Pico) ; les signaux logiques restent à 3,3 V (le module a ses level shifters).
- Bus SPI0 : SCK=GP18, MOSI=GP19, MISO=GP16, TFT_CS=GP17, DC=GP20, RST=GP21, T_CS=GP22, SD_CS=GP26.
- **À vérifier au câblage** : l'ordre réel des 14 broches du module (variable selon les fabricants MSP2807).

### 5. Encodeur rotatif (module type KY-040) — `Encoder_Rotatif` / `PinHeader_1x05_P2.54mm_Vertical`
- 5 broches : **1=A, 2=B, 3=SW, 4=+, 5=GND**.
- Types : A, B, SW = output (open-collector, pull-ups intégrées vers + sur le module KY-040) ; + = power_in ; GND = power_in.
- Hypothèse de travail = module KY-040 ; si encodeur nu, le symbole reste valable mais prévoir pull-ups externes.
- **SW (poussoir) non encore assigné** dans la table GPIO (candidats : GP23 ou GP29, tous deux libres) — à trancher au câblage. Pull-up externe requise pour SW.

### 6. Carte driver Nixie (module externe) — `Nixie_Driver_Board` / `apex:Nixie_Board_Headers`
- Boîte noire 13 broches, 4 connecteurs :
  - J1 (1-3) : **9-12V, GND, SHDN**
  - J2 (4-7) : **SER, SRCLK, RCLK, GND**
  - J3 (8-9) : **GND, HV** (sortie ~170 V vers les tubes — ne passe pas sur notre PCB)
  - J4 (10-13) : **5V, GND, C2, C1**
- Types : 9-12V, 5V, GND = power_in ; SHDN, SER, SRCLK, RCLK = input ; HV = power_out ; C1, C2 = passive.
- Fonction : driver 74HC595 + boost HV ~170 V, pilotage **SPI matériel** : SER=SPI1 MOSI (GP11), SRCLK=SPI1 SCK (GP10), RCLK=latch GPIO (GP13), SHDN=GP27.
- **À confirmer** : polarité/active level de SHDN, fonction exacte de C1/C2.
- Empreinte custom `Nixie_Board_Headers` : 4 groupes de headers 2,54 mm (J1×3, J2×4, J3×2, J4×4) alignés, 13 pads THT perçage 1,0 mm / pad Ø1,7 mm (pad 1 carré), boîtes sérigraphiées + labels J1-J4, courtyard généreux.

### 8. Alim HV chambre 3-5 kV (module externe) — `HV_Module` / `apex:HV_Module_Conn`
- 4 broches : **1=VCC (5 V), 2=GND, 3=HV_OUT (3-5 kV), 4=EN** (activation, optionnelle).
- Disposition : VCC/GND/EN à gauche du symbole, **HV_OUT isolé à droite** — l'empreinte place le pad HV à 20,32 mm du header logique (perçage 1,2 mm, pad Ø2,54 mm).
- **TENSION MORTELLE** : la sortie HV ne passe pas sur le PCB — câble direct du module vers un bornier de départ dédié (`Conn_Bornier_2P`), largement espacé du reste. Le relais 5 V (composant 11) coupe l'entrée 5 V de ce module.
- **À confirmer** : polarité/nécessité de EN sur le module réel (PCB vert).

### 9. SSR-25 DA (relais statique) — `SSR_25DA` / *(empreinte vide)*
- 4 bornes : **1=IN+ , 2=IN-** (entrée DC 3-32 V) ; **3=OUT1, 4=OUT2** (sortie 24-380 VAC, 25 A).
- Tous les types = passive (boîte noire) ; ligne pointillée de séparation DC/AC dans le glyphe.
- `on_board = no` : montage **hors PCB** (rail DIN/panier), connexion par les bornes à vis du module — l'empreinte est volontairement **vide** (champ Footprint non renseigné).
- Commande : GP9 → S9014 → SSR IN (le GPIO ne peut pas piloter la LED du SSR directement si plusieurs unités ; à figer au câblage).

### 10. Module relais 12 V (×2) — `Relay_Module_12V` / `apex:Relay_Module_Conn`
- 6 broches : gauche **1=IN, 2=VCC (12 V), 3=GND** ; droite **4=COM, 5=NO, 6=NC**.
- Types : IN = input (**commande active basse** sur ces modules) ; VCC/GND = power_in ; COM/NO/NC = passive.
- ×2 dans le design (GP5, GP6). Contact 10 A 250 VAC.
- Empreinte custom `Relay_Module_Conn` : header commande 1×03 + header contact 1×03, pas 2,54 mm, perçage 1,0 mm, labels CTRL/CONTACT.

### 11. Relais 5 V (coupe l'alim HV) — `Relay_Module_5V` / `apex:Relay_Module_Conn`
- Même structure que le composant 10, bobine **5 V**, piloté par GP4.
- **Décision non tranchée** : module tout fait (empreinte `Relay_Module_Conn`) ou relais nu. Si relais nu : empreinte `Relay_THT:Relay_SPDT_Finder_34.51_Vertical` + **diode de roue libre 1N4007** à prévoir (le symbole actuel suppose un module avec driver intégré).

### 12. BT137-700E (triac) — `BT137-700E` / `Package_TO_SOT_THT:TO-220-3_Vertical`
- TO-220-3 : **1=MT1, 2=MT2 (languette), 3=G**. 8 A RMS, 700 V.
- Types : MT1, MT2 = passive ; G = input. Glyphe triac standard (copie de `Device:Q_Triac`).
- ×2, gâchette commandée via MOC3043 ; snubber RC **39 Ω + 10 nF** à prévoir en parallèle sur MT1/MT2.
- Datasheet NXP renseignée dans le champ Datasheet.

### 13. MOC3043 (optotriac zero-cross) — `MOC3043` / `Package_DIP:DIP-6_W7.62mm`
- DIP-6 : **1=anode, 2=cathode** (LED, courant de déclenchement 5 mA max) ; **3=NC** ; **4=MAIN1, 6=MAIN2** (bornes principales) ; **5=NC = substrat, NE PAS CONNECTER**.
- Types : A, K, MAIN1, MAIN2 = passive ; broches 3 et 5 = **no_connect** (ERC propre). Ligne pointillée d'isolation LED/triac dans le glyphe.
- ×2 (GP7, GP8) ; résistance LED **330-390 Ω** depuis le GPIO 3,3 V.
- Datasheet onsemi renseignée.

### 14. S9014 (NPN) — `S9014` / `Package_TO_SOT_THT:TO-92`
- TO-92 : **1=E, 2=B, 3=C** — ordre E-B-C **confirmé par la datasheet** (recherche web en session).
- Types : B = input ; C, E = passive. Glyphe NPN standard (copie de `Device:Q_NPN`).
- ×6, un par canal de puissance (relais 12 V ×2, relais 5 V, SSR, MOC ×2 — répartition exacte au câblage), montage tout-ou-rien, **résistance de base 1 kΩ**.

### 15. Résistances — `R` / `Resistor_THT:R_Axial_DIN0207_L6.3mm_D2.5mm_P10.16mm_Horizontal`
- Symbole générique 2 broches (glyphe `Device:R`), empreinte **0207 traversant** (résiste aux vibrations, facile à souder).
- Valeurs projet (à la pose dans le schéma) : **1 kΩ ×7** (1 pull-up 1-Wire + 6 bases S9014), **330-390 Ω ×2** (LED MOC3043), **180-360 Ω ×2** (gâchette BT137), **39 Ω ×2** (snubber triac).

### 16. Condensateurs — `C` / `CP`
- `C` (glyphe `Device:C`, 2 plaques) : **100 nF** découplage (un par CI) et **10 nF** snubber triac ×2 — empreinte `Capacitor_THT:C_Rect_L7.0mm_W2.5mm_P5.00mm`. **Classe X2 côté secteur à confirmer** (pas d'empreinte X2 dédiée dans KiCad 10 ; si X2 retenu, choisir une empreinte film MKT adaptée au pas).
- `CP` (glyphe `Device:C_Polarized`, plaque + arc, signe +) : **100 µF** sur le rail 5 V — empreinte `Capacitor_THT:CP_Radial_D8.0mm_P3.50mm`, polarité **+ = broche 1**.

### 17. Diodes — `D` / `Diode_THT:D_DO-41_SOD81_P10.16mm_Horizontal`
- Symbole générique (glyphe `Device:D`) : **broche 1 = cathode (côté bague)**, broche 2 = anode.
- **1N4007** ×6-8 (roue libre, ponts côté puissance) → DO-41. **1N4148** côté logique → changer l'empreinte pour `Diode_THT:D_DO-35_SOD27_P10.16mm_Horizontal` à la pose.

### 18. Connecteurs — `Conn_Bornier_2P`, `Conn_Bornier_3P`, `Conn_Header_1x02`, `Conn_Header_1x03`
- `Conn_Bornier_2P` / `TerminalBlock_RND:TerminalBlock_RND_205-00232_1x02_P5.08mm_Horizontal` : bornier à vis 5,08 mm — entrée 230 V, sortie moteur, entrées 12 V et 5 V, départ câble HV chambre.
- `Conn_Bornier_3P` / `TerminalBlock_RND:TerminalBlock_RND_205-00233_1x03_P5.08mm_Horizontal` : contacts relais (COM/NO/NC) et autres départs puissance.
- `Conn_Header_1x02` / `PinHeader_1x02_P2.54mm_Vertical` : UART debug (GP0/GP1) et signaux divers.
- `Conn_Header_1x03` / `PinHeader_1x03_P2.54mm_Vertical` : **bus 1-Wire des 8 sondes DS18B20 (3V3, DQ, GND)**.
- Les headers des modules (écran 1×14, carte Nixie custom, encodeur 1×05) sont portés par les symboles modules eux-mêmes — pas de symboles séparés.
- Tous pins passives nommées « ~ », référence J.

### 19. Fusible + varistance — `Fuse` / `Varistor`
- `Fuse` (glyphe `Device:Fuse`) / `Fuse:Fuse_Blade_ATO_directSolder` : fusible côté 230 V primaire. **DIMENSIONNEMENT À VALIDER avec l'utilisateur** (empreinte provisoire fusible lame à souder).
- `Varistor` (glyphe `Device:Varistor`) / `Varistor:RV_Disc_D12mm_W4.6mm_P7.5mm` : MOV de protection d'entrée secteur. **DIMENSIONNEMENT À VALIDER** (provisoire : disque 12 mm).

## Points ouverts (à trancher avant/au câblage)
1. **Fusible + MOV** : dimensionnement secteur 230 V (demande explicite de l'utilisateur : ne pas figer sans validation).
2. **SW encodeur** : non assigné dans la table GPIO — candidats GP23 ou GP29.
3. **Ordre des broches** BME280 (breakout), TFT MSP2807, encodeur KY-040 : à vérifier sur les cartes réelles.
4. **Relais 5 V** : module tout fait vs relais nu (+ diode roue libre).
5. **Carte Nixie** : polarité SHDN, fonction C1/C2.
6. **Module HV** : polarité/nécessité de EN.
7. **VOUT Pico (40)** : sémantique inconnue — laissée NC.
8. **Condensateur snubber** : classe X2 ou film standard côté secteur.

## Journal
- **2026-11-15** — Création de `apex.kicad_sym` (vide) et `apex.pretty/`, enregistrement dans `sym-lib-table` et `fp-lib-table`, syntaxe validée par `kicad-cli sym upgrade`.
- **2026-11-15** — Décisions utilisateur : Pico soudé castellated ; tubes Nixie hors périmètre (carte d'affichage existante, connecteurs du composant 6) ; relais 5 V générique affinable.
- **2026-11-15** — Composant 1 créé : symbole `Raspberry_Pi_Pico` (40 broches, 26 alternates) + empreinte `RPi_Pico_SMD` ; validé par parse et export SVG.
- **2026-11-15** — Bibliothèques déplacées dans `lib/` du repo (convention README, versionnement git) ; tables globales repointées vers le repo ; validation reconfirmée depuis le nouvel emplacement.
- **2026-11-15** — Brochage Pico corrigé d'après la carte réelle de l'utilisateur (variante : 33=GND, 35=GP29, 36=3V3, 37=GP23, 39=VIN, 40=VOUT) ; symbole régénéré et revalidé (parse + SVG).
- **2026-11-15** — Composant 1 verrouillé : clone générique USB-C au format Pico officiel (confirmé), VOUT (40) laissée NC (type power_out), empreinte castellated validée.
- **2026-11-15** — Composants 2-19 créés d'un bloc (autorisation utilisateur « tu peux faire tous les autres ») : 22 nouveaux symboles + 3 empreintes custom (`Nixie_Board_Headers`, `HV_Module_Conn`, `Relay_Module_Conn`). Glyphes passifs/triac/NPN/diode/fusible/varistance copiés de `Device.kicad_sym` (géométries exactes extraites par awk). Générateur : `/tmp/gen_apex_lib.py` (Pico conservé tel quel). Validations : `sym upgrade` rc 0, `fp upgrade` rc 0, exports SVG des 23 symboles et 4 empreintes OK (`/tmp/apex_sym_svg2/`, `/tmp/apex_fp_svg2/`). Brochages critiques re-vérifiés dans le fichier (MOC3043, Nixie, TFT, HV, S9014, BT137).

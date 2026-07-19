# Pneumatic Drilling System – HIL aplikacija (cRIO)

Upravljačka aplikacija za automatizovani pneumatski sistem bušenja, realizovana kao Hardware-in-the-Loop (HIL) projekat. Logika je napisana u LabVIEW i izvršava se na NI cRIO-9024 kontroleru, povezanom preko fizičkih I/O modula sa Lucas-Nülle Pro/Train softverskom simulacijom procesa (model SO6006-6CB).

## Autori
Ines Bogdanović, Miljana Matejić, Anđela Jelić — FTN Novi Sad, predmet *Upravljački algoritmi u realnom vremenu*, jul 2026.

## Hardver
- **NI cRIO-9024** (Real-Time kontroler) + **NI cRIO-9114** (FPGA šasija)
- **NI 9425** – digitalni ulazi (senzori B1–B9, dugmad S1–S4)
- **NI 9476** – digitalni izlazi (ventili Y1–Y7, relej K1, LED H1/H2)
- Rad u **Scan Interface** modu (IO Variable čvorovi, bez posebnog FPGA VI-ja)
- Veza sa Pro/Train simulacijom preko terminal panela, Ethernet IP 10.1.203.202

## Kako radi
Upravljačka logika je automat stanja (state machine) unutar jedne While petlje:

`Init → Wait_Start → Discharge_Mid → Separation_Forward → Drill_Down → Drill_Up → Separation_Back → Discharge_Up → Blow_Off → Discharge_Down → (nazad na Wait_Start)`

Dodatno:
- **Fault** – bezbednosno stanje, svi izlazi isključeni, H2 trepće
- **vracanje_u_pocetno** – aktivno vraćanje cilindara nakon greške
- **System_Off** – kontrolisano gašenje nakon završetka tekućeg ciklusa

## Bezbednosna logika
- Detekcija logičkih grešaka (Y1∧Y2, Y3∧Y4, Y5∧Y6, nelogična pozicija discharge cilindra, dva komada u sistemu, Emergency-Off)
- **End-position supervision** – opšti timeout mehanizam koji detektuje da li je cilindar stigao do krajnje pozicije u zadatom vremenskom roku
- Emergency-Off (S4, NC kontakt) trenutno isključuje sve izlaze

## Pokretanje
1. Otvoriti LabVIEW projekat, deploy-ovati glavni VI na cRIO Real-Time target
2. Pokrenuti Pro/Train Pneumatic Drilling System simulaciju na povezanom računaru
3. Proveriti fizičku vezu terminal panela (Y1–Y7, K1, H1, H2 → cRIO izlazi; B1–B9, S1–S4 → cRIO ulazi)
4. Pokrenuti VI, zatim pritisnuti S1 na simulaciji za start ciklusa

## Poznati problemi / napomene
- S1, S2 i S4 su NC (normalno zatvoreni) kontakti — signali se čitaju invertovano
- Redosled Discharge_Mid pre Separation_Forward je obavezan (sprečava upadanje komada u šaht)
- Blow_Off tajmer koristi ručni Tick Count mehanizam umesto Elapsed Time Express VI-ja (nepouzdan pri ponovljenim ulascima)

Detaljan opis, screenshotovi block dijagrama i puna lista rešenih problema nalaze se u pratećoj dokumentaciji (`Dokumentacija_HIL_Busilica.docx`).

# 01 — Visione e Obiettivi Strategici

## 1.1 Visione

Realizzare una **piattaforma informatica nazionale**, multiutente e sincronizzata in tempo reale, per la gestione integrata dei processi di fisica sanitaria e radioprotezione, conforme al D.Lgs. 101/20 e s.m.i.

Il sistema deve essere:
- **Replicabile** su qualsiasi ASL o Azienda Ospedaliera italiana
- **Multi-azienda**: una singola installazione serve più enti contemporaneamente
- **Multiutente profilato**: ogni utente vede solo ciò che compete al suo ruolo
- **Mobile-first**: accessibile da PC, tablet e smartphone
- **Sincronizzato in tempo reale**: nessun dato obsoleto tra sessioni diverse

---

## 1.2 Obiettivi strategici

### Standardizzazione
Uniformare i processi di fisica sanitaria e radioprotezione tra le diverse ASL e Aziende Ospedaliere italiane, eliminando le variazioni di qualità legate alle prassi locali.

### Armonizzazione normativa
Garantire livelli omogenei di qualità e sicurezza attraverso l'implementazione diretta dei requisiti del D.Lgs. 101/20 (Titoli XI e XIII) e del DM 01/21 nella logica applicativa.

### Trasparenza e tracciabilità
Ogni attività soggetta a vigilanza (controlli di qualità, dosimetria, verifiche periodiche, notifiche) deve essere tracciata con autore, data e allegati, mantenendo uno storico immutabile.

### Governance delle strutture
Fornire a responsabili e datori di lavoro dati integrati e indicatori aggiornati in tempo reale per supportare decisioni informate sulla gestione del parco tecnologico radiologico.

### Health Technology Assessment (HTA) di processo
Supportare l'intero ciclo di vita delle tecnologie radiologiche: dalla selezione e accettazione, alla gestione operativa, fino alla cessazione.

### Appropriatezza
Migliorare l'appropriatezza amministrativa, tecnica e clinica nell'utilizzo delle risorse tecnologiche, riducendo adempimenti manuali e il rischio di omissioni.

---

## 1.3 Modello di commercializzazione previsto

Il sistema, già sperimentato presso **AOU Federico II di Napoli**, è destinato a diventare un **servizio AIFM** per i datori di lavoro delle strutture sanitarie.

### Fasi previste

| Fase | Attività |
|------|----------|
| **Validazione** | Validazione di ciascun modulo mediante piccoli team di EFM e EDR dello specifico settore |
| **Industrializzazione** | Costituzione di una software house dedicata per progettare la piattaforma integrata, strutturata, multiutente e multi-azienda per utilizzo nazionale |
| **Commercializzazione** | Distribuzione come servizio/prodotto AIFM (modello SaaS o licenza) |

---

## 1.4 Ambiti di adempimento

Il sistema gestisce due macro-ambiti normativi distinti ma collegati, che spesso insistono sullo stesso centro di responsabilità:

### Ambito A — Fisica Medica (Titolo XIII D.Lgs. 101/20)
Riguarda tutto ciò che concerne **il paziente** nelle esposizioni mediche.

**Figure professionali responsabili (a supporto del Datore di Lavoro):**
- Responsabile Impianto Radiologico (RIR)
- Specialista in Fisica Medica (SFM)

### Ambito B — Radioprotezione (Titolo XI D.Lgs. 101/20)
Riguarda **ambienti e lavoratori** (sorveglianza fisica, dosimetria, zone classificate).

**Figure professionali responsabili** — dipendono dalla tipologia di area:

| Tipologia di area | Esperto di Radioprotezione richiesto |
|-------------------|--------------------------------------|
| Radiologia diagnostica (TAC, Endorale, Telecomandati) | 1° grado di abilitazione |
| Radiologia interventistica (sale operatorie con RX) | 1° grado di abilitazione |
| Medicina Nucleare | 2° grado di abilitazione |
| Radioterapia e impianti che producono neutroni | 3° grado di abilitazione (con ulteriore suddivisione "sanitario" / "industriale") |

> **Nota implementativa:** I due ambiti devono essere gestiti in maniera distinta e autonoma all'interno dell'applicazione, pur condividendo i dati anagrafici delle apparecchiature e degli utenti.

---

## 1.5 Integrazione con sistemi esterni

| Sistema esterno | Tipo di integrazione | Note |
|----------------|----------------------|------|
| **STRIMS** (ISIN) | Registrazione apparecchiature, Notifica di Pratica, Notifica di Cessazione | Attualmente manuale via credenziali incaricato DL |
| **INAIL** | Registrazione polizze assicurative apparecchiature | Ricevuta, stato, data registrazione |
| **RSPP** | Trasmissione Documento Valutazione Rischio radiazioni ionizzanti | Collegamento informativo |
| **MS Office** | Import/archiviazione documenti Excel, Word, PDF, immagini | Requisito funzionale esplicito |
| **SAP / SIAP** | Dati identificativi bene aziendale (cespiti) | Collegamento opzionale |
| **MedSquare** | Software installato su apparecchiatura per determinazione LDR | Lettura dati/parametri |
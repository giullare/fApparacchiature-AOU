# Piattaforma Unificata — Fisica Sanitaria e Radioprotezione
## Documentazione Tecnica per lo Sviluppo

> **Versione documento:** 1.0  
> **Riferimento normativo principale:** D.Lgs. 101/20 e s.m.i., DM 01/21 e s.m.i.  
> **Origine:** UOSD Fisica Sanitaria e Radioprotezione, AOU Federico II  
> **Proposta istituzionale:** AIFM (Associazione Italiana di Fisica Medica)

---

## Indice dei file di specifica

| File | Contenuto |
|------|-----------|
| `01_vision_e_obiettivi.md` | Visione di progetto, obiettivi strategici, destinatari |
| `02_architettura_generale.md` | Architettura tecnica, moduli, stack consigliato |
| `03_modulo_apparecchiature_radiologiche.md` | Modulo 1 — Gestione grandi apparecchiature (D.Lgs. 101/20) |
| `04_modulo_risonanza_magnetica.md` | Modulo 2 — Gestione RM (DM 01/21) |
| `05_modulo_registri_medicina_nucleare.md` | Modulo 3 — Registri sorveglianza Medicina Nucleare |
| `06_modulo_radioterapia_metabolica.md` | Modulo 4 — Radioterapia metabolica con Lu177 |
| `07_anagrafica_apparecchiature.md` | Schema dati completo dell'entità Apparecchiatura |
| `08_adempimenti_fisica_medica.md` | Flussi adempimenti Fisica Medica (Titolo XIII) |
| `09_adempimenti_radioprotezione.md` | Flussi adempimenti Radioprotezione (Titolo XI) |
| `10_gestione_utenti_e_ruoli.md` | Autenticazione, ruoli, permessi |
| `11_statistiche_e_reportistica.md` | Cruscotti, KPI, reportistica operativa |
| `12_modello_dati_database.md` | Schema E/R e tabelle principali |
| `13_requisiti_non_funzionali.md` | Performance, sicurezza, accessibilità, scalabilità |
| `14_roadmap_e_milestones.md` | Fasi di sviluppo, validazione, rilascio |

---

## Descrizione sintetica del progetto

La **Piattaforma Unificata** è un sistema software gestionale multi-tenant, accessibile da web e dispositivi mobili, progettato per centralizzare e digitalizare tutti gli adempimenti normativi a carico delle strutture di **Fisica Sanitaria e Radioprotezione** degli ospedali e delle ASL italiane.

Il sistema integra in un unico ambiente:
- la gestione dell'inventario e del ciclo di vita delle apparecchiature radiologiche e di Risonanza Magnetica
- i registri di sorveglianza in Medicina Nucleare
- il monitoraggio dosimetrico ambientale e degli operatori
- la documentazione regolamentare richiesta da D.Lgs. 101/20 e DM 01/21
- la gestione dei pazienti in radioterapia metabolica con Lu177

### Contesto e motivazione

In un contesto caratterizzato da:
- **eterogeneità organizzativa** tra le diverse strutture sanitarie
- **limitata disponibilità di personale** specializzato (Fisici Medici, Esperti di Radioprotezione)
- **crescente complessità normativa** (D.Lgs. 101/20, DM 01/21)
- **necessità di tracciabilità** per le attività soggette a vigilanza

…diventa strategico disporre di uno strumento digitale che garantisca standardizzazione, tracciabilità e armonizzazione dei processi su scala nazionale.

### Destinatari finali

- **Esperti di Radioprotezione (EDR/EFM)** — principali utenti operativi
- **Responsabili Impianto Radiologico (RIR)** — approvazioni e supervisione clinica
- **Specialisti in Fisica Medica (SFM)** — validazione tecnica e protocolli
- **Medici Autorizzati (MA)** — per RM e medicina nucleare
- **Datore di Lavoro e RSPP** — accesso in lettura a reportistica e compliance
- **Amministratori di sistema** — gestione multi-azienda

---

## Tre pilastri fondamentali

```
┌─────────────────────────────────────────────────────────────┐
│  1. INTEGRAZIONE E CENTRALIZZAZIONE                         │
│     Un unico ambiente per aree tecnologiche diverse         │
├─────────────────────────────────────────────────────────────┤
│  2. CONFORMITÀ SEMPLIFICATA                                 │
│     Gestione documentale strutturata, registri digitali,    │
│     tracciabilità dei controlli (D.Lgs. 101/20, DM 01/21)   │
├─────────────────────────────────────────────────────────────┤
│  3. CONTROLLO BASATO SUI DATI                               │
│     Analisi, reporting, KPI, monitoraggio continuo          │
└─────────────────────────────────────────────────────────────┘
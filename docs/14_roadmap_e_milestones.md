# 14 — Roadmap e Milestones di Sviluppo

---

## 14.1 Principi di sviluppo

Lo sviluppo segue un approccio **iterativo e modulare**: ogni modulo è sviluppato, validato e rilasciato in modo indipendente, permettendo di avere valore operativo sin dalle prime fasi.

La sequenza è determinata da:
1. **Priorità normativa** — adempimenti con scadenze più critiche
2. **Volume di utilizzo** — moduli usati quotidianamente da più utenti
3. **Dipendenze tecniche** — infrastruttura e funzionalità trasversali prima dei moduli specifici

---

## 14.2 Fasi di sviluppo

### FASE 0 — Fondamenta (Mese 1–2)
> Infrastruttura, autenticazione, struttura base. Nessuna funzionalità di business.

**Deliverables:**
- [ ] Setup monorepo (frontend React + backend NestJS)
- [ ] Pipeline CI/CD (GitHub Actions: lint, test, build, deploy su staging)
- [ ] Database PostgreSQL con RLS e schema base (`organizations`, `users`, `tenant_memberships`)
- [ ] Sistema di autenticazione completo (login, JWT, refresh token, MFA opzionale)
- [ ] Gestione multi-tenant: selezione tenant post-login
- [ ] Struttura RBAC: definizione ruoli e middleware di autorizzazione
- [ ] Bucket S3 + gestione file allegati (upload, download presigned URL, virus scan)
- [ ] Sistema di notifiche email (SMTP, template base)
- [ ] Audit log infrastruttura
- [ ] Ambiente di staging operativo
- [ ] Design system UI (palette colori, componenti base: Button, Input, Table, Modal, Toast)

**Criteri di accettazione Fase 0:**
- Un utente può creare un'organizzazione, fare login, selezionare il tenant e accedere alla dashboard vuota
- I ruoli limitano l'accesso alle sezioni (anche se ancora vuote)
- Un file può essere caricato e scaricato

---

### FASE 1 — Modulo 1: Anagrafica Apparecchiature (Mese 3–5)
> Il cuore del sistema. Registrazione e gestione del parco apparecchiature.

**Deliverables:**
- [ ] Tabelle database: `apparecchiature`, `parametri_radiologici`, `figure_responsabili`, `riferimenti_gestionali`, collocazione (siti/immobili/piani/locali/reparti)
- [ ] API CRUD completa per apparecchiature con validazioni BIZ-001..010
- [ ] UI: lista apparecchiature con filtri (ambito, stato, reparto, tipologia)
- [ ] UI: scheda apparecchiatura completa (tutte le sezioni §3.3)
- [ ] UI: wizard creazione nuova apparecchiatura
- [ ] Gestione upload foto e allegati generici
- [ ] Gestione adempimenti INAIL e STRIMS (campi + upload ricevute)
- [ ] Ciclo di vita apparecchiatura (stati e transizioni)
- [ ] Sezione Apparecchiature Cessate e Da Installare
- [ ] Struttura norme interne (upload e associazione a ambito/tipo/apparecchiatura)
- [ ] Struttura segnaletica (upload template per zona)
- [ ] Pannello compliance per singola apparecchiatura (§8.8, §9.12)
- [ ] Notifiche: alert scadenze (struttura base)

**Criteri di accettazione Fase 1:**
- Un EFM può censire un'apparecchiatura con tutti i suoi dati
- I documenti sono associabili all'apparecchiatura
- Il pannello compliance mostra lo stato degli adempimenti (anche con dati mancanti)

---

### FASE 2 — Modulo 1: Adempimenti Fisica Medica (Mese 5–7)
> Protocolli di verifica, collaudo, CQ periodici, LDR.

**Deliverables:**
- [ ] Tabelle database: `protocolli_verifica`, `record_verifiche`
- [ ] Gestione catalogo protocolli (CRUD con versioning)
- [ ] Flusso accettazione completo (§8.3): creazione record, allegati, benestare SFM + RIR
- [ ] Flusso verifiche periodiche (§8.4): calendario, creazione record, report, esito
- [ ] Flusso verifiche post-manutenzione (§8.5): cambio stato apparecchiatura, verifica bloccante
- [ ] Gestione LDR (§8.6): record annuale/biennale, upload report
- [ ] Generazione Manuale di Qualità in PDF (parti statiche + sezioni dinamiche auto-popolate)
- [ ] Alert automatici per scadenze CQ (30gg, 7gg, giorno stesso)
- [ ] Dashboard EFM: lista CQ in scadenza per apparecchiatura
- [ ] Statistiche Modulo 1 — Fisica Medica (§11.3)

**Criteri di accettazione Fase 2:**
- L'intero ciclo CQ (protocollo → esecuzione → report → esito) è gestibile dalla piattaforma
- Il Manuale di Qualità si genera in PDF correttamente con le sezioni dinamiche aggiornate
- Gli alert via email arrivano con l'anticipo configurato

---

### FASE 3 — Modulo 1: Adempimenti Radioprotezione (Mese 7–9)
> Notifiche, NO, prima verifica, sorveglianza periodica, verbali EDR.

**Deliverables:**
- [ ] Tabelle database: `notifiche_pratica`, `nulla_osta`, `benestare_utilizzo`, `verbali_sopralluogo`, `adempimenti_strims`
- [ ] Gestione Notifica di Pratica (§9.2): form, upload, protocollo PEC, fac-simile
- [ ] Gestione Nulla Osta (§9.3): tipo A/B, scadenza, stato
- [ ] Gestione Benestare Utilizzo (§9.4)
- [ ] Flusso Prima Verifica EDR (§9.5)
- [ ] Flusso Sorveglianza Periodica EDR (§9.6)
- [ ] Gestione Notifica di Cessazione (§9.7) con flusso cambio stato apparecchiatura
- [ ] Gestione STRIMS (§9.8) e INAIL (§9.9) — campi avanzati
- [ ] Verbali di Sopralluogo (§9.10): form + upload + gestione azioni correttive
- [ ] Pannello compliance EDR per apparecchiatura (§9.12) — completamento
- [ ] Alert scadenze EDR (NO, sorveglianze)

**Criteri di accettazione Fase 3:**
- Un EDR può gestire l'intero ciclo documentale di un'apparecchiatura (dalla notifica alla cessazione)
- Il pannello compliance mostra verde/giallo/rosso corretto per ogni adempimento EDR

---

### FASE 4 — Modulo 2: Risonanza Magnetica (Mese 9–11)

**Deliverables:**
- [ ] Schema database RM (estensione `apparecchiature` con dati RM-specifici)
- [ ] UI: scheda apparecchiatura RM con campi specifici (§4.3)
- [ ] Gestione CAI (§4.4.1)
- [ ] Gestione Benestare RM (§4.4.2)
- [ ] Controlli periodici semestrali con checklist RM (§4.4.3)
- [ ] Verbali periodici RM (§4.4.4)
- [ ] Inventario strumentazione RM + alert calibrazioni (§4.4.5)
- [ ] Segnaletica RM
- [ ] Statistiche Modulo 2 (§11.4)

---

### FASE 5 — Modulo 3: Registri Medicina Nucleare (Mese 11–14)

**Deliverables:**
- [ ] Schema database MN completo (§5.x)
- [ ] UI: sezione Dosimetria Ambientale — dosimetri passivi e sonde
- [ ] UI: sezione Contaminazione Superficiale
- [ ] UI: Inventario Radiofarmaci e Sorgenti con calcolo decadimento automatico
- [ ] UI: Registro Rifiuti con calcolo data smaltimento automatica
- [ ] UI: Inventario Strumentazione
- [ ] UI: Controllo Ventilazione e Pressione
- [ ] Generazione Verbale Periodico auto-compilato (PDF + versione stampabile) — §5.10
- [ ] Statistiche Modulo 3 (§11.5)
- [ ] Alert calibrazioni strumentazione

---

### FASE 6 — Modulo 4: Radioterapia Metabolica Lu177 (Mese 14–17)

**Deliverables:**
- [ ] Schema database Lu177 completo (§6.x)
- [ ] UI: Anagrafica Pazienti Lu177 con pseudonimizzazione
- [ ] UI: Gestione Cicli di Trattamento
- [ ] UI: Acquisizioni scintigrafiche per dosimetria
- [ ] UI: Dosimetria per organo con calcolo dose cumulativa
- [ ] UI: Follow-up ematologico con grading CTCAE
- [ ] UI: Gestione Rifiuti correlati al trattamento
- [ ] Statistiche cliniche aggregate anonimizzate (§11.6)
- [ ] Export dati pseudonimizzati per analisi statistica

---

### FASE 7 — Reportistica Avanzata e Dashboard (Mese 17–19)

**Deliverables:**
- [ ] Dashboard principale personalizzata per ruolo (§11.2)
- [ ] Report Annuale CQ (PDF multi-apparecchiatura)
- [ ] Report Compliance (PDF, Excel)
- [ ] Report Scadenze settimanale automatico
- [ ] Export CSV/Excel per tutte le sezioni
- [ ] Sistema notifiche completo (digest configurabile, notifiche in-app con campana)
- [ ] Statistiche generali e per apparecchiatura — Modulo 1 completate (§11.3)
- [ ] Report dosimetrico operatori (struttura base, in attesa Fase 8)

---

### FASE 8 — Dosimetria Operatori (Mese 19–22)

**Deliverables:**
- [ ] Schema database dosimetria personale (§9.11)
- [ ] Anagrafica lavoratori esposti
- [ ] Gestione assegnazione dosimetri personali
- [ ] Inserimento letture periodiche
- [ ] Confronto con limiti normativi e livelli di indagine
- [ ] "Cassetto virtuale" operatore: vista personale dosi cumulate
- [ ] Report dosimetrico per lavoratore

---

### FASE 9 — Hardening, Accessibilità e Go-Live (Mese 22–24)

**Deliverables:**
- [ ] Audit di accessibilità WCAG 2.1 AA completo + remediation
- [ ] Penetration test esterno + remediation vulnerabilità critiche/alte
- [ ] Load test: simulazione 100 utenti concorrenti per tenant, 50 tenant simultanei
- [ ] Verifica SLA performance (§13.2)
- [ ] Procedure di backup/restore testate e documentate
- [ ] Disaster recovery drill
- [ ] Documentazione utente (manuale operativo per ogni ruolo)
- [ ] Materiale di formazione (video tutorial, FAQ)
- [ ] Piano di migrazione dati per i tenant pilota (da Excel/sistemi attuali)
- [ ] Go-live con primi tenant pilota (validazione modulare con team EFM/EDR)

---

## 14.3 Riepilogo Timeline

```
Mese:   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24
        ├───────────┤
FASE 0  █████████████
        
FASE 1              ███████████████
        
FASE 2                          ███████████████
        
FASE 3                                      ███████████████
        
FASE 4                                                  ███████████████
        
FASE 5                                                              ████████████████████
        
FASE 6                                                                                  ████████████
        
FASE 7                                                                                      ████████
        
FASE 8                                                                                          ████████████
        
FASE 9                                                                                                  █████
```

---

## 14.4 Validazione Modulare (come da proposta AIFM)

Prima della commercializzazione, ogni modulo deve essere **validato** da un team dedicato di EFM e EDR dello specifico settore.

| Modulo | Team di validazione | Criteri |
|--------|--------------------|---------| 
| Modulo 1 — Fisica Medica | 3–5 SFM/EFM da strutture diverse | Completezza adempimenti, usabilità, correttezza Manuale di Qualità |
| Modulo 1 — Radioprotezione | 3–5 EDR da strutture diverse | Completezza adempimenti, flusso documentale |
| Modulo 2 — RM | 2–3 ES/MR | Conformità DM 01/21 |
| Modulo 3 — MN | 2–3 EDR specializzati MN | Registri corretti, calcolo decadimento |
| Modulo 4 — Lu177 | 2–3 EFM + oncologi nucleari | Dosimetria corretta, flusso clinico |

**Processo di validazione:**
1. Accesso al sistema staging con dati reali anonimizzati
2. Svolgimento delle attività operative tipiche per 4–8 settimane
3. Raccolta feedback strutturato (questionario + sessioni di walkthrough)
4. Prioritizzazione e implementazione feedback critici
5. Sign-off formale prima del rilascio modulo in produzione

---

## 14.5 Commercializzazione come servizio AIFM

| Step | Attività |
|------|----------|
| **Costituzione software house** | Entità legale dedicata allo sviluppo e manutenzione della piattaforma |
| **Modello di pricing** | SaaS per tenant (es. tariffa per moduli attivi + numero apparecchiature) |
| **Contratto DPA** | Accordo sul trattamento dei dati personali tra AIFM/software house e ogni ente |
| **Supporto** | Livelli di supporto: base (email, 5 giorni lavorativi), premium (email+telefono, 1 giorno), enterprise (SLA personalizzato) |
| **Aggiornamenti normativi** | Processo per aggiornare la piattaforma a seguito di modifiche normative (D.Lgs., DM, circolari ISIN) entro 60 giorni dalla pubblicazione |
| **Community** | Forum AIFM per condivisione best practice tra utenti della piattaforma |
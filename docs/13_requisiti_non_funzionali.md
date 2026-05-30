# 13 — Requisiti Non Funzionali

---

## 13.1 Panoramica

I requisiti non funzionali definiscono i **vincoli di qualità** del sistema, indipendenti dalle funzionalità specifiche. Sono vincolanti per l'architettura e devono essere verificati tramite test dedicati prima del rilascio in produzione.

---

## 13.2 Performance

### 13.2.1 Tempi di risposta API

| Tipo di operazione | Target p50 | Target p95 | Limite assoluto |
|--------------------|:----------:|:----------:|:---------------:|
| GET lista apparecchiature (paginata) | < 200 ms | < 500 ms | 2 s |
| GET scheda singola apparecchiatura | < 150 ms | < 400 ms | 1 s |
| POST / PUT (creazione/modifica record) | < 300 ms | < 800 ms | 3 s |
| Generazione report PDF (semplice) | < 3 s | < 8 s | 30 s |
| Generazione report PDF (complesso, >20 pagine) | < 10 s | < 25 s | 60 s |
| Upload file (escluso trasferimento binario) | < 500 ms | < 1.5 s | 5 s |
| Query statistiche aggregate | < 1 s | < 3 s | 10 s |
| Login + emissione token | < 500 ms | < 1 s | 3 s |

> Tutti i valori escludono la latenza di rete client-server.

### 13.2.2 Carico concorrente

| Scenario | Target |
|----------|--------|
| Utenti attivi simultanei per tenant (tipico) | 10–50 |
| Utenti attivi simultanei per tenant (picco) | 100 |
| Tenant attivi simultaneamente sulla piattaforma | 50+ |
| Request/sec totali in condizioni di picco | 500 rps |

### 13.2.3 Dimensione dati

| Entità | Stima per tenant medio | Note |
|--------|----------------------|------|
| Apparecchiature | 50–500 | Crescita lenta |
| Record verifiche per anno | 200–2000 | — |
| File allegati per anno | 500–5000 | Dimensione media 2–10 MB |
| Pazienti Lu177 attivi | 10–100 | — |
| Letture dosimetri per anno | 1000–10000 | — |

**Storage stimato per tenant/anno:** 10–50 GB (file allegati dominanti)

### 13.2.4 Ottimizzazioni obbligatorie

- **Paginazione cursor-based** su tutte le list API (nessun `OFFSET` su tabelle > 10.000 righe)
- **Lazy loading** dei file allegati (URL presignati S3 generati on-demand, non in listing)
- **Indici database** come da schema §12 — validati con `EXPLAIN ANALYZE` prima del rilascio
- **Caching Redis** per: sessioni JWT, lookup table (tipologie, norme), configurazioni tenant
- **CDN** per asset statici frontend (JS, CSS, immagini UI)
- **Job asincroni** per: generazione PDF, invio email, calcolo statistiche aggregate pesanti

---

## 13.3 Disponibilità e Affidabilità

### 13.3.1 SLA target

| Metrica | Target |
|---------|--------|
| Uptime annuale | ≥ 99.5% (≤ 43 ore/anno di downtime) |
| Uptime mensile | ≥ 99.0% |
| RTO (Recovery Time Objective) | ≤ 4 ore |
| RPO (Recovery Point Objective) | ≤ 1 ora |

### 13.3.2 Backup

| Tipo | Frequenza | Retention | Storage |
|------|-----------|-----------|---------|
| Dump completo PostgreSQL | Giornaliero (ore 2:00) | 30 giorni | Bucket S3 separato |
| WAL / Point-in-Time Recovery | Continuo | 7 giorni | — |
| Snapshot file storage (S3) | Giornaliero | 30 giorni | — |
| Backup mensile cold | Mensile | 12 mesi | Storage a lungo termine |

### 13.3.3 Disaster Recovery

- Il sistema deve essere ripristinabile da backup in meno di 4 ore (RTO)
- La procedura di ripristino deve essere **documentata e testata** almeno una volta l'anno
- In ambienti di produzione: database su istanza multi-AZ (se cloud) o replica sincrona (se on-premise)

### 13.3.4 Graceful degradation

- In caso di indisponibilità del servizio di invio email: le notifiche vengono accodate e inviate al ripristino
- In caso di indisponibilità del bucket S3: l'applicazione continua a funzionare (lettura, inserimento dati) ma l'upload è disabilitato con messaggio esplicito
- In caso di perdita di connessione Redis: il sistema cade su verifica JWT stateless (senza blacklist)

---

## 13.4 Sicurezza

### 13.4.1 Sicurezza delle comunicazioni

| Requisito | Specifica |
|-----------|-----------|
| Trasporto | TLS 1.2 minimo, TLS 1.3 raccomandato. HTTP reindirizzato a HTTPS |
| HSTS | `Strict-Transport-Security: max-age=31536000; includeSubDomains` |
| Certificati | Certificati validi (Let's Encrypt o CA aziendale), nessun self-signed in produzione |
| CORS | Origin whitelist esplicita, nessun wildcard `*` in produzione |

### 13.4.2 Sicurezza applicativa

| Requisito | Implementazione |
|-----------|----------------|
| Autenticazione | JWT con RS256 (chiavi asimmetriche), refresh token rotation |
| Password | bcrypt cost 12, policy: ≥12 char, maiuscola + numero + speciale |
| Rate limiting | 5 tentativi login falliti → blocco 30 min; 100 req/min per IP su API generiche |
| CSRF | Token CSRF per form, SameSite=Strict sui cookie |
| XSS | Content Security Policy (CSP) restrittiva, escape output React (nativo) |
| SQL Injection | ORM con parametrizzazione, nessuna query SQL interpolata con input utente |
| File upload | Validazione MIME type lato server, scansione antivirus (ClamAV o equivalente), limite dimensione 50 MB/file |
| Secrets | Nessuna chiave hardcoded nel codice; uso di variabili d'ambiente o secret manager (Vault, AWS Secrets Manager) |
| Dipendenze | `npm audit` / `pip-audit` in CI/CD; aggiornamento dipendenze con vulnerabilità critiche entro 72 ore |

### 13.4.3 Sicurezza dei dati

| Requisito | Specifica |
|-----------|-----------|
| Dati a riposo | Database: cifratura del volume (AES-256 a livello OS/cloud). File S3: cifratura SSE-S3 o SSE-KMS |
| Dati MFA secret | Cifrati a livello applicativo prima di salvarli in DB (AES-256-GCM, chiave da vault) |
| Password dimenticata | Token monouso con scadenza 1 ora, invalidato al primo utilizzo |
| PII pazienti Lu177 | Accesso limitato ai ruoli EFM/EDR/ADMIN_ORG; pseudonimizzazione negli export statistici |
| Isolamento tenant | Row Level Security PostgreSQL + validazione tenant_id in ogni query ORM |
| Log sensibili | I log applicativi non contengono mai: password, token, dati anagrafici completi |

### 13.4.4 Penetration testing e vulnerability assessment

- **Prima del go-live:** vulnerability assessment completo (OWASP Top 10)
- **Annualmente:** penetration test esterno con report formale
- **Continuo:** SAST (static analysis) integrato nella pipeline CI/CD (es. SonarQube, Semgrep)
- **Dipendenze:** DAST passivo su ambiente di staging

---

## 13.5 Usabilità e Accessibilità

### 13.5.1 Usabilità

| Requisito | Dettaglio |
|-----------|-----------|
| Lingua interfaccia | Italiano (lingua primaria); predisposizione per internazionalizzazione (i18n) |
| Responsive design | Funzionale su desktop (1024px+), tablet (768px+), smartphone (360px+) |
| Browser supportati | Chrome 110+, Firefox 110+, Edge 110+, Safari 16+ |
| Onboarding | Wizard guidato per inserimento nuova apparecchiatura |
| Feedback utente | Spinner su operazioni async, toast di conferma/errore, progress bar per upload |
| Gestione errori | Messaggi di errore in italiano chiari, con indicazione dell'azione correttiva |
| Conferma azioni distruttive | Dialog di conferma per ogni azione irreversibile |
| Paginazione | Tutte le liste con paginazione visibile, numero risultati mostrato |

### 13.5.2 Accessibilità (WCAG 2.1)

Il sistema deve rispettare il livello **WCAG 2.1 AA**, come richiesto dalla normativa italiana per le PA (D.Lgs. 106/2018):

| Criterio | Requisito |
|----------|-----------|
| Contrasto testo | Rapporto ≥ 4.5:1 per testo normale, ≥ 3:1 per testo grande |
| Navigazione da tastiera | Tutti i componenti interattivi raggiungibili con Tab, focus visibile |
| Screen reader | Markup semantico HTML5, ARIA label dove necessario |
| Testo alternativo | Ogni immagine informativa ha `alt` descrittivo |
| Form | Label associato a ogni campo input, messaggi di errore associati all'input |
| Timeout | Avviso prima della scadenza sessione (con opzione di prolungamento) |
| Dimensione testo | Interfaccia funzionante fino a 200% zoom |

---

## 13.6 Scalabilità

### 13.6.1 Scalabilità orizzontale

L'applicazione deve essere progettata **stateless** per consentire la scalabilità orizzontale:

- **Backend API:** nessuno stato in memoria (sessioni su Redis, file su S3)
- **Scalabilità automatica:** compatibile con autoscaling Kubernetes (HPA)
- **Database:** read replica per query di reportistica; connection pooling con PgBouncer

### 13.6.2 Crescita prevista

| Dimensione | Anno 1 | Anno 3 | Anno 5 |
|------------|--------|--------|--------|
| Tenant attivi | 5–20 | 50–150 | 200–500 |
| Apparecchiature totali | 500 | 10.000 | 50.000 |
| Utenti totali | 100 | 1.500 | 5.000 |
| Storage file (totale) | 50 GB | 2 TB | 10 TB |

### 13.6.3 Strategia di crescita database

- **Fase 1** (< 100 tenant): singola istanza PostgreSQL con read replica
- **Fase 2** (100–500 tenant): valutare partizionamento tabelle per `tenant_id` (PostgreSQL native partitioning)
- **Fase 3** (> 500 tenant): schema separato per tenant o database separato per cluster geografici

---

## 13.7 Manutenibilità

### 13.7.1 Qualità del codice

| Requisito | Standard |
|-----------|---------|
| Linguaggio backend | TypeScript strict mode (nessun `any` non giustificato) |
| Copertura test unitari | ≥ 70% su logica di business |
| Copertura test integrazione | ≥ 50% su API endpoints |
| Linting | ESLint + Prettier (configurazione condivisa monorepo) |
| Code review | Almeno 1 reviewer per ogni PR in produzione |
| Documentazione API | Swagger/OpenAPI aggiornato ad ogni modifica endpoint |

### 13.7.2 Versionamento API

- Le API pubbliche seguono **Semantic Versioning** con prefisso di versione nell'URL: `/api/v1/...`
- Breaking changes richiedono nuova versione major con periodo di deprecazione di almeno 6 mesi
- Il changelog delle API è mantenuto e pubblicato

### 13.7.3 Monitoraggio e osservabilità

| Strumento | Utilizzo |
|-----------|---------|
| **Prometheus + Grafana** | Metriche di sistema (CPU, memoria, latenza, error rate) |
| **Loki** | Aggregazione log applicativi strutturati (JSON) |
| **Sentry** | Error tracking frontend e backend, alerting su nuovi errori |
| **Uptime monitor** | Check ogni minuto su endpoint di health (`/health`, `/ready`) |
| **Alert** | Notifica via email/Slack per: error rate > 1%, latenza p95 > soglia, disk > 85% |

### 13.7.4 Ambienti

| Ambiente | Scopo | Dati |
|----------|-------|------|
| `development` | Sviluppo locale | Dati sintetici (seed) |
| `staging` | Test integrazione, UAT | Anonimizzati da produzione |
| `production` | Utenti reali | Dati reali |

Promozione del codice: `development` → `staging` → `production` via CI/CD con approvazione manuale per il passaggio a produzione.

---

## 13.8 Conformità Normativa del Software

### 13.8.1 GDPR (Reg. UE 2016/679)

| Obbligo | Implementazione |
|---------|----------------|
| Consenso | Registrazione del consenso al trattamento dati al primo accesso |
| Diritto di accesso | Funzione export dati personali utente (JSON) entro 30 giorni |
| Diritto alla cancellazione | Procedura di anonimizzazione (non cancellazione per audit trail) |
| Data breach | Procedura documentata di notifica entro 72 ore al Garante |
| DPA (Data Processing Agreement) | Contratto con ogni tenant che definisce ruoli Controller/Processor |
| Registro dei trattamenti | Mantenuto e aggiornato dal team legale della software house |

### 13.8.2 Normativa sanitaria IT

| Norma | Impatto sul software |
|-------|---------------------|
| D.Lgs. 101/20 | Struttura dei moduli, campi obbligatori, flussi di adempimento |
| DM 01/21 | Modulo RM, periodicità controlli, documentazione CAI |
| D.Lgs. 196/03 (Privacy) | Gestione dati pazienti e lavoratori |
| CAD (D.Lgs. 82/05) | Dematerializzazione documenti, firme digitali |

### 13.8.3 Conservazione dei documenti

| Categoria documento | Periodo minimo conservazione | Note |
|--------------------|------------------------------|------|
| Manuale di Qualità | Vita dell'apparecchiatura + 5 anni | D.Lgs. 101/20 |
| Verbali sopralluogo | 5 anni | — |
| Dosimetrie personali | Fino a 30 anni dopo fine esposizione | D.Lgs. 101/20 art. specifico |
| Registri MN | 5 anni | — |
| Audit log | 5 anni | Compliance interna |
| Dati pazienti Lu177 | 10 anni dalla fine del trattamento | — |

Il sistema deve **impedire la cancellazione** di documenti entro il periodo di conservazione obbligatoria. La cancellazione logica è permessa solo dopo la scadenza del periodo, previa approvazione ADMIN_ORG.
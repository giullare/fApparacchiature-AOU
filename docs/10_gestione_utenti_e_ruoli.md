# 10 — Gestione Utenti, Ruoli e Permessi

---

## 10.1 Panoramica

Il sistema implementa un modello **RBAC (Role-Based Access Control)** multilivello con isolamento per tenant. Ogni utente appartiene a una o più organizzazioni (tenant) e ha uno o più ruoli che determinano cosa può vedere e fare.

---

## 10.2 Entità Utente

```typescript
interface Utente {
  id: string;                          // UUID
  email: string;                       // Identificatore univoco globale
  password_hash: string;               // bcrypt
  nome: string;
  cognome: string;
  telefono?: string;
  avatar_url?: string;

  // Multi-tenancy: un utente può appartenere a più organizzazioni
  tenant_memberships: TenantMembership[];

  // Stato account
  attivo: boolean;
  email_verificata: boolean;
  ultimo_accesso?: Date;
  password_cambiata_il?: Date;
  richiede_cambio_password: boolean;

  // Timestamps
  created_at: Date;
  updated_at: Date;
}

interface TenantMembership {
  utente_id: string;
  tenant_id: string;
  ruoli: Ruolo[];
  reparto_id?: string;                 // Se l'utente è limitato a un reparto
  attivo: boolean;
  data_inizio: Date;
  data_fine?: Date;                    // Per accessi temporanei
}
```

---

## 10.3 Organizzazioni (Tenant)

```typescript
interface Organizzazione {
  id: string;                          // UUID
  nome: string;                        // Es. "AOU Federico II"
  tipo: 'ASL' | 'AOU' | 'IRCCS' | 'PRIVATO' | 'ALTRO';
  codice_fiscale?: string;
  indirizzo: string;
  email_amministrativa: string;
  telefono?: string;

  // Configurazione
  logo_url?: string;
  fuso_orario: string;                 // Default "Europe/Rome"
  lingua: string;                      // Default "it"

  // Piano di abbonamento (per futura commercializzazione)
  piano: 'TRIAL' | 'BASE' | 'PROFESSIONAL' | 'ENTERPRISE';
  moduli_attivi: ('MOD_1' | 'MOD_2' | 'MOD_3' | 'MOD_4')[];
  scadenza_licenza?: Date;

  attiva: boolean;
  created_at: Date;
}
```

---

## 10.4 Ruoli e Permessi

### 10.4.1 Ruoli di sistema

| Ruolo | Sigla | Descrizione |
|-------|-------|-------------|
| Super Admin | `SUPER_ADMIN` | Accesso completo a tutti i tenant (uso interno software house) |
| Admin Organizzazione | `ADMIN_ORG` | Gestione completa del proprio tenant: utenti, configurazioni, tutti i moduli |
| Esperto in Fisica Medica | `EFM` | Gestione Modulo 1 (fisica medica), Modulo 4 |
| Esperto di Radioprotezione | `EDR` | Gestione Modulo 1 (radioprotezione), Modulo 2, Modulo 3 |
| Responsabile Impianto Radiologico | `RIR` | Approvazioni cliniche, lettura tutto il Modulo 1 |
| Specialista in Fisica Medica | `SFM` | Come EFM, focus su verifiche tecniche |
| Medico Autorizzato | `MA` | Specifico per RM, lettura Modulo 2 |
| Esperto Sicurezza RM | `ES` | Gestione Modulo 2 |
| Responsabile Qualità | `RQ` | Accesso in lettura a tutti i moduli, reportistica |
| Operatore | `OPERATORE` | Inserimento dati limitato (es. letture dosimetri) |
| Sola Lettura | `READER` | Accesso in sola lettura configurabile per modulo |

### 10.4.2 Matrice permessi

| Azione | SUPER_ADMIN | ADMIN_ORG | EFM | EDR | RIR | MA | ES | RQ | OPERATORE |
|--------|:-----------:|:---------:|:---:|:---:|:---:|:--:|:--:|:--:|:---------:|
| Gestione utenti tenant | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Configurazione organizzazione | ✓ | ✓ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ | ✗ |
| Anagrafica apparecchiature (CRUD) | ✓ | ✓ | ✓ | ✓ | R | ✗ | ✗ | R | ✗ |
| Verifiche CQ (CRUD) | ✓ | ✓ | ✓ | ✗ | R | ✗ | ✗ | R | ✗ |
| Adempimenti EDR (CRUD) | ✓ | ✓ | ✗ | ✓ | R | ✗ | ✗ | R | ✗ |
| Benestare clinico | ✓ | ✓ | ✗ | ✗ | ✓ | ✗ | ✗ | ✗ | ✗ |
| Modulo RM (CRUD) | ✓ | ✓ | ✗ | ✓ | ✗ | ✓ | ✓ | R | ✗ |
| Registri MN (CRUD) | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | R | ✓* |
| Pazienti Lu177 (CRUD) | ✓ | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ | R | ✗ |
| Upload documenti | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ | ✓* |
| Reportistica | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ |
| Export dati | ✓ | ✓ | ✓ | ✓ | R | R | R | ✓ | ✗ |

> **Legenda:** ✓ = permesso completo, R = sola lettura, ✗ = nessun accesso, * = limitato a specifiche sezioni

### 10.4.3 Permessi granulari per OPERATORE

L'operatore può avere accesso limitato configurato dall'ADMIN_ORG:
- Inserimento letture dosimetri (Modulo 3)
- Inserimento letture sonde area monitor
- Upload file (solo allegati, non creazione record)

---

## 10.5 Autenticazione

### 10.5.1 Flusso di login

```
1. POST /auth/login { email, password }
       ↓
2. Verifica password hash (bcrypt)
       ↓
3. Verifica account attivo + email verificata
       ↓
4. Generazione JWT (access token, 1h)
   + Refresh token (httpOnly cookie, 30 giorni)
       ↓
5. Se utente ha più tenant → redirect a selezione tenant
   Altrimenti → redirect a dashboard tenant
       ↓
6. JWT include: { userId, tenantId, ruoli, exp }
```

### 10.5.2 Struttura JWT

```json
{
  "sub": "uuid-utente",
  "tenant_id": "uuid-tenant",
  "ruoli": ["EFM", "EDR"],
  "nome": "Mario",
  "cognome": "Rossi",
  "iat": 1704067200,
  "exp": 1704070800
}
```

### 10.5.3 Sicurezza

| Misura | Dettaglio |
|--------|-----------|
| Password policy | Min. 12 caratteri, almeno 1 maiuscola, 1 numero, 1 carattere speciale |
| Hashing | bcrypt con cost factor 12 |
| Rate limiting | Max 5 tentativi falliti in 15 minuti → blocco temporaneo 30 min |
| Sessioni | Access token 1h, refresh token 30 giorni (rotazione ad ogni rinnovo) |
| MFA | TOTP (Google Authenticator) — consigliato, configurabile come obbligatorio |
| HTTPS | Obbligatorio in produzione, HSTS abilitato |
| Log accessi | Ogni login (successo/fallimento) loggato con IP e user agent |

---

## 10.6 Gestione Utenti (interfaccia ADMIN_ORG)

### Funzionalità richieste

```
Gestione Utenti
├── Lista utenti dell'organizzazione (con filtri per ruolo, stato, reparto)
├── Invita nuovo utente (invia email con link di registrazione)
├── Modifica ruoli utente
├── Assegna reparto (per limitare visibilità)
├── Disattiva/Riattiva utente
├── Reset password (invia email di reset)
├── Storico accessi utente
└── Export lista utenti (CSV)
```

### Flusso invito nuovo utente

```
ADMIN_ORG inserisce email + ruolo(i)
       ↓
Sistema invia email con token di invito (scade in 48h)
       ↓
Utente clicca link → imposta password + profilo
       ↓
Account attivato con ruolo assegnato
```

---

## 10.7 Sicurezza Dati e Privacy

### Principi GDPR applicati

| Principio | Implementazione |
|-----------|----------------|
| **Minimizzazione** | Raccogliere solo i dati necessari agli adempimenti |
| **Limitazione finalità** | Dati usati solo per gestione adempimenti normativi |
| **Accesso limitato** | RBAC + Row Level Security per tenant |
| **Diritto all'oblio** | Procedura di anonimizzazione (non cancellazione, per audit trail) |
| **Pseudonimizzazione** | Pazienti Lu177 identificati con codice interno, non nome nei export statistici |
| **Log di accesso** | Ogni accesso a dati sensibili è loggato |

### Audit Log

```typescript
interface AuditLog {
  id: string;
  timestamp: Date;
  utente_id: string;
  tenant_id: string;
  azione: string;                      // Es. "CREATE", "UPDATE", "DELETE", "VIEW", "DOWNLOAD"
  risorsa_tipo: string;                // Es. "Apparecchiatura", "FileAllegato"
  risorsa_id: string;
  ip_address: string;
  user_agent: string;
  dati_precedenti?: JSON;              // Snapshot prima della modifica
  dati_nuovi?: JSON;                   // Snapshot dopo la modifica
}
```

L'audit log è **immutabile**: nessun record può essere cancellato o modificato.
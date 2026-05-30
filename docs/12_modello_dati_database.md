# 12 — Modello Dati e Schema Database

---

## 12.1 Panoramica

Il database è **PostgreSQL 15+** con schema relazionale normalizzato (3NF). L'isolamento multi-tenant è garantito da una colonna `tenant_id` presente in ogni tabella principale, rafforzata da **Row Level Security (RLS)** a livello di database.

---

## 12.2 Diagramma E/R — Entità Principali

```
organizations (tenant)
    │
    ├── users ──────────────────── tenant_memberships
    │                                      │
    ├── siti                           user_roles
    │    └── immobili
    │         └── piani
    │              └── locali
    │
    ├── reparti
    │
    ├── apparecchiature ◄──────────────────────────────────────────┐
    │    │                                                         │
    │    ├── parametri_radiologici                                 │
    │    ├── figure_responsabili                                   │
    │    ├── riferimenti_gestionali                                │
    │    ├── adempimenti_inail                                     │
    │    ├── adempimenti_strims                                    │
    │    │                                                         │
    │    ├── record_verifiche ──── protocolli_verifica             │
    │    │    └── file_allegati                                    │
    │    │                                                         │
    │    ├── notifiche_pratica                                     │
    │    ├── nulla_osta                                            │
    │    ├── benestare_utilizzo                                    │
    │    ├── verbali_sopralluogo                                   │
    │    └── file_allegati (generici)                              │
    │                                                              │
    ├── dosimetri ──── letture_dosimetri                           │
    ├── sonde ──── letture_sonde                                   │
    ├── strumentazione_radioprotezione                             │
    │                                                              │
    ├── radiofarmaci_inventario                                    │
    ├── sorgenti_inventario                                        │
    ├── registro_rifiuti                                           │
    ├── misure_contaminazione                                      │
    ├── controlli_ventilazione                                     │
    │                                                              │
    └── pazienti_lu177                                             │
         ├── cicli_trattamento ─── file_allegati                  │
         ├── acquisizioni_scintigrafiche ─── apparecchiature ─────┘
         ├── dosimetria_organi
         └── dati_ematologici
```

---

## 12.3 Schema SQL Dettagliato

### 12.3.1 Tabella: `organizations`

```sql
CREATE TABLE organizations (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    nome            VARCHAR(255) NOT NULL,
    tipo            VARCHAR(50) NOT NULL
                    CHECK (tipo IN ('ASL','AOU','IRCCS','PRIVATO','ALTRO')),
    codice_fiscale  VARCHAR(16),
    indirizzo       TEXT NOT NULL,
    email_amm       VARCHAR(255) NOT NULL,
    telefono        VARCHAR(30),
    logo_url        TEXT,
    fuso_orario     VARCHAR(50) DEFAULT 'Europe/Rome',
    lingua          VARCHAR(10) DEFAULT 'it',
    piano           VARCHAR(30) NOT NULL DEFAULT 'TRIAL'
                    CHECK (piano IN ('TRIAL','BASE','PROFESSIONAL','ENTERPRISE')),
    moduli_attivi   TEXT[]      NOT NULL DEFAULT '{}',
    scadenza_lic    DATE,
    attiva          BOOLEAN NOT NULL DEFAULT TRUE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### 12.3.2 Tabella: `users`

```sql
CREATE TABLE users (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email                       VARCHAR(255) NOT NULL UNIQUE,
    password_hash               TEXT NOT NULL,
    nome                        VARCHAR(100) NOT NULL,
    cognome                     VARCHAR(100) NOT NULL,
    telefono                    VARCHAR(30),
    avatar_url                  TEXT,
    attivo                      BOOLEAN NOT NULL DEFAULT TRUE,
    email_verificata            BOOLEAN NOT NULL DEFAULT FALSE,
    ultimo_accesso              TIMESTAMPTZ,
    password_cambiata_il        TIMESTAMPTZ,
    richiede_cambio_password    BOOLEAN NOT NULL DEFAULT FALSE,
    mfa_secret                  TEXT,           -- TOTP secret (cifrato a riposo)
    mfa_abilitato               BOOLEAN NOT NULL DEFAULT FALSE,
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at                  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### 12.3.3 Tabella: `tenant_memberships`

```sql
CREATE TABLE tenant_memberships (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    tenant_id       UUID NOT NULL REFERENCES organizations(id) ON DELETE CASCADE,
    ruoli           TEXT[] NOT NULL DEFAULT '{}',
    reparto_id      UUID,               -- FK → reparti (accesso limitato a reparto)
    attivo          BOOLEAN NOT NULL DEFAULT TRUE,
    data_inizio     DATE NOT NULL DEFAULT CURRENT_DATE,
    data_fine       DATE,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (user_id, tenant_id)
);
```

---

### 12.3.4 Tabelle Collocazione Geografica

```sql
CREATE TABLE siti (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id   UUID NOT NULL REFERENCES organizations(id),
    nome        VARCHAR(255) NOT NULL,
    indirizzo   TEXT,
    created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE immobili (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id   UUID NOT NULL REFERENCES organizations(id),
    sito_id     UUID NOT NULL REFERENCES siti(id),
    nome        VARCHAR(255) NOT NULL
);

CREATE TABLE piani (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL REFERENCES organizations(id),
    immobile_id     UUID NOT NULL REFERENCES immobili(id),
    nome            VARCHAR(100) NOT NULL,
    numero          SMALLINT
);

CREATE TABLE locali (
    id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id   UUID NOT NULL REFERENCES organizations(id),
    piano_id    UUID NOT NULL REFERENCES piani(id),
    nome        VARCHAR(255) NOT NULL,
    codice      VARCHAR(50)
);

CREATE TABLE reparti (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id       UUID NOT NULL REFERENCES organizations(id),
    nome            VARCHAR(255) NOT NULL,
    responsabile    VARCHAR(255),
    email           VARCHAR(255)
);
```

---

### 12.3.5 Tabella: `apparecchiature` (centrale)

```sql
CREATE TABLE apparecchiature (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id               UUID NOT NULL REFERENCES organizations(id),
    codice                  VARCHAR(100) NOT NULL,
    descrizione             VARCHAR(255) NOT NULL,
    modulo                  VARCHAR(20) NOT NULL
                            CHECK (modulo IN ('RADIOLOGICA','RM')),
    ambito_intervento       VARCHAR(50)
                            CHECK (ambito_intervento IN (
                                'RADIOLOGIA',
                                'RADIOLOGIA_INTERVENTISTICA',
                                'MEDICINA_NUCLEARE',
                                'RADIOTERAPIA'
                            )),
    tipologia               VARCHAR(100) NOT NULL,

    -- Dati tecnici identificativi
    modello                 VARCHAR(255) NOT NULL,
    costruttore             VARCHAR(255) NOT NULL,
    matricola               VARCHAR(100) NOT NULL,
    serial_number           VARCHAR(100),

    -- Collocazione
    locale_id               UUID REFERENCES locali(id),
    reparto_id              UUID REFERENCES reparti(id),
    caposala                VARCHAR(255),

    -- Ciclo di vita
    stato                   VARCHAR(30) NOT NULL DEFAULT 'IN_INSTALLAZIONE'
                            CHECK (stato IN (
                                'IN_INSTALLAZIONE','ATTIVA',
                                'IN_MANUTENZIONE','CESSATA'
                            )),
    data_accettazione       DATE,
    data_cessazione         DATE,

    -- Flags tecnici
    lan_collegata           BOOLEAN NOT NULL DEFAULT FALSE,
    medsquare_installato    BOOLEAN NOT NULL DEFAULT FALSE,

    -- Bene aziendale
    sap_id                  VARCHAR(100),
    siap_descrizione        TEXT,

    -- Timestamps e autore
    created_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by              UUID NOT NULL REFERENCES users(id),

    UNIQUE (tenant_id, codice),

    CONSTRAINT ck_ambito_per_modulo CHECK (
        (modulo = 'RADIOLOGICA' AND ambito_intervento IS NOT NULL) OR
        (modulo = 'RM')
    )
);

-- Indici principali
CREATE INDEX idx_app_tenant     ON apparecchiature(tenant_id);
CREATE INDEX idx_app_stato      ON apparecchiature(tenant_id, stato);
CREATE INDEX idx_app_ambito     ON apparecchiature(tenant_id, ambito_intervento);
CREATE INDEX idx_app_reparto    ON apparecchiature(reparto_id);
```

---

### 12.3.6 Tabella: `parametri_radiologici`

```sql
CREATE TABLE parametri_radiologici (
    apparecchiatura_id      UUID PRIMARY KEY REFERENCES apparecchiature(id),
    tenant_id               UUID NOT NULL REFERENCES organizations(id),
    corrente_max_ma         NUMERIC(10,2),
    tensione_max_kvolt      NUMERIC(10,2),
    energia_max_kev         NUMERIC(10,2),
    intensita_campo_tesla   NUMERIC(5,2),          -- Solo RM
    tipo_magnete            VARCHAR(30),            -- Solo RM
    campo_max_fringe_mt     NUMERIC(10,4),          -- Solo RM
    updated_at              TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### 12.3.7 Tabella: `figure_responsabili`

```sql
CREATE TABLE figure_responsabili (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id               UUID NOT NULL REFERENCES organizations(id),
    apparecchiatura_id      UUID NOT NULL REFERENCES apparecchiature(id),
    ruolo                   VARCHAR(10) NOT NULL
                            CHECK (ruolo IN ('RIR','EFM','EdR','MA','ES','MR')),
    nome                    VARCHAR(100) NOT NULL,
    cognome                 VARCHAR(100) NOT NULL,
    email                   VARCHAR(255),
    telefono                VARCHAR(30),
    grado_abilitazione      CHAR(1)
                            CHECK (grado_abilitazione IN ('1','2','3')),
    valido_dal              DATE NOT NULL DEFAULT CURRENT_DATE,
    valido_al               DATE,
    created_at              TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE INDEX idx_fig_resp_app ON figure_responsabili(apparecchiatura_id, ruolo);
```

---

### 12.3.8 Tabella: `file_allegati`

```sql
CREATE TABLE file_allegati (
    id                      UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id               UUID NOT NULL REFERENCES organizations(id),
    nome_originale          VARCHAR(500) NOT NULL,
    nome_storage            TEXT NOT NULL,         -- Chiave nel bucket S3
    mime_type               VARCHAR(100) NOT NULL,
    dimensione_bytes        BIGINT NOT NULL,
    categoria               VARCHAR(60) NOT NULL,
    -- Relazioni polimorfiche (solo una valorizzata)
    apparecchiatura_id      UUID REFERENCES apparecchiature(id),
    verifica_id             UUID,                  -- FK → record_verifiche
    verbale_id              UUID,                  -- FK → verbali_sopralluogo
    paziente_id             UUID,                  -- FK → pazienti_lu177
    ciclo_id                UUID,                  -- FK → cicli_trattamento
    -- Versioning
    versione                SMALLINT NOT NULL DEFAULT 1,
    sostituisce_file_id     UUID REFERENCES file_allegati(id),
    descrizione             TEXT,
    -- Audit
    uploaded_at             TIMESTAMPTZ NOT NULL DEFAULT now(),
    uploaded_by             UUID NOT NULL REFERENCES users(id)
);

CREATE INDEX idx_file_app      ON file_allegati(apparecchiatura_id);
CREATE INDEX idx_file_verifica ON file_allegati(verifica_id);
CREATE INDEX idx_file_tenant   ON file_allegati(tenant_id);
```

---

### 12.3.9 Tabella: `protocolli_verifica`

```sql
CREATE TABLE protocolli_verifica (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id                   UUID NOT NULL REFERENCES organizations(id),
    codice                      VARCHAR(50) NOT NULL,
    descrizione                 TEXT NOT NULL,
    tipo                        VARCHAR(40) NOT NULL
                                CHECK (tipo IN (
                                    'ACCETTAZIONE',
                                    'PERIODICO',
                                    'POST_MANUTENZIONE',
                                    'LDR',
                                    'PRIMA_VERIFICA_EDR',
                                    'SORVEGLIANZA_PERIODICA_EDR'
                                )),
    ambiti_applicabilita        TEXT[] NOT NULL DEFAULT '{}',
    tipologie_applicabilita     TEXT[] NOT NULL DEFAULT '{}',
    revisione                   VARCHAR(20) NOT NULL,
    data_entrata_vigore         DATE NOT NULL,
    periodicita_mesi            SMALLINT,          -- NULL per non periodici
    documento_id                UUID REFERENCES file_allegati(id),
    created_by                  UUID NOT NULL REFERENCES users(id),
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    UNIQUE (tenant_id, codice, revisione)
);
```

---

### 12.3.10 Tabella: `record_verifiche`

```sql
CREATE TABLE record_verifiche (
    id                              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id                       UUID NOT NULL REFERENCES organizations(id),
    apparecchiatura_id              UUID NOT NULL REFERENCES apparecchiature(id),
    protocollo_id                   UUID NOT NULL REFERENCES protocolli_verifica(id),
    tipo                            VARCHAR(40) NOT NULL,
    data_inizio                     DATE NOT NULL,
    data_fine                       DATE,
    eseguito_da                     UUID NOT NULL REFERENCES users(id),
    esito                           VARCHAR(20) NOT NULL DEFAULT 'IN_CORSO'
                                    CHECK (esito IN (
                                        'IN_CORSO','SUPERATO',
                                        'NON_SUPERATO','CON_RISERVA'
                                    )),
    note                            TEXT,
    -- Benestare (per accettazione)
    ben_qualita_tecnica_data        DATE,
    ben_qualita_tecnica_by          UUID REFERENCES users(id),
    ben_clinico_data                DATE,
    ben_clinico_by                  UUID REFERENCES users(id),
    -- Periodicità
    anno                            SMALLINT,
    semestre                        VARCHAR(10),       -- Es. "1S-2024" (RM)
    prossima_verifica_data          DATE,
    -- Guasto
    info_guasto                     TEXT,
    tipo_guasto                     VARCHAR(30),
    -- Manutenzione
    tipo_intervento_manut           TEXT,
    tecnico_manutentore             VARCHAR(255),
    societa_manutenzione            VARCHAR(255),
    data_intervento_manut           DATE,
    rapporto_tecnico_manut_id       UUID REFERENCES file_allegati(id),
    -- Timestamps
    created_at                      TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by                      UUID NOT NULL REFERENCES users(id)
);

CREATE INDEX idx_verifica_app  ON record_verifiche(apparecchiatura_id);
CREATE INDEX idx_verifica_tipo ON record_verifiche(tenant_id, tipo);
CREATE INDEX idx_verifica_anno ON record_verifiche(tenant_id, anno);
```

---

### 12.3.11 Tabella: `adempimenti_strims`

```sql
CREATE TABLE adempimenti_strims (
    apparecchiatura_id          UUID PRIMARY KEY REFERENCES apparecchiature(id),
    tenant_id                   UUID NOT NULL REFERENCES organizations(id),
    strims_id_app               VARCHAR(100),
    stato                       VARCHAR(30) NOT NULL DEFAULT 'DA_REGISTRARE'
                                CHECK (stato IN (
                                    'DA_REGISTRARE','REGISTRATO','CESSATO'
                                )),
    data_registrazione          DATE,
    ricevuta_file_id            UUID REFERENCES file_allegati(id),
    np_caricata                 BOOLEAN NOT NULL DEFAULT FALSE,
    data_np                     DATE,
    nc_caricata                 BOOLEAN NOT NULL DEFAULT FALSE,
    data_nc                     DATE,
    updated_at                  TIMESTAMPTZ NOT NULL DEFAULT now()
);
```

---

### 12.3.12 Tabella: `pazienti_lu177`

```sql
CREATE TABLE pazienti_lu177 (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id                   UUID NOT NULL REFERENCES organizations(id),
    codice_paziente             VARCHAR(50) NOT NULL,
    cognome                     VARCHAR(100) NOT NULL,
    nome                        VARCHAR(100) NOT NULL,
    data_nascita                DATE NOT NULL,
    sesso                       CHAR(1) NOT NULL CHECK (sesso IN ('M','F')),
    codice_fiscale              VARCHAR(16),
    numero_nosologico           VARCHAR(100),
    medico_inviante             VARCHAR(255) NOT NULL,
    reparto_inviante            VARCHAR(255),
    diagnosi_principale         TEXT NOT NULL,
    indicazione                 VARCHAR(30) NOT NULL
                                CHECK (indicazione IN (
                                    'PSMA_LU177','DOTATATE_LU177','ALTRO'
                                )),
    data_prima_visita           DATE,
    stato_paziente              VARCHAR(20) NOT NULL DEFAULT 'IN_TRATTAMENTO'
                                CHECK (stato_paziente IN (
                                    'IN_TRATTAMENTO','CONCLUSO',
                                    'DECEDUTO','PERSO_FU'
                                )),
    -- Dati clinici
    peso_kg                     NUMERIC(5,1),
    altezza_cm                  NUMERIC(5,1),
    egfr_ml_min                 NUMERIC(6,1),
    -- Piano terapeutico
    protocollo_terapeutico      VARCHAR(100),
    n_cicli_pianificati         SMALLINT,
    attivita_per_ciclo_gbq      NUMERIC(5,2),
    intervallo_settimane        SMALLINT,
    data_inizio_trattamento     DATE,
    -- Timestamps
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by                  UUID NOT NULL REFERENCES users(id),
    UNIQUE (tenant_id, codice_paziente)
);

CREATE INDEX idx_paz_tenant    ON pazienti_lu177(tenant_id);
CREATE INDEX idx_paz_stato     ON pazienti_lu177(tenant_id, stato_paziente);
```

---

### 12.3.13 Tabella: `cicli_trattamento`

```sql
CREATE TABLE cicli_trattamento (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id                   UUID NOT NULL REFERENCES organizations(id),
    paziente_id                 UUID NOT NULL REFERENCES pazienti_lu177(id),
    numero_ciclo                SMALLINT NOT NULL,
    data_somministrazione       DATE NOT NULL,
    ora_somministrazione        TIME NOT NULL,
    attivita_somministrata_gbq  NUMERIC(6,3) NOT NULL,
    attivita_residua_sir_gbq    NUMERIC(6,3) NOT NULL,
    attivita_effettiva_gbq      NUMERIC(6,3) GENERATED ALWAYS AS
                                (attivita_somministrata_gbq - attivita_residua_sir_gbq) STORED,
    lotto_radiofarmaco          VARCHAR(100) NOT NULL,
    fornitore_radiofarmaco      VARCHAR(255) NOT NULL,
    peso_paziente_ciclo_kg      NUMERIC(5,1) NOT NULL,
    medico_prescrivente         UUID NOT NULL REFERENCES users(id),
    fisico_medico               UUID NOT NULL REFERENCES users(id),
    infermiere_somm             VARCHAR(255),
    note_cliniche               TEXT,
    effetti_avversi             TEXT,
    esito_ciclo                 VARCHAR(20) NOT NULL DEFAULT 'COMPLETATO'
                                CHECK (esito_ciclo IN (
                                    'COMPLETATO','RIDOTTO',
                                    'SOSPESO','INTERROTTO'
                                )),
    motivo_modifica             TEXT,
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by                  UUID NOT NULL REFERENCES users(id),
    UNIQUE (paziente_id, numero_ciclo)
);
```

---

### 12.3.14 Tabella: `dati_ematologici`

```sql
CREATE TABLE dati_ematologici (
    id                          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    tenant_id                   UUID NOT NULL REFERENCES organizations(id),
    paziente_id                 UUID NOT NULL REFERENCES pazienti_lu177(id),
    ciclo_id                    UUID REFERENCES cicli_trattamento(id),
    data_prelievo               DATE NOT NULL,
    timing                      VARCHAR(30) NOT NULL,
    wbc_x10_9_l                 NUMERIC(6,2),
    hgb_g_dl                    NUMERIC(5,2),
    plt_x10_9_l                 NUMERIC(7,1),
    neutrofili_x10_9_l          NUMERIC(6,2),
    creatinina_mg_dl            NUMERIC(5,2),
    egfr_ml_min                 NUMERIC(6,1),
    psa_ng_ml                   NUMERIC(10,3),
    cromogranina_a              NUMERIC(10,2),
    tossicita_ematologica       CHAR(2)
                                CHECK (tossicita_ematologica IN
                                    ('G0','G1','G2','G3','G4')),
    note                        TEXT,
    created_at                  TIMESTAMPTZ NOT NULL DEFAULT now(),
    created_by                  UUID NOT NULL REFERENCES users(id)
);
```

---

### 12.3.15 Tabella: `audit_logs`

```sql
CREATE TABLE audit_logs (
    id              BIGSERIAL PRIMARY KEY,         -- Non UUID per performance su insert massivi
    timestamp       TIMESTAMPTZ NOT NULL DEFAULT now(),
    user_id         UUID NOT NULL,                 -- Non FK: il log sopravvive alla cancellazione utente
    tenant_id       UUID NOT NULL,
    azione          VARCHAR(20) NOT NULL
                    CHECK (azione IN (
                        'CREATE','READ','UPDATE',
                        'DELETE','LOGIN','LOGOUT',
                        'DOWNLOAD','EXPORT','LOGIN_FAIL'
                    )),
    risorsa_tipo    VARCHAR(100) NOT NULL,
    risorsa_id      TEXT,
    ip_address      INET,
    user_agent      TEXT,
    dati_precedenti JSONB,
    dati_nuovi      JSONB
);

CREATE INDEX idx_audit_tenant    ON audit_logs(tenant_id, timestamp DESC);
CREATE INDEX idx_audit_user      ON audit_logs(user_id, timestamp DESC);
CREATE INDEX idx_audit_risorsa   ON audit_logs(risorsa_tipo, risorsa_id);

-- Tabella append-only: nessun UPDATE o DELETE permesso
-- (da garantire con trigger o policy specifiche)
```

---

## 12.4 Row Level Security (RLS)

```sql
-- Abilitare RLS su ogni tabella con tenant_id
ALTER TABLE apparecchiature ENABLE ROW LEVEL SECURITY;

-- Policy: ogni utente vede solo i dati del proprio tenant (passato via session variable)
CREATE POLICY tenant_isolation ON apparecchiature
    USING (tenant_id = current_setting('app.current_tenant')::UUID);

-- La session variable viene impostata dal backend dopo l'autenticazione:
-- SET LOCAL app.current_tenant = '<tenant_uuid>';
```

---

## 12.5 Migrazioni e Seed

### Struttura directory migrazioni

```
database/
├── migrations/
│   ├── V001__create_organizations.sql
│   ├── V002__create_users_and_memberships.sql
│   ├── V003__create_location_tables.sql
│   ├── V004__create_apparecchiature.sql
│   ├── V005__create_parametri_and_figure.sql
│   ├── V006__create_file_allegati.sql
│   ├── V007__create_protocolli_and_verifiche.sql
│   ├── V008__create_adempimenti.sql
│   ├── V009__create_medicina_nucleare.sql
│   ├── V010__create_lu177.sql
│   ├── V011__create_audit_logs.sql
│   └── V012__create_rls_policies.sql
│
└── seeds/
    ├── 001_tipologie_apparecchiature.sql   -- Lookup table tipologie
    ├── 002_categorie_file.sql
    ├── 003_radionuclidi.sql               -- Lista radionuclidi comuni con t½
    └── 004_demo_organization.sql          -- Dati demo per sviluppo
```

### Strumento di migrazione consigliato

**Flyway** (Java, integrabile in CI/CD) o **Prisma Migrate** (se si usa l'ORM Prisma).

---

## 12.6 Note di Performance

| Ottimizzazione | Descrizione |
|----------------|-------------|
| Indici tenant_id | Ogni tabella principale ha indice su `(tenant_id, ...)` |
| Partial index | Es. `WHERE stato = 'ATTIVA'` per query su apparecchiature attive |
| Colonne JSONB | Usate solo per dati strutturati variabili (audit log), non per dati query-abili |
| Paginazione | Tutte le list API usano cursor-based pagination (no OFFSET su tabelle grandi) |
| Connection pool | PgBouncer in modalità transaction per gestire molte connessioni concorrenti |
| Read replica | Per query di reportistica pesante, indirizzare su replica in lettura |
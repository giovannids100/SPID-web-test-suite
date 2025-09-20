# Elaborato finale di Tesi Ing. Informatica
# Autenticazione d'Accesso a Servizi Web Tramite SPID: Protocolli e Ambiente di Verifica

Questo progetto documenta l’utilizzo dei protocolli alla base di **SPID** (Sistema Pubblico di Identità Digitale) e gli strumenti di testing messi a disposizione da **AgID**.  
L’obiettivo è mostrare come configurare e validare un **Service Provider (SP)** attraverso i tool ufficiali.

---

## Tecnologie e Standard Utilizzati

- **SAML 2.0** – Security Assertion Markup Language  
  - Standard aperto per autenticazione e autorizzazione.  
  - Basato sullo scambio di pacchetti XML (*Assertions*) tra **Identity Provider (IdP)** e **Service Provider (SP)**.  
  - Supporta diversi metodi di trasporto (*binding*): HTTP Redirect, HTTP POST, HTTP Artifact.  
  - Fondamentale per il **Single Sign-On (SSO)**.

- **AgID e Metadata**  
  - Ogni SP deve essere descritto da un file **metadata.xml** contenente:  
    - `entityID`, `KeyDescriptor` (certificati), `AssertionConsumerService`, `AttributeConsumingService`, ecc.  
  - L’AgID fornisce linee guida per garantire sicurezza e interoperabilità.

---

## Ambiente di Verifica SPID

AgID fornisce tre strumenti principali per il testing:

1. **SPID Validator**  
   - Controlla la validità dei metadata e delle SAML request/response.  
   - Esegue oltre 300 test automatici.  
   - Disponibile sia via web che tramite container.

2. **Ambiente Demo SPID**  
   - Simula un IdP SPID con credenziali fittizie.  
   - Permette di verificare casi d’uso come autenticazione, errori di livello, timeout, consenso negato, identità sospesa.

3. **SPID SAML Check**  
   - Suite completa (Validator + Demo + SP Test).  
   - Disponibile su [GitHub](https://github.com/italia/spid-saml-check).  
   - Eseguibile in container per test più avanzati.

---

## Implementazione e Testing

### 1. Setup con Docker

Clonare il repository ufficiale:

```bash
git clone https://github.com/italia/spid-saml-check.git
cd spid-saml-check
docker build -t spid-saml-check .
```

Oppure scaricare l’immagine da DockerHub:

```bash
docker pull italia/spid-saml-check:latest
```

Avviare il container sulla porta `8443`:

```bash
docker run -t -i -p 8443:8443 spid-saml-check
```

---

### 2. Generazione del Metadata

Scaricare il metadata generato dall’istanza locale del Validator:

```bash
wget https://localhost:8443/metadata.xml -O ./metadata_demo.xml
```

Personalizzare il file con i dati del proprio SP (es. nome organizzazione, contatti, certificati).  
Esempio di sezione modificata:

```xml
<md:Organization>
  <md:OrganizationName xml:lang="it">SPID Identity Provider</md:OrganizationName>
  <md:OrganizationDisplayName xml:lang="it">SPID Identity Provider</md:OrganizationDisplayName>
  <md:OrganizationURL xml:lang="it">https://spid.identityprovider.it</md:OrganizationURL>
</md:Organization>
```

---

### 3. Validazione del Metadata

Accedere a **SPID Validator** in locale (`https://localhost:8443`), autenticarsi con:

```
username: validator
password: validator
```

Inserire l’URL del proprio `metadata.xml` e avviare i test:  
- **Check Strict** → validazione formale.  
- **Check Extra** → controlli aggiuntivi.  

> Se il file non è conforme alle regole AgID, i test falliranno (comportamento previsto per configurazioni dimostrative).

---

## Quick Start

Per replicare subito l’ambiente di test:

```bash
# 1. Scarica l'immagine Docker
docker pull italia/spid-saml-check:latest

# 2. Avvia il container
docker run -t -i -p 8443:8443 spid-saml-check

# 3. Scarica il metadata
wget https://localhost:8443/metadata.xml -O ./metadata_demo.xml

# 4. Apri il browser su https://localhost:8443
#    Login: validator / validator
#    Carica il metadata e avvia i test
```

---

## Risultati Attesi

- Un Service Provider deve avere metadata validi e firmati con certificati riconosciuti.  
- In un’implementazione reale, solo i metadata conformi e approvati da AgID possono essere utilizzati in produzione.  
- L’ambiente di test permette di simulare accessi e verificare la corretta gestione delle sessioni SPID.  

---

## Riferimenti

- [SPID – Procedura tecnica](https://www.spid.gov.it/cos-e-spid/diventa-fornitore-di-servizi/procedura-tecnica)  
- [Regole tecniche SPID (Docs Italia)](https://docs.italia.it/italia/spid/spid-regole-tecniche/it/stabile)  
- [SPID SAML Check – GitHub](https://github.com/italia/spid-saml-check)  
- [Ambiente Demo SPID](https://demo.spid.gov.it)  
- [Docker – opensource.com](https://opensource.com/resources/what-docker)  

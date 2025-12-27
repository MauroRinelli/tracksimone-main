# Automazione Email con n8n - Edicola Calcio Point

Questa guida spiega come configurare l'automazione per l'invio di email multiple tramite n8n e la webmail Siteground.

## 📋 Panoramica

Il sistema permette di:
- Inviare email multiple a destinatari selezionati
- Personalizzare il mittente e il messaggio
- Validare automaticamente i dati prima dell'invio
- Gestire errori e risposte

## 🔧 Prerequisiti

1. **n8n installato** - Puoi usare:
   - n8n Cloud (https://n8n.io)
   - n8n self-hosted (Docker, npm, ecc.)

2. **Credenziali SMTP Siteground** per simone@solespress.it:
   - Server SMTP: `smtp.siteground.com` (oppure verificare nelle impostazioni email Siteground)
   - Porta: `465` (SSL) o `587` (TLS)
   - Username: `simone@solespress.it`
   - Password: la password dell'account email

## 📦 Installazione

### Passo 1: Importare il Workflow in n8n

1. Accedi alla tua istanza n8n
2. Clicca su **"+ Add workflow"** o **"Import from file"**
3. Carica il file `n8n-email-automation-workflow.json`
4. Il workflow verrà importato con tutti i nodi configurati

### Passo 2: Configurare le Credenziali SMTP

1. Nel workflow, clicca sul nodo **"Invia Email (SMTP)"**
2. Nelle impostazioni, clicca su **"Credentials"**
3. Clicca su **"Create New"** per creare nuove credenziali SMTP
4. Compila i campi:
   - **Name**: `Siteground SMTP - simone@solespress.it`
   - **User**: `simone@solespress.it`
   - **Password**: [la tua password email]
   - **Host**: `smtp.siteground.com` (verifica con il supporto Siteground se necessario)
   - **Port**: `465` (per SSL) o `587` (per TLS/STARTTLS)
   - **Security**: Seleziona `SSL/TLS` se usi porta 465, `STARTTLS` se usi porta 587
5. Clicca su **"Save"**

### Passo 3: Attivare il Webhook

1. Nel workflow, clicca sul nodo **"Webhook - Form Trigger"**
2. In alto a destra, clicca su **"Active"** per attivare il workflow
3. Una volta attivato, vedrai l'URL del webhook (esempio: `https://your-n8n-instance.com/webhook/form-email-sender`)
4. **Copia questo URL** - ti servirà per il passo successivo

### Passo 4: Configurare il Form HTML

1. Apri il file `email-sender.html`
2. Trova la riga 524 che contiene:
   ```javascript
   const N8N_WEBHOOK_URL = 'YOUR_N8N_WEBHOOK_URL_HERE';
   ```
3. Sostituisci `YOUR_N8N_WEBHOOK_URL_HERE` con l'URL del webhook che hai copiato:
   ```javascript
   const N8N_WEBHOOK_URL = 'https://your-n8n-instance.com/webhook/form-email-sender';
   ```
4. Salva il file

## 🎯 Come Funziona il Workflow

### Struttura del Workflow

```
Webhook (Trigger)
    ↓
Validazione Dati
    ↓
IF Valido? ──┐
    │        │
   Sì        No
    │        │
    ↓        ↓
Elaborazione  Response Error
    ↓
Invia Email (SMTP)
    ↓
Response OK
```

### Dettagli dei Nodi

1. **Webhook - Form Trigger**
   - Riceve i dati POST dal form HTML
   - Path: `/webhook/form-email-sender`
   - Method: POST
   - Formato dati atteso:
     ```json
     {
       "senderName": "Edicola Calcio Point",
       "message": "Il tuo messaggio qui...",
       "recipients": [
         { "name": "Mario Rossi", "email": "mario@example.com" },
         { "name": "Luigi Verdi", "email": "luigi@example.com" }
       ]
     }
     ```

2. **Validazione Dati**
   - Verifica che tutti i campi richiesti siano presenti
   - Valida il formato delle email
   - Controlla che ci sia almeno un destinatario
   - Output: `{ valid: true/false, error?: string, ... }`

3. **IF Valido**
   - Controlla il campo `valid` dalla validazione
   - Se `true`: procede con l'elaborazione
   - Se `false`: risponde con errore

4. **Elaborazione**
   - Crea un item separato per ogni destinatario
   - Prepara i dati per l'invio email
   - Genera l'oggetto dell'email: `"Tracking Spedizione - {senderName}"`

5. **Invia Email (SMTP)**
   - Invia un'email per ogni destinatario
   - Mittente: `simone@solespress.it`
   - Nome mittente: il valore di `senderName`
   - Destinatario: l'email del recipient
   - Corpo: il messaggio inserito nel form

6. **Response OK / Response Error**
   - Risponde al webhook con successo o errore
   - Formato risposta successo:
     ```json
     {
       "success": true,
       "message": "Email inviate con successo",
       "sent": 5
     }
     ```
   - Formato risposta errore:
     ```json
     {
       "success": false,
       "error": "Descrizione errore"
     }
     ```

## 🧪 Test del Workflow

### Test Manuale in n8n

1. Nel workflow, clicca su **"Execute Workflow"**
2. Nel nodo Webhook, clicca su **"Listen for Test Event"**
3. Apri il form `email-sender.html` nel browser
4. Compila i campi e invia
5. Torna su n8n e verifica che il workflow sia stato eseguito
6. Controlla ogni nodo per vedere i dati elaborati

### Test con cURL

Puoi testare il webhook direttamente con cURL:

```bash
curl -X POST https://your-n8n-instance.com/webhook/form-email-sender \
  -H "Content-Type: application/json" \
  -d '{
    "senderName": "Test Sender",
    "message": "Questo è un messaggio di test",
    "recipients": [
      {
        "name": "Test Recipient",
        "email": "test@example.com"
      }
    ]
  }'
```

## ⚙️ Configurazioni Aggiuntive

### Modificare il Mittente

Se vuoi cambiare l'email del mittente:
1. Apri il nodo **"Invia Email (SMTP)"**
2. Modifica il campo **"From Email"**
3. Assicurati di avere le credenziali SMTP per quel mittente

### Personalizzare l'Oggetto Email

Per modificare l'oggetto delle email:
1. Apri il nodo **"Elaborazione"**
2. Modifica la riga:
   ```javascript
   subject: `Tracking Spedizione - ${senderName}`
   ```
3. Cambiala come preferisci, ad esempio:
   ```javascript
   subject: `Notifica da ${senderName}`
   ```

### Aggiungere Allegati

Per aggiungere allegati alle email:
1. Nel nodo **"Invia Email (SMTP)"**
2. Espandi la sezione **"Options"**
3. Abilita **"Attachments"**
4. Configura gli allegati (puoi usare dati binari da nodi precedenti)

## 🔒 Sicurezza

### Best Practices

1. **NON committare credenziali**
   - Le credenziali SMTP sono salvate solo in n8n
   - Non includere mai password nei file del repository

2. **Validazione dei dati**
   - Il workflow valida automaticamente tutti i dati in ingresso
   - Non permette l'invio senza validazione

3. **Rate Limiting**
   - Verifica i limiti di invio email del tuo piano Siteground
   - Aggiungi un nodo di delay se necessario per rispettare i limiti

4. **HTTPS**
   - Usa sempre HTTPS per il webhook in produzione
   - Questo protegge i dati in transito

### Limitazioni Siteground

Controlla i limiti del tuo piano Siteground:
- Numero massimo di email/ora
- Numero massimo di email/giorno
- Dimensione massima allegati

## 🐛 Troubleshooting

### Email non vengono inviate

1. **Verifica credenziali SMTP**
   - Controlla username, password, server e porta
   - Verifica che l'account email sia attivo su Siteground

2. **Controlla i log n8n**
   - Apri il workflow in n8n
   - Clicca su "Executions" per vedere i log di esecuzione
   - Controlla gli errori nei nodi

3. **Test connessione SMTP**
   - In n8n, nel nodo Email Send, c'è un pulsante "Test"
   - Usalo per verificare la connessione SMTP

### Webhook non risponde

1. **Workflow attivo?**
   - Verifica che il workflow sia attivo (interruttore in alto a destra)

2. **URL corretto?**
   - Controlla che l'URL nel form corrisponda all'URL del webhook

3. **CORS issues?**
   - Se usi n8n cloud, i CORS dovrebbero essere gestiti automaticamente
   - Se self-hosted, verifica le configurazioni CORS di n8n

### Errori di validazione

Se ricevi sempre errori di validazione:
1. Apri la console browser (F12)
2. Verifica i dati inviati nella tab "Network"
3. Controlla che il formato JSON sia corretto
4. Verifica che tutti i campi richiesti siano presenti

## 📚 Risorse

- [Documentazione n8n](https://docs.n8n.io)
- [n8n SMTP Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.emailsend/)
- [Siteground Email Setup](https://www.siteground.com/kb/email-settings/)

## 📞 Supporto

Per problemi con:
- **n8n**: Controlla la [community n8n](https://community.n8n.io)
- **Siteground SMTP**: Contatta il supporto Siteground
- **Questo workflow**: Verifica i log di esecuzione in n8n

---

**Versione**: 1.0
**Ultimo aggiornamento**: 2025-01-15

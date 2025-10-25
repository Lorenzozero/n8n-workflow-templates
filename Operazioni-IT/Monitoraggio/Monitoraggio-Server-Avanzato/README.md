# Monitoraggio Server Avanzato

## Descrizione
Workflow avanzato per il monitoraggio proattivo di server e infrastrutture con alert intelligenti multi-canale. Integra controlli di uptime, analisi delle risorse di sistema (CPU, RAM, Disco) e escalation automatica tramite Slack, Email e PagerDuty.

## Flusso di Lavoro

### 1. Trigger Schedulato
- **Frequenza**: Ogni 5 minuti (configurabile)
- **Modalità**: Cron job automatico
- **Scopo**: Monitoraggio continuo dell'infrastruttura

### 2. Controllo Uptime
- **Metodo**: HTTP Request al endpoint `/health`
- **Timeout**: 30 secondi
- **Verifica**: Status code 200 per conferma operatività
- **Fallback**: Alert immediato in caso di mancata risposta

### 3. Analisi Risorse Sistema
- **SSH Connection**: Accesso diretto al server per metriche real-time
- **Comandi**: `df -h`, `free -m`, `top -bn1`
- **Parsing**: Estrazione automatica di CPU%, Memory%, Disk%
- **Soglie**: Configurabili per ogni tipologia di risorsa

### 4. Sistema Alert Intelligente
- **Livello 1**: Slack notification per team IT
- **Livello 2**: Email alert per responsabili
- **Livello 3**: PagerDuty escalation per emergenze
- **Personalizzazione**: Messaggi contestualizzati con metriche dettagliate

## Istruzioni di Setup

### Prerequisiti
1. **Credenziali n8n configurate**:
   - Slack API Token (scope: `chat:write`)
   - Gmail OAuth2 o SMTP credentials
   - PagerDuty Integration Key
   - SSH access al server target

2. **Endpoint server**:
   - Health check endpoint attivo (es. `/health`, `/status`)
   - SSH abilitato per monitoring user
   - Firewall configurato per connessioni n8n

### Configurazione Step-by-Step

1. **Import workflow**: Carica il file JSON in n8n
2. **Configura credenziali**:
   ```bash
   # Slack API
   - Token: xoxb-your-slack-token
   - Channel: #alerts-it
   
   # Gmail
   - OAuth2 setup completo
   - Email destinatario: sysadmin@azienda.it
   
   # SSH
   - Host: server.azienda.it
   - Username: monitoring
   - SSH Key o Password
   
   # PagerDuty
   - Integration Key: your-routing-key
   - Service: Critical Infrastructure
   ```

3. **Personalizza configurazione**:
   - Modifica nodo "Lista Server" con i tuoi endpoint
   - Aggiorna soglie nel nodo "Configurazione"
   - Personalizza messaggi di alert

4. **Test del workflow**:
   - Esegui manualmente per verificare connessioni
   - Simula condizioni di alert modificando temporaneamente le soglie
   - Controlla ricezione notifiche su tutti i canali

5. **Attivazione**:
   - Abilita il trigger Cron
   - Monitora logs per le prime 24h
   - Ottimizza soglie basandosi sui pattern reali

## Funzionalità Chiave

### 🔍 Monitoring Proattivo
- Controllo uptime continuo ogni 5 minuti
- Analisi real-time delle risorse di sistema
- Rilevamento precoce di anomalie e degradazioni performance

### 📊 Metriche Avanzate
- **CPU Usage**: Percentuale utilizzo processore
- **Memory Usage**: RAM occupata vs disponibile
- **Disk Usage**: Spazio disco utilizzato per partizione
- **Response Time**: Latenza endpoint health check

### 🚨 Alert Intelligenti
- **Escalation automatica**: Da Slack → Email → PagerDuty
- **Messaggi contestualizzati**: Include metriche specifiche e timestamp
- **Throttling**: Evita spam di notifiche per stesso problema
- **Rich formatting**: HTML email e Slack markup per migliore leggibilità

### ⚙️ Configurabilità
- Soglie personalizzabili per ogni metrica
- Server multipli gestiti tramite array/loop
- Intervalli di monitoraggio adattabili
- Canali di notifica modulari

## Requisiti Sistema

### Server Target
- **OS**: Linux/Unix con supporto SSH
- **Comandi**: `df`, `free`, `top` disponibili
- **Network**: Accesso HTTP/HTTPS per health check
- **Security**: SSH key-based auth raccomandato

### n8n Instance
- **Versione**: n8n 1.0+ 
- **Nodi richiesti**: HTTP Request, SSH, Code, Gmail, Cron
- **Memoria**: Minimo 512MB per processing metriche
- **Network**: Accesso outbound verso server e servizi esterni

### Servizi Esterni
- **Slack Workspace** con app installata
- **Gmail account** o server SMTP
- **PagerDuty account** con service configurato
- **Server SSH** accessibile da n8n

## Troubleshooting

### Problemi Comuni

#### ❌ "Connection timeout" su SSH
**Causa**: Firewall o credenziali SSH non valide
**Soluzione**: 
- Verifica connettività: `telnet server.azienda.it 22`
- Testa credenziali SSH manualmente
- Controlla whitelist IP n8n su server target

#### ❌ "Slack API error: channel_not_found"
**Causa**: Canale Slack non esistente o bot non invitato
**Soluzione**:
- Verifica nome canale (includi #)
- Invita bot Slack nel canale target
- Controlla scope `chat:write` nel token

#### ❌ "Gmail authentication failed"
**Causa**: OAuth2 scaduto o configurazione incompleta
**Soluzione**:
- Rigenera token OAuth2 Gmail
- Verifica abilitazione "Less secure apps" se usando password
- Controlla 2FA e app-specific password

#### ❌ "PagerDuty invalid routing key"
**Causa**: Integration key non valida o service disabilitato
**Soluzione**:
- Verifica routing key in PagerDuty console
- Controlla stato service (deve essere attivo)
- Testa con curl prima di n8n

### Performance Optimization

#### 🚀 Riduzione Latenza
- Aumenta timeout SSH da 30s a 60s per server lenti
- Usa connection pooling per SSH se supportato
- Implementa caching locale delle metriche per 2-3 minuti

#### 💾 Gestione Memoria
- Limita history metriche a 24h per evitare memory leak
- Usa webhook output invece di full JSON per grandi dataset
- Implementa cleanup automatico dei logs ogni settimana

## Limiti Noti

### Rate Limiting
- **Slack API**: 1 messaggio/secondo per canale
- **Gmail API**: 250 quota units per user per secondo
- **PagerDuty**: 120 events/minute per service

### Scalabilità
- **Server multipli**: Max 50 server per workflow (limitazione memoria)
- **Metriche**: Storage locale limitato a 1000 entries
- **Concurrent SSH**: Max 10 connessioni simultanee

## Estensioni Consigliate

### 📈 Dashboard Integration
- **Grafana**: Export metriche via webhook per visualizzazione
- **Prometheus**: Push metriche usando pushgateway
- **DataDog**: Invia custom metrics via API

### 🤖 Auto-Remediation
- **Service Restart**: SSH command per restart servizi critici
- **Load Balancer**: Rimozione automatica server non healthy
- **Scaling**: Trigger auto-scaling basato su soglie CPU/Memory

### 📱 Mobile Integration
- **Telegram Bot**: Notifiche push istantanee
- **SMS Gateway**: Alert via SMS per emergenze critiche
- **Mobile App**: Dashboard personalizzata per mobile

## Changelog

### v1.0.0 (Current)
- ✅ Monitoring base uptime e risorse
- ✅ Alert multi-canale (Slack/Email/PagerDuty)
- ✅ Configurazione flessibile soglie
- ✅ Parsing automatico metriche sistema
- ✅ Error handling e retry logic

### v1.1.0 (Planned)
- 🔄 Support server multipli con loop
- 🔄 Historical metrics storage
- 🔄 Trend analysis e alerting predittivo
- 🔄 Dashboard web integrato
- 🔄 API endpoint per metriche esterne

### v1.2.0 (Future)
- 🔮 Machine learning per anomaly detection
- 🔮 Integration con tool ITSM (ServiceNow, Jira)
- 🔮 Mobile app dedicata
- 🔮 Multi-tenant support per MSP

---

## Supporto

Per supporto tecnico o personalizzazioni:
- 📧 Email: support@azienda.it
- 💬 Slack: #n8n-support
- 📚 Documentazione: [Wiki interno](https://wiki.azienda.it/n8n)
- 🐛 Bug Report: [GitHub Issues](https://github.com/azienda/n8n-workflows/issues)
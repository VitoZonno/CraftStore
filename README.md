# 🎮 CraftStore - Guida Completa

## 📋 Indice

1. [Panoramica del Sistema](#panoramica-del-sistema)
2. [Requisiti](#requisiti)
3. [Installazione Plugin Minecraft](#installazione-plugin-minecraft)
4. [Configurazione Backend API](#configurazione-backend-api)
5. [Configurazione Sito Web](#configurazione-sito-web)
6. [Test e Verifica](#test-e-verifica)
7. [Risoluzione Problemi](#risoluzione-problemi)
8. [Configurazione Produzione](#configurazione-produzione)

---

## 🎯 Panoramica del Sistema

CraftStore è un sistema **completamente gratuito** per gestire uno store Minecraft online. È composto da:

1. **Plugin Minecraft** (Java) - Gestisce la consegna degli items in-game
2. **Backend API** (Node.js) - Gestisce il sito web e comunica con il plugin
3. **Sito Web** (Next.js) - Interfaccia utente per lo store

### Flusso di Funzionamento

```
Utente acquista sul sito 
    ↓
Backend crea ordine nel database
    ↓
Backend invia ordine al plugin Minecraft (HTTP POST)
    ↓
Plugin salva ordine e lo mette in coda
    ↓
Giocatore si connette al server
    ↓
Plugin consegna automaticamente gli items
```

---

## 📦 Requisiti

### Per il Plugin Minecraft:
- **Java 17** o superiore
- **Maven 3.6+** (per compilare)
- **Server Minecraft**: Paper o Spigot 1.20.1+
- **Database**: SQLite (incluso) o MySQL (opzionale)

### Per il Backend API:
- **Node.js 18+** e npm
- Accesso al server Minecraft (stessa rete o IP pubblico)

### Per il Sito Web:
- **Node.js 18+** e npm (già configurato)

---

## 🔨 Installazione Plugin Minecraft

### Passo 1: Compilare il Plugin

#### Se sei su Windows (PowerShell):
```powershell
# Vai nella directory del progetto
cd C:\Users\Fujitsu\Desktop\CraftStore\CraftStore-Plugin

# Verifica che pom.xml esista
dir pom.xml

# Compila il plugin
mvn clean package
```

#### Se sei su WSL/Linux:
```bash
# Vai nella directory del progetto
cd /mnt/c/Users/Fujitsu/Desktop/CraftStore/CraftStore-Plugin

# Verifica che pom.xml esista
ls pom.xml

# Compila il plugin
mvn clean package
```

#### Se non hai Maven installato:

**Windows:**
1. Scarica Maven da: https://maven.apache.org/download.cgi
2. Estrai l'archivio
3. Aggiungi Maven al PATH di sistema
4. Oppure usa un IDE come IntelliJ IDEA che include Maven

**Linux/WSL:**
```bash
sudo apt update
sudo apt install maven
```

**Verifica installazione:**
```bash
mvn --version
```

### Passo 2: Trovare il File JAR

Dopo la compilazione, il file JAR sarà in:
```
CraftStore-Plugin/target/craftstore-plugin-1.0.0.jar
```

### Passo 3: Installare sul Server Minecraft

1. **Copia il file JAR** nella cartella `plugins/` del tuo server Minecraft
2. **Avvia il server** (Paper o Spigot 1.20.1+)
3. Il plugin creerà automaticamente la cartella `plugins/CraftStore/`

### Passo 4: Configurare il Plugin

Modifica il file `plugins/CraftStore/config.yml`:

```yaml
# Configurazione API Server
api:
  # Porta su cui il server API ascolta le richieste dal sito web
  port: 8080
  # Token di sicurezza per autenticare le richieste (CAMBIALO!)
  token: "GENERA-UN-TOKEN-SICURO-QUI"
  # Indirizzo IP su cui ascoltare (0.0.0.0 = tutti gli IP)
  host: "0.0.0.0"
  # Abilita CORS per il sito web
  enable-cors: true

# Configurazione Database
database:
  # Tipo di database: sqlite o mysql
  type: "sqlite"
  # Per SQLite
  sqlite:
    file: "craftstore.db"
  # Per MySQL (se type = mysql)
  mysql:
    host: "localhost"
    port: 3306
    database: "craftstore"
    username: "root"
    password: "password"
    pool-size: 10

# Configurazione Store
store:
  # Nome dello store
  name: "CraftStore"
  # Valuta virtuale (es: Coins, Gems, ecc.)
  virtual-currency-name: "Coins"
  # Messaggio quando un ordine viene completato
  order-complete-message: "&a[Store] &7Il tuo ordine è stato completato! Controlla il tuo inventario."
  # Messaggio quando un ordine fallisce
  order-failed-message: "&c[Store] &7Il tuo ordine non può essere completato. Contatta un amministratore."
  # Tempo di attesa prima di dare gli items (in secondi)
  delivery-delay: 5
  # Modalità debug (mostra più informazioni nei log)
  debug: false
```

**⚠️ IMPORTANTE**: 
- **Genera un token sicuro** (almeno 32 caratteri)
- **Ricorda questo token!** Ti servirà per il backend

**Come generare un token sicuro:**

**Windows (PowerShell):**
```powershell
-join ((48..57) + (65..90) + (97..122) | Get-Random -Count 32 | ForEach-Object {[char]$_})
```

**Linux/WSL:**
```bash
openssl rand -hex 32
```

**Online:**
Usa un generatore di token online (cerca "random token generator")

### Passo 5: Riavviare il Server

Riavvia il server Minecraft per applicare le modifiche.

### Passo 6: Verificare l'Installazione

Nel server Minecraft, esegui:
```
/craftstore status
```

Dovresti vedere:
- Versione plugin
- Server API: Online
- Porta API: 8080
- Token API: (il tuo token)

---

## 💻 Configurazione Backend API

### Passo 1: Vai nella Directory Backend

**Windows (PowerShell):**
```powershell
cd C:\Users\Fujitsu\Desktop\CraftStore\craftstore-plugin
```

**WSL/Linux:**
```bash
cd /mnt/c/Users/Fujitsu/Desktop/CraftStore/craftstore-plugin
```

### Passo 2: Installa Dipendenze

```bash
npm install
```

Questo installerà tutte le dipendenze necessarie (express, axios, ecc.)

### Passo 3: Crea File .env

Crea un file chiamato `.env` nella cartella `craftstore-plugin/`:

**Windows (PowerShell):**
```powershell
New-Item -Path .env -ItemType File
```

**Linux/WSL:**
```bash
touch .env
```

Poi aggiungi questo contenuto al file `.env`:

```env
# URL del server Minecraft
# Se il server è sulla stessa macchina: http://localhost:8080
# Se il server è su un altro computer: http://IP-DEL-SERVER:8080
MINECRAFT_API_URL=http://localhost:8080

# Token API (DEVE essere identico a quello nel config.yml del plugin!)
MINECRAFT_API_TOKEN=GENERA-UN-TOKEN-SICURO-QUI

# Porta del backend API
PORT=3001

# URL del frontend (per CORS)
FRONTEND_URL=http://localhost:3000
```

**⚠️ IMPORTANTE**: 
- `MINECRAFT_API_TOKEN` deve essere **IDENTICO** al token nel `config.yml` del plugin!
- Se il server Minecraft è su un altro computer, cambia `localhost` con l'IP del server

### Passo 4: Avvia il Backend

```bash
npm start
```

Dovresti vedere:
```
🚀 CraftStore Plugin Server running on port 3001
📡 API available at http://localhost:3001/api
```

### Passo 5: Verificare il Backend

Apri il browser e vai a:
```
http://localhost:3001/health
```

Dovresti vedere:
```json
{
  "status": "ok",
  "timestamp": "2024-..."
}
```

---

## 🌐 Configurazione Sito Web

### Passo 1: Vai nella Directory del Sito

**Windows (PowerShell):**
```powershell
cd C:\Users\Fujitsu\Desktop\CraftStore\minestore-next-official-main
```

**WSL/Linux:**
```bash
cd /mnt/c/Users/Fujitsu/Desktop/CraftStore/minestore-next-official-main
```

### Passo 2: Crea/Modifica File .env

Crea o modifica il file `.env.local` (o `.env`) nella root del progetto Next.js:

```env
# URL del backend API
NEXT_PUBLIC_API_URL=http://localhost:3001
```

### Passo 3: Installa Dipendenze (se necessario)

```bash
npm install
# oppure
pnpm install
```

### Passo 4: Avvia il Sito (Modalità Sviluppo)

```bash
npm run dev
# oppure
pnpm dev
```

Il sito sarà disponibile su: `http://localhost:3000`

---

## ✅ Test e Verifica

### Test 1: Verifica Plugin

Nel server Minecraft:
```
/craftstore status
```

Dovresti vedere:
- Plugin versione
- Server API: Online
- Porta: 8080
- Ordini in sospeso: 0
- Items nello store: 0

### Test 2: Verifica Backend

Apri il browser:
```
http://localhost:3001/health
```

Dovresti vedere una risposta JSON con `"status": "ok"`

### Test 3: Verifica Connessione Backend → Plugin

**Windows (PowerShell):**
```powershell
# Test connessione al plugin
Invoke-WebRequest -Uri "http://localhost:8080/api/status" -Method GET
```

**Linux/WSL:**
```bash
curl http://localhost:8080/api/status
```

Dovresti vedere:
```json
{
  "status": "online",
  "plugin_version": "1.0.0",
  "server_version": "...",
  "online_players": 0,
  "max_players": 100
}
```

### Test 4: Test Completo (Creare Ordine)

1. **Registrati sul sito web** (`http://localhost:3000`)
2. **Aggiungi un item al carrello**
3. **Completa il checkout**
4. **Controlla i log del backend** - Dovresti vedere:
   ```
   Ordine inviato al plugin Minecraft: ORD-...
   ```
5. **Controlla i log del server Minecraft** - Dovresti vedere:
   ```
   [CraftStore] Ordine creato: ORD-... per PlayerName
   ```
6. **Connettiti al server Minecraft** con l'account usato
7. **Dovresti ricevere gli items automaticamente!**

### Test 5: Verifica Ordini in Sospeso

Nel server Minecraft:
```
/craftstore orders
```

Mostra tutti gli ordini in attesa di consegna.

---

## 🐛 Risoluzione Problemi

### Problema: "Maven non trovato"

**Soluzione:**
- Installa Maven (vedi sezione Requisiti)
- Verifica: `mvn --version`

### Problema: "No POM in this directory"

**Soluzione:**
- Assicurati di essere nella directory `CraftStore-Plugin/`
- Verifica: `ls pom.xml` (Linux) o `dir pom.xml` (Windows)

### Problema: "Plugin non si avvia"

**Soluzioni:**
1. Verifica Java 17+: `java -version`
2. Verifica che sia Paper/Spigot 1.20.1+
3. Controlla i log del server per errori
4. Verifica che il file JAR sia nella cartella `plugins/`

### Problema: "Backend non si connette al plugin"

**Soluzioni:**
1. **Verifica token API:**
   - Deve essere identico in `config.yml` (plugin) e `.env` (backend)
   - Controlla spazi o caratteri nascosti

2. **Verifica URL:**
   - Se server su stessa macchina: `http://localhost:8080`
   - Se server su altro computer: `http://IP-SERVER:8080`

3. **Verifica porta:**
   - Controlla che la porta 8080 sia aperta
   - Verifica firewall

4. **Verifica server Minecraft:**
   - Il server deve essere avviato
   - Usa `/craftstore status` per verificare

5. **Test connessione manuale:**
   ```bash
   # Linux/WSL
   curl http://localhost:8080/api/status
   
   # Windows PowerShell
   Invoke-WebRequest -Uri "http://localhost:8080/api/status"
   ```

### Problema: "Ordini non vengono consegnati"

**Soluzioni:**
1. **Verifica giocatore online:**
   - Il giocatore deve essere connesso al server
   - Verifica con `/list` nel server

2. **Controlla ordini in sospeso:**
   ```
   /craftstore orders
   ```

3. **Verifica log server:**
   - Cerca errori nella consegna
   - Attiva debug: `debug: true` in `config.yml`

4. **Verifica item esiste:**
   - L'item deve esistere nel database del plugin
   - Usa `/storeadmin listitems`

### Problema: "Errore npm install"

**Soluzioni:**
1. Verifica Node.js: `node --version` (deve essere 18+)
2. Cancella cache: `npm cache clean --force`
3. Elimina `node_modules` e `package-lock.json`, poi `npm install`

### Problema: "Porta già in uso"

**Soluzioni:**
1. Cambia porta nel `config.yml` (plugin) o `.env` (backend)
2. Verifica processi in ascolto:
   ```bash
   # Linux
   netstat -tulpn | grep :8080
   
   # Windows
   netstat -ano | findstr :8080
   ```

---

## 🚀 Configurazione Produzione

### Se il Server Minecraft è su un Altro Computer

#### 1. Configurazione Plugin

Nel file `plugins/CraftStore/config.yml`:
```yaml
api:
  host: "0.0.0.0"  # Ascolta su tutti gli IP
  port: 8080
  token: "your-secure-token"
```

#### 2. Configurazione Backend

Nel file `.env` del backend:
```env
MINECRAFT_API_URL=http://IP-DEL-SERVER-MINECRAFT:8080
MINECRAFT_API_TOKEN=your-secure-token
```

#### 3. Firewall

**Linux:**
```bash
sudo ufw allow 8080/tcp
```

**Windows:**
- Apri Windows Firewall
- Aggiungi regola per porta 8080 in entrata

#### 4. Verifica Connettività

Dal computer del backend, testa:
```bash
curl http://IP-SERVER-MINECRAFT:8080/api/status
```

### Sicurezza Produzione

1. **Token API Forte:**
   - Usa almeno 32 caratteri casuali
   - Non committare il token su Git
   - Usa variabili d'ambiente

2. **HTTPS (se possibile):**
   - Usa un reverse proxy (nginx) con SSL
   - Configura certificati SSL

3. **Firewall:**
   - Limita accesso alla porta 8080 solo al server backend
   - Non esporre la porta pubblicamente se non necessario

4. **Backup Database:**
   - Fai backup regolari del database SQLite
   - Configura backup automatici

### Ordine di Avvio Produzione

1. **Prima**: Avvia server Minecraft con plugin
2. **Poi**: Avvia backend API
3. **Infine**: Avvia sito web Next.js

---

## 📚 Comandi Utili

### Plugin Minecraft

```
/craftstore                    - Mostra comandi disponibili
/craftstore reload             - Ricarica configurazione
/craftstore status            - Stato del plugin
/craftstore orders            - Ordini in sospeso
/storeadmin listitems         - Lista tutti gli items
/storeadmin removeitem <id>   - Rimuovi un item
```

### Backend API

```bash
# Avvia backend
npm start

# Avvia in modalità sviluppo (con auto-reload)
npm run dev

# Verifica salute API
curl http://localhost:3001/health
```

### Database Plugin

Il database SQLite si trova in:
```
plugins/CraftStore/craftstore.db
```

Puoi visualizzarlo con:
- **DB Browser for SQLite** (Windows/Mac/Linux)
- **SQLite CLI**: `sqlite3 craftstore.db`

---

## 📊 Struttura Database

### Tabella `users`
- `id` - ID utente
- `username` - Nome utente Minecraft
- `uuid` - UUID del giocatore
- `virtual_currency` - Valuta virtuale

### Tabella `store_items`
- `id` - ID item
- `name` - Nome item
- `description` - Descrizione
- `price` - Prezzo
- `virtual_price` - Prezzo in valuta virtuale
- `item_type` - Tipo (item, command, permission, package)
- `item_data` - Dati JSON dell'item
- `commands` - Comandi da eseguire (JSON)
- `permissions` - Permessi da dare (JSON)

### Tabella `orders`
- `id` - ID ordine
- `order_id` - ID ordine univoco
- `username` - Nome utente
- `uuid` - UUID giocatore
- `item_id` - ID item acquistato
- `item_name` - Nome item
- `price` - Prezzo pagato
- `status` - Stato (pending, delivered, failed)
- `delivered` - Se è stato consegnato
- `created_at` - Data creazione
- `delivered_at` - Data consegna

---

## 🎯 Checklist Finale

Prima di andare in produzione:

- [ ] Plugin compilato e installato
- [ ] Token API generato e configurato (identico in entrambi i posti)
- [ ] Backend API avviato e funzionante
- [ ] Sito web configurato con URL backend corretto
- [ ] Testato creazione ordine dal sito
- [ ] Testato consegna items in-game
- [ ] Firewall configurato (se server su computer diversi)
- [ ] Backup del database configurato
- [ ] Log verificati e funzionanti
- [ ] Comandi admin testati

---

## 📞 Supporto

### File di Documentazione

- `GUIDA-COMPLETA.md` - Questa guida
- `CraftStore-Plugin/README.md` - Documentazione plugin
- `CraftStore-Plugin/INTEGRATION.md` - Dettagli integrazione
- `CraftStore-Plugin/SPIEGAZIONE.md` - Spiegazione sistema
- `RIEPILOGO.md` - Riepilogo generale

### Log da Controllare

**Plugin Minecraft:**
- Log del server Minecraft (console o file `logs/latest.log`)

**Backend API:**
- Console dove hai avviato `npm start`
- File di log se configurati

**Sito Web:**
- Console del browser (F12)
- Log del server Next.js

---

## ✨ Risultato Finale

Hai un sistema **completo, gratuito e open-source** per gestire uno store Minecraft:

- ✅ Sito web professionale (Next.js)
- ✅ Backend API funzionante
- ✅ Plugin Minecraft che consegna items
- ✅ Sistema di pagamenti
- ✅ Gestione utenti e ordini
- ✅ Database SQLite/MySQL
- ✅ API RESTful completa
- ✅ **Tutto completamente gratuito!**

---

**Buona fortuna con CraftStore!** 🎮✨

*Se hai problemi, controlla la sezione "Risoluzione Problemi" o i file di documentazione.*


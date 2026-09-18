# 🏓 Ping Pong Multiplayer

**Un gioco di Ping Pong multiplayer in tempo reale, via browser, pensato per il telefono — nessuna app da installare, basta condividere un link.**

[![Piattaforma](https://img.shields.io/badge/piattaforma-Web%20%28Mobile--first%29-blue)](public/index.html)
[![Node](https://img.shields.io/badge/node-%3E%3D14-339933)](https://nodejs.org/)
[![Tempo reale](https://img.shields.io/badge/tempo%20reale-Socket.IO-black)](https://socket.io/)
[![Licenza](https://img.shields.io/badge/licenza-MIT-green)](LICENSE)

🇬🇧 [Read in English](README.md)

Due giocatori, due telefoni, una sola partita. Ping Pong Multiplayer è una piccola applicazione
Node.js + Socket.IO che gestisce l'intera partita lato server — fisica della palla, collisioni e
punteggio — e trasmette lo stato a entrambi i browser in tempo reale via WebSocket. I giocatori si
collegano con nient'altro che un URL condiviso, scelgono un lato e iniziano a giocare.

---

## Indice

- [Caratteristiche](#caratteristiche)
- [Per iniziare](#per-iniziare)
- [Come si gioca](#come-si-gioca)
- [Configurazione](#configurazione)
- [Distribuzione](#distribuzione)
- [Stack tecnologico](#stack-tecnologico)
- [Struttura del progetto](#struttura-del-progetto)
- [Come funziona](#come-funziona)
- [Licenza](#licenza)

---

## Caratteristiche

- **Multiplayer in tempo reale** — il game loop autoritativo gira sul server a 60 FPS e trasmette
  gli aggiornamenti di palla e racchette via Socket.IO
- **Mobile-first, controlli touch** — si muove la racchetta trascinando il dito, niente tastiera
- **Due modalità di visualizzazione** — campo intero visibile, oppure vista sul solo proprio lato
- **Orientamento orizzontale e verticale** — il primo giocatore che entra decide se il tavolo è
  disposto sinistra/destra o alto/basso
- **Punteggio e condizione di vittoria** — vince chi arriva per primo a 10 punti
- **Effetto sulla racchetta** — il punto in cui la palla colpisce la racchetta ne cambia l'angolo
  di rimbalzo
- **Nessuna installazione** — gira interamente nel browser del telefono, niente da scaricare

## Per iniziare

### Prerequisiti

- [Node.js](https://nodejs.org/) v14 o superiore
- npm (incluso con Node.js)

### Installazione

```bash
git clone https://github.com/iAlias/PingPongMultiplayer.git
cd PingPongMultiplayer
npm install
```

### Avvio

```bash
npm start
```

Poi apri un browser su:

```
http://localhost:3000
```

Apri lo stesso URL da un secondo dispositivo (o una seconda scheda/finestra del browser) per far
entrare il secondo giocatore.

## Come si gioca

1. **Entrambi i giocatori** aprono lo stesso URL sui propri dispositivi
2. Inseriscono un nome
3. Scelgono un lato (Sinistra o Destra)
4. Scelgono una modalità di visualizzazione (campo intero o mezzo campo)
5. Toccano **"Entra in partita"**
6. Attendono che l'altro giocatore entri — la partita parte automaticamente quando entrambi i
   lati sono occupati
7. Si muove la racchetta toccando e trascinando su/giù (o sinistra/destra in modalità verticale)
8. Vince chi arriva per primo a 10 punti

## Configurazione

Il server legge le proprie impostazioni da variabili d'ambiente:

| Variabile | Predefinito | Descrizione |
|---|---|---|
| `PORT` | `3000` | La porta su cui il server HTTP/WebSocket resta in ascolto |
| `ALLOWED_ORIGINS` | `*` (tutte le origini) | Elenco separato da virgole delle origini CORS ammesse per le connessioni Socket.IO |

Esempio, limitando a un dominio specifico di produzione:

```bash
ALLOWED_ORIGINS=https://your-app.onrender.com,https://your-custom-domain.com npm start
```

Se `ALLOWED_ORIGINS` non viene impostata, il server stampa un avviso all'avvio e accetta
connessioni da qualsiasi origine — comodo per i test locali, ma sconsigliato in produzione.

## Distribuzione

L'app è un semplice server Node.js/Express, quindi si distribuisce su qualsiasi hosting
compatibile con Node. Alcune opzioni comuni:

### Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

### Render

1. Crea un nuovo Web Service su [Render](https://render.com) e collega questo repository
2. Comando di build: `npm install`
3. Comando di avvio: `npm start`
4. (Opzionale) imposta la variabile d'ambiente `ALLOWED_ORIGINS` con l'URL della tua app Render

### Heroku

```bash
heroku login
heroku create your-app-name
git push heroku main
```

## Stack tecnologico

- **Frontend** — HTML5 Canvas, CSS3, JavaScript vanilla (`public/game.js`, `public/socket.js`)
- **Backend** — Node.js con Express, che serve il frontend statico e ospita il game loop
- **Comunicazione in tempo reale** — [Socket.IO](https://socket.io/) su WebSocket

## Struttura del progetto

```
PingPongMultiplayer/
├── server/
│   └── server.js       Server backend: app Express, handler Socket.IO, game loop a 60 FPS
├── public/
│   ├── index.html       Struttura della pagina e schermata di configurazione pre-partita
│   ├── style.css        Stile e layout mobile-first
│   ├── game.js           Rendering su canvas e game loop lato client
│   └── socket.js         Connessione client Socket.IO e gestione eventi
├── package.json          Dipendenze e script npm
└── LICENSE
```

## Come funziona

`server/server.js` è l'unica fonte di verità della partita: tiene in memoria `gameState`
(giocatori, posizione/velocità della palla, punteggio, orientamento, flag di inizio/fine) e
gestisce ogni regola del gioco.

- **Ingresso in partita** — un evento `joinGame` assegna il socket richiedente al lato `left` o
  `right` (oppure sceglie automaticamente un lato libero in modalità verticale), rifiuta un lato
  già occupato o una partita piena, e avvia la partita quando entrambi i posti sono occupati
- **Input** — i client emettono `movePaddle` con la posizione normalizzata della propria
  racchetta; il server la ritrasmette all'altro giocatore tramite `paddleUpdate`, così ogni client
  disegna localmente entrambe le racchette
- **Fisica** — un loop `setInterval` scandisce ogni 1000/60 ms, avanza la palla secondo la sua
  velocità, la fa rimbalzare sulle pareti, rileva le collisioni con le racchette (con un fattore
  di effetto basato sul punto di impatto) e applica la velocità tramite
  `BALL_SPEED_INCREASE_FACTOR`, poi emette `ballUpdate` a entrambi i client
- **Orientamento** — la stessa funzione di fisica (`updateBall`) gestisce sia il layout
  tradizionale sinistra/destra sia quello ruotato alto/basso ("verticale"), deciso dal primo
  giocatore che si collega
- **Punteggio e disconnessioni** — un punto incrementa `gameState.score` e trasmette
  `scoreUpdate`; arrivare a 10 punti termina la partita con `gameEnded`; la disconnessione di un
  giocatore azzera la partita e avvisa il client rimasto tramite `playerLeft`

## Licenza

Questo progetto è distribuito con licenza [MIT](LICENSE) © 2026 Antonio Di Stefano.

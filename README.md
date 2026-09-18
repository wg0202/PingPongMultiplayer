# 🏓 Ping Pong Multiplayer

**A real-time, browser-based two-player Ping Pong game built for phones — no app to install, just share a link.**

[![Platform](https://img.shields.io/badge/platform-Web%20%28Mobile--first%29-blue)](public/index.html)
[![Node](https://img.shields.io/badge/node-%3E%3D14-339933)](https://nodejs.org/)
[![Realtime](https://img.shields.io/badge/realtime-Socket.IO-black)](https://socket.io/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)

🇮🇹 [Leggi in italiano](README.it.md)

Two players, two phones, one game. Ping Pong Multiplayer is a small Node.js + Socket.IO
application that runs the entire match server-side — ball physics, collisions and scoring — and
streams the state to both browsers in real time over WebSockets. Players connect with nothing
more than a shared URL, pick a side, and start playing.

---

## Contents

- [Features](#features)
- [Getting started](#getting-started)
- [How to play](#how-to-play)
- [Configuration](#configuration)
- [Deployment](#deployment)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [How it works](#how-it-works)
- [License](#license)

---

## Features

- **Real-time multiplayer** — the authoritative game loop runs on the server at 60 FPS and
  broadcasts ball and paddle updates over Socket.IO
- **Mobile-first, touch-controlled** — drag your finger to move the paddle, no keyboard needed
- **Two view modes** — play with the full field visible, or a half-field view focused on your
  own side
- **Horizontal and vertical orientation** — the first player to join sets whether the table is
  laid out left/right or top/bottom
- **Score tracking and win condition** — first player to 10 points wins the match
- **Spin on paddle hits** — where the ball strikes the paddle changes its rebound angle
- **Zero installation** — runs entirely in the mobile browser, nothing to download

## Getting started

### Prerequisites

- [Node.js](https://nodejs.org/) v14 or later
- npm (bundled with Node.js)

### Installation

```bash
git clone https://github.com/iAlias/PingPongMultiplayer.git
cd PingPongMultiplayer
npm install
```

### Run

```bash
npm start
```

Then open a browser to:

```
http://localhost:3000
```

Open the same URL from a second device (or a second browser tab/window) to bring in the other
player.

## How to play

1. **Both players** open the same URL on their devices
2. Enter a display name
3. Choose a side (Left or Right)
4. Choose a view mode (full field or half field)
5. Tap **"Entra in partita"** (Join Game)
6. Wait for the other player to join — the match starts automatically once both sides are taken
7. Move your paddle by touching and dragging up/down (or left/right in vertical mode)
8. First player to reach 10 points wins

## Configuration

The server reads its settings from environment variables:

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3000` | The port the HTTP/WebSocket server listens on |
| `ALLOWED_ORIGINS` | `*` (all origins) | Comma-separated list of allowed CORS origins for Socket.IO connections |

Example, restricting to a specific deployed domain:

```bash
ALLOWED_ORIGINS=https://your-app.onrender.com,https://your-custom-domain.com npm start
```

If `ALLOWED_ORIGINS` is left unset, the server logs a warning at startup and accepts connections
from any origin — convenient for local testing, but not recommended in production.

## Deployment

The app is a plain Node.js/Express server, so it deploys to any Node-friendly host. A few common
options:

### Railway

```bash
npm install -g @railway/cli
railway login
railway init
railway up
```

### Render

1. Create a new Web Service on [Render](https://render.com) and connect this repository
2. Build command: `npm install`
3. Start command: `npm start`
4. (Optional) set the `ALLOWED_ORIGINS` environment variable to your Render app's URL

### Heroku

```bash
heroku login
heroku create your-app-name
git push heroku main
```

## Technology stack

- **Frontend** — HTML5 Canvas, CSS3, vanilla JavaScript (`public/game.js`, `public/socket.js`)
- **Backend** — Node.js with Express, serving the static frontend and hosting the game loop
- **Real-time communication** — [Socket.IO](https://socket.io/) over WebSockets

## Project structure

```
PingPongMultiplayer/
├── server/
│   └── server.js       Backend server: Express app, Socket.IO handlers, 60 FPS game loop
├── public/
│   ├── index.html       Page structure and pre-game setup screen
│   ├── style.css        Mobile-first styling and layout
│   ├── game.js           Canvas rendering and client-side game loop
│   └── socket.js         Socket.IO client connection and event handling
├── package.json          Dependencies and npm scripts
└── LICENSE
```

## How it works

`server/server.js` is the single source of truth for the match: it holds `gameState` (players,
ball position/velocity, score, orientation, started/ended flags) in memory and owns every rule
of the game.

- **Joining** — a `joinGame` event assigns the requesting socket to `left` or `right` (or
  auto-picks a free side in vertical mode), rejects a full or already-taken side, and starts the
  match once both seats are occupied
- **Input** — clients emit `movePaddle` with their normalized paddle position; the server
  rebroadcasts it to the other player via `paddleUpdate`, so each client renders both paddles
  locally
- **Physics** — a `setInterval` loop ticks at 1000/60 ms, advancing the ball by its velocity,
  bouncing it off the walls, detecting paddle collisions (with a spin factor based on where the
  ball struck the paddle), and applying speed via `BALL_SPEED_INCREASE_FACTOR`, then emits
  `ballUpdate` to both clients
- **Orientation** — the same physics function (`updateBall`) is used for both the traditional
  left/right layout and a rotated top/bottom ("vertical") layout, decided by whichever player
  connects first
- **Scoring and disconnects** — a goal increments `gameState.score` and broadcasts
  `scoreUpdate`; reaching 10 points ends the match via `gameEnded`; a player disconnecting resets
  the match and notifies the remaining client through `playerLeft`

## License

This project is licensed under the [MIT License](LICENSE) © 2026 Antonio Di Stefano.

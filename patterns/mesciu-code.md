# Pattern di programmazione di Mesciu

Convenzioni del codice di Mesciu (Electron: main CommonJS + preload bridge + renderer React).
Carica questo file quando scrivi codice in stile Mesciu o vuoi riusarne l'architettura.

## Architettura a tre strati

- **Main** (`electron/main/*.js`, `'use strict'`, CommonJS): tutta la logica, i file, la rete,
  lo store. Un modulo per dominio (`presets.js`, `updater.js`, `remote.js`…), con un commento
  di testa che spiega COSA fa e le decisioni non ovvie.
- **Preload** (`electron/preload/index.js`): bridge 1:1 — ogni handler IPC ha un metodo
  nominato `dominioAzione(args) => ipcRenderer.invoke('dominio:azione', {...})`. Niente logica.
- **Renderer** (React): chiama `window.api.*` sempre con optional chaining
  (`window.api.metodo?.(…)`) e gestisce il risultato, mai eccezioni.

## Il contratto IPC: `{ ok, … } | { ok:false, error }`

Gli handler **non lanciano mai verso il renderer**: ritornano un oggetto con `ok` e, in caso
di errore, un messaggio in italiano già pronto per la UI.

```js
handle('presets:install', async (_, { sessionId, id }) => presets.install(await gitCwd(sessionId), id))
// nel modulo: return { ok: false, error: 'Preset non trovato nella libreria.' }
```

Per feature che valgono sia in locale sia in remoto, il renderer usa un **adapter `ops`**:
stesso set di funzioni (`status/install/uninstall/…`) implementate una volta con gli IPC locali
e una volta col canale remoto — la UI resta unica.

## Dati non fidati

Tutto ciò che arriva da fuori (manifest remoti, URL, body di rete) passa da **validatori puri
che scartano senza lanciare**: whitelist sui valori (`['skill','agent','file'].includes(...)`),
normalizzazione con default, id duplicati ignorati. Regole fisse:

- **Path-safety**: destinazione = `path.resolve(base, rel)` e si accetta solo se
  `abs.startsWith(base + path.sep)`. Vale per scritture E cancellazioni.
- **URL**: solo `https`, estensioni ammesse esplicite, `new URL()` per il parsing.
- **Download**: cap di dimensione (1 MB), redirect limitati, timeout.

## Funzioni pure esportate per i test

La logica testabile si estrae in funzioni pure esportate col prefisso `_`
(`_validateManifest`, `_parseMdUrl`, `_buildLoadPrompt`) e si testa con vitest; il modulo
`electron` è aliasato a uno stub (`test/electron-stub.js`) così i moduli del main si importano
in Node puro. Tre livelli di test, tutti con marcatori di esito espliciti:

```
npm test          # unit (vitest) — veloci, funzioni pure
npm run smoke     # SMOKE_OK: l'app Electron reale parte senza crash
npm run test:e2e  # E2E_OK: pilota l'app vera e verifica gli artefatti su disco
```

Negli e2e **mai sleep fissi**: si aspetta sempre lo stato del DOM o un marcatore nell'output.

## Altri pattern ricorrenti

- **Cascata di disponibilità** per le risorse remote: remoto → cache persistente nello store
  (`{ at, items }`) → copia bundled nell'app. La UI riceve `from` per dire da dove viene il dato.
- **Store**: schema centralizzato con tipo + default + commento per OGNI chiave; letto/scritto
  solo dal main.
- **Protocollo remoto**: azioni-stringa su un unico endpoint di controllo
  (`control(action, sessionId, body)` con body JSON); il peer esegue e ritorna il payload,
  il client incapsula in `{ ok, result }` e rigetta su errore di rete.
- **Lazy require** per spezzare i cicli tra moduli (`require('./bot')` dentro la funzione,
  con commento che spiega il ciclo evitato).
- **Commenti**: spiegano il PERCHÉ e i vincoli non visibili dal codice (race, bug storici,
  scelte semver), in italiano; niente commenti che ripetono il codice.
- **Commit**: Conventional Commits in italiano (`feat(scope): …`), corpo che spiega il perché;
  commit locali granulari durante il lavoro, squash in un commit riassuntivo alla release.

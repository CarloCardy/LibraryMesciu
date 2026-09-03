# Telecamere Casa sul NAS UGREEN (UGOS Pro)

L'app gira sul NAS e si apre da telefono o PC all'indirizzo del NAS, come una
qualsiasi applicazione del NAS. Chi accede lo decidi tu: c'è un amministratore
e gli utenti che crei, ciascuno con le proprie telecamere.

**Le telecamere vengono contattate solo mentre l'app è aperta.** Chiudendo la
pagina (o l'app sul telefono) il NAS smette entro pochi secondi di ricevere e
rielaborare qualsiasi flusso: nessun processo, nessuna connessione all'NVR,
consumo praticamente nullo.

---

> **Hai un DH2300?** Quel modello **non ha Docker** nel suo App Center (UGOS Pro
> ridotto, verificato sul catalogo ufficiale UGREEN). Per il DH2300 usa il
> pacchetto nativo: **[upk/README.md](upk/README.md)**. Questa guida vale per i
> modelli con Docker (DXP2800/4800/6800/8800, DH4300 Plus).

## Perché Docker e non un pacchetto `.upk`

UGREEN non pubblica un SDK né la specifica del formato `.upk`: quei pacchetti
sono firmati e installabili solo tramite l'App Center. Anche progetti noti
(Tailscale, NetBird) hanno chiesto un pacchetto UGOS senza ottenerlo, e la via
indicata da UGREEN è Docker, incluso nell'App Center di UGOS Pro.

Il risultato per te è identico: l'app compare fra i container del NAS, parte da
sola all'accensione e si apre da browser. Su telefono, con "Aggiungi a schermata
Home", ottieni icona propria e apertura a schermo intero.

---

## 1. Prepara l'immagine (sul Mac)

Serve Docker Desktop (o OrbStack/Podman) sul computer, una volta sola.

```bash
cd "Telecamere Casa"
bash docker/build.sh              # NAS con CPU Intel/AMD (la maggior parte)
ARCH=arm64 bash docker/build.sh   # solo se il tuo NAS ha CPU ARM
```

Ottieni **`telecamere-casa.tar`**. Copialo sul NAS, ad esempio nella cartella
condivisa `docker`.

> In dubbio sull'architettura: UGOS Pro → Pannello di controllo → Informazioni
> di sistema → Processore. Se dice Intel o AMD, usa il comando senza `ARCH`.

---

## 2. Crea le cartelle sul NAS

In **File Manager**, dentro la cartella condivisa `docker`, crea:

```
docker/telecamere/config
```

Qui finiscono la lista delle telecamere, gli utenti e la chiave di sessione.
È l'unica cartella da salvare se un giorno rifai tutto.

---

## 3. Importa l'immagine

**App Center → Docker → Immagini → Importa** (o "Aggiungi → Da file"), scegli
`telecamere-casa.tar` e attendi. Al termine compare `telecamere-casa:latest`.

---

## 4. Crea il container

Dall'immagine importata, **Crea container** con queste impostazioni:

| Voce | Valore |
|---|---|
| Nome | `telecamere-casa` |
| Riavvio automatico | attivo |
| Rete | **host** |
| Volume | `docker/telecamere/config` → `/config` |

Variabili d'ambiente:

| Variabile | Valore | A cosa serve |
|---|---|---|
| `PORT` | `3000` | Porta di accesso |
| `LOW_POWER` | `1` | Modalità risparmio (usa il sub-stream dell'NVR) |
| `TZ` | `Europe/Rome` | Ora corretta nell'interfaccia |
| `PUID` / `PGID` | `1000` | Proprietario della cartella config |

> **Rete host** è la scelta consigliata: permette al container di raggiungere
> NVR e telecamere sulla rete di casa e pubblica da sé le porte aggiuntive che
> il server apre automaticamente ogni 6 telecamere (3001, 3002…), necessarie
> perché i browser accettano al massimo 6 connessioni per indirizzo.
>
> Se la rete host non fosse disponibile, usa la modalità bridge e pubblica a
> mano l'intervallo `3000-3005`.

Avvia il container.

---

## 5. Primo accesso

Da un browser sulla stessa rete:

```
http://<indirizzo-del-nas>:3000
```

Alla prima apertura compare **Primo avvio**: scegli nome utente e password
dell'amministratore. Da quel momento l'accesso è protetto.

> In alternativa puoi crearlo senza wizard aggiungendo le variabili
> `ADMIN_USER` e `ADMIN_PASSWORD` al container prima del primo avvio, e
> rimuovendole subito dopo.

Poi: **Impostazioni → Telecamere → + Aggiungi** (o "Cerca canali" per trovare
automaticamente i canali dell'NVR).

---

## 6. Utenti e permessi

**Impostazioni → Utenti** (visibile solo agli amministratori).

- **Amministratore** — vede tutto, configura le telecamere, gestisce gli utenti.
- **Visualizzatore** — vede solo le telecamere che gli assegni, non può
  modificare nulla e non riceve mai le credenziali dell'NVR.

Ogni utente ha la sua password, gestita dall'app: gli account di UGOS non sono
leggibili da dentro un container, quindi l'elenco è indipendente da quello del
NAS. Puoi usare gli stessi nomi utente per comodità.

I permessi non sono solo grafici: un visualizzatore che provasse ad aprire
l'indirizzo di una telecamera non sua riceve un rifiuto dal server.

Ognuno può cambiare la propria password da **Impostazioni → Il mio account**.
Cambiandola, le sessioni aperte altrove decadono.

---

## 7. Aggiungere l'app al telefono

Apri `http://<indirizzo-del-nas>:3000` dal telefono, accedi, poi:

- **iPhone/iPad** — Condividi → *Aggiungi a schermata Home*
- **Android** — menu ⋮ → *Installa app* / *Aggiungi a schermata Home*

L'icona apre l'app a schermo intero, senza barra del browser. In alto a destra
c'è il pulsante per uscire dall'account.

---

## Come funziona il risparmio energetico

| Momento | Cosa fa il NAS |
|---|---|
| App chiusa | Solo il server web in attesa. Nessun FFmpeg, nessuna connessione alle telecamere. |
| App aperta | Un processo per ogni telecamera **realmente a video** (le altre pagine no). |
| App in secondo piano | Dopo 5 secondi tutto viene chiuso. |
| App chiusa di colpo | L'app batte un "sono qui" ogni 5 secondi: se manca, entro 15 secondi il server chiude tutto da sé. |

Il NAS non decodifica né ricodifica il video: cambia solo il contenitore
(`-c:v copy`). La decodifica avviene sul dispositivo che guarda. Anche con l'app
aperta il carico sul NAS resta molto basso.

Nessuna registrazione, nessun rilevamento movimento, nessuna analisi: l'app
mostra il flusso mentre lo guardi e nient'altro.

---

## Manutenzione

**Aggiornare l'app** — ricostruisci l'immagine sul Mac (`bash docker/build.sh`),
importa il nuovo `.tar` sul NAS, ferma ed elimina il container, ricrealo dalla
nuova immagine con lo stesso volume. La configurazione in `/config` resta.

**Backup** — copia la cartella `docker/telecamere/config`. Contiene telecamere,
utenti e chiave di sessione. In alternativa, da Impostazioni → Backup
configurazione puoi esportare le telecamere in JSON.

**Password dimenticata dell'amministratore** — ferma il container, elimina
`config/users.json`, riavvia: riparte il wizard di primo avvio. Le telecamere
restano.

**Log** — App Center → Docker → Container → `telecamere-casa` → Log. Le righe
`[idle]` mostrano gli stream chiusi per inattività, `[stream:…]` quelli aperti.

---

## Se qualcosa non va

| Sintomo | Causa più probabile |
|---|---|
| La pagina non si apre | Container fermo, o rete non in modalità host con porte non pubblicate. |
| "Riconnessione…" su tutte le celle | Il NAS non raggiunge l'NVR: controlla IP, porta 554 e credenziali. |
| Errore di scrittura sulla configurazione | `PUID`/`PGID` non corrispondono al proprietario della cartella `config`. |
| Oltre 6 telecamere, alcune restano nere | Servono le porte aggiuntive: usa la rete host, oppure pubblica `3000-3005`. |
| Sessione che scade subito | La cartella `/config` non è persistente: verifica il volume. |

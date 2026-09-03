---
name: nas-ugreen
description: Lavorare sul NAS UGREEN (UGOS Pro, DH2300 arm64) via SSH e costruire o installare un'app per UGOS. Usala prima di toccare il NAS, per un deploy di Telecamere Casa, per leggere log o sbloccare un account, e ogni volta che si parla di .upk, App Center, portale UGOS o Docker sul NAS. Non contiene dati personali: li chiede all'utente.
---

# NAS UGREEN: lavoro via SSH e app per UGOS Pro

Obiettivo: intervenire sul NAS senza riscoprire i limiti di UGOS, con un deploy
verificabile e senza lasciare in giro segreti.
**Questo file non contiene dati personali e non deve mai contenerli:** indirizzi,
account, chiavi e percorsi si chiedono all'utente e restano nella conversazione.

## Quando usarla

- Prima di qualsiasi comando `ssh` verso il NAS.
- Per portare sul NAS un aggiornamento del codice e confermarlo.
- Per diagnosticare (log, porte, permessi) o sbloccare un account con `recovery.js`.
- Quando l'utente chiede di "fare un'app per il NAS", di un `.upk`, dell'App Center,
  del portale UGOS o di Docker sul NAS.

---

## Passo 0. Chiedi i dati all'utente

| Segnaposto | Cosa chiedere |
|---|---|
| `<IP_NAS>` | indirizzo IP del NAS sulla rete di casa |
| `<UTENTE_SSH>` | account amministratore UGOS usato per SSH |
| `<CHIAVE_SSH>` | percorso della chiave privata autorizzata (mai leggerla né copiarla) |
| `<REPO_LOCALE>` | cartella del progetto sul computer dell'utente |
| alias UGREENlink | `https://ug.link/<alias>`, solo per provare l'accesso da fuori |

Le password non si chiedono mai: SSH è a chiave, `sudo` non la richiede,
gli account dell'app si sbloccano con `recovery.js`.

### Se l'utente non li ricorda

Comandi di sola lettura; i risultati **non vanno trascritti** in file, memorie condivise o commit.

- **`<IP_NAS>`**: UGOS Pro → *Pannello di controllo* → *Rete*. Dal Mac:
  `dns-sd -B _ssh._tcp local.` (Ctrl+C per fermare), `arp -a`, oppure `ping <nome>.local`.
  Verifica: `curl -s -o /dev/null -w '%{http_code}\n' http://<IP_NAS>:9765/api/health` → 200.
- **`<UTENTE_SSH>`**: il primo utente creato in UGOS, ruolo amministratore
  (*Pannello di controllo* → *Utenti*). SSH si abilita in *Terminale e SNMP*, porta 22.
- **`<CHIAVE_SSH>`**: `ls -l ~/.ssh/` (le private sono i file senza `.pub`),
  `grep -i -B1 -A6 'nas\|ugreen' ~/.ssh/config`, `ssh-add -l`. Prova ciascuna con
  `ssh -i <chiave> -o BatchMode=yes <UTENTE_SSH>@<IP_NAS> true`: esce 0 con quella giusta.
  Se nessuna funziona, l'utente lancia `ssh-copy-id` **da un Terminale vero**: Claude non può.
- **`<REPO_LOCALE>`**: `git rev-parse --show-toplevel`. Nome del remoto: `git remote -v` (resta privato).
- **UGREENlink**: *Pannello di controllo* → *Accesso remoto*. Apre il desktop UGOS, non è un proxy di percorsi.

---

## Scheda del NAS (fatti verificati)

| Voce | Valore |
|---|---|
| Modello | UGREEN NASync **DH2300**, Rockchip RK3576, 4 GB |
| Architettura | **aarch64 / arm64**: un binario amd64 non parte |
| Sistema | UGOS Pro su Debian 12 bookworm, systemd, nginx, sqlite3 |
| Accesso | SSH a chiave, `<UTENTE_SSH>` uid 1000 gruppo `admin`; `sudo -n` senza password |
| Portale UGOS | nginx su `:9999` (http) e `:9443` (https) |
| App | `http://<IP_NAS>:9765`; dietro il portale `http(s)://<IP_NAS>:9999|9443/telecamere/` |
| Codice | `~/telecamere-casa/app/` (**non** `~/telecamere-casa/`) |
| Dati | `~/.config/telecamere/` |
| Servizio | `systemd --user`, unit `~/.config/systemd/user/telecamere.service`, linger attivo |
| Runtime | Node 20.18.1 in `~/telecamere-casa/sbin/node` (quello di sistema è 18: non usarlo); ffmpeg 7.0.2 statico in `~/telecamere-casa/sbin/ffmpeg` |
| App Center | ridotto, **senza Docker** (il DH4300 Plus, stesso arm64, lo ha) |

Comando base da usare sempre:

```bash
NAS='ssh -i <CHIAVE_SSH> <UTENTE_SSH>@<IP_NAS>'
$NAS 'systemctl --user is-active telecamere'
```

Ogni chiamata stampa su stderr un avviso post-quantum ("store now, decrypt later"): innocuo.

---

## Passi

### A. Collegarsi e leggere lo stato

```bash
$NAS 'uname -m; head -3 /etc/os-release'
$NAS 'cat ~/.config/systemd/user/telecamere.service'
$NAS 'ls ~/telecamere-casa/app ~/.config/telecamere'
$NAS 'ss -ltn | grep -E ":9765|:9999|:9443"'
$NAS 'sudo -n true && echo "sudo senza password"'
```

Limiti di UGOS da rispettare (già verificati, non riprovarli):

- `scp` e SFTP **bloccati**: trasferisci con `ssh NAS 'cat > dest' < locale` o pipe `tar`.
- `/tmp` **non scrivibile** dall'utente: appoggiati nella home.
- Nessun prompt di password funziona dai comandi di Claude Code (niente TTY).
- Le ACL della home (`+` in `ls -l`) rendono i file copiati con `tar` `rwxrwxrwx`:
  il server all'avvio esegue `hardenConfigPerms()` e riporta i segreti a `0600`.
- `timeout` non esiste su macOS.
- SSH può risultare `Connection refused` per qualche minuto: aspetta e controlla via HTTP.

### B. Deploy di un aggiornamento

Solo su richiesta dell'utente e con i test locali verdi. La destinazione è
**sempre** `~/telecamere-casa/app/<percorso relativo>`: il livello superiore
contiene gli avanzi dello staging `.upk` e il deploy lì fallisce in silenzio su `public/`.

```bash
cd <REPO_LOCALE>
NAS='ssh -i <CHIAVE_SSH> <UTENTE_SSH>@<IP_NAS>'

# 1. copia (un file, oppure tutti i file di runtime)
$NAS 'cat > ~/telecamere-casa/app/server.js' < server.js
tar czf - server.js auth.js detect.js notify.js devices.js ai.js ai-worker.js recovery.js public \
  | $NAS 'tar xzf - -C ~/telecamere-casa/app'

# 2. riavvio e salute
$NAS 'systemctl --user restart telecamere && sleep 2 && systemctl --user is-active telecamere'
curl -s -o /dev/null -w '%{http_code}\n' http://<IP_NAS>:9765/api/health      # 200

# 3. il NAS ha esattamente l'HEAD locale?
for f in server.js auth.js detect.js notify.js devices.js ai.js ai-worker.js recovery.js \
         public/index.html public/login.html public/js/app.js public/js/login.js public/css/style.css; do
  L=$(shasum -a 256 "$f" | cut -d' ' -f1)
  R=$($NAS "sha256sum ~/telecamere-casa/app/$f" 2>/dev/null | cut -d' ' -f1)
  [ "$L" = "$R" ] || echo "DIVERSO: $f"
done

# 4. permessi dei segreti
$NAS 'ls -l ~/.config/telecamere'    # cameras, users, secret.key, notify, devices, shares: -rw-------
```

In zsh una variabile con più righe non viene divisa in parole: elenchi inline o `bash -c`.

Se cambia `package.json`, le dipendenze si installano **sul NAS** (binari nativi arm64,
per esempio `onnxruntime-node`), mai copiate dal Mac:

```bash
$NAS 'cd ~/telecamere-casa/app && PATH=~/telecamere-casa/sbin:$PATH npm install --omit=dev --ignore-scripts --no-audit --no-fund'
```

### C. Diagnostica, rollback, recupero account

```bash
$NAS 'systemctl --user status telecamere --no-pager'
$NAS 'journalctl --user -u telecamere -n 100 --no-pager'
$NAS 'journalctl --user -u telecamere -f'
git show <commit>:server.js | $NAS 'cat > ~/telecamere-casa/app/server.js'   # rollback di un file, poi riavvio
$NAS 'cd ~/telecamere-casa/app && ~/telecamere-casa/sbin/node recovery.js'                  # utenti e stato
$NAS 'cd ~/telecamere-casa/app && ~/telecamere-casa/sbin/node recovery.js unlock <utente>'  # sblocco, senza riavvio
```

I dati in `~/.config/telecamere` non si toccano e non si copiano sul Mac: contengono
le password dell'NVR in chiaro, gli utenti, la chiave di sessione, il token Telegram
e le chiavi dei dispositivi.

### D. Interventi da root (solo parte portale)

Con `sudo -n`, backup in `/root/backup-telecamere`, niente riavvii del NAS.

```bash
$NAS 'sudo -n cat /etc/nginx/conf.d/telecamere.conf'
$NAS 'sudo -n nginx -t && sudo -n nginx -s reload'
```

---

## Com'è fatto UGOS Pro (riferimento)

- **Portale**: `/etc/nginx/ugreen.conf` ascolta su 9999 e include `/etc/nginx/conf.d/*.conf`;
  ogni servizio ha lì il suo `*_serv.conf` con un blocco `location`.
- **App Center**: demone `app_serv`; app in `/ugreen/@appstore/<appId>/` con `config.json`,
  `icon.png`, `init.d/`, `nginx/` o `http.d/`, e `.check-upk`
  (json `{file: {hash md5, modify_time, size}}`). Icone desktop in `/ugreen/static/icons/<appId>.png`.
- **Database runtime** `/ugreen/.config/.appstore/appstore_appruntime.db` (sqlite3):
  `app_ports(appid, http_port, https_port, …)` e `app_users(appid, user_name, group_name, uid, …)`.
  Catalogo scaricato in `appstore_app.db`.
- **Servizi**: unit `<nome>_serv.service`, `After=app_serv.service network-online.target`,
  `ConditionPathExists=/etc/sysconfig/<nome>_serv.sh` ed `ExecStartPre` sullo stesso script,
  che crea i link in `/var/targets/<app>/` e la cartella dati `/var/ugreen/<app>/`.
- **Utenti UGOS**: non leggibili da un'app né da un container. L'app tiene i propri.
- **API pubbliche del catalogo**:
  modelli `POST https://api-us.ugnas.com/api/system/v2/sa/official/query/model` `{"type":"ugos-pro"}`;
  app di un modello `POST https://api-us.ugnas.com/api/system/v1/application/web/appInfo/list` `{"id":<modelId>}`;
  link a un pacchetto `GET https://api-us.ugnas.com/api/system/v1/ua/temp/link?appType=app&id=<appId>&model=<Modello>`
  (Docker del DH4300 Plus: `id=14607`).

---

## Creare un'app per UGOS (ricetta)

Tre strade, in ordine di pulizia **e** percorribilità sul DH2300.

### Strada A. Servizio utente senza root (in produzione)

1. Rootfs sul Mac: `ARCH=arm64 bash upk/build.sh`, poi
   `python3 upk/unpack.py telecamere-casa-1.0.0-arm64.upk --out /tmp/x`
   (o a mano: `sbin/node`, `sbin/ffmpeg`, `app/`).
2. Copia: `tar czf - -C /tmp/x/rootfs . | $NAS 'mkdir -p ~/telecamere-casa && tar xzf - -C ~/telecamere-casa'`.
3. `npm install` **sul NAS** (vedi B).
4. Unit in `~/.config/systemd/user/<nome>.service`:

   ```ini
   [Unit]
   Description=Telecamere Casa
   After=network-online.target

   [Service]
   Type=simple
   Environment=CONFIG_DIR=%h/.config/telecamere
   Environment=PORT=9765
   Environment=LOW_POWER=1
   Environment=LOW_POWER_SUBSTREAM=1
   Environment=FFMPEG_BIN=%h/telecamere-casa/sbin/ffmpeg
   WorkingDirectory=%h/telecamere-casa/app
   ExecStart=%h/telecamere-casa/sbin/node %h/telecamere-casa/app/server.js
   Restart=on-failure
   RestartSec=3

   [Install]
   WantedBy=default.target
   ```

5. `systemctl --user daemon-reload && systemctl --user enable --now <nome> && loginctl enable-linger $USER`
   (il linger **riesce senza root** e fa partire il servizio a ogni accensione).
6. Verifica come in B.2 e B.4.

Limiti: nessuna icona nell'App Center, accesso diretto alla porta.

### Strada B. Integrazione nel portale, con root

Fatta sopra la strada A per la sola esposizione.

1. **Proxy** `/etc/nginx/conf.d/<nome>.conf`:

   ```nginx
   location /telecamere/ {
       proxy_pass http://127.0.0.1:9765/;
       proxy_http_version 1.1;
       proxy_set_header Host $host;
       proxy_set_header X-Real-IP $remote_addr;
       proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
       proxy_set_header X-Forwarded-Proto $scheme;
       proxy_set_header X-Forwarded-Prefix /telecamere;
       proxy_set_header Connection "";
       proxy_buffering off;
       proxy_read_timeout 3600s;
       client_max_body_size 100m;
   }
   ```

   L'app deve essere **prefix-aware**: il server legge `X-Forwarded-Prefix` e inietta
   `window.__BASE__`, il client costruisce gli URL con `u()`, redirect e manifest rispettano
   il prefisso, dietro prefisso le porte extra degli stream collassano su un'unica origine.
   Asset delle pagine su percorsi profondi (`/share/<token>`) sempre **assoluti**.
2. **Registrazione** in `/ugreen/@appstore/<appId>/`: `config.json` (modello in
   `upk/rootfs-template/config.json`: `arch: "arm64"`, `appType: 1`, `daemon`, `serviceName`,
   `route: "/telecamere"`, `baseAccessInfo.portInfo.port: "9765"`, `i18n` it-IT ed en-US;
   `version.version` nel formato `1.0.0.0000` con `versionNum` coerente, altrimenti
   `invalid version`), `icon.png`, `init.d/`, `http.d/`, `.check-upk`. Icona anche in
   `/ugreen/static/icons/<appId>.png`.
3. **Database**: `INSERT INTO app_ports(appid, http_port, https_port, created_at, updated_at)`.
4. **Aperto**: icona sul desktop UGOS non confermata. Le app di terzi (`appType=1`)
   sembrano volere anche una riga in `app_users` con un account di sistema dedicato:
   non tentato senza un esempio reale da copiare.

Variante tutta-root: `upk/install-ssh.sh` (rootfs in `/opt/telecamere-casa`, script in
`/etc/sysconfig/`, unit di sistema, dati in `/var/ugreen/telecamere`). Usa `scp`: adattarlo prima.

### Strada C. Docker

Per i modelli con Docker (DXP, DH4300 Plus). Sul DH2300 richiede prima il pacchetto Docker
ufficiale del DH4300 Plus (firmato UGREEN, `id=14607`). Poi `ARCH=arm64 bash docker/build.sh`
→ `.tar` → App Center → Docker → Immagini → Importa, `network_mode: host`, volume `/config`.
Guida utente in `NAS-UGREEN.md`. **Mai provata sul DH2300** e senza vantaggi sulla strada A.

### Requisiti di un'app "buona cittadina"

- linux/**arm64**, binari nativi installati sul NAS;
- zero lavoro a riposo (heartbeat + reaper), salvo `keepWarm` scelto dall'utente;
- porta alta, dati nella home o in `/var/ugreen/<app>`, segreti `0600`;
- prefix-aware per il portale, PWA installabile dal telefono;
- utenti propri (Master / Moderatore / Visualizzatore), permessi per telecamera
  controllati **server-side**, credenziali NVR solo al Master.

---

## Il formato `.upk` e perché non si può produrre

```
UGREEN-PKG-V2-FORMAT
filesig:<len>:<firma RSA-2048 SHA-256 del payload>
userpub:<len>:<chiave "utente" SPKI DER b64>   usersig:<len>:<firmata dalla intermedia>
midpub:<len>:<chiave "intermedia">              midsig:<len>:<firmata dalla root CA UGREEN>
ico:<len>:<PNG>
ugb:<len>:<tar.gz con <appId>.ugb>
obj2:<len>:<16 byte: tag di autenticazione AEAD>
```

Il rootfs dentro `<appId>.ugb`: `config.json`, `.check-app`, `init.d/`, `nginx/`, `sbin/`, `app/`.

**Esito definitivo (11 agosto 2026):** l'App Center risponde
`cipher: message authentication failed` (`ugapp/appmaker/entry.go → loadUpk`). Il payload
ufficiale è **cifrato AEAD** con una chiave di UGREEN: senza di essa il pacchetto non viene
neppure decifrato e la catena di firme è irrilevante. **Non riprovare** con firme o header diversi.

Strumenti del repo ancora utili: `upk/build.sh` (rootfs completo arm64), `upk/unpack.py`
(estrae i nostri `.upk`, dei pacchetti ufficiali legge solo l'header), `upk/rootfs-template/`
(manifest, unit, preavvio, `location` nginx: il modello per una nuova app). `build.sh` e il
`Dockerfile` copiano solo `server.js auth.js`: per un rootfs aggiornato aggiungere
`detect.js notify.js devices.js ai.js ai-worker.js recovery.js models`.

---

## Vicoli ciechi già esplorati

| Tentativo | Esito | Data |
|---|---|---|
| `.upk` autofirmato dall'App Center | rifiutato: payload AEAD | 11 ago 2026 |
| `scp` / SFTP | bloccati da UGOS | 11 ago 2026 |
| appoggi in `/tmp` sul NAS | non scrivibile | 11 ago 2026 |
| password SSH o sudo da Claude Code | nessun TTY, errori fuorvianti | 11 ago 2026 |
| deploy in `~/telecamere-casa/` | fallisce in silenzio su `public/` | 20 ago 2026 |
| leggere gli account UGOS dall'app | impossibile | 11 ago 2026 |
| Chrome headless nella sandbox Bash del Mac | appeso; serve la sandbox disattivata | 3 set 2026 |
| estensione claude-in-chrome | "not connected" | 3 set 2026 |

---

## Regole

1. **Build sempre arm64.** Mai pacchetti o immagini amd64 per il DH2300.
2. **Segreti mai nel repository**, neppure privato: `config/`, `users.json`, `secret.key`,
   `notify.json`, `devices.json`. Il repository remoto resta privato.
3. **Dati personali mai in questo file**, nelle memorie condivise o nei commit.
4. **Commit locali di routine** (skill `commit-locale`); **push e deploy solo su richiesta**.
5. Deploy sempre completo: copia in `app/`, riavvio, `is-active`, health 200, sha256, `0600`.
6. Test locali verdi prima del deploy: server su porta libera con `CONFIG_DIR` temporanea e
   `TELEGRAM_API=http://127.0.0.1:1`; harness browser via CDP fuori sandbox.
7. Da root solo la parte portale, con backup; niente riavvii del NAS.
8. Segreti nel browser mascherati (`••••••`): un valore mascherato che torna significa "conserva".
9. Rispondere in italiano, con tutti gli accenti.

## Riferimenti

- `NAS-UGREEN.md` guida utente Docker; `upk/README.md` storia del formato e dei tre percorsi.
- `upk/rootfs-template/`, `docker/`, `Dockerfile`, `raspberry/install.sh`, `recovery.js`.
- Memorie di Claude: `project_nas_hardware`, `project_nas_ssh_install`,
  `project_nas_portal_integration`, `project_deploy_targets`, `project_nas_auth_and_idle`,
  `project_test_browser_headless`.

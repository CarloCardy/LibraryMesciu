# Checklist di security review

Da usare quando una modifica tocca: input utente, file system, rete, subprocess,
segreti/token, autenticazione o superfici esposte (IPC, HTTP, WebSocket).
Riporta solo vulnerabilità concrete e sfruttabili (file:riga + scenario + fix),
non mancanze generiche di hardening.

## Input e injection

- [ ] Input utente in subprocess: argomenti come array (mai stringhe in shell),
      separatore `--`/`--end-of-options` dove un valore può iniziare con `-`.
- [ ] Query SQL/NoSQL parametrizzate; niente concatenazione di input.
- [ ] Path costruiti da input: sanificati (niente `..`, path assoluti, symlink)
      e verificati dentro la cartella base attesa dopo `resolve`.
- [ ] Deserializzazione: mai `eval`/`pickle`/YAML unsafe su dati non fidati.

## Segreti e token

- [ ] Nessun segreto hardcoded, loggato o messo negli argv (visibili in `ps`):
      passare via env var o stdin.
- [ ] I token viaggiano solo verso gli host previsti (attenzione ai redirect:
      rimuovere l'Authorization se cambia host).
- [ ] I segreti non finiscono in messaggi d'errore, URL o payload verso terzi.

## Superfici esposte

- [ ] Endpoint nuovi (HTTP/WS/IPC): chi può chiamarli? Serve autenticazione?
      Un peer non autenticato può raggiungere funzioni pericolose?
- [ ] Dati esterni (manifest remoti, risposte API) trattati come NON fidati:
      validati campo per campo prima dell'uso.
- [ ] `openExternal`/apertura URL: solo schemi attesi (https), mai `file:` da input.

## File e permessi

- [ ] Scritture su disco: destinazione confinata, niente overwrite silenzioso di
      file di sistema o dell'utente fuori dallo scope.
- [ ] File temporanei con permessi corretti e cleanup su errore.

## Esito

Elenca i finding con severità (alta/media/bassa) e confidenza; se non c'è nulla
di concreto, dichiaralo esplicitamente («nessun finding sopra soglia»).

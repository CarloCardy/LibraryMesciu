# <Nome progetto>

> Istruzioni per Claude Code. Compila le sezioni e cancella ciò che non serve:
> qui va SOLO ciò che non si deduce dal codice (convenzioni, routine, vincoli).

## Panoramica

- **Cosa fa il progetto**: <una frase>.
- **Stack**: <linguaggi, framework, runtime>.
- **Entry point**: <es. `src/main.ts`, `app/server.py`>.

## Comandi

```bash
<gestore> install        # dipendenze
<comando dev>            # sviluppo locale
<comando build>          # build di produzione
<comando test>           # test veloci (unit)
<comando e2e>            # test end-to-end (se esistono)
```

## Testing

- Dopo modifiche a <area critica> lancia almeno `<comando test>`.
- Non dichiarare concluso un lavoro con test rossi: riporta l'errore e correggi.
- Nuovo comportamento → nuovo test; comportamento cambiato → aggiorna il test.

## Convenzioni di codice

- Stile: <linter/formatter e come lanciarlo>.
- Naming: <convenzioni particolari>.
- Errori: <come si gestiscono/loggano>.
- Commenti: spiega il PERCHÉ, non il cosa; commenta le scelte non ovvie.

## Routine automatiche

- **Commit**: commit locale a fine modifica completata (Conventional Commits).
  Niente push/PR di iniziativa: solo su richiesta esplicita.
- **Review**: prima di chiudere feature importanti valuta code review e, se si
  toccano input/file/rete/segreti, una security review.

## Vincoli e note

- <Cosa NON toccare (file generati, cartelle vendor, migrazioni…)>
- <Segreti e configurazione: dove vivono, cosa non va mai committato>
- <Particolarità di ambiente: OS, versioni bloccate, servizi esterni>

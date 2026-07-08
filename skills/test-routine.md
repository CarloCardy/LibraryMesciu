---
name: test-routine
description: Esegue la routine di test del progetto (unit → lint → e2e) e riporta gli esiti. Usala quando l'utente chiede di testare, verificare o validare una modifica, o prima di dichiarare concluso un lavoro.
---

# Routine di test

Obiettivo: nessun lavoro si dichiara «fatto» con test rossi.

## Passi

1. **Scopri i comandi di test del progetto**, in quest'ordine:
   - script in `package.json` (`test`, `test:*`, `lint`, `e2e`), `Makefile`, `justfile`;
   - sezione Testing di `CLAUDE.md` o `README.md`;
   - se non esiste nulla, dillo esplicitamente e proponi come impostare i test.
2. **Esegui prima i test veloci** (unit), poi lint/typecheck, poi gli e2e se esistono.
   In ambienti senza display salta i test che richiedono UI reale e dichiaralo.
3. **Riporta l'esito con i marcatori del progetto** (es. `PASS/FAIL`, exit code,
   nome del test rosso). Copia il messaggio d'errore rilevante, non tutto il log.
4. **Se un test fallisce**: correggi e riesegui SOLO il test fallito finché è verde,
   poi rilancia la suite completa una volta.
5. **Aggiorna i test quando cambi il comportamento**: nuova logica → nuovo test;
   comportamento modificato → aggiorna l'asserzione, mai cancellare il test per
   farlo passare.

## Regole

- Mai `sleep` fissi nei test: attendere sempre uno stato osservabile.
- Un test flaky va segnalato all'utente, non ritentato in silenzio più di una volta.
- Se tocchi codice di processo principale/backend, esegui almeno gli unit prima del commit.

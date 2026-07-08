---
name: commit-locale
description: Crea un commit locale ben fatto (Conventional Commits, messaggio che spiega il perché). Usala quando l'utente chiede di committare o alla fine di una modifica completata. Non pusha mai di sua iniziativa.
---

# Commit locale convenzionale

Obiettivo: una storia git leggibile, fatta di commit piccoli e autoesplicativi.
**Solo commit locali: niente push, branch remote o PR senza richiesta esplicita.**

## Passi

1. `git status` + `git diff` per rivedere COSA stai committando; escludi file
   estranei (build, lockfile toccati per sbaglio, segreti, `.env`).
2. Raggruppa: se la modifica contiene due cambi indipendenti, fai due commit.
3. Scrivi il messaggio nel formato Conventional Commits:
   - `feat(scope): cosa aggiunge` · `fix(scope): cosa corregge`
   - `refactor|docs|test|chore(scope): …`
   - prima riga ≤ 72 caratteri, all'imperativo, senza punto finale;
   - nel corpo spiega il PERCHÉ e le scelte non ovvie (il "cosa" lo dice il diff).
4. Prima del commit fai passare i test pertinenti (vedi la skill di test, se presente).
5. `git add` mirato (mai `git add .` alla cieca) → `git commit`.

## Regole

- Un commit che rompe i test non si crea: prima verde, poi commit.
- Mai riscrivere la storia già condivisa (`rebase`/`amend` solo su commit locali).
- Se l'utente ha convenzioni proprie in `CLAUDE.md`, quelle vincono su questa skill.

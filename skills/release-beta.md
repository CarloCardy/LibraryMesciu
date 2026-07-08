---
name: release-beta
description: Routine di rilascio a tre canali (alpha locale → beta → stabile) con tag semver. Usala quando l'utente chiede di pubblicare una versione, una beta o di promuovere a stabile. Chiede sempre conferma prima di taggare o pushare.
---

# Rilascio alpha → beta → stabile

Tre canali, decisi dal suffisso della versione semver:

| Canale  | Versione                    | Chi la riceve                     |
|---------|-----------------------------|-----------------------------------|
| alpha   | `X.Y.Z-beta.(N+1).alpha.M`  | solo build locali, mai pubblicate |
| beta    | `X.Y.Z-beta.N`              | tester con canale beta attivo     |
| stabile | `X.Y.Z`                     | tutti                             |

## Regole fondamentali

1. **Il tag combacia SEMPRE con la versione** del manifest del progetto
   (`package.json`, `pyproject.toml`, …): il bump fa parte dell'atto di rilascio.
2. **Le alpha restano locali**: build sperimentali, niente tag né push; la versione
   si posiziona appena sopra la base (`beta.N` → `beta.(N+1).alpha.M`, col punto).
3. **Le beta si taggano sul branch principale** dopo averlo riallineato alla base
   scelta CON l'utente (ultima stabile o ultima beta): chiedi, non decidere da solo.
4. **Squash pre-release (default)**: al momento di pubblicare la beta, i commit
   locali accumulati dall'ultima release si COMPATTANO in un solo commit sul
   branch principale, **senza chiudere i commit locali**: prima dello squash fai
   avanzare (fast-forward) la branch locale di lavoro sull'HEAD granulare, così
   la cronologia dettagliata resta consultabile. Verifica che l'albero compattato
   sia identico (`git diff <squash> <branch-locale>` vuoto). Si salta lo squash
   **solo su richiesta differente dell'utente**.
5. **La stabile si pubblica solo su richiesta esplicita** («promuovi a stabile»):
   togli il suffisso (`X.Y.Z`), tagga `vX.Y.Z`.
6. Prima di ogni tag: test verdi + changelog/note di rilascio aggiornate.
7. Dopo il push del tag: verifica che la CI di release completi per TUTTE le
   piattaforme previste; una release con artefatti parziali non è chiusa.

## Comandi tipici (npm)

```bash
# squash pre-release (cronologia granulare preservata sulla branch locale)
git branch -f <branch-locale> HEAD
git reset --soft origin/main && git commit   # un solo commit riassuntivo

npm version prepatch --preid=beta --no-git-tag-version   # X.Y.Z → X.Y.(Z+1)-beta.0
npm version prerelease --preid=beta --no-git-tag-version # beta.N → beta.N+1
npm version patch --no-git-tag-version                   # rimuove il suffisso → stabile
git tag vX.Y.Z[-beta.N] && git push origin <branch> --tags
```

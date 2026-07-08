# Checklist di code review

Rivedi il diff con questa checklist e riporta SOLO i problemi concreti trovati
(file:riga + spiegazione + proposta), in ordine di gravità. Niente osservazioni
di stile se il progetto ha già un linter che le copre.

## Correttezza

- [ ] Il codice fa ciò che la richiesta chiedeva (né di più, né di meno)?
- [ ] Casi limite: input vuoti/nulli, liste a zero/un elemento, errori di rete/IO.
- [ ] Le condizioni di errore sono gestite e comunicate (niente `catch` silenziosi)?
- [ ] Concorrenza: race su stato condiviso, doppie sottomissioni, cleanup mancanti.
- [ ] Le modifiche di comportamento sono retrocompatibili con dati/config esistenti?

## Semplicità e riuso

- [ ] Esisteva già una funzione/pattern nel codebase da riusare invece di duplicare?
- [ ] C'è codice morto, parametri inutilizzati, astrazioni premature da togliere?
- [ ] La soluzione è alla giusta "altitudine" (né hack puntuale, né framework generico)?

## Leggibilità

- [ ] Nomi chiari e coerenti con le convenzioni del progetto.
- [ ] I commenti spiegano il perché delle scelte non ovvie (e solo quelle).
- [ ] Funzioni brevi, un livello di astrazione per funzione.

## Test

- [ ] Il nuovo comportamento ha un test che fallirebbe senza la modifica?
- [ ] I test toccati restano significativi (niente asserzioni svuotate per "far passare")?
- [ ] La suite pertinente è stata eseguita ed è verde?

## Esito

Concludi con: **approvato** / **approvato con note** / **da correggere**, più
l'elenco dei punti bloccanti.

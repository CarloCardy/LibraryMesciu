---
name: html-docs
description: Produce documentazione, report, review e diagrammi come file .html autonomi e interattivi invece di muri di markdown. Usala quando devi creare un documento di consegna, un report di analisi, una code review, una ricerca o uno schema architetturale.
---

# Documentazione in HTML autonomo

Quando produci un **documento destinato a un umano** (report, analisi, code review,
ricerca, spiegazione di architettura, confronto di opzioni, deck di sintesi), crea un
**file `.html` autonomo e interattivo** invece di un file `.md`.

> Riferimento: "The Unreasonable Effectiveness of HTML"
> (https://thariqs.github.io/html-effectiveness/) — i formati visivi e interattivi
> comunicano informazioni complesse meglio di un muro di testo lineare.

## Regole del file

1. **Un solo file, autonomo**: CSS in un `<style>` e JS in uno `<script>` inline.
   **Zero dipendenze esterne** (niente CDN, font remoti, immagini linkate): il file deve
   aprirsi offline con un doppio click.
2. **Diagrammi = SVG inline**: architetture, flussi, timeline e grafici si disegnano con
   `<svg>` direttamente nel file. Niente ASCII-art per concetti spaziali.
3. **Struttura navigabile**:
   - `<details>/<summary>` per sezioni collassabili (approfondimenti, log, appendici);
   - tab per confrontare varianti o file (poche righe di JS);
   - indice iniziale con anchor link se il documento supera le 3 sezioni.
4. **Codice**: blocchi `<pre><code>` con un minimo di evidenziazione via CSS; per i
   confronti prima/dopo usa due colonne o tab, non blocchi in sequenza.
5. **Dati**: tabelle vere (`<table>`) con intestazioni; se utile, una riga di JS per
   ordinamento o filtro.
6. **Sobrietà**: palette leggibile anche in stampa, niente animazioni gratuite; il
   documento è informazione, non una demo.

## Quando restare su Markdown

Il `.md` resta obbligatorio dove **uno strumento lo richiede**: `CLAUDE.md`, `SKILL.md`
e file agente, `README` destinati a GitHub, changelog e file letti da parser. In quei
casi non convertire.

## Come consegnare

Salva il file nella cartella concordata del progetto (es. `docs/` o la radice), con nome
parlante (`report-refactor-auth.html`, `review-pr-42.html`) e — se il progetto ha la
cartella `.Mesciu/` — consideralo un artefatto del progetto: versionalo col repo.

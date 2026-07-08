# LibraryMesciu

Libreria ufficiale **Smart AI Programming** di [Mesciu](https://github.com/CarloCardy/Mesciu):
preset `.md` (skill, agenti, routine, template) pronti da importare nei progetti direttamente
dall'app. Questo repository è la **sorgente remota** della libreria: si aggiorna in modo
indipendente dalle release di Mesciu — un commit qui è subito live per tutte le installazioni.

## Struttura

```
index.json      manifest dei preset NATIVI (i file .md vivono in questo repo)
links.json      manifest dei LINK community (puntano a .md di altri repository)
skills/         file .md dei preset di tipo skill
agents/         file .md dei preset di tipo agente
routine/        checklist e routine operative
templates/      template di partenza (es. CLAUDE.md)
```

## Schema di `index.json` (preset nativi)

```json
{
  "version": 1,
  "items": [
    {
      "id": "identificativo-unico",
      "name": "Nome mostrato nella libreria",
      "category": "skill | agent | routine | template | doc",
      "description": "A cosa serve, in una o due righe.",
      "tags": ["ricerca", "parole", "chiave"],
      "file": "skills/nome-file.md",
      "url": "https://github.com/CarloCardy/LibraryMesciu/blob/main/skills/nome-file.md",
      "install": { "type": "skill | agent | file", "slug": "nome-skill", "suggestedPath": "" }
    }
  ]
}
```

- `file` è il percorso **relativo alla radice di questo repo**.
- `url` è opzionale (pulsante "apri su GitHub" nell'app); solo `https`.
- `install.type`: `skill` → `.claude/skills/<slug>/SKILL.md`, `agent` → `.claude/agents/<slug>.md`,
  `file` → `suggestedPath` (o il nome del file).

## Schema di `links.json` (link community)

Come sopra, ma al posto di `file` c'è **`url`**: un link `https` a un file **`.md` o `.html`** di un altro
repository (forma `blob` o `raw` di GitHub, o un URL https generico). L'app mostra la voce nella
stessa libreria con il badge del creatore e la installa scaricando il file dal link.

```json
{
  "id": "creatore-nome",
  "name": "Nome",
  "category": "skill",
  "description": "…",
  "tags": [],
  "url": "https://github.com/creatore/repo/blob/main/percorso/file.md",
  "install": { "type": "skill", "slug": "nome" }
}
```

## Contribuire

Le voci di `links.json` si possono modificare anche **dall'app** (Impostazioni → Avanzate →
Modalità Developer): l'editor è visibile solo a chi ha permessi di scrittura su questo repository
e salva committando qui. Per i preset nativi: aggiungere il file `.md` nella cartella giusta e la
voce in `index.json` (id unico, path relativo corretto).

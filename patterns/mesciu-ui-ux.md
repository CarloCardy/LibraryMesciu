# Pattern UI/UX di Mesciu

Linguaggio visivo dell'app Mesciu (Electron + React + Tailwind, tema scuro). Carica questo
file quando devi costruire o modificare interfaccia in stile Mesciu: componenti nuovi devono
sembrare nati nello stesso posto di quelli esistenti.

## Palette e gerarchia

- **Sfondi**: `bg-slate-950` (aree terminale/contenuto), `bg-slate-900` (pannelli e modali),
  `bg-slate-800`/`bg-slate-800/40` (card, input, superfici interattive).
- **Bordi**: `border-slate-800` (divisori), `border-slate-700` (contenitori), più chiari in
  hover (`hover:border-slate-500`).
- **Testo**: scala slate — `text-white`/`slate-100` (titoli), `slate-300` (corpo), `slate-400/500`
  (secondario), `slate-600` (placeholder/empty state). Percorsi, versioni e comandi in `font-mono`.
- **Colori semantici**: blu = azione primaria (`bg-blue-600 hover:bg-blue-500`); emerald =
  successo/positivo; red = distruttivo; amber = warning/build; sky = variante secondaria di
  un'azione blu (es. "Ricarica" vs "Carica"); violet = fork/speciale.
- **Micro-tipografia**: etichette e badge `text-[9px]`/`text-[10px]`, testi di card `text-[11px]`,
  titoli di sezione `text-xs`/`text-sm font-semibold`.

## Componenti ricorrenti

**Modale**: overlay `fixed inset-0 bg-black/60 backdrop-blur-sm flex items-center justify-center`
con `onClick={onClose}`; contenitore `bg-slate-900 border border-slate-700 rounded-2xl shadow-2xl`
con `onClick={e => e.stopPropagation()}`; header con titolo (icona + `text-sm font-semibold`),
sottotitolo `text-[11px] text-slate-500 truncate` e ✕ in alto a destra. Le sotto-modali
(conferme, form) usano z-index maggiore.

**Pulsanti pill**: `inline-flex items-center gap-1 px-2 py-1 text-[10px] rounded-md
transition-colors disabled:opacity-40 flex-shrink-0` + colore semantico. Con icona lucide 11-12px;
lo stato busy sostituisce l'icona con `<Loader2 className="animate-spin" />`.

**Chip/badge**: `px-1.5 py-px text-[9px] rounded border` + tinta al 15% (`bg-blue-500/15
text-blue-300 border-blue-500/30`). Ogni categoria/stato ha la sua tinta, definita UNA volta in
una mappa `*_META` e riusata ovunque (chip, copertine, icone).

**Card record (lista)**: `bg-slate-800/40 border border-slate-700 rounded-lg px-3 py-2.5` —
riga superiore con nome (bottone `truncate min-w-0`), chip, spaziatore `flex-1`, azioni
`flex-shrink-0`; sotto, descrizione `text-[10px] text-slate-500` e meta-riga `text-[9px]`.

**Card poster (galleria)**: copertina `h-32` con gradiente di categoria
(`bg-gradient-to-br from-<tinta>-600/50 via-<tinta>-900/40 to-slate-950`) e icona-artwork
grande (`size={44}`, `group-hover:scale-110`); badge sovrapposti alla copertina
(tipo in alto a sinistra, stato in alto a destra, con `backdrop-blur-sm`); corpo con titolo,
riga autore, descrizione `line-clamp-2`; footer azioni separato da `border-t`. Hover della card:
`hover:border-slate-400 hover:shadow-lg hover:-translate-y-0.5`.

**Toolbar icone**: bottoni quadrati `w-8 h-7 rounded-lg text-slate-400 hover:bg-slate-800`
con colore semantico in hover (`hover:text-emerald-400`…), sempre con `title=` descrittivo.

## Stati e feedback

- **Loading**: `<Loader2 size={12} className="animate-spin" />` + testo `text-slate-600`
  centrato con `py-8`, mai spinner a schermo intero per operazioni locali.
- **Empty state**: frase breve `text-[11px] text-slate-600 text-center py-8`, possibilmente
  con l'azione per uscirne ("Nessun link: aggiungi il primo.").
- **Errori inline**: `text-red-300 bg-red-500/10 border border-red-500/20 rounded-lg px-3 py-2`.
- **Notice ok/errore**: stesso blocco con tinta emerald per il successo; testo che dice cosa è
  successo E cosa fare dopo ("…: rivedi e invia.").
- **Azioni distruttive**: mai dirette — sempre modale di conferma dedicata con bottone rosso e
  descrizione di cosa verrà rimosso (percorso in `font-mono`).

## Regole trasversali

1. Tooltips (`title=`) su ogni icona e azione non ovvia; testi UI in italiano.
2. Testi che possono crescere: `truncate` + `min-w-0` sul testo, `flex-shrink-0` sulle azioni.
3. `transition-colors` (o `transition-all` breve) su ogni elemento interattivo.
4. Le preferenze di vista si persistono in `localStorage` con prefisso `cr_`.
5. Niente dipendenze UI nuove: lucide-react per le icone, Tailwind per tutto il resto.

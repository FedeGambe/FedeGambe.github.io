# Come aggiungere un progetto al sito

Il sito [progetti.federicogamberini.it](https://progetti.federicogamberini.it) mostra una card per ogni voce di `progetti.json`.
Ogni card legge i suoi dati direttamente dalla repo del progetto:

```
FedeGambe.github.io/progetti.json        →  { "repo": "Nome_Repo" }
                                              │
                                              ▼
Nome_Repo/docs/progetto.json             →  titolo, descrizione, tag, anno, link "Apri"
Nome_Repo/docs/img/cover_verticale.svg   →  immagine della card
Nome_Repo/docs/index.html                →  pagina pubblicata su /Nome_Repo/
```

Quindi, per un progetto nuovo, quasi tutto il lavoro si fa **nella repo del progetto**. Qui basta aggiungere una riga.

---

## Prima volta: installare le skill

Le skill sono in [skill-vetrina](https://github.com/FedeGambe/skill-vetrina). Si installano una volta per PC:

```
git clone https://github.com/FedeGambe/skill-vetrina
cd skill-vetrina/crea-copertina
pip install -e .
python -m copertine installa-skill
cp -r ../crea-pagina-html     ~/.claude/skills/crea-pagina-html
cp -r ../crea-dashboard-html  ~/.claude/skills/crea-dashboard-html
cp -r ../crea-scheda-progetto ~/.claude/skills/crea-scheda-progetto
```

Poi riavvia Claude Code.

---

## I passi

Apri Claude Code **dentro la cartella del progetto** (es. `~/Nome_Repo`) e segui l'ordine.

### 1. Copertina: skill `crea-copertina`

> "fammi una copertina per questo progetto"

Crea in `docs/img/` tre file con questi nomi esatti:

| File | Uso |
|---|---|
| `cover_orizzontale.svg` | cover della pagina HTML su schermo orizzontale (1600×900) |
| `cover_verticale.svg` | **immagine della card sul sito**, e cover su mobile (1080×1350) |
| `favicon.svg` | icona della scheda del browser (64×64, angoli arrotondati) |

Puoi dare dei riferimenti (screenshot, link) e chiedere modifiche finché ti va bene. Ogni copertina deve essere diversa da quelle già fatte.

### 2. Scheda: skill `crea-scheda-progetto`

> "crea la scheda del progetto"

Di solito parte da sola insieme alle altre skill. Crea `docs/progetto.json`:

```json
{
    "repo": "Nome_Repo",
    "t": "Titolo leggibile",
    "d": "Una frase su cosa fa il progetto (max ~110 caratteri).",
    "tags": ["Finanza", "Scraping"],
    "y": 2026,
    "live": "/Nome_Repo/",
    "img": "img/cover_verticale.svg"
}
```

- `repo`: nome **esatto** della repo su GitHub. Maiuscole comprese: `/When_to_buy_iPhone/` funziona, `/when-to-buy-iphone/` dà 404.
- `tags`: da 2 a 4, con l'iniziale maiuscola. Diventano anche i filtri del sito.
- `live`: il link del pulsante "Apri". Resta `""` finché non c'è una pagina HTML. Vale `/Nome_Repo/` per `docs/index.html` e `/Nome_Repo/dashboard.html` per una dashboard. Per un sito esterno usa l'URL completo (es. Wally: `https://wally.federicogamberini.it/`).
- `img`: sempre `img/cover_verticale.svg`.

### 3. Pagina o dashboard HTML

Scegline una (o entrambe):

| Skill | Cosa chiedere | Cosa produce | Quando usarla |
|---|---|---|---|
| `crea-pagina-html` | "fai la pagina HTML di questo repo" | `docs/index.html` | presentazione o riassunto del progetto: cover a tutto schermo, indice, capitoli, grafici |
| `crea-dashboard-html` | "fai una dashboard / un simulatore" | `docs/dashboard.html` (o `docs/index.html` se è l'unica pagina) | pagina interattiva: controlli a sinistra, risultato e grafico a destra |

Alla fine la skill scrive da sola il campo `live` in `docs/progetto.json`. Prima di pubblicare, controlla la pagina nel browser: tema chiaro e scuro, e larghezza da telefono.

### 4. Pubblicare la repo del progetto

```
git add docs
git commit -m "Pagina, copertina e scheda progetto"
git push
```

Poi, **solo la prima volta**, su GitHub vai in *Settings → Pages* e scegli *Branch* `main`, cartella `/docs`.
La pagina sarà su `https://progetti.federicogamberini.it/Nome_Repo/`. Il dominio è già quello del sito, non serve configurare altro.

Aggiorna anche le informazioni della repo, cioè descrizione, tag e sito:

```
gh repo edit FedeGambe/Nome_Repo \
  --description "Una frase su cosa fa" \
  --homepage https://progetti.federicogamberini.it/Nome_Repo/ \
  --add-topic data-science,python
```

### 5. Aggiungere la card al sito (questa repo)

In `progetti.json` aggiungi una voce con il solo nome della repo:

```json
[
    { "repo": "Master_s_thesis_Data_science", "t": "...", "...": "..." },
    { "repo": "Wally" },
    { "repo": "Nome_Repo" }
]
```

- L'ordine nel file è l'ordine delle card. La prima è "N° 01".
- **Attenzione alle virgole.** Va una virgola tra una voce e l'altra, mai dopo l'ultima. Una virgola di troppo rompe tutto il file, e il sito mostra "Impossibile caricare i progetti".
  Per controllare: `python3 -m json.tool progetti.json`, che non deve dare errori.
- Se la cover è chiara (fondo chiaro), aggiungi il nome della repo alla lista `LIGHT` in `index.html`. Così il testo della card diventa nero.

Poi fai commit e push. GitHub Pages aggiorna il sito in circa un minuto.

### 6. Verificare

- Apri https://progetti.federicogamberini.it e ricarica con **Cmd+Shift+R**.
- Controlla la nuova card: immagine, titolo, tag, e che il pulsante "Apri" porti alla pagina giusta.
- Le schede si leggono da `raw.githubusercontent.com`, che tiene in cache i file **fino a 5 minuti**. Se hai appena cambiato `progetto.json`, aspetta un attimo.

---

## Cosa aggiornare, e dove

| Voglio cambiare… | File da modificare | Repo |
|---|---|---|
| titolo, descrizione, tag, anno della card | `docs/progetto.json` | progetto |
| link del pulsante "Apri" | `live` in `docs/progetto.json` | progetto |
| immagine della card | `docs/img/cover_verticale.svg` | progetto |
| contenuto della pagina del progetto | `docs/index.html` / `docs/dashboard.html` | progetto |
| aggiungere, togliere o riordinare le card | `progetti.json` | questa |
| testo nero su cover chiara | lista `LIGHT` in `index.html` | questa |
| grafica, testi o struttura del sito | `index.html` | questa |
| descrizione, tag e link della repo su GitHub | `gh repo edit` (o la rotella ⚙ di *About*) | progetto |

`progetti.json` accetta anche una voce scritta per intero (con `t`, `d`, `tags`, `y`, `live`, `img` e la cover in `img/` di questa repo). Una voce così non legge `docs/progetto.json` e va modificata qui. Meglio evitarla: oggi tutte le voci sono `{ "repo": ... }`.

---

## Nota su `python -m copertine pubblica`

Le skill citano anche il comando `python -m copertine pubblica`. Usalo con attenzione: **riscrive tutto `progetti.json`** prendendo le voci da `skill-vetrina/crea-copertina/fatte/`, che è una cartella solo locale. Le voci complete prendono il posto di quelle `{ "repo": ... }`, e il comando cancella da `img/` le cover che non trova. Dopo averlo lanciato, prima di fare push guarda il diff (`git diff progetti.json img/`).
Per un progetto nuovo basta il passo 5 a mano.

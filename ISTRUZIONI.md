# Come aggiungere un progetto al sito

Il sito [progetti.federicogamberini.it](https://progetti.federicogamberini.it) mostra una card per ogni voce di `progetti.json`, divise in tre gruppi: **Strumenti** (da usare), **Analisi** (da leggere), **Studio** (tesi e ricerca).
Ogni card legge i suoi dati direttamente dalla repo del progetto:

```
FedeGambe.github.io/progetti.json        →  { "repo": "Nome_Repo", "tipo": "analisi", "formato": "Pagina" }
                                              │
                                              ▼
Nome_Repo/docs/progetto.json             →  titolo, descrizione, tag, anno, link "Apri"
Nome_Repo/docs/img/cover_sfondo.svg + cover_soggetto.svg  →  immagine della card
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
| `cover_orizzontale.svg` | cover della pagina HTML, **e immagine della card e della cover su telefono** (1600×900, grafica nella parte bassa) |
| `cover_verticale.svg` | formato verticale per i social (1080×1350); il sito non la usa più |
| `favicon.svg` | icona della scheda del browser (64×64, angoli arrotondati) |
| `cover_sfondo.svg` | **solo il fondo** della cover (colore, sfumatura, grafica leggera, grana), senza il soggetto. Su telefono riempie tutta la cover e la card, titolo compreso |
| `cover_soggetto.svg` | **solo il soggetto** (la B di Bitcoin, l'auto, l'icona di Wally…) su fondo trasparente, con il riquadro stretto attorno al disegno. Su telefono va sotto il titolo |
| `banner_sfondo.svg`, `banner_soggetto.svg` | solo per le **dashboard**: i due livelli del banner in testa alla pagina (fondo; soggetto a destra su desktop, sotto il titolo su telefono). Di solito sono una copia di `cover_sfondo.svg` e `cover_soggetto.svg` |

**Sfondo e soggetto sono la fonte**: `cover_orizzontale.svg` e `cover_verticale.svg` si rigenerano da loro con `python skill-vetrina/crea-copertina/componi.py docs/img` (fondo su tutta la tela, soggetto nella metà bassa). Per spezzare una cover esistente nei due livelli: `livelli.py` e poi `livelli_bbox.js`, che stringe il riquadro del soggetto e ne scrive larghezza e altezza.

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
- `tags`: da 2 a 4, con l'iniziale maiuscola. Sulla card se ne vedono al massimo 3 (solo su desktop) e servono alla ricerca.
- `live`: il link del pulsante "Apri". Resta `""` finché non c'è una pagina HTML. Vale `/Nome_Repo/` per `docs/index.html` e `/Nome_Repo/dashboard.html` per una dashboard. Per un sito esterno usa l'URL completo (es. Wally: `https://wally.federicogamberini.it/`).
- `colore` (facoltativo): il colore del progetto, in esadecimale (es. `"#2f6b52"`). Colora l'etichetta del formato e il bordo della card al passaggio del mouse; se manca si usa l'arancio del sito. Prendilo dalla copertina e che non sia troppo scuro: sta sopra un velo nero.
- `codice` (facoltativo): metti `false` per togliere il pulsante "Codice" dalla card (es. `simulatori`). Se manca, il pulsante c'è.
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

In `progetti.json` aggiungi una voce con il nome della repo, il gruppo e cosa si apre:

```json
[
    { "repo": "simulatori", "tipo": "strumento", "formato": "Dashboard" },
    { "repo": "Wally", "tipo": "strumento", "formato": "App" },
    { "repo": "Nome_Repo", "tipo": "analisi", "formato": "Pagina" }
]
```

- `tipo`: `strumento` (dashboard, simulatori, app: si usano), `analisi` (pagine da leggere) o `studio` (tesi). Se manca vale `analisi`.
- `formato`: cosa si apre cliccando, scritto sopra il titolo della card (`Pagina`, `Dashboard`, `App`, `Pagina + dashboard`). Se manca vale `Pagina`.
- L'ordine nel file è l'ordine delle card dentro ogni gruppo.
- **Attenzione alle virgole.** Va una virgola tra una voce e l'altra, mai dopo l'ultima. Una virgola di troppo rompe tutto il file, e il sito mostra "Impossibile caricare i progetti".
  Per controllare: `python3 -m json.tool progetti.json`, che non deve dare errori.

Poi fai commit e push. GitHub Pages aggiorna il sito in circa un minuto.

### Come appare sul telefono

- **Card su desktop:** `cover_sfondo.svg` fa da fondo a tutta la card, `cover_soggetto.svg` sta in alto (16:9); sotto, in bianco su un velo scuro, formato e anno, titolo, descrizione su 2 righe, tag e pulsante Codice. Tutta la card apre la pagina (`live`, o la repo se `live` è vuoto).
- **Card su telefono:** in riga, soggetto a sinistra e testo a destra, sullo stesso fondo; niente tag.
- **Cover chiara** (es. la tesi, fondo beige): aggiungi `"chiara": true` alla sua voce in `progetti.json`, così il testo della card diventa scuro.
- **Pagine su desktop:** cover a tutto schermo con `cover_sfondo.svg` come fondo; i testi in alto, il soggetto parte da metà schermo (o dopo i testi, se sono più lunghi).
- **Pagine su telefono:** la cover non è a tutto schermo. `cover_sfondo.svg` copre tutta la cover; solo il titolo sta prima della grafica, poi il soggetto, con testo introduttivo e tag sopra la sua parte bassa su una sfumatura. Niente logo in basso a destra. Serve `<img class="soggetto" src="img/cover_soggetto.svg" alt="">` dentro la `<section class="copertina">`; per i soggetti a tutta larghezza (strisce) aggiungi la classe `largo`. Lo fanno i blocchi `COVER A DUE LIVELLI` (pagine) e `FONT MOBILE` (dashboard) dei template delle skill. Senza i due livelli la pagina usa la cover orizzontale intera.

### 6. Verificare

- Apri https://progetti.federicogamberini.it e ricarica con **Cmd+Shift+R**.
- Controlla la nuova card: gruppo giusto, immagine, titolo, formato, e che il clic sulla card porti alla pagina giusta.
- Prova anche su telefono (o con la finestra stretta): card in riga con la miniatura, e un tocco sulla card apre la pagina.
- Le schede si leggono da `raw.githubusercontent.com`, che tiene in cache i file **fino a 5 minuti**. Se hai appena cambiato `progetto.json`, aspetta un attimo.

---

## Cosa aggiornare, e dove

| Voglio cambiare… | File da modificare | Repo |
|---|---|---|
| titolo, descrizione, tag, anno della card | `docs/progetto.json` | progetto |
| link della card | `live` in `docs/progetto.json` | progetto |
| colore del formato e del bordo al passaggio del mouse | `colore` in `docs/progetto.json` | progetto |
| nascondere il pulsante "Codice" | `"codice": false` in `docs/progetto.json` | progetto |
| immagine della card | `docs/img/cover_sfondo.svg` + `cover_soggetto.svg` | progetto |
| contenuto della pagina del progetto | `docs/index.html` / `docs/dashboard.html` | progetto |
| aggiungere, togliere o riordinare le card | `progetti.json` | questa |
| gruppo della card e cosa si apre | `tipo` e `formato` in `progetti.json` | questa |
| testo scuro su cover chiara | `"chiara": true` in `progetti.json` | questa |
| grafica, testi o struttura del sito | `index.html` | questa |
| descrizione, tag e link della repo su GitHub | `gh repo edit` (o la rotella ⚙ di *About*) | progetto |

`progetti.json` accetta anche una voce scritta per intero (con `t`, `d`, `tags`, `y`, `live`, `img` e la cover in `img/` di questa repo). Una voce così non legge `docs/progetto.json` e va modificata qui. Meglio evitarla: oggi tutte le voci sono `{ "repo": ... }`.

---

## Nota su `python -m copertine pubblica`

Le skill citano anche il comando `python -m copertine pubblica`. Usalo con attenzione: **riscrive tutto `progetti.json`** prendendo le voci da `skill-vetrina/crea-copertina/fatte/`, che è una cartella solo locale. Le voci complete prendono il posto di quelle `{ "repo": ... }`, e il comando cancella da `img/` le cover che non trova. Dopo averlo lanciato, prima di fare push guarda il diff (`git diff progetti.json img/`).
Per un progetto nuovo basta il passo 5 a mano.

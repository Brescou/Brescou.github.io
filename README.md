# brescou.github.io

Portfolio — Astro, dark minimal. Déployé automatiquement sur GitHub Pages.

## Setup (une seule fois)

1. Crée un repo GitHub nommé exactement **`Brescou.github.io`** (public).
2. Push ce dossier dessus :
   ```bash
   git init && git add -A && git commit -m "init: portfolio"
   git branch -M main
   git remote add origin git@github.com:Brescou/Brescou.github.io.git
   git push -u origin main
   ```
3. Dans le repo GitHub : **Settings → Pages → Source → GitHub Actions**.
4. C'est tout. Chaque push sur `main` rebuild et redéploie le site à
   https://brescou.github.io (2-3 min).

## Dev local

```bash
npm install
npm run dev      # http://localhost:4321
npm run build    # vérifie que tout compile
```

## Structure

```
src/
├── pages/index.astro        ← page principale (hero, work, about, contact)
├── pages/blog/              ← blog (masqué tant que SHOW_BLOG = false)
├── layouts/Base.astro       ← layout global + nav + SHOW_BLOG
├── styles/global.css        ← design tokens (couleurs, typo)
├── data/projects.json       ← liste des projets (édite ici)
└── content/blog/            ← articles markdown
```

## Publier ton premier article de blog

1. Renomme `src/content/blog/_example.md` (enlève le `_`), édite-le,
   passe `draft: false`.
2. Dans `src/layouts/Base.astro`, passe `SHOW_BLOG = true`.
3. Push. Le lien "Writing" apparaît dans la nav.

## Personnaliser

- **Projets** : `src/data/projects.json`
- **Couleurs/typo** : variables CSS dans `src/styles/global.css`
- **LinkedIn** : mets ton vrai URL dans `src/layouts/Base.astro` (footer)

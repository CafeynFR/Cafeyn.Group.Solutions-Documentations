# Cafeyn Group Solutions — Documentations

Documentation publique des stacks CGS, publiée via GitHub Pages :
https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/

## Structure

```
index.html                  Portail d'accueil (liste des stacks)
sdk-web/index.html          Documentation du SDK Web (dont la section « Télécharger la version vanilla »)
sdk-web/downloads/vanilla/latest/   Dernière version du SDK vanilla (core/ + components/)
sdk-web/downloads/vanilla/vX.Y.Z/   Versions archivées
backend/index.html          Documentation Backend (à venir)
ios/index.html              Documentation iOS (à venir)
android/index.html          Documentation Android (à venir)
tools/sdk-repo/             Workflow à copier dans le repo du SDK
.nojekyll                   Désactive Jekyll (fichiers servis tels quels)
```

Chaque documentation est un fichier HTML autonome : aucun build n'est nécessaire.

## Mettre à jour une documentation

1. Modifier le fichier `<stack>/index.html` (dans Claude, un éditeur, ou Claude Code directement dans ce repo).
2. Ouvrir une PR (ou pousser sur `main`).
3. GitHub Pages redéploie automatiquement en 1 à 2 minutes.

## Ajouter une nouvelle stack

1. Remplacer le `index.html` du dossier concerné par la documentation.
2. Dans le `index.html` racine, transformer la ligne `<span class="soon">` de la stack en `<a href="dossier/">` et passer son statut à « Disponible ».

## Publier une nouvelle version du SDK vanilla

Automatique : le workflow `tools/sdk-repo/publish-to-docs.yml`, installé dans le repo du SDK,
copie le build dans `sdk-web/downloads/vanilla/vX.Y.Z/` et `sdk-web/downloads/vanilla/latest/` à chaque tag `vX.Y.Z`.

Manuel : copier les dossiers `core/` et `components/` dans `sdk-web/downloads/vanilla/vX.Y.Z/` et `sdk-web/downloads/vanilla/latest/`, puis commit.
Penser à mettre à jour le badge de version et l'exemple d'URL figée dans `sdk-web/index.html`.

URLs d'intégration :
- Pages : `https://cafeynfr.github.io/Cafeyn.Group.Solutions-Documentations/sdk-web/downloads/vanilla/latest/core/index.iife.js`
- CDN (jsDelivr, version figée) : `https://cdn.jsdelivr.net/gh/CafeynFR/Cafeyn.Group.Solutions-Documentations@main/sdk-web/downloads/vanilla/vX.Y.Z/core/index.iife.js`

## Rappel

Ce repo est **public** : ne jamais y committer de clés, tokens, URLs internes ou données clients.

# Amazon Reviews Scraper - Analyseur d'Avis

**Workflow ID:** `Jl8qD1CMR9p2kwUH`
**Statut:** Actif
**Version:** 2.0
**Dernière mise à jour:** 2026-02-05

## Description

Ce workflow n8n analyse les avis clients Amazon et génère une synthèse intelligente via OpenAI (GPT-4o-mini). L'utilisateur uploade un fichier .txt contenant le HTML de la page des avis, et reçoit une analyse complète.

## Fonctionnalités

- Interface web simple (formulaire d'upload)
- Extraction automatique des avis Amazon (titre, note, auteur, date, texte)
- Parsing JavaScript avec expressions régulières
- Synthèse automatique avec OpenAI :
  - Note moyenne calculée
  - Points positifs / négatifs identifiés
  - Recommandation d'achat (Acheter / Mitigé / À éviter)
- Page de résultat formatée

## Architecture

```
┌───────────────────┐     ┌──────────────────┐     ┌───────────────┐
│ Formulaire Upload │────>│ Préparer données │────>│ Parser Avis   │
│ (Form Trigger)    │     │ (Code)           │     │ JS (Code)     │
└───────────────────┘     └──────────────────┘     └───────┬───────┘
                                                          │
                                                          ▼
┌───────────────────┐     ┌──────────────────┐     ┌───────────────┐
│ Page Résultat     │<────│ Synthèse OpenAI  │<────│ Formater JSON │
│ (Form)            │     │ (GPT-4o-mini)    │     │ (Code)        │
└───────────────────┘     └──────────────────┘     └───────────────┘
```

**6 nodes** | **Pipeline linéaire** | **~10-15 secondes d'exécution**

## Utilisation

### Étape 1 : Obtenir le HTML Amazon

1. Ouvrir la page des avis Amazon du produit
   Ex: `https://www.amazon.fr/product-reviews/B089VNJSXR`
2. Appuyer sur `Ctrl+U` pour afficher le code source
3. `Ctrl+A` pour tout sélectionner, puis `Ctrl+C` pour copier
4. Coller dans un fichier `.txt` et sauvegarder

### Étape 2 : Analyser les avis

1. Accéder au formulaire : **https://<VOTRE_INSTANCE>.app.n8n.cloud/form/amazon-reviews-form**
2. Uploader le fichier `.txt`
3. (Optionnel) Renseigner l'URL du produit
4. Cliquer sur "Submit"
5. Attendre le résultat (~10-15 secondes)

## Exemple de résultat

```
## Résultat de l'analyse

**Note moyenne:** 4.7/5
**Nombre d'avis analysés:** 10

### Points positifs
- Robustesse et bonne qualité des matériaux
- Hermétique et facile à nettoyer
- Variété de tailles pour différents besoins
- Pratique pour un usage quotidien

### Points négatifs
- Aucun point négatif mentionné

### Résumé
Les utilisateurs apprécient fortement ces boîtes de repas
pour leur robustesse, leur qualité et leur praticité.

### Recommandation
**Acheter**
```

## Credentials requis

| Service | ID | Utilisation |
|---------|-----|-------------|
| OpenAI API | `<CREDENTIAL_ID>` | Synthèse des avis (GPT-4o-mini) |

## Fichiers du projet

| Fichier | Description |
|---------|-------------|
| `README.md` | Documentation principale (ce fichier) |
| `ARCHITECTURE.md` | Détails techniques de chaque node |
| `API.md` | Documentation de l'ancienne API webhook |
| `TODO.md` | Roadmap et améliorations prévues |
| `workflow.json` | Export du workflow n8n |

## Limitations

- **Taille du fichier** : Le HTML doit faire moins de ~500KB pour une performance optimale
- **Format** : Uniquement les pages d'avis Amazon (structure HTML spécifique)
- **Langue** : Optimisé pour Amazon.fr (français)

## Changelog

### v2.0 (2026-02-05)
- Remplacement du webhook par un formulaire web
- Suppression du mode URL (bloqué par Amazon)
- Interface utilisateur simplifiée
- Page de résultat intégrée

### v1.0 (2026-02-04)
- Version initiale avec webhook
- Support HTML direct et URL (URL bloquée par CAPTCHA)

## Auteur

**Aziz Goumiri**

## Repository

https://github.com/Azgoum/amazon-reviews-scraper

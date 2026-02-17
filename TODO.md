# Roadmap - Amazon Reviews Scraper

## Statut actuel

| Fonctionnalité | Statut | Notes |
|----------------|--------|-------|
| Mode HTML direct | OK | Fonctionne avec HTML < 150KB |
| Mode URL (scraping) | BLOQUE | CAPTCHA Amazon |
| Parser JavaScript | OK | Extraction des 5 champs principaux |
| Synthèse OpenAI | OK | GPT-4o-mini |
| Réponse webhook | OK | Format JSON structuré |

---

## En cours

### Intégration Browserless.io

**Objectif:** Permettre le scraping automatique des URLs Amazon en contournant les protections anti-bot.

**Étapes:**
- [x] Création compte Browserless.io
- [x] Obtention API Key
- [ ] Créer credential "Header Auth" dans n8n
- [ ] Remplacer HTTP Request par appel API Browserless `/content`
- [ ] Configurer Puppeteer pour rendu JavaScript
- [ ] Tester avec URL Amazon
- [ ] Gérer les cas d'échec (retry, fallback)

**Documentation Browserless.io:**
- Endpoint: `https://chrome.browserless.io/content`
- Auth: Header `Authorization: Bearer <API_KEY>`

---

## Backlog

### Priorité Haute

#### 1. Pré-traitement HTML automatique
- Extraire automatiquement la section `#cm_cr-review_list` du HTML
- Réduire la taille de ~584KB à ~150KB
- Permettre l'envoi du HTML complet sans erreur

#### 2. Pagination des avis
- Actuellement : seulement les 10 premiers avis
- Objectif : supporter la pagination Amazon (`?pageNumber=2`)
- Collecter jusqu'à 50-100 avis pour une meilleure analyse

#### 3. Gestion d'erreurs améliorée
- Détecter les pages CAPTCHA
- Retourner des messages d'erreur explicites
- Implémenter retry avec backoff

### Priorité Moyenne

#### 4. Support multi-marketplace
- Amazon.fr (actuel)
- Amazon.com
- Amazon.de
- Amazon.co.uk
- Amazon.es
- Amazon.it

#### 5. Extraction données produit
- Nom du produit
- Prix
- Note globale
- Nombre total d'avis
- Images

#### 6. Cache des résultats
- Éviter de re-scraper les mêmes produits
- TTL configurable (24h par défaut)
- Stockage dans n8n ou externe (Redis)

### Priorité Basse

#### 7. Interface utilisateur
- Formulaire web pour soumettre des URLs
- Dashboard des analyses effectuées
- Export CSV/Excel

#### 8. Analyse avancée
- Détection de faux avis
- Analyse de sentiment par aspect
- Comparaison avec produits concurrents
- Évolution des avis dans le temps

#### 9. Notifications
- Alerte Slack/Email quand note < seuil
- Rapport hebdomadaire automatique

---

## Bugs connus

| ID | Description | Workaround |
|----|-------------|------------|
| #1 | HTML > 150KB cause timeout | Extraire section avis manuellement |
| #2 | Mode URL bloqué par CAPTCHA | Utiliser mode HTML direct |
| #3 | Certains avis sans titre non parsés | Améliorer regex titre |

---

## Historique des versions

### v26 (2026-02-04) - Actuelle
- Parser JavaScript v3 avec regex optimisées
- Support mode dual (HTML direct / URL)
- Synthèse OpenAI avec GPT-4o-mini

### v1-25 (2026-02-01 à 2026-02-04)
- Itérations de développement
- Corrections bugs parsing
- Ajustements connexions nodes

---

## Notes techniques

### Structure HTML Amazon (à jour 2026-02)

```html
<!-- Container principal des avis -->
<div id="cm_cr-review_list">

  <!-- Un avis -->
  <li id="R2NFKUYRMTQBIW" data-hook="review">

    <!-- Auteur -->
    <span class="a-profile-name">Nom Auteur</span>

    <!-- Note -->
    <span class="a-icon-alt">4,5 sur 5 étoiles</span>

    <!-- Titre -->
    <a class="review-title-content a-text-bold">
      <span>Titre de l'avis</span>
    </a>

    <!-- Date -->
    <span data-hook="review-date">
      Commenté en France le 11 novembre 2025
    </span>

    <!-- Texte -->
    <span data-hook="review-body">
      <span class="review-text-content">
        <span>Texte complet de l'avis...</span>
      </span>
    </span>

  </li>

</div>
```

### API Browserless.io - Exemple de configuration

```javascript
// POST https://chrome.browserless.io/content
{
  "url": "https://www.amazon.fr/product-reviews/B089VNJSXR",
  "waitFor": "#cm_cr-review_list",
  "gotoOptions": {
    "waitUntil": "networkidle2"
  }
}
```

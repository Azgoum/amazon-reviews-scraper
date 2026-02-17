# Documentation API - Amazon Reviews Scraper

## Endpoint

```
POST https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews
```

## Headers requis

| Header | Valeur |
|--------|--------|
| Content-Type | application/json |

## Modes d'utilisation

### Mode 1: HTML Direct (Recommandé)

Ce mode permet d'envoyer directement le HTML de la page Amazon. C'est le mode le plus fiable car il contourne les protections anti-bot.

**Request:**
```bash
curl -X POST "https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews" \
  -H "Content-Type: application/json" \
  -d '{"html": "<HTML de la page Amazon>"}'
```

**Body:**
```json
{
  "html": "<DOCTYPE html>...",
  "url": "https://www.amazon.fr/...",  // Optionnel
  "asin": "B089VNJSXR"                 // Optionnel
}
```

| Champ | Type | Requis | Description |
|-------|------|--------|-------------|
| `html` | string | Oui | Code source HTML de la page des avis Amazon |
| `url` | string | Non | URL originale du produit (pour référence) |
| `asin` | string | Non | ASIN du produit Amazon |

**Comment obtenir le HTML:**
1. Ouvrir la page des avis : `https://www.amazon.fr/product-reviews/{ASIN}`
2. `Ctrl+U` pour afficher le code source
3. `Ctrl+A` puis `Ctrl+C` pour copier
4. Envoyer dans le champ `html`

**Limitation:** Le HTML doit faire moins de ~150KB. Pour les pages volumineuses, extraire uniquement la section contenant les avis (balise `<div id="cm_cr-review_list">`).

---

### Mode 2: URL (Actuellement bloqué)

Ce mode permet d'envoyer une URL Amazon et le workflow récupère automatiquement le HTML.

**Request:**
```bash
curl -X POST "https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews" \
  -H "Content-Type: application/json" \
  -d '{"link": "https://www.amazon.fr/product-reviews/B089VNJSXR"}'
```

**Body:**
```json
{
  "link": "https://www.amazon.fr/product-reviews/B089VNJSXR"
}
```

**Formats d'URL acceptés:**
- `https://www.amazon.fr/product-reviews/B089VNJSXR`
- `https://www.amazon.fr/dp/B089VNJSXR`
- `https://www.amazon.fr/gp/product/B089VNJSXR`

**Note:** Ce mode est actuellement bloqué par les protections CAPTCHA d'Amazon. Une intégration Browserless.io est prévue pour contourner cette limitation.

---

### Mode 3: URLs multiples

**Body:**
```json
{
  "links": [
    "https://www.amazon.fr/product-reviews/B089VNJSXR",
    "https://www.amazon.fr/product-reviews/B08XYZ1234"
  ]
}
```

---

## Réponse

### Succès (200 OK)

```json
{
  "output": [
    {
      "content": {
        "note_moyenne": 4.7,
        "nombre_avis": 10,
        "points_positifs": [
          "Robustesse et bonne qualité des matériaux",
          "Hermétique et facile à nettoyer",
          "Variété de tailles pour différents besoins",
          "Emballage soigné et protection des produits",
          "Pratique pour un usage quotidien",
          "Convient au micro-ondes et lave-vaisselle"
        ],
        "points_negatifs": [
          "Aucun point négatif mentionné"
        ],
        "resume": "Les utilisateurs apprécient fortement ces boîtes de repas pour leur robustesse, leur qualité et leur praticité.",
        "recommandation": "Acheter"
      },
      "finish_reason": "stop",
      "model": "gpt-4o-mini-2024-07-18",
      "usage": {
        "prompt_tokens": 1234,
        "completion_tokens": 256,
        "total_tokens": 1490
      }
    }
  ]
}
```

### Champs de réponse

| Champ | Type | Description |
|-------|------|-------------|
| `note_moyenne` | number | Note moyenne calculée (0-5) |
| `nombre_avis` | number | Nombre d'avis analysés |
| `points_positifs` | string[] | Liste des points forts mentionnés |
| `points_negatifs` | string[] | Liste des points faibles mentionnés |
| `resume` | string | Synthèse globale en français |
| `recommandation` | string | "Acheter", "À éviter", ou "Mitigé" |

---

## Exemples d'utilisation

### PowerShell

```powershell
# Lire le HTML depuis un fichier
$html = Get-Content -Raw "page_amazon.html"
$body = @{ html = $html } | ConvertTo-Json -Depth 1 -Compress

# Envoyer la requête
$response = Invoke-RestMethod `
  -Uri "https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews" `
  -Method Post `
  -ContentType "application/json" `
  -Body $body

# Afficher le résultat
$response | ConvertTo-Json -Depth 10
```

### Python

```python
import requests
import json

# Lire le HTML
with open("page_amazon.html", "r", encoding="utf-8") as f:
    html = f.read()

# Envoyer la requête
response = requests.post(
    "https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews",
    headers={"Content-Type": "application/json"},
    json={"html": html}
)

# Afficher le résultat
result = response.json()
print(json.dumps(result, indent=2, ensure_ascii=False))
```

### JavaScript (Node.js)

```javascript
const fs = require('fs');

const html = fs.readFileSync('page_amazon.html', 'utf-8');

fetch('https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ html })
})
  .then(res => res.json())
  .then(data => console.log(JSON.stringify(data, null, 2)));
```

---

## Codes d'erreur

| Code | Description |
|------|-------------|
| 200 | Succès |
| 422 | Body JSON invalide |
| 500 | Erreur interne (parsing ou OpenAI) |

---

## Rate Limits

- **n8n Cloud:** Dépend du plan souscrit
- **OpenAI API:** Limites standard de l'API OpenAI

---

## Webhooks de test

Pour tester le workflow sans données réelles :

```bash
# Test minimal
curl -X POST "https://<VOTRE_INSTANCE>.app.n8n.cloud/webhook/amazon-reviews" \
  -H "Content-Type: application/json" \
  -d '{
    "html": "<li data-hook=\"review\"><span class=\"a-profile-name\">Test User</span><span class=\"a-icon-alt\">4,5 sur 5 etoiles</span><a class=\"review-title-content a-text-bold\"><span>Super produit</span></a><span data-hook=\"review-date\">Le 1 janvier 2026</span><span class=\"review-text-content\"><span>Tres bon achat!</span></span></li>"
  }'
```

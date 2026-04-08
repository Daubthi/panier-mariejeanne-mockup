# Brief Technique - Module Panier Marie Jeanne CBD

**Maquette de reference :** [https://daubthi.github.io/panier-mariejeanne-mockup/](https://daubthi.github.io/panier-mariejeanne-mockup/)

**Derniere mise a jour :** 2026-04-08

---

## Table des matieres

1. [Vue d'ensemble & Objectifs](#1-vue-densemble--objectifs)
2. [Architecture & Dependances](#2-architecture--dependances)
3. [Module Cadeau - Specifications](#3-module-cadeau---specifications)
4. [Barre de Progression - Specifications](#4-barre-de-progression---specifications)
5. [Cagnotte Fidelite - Specifications](#5-cagnotte-fidelite---specifications)
6. [Produits Recommandes](#6-produits-recommandes)
7. [Responsive & Mobile](#7-responsive--mobile)
8. [Back-office (configuration admin)](#8-back-office-configuration-admin)
9. [Cas limites & Edge Cases](#9-cas-limites--edge-cases)
10. [Checklist de recette (QA)](#10-checklist-de-recette-qa)

---

## 1. Vue d'ensemble & Objectifs

### Contexte

Refonte de la page panier (cart page) du site Shopify Marie Jeanne CBD. L'objectif est d'augmenter le panier moyen et la fidelisation client via trois leviers :

- **Module cadeau** : inciter le client a atteindre des paliers de depense pour obtenir un produit gratuit.
- **Barre de progression** : feedback visuel en temps reel sur la progression vers les paliers (livraison gratuite, cadeau tier 1, cadeau tier 2).
- **Cagnotte fidelite** : systeme de cashback en euros encourageant le retour client.

### Objectifs techniques

- Implementation 100% front-end compatible Shopify Liquid + JavaScript vanilla (ou Alpine.js si deja present dans le theme).
- Aucune dependance externe lourde (pas de React/Vue sauf si deja utilise).
- Temps de chargement additionnel < 50ms (lazy load des images cadeaux).
- Compatibilite mobile-first, breakpoints a 960px et 420px.

---

## 2. Architecture & Dependances

### Stack technique

| Composant | Technologie |
|---|---|
| Template engine | Shopify Liquid |
| Logique front-end | JavaScript vanilla (ES6+) / Alpine.js |
| Styles | CSS custom properties + BEM naming |
| API panier | Shopify Cart API (`/cart.js`, `/cart/add.js`, `/cart/change.js`) |
| Cagnotte | API custom ou metafields client Shopify |
| Back-office | Shopify metaobjects ou theme settings (`settings_schema.json`) |

### Structure de fichiers suggeree

```
snippets/
  cart-progress-bar.liquid
  cart-gift-module.liquid
  cart-loyalty.liquid
  cart-recommended-products.liquid
  cart-fixed-cta.liquid

assets/
  cart-module.js          # Logique principale (thresholds, gift selection, progress)
  cart-module.css         # Styles du module complet
```

### Flux de donnees

```
[Cart Update Event]
       |
       v
  fetchCart() --- GET /cart.js
       |
       v
  calculateSubtotal()  --> exclut les line items gift (property: _gift = true)
       |
       +---> updateProgressBar(subtotal)
       +---> evaluateGiftEligibility(subtotal)
       +---> updateLoyalty(subtotal)
       +---> updateCTA(subtotal)
```

---

## 3. Module Cadeau - Specifications

### 3.1 Constantes et paliers

```javascript
const THRESHOLDS = {
  SHIPPING_FREE: 39.90,  // Livraison gratuite
  GIFT_TIER_1:   50.00,  // 1 cadeau parmi la selection tier 1
  GIFT_TIER_2:  100.00   // 1 cadeau parmi la selection tier 1 + tier 2
};
```

### 3.2 Regles metier

| Regle | Detail |
|---|---|
| Base de calcul | Le sous-total est calcule **AVANT** frais de livraison, **AVANT** reduction/cagnotte |
| Limite | **1 seul cadeau par commande**. Selectionner un nouveau cadeau remplace le precedent |
| Retrait automatique | Si le sous-total passe sous le palier (suppression article, diminution quantite), le cadeau est automatiquement retire du panier |
| Exclusion du calcul | Le cadeau ne compte **pas** dans le sous-total pour le calcul des paliers |
| Tier 2 debloque tout | Atteindre 100 EUR donne acces a **tous** les cadeaux (tier 1 + tier 2) |
| Tier 2 verrouille | Quand tier 2 n'est pas atteint, les cartes tier 2 restent visibles mais grisees |

### 3.3 Structure de donnees des produits cadeaux

```javascript
// Injecte via Liquid depuis les metaobjects ou theme settings
const giftProducts = {
  tier_1: [
    {
      id: 12345678,              // Shopify variant ID
      name: "Baume CBD 10ml",
      image: "https://cdn.shopify.com/.../baume-cbd.jpg",
      value_price: "12,90 €"     // Prix barre affiche (valeur perçue)
    },
    // ...
  ],
  tier_2: [
    {
      id: 87654321,
      name: "Huile CBD 30ml",
      image: "https://cdn.shopify.com/.../huile-cbd.jpg",
      value_price: "29,90 €"
    },
    // ...
  ]
};
```

### 3.4 Affichage des cartes cadeaux

Chaque carte cadeau affiche :
- Image produit
- Nom du produit
- Prix barre (`value_price`) pour montrer la valeur offerte
- Badge "OFFERT" ou "GRATUIT"
- Bouton de selection / etat selectionne

**Etats visuels des cartes :**

```css
/* Carte tier 2 verrouillee (sous-total < 100€) */
.gift-card--locked {
  opacity: 0.4;
  pointer-events: none;
  filter: grayscale(30%);
}

/* Carte selectionnee */
.gift-card--selected {
  border: 2px solid var(--color-primary);
  box-shadow: 0 0 0 3px rgba(var(--color-primary-rgb), 0.15);
}
```

### 3.5 Logique de selection JavaScript

```javascript
async function selectGift(variantId, tier) {
  const cart = await fetchCart();
  const subtotal = calculateSubtotal(cart);

  // Verifier l'eligibilite
  if (tier === 1 && subtotal < THRESHOLDS.GIFT_TIER_1) return;
  if (tier === 2 && subtotal < THRESHOLDS.GIFT_TIER_2) return;

  // Retirer le cadeau existant s'il y en a un
  const existingGift = cart.items.find(item => item.properties?._gift === 'true');
  if (existingGift) {
    await fetch('/cart/change.js', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id: existingGift.key, quantity: 0 })
    });
  }

  // Ajouter le nouveau cadeau
  await fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: [{
        id: variantId,
        quantity: 1,
        properties: {
          _gift: 'true',
          _gift_tier: String(tier)
        }
      }]
    })
  });

  refreshCartUI();
}
```

### 3.6 Re-evaluation apres modification du panier

```javascript
async function onCartUpdated() {
  const cart = await fetchCart();
  const subtotal = calculateSubtotal(cart);

  const existingGift = cart.items.find(item => item.properties?._gift === 'true');

  if (existingGift) {
    const giftTier = parseInt(existingGift.properties._gift_tier);
    const requiredThreshold = giftTier === 2
      ? THRESHOLDS.GIFT_TIER_2
      : THRESHOLDS.GIFT_TIER_1;

    if (subtotal < requiredThreshold) {
      // Retirer automatiquement le cadeau
      await fetch('/cart/change.js', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ id: existingGift.key, quantity: 0 })
      });
    }
  }

  updateProgressBar(subtotal);
  updateGiftModule(subtotal);
  updateLoyalty(subtotal);
}
```

### 3.7 Calcul du sous-total (excluant les cadeaux)

```javascript
function calculateSubtotal(cart) {
  return cart.items
    .filter(item => item.properties?._gift !== 'true')
    .reduce((sum, item) => sum + item.final_line_price, 0) / 100;
  // Shopify retourne les prix en centimes
}
```

### 3.8 API / Backend Requirements

| Besoin | Implementation |
|---|---|
| Stockage du cadeau selectionne | Line item avec `properties: { _gift: 'true', _gift_tier: '1' }` |
| Prix du cadeau | Le variant cadeau doit avoir `price = 0` **OU** un Shopify Script / Discount automatique ramene le prix a 0 pour les items avec property `_gift` |
| Securite | Cote serveur (Shopify Script ou webhook `orders/create`), valider que le cadeau est legitime par rapport au sous-total |
| Configuration admin | Via metaobjects ou `settings_schema.json` : paliers, produits par tier, activation/desactivation du module |

---

## 4. Barre de Progression - Specifications

### 4.1 Paliers et mapping visuel

La barre affiche 3 jalons avec un mapping **non-lineaire** pour garantir un espacement visuel equilibre :

| Palier | Montant | Position visuelle |
|---|---|---|
| Livraison gratuite | 39,90 EUR | 33% |
| Cadeau tier 1 | 50,00 EUR | 60% |
| Cadeau tier 2 | 100,00 EUR | 100% |

### 4.2 Formule de calcul de la progression

```javascript
function calculateProgress(subtotal) {
  if (subtotal <= 0) return 0;

  if (subtotal <= THRESHOLDS.SHIPPING_FREE) {
    // 0% -> 33%
    return (subtotal / THRESHOLDS.SHIPPING_FREE) * 33;
  }

  if (subtotal <= THRESHOLDS.GIFT_TIER_1) {
    // 33% -> 60%
    return 33 + ((subtotal - THRESHOLDS.SHIPPING_FREE)
      / (THRESHOLDS.GIFT_TIER_1 - THRESHOLDS.SHIPPING_FREE)) * 27;
  }

  if (subtotal <= THRESHOLDS.GIFT_TIER_2) {
    // 60% -> 100%
    return 60 + ((subtotal - THRESHOLDS.GIFT_TIER_1)
      / (THRESHOLDS.GIFT_TIER_2 - THRESHOLDS.GIFT_TIER_1)) * 40;
  }

  return 100;
}
```

### 4.3 Implementation CSS

```css
.progress-bar {
  width: 100%;
  height: 8px;
  background: var(--color-grey-light, #e5e5e5);
  border-radius: 4px;
  overflow: hidden;
  position: relative;
}

.progress-bar__fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.5s ease-out;
  /* Gradient qui evolue avec la progression */
  background: linear-gradient(90deg,
    var(--color-progress-start, #a8d5a2),
    var(--color-progress-end, #4caf50)
  );
}

/* Marqueurs de jalons */
.progress-bar__milestone {
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  width: 16px;
  height: 16px;
  border-radius: 50%;
  border: 2px solid var(--color-grey-light);
  background: white;
  transition: background 0.3s, border-color 0.3s;
}

.progress-bar__milestone--reached {
  background: var(--color-primary);
  border-color: var(--color-primary);
}

.progress-bar__milestone:nth-child(1) { left: 33%; }
.progress-bar__milestone:nth-child(2) { left: 60%; }
.progress-bar__milestone:nth-child(3) { left: 100%; }
```

### 4.4 Messages dynamiques

Les messages changent en fonction du sous-total :

```javascript
function getProgressMessage(subtotal) {
  const remaining_shipping = THRESHOLDS.SHIPPING_FREE - subtotal;
  const remaining_tier1 = THRESHOLDS.GIFT_TIER_1 - subtotal;
  const remaining_tier2 = THRESHOLDS.GIFT_TIER_2 - subtotal;

  if (subtotal < THRESHOLDS.SHIPPING_FREE) {
    return `Plus que <strong>${remaining_shipping.toFixed(2)} €</strong> pour la livraison gratuite !`;
  }

  if (subtotal < THRESHOLDS.GIFT_TIER_1) {
    return `Livraison gratuite ! Plus que <strong>${remaining_tier1.toFixed(2)} €</strong> pour recevoir un cadeau !`;
  }

  if (subtotal < THRESHOLDS.GIFT_TIER_2) {
    return `Choisissez votre cadeau ! Plus que <strong>${remaining_tier2.toFixed(2)} €</strong> pour debloquer les cadeaux premium !`;
  }

  return `Felicitations ! Tous les cadeaux sont debloques !`;
}
```

### 4.5 Gradient dynamique

Le gradient de la barre evolue visuellement a mesure que les paliers sont atteints. Mettre a jour la variable CSS depuis JavaScript :

```javascript
function updateProgressBarGradient(subtotal) {
  const bar = document.querySelector('.progress-bar__fill');

  if (subtotal >= THRESHOLDS.GIFT_TIER_2) {
    bar.style.background = 'linear-gradient(90deg, #a8d5a2, #4caf50, #2e7d32)';
  } else if (subtotal >= THRESHOLDS.GIFT_TIER_1) {
    bar.style.background = 'linear-gradient(90deg, #a8d5a2, #4caf50)';
  } else {
    bar.style.background = 'linear-gradient(90deg, #c8e6c9, #a8d5a2)';
  }

  bar.style.width = calculateProgress(subtotal) + '%';
}
```

---

## 5. Cagnotte Fidelite - Specifications

### 5.1 Systeme de points

Le systeme utilise des **euros** (pas de points) avec un ratio simple :

| Parametre | Valeur |
|---|---|
| Ratio | 10 EUR depenses = 1 EUR de cagnotte (soit **1%**) |
| Unite | Euros (EUR) |
| Calcul | `cagnotte_earned = subtotal * 0.01` |

### 5.2 Affichage

```html
<!-- Gain sur cette commande -->
<div class="loyalty__earn">
  <span class="loyalty__icon">🎁</span>
  <p>Cette commande vous rapporte <strong>{{ cagnotte_earned }} €</strong> de cagnotte</p>
</div>

<!-- Solde existant (si > 0) -->
<div class="loyalty__balance" data-balance="{{ customer_cagnotte }}">
  <p>Votre cagnotte : <strong>{{ customer_cagnotte }} €</strong></p>
  <button class="loyalty__use-btn" onclick="applyCagnotte()">Utiliser</button>
</div>

<!-- Apres application -->
<div class="loyalty__applied" style="display: none;">
  <p>Cagnotte appliquee : <strong>-{{ applied_amount }} €</strong></p>
  <button class="loyalty__cancel-btn" onclick="cancelCagnotte()">Annuler</button>
</div>
```

### 5.3 Logique d'application

```javascript
function applyCagnotte() {
  const balance = parseFloat(
    document.querySelector('.loyalty__balance').dataset.balance
  );
  const cartTotal = getCurrentCartTotal(); // total TTC apres livraison

  // Appliquer le minimum entre la cagnotte et le total
  const appliedAmount = Math.min(balance, cartTotal);

  // Envoyer au backend (cart attribute ou discount code)
  fetch('/cart/update.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      attributes: {
        cagnotte_applied: appliedAmount.toFixed(2)
      }
    })
  }).then(() => {
    // Mettre a jour l'UI
    document.querySelector('.loyalty__balance').style.display = 'none';
    document.querySelector('.loyalty__applied').style.display = 'flex';
    document.querySelector('.loyalty__applied strong').textContent =
      `-${appliedAmount.toFixed(2)} €`;
    refreshCartUI();
  });
}

function cancelCagnotte() {
  fetch('/cart/update.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      attributes: { cagnotte_applied: '0' }
    })
  }).then(() => {
    document.querySelector('.loyalty__balance').style.display = 'flex';
    document.querySelector('.loyalty__applied').style.display = 'none';
    refreshCartUI();
  });
}
```

### 5.4 Rappel important

Le sous-total utilise pour le **calcul des paliers cadeaux** est celui **avant** application de la cagnotte. La cagnotte n'affecte pas l'eligibilite aux cadeaux.

---

## 6. Produits Recommandes

### 6.1 Format des donnees

```javascript
const recommendedProducts = [
  {
    id: 11111,
    name: "Fleur CBD OG Kush",
    price: "8,90 €",
    price_cents: 890,
    rating: 4.5,           // note /5
    review_count: 128,
    image: "https://cdn.shopify.com/.../og-kush.jpg",
    variant_id: 22222
  },
  // ...
];
```

### 6.2 Carousel horizontal

```css
.recommended__list {
  display: flex;
  gap: 16px;
  overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch;
  padding-bottom: 8px; /* espace pour scrollbar */
}

.recommended__card {
  scroll-snap-align: start;
  flex: 0 0 180px; /* largeur fixe des cartes */
  border-radius: 8px;
  overflow: hidden;
}

/* Masquer la scrollbar sur desktop */
.recommended__list::-webkit-scrollbar {
  height: 4px;
}

.recommended__list::-webkit-scrollbar-thumb {
  background: var(--color-grey-medium);
  border-radius: 2px;
}
```

### 6.3 Bouton "Ajouter"

Le bouton "Ajouter" sur chaque carte ajoute le produit au panier et rafraichit toute l'UI :

```javascript
async function addRecommendedProduct(variantId) {
  const btn = event.currentTarget;
  btn.disabled = true;
  btn.textContent = 'Ajout...';

  await fetch('/cart/add.js', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      items: [{ id: variantId, quantity: 1 }]
    })
  });

  btn.textContent = 'Ajoute !';
  setTimeout(() => {
    btn.disabled = false;
    btn.textContent = 'Ajouter';
  }, 1500);

  // Refresh complet : panier, barre, cadeaux, cagnotte
  await onCartUpdated();
}
```

---

## 7. Responsive & Mobile

### 7.1 Breakpoints

```css
/* Desktop : > 960px (layout par defaut) */
/* Tablet / Mobile : <= 960px */
@media (max-width: 960px) { /* ... */ }

/* Petit mobile : <= 420px */
@media (max-width: 420px) { /* ... */ }
```

### 7.2 Articles du panier - Layout mobile

```css
@media (max-width: 960px) {
  .cart-item {
    display: grid;
    grid-template-columns: 56px 1fr;
    grid-template-rows: auto auto;
    gap: 8px 12px;
  }

  .cart-item__image {
    grid-row: 1 / 3;
    width: 56px;
    height: 56px;
    object-fit: cover;
    border-radius: 4px;
  }

  .cart-item__info {
    grid-column: 2;
    grid-row: 1;
  }

  .cart-item__actions {
    grid-column: 1 / -1; /* pleine largeur, seconde ligne */
    grid-row: 2;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }
}
```

### 7.3 Module cadeau - Accordion mobile

```css
@media (max-width: 960px) {
  .gift-module__content {
    max-height: 0;
    overflow: hidden;
    transition: max-height 0.35s ease-out;
  }

  .gift-module__content--open {
    max-height: 600px; /* valeur suffisamment grande */
  }
}
```

```javascript
// Toggle accordion
document.querySelector('.gift-module__header').addEventListener('click', () => {
  const content = document.querySelector('.gift-module__content');
  content.classList.toggle('gift-module__content--open');
});
```

### 7.4 Barre CTA fixe (mobile)

La barre fixe apparait quand l'utilisateur scrolle au-dela de la section produits recommandes :

```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    const ctaBar = document.querySelector('.fixed-cta-bar');
    if (entry.isIntersecting) {
      ctaBar.classList.remove('fixed-cta-bar--visible');
    } else {
      ctaBar.classList.add('fixed-cta-bar--visible');
    }
  });
}, { threshold: 0 });

observer.observe(document.querySelector('.recommended-products'));
```

```css
.fixed-cta-bar {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: white;
  box-shadow: 0 -2px 10px rgba(0, 0, 0, 0.1);
  padding: 12px 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transform: translateY(100%);
  transition: transform 0.3s ease-out;
  z-index: 100;
}

.fixed-cta-bar--visible {
  transform: translateY(0);
}
```

**Contenu de la barre CTA :**
- Total TTC
- Indication cagnotte (ex: "+ 0,50 EUR de cagnotte")
- Bouton "Commander"

### 7.5 Comportement mobile du bouton "Commander"

Au **premier tap** sur "Commander" : si la cagnotte du client est > 0 EUR et n'a pas encore ete utilisee, le scroll amene l'utilisateur a la section cagnotte fidelite plutot que de valider la commande.

```javascript
let cagnottePrompted = false;

function handleCheckout() {
  const balance = getCustomerCagnotteBalance();
  const applied = getCagnotteApplied();

  if (!cagnottePrompted && balance > 0 && applied === 0 && isMobile()) {
    cagnottePrompted = true;
    document.querySelector('.loyalty').scrollIntoView({
      behavior: 'smooth',
      block: 'center'
    });
    return; // ne pas valider la commande
  }

  // Proceder au checkout
  window.location.href = '/checkout';
}

function isMobile() {
  return window.innerWidth <= 960;
}
```

---

## 8. Back-office (configuration admin)

### 8.1 Parametres configurables

L'administrateur doit pouvoir modifier les elements suivants **sans toucher au code** :

| Parametre | Type | Defaut |
|---|---|---|
| `gift_module_enabled` | Boolean | `true` |
| `shipping_threshold` | Number (EUR) | `39.90` |
| `gift_tier_1_threshold` | Number (EUR) | `50.00` |
| `gift_tier_2_threshold` | Number (EUR) | `100.00` |
| `gift_tier_1_products` | Liste de variants | (a configurer) |
| `gift_tier_2_products` | Liste de variants | (a configurer) |
| `loyalty_rate` | Number (%) | `1` |
| `loyalty_enabled` | Boolean | `true` |

### 8.2 Implementation Shopify recommandee

**Option A - `settings_schema.json` (theme settings) :**

```json
{
  "name": "Module Cadeau",
  "settings": [
    {
      "type": "checkbox",
      "id": "gift_module_enabled",
      "label": "Activer le module cadeau",
      "default": true
    },
    {
      "type": "text",
      "id": "shipping_threshold",
      "label": "Seuil livraison gratuite (€)",
      "default": "39.90"
    },
    {
      "type": "text",
      "id": "gift_tier_1_threshold",
      "label": "Seuil cadeau tier 1 (€)",
      "default": "50"
    },
    {
      "type": "text",
      "id": "gift_tier_2_threshold",
      "label": "Seuil cadeau tier 2 (€)",
      "default": "100"
    },
    {
      "type": "product_list",
      "id": "gift_tier_1_products",
      "label": "Produits cadeau - Tier 1",
      "limit": 6
    },
    {
      "type": "product_list",
      "id": "gift_tier_2_products",
      "label": "Produits cadeau - Tier 2",
      "limit": 6
    }
  ]
}
```

**Option B - Metaobjects Shopify (plus flexible, recommande pour les listes de produits importantes) :**

Creer un metaobject `gift_tier` avec les champs :
- `tier_level` (integer)
- `threshold` (decimal)
- `products` (list.product_reference)
- `enabled` (boolean)

---

## 9. Cas limites & Edge Cases

### 9.1 Panier vide

- La barre de progression est a 0%, message generique d'accueil.
- Le module cadeau est masque.
- La section cagnotte affiche uniquement le solde (pas de bouton "Utiliser").

### 9.2 Sous-total exactement egal a un palier

- `subtotal === 39.90` : livraison gratuite **activee**.
- `subtotal === 50.00` : tier 1 **debloque**.
- `subtotal === 100.00` : tier 2 **debloque**.
- Utiliser `>=` dans toutes les comparaisons.

### 9.3 Suppression d'article faisant passer sous un palier

1. L'utilisateur a un cadeau tier 1 et retire un article (sous-total passe de 55 EUR a 42 EUR).
2. Le cadeau tier 1 est **automatiquement retire** du panier.
3. Un toast/notification informe l'utilisateur : "Votre cadeau a ete retire car le montant minimum n'est plus atteint."
4. La barre de progression se met a jour.

### 9.4 Changement de tier vers le bas

1. L'utilisateur a un cadeau tier 2, retire un article (sous-total passe de 110 EUR a 65 EUR).
2. Le cadeau tier 2 est retire.
3. L'utilisateur reste eligible au tier 1 mais doit **re-selectionner** manuellement un cadeau tier 1 (pas de substitution automatique).

### 9.5 Produit cadeau en rupture de stock

- Si un variant cadeau est en rupture, la carte est grisee avec la mention "Indisponible".
- Le cadeau ne peut pas etre selectionne.
- Verifier la disponibilite via `variant.available` en Liquid ou API.

### 9.6 Cagnotte superieure au total

- Si la cagnotte (ex: 15 EUR) depasse le total (ex: 12 EUR), n'appliquer que le total : `min(cagnotte, total)`.
- Le reliquat reste sur la cagnotte du client.

### 9.7 Sessions concurrentes / onglets multiples

- Chaque action panier doit faire un `GET /cart.js` frais avant de modifier quoi que ce soit.
- Eviter les race conditions en chainant les appels (pas de modifications paralleles).

### 9.8 Discount codes + cagnotte

- Si un code promo est applique, le sous-total pour les paliers cadeaux reste calcule **avant** la reduction (prix de base des articles).
- La cagnotte s'applique sur le total **apres** le discount code.

---

## 10. Checklist de recette (QA)

### Barre de progression

- [ ] La barre est a 0% quand le panier est vide
- [ ] La barre atteint 33% quand le sous-total = 39,90 EUR
- [ ] La barre atteint 60% quand le sous-total = 50,00 EUR
- [ ] La barre atteint 100% quand le sous-total = 100,00 EUR
- [ ] Les jalons (icones/cercles) changent d'etat visuellement quand atteints
- [ ] Le message dynamique change a chaque palier
- [ ] Le gradient de couleur evolue correctement
- [ ] L'animation (transition CSS) est fluide a chaque mise a jour

### Module cadeau

- [ ] Les cartes tier 1 s'affichent quand sous-total >= 50 EUR
- [ ] Les cartes tier 2 sont visibles mais grisees quand sous-total < 100 EUR
- [ ] Les cartes tier 2 se debloquent quand sous-total >= 100 EUR
- [ ] Cliquer sur un cadeau l'ajoute au panier avec `price = 0` et property `_gift`
- [ ] Selectionner un 2eme cadeau remplace le premier (jamais 2 cadeaux simultanes)
- [ ] Retirer un article qui fait passer sous le palier retire automatiquement le cadeau
- [ ] Un toast de notification s'affiche lors du retrait automatique
- [ ] Le cadeau n'est pas comptabilise dans le sous-total des paliers
- [ ] Les produits en rupture de stock sont marques "Indisponible"

### Cagnotte fidelite

- [ ] Le message "Cette commande vous rapporte X EUR de cagnotte" s'affiche avec le bon montant (subtotal * 0.01)
- [ ] Le solde de cagnotte existant s'affiche si > 0
- [ ] Le bouton "Utiliser" applique la cagnotte (min entre cagnotte et total)
- [ ] Le bouton "Annuler" retire la cagnotte et restaure le total initial
- [ ] La cagnotte appliquee n'affecte pas le calcul des paliers cadeaux

### Produits recommandes

- [ ] Le carousel scroll horizontalement avec snap
- [ ] Le bouton "Ajouter" ajoute le produit au panier
- [ ] L'ajout d'un produit recommande rafraichit : panier, barre, module cadeau, cagnotte
- [ ] Le bouton passe en etat "Ajoute !" temporairement

### Responsive / Mobile

- [ ] A 960px, le layout passe en mode mobile
- [ ] Les articles du panier utilisent le grid 56px + 1fr avec actions en 2eme ligne
- [ ] Le module cadeau est en accordion (repliable)
- [ ] La barre CTA fixe apparait au scroll (apres la section produits recommandes)
- [ ] La barre CTA affiche : total TTC, cagnotte hint, bouton Commander
- [ ] Premier tap sur "Commander" scrolle vers la cagnotte si solde > 0 et non utilisee
- [ ] A 420px, les elements sont correctement lisibles et utilisables

### Back-office

- [ ] L'admin peut activer/desactiver le module cadeau
- [ ] L'admin peut modifier les seuils (39.90, 50, 100)
- [ ] L'admin peut configurer les produits par tier
- [ ] L'admin peut activer/desactiver la cagnotte
- [ ] Les modifications sont visibles immediatement sur le front

### Performance

- [ ] Pas de layout shift au chargement (CLS < 0.1)
- [ ] Les images cadeaux sont lazy-loaded
- [ ] Le JavaScript total du module < 15 KB gzippe
- [ ] Le temps de reponse des appels `/cart/*.js` est < 300ms

---

*Document genere le 2026-04-08. Maquette de reference : [daubthi.github.io/panier-mariejeanne-mockup](https://daubthi.github.io/panier-mariejeanne-mockup/)*

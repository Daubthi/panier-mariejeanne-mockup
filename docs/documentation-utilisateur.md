# Documentation Utilisateur -- Page Panier Marie Jeanne CBD

> **Mockup interactif** : [https://daubthi.github.io/panier-mariejeanne-mockup/](https://daubthi.github.io/panier-mariejeanne-mockup/)

---

## Table des matieres

1. [Vue d'ensemble](#1-vue-densemble)
2. [Module Cadeau -- Fonctionnement](#2-module-cadeau--fonctionnement)
3. [Barre de progression](#3-barre-de-progression)
4. [Programme de fidelite / Cagnotte](#4-programme-de-fidelite--cagnotte)
5. [Produits recommandes](#5-produits-recommandes)
6. [Experience mobile](#6-experience-mobile)
7. [FAQ / Cas particuliers](#7-faq--cas-particuliers)

---

## 1. Vue d'ensemble

La nouvelle page panier de Marie Jeanne CBD integre plusieurs modules destines a ameliorer l'experience d'achat et a augmenter le panier moyen :

- **Module Cadeau** : offre un produit gratuit au client en fonction du montant de son panier.
- **Barre de progression** : indique au client sa progression vers les differents avantages (livraison gratuite, cadeaux).
- **Cagnotte fidelite** : permet au client de cumuler et d'utiliser un solde de fidelite.
- **Produits recommandes** : carrousel de produits complementaires pour encourager l'ajout au panier.

L'ensemble de ces fonctionnalites est adapte pour une experience optimale sur desktop et mobile.

[voir mockup desktop]

---

## 2. Module Cadeau -- Fonctionnement

### Emplacement

Le module cadeau apparait sur la page panier, en dessous des articles du panier et du carrousel de produits recommandes.

### Principe

Le client peut choisir **un produit offert** en fonction du sous-total de son panier (hors frais de livraison). Deux paliers sont disponibles :

### Palier 1 -- A partir de 50 EUR

Le client peut choisir **1 cadeau** parmi les 4 produits suivants :

| Produit               | Description                   |
|-----------------------|-------------------------------|
| Bonbons CBD           | Confiseries au CBD            |
| Infusion CBD          | Tisane au CBD                 |
| Sample Huile CBD 5%   | Echantillon d'huile CBD a 5%  |
| Grinder Marie Jeanne  | Grinder aux couleurs de la marque |

### Palier 2 -- A partir de 100 EUR

Le client peut choisir **1 cadeau** parmi les 4 produits premium suivants :

| Produit                          | Description                              |
|----------------------------------|------------------------------------------|
| E-liquide Gelato                 | E-liquide CBD saveur Gelato              |
| Huile CBD 10% Full Spectrum      | Huile CBD concentration 10%              |
| Pack Decouverte Fleurs           | Assortiment de fleurs CBD                |
| Vaporisateur Pocket              | Vaporisateur portable                    |

### Regles de fonctionnement

- **Un seul cadeau a la fois** : le client ne peut selectionner qu'un seul cadeau. Si un cadeau du palier 1 est deja selectionne et que le client choisit un cadeau du palier 2, le cadeau du palier 1 est automatiquement remplace.
- **Suppression automatique** : si le montant du panier passe en dessous du seuil du palier concerne (par exemple, retrait d'un article faisant passer le sous-total sous 50 EUR), le cadeau est automatiquement retire du panier.
- **Affichage dans le panier** : le cadeau selectionne apparait comme une ligne d'article dans le panier avec un badge **"CADEAU"** et un prix affiche **"GRATUIT"**.

[voir mockup desktop]

---

## 3. Barre de progression

### Fonctionnement

La barre de progression affiche visuellement l'avancement du client vers les differents avantages. Elle comporte **3 jalons** :

| Jalon   | Seuil    | Avantage                           |
|---------|----------|------------------------------------|
| Jalon 1 | 39,90 EUR | Livraison gratuite                 |
| Jalon 2 | 50 EUR    | Cadeau offert (palier 1)           |
| Jalon 3 | 100 EUR   | Cadeau premium offert (palier 2)   |

### Messages dynamiques

En fonction du montant actuel du panier, des messages encouragent le client a augmenter son panier. Par exemple :

- *"Plus que 12,00 EUR pour obtenir la livraison gratuite !"*
- *"Ajoutez 8,50 EUR pour debloquer votre cadeau !"*

### Affichage mobile

Sur mobile, la barre de progression est **dupliquee en haut de la page** pour garantir sa visibilite sans que le client ait besoin de faire defiler la page.

[voir mockup mobile]

---

## 4. Programme de fidelite / Cagnotte

### Cumul de la cagnotte

Chaque commande permet au client de cumuler **1% du sous-total** en cagnotte fidelite. Le montant gagne est affiche clairement dans l'encart fidelite de la page panier.

> Exemple : pour un panier de 75 EUR, le client cumulera 0,75 EUR de cagnotte.

### Utilisation de la cagnotte

Si le client dispose d'un solde de cagnotte existant :

1. Le montant disponible est affiche dans l'encart fidelite.
2. Un bouton **"Utiliser"** permet d'appliquer la cagnotte en deduction du total.
3. Une fois appliquee, le bouton se transforme en **"Applique"** accompagne d'un lien **"Annuler"** pour revenir en arriere.

### Comportement mobile

- La barre fixe d'action (CTA) en bas de l'ecran affiche une indication de la cagnotte disponible.
- Au **premier appui** sur le bouton "Commander", la page defile automatiquement jusqu'a l'encart fidelite pour encourager le client a utiliser sa cagnotte.
- Au **second appui**, le client est redirige vers la page de paiement (checkout).

[voir mockup desktop] [voir mockup mobile]

---

## 5. Produits recommandes

### Emplacement

Le carrousel de produits recommandes est positionne entre les articles du panier et le module cadeau.

### Contenu

Le carrousel affiche **5 fiches produit** presentees horizontalement et navigables par defilement (scroll). Chaque fiche contient :

- Nom du produit
- Prix
- Note / avis clients
- Bouton **"Ajouter"** pour ajouter directement le produit au panier

### Objectif

Ce module vise a augmenter le panier moyen en suggerant des produits complementaires pertinents.

[voir mockup desktop]

---

## 6. Experience mobile

L'ensemble de la page panier est adapte pour les ecrans mobiles avec les specificites suivantes :

### Mise en page des articles

Les articles du panier utilisent une **grille compacte a 2 colonnes** pour optimiser l'espace a l'ecran.

### Barre de progression dupliquee

La barre de progression est affichee **en haut de la page** en plus de son emplacement standard, pour que le client voie immediatement sa progression des l'ouverture du panier.

### Module cadeau en accordeon

Sur mobile, le module cadeau est presente sous forme d'**accordeon repliable** afin de ne pas surcharger l'affichage. Le client peut le deplier pour consulter et selectionner son cadeau.

### Barre CTA fixe

Une barre d'action fixe apparait en bas de l'ecran une fois que le client a fait defiler la page au-dela du carrousel de produits recommandes. Cette barre affiche :

- Le **montant total** du panier
- Une **indication de la cagnotte** disponible
- Le bouton **"Commander"**

[voir mockup mobile]

---

## 7. FAQ / Cas particuliers

### Le client a un cadeau dans son panier et retire un article : que se passe-t-il ?

Si le sous-total passe en dessous du seuil du palier correspondant au cadeau selectionne, **le cadeau est automatiquement retire** du panier. Un message informe le client de cette suppression.

### Le client peut-il cumuler un cadeau du palier 1 et un cadeau du palier 2 ?

Non. **Un seul cadeau peut etre present dans le panier a la fois.** Si le client atteint le palier 2 et choisit un cadeau premium, celui-ci remplace automatiquement le cadeau du palier 1.

### Le client a un panier a 95 EUR et ajoute un produit recommande a 10 EUR. Que se passe-t-il ?

Le sous-total passe a 105 EUR, ce qui debloque le palier 2. Le client peut desormais choisir un cadeau parmi les produits premium du palier 2 en plus des produits du palier 1. S'il avait deja selectionne un cadeau du palier 1, celui-ci reste dans le panier jusqu'a ce que le client decide de le remplacer.

### La cagnotte est-elle cumulable avec les cadeaux ?

Oui. La cagnotte fidelite et les cadeaux sont deux mecanismes independants. Le client peut tout a fait utiliser sa cagnotte tout en beneficiant d'un cadeau offert.

### Comment est calcule le seuil des paliers cadeau ?

Les seuils sont calcules sur le **sous-total du panier avant frais de livraison**. Les reductions, codes promo et cagnotte appliques peuvent impacter le sous-total et donc l'eligibilite aux cadeaux.

### Le client annule l'utilisation de sa cagnotte. Que se passe-t-il ?

Le montant de la cagnotte est restitue dans le solde du client et le total du panier revient a sa valeur initiale. Le lien "Annuler" permet cette action a tout moment avant la validation de la commande.

---

> **Reference mockup** : [https://daubthi.github.io/panier-mariejeanne-mockup/](https://daubthi.github.io/panier-mariejeanne-mockup/)

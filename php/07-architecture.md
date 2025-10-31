# 🧭 Comprendre le Domain-Driven Design (DDD)

Le **Domain-Driven Design** (DDD) est une approche de conception
logicielle centrée sur le **domaine métier**.\
Elle vise à modéliser le logiciel en se basant sur la **réalité du
métier**, pour obtenir un code plus **expressif**, **robuste** et
**évolutif**.

------------------------------------------------------------------------

## 🧩 Les 3 couches fondamentales

### 1. **Domain Layer (le cœur du métier)**

C'est le **centre de gravité** de l'application.\
On y trouve : - Les **Entités** (avec une identité persistante) - Les
**Value Objects** - Les **Domain Services** - Les **Aggregates** - Les
**Domain Events** - Les **Policies**

#### Exemple de Value Object :

``` php
final class Price
{
    public function __construct(
        private float $amount,
        private string $currency
    ) {
        if ($amount < 0) {
            throw new InvalidArgumentException('Le prix ne peut pas être négatif');
        }
    }

    public function add(Price $other): self
    {
        if ($this->currency !== $other->currency) {
            throw new InvalidArgumentException('Les devises doivent être identiques');
        }

        return new self($this->amount + $other->amount, $this->currency);
    }
}
```

------------------------------------------------------------------------

### 2. **Application Layer (les cas d'usage)**

C'est la couche **d'orchestration** : elle exécute les **use cases** et
manipule les entités et services du domaine.\
Elle ne contient pas de logique métier, uniquement la **coordination**.

Exemples : - `CreateProductHandler` - `UpdateProductPriceHandler` -
`CheckoutOrderHandler`

------------------------------------------------------------------------

### 3. **Infrastructure Layer (le monde extérieur)**

C'est ici que se trouvent les implémentations techniques : -
Repositories Doctrine ou API - Contrôleurs Symfony - Envoi d'emails,
logs, events, etc.

Elle implémente les **interfaces définies dans le domaine** pour
découpler la logique métier de la technique.

------------------------------------------------------------------------

## 🧠 Concepts clés du DDD

### 🗣️ Ubiquitous Language (langage omniprésent)

Le vocabulaire métier est **partagé entre développeurs et experts
métier** et **intégré au code** : noms de classes, méthodes, tests.

> Exemple : utiliser `AddProductToCart` plutôt que `insertItem()`.

------------------------------------------------------------------------

### 🏠 Bounded Contexts

Une application complexe est divisée en **contextes délimités** (bounded
contexts).\
Chaque contexte a son propre modèle métier.

Exemples : - `Catalog` (produits, catégories) - `Sales` (commandes,
paiements) - `Inventory` (stocks) - `Customer` (comptes, fidélité)

> Un "Produit" dans `Catalog` n'est pas forcément le même concept que
> dans `Inventory`.

------------------------------------------------------------------------

### ⚙️ Aggregates (agrégats)

Un **agrégat** est un **regroupement cohérent d'entités et de value
objects** formant une **unité logique et transactionnelle**.

-   Chaque agrégat a une **racine d'agrégat** (Aggregate Root), qui
    contrôle l'accès aux autres entités internes.
-   On **ne modifie jamais directement** les entités internes depuis
    l'extérieur.
-   L'agrégat garantit la **cohérence métier** dans sa limite.

#### Exemple :

``` php
final class Order
{
    /** @var OrderLine[] */
    private array $lines = [];

    public function __construct(
        private string $id,
        private CustomerId $customerId
    ) {}

    public function addProduct(ProductId $productId, int $quantity): void
    {
        if ($quantity <= 0) {
            throw new InvalidArgumentException('Quantité invalide');
        }

        $this->lines[] = new OrderLine($productId, $quantity);
    }

    public function total(): float
    {
        return array_sum(array_map(fn($line) => $line->subtotal(), $this->lines));
    }
}
```

Ici, `Order` est la **racine d'agrégat**, et `OrderLine` une **entité
interne**.\
On ne modifie pas `OrderLine` sans passer par `Order`.

------------------------------------------------------------------------

### 🧱 Repositories

Les **repositories** représentent des **collections d'agrégats**.

-   Le domaine définit une **interface** (`ProductRepository`).
-   L'infrastructure fournit **l'implémentation concrète** (Doctrine,
    API...).

------------------------------------------------------------------------

### 🎲 Domain Events

Les **événements de domaine** représentent des faits importants qui se
sont produits dans le domaine.

Exemple : `ProductPriceChanged`, `OrderPlaced`, `CustomerRegistered`.

Ils permettent : - Le **découplage** entre parties du domaine, -
L'**audit** et la traçabilité, - L'intégration avec d'autres contextes
(via EventBus, par exemple).

#### Exemple :

``` php
final class ProductPriceChanged
{
    public function __construct(
        public readonly string $productId,
        public readonly float $newPrice
    ) {}
}
```

------------------------------------------------------------------------

### 🏭 Factories

Les **factories** centralisent la création complexe d'objets (agrégats,
entités...).

Elles sont utiles quand : - La création d'un objet nécessite beaucoup de
logique, - L'initialisation dépend de règles métier.

Exemple :

``` php
final class OrderFactory
{
    public function create(CustomerId $customerId): Order
    {
        return new Order(Uuid::uuid4()->toString(), $customerId);
    }
}
```

------------------------------------------------------------------------

### 🧩 Policies (Règles transversales)

Les **policies** définissent des règles qui peuvent s'appliquer à
plusieurs agrégats.\
Exemple : "Un client ne peut pas commander s'il a des factures
impayées".

Ces règles peuvent être exprimées comme des **Domain Services**.

------------------------------------------------------------------------

### 🧱 Anti-Corruption Layer (ACL)

Lorsque tu dois **intégrer un autre système** (externe ou ancien), l'ACL
agit comme une **barrière**.\
Elle traduit les modèles externes en ton modèle DDD, pour **protéger la
cohérence du domaine**.

------------------------------------------------------------------------

## 🧪 Tests et maintenabilité

-   Les **tests du domaine** : unitaires, sans base de données.\
-   Les **tests d'application** : testent les use cases avec des
    repositories en mémoire.\
-   Les **tests d'intégration** : valident l'infrastructure (Doctrine,
    API...).

➡️ Objectif : garantir un code **robuste, découplé et évolutif**.

------------------------------------------------------------------------

## 📐 Exemple d'arborescence DDD (simplifiée)

    src/
    ├── Catalog/
    │   ├── Domain/
    │   │   ├── Entity/
    │   │   ├── ValueObject/
    │   │   ├── Aggregate/
    │   │   ├── Event/
    │   │   ├── Repository/
    │   │   ├── Service/
    │   │   ├── Factory/
    │   │   └── Policy/
    │   ├── Application/
    │   │   ├── Command/
    │   │   ├── Handler/
    │   │   └── DTO/
    │   └── Infrastructure/
    │       ├── Doctrine/
    │       ├── Symfony/
    │       └── Messaging/

------------------------------------------------------------------------

## 🚀 Avantages du DDD

-   ✅ Code aligné avec le **métier réel**
-   ✅ Logique claire et **modulaire**
-   ✅ Indépendance du framework et de la base de données
-   ✅ **Cohérence** grâce aux agrégats et événements
-   ✅ Architecture **durable et testable**

# 🧭 Comprendre le Domain-Driven Design (DDD)

Le **Domain-Driven Design** est une approche de conception logicielle
qui met **le domaine métier au centre du code**.\
Son objectif est de créer un code qui **reflète fidèlement la logique du
métier** et **évolue avec lui**, plutôt que d'être dicté par la
technique (framework, base de données, etc.).

------------------------------------------------------------------------

## 🧩 Les 3 couches fondamentales

### 1. **Domain Layer (le cœur du métier)**

C'est **le cœur de ton application** : il contient les **règles
métier**, les **entités**, les **Value Objects**, et parfois des
**domain services**.

#### Exemples :

-   `Product`, `Order`, `Customer` : des **entités** avec identité
    unique.\
-   `Money`, `Email`, `Quantity` : des **Value Objects** (valeurs sans
    identité).\
-   `PricingPolicy`, `DiscountService` : des **Domain Services**
    (logique métier ne dépendant pas d'une seule entité).

➡️ Le **Domain** ne dépend de rien d'autre.\
Il ne connaît **ni Symfony, ni Doctrine, ni HTTP, ni la base de
données**.

#### Exemple :

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

Ce `Value Object` encapsule la logique du prix (validation, addition,
cohérence des devises) sans dépendre d'aucune technologie.

------------------------------------------------------------------------

### 2. **Application Layer (les cas d'usage)**

C'est le **chef d'orchestre** : il exécute des **use cases** (actions
métier complètes) comme : - "Créer un produit" - "Mettre à jour un
prix" - "Passer une commande"

Chaque cas d'usage est implémenté dans un **service d'application**.\
Il coordonne les entités et services du domaine, appelle les
**repositories**, et publie éventuellement des **événements de
domaine**.

#### Exemple :

``` php
final class UpdateProductPriceHandler
{
    public function __construct(
        private ProductRepository $repository
    ) {}

    public function __invoke(UpdateProductPriceCommand $command): void
    {
        $product = $this->repository->find($command->id);
        $product->changePrice(new Price($command->price, 'EUR'));
        $this->repository->save($product);
    }
}
```

> 💡 L'Application layer **ne contient pas** la logique métier --- elle
> **l'orchestration** seulement.

------------------------------------------------------------------------

### 3. **Infrastructure Layer (le monde extérieur)**

C'est la couche où tu interagis avec le **monde technique** : - Base de
données (Doctrine, SQL...) - Framework Symfony (contrôleurs,
événements) - API externes, services tiers - Files d'attente, logs, etc.

C'est ici que tu **implémentes les interfaces** définies dans le domaine
(ex. `ProductRepository`).

#### Exemple :

``` php
final class DoctrineProductRepository implements ProductRepository
{
    public function __construct(private EntityManagerInterface $em) {}

    public function find(string $id): Product
    {
        return $this->em->find(Product::class, $id);
    }

    public function save(Product $product): void
    {
        $this->em->persist($product);
        $this->em->flush();
    }
}
```

➡️ Cette couche dépend de Symfony, Doctrine, etc.\
Mais **le domaine** et **l'application** n'en savent rien.

------------------------------------------------------------------------

## 🧠 Concepts clés du DDD

### 🗣️ Ubiquitous Language (langage omniprésent)

-   C'est **le vocabulaire partagé** entre les développeurs et les
    experts métier.
-   Tu le réutilises **dans ton code** (noms de classes, méthodes,
    tests...).

> Exemple : ne dis pas `AddItemToBasket()` si le métier dit "ajouter un
> produit au panier".

------------------------------------------------------------------------

### 🏠 Bounded Contexts

Une **application complexe** est souvent divisée en plusieurs
**contextes délimités** (bounded contexts).

Par exemple, dans ton e-commerce : - `Catalog` (produits, catégories) -
`Sales` (commandes, paiements) - `Inventory` (stocks) - `Customer`
(comptes, fidélité)

Chaque contexte a son propre **modèle métier**, parfois différent pour
un même concept.

> Un "Produit" dans `Catalog` n'a pas forcément les mêmes attributs
> qu'un "Produit" dans `Inventory`.

------------------------------------------------------------------------

### 🔁 Repositories & Interfaces

Les **repositories** abstraient la persistance : - Le domaine les
utilise **comme interfaces** (`ProductRepository`). - L'infrastructure
les **implémente** (Doctrine, API...).

Ainsi, le domaine reste **indépendant du stockage**.

------------------------------------------------------------------------

## 🧪 Tests et maintenabilité

-   Les **tests du domaine** sont **unitaires**, sans base de données.
-   Les **tests d'application** peuvent simuler les repositories.
-   Les **tests d'infrastructure** sont plus rares (intégration).

➡️ Résultat : ton code est **modulaire**, **testable**, et **évolutif**.

------------------------------------------------------------------------

## 📐 Exemple d'arborescence DDD (simplifiée)

    src/
    ├── Catalog/
    │   ├── Domain/
    │   │   ├── Entity/
    │   │   ├── ValueObject/
    │   │   ├── Repository/
    │   │   └── Service/
    │   ├── Application/
    │   │   ├── Command/
    │   │   ├── Handler/
    │   │   └── DTO/
    │   └── Infrastructure/
    │       ├── Doctrine/
    │       └── Symfony/

------------------------------------------------------------------------

## 🚀 Avantages du DDD

-   ✅ Code aligné avec le **métier réel**
-   ✅ Facilité d'évolution et de test
-   ✅ Indépendance vis-à-vis du framework
-   ✅ Architecture claire et durable

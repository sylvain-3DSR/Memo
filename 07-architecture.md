# 🏗️ Architecture & DDD
## 🎯 Objectif
Structurer ton code selon Domain-Driven Design (DDD).

## 📦 Couches principales
- **Domain** → logique métier pure  
- **Application** → cas d’usage, orchestration  
- **Infrastructure** → base de données, API, framework  

## Exemple d’entité
```php
class Product
{
    public function __construct(
        public string $name,
        public float $price
    ) {}
}
```

## Exemple de service d’application
```php
class ProductService
{
    public function __construct(private ProductRepository $repository) {}

    public function updatePrice(string $id, float $price): void
    {
        $product = $this->repository->find($id);
        $product->price = $price;
        $this->repository->save($product);
    }
}
```

## 🧠 Bonnes pratiques
- ✅ Domain sans dépendances Symfony
- ✅ Respecter SOLID
- ✅ Utiliser les Value Objects
- ✅ Tester le domaine sans mocks d’infrastructure

# 🧱 Programmation orientée objet (OOP)
## 🎯 Objectif
Appliquer les principes SOLID et utiliser une POO moderne.

## Exemple simple
```php
class Product
{
    public function __construct(
        public string $name,
        public float $price
    ) {}
}
```

## Exemple avec héritage
```php
class Instrument extends Product
{
    public string $category = 'Music';
}
```

## 🧠 Bonnes pratiques
- ✅ Respecter les principes SOLID
- ✅ Utiliser l’injection de dépendances
- ✅ Rendre les entités immuables si possible

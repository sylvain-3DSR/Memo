# 🧩 Fonctions
## 🎯 Objectif
Écrire des fonctions claires, typées et réutilisables.

## ✅ Exemple de base
```php
function greet(string $name): string
{
    return "Bonjour $name !";
}
```

## ⚙️ Fonctions anonymes / fléchées
```php
$add = fn(int $a, int $b): int => $a + $b;
```

## 🧠 Bonnes pratiques
- ✅ Toujours typer les arguments et retours
- ✅ Utiliser les promotions de propriété dans les constructeurs (PHP 8+)
- ✅ Documenter avec PHPDoc si nécessaire

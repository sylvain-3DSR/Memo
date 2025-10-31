# ✅ Conditions
## 🎯 Objectif
Écrire des conditions plus concises, expressives et sûres.

## 🔰 Exemple de base
```php
if ($data && isset($data['value'])) {
    $value = $data['value'];
} else {
    $value = false;
}
```

## 🟢 PHP 7.4 – Version simplifiée
```php
$value = $data['value'] ?? false;
```

## 🟣 PHP 8.x – Version moderne
```php
$value = $object?->getData()?->value ?? false;
```

## 🧠 Bonnes pratiques
- ✅ Utiliser `??` plutôt que `isset()`
- ✅ Préférer le `?->` pour naviguer dans les objets
- ✅ Favoriser `match` pour remplacer certains `switch`

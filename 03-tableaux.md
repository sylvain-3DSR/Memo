# 🧮 Tableaux
## 🎯 Objectif
Manipuler les tableaux de manière fluide et idiomatique.

## 🧩 Exemples
```php
$fruits = ['pomme', 'banane', 'cerise'];

$fruits[] = 'orange';
unset($fruits[1]);
$all = array_merge($fruits, ['kiwi']);
$filtered = array_filter($fruits, fn($f) => $f !== 'banane');
$upper = array_map('strtoupper', $fruits);
```

## 🧠 Bonnes pratiques
- ✅ Utiliser `array_map`, `array_filter`, `array_reduce`
- ✅ Éviter la mutation inutile
- ✅ Exploiter la déstructuration :
```php
[$first, $second] = $fruits;
```

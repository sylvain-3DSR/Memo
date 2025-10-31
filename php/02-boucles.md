# 🔁 Boucles
## 🎯 Objectif
Parcourir des collections de manière claire et moderne.

## 🔰 Exemple de base
```php
foreach ($users as $user) {
    echo $user['name'];
}
```

## ⚙️ Boucle avec index
```php
foreach ($users as $id => $user) {
    echo "#$id : {$user['name']}";
}
```

## 🧩 Approche fonctionnelle
```php
$names = array_map(fn($u) => $u['name'], $users);
```

## 🧠 Bonnes pratiques
- ✅ Utiliser `foreach` par défaut
- ✅ Préférer `array_map` / `array_filter` pour du code plus expressif
- ❌ Éviter `while` ou `for` sauf besoin particulier

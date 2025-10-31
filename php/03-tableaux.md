# 🧮 Tableaux

## 🎯 Objectif
Savoir **manipuler les tableaux de manière fluide, expressive et moderne**,  
en utilisant les nouveautés de PHP **8.x** tout en comprenant les pratiques de **PHP 7.4**.

Les tableaux (`array`) sont **la structure de données la plus utilisée en PHP** :  
ils peuvent représenter des listes, des dictionnaires (clés/valeurs), ou même des objets légers.

---

## 🧩 1. Déclaration et manipulation de base

### ⚙️ PHP 7.4 — syntaxe classique
```php
$fruits = ['pomme', 'banane', 'cerise'];

$fruits[] = 'orange';              // Ajout d’un élément
unset($fruits[1]);                 // Suppression de l’index 1
$merged = array_merge($fruits, ['kiwi']); // Fusion de deux tableaux
```

### ✅ PHP 8.x — syntaxe identique, mais plus expressive
```php
$fruits = ['pomme', 'banane', 'cerise', ...['orange', 'kiwi']];
// L’opérateur “spread” (...) étend un tableau à l’intérieur d’un autre
```

### 📚 Explication
- En **PHP 7.4**, `array_merge()` était obligatoire pour fusionner plusieurs tableaux.  
- En **PHP 8.0+**, l’opérateur **spread (`...`)** permet de les combiner directement :
  ```php
  $combined = [...$fruits, ...$moreFruits];
  ```
  → Plus simple, plus rapide, plus lisible.

---

## 🧩 2. Fonctions utilitaires (`map`, `filter`, `reduce`)

### ⚙️ PHP 7.4
```php
$numbers = [1, 2, 3, 4, 5];

$doubled = array_map(fn($n) => $n * 2, $numbers);
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
```

### ✅ PHP 8.x — mêmes fonctions, mais avec flèches améliorées
```php
$numbers = [1, 2, 3, 4, 5];

$doubled = array_map(fn($n) => $n * 2, $numbers);
$evens = array_filter($numbers, fn($n) => $n % 2 === 0);
$sum = array_reduce($numbers, fn($carry, $n) => $carry + $n, 0);
```

### 📚 Explication
- Ces fonctions sont **disponibles depuis PHP 5**, mais deviennent **plus lisibles** grâce aux **arrow functions (`fn`)** introduites en PHP 7.4.  
- Elles favorisent un style **fonctionnel** : au lieu de modifier le tableau, tu **crées un nouveau résultat**.

💡 *PHP 8.1 a aussi amélioré les performances internes d’`array_map` et `array_filter`.*

---

## 🧩 3. Tableaux associatifs

### ⚙️ PHP 7.4
```php
$user = [
    'name' => 'Alice',
    'age' => 28,
];

echo $user['name']; // Alice
```

### ✅ PHP 8.x — avec “spread” pour fusionner
```php
$base = ['name' => 'Alice', 'age' => 28];
$extra = ['role' => 'admin'];

$user = [...$base, ...$extra]; // fusion propre sans array_merge()
```

### 📚 Explication
Avant PHP 8.0, tu devais utiliser :
```php
$user = array_merge($base, $extra);
```
Depuis PHP 8.0 :
- le spread `...` est **plus rapide et plus lisible**,  
- il **préserve les clés** en cas de tableaux associatifs (contrairement à `array_merge` qui peut les réindexer).

---

## 🧩 4. Déstructuration (destructuring)

La **déstructuration** permet d’extraire facilement plusieurs valeurs d’un tableau en une ligne.

### ⚙️ PHP 7.4
```php
$fruits = ['pomme', 'banane', 'cerise'];
[$first, $second, $third] = $fruits;

echo $first;  // pomme
echo $second; // banane
```

### ✅ PHP 8.x — déstructuration avec clés nommées
```php
$user = ['name' => 'Alice', 'age' => 28, 'role' => 'admin'];

['name' => $name, 'role' => $role] = $user;

echo $name; // Alice
echo $role; // admin
```

### 📚 Explication
- En **PHP 7.4**, la déstructuration ne fonctionnait qu’avec des **index numériques**.  
- En **PHP 8.1+**, tu peux extraire **des clés nommées** directement → idéal pour les tableaux associatifs.

---

## 🧩 5. Tri et transformations

### ⚙️ PHP 7.4 — tri classique
```php
$fruits = ['pomme', 'banane', 'cerise'];
sort($fruits); // Trie par ordre alphabétique
```

### ✅ PHP 8.x — tri personnalisé
```php
$fruits = ['pomme', 'banane', 'cerise'];
usort($fruits, fn($a, $b) => strlen($a) <=> strlen($b));
```

### 📚 Explication
- `usort()` permet un tri **personnalisé** à l’aide d’une fonction de comparaison.  
- L’opérateur **spatial (`<=>`)** compare directement deux valeurs :
  - retourne `-1` si $a < $b  
  - retourne `0` si $a == $b  
  - retourne `1` si $a > $b  
→ C’est une **simplification syntaxique** introduite en PHP 7.0 mais très utilisée à partir de PHP 8.x.

---

## 🧩 6. Nouveautés PHP 8.x sur les tableaux

### 🧠 Clés en chaînes plus souples (PHP 8.0)
```php
$array = ['1' => 'a', 1 => 'b'];
var_dump($array);
// Résultat : ['1' => 'b'] — les clés numériques et chaînes identiques fusionnent
```

### 🧠 spread dans les expressions (PHP 8.1+)
```php
$colors = [...['rouge', 'vert'], ...['bleu']];
```

### 🧠 `array_is_list()` (PHP 8.1)
```php
$fruits = ['pomme', 'banane', 'cerise'];
var_dump(array_is_list($fruits)); // true
```
> Vérifie si un tableau est une liste (indexée à partir de 0, sans trous).

---

## 🧠 Bonnes pratiques

- ✅ Utiliser `array_map`, `array_filter`, `array_reduce` plutôt que des `foreach` pour des opérations simples.  
- ✅ Préférer les **tableaux immuables** (créer un nouveau tableau au lieu de modifier l’existant).  
- ✅ Exploiter la **déstructuration** pour extraire des données.  
- ✅ Utiliser l’opérateur **spread (`...`)** au lieu de `array_merge()` dès PHP 8.0.  
- ✅ Utiliser `array_is_list()` pour distinguer tableaux de listes et associatifs.  
- ⚠️ Éviter les mutations dans des boucles imbriquées pour garder le code lisible et prévisible.  

---

## 🧾 Résumé des apports PHP 8.x (vs 7.4)

| Fonctionnalité | Introduite en | Description | Exemple |
|----------------|---------------|--------------|----------|
| **Arrow functions `fn()`** | 7.4 | Fonctions anonymes plus lisibles | `array_map(fn($x) => $x*2, $arr)` |
| **Spread `...` pour tableaux** | 8.0 | Fusion directe sans `array_merge()` | `['a', ...$b]` |
| **Déstructuration avec clés** | 8.1 | Extraction par clés associatives | `['name'=>$n] = $user;` |
| **`array_is_list()`** | 8.1 | Vérifie si un tableau est une liste simple | `array_is_list($arr)` |

---

📘 *En résumé :*  
PHP 8.x rend la manipulation des tableaux :
- plus **expressive** (spread, déstructuration),  
- plus **fonctionnelle** (`map`, `filter`, `reduce`),  
- plus **sûre** (détection de listes, clés explicites),  
- et plus **performante** grâce aux optimisations internes.

# ✅ Conditions

## 🎯 Objectif
Apprendre à écrire des **conditions plus concises, plus sûres et plus lisibles**  
en profitant des nouveautés de PHP **8.x** tout en comprenant les pratiques de **PHP 7.4**.

Les conditions permettent à ton code de **prendre des décisions** : exécuter certains blocs seulement si une expression est vraie.

---

## 🧩 1. Condition classique (if / else)

### 🔰 Exemple de base (approche traditionnelle)
```php
if ($data && isset($data['value'])) {
    $value = $data['value'];
} else {
    $value = false;
}
```

### ⚙️ PHP 7.4 — *simplification avec l’opérateur de fusion null (`??`)*
```php
$value = $data['value'] ?? false;
```

### ✅ PHP 8.x — *navigation sûre avec l’opérateur nullsafe (`?->`)*
```php
$value = $object?->getData()?->value ?? false;
```

### 📚 Explication
- En **PHP 7.4**, l’opérateur `??` renvoie la première valeur non nulle.  
  → Plus besoin de `isset()` avant d’accéder à une clé potentiellement absente.  
- En **PHP 8.0+**, l’opérateur **nullsafe (`?->`)** permet d’appeler une méthode ou propriété **seulement si l’objet n’est pas `null`**.  
  → Évite les erreurs de type *"Call to a member function on null"*.

🧠 Exemple pratique :
```php
$userName = $user?->getProfile()?->getName() ?? 'Invité';
```
> Si `$user` ou `$user->getProfile()` est `null`, la chaîne `'Invité'` sera utilisée.

---

## 🧩 2. Condition ternaire

Une **ternaire** permet d’écrire un `if` / `else` sur une seule ligne.

### PHP 7.4 – forme classique
```php
$status = ($isActive) ? 'Actif' : 'Inactif';
```

### PHP 8.x – identique, mais plus sûre avec `??` ou `?->`
```php
$status = $user?->isActive() ? 'Actif' : 'Inactif';
```

### 📚 Explication
- Les ternaires sont idéales pour **retourner rapidement une valeur simple**.  
- À partir de PHP 8.0, tu peux les combiner avec `?->` ou `??` pour éviter les exceptions.  
- Attention à ne pas imbriquer trop de ternaires → lisibilité diminue vite.

---

## 🧩 3. Condition `switch`

Le **switch** est utile pour comparer une même variable à plusieurs valeurs possibles.

### ⚙️ PHP 7.4 – switch classique
```php
$role = 'editor';

switch ($role) {
    case 'admin':
        $access = 'Accès complet';
        break;
    case 'editor':
        $access = 'Accès limité';
        break;
    default:
        $access = 'Aucun accès';
}
```

### 📚 Inconvénients
- Syntaxe **verbeuse**, répétition de `break`.  
- Comparaison **non stricte** (`==` et non `===`).  
- Impossible d’exprimer des conditions plus complexes sans imbriquer.

---

## 🧩 4. `match` (PHP 8.0+) — la version moderne du switch

### ✅ PHP 8.x – syntaxe concise et expressive
```php
$role = 'editor';

$access = match ($role) {
    'admin'  => 'Accès complet',
    'editor' => 'Accès limité',
    default  => 'Aucun accès',
};
```

### 📚 Explication
- Introduit en **PHP 8.0**, `match` est une **expression** (elle renvoie une valeur).  
- Comparaison **stricte** (`===`) → plus sûre.  
- Pas besoin de `break`.  
- Peut être **imbriquée dans une variable ou un retour de fonction**.  
- Accepte plusieurs valeurs par condition :
  ```php
  $status = match ($code) {
      200, 201 => 'Succès',
      400, 404 => 'Erreur client',
      500      => 'Erreur serveur',
      default  => 'Inconnu',
  };
  ```

🧠 *`match` rend le code plus court, plus sûr et plus lisible que `switch`.*

---

## 🧩 5. Conditions combinées (avec opérateurs logiques)

### PHP 7.4 — opérateurs de base
```php
if ($isAdmin && $isLoggedIn) {
    echo "Bienvenue, administrateur.";
}
```

### PHP 8.x — idem, mais avec des expressions plus sûres
```php
if ($user?->isLoggedIn() && $user?->hasRole('admin')) {
    echo "Bienvenue, administrateur.";
}
```

➡️ Grâce au **nullsafe operator (`?->`)**, tu peux éviter de vérifier si `$user` est null avant d’appeler ses méthodes.

---

## 🧩 6. `match` avancé — logique conditionnelle complexe

### PHP 8.x – avec expressions et retours multiples
```php
$result = match (true) {
    $score >= 90 => 'Excellent',
    $score >= 75 => 'Bien',
    $score >= 50 => 'Passable',
    default => 'Insuffisant',
};
```

### 📚 Explication
Ici, `match(true)` permet de transformer un bloc conditionnel en expression fluide :
> Chaque condition est testée jusqu’à la première vraie (`true`).

Cela remplace :
```php
if ($score >= 90) {
    $result = 'Excellent';
} elseif ($score >= 75) {
    $result = 'Bien';
} elseif ($score >= 50) {
    $result = 'Passable';
} else {
    $result = 'Insuffisant';
}
```
💡 *Résultat identique, mais code plus compact et plus clair.*

---

## 🧠 Bonnes pratiques

- ✅ **Préférer `??` à `isset()`** → plus lisible et concis.  
- ✅ **Utiliser `?->`** pour éviter les erreurs de type sur des objets nuls.  
- ✅ **Préférer `match`** à `switch` dès PHP 8.0+.  
- ✅ **Utiliser `match(true)`** pour remplacer plusieurs `if / elseif`.  
- ⚠️ Ne pas abuser des ternaires imbriquées → privilégier `if` pour la lisibilité.  
- ✅ Toujours activer `declare(strict_types=1);` pour des comparaisons strictes.  

---

## 🧾 Résumé des apports PHP 8.x (vs 7.4)

| Fonctionnalité | Introduite en | Description | Exemple |
|----------------|---------------|--------------|----------|
| **Opérateur de fusion null `??`** | 7.0 | Retourne la 1ère valeur non nulle | `$value = $data['x'] ?? false;` |
| **Opérateur nullsafe `?->`** | 8.0 | Appelle une méthode uniquement si l’objet n’est pas `null` | `$user?->getName()` |
| **`match`** | 8.0 | Version stricte, expressive et concise de `switch` | `match($var) { ... }` |

---

📘 *En résumé :*  
PHP 8.x modernise les conditions en rendant leur écriture :
- plus **sûre** (comparaison stricte, gestion de `null`),  
- plus **courte** (moins de répétition),  
- plus **claire** (avec `match` et `?->`).  
Ces outils rendent le code plus lisible, plus robuste et plus professionnel.

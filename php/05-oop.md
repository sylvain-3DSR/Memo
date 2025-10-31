# 🧱 Programmation orientée objet (OOP)

## 🎯 Objectif
Apprendre à concevoir et structurer le code selon les principes de la **Programmation Orientée Objet (POO)**.  
L’objectif est d’écrire un code :
- plus lisible,  
- plus réutilisable,  
- plus facile à tester et à maintenir,  
en appliquant les principes **SOLID**.

📘 Ce chapitre présente les **concepts clés** de la POO en PHP, avec des comparaisons entre **PHP 7.4** et **PHP 8.4 / 8.5** pour comprendre les évolutions majeures du langage.

---

## 🧩 1. Classe concrète (de base)

Une **classe concrète** est une classe instanciable.  
Elle regroupe des **propriétés** (données) et des **méthodes** (comportements).

### ✅ PHP 8.x (≥ 8.0) — *promotion de propriété*
```php
class Product
{
    public function __construct(
        public string $name,
        public float $price
    ) {}

    public function getFormattedPrice(): string
    {
        return number_format($this->price, 2) . ' €';
    }
}
```

### ⚙️ PHP 7.4 — *ancienne écriture*
```php
class Product
{
    public string $name;
    public float $price;

    public function __construct(string $name, float $price)
    {
        $this->name = $name;
        $this->price = $price;
    }

    public function getFormattedPrice(): string
    {
        return number_format($this->price, 2) . ' €';
    }
}
```

### 📚 Explication
👉 **La promotion de propriété** (PHP 8.0+) permet de déclarer et d’initialiser les propriétés directement dans la signature du constructeur.  
Cela réduit la répétition et rend le code plus lisible.

**PHP 7.4 :** déclaration + initialisation manuelle.  
**PHP 8.0 :** tout en une ligne → gain de concision et de clarté.

---

## 🪶 2. Héritage

L’**héritage** permet à une classe fille de réutiliser et spécialiser le comportement d’une classe parent.

```php
class Instrument extends Product
{
    public string $category = 'Music';

    public function play(): void
    {
        echo "Playing {$this->name}...";
    }
}
```

### ⚙️ PHP 8.2+ — *propriétés immuables (readonly)*
```php
class Instrument extends Product
{
    public readonly string $category = 'Music';
}
```

### 📚 Explication
- En **PHP 7.4**, il fallait protéger la modification des propriétés via des *getters* (lecture seule simulée).  
- En **PHP 8.2**, le mot-clé `readonly` indique qu’une propriété ne peut être modifiée qu’une seule fois (à la construction).  
  → Cela renforce la **sécurité** et la **prévisibilité** des objets.

---

## 🧱 3. Abstraction

Une **classe abstraite** est une classe **non instanciable**, qui sert de **modèle commun**.  
Elle peut définir :
- des méthodes concrètes (implémentées),  
- des méthodes abstraites (à implémenter par les enfants).

### Exemple générique
```php
abstract class PaymentMethod
{
    protected float $amount;

    public function __construct(float $amount)
    {
        $this->amount = $amount;
    }

    abstract public function pay(): bool;

    public function getAmount(): float
    {
        return $this->amount;
    }
}
```

### PHP 8.1+ — *constantes `final` et propriétés `readonly`*
```php
abstract class PaymentMethod
{
    final public const TYPE = 'GENERIC';
    public readonly float $amount;

    public function __construct(float $amount)
    {
        $this->amount = $amount;
    }

    abstract public function pay(): bool;
}
```

### 📚 Explication
- Les classes abstraites existent depuis PHP 5.  
- PHP 8.1 a ajouté :  
  - les **constantes `final`** → pour empêcher leur redéfinition,  
  - les **propriétés `readonly`** → pour garantir l’immuabilité.  
Cela renforce la **stabilité** des classes de base dans les architectures métier (DDD).

---

## 🧩 4. Interface

Une **interface** définit un **contrat** :  
les méthodes que toute classe qui l’implémente doit contenir.  
Elle ne contient **aucune implémentation**.

```php
interface LoggerInterface
{
    public function info(string $message): void;
    public function error(string $message): void;
}
```

### PHP 8.0+ — *types unions et mixed*
```php
interface TransportInterface
{
    public function send(string|array $payload): bool;
}
```

### 📚 Explication
- En **PHP 7.4**, on ne pouvait typer qu’un seul type de paramètre (`string` **ou** `array`).  
- En **PHP 8.0**, on peut combiner plusieurs types avec `|` (union types).  
Cela simplifie le code et évite de dupliquer des méthodes pour gérer plusieurs formats.

---

## ⚙️ 5. Interface + Abstraction (modèle complet)

```php
interface NotifierInterface
{
    public function send(string $message): void;
}

abstract class AbstractNotifier implements NotifierInterface
{
    public function log(string $message): void
    {
        echo "[LOG] $message\n";
    }
}

class EmailNotifier extends AbstractNotifier
{
    public function send(string $message): void
    {
        $this->log("Email envoyé : $message");
    }
}
```

### PHP 8.4+ — *Property Hooks (nouveauté)*
```php
class EmailNotifier extends AbstractNotifier
{
    public string $sender { set(string $s) { $this->sender = strtolower($s); } }

    public function send(string $message): void
    {
        $this->log("Email envoyé par {$this->sender}: $message");
    }
}
```

### 📚 Explication
Les **Property Hooks** (PHP 8.4) permettent de définir un code exécuté automatiquement
lors de l’accès ou la modification d’une propriété (`get` / `set`).  
C’est une amélioration majeure de la POO :
- plus besoin de *getters/setters* explicites,  
- syntaxe plus claire et expressive.  

💡 *Fonctionnalité absente en PHP 7.4.*

---

## 🧩 6bis. Qu’est-ce que l’injection de dépendances ?

### 💡 Définition
L’**injection de dépendances (DI)** consiste à **fournir à une classe ses dépendances depuis l’extérieur**, plutôt que de les créer elle-même à l’intérieur.  
Cela favorise le **découplage**, la **testabilité** et la **modularité** du code.

### 🚫 Mauvais exemple — sans injection
```php
class CheckoutService
{
    private StripePaymentProcessor $processor;

    public function __construct()
    {
        $this->processor = new StripePaymentProcessor(); // dépendance en dur ❌
    }

    public function checkout(float $amount): void
    {
        $this->processor->process($amount);
    }
}
```

### ✅ Bon exemple — avec injection
```php
class CheckoutService
{
    public function __construct(private PaymentProcessorInterface $processor) {}

    public function checkout(float $amount): void
    {
        $this->processor->process($amount);
    }
}

$service = new CheckoutService(new StripePaymentProcessor());
```
➡️ Avantages :  
- Découplage complet entre la classe et ses dépendances.  
- Plus facile à tester avec des mocks ou fakes.  
- Permet de remplacer facilement l’implémentation (ex : Stripe → PayPal).  

---

## 🧩 6ter. Qu’est-ce qu’une entité immuable ?

### 💡 Définition
Une **entité immuable** est un objet dont l’état ne change jamais après sa création.

```php
class Money
{
    public function __construct(
        public readonly float $amount,
        public readonly string $currency
    ) {}
}
```

```php
$price = new Money(100.0, 'EUR');
$price->amount = 200.0; // ❌ Erreur : propriété readonly
```

➡️ En PHP 8.2+, `readonly` garantit cette immuabilité.  
En PHP 7.4, on simulait ce comportement avec des propriétés `private` et des *getters* sans *setters*.

🎯 Avantages :  
- Données stables et cohérentes.  
- Moins d’effets de bord.  
- Code plus sûr et prévisible.

---

## 🧱 6quater. Pourquoi combiner interface + abstraction ?

### 💡 Idée clé
Associer **interface** et **classe abstraite** permet de combiner les forces des deux :
- L’interface définit un **contrat** clair.  
- La classe abstraite fournit du **comportement commun**.  
- Les classes concrètes restent libres d’étendre ou de modifier ce comportement.

### 🔧 Exemple concret
```php
interface NotifierInterface
{
    public function send(string $message): void;
}

abstract class AbstractNotifier implements NotifierInterface
{
    public function log(string $message): void
    {
        echo "[LOG] $message\n";
    }
}

class SmsNotifier extends AbstractNotifier
{
    public function send(string $message): void
    {
        $this->log("SMS envoyé : $message");
    }
}
```

### 📚 Explication
- L’interface impose un contrat (chaque notifier doit savoir `send`).  
- La classe abstraite mutualise le code commun (`log()`).  
- Les classes concrètes restent libres d’implémenter leur propre logique.

🎯 Avantage : on réduit la duplication, tout en maintenant la flexibilité.

---

## 🧠 7. Bonnes pratiques générales

- ✅ Respecter les principes **SOLID** :
  - **S** : une classe = une seule responsabilité  
  - **O** : ouverte à l’extension, fermée à la modification  
  - **L** : les sous-classes peuvent remplacer la classe parente  
  - **I** : interfaces spécifiques (éviter les “grosses” interfaces)  
  - **D** : dépendre d’abstractions, pas de classes concrètes  
- ✅ Utiliser l’**injection de dépendances**  
- ✅ Rendre les **entités immuables**  
- ✅ Activer le **typage strict** (`declare(strict_types=1);`)  
- ✅ En PHP 8.2+, exploiter `readonly` pour la sécurité  
- ✅ En PHP 8.4+, utiliser les *Property Hooks* pour plus de clarté

---

## 💡 8. Exemple final — Code “SOLID” et moderne

```php
interface PaymentProcessorInterface
{
    public function process(float $amount): bool;
}

class StripePaymentProcessor implements PaymentProcessorInterface
{
    public function process(float $amount): bool
    {
        echo "Paiement Stripe de {$amount}€ effectué.";
        return true;
    }
}

class CheckoutService
{
    public function __construct(private PaymentProcessorInterface $processor) {}

    public function checkout(float $amount): void
    {
        if ($this->processor->process($amount)) {
            echo "✔️ Paiement confirmé.";
        }
    }
}
```

---

## 🧾 Résumé des apports de PHP 8.x (par rapport à 7.4)

| Fonctionnalité | Introduite en | Description |
|----------------|---------------|--------------|
| **Promotion de propriété** | 8.0 | Déclaration des propriétés directement dans le constructeur |
| **Union types (`|`)** | 8.0 | Combiner plusieurs types de paramètres ou retours |
| **Typage `mixed`, `static`, `never`** | 8.0–8.1 | Typage plus précis |
| **Propriétés `readonly`** | 8.2 | Empêche toute modification après initialisation |
| **Property Hooks (`get`, `set`)** | 8.4 | Code exécuté lors de la lecture/écriture d’une propriété |
| **Constantes `final`** | 8.1 | Empêche la redéfinition dans les sous-classes |

---

📘 *En résumé :*  
PHP 8.x rend la POO **plus concise, plus sûre et plus expressive** qu’en 7.4,  
grâce à une syntaxe simplifiée et de nouveaux outils de conception orientée objet.

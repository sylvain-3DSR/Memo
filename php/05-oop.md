# 🧱 Programmation orientée objet (OOP)

## 🎯 Objectif
Appliquer les principes **SOLID**, structurer le code selon les responsabilités et exploiter pleinement les capacités de la **POO moderne**
(compatibilité PHP 7.4 → 8.4 / 8.5).

---

## 🧩 Exemple simple (classe concrète)

### ✅ Version PHP 8.x (≥ 8.0) — *promotion de propriété*
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

### ⚙️ Version PHP 7.4 — *sans promotion de propriété*
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

🧠 *La “promotion de propriété” (déclaration directement dans le constructeur)
est une nouveauté PHP 8.0+.*

---

## 🪶 Exemple avec héritage

```php
class Instrument extends Product
{
    public string $category = 'Music';

    public function play(): void
    {
        echo "Playing {$this->name}...";
    }
}

$guitar = new Instrument('Gibson Les Paul', 2499.99);
echo $guitar->getFormattedPrice(); // "2 499,99 €"
$guitar->play(); // "Playing Gibson Les Paul..."
```

### ⚙️ PHP 8.2+ — propriétés immuables (`readonly`)
```php
class Instrument extends Product
{
    public readonly string $category = 'Music';
}
```

➡️ *Les propriétés `readonly` ont été introduites en PHP 8.2.*
Avant cela (en 7.4-8.1), il fallait protéger la modification via un getter sans setter.

---

## 🧱 Exemple d’abstraction (classe abstraite)

Une **classe abstraite** sert de modèle commun.
Elle peut contenir des propriétés, méthodes concrètes et abstraites.

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

### PHP 8.1+ — constantes `final` et propriétés `readonly`
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

---

## 🧩 Exemple d’interface

```php
interface LoggerInterface
{
    public function info(string $message): void;
    public function error(string $message): void;
}
```

### PHP 8.0+ — *types unions / mixed*
```php
interface TransportInterface
{
    public function send(string|array $payload): bool;
}
```

➡️ *Avant PHP 8.0, il fallait écrire deux méthodes séparées ou vérifier manuellement le type.*

---

## ⚙️ Exemple combiné : abstraction + interface

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

### 🧩 PHP 8.4+ — *Property Hooks (nouveauté 8.4)*
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
➡️ *Les **property hooks** permettent d’exécuter du code lors des accès à une propriété (getter/setter implicite).
C’est une nouveauté PHP 8.4, inexistante en PHP 7.4.*

---

## 🧠 Bonnes pratiques

- ✅ Respecter les **principes SOLID** :
  - **S** → une classe = une seule responsabilité
  - **O** → ouverte à l’extension, fermée à la modification
  - **L** → remplaçable par ses sous-classes
  - **I** → interfaces spécifiques, pas trop larges
  - **D** → dépendre d’abstractions, pas de classes concrètes
- ✅ Utiliser l’**injection de dépendances**
- ✅ Rendre les **entités immuables**
- ✅ Activer le **typage strict** (`declare(strict_types=1);`)
- ✅ Préférer les **Value Objects** pour représenter les données métier
- ✅ En PHP 8.2+, tirer parti de `readonly`
- ✅ En PHP 8.4+, utiliser les *Property Hooks* avec parcimonie

---

## 💡 Exemple final (SOLID + clean)

### ✅ PHP 8.x (promotion de propriété + typage strict)
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

### ⚙️ PHP 7.4 (ancienne écriture du constructeur)
```php
class CheckoutService
{
    private PaymentProcessorInterface $processor;

    public function __construct(PaymentProcessorInterface $processor)
    {
        $this->processor = $processor;
    }

    public function checkout(float $amount): void
    {
        if ($this->processor->process($amount)) {
            echo "✔️ Paiement confirmé.";
        }
    }
}
```

➡️ Ce code respecte :
- **OCP** : tu peux ajouter un autre processeur sans modifier `CheckoutService`
- **DIP** : tu dépends d’une interface, pas d’une classe concrète
- **SRP** : chaque classe a une seule responsabilité

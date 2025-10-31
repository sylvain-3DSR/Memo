# 🧱 Programmation orientée objet (OOP)

## 🎯 Objectif
Appliquer les principes **SOLID**, structurer le code selon les responsabilités et exploiter pleinement les capacités de la **POO moderne** (PHP 7.4 → 8.5).

---

## 🧩 Exemple simple (classe concrète)

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

🧠 *L’héritage permet de spécialiser une classe existante sans dupliquer la logique.*

---

## 🧱 Exemple d’abstraction (classe abstraite)

Une **classe abstraite** sert de **modèle commun** à plusieurs classes, mais **ne peut pas être instanciée** directement.  
Elle définit les **comportements communs** et les **méthodes à implémenter**.

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

class CreditCardPayment extends PaymentMethod
{
    public function pay(): bool
    {
        echo "Paiement de {$this->amount}€ par carte.";
        return true;
    }
}

class PaypalPayment extends PaymentMethod
{
    public function pay(): bool
    {
        echo "Paiement de {$this->amount}€ via PayPal.";
        return true;
    }
}
```

🧩 *Chaque sous-classe implémente sa propre logique de paiement, tout en partageant une interface commune (`pay()`).*

---

## 🧩 Exemple d’interface

Une **interface** définit **un contrat** que les classes doivent respecter.  
Elle ne contient **aucune implémentation**.

```php
interface LoggerInterface
{
    public function info(string $message): void;
    public function error(string $message): void;
}

class FileLogger implements LoggerInterface
{
    public function info(string $message): void
    {
        file_put_contents('app.log', "[INFO] $message\n", FILE_APPEND);
    }

    public function error(string $message): void
    {
        file_put_contents('app.log', "[ERROR] $message\n", FILE_APPEND);
    }
}
```

➡️ Toute classe qui implémente `LoggerInterface` **doit** fournir ces deux méthodes.

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

🧠 *L’interface définit le contrat, la classe abstraite fournit une implémentation partielle, et la classe concrète complète le tout.*

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

---

## 💡 Exemple final (SOLID + clean)

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

// Injection d'une dépendance abstraite
$service = new CheckoutService(new StripePaymentProcessor());
$service->checkout(99.99);
```

➡️ Ce code respecte :
- **OCP** : tu peux ajouter un autre processeur sans modifier `CheckoutService`
- **DIP** : tu dépends d’une interface, pas d’une classe concrète
- **SRP** : chaque classe a une seule responsabilité

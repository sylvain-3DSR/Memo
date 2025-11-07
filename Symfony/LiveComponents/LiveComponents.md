# Symfony UX Live Components — Cheatsheet des *Traits* (v2.31) avec exemples

Ce document récapitule les principaux traits fournis par Symfony UX Live Components et propose, pour chacun, un exemple synthétique d’utilisation côté composant et/ou template.

## Sommaire
- [DefaultActionTrait](#defaultactiontrait)
- [ComponentToolsTrait](#componenttoolstrait)
- [ComponentWithFormTrait](#componentwithformtrait)
- [LiveCollectionTrait](#livecollectiontrait)
- [ValidatableComponentTrait](#validatablecomponenttrait)
- [Tableau récapitulatif](#tableau-récapitulatif)
- [Notes pratiques](#notes-pratiques)

---

## `DefaultActionTrait`

**Objet** : fournit l’« action par défaut » permettant le rendu et le re-rendu d’un composant côté client.  
**Usage** : à inclure systématiquement dans un composant annoté `#[AsLiveComponent]`, même sans actions personnalisées.

### Exemple minimal

```php
<?php
namespace App\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\Attribute\LiveProp;

#[AsLiveComponent('hello_badge', template: 'components/hello_badge.html.twig')]
class HelloBadge
{
    use DefaultActionTrait;

    #[LiveProp(writable: true)]
    public string $name = 'World';
}
```

```twig
{# templates/components/hello_badge.html.twig #}
<div {{ attributes }}>
  Hello {{ name }}!
</div>
```

> Ce trait active l’action par défaut attendue par le moteur Live Components pour (re)rendre le composant.


---

## `ComponentToolsTrait`

**Objet** : expose des utilitaires de communication et d’intégration front.  
**Fonctions clés** :
- `emit('event', payload)` / `emitSelf('event', payload)` : communication entre composants (émission d’événements).  
- `dispatchBrowserEvent('event', detail)` : déclenchement d’un événement JavaScript sur le nœud DOM racine (fermeture de modale, focus, notifications, etc.).

### Exemple : émettre un événement et déclencher un événement navigateur

```php
<?php
namespace App\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentToolsTrait;
use Symfony\UX\LiveComponent\Attribute\LiveAction;
use Symfony\UX\LiveComponent\Attribute\LiveProp;

#[AsLiveComponent('notifier', template: 'components/notifier.html.twig')]
class Notifier
{
    use DefaultActionTrait;
    use ComponentToolsTrait; // <-- apporte emit(), emitSelf(), dispatchBrowserEvent()

    #[LiveProp(writable: true)]
    public string $message = 'Prêt.';

    #[LiveAction]
    public function notify(): void
    {
        // Événement côté composants
        $this->emit('notification:created', ['message' => $this->message]);

        // Événement côté navigateur (DOM)
        $this->dispatchBrowserEvent('toast', ['text' => 'Notification envoyée']);
    }
}
```

```twig
{# templates/components/notifier.html.twig #}
<div {{ attributes }}>
  <input type="text" data-model="debounce(300)|message" placeholder="Message..." />
  <button {{ live_action('notify') }}>Notifier</button>
</div>

<script>
// Exemple JS minimal pour consommer l'événement navigateur "toast"
document.addEventListener('toast', (e) => {
  // e.detail.text peut être utilisé pour un toast custom
  console.log('[toast]', e.detail?.text);
});
</script>
```

> `emit()` permet de chaîner des comportements entre composants ; `dispatchBrowserEvent()` permet de déléguer une partie UX au front sans écrire de logique Ajax supplémentaire.


---

## `ComponentWithFormTrait`

**Objet** : facilite l’intégration d’un **Form Symfony** dans un composant live (instanciation, soumission Ajax, réinitialisation).

### Exemple : formulaire live simple

```php
<?php
namespace App\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentWithFormTrait;
use Symfony\UX\LiveComponent\Attribute\LiveAction;
use Symfony\Component\Form\Extension\Core\Type\TextType;
use Symfony\Component\Form\FormFactoryInterface;
use Symfony\Component\Form\FormInterface;

#[AsLiveComponent('profile_editor', template: 'components/profile_editor.html.twig')]
class ProfileEditor
{
    use DefaultActionTrait;
    use ComponentWithFormTrait; // <-- fournit getForm(), submitForm(), resetForm()

    public function __construct(private FormFactoryInterface $formFactory) {}

    protected function instantiateForm(): FormInterface
    {
        return $this->formFactory->createBuilder()
            ->add('displayName', TextType::class, ['required' => true, 'label' => 'Nom affiché'])
            ->getForm();
    }

    #[LiveAction]
    public function save(): void
    {
        $this->submitForm(); // Hydrate depuis la requête LiveComponent et valide le formulaire
        if ($this->getForm()->isValid()) {
            $data = $this->getForm()->getData();
            // ... persistance / traitement ...
            $this->resetForm();
        }
    }
}
```

```twig
{# templates/components/profile_editor.html.twig #}
<div {{ attributes }}>
  {{ form_start(this.form) }}
    {{ form_row(this.form.displayName) }}
    <button {{ live_action('save') }}>Enregistrer</button>
  {{ form_end(this.form) }}
</div>
```

> `ComponentWithFormTrait` gère le cycle de vie du formulaire (instanciation, soumission, reset) côté composant.


---

## `LiveCollectionTrait`

**Objet** : ajoute des aides pour manipuler des **collections** de formulaires (`CollectionType`) en live, en particulier avec `LiveCollectionType`.

### Exemple : collection dynamique d’items (ajout/suppression)

```php
<?php
namespace App\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\ComponentWithFormTrait;
use Symfony\UX\LiveComponent\LiveCollectionTrait;
use Symfony\Component\Form\FormFactoryInterface;
use Symfony\Component\Form\FormInterface;
use Symfony\Component\Form\Extension\Core\Type\CollectionType;
use Symfony\UX\LiveComponent\Form\Type\LiveCollectionType;
use Symfony\Component\Form\Extension\Core\Type\TextType;

#[AsLiveComponent('todo_manager', template: 'components/todo_manager.html.twig')]
class TodoManager
{
    use DefaultActionTrait;
    use ComponentWithFormTrait;
    use LiveCollectionTrait; // <-- helpers add/remove d’éléments

    public function __construct(private FormFactoryInterface $formFactory) {}

    protected function instantiateForm(): FormInterface
    {
        return $this->formFactory->createBuilder()
            ->add('items', LiveCollectionType::class, [
                'entry_type' => TextType::class,
                'allow_add' => true,
                'allow_delete' => true,
                'prototype' => true,
                // Optionnel : 'entry_options' => ['label' => false],
            ])
            ->getForm();
    }
}
```

```twig
{# templates/components/todo_manager.html.twig #}
<div {{ attributes }}>
  {{ form_start(this.form) }}
    <div data-collection-holder>
      {{ form_widget(this.form.items) }}
    </div>

    {# Boutons pilotés par LiveCollectionTrait #}
    <button type="button"
            data-action="live#action"
            data-live-action-param="live:collection:add"
            data-live-name-param="{{ this.form.items.vars.full_name }}">
      Ajouter un item
    </button>

    <button type="button"
            data-action="live#action"
            data-live-action-param="live:collection:remove"
            data-live-name-param="{{ this.form.items.vars.full_name }}"
            data-live-index-param="0">
      Supprimer l’item #0
    </button>
  {{ form_end(this.form) }}
</div>
```

> `LiveCollectionTrait` fonctionne de pair avec `LiveCollectionType` pour ajouter/retirer des sous-formulaires sans JavaScript spécifique.


---

## `ValidatableComponentTrait`

**Objet** : active la **validation** sur les propriétés d’un composant, avec ou sans **Form Symfony**.

### Exemple : validation de propriétés sans `FormType`

```php
<?php
namespace App\Component;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\DefaultActionTrait;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\LiveComponent\Attribute\LiveAction;
use Symfony\UX\LiveComponent\ValidatableComponentTrait;
use Symfony\Component\Validator\Constraints as Assert;

#[AsLiveComponent('signup_step', template: 'components/signup_step.html.twig')]
class SignupStep
{
    use DefaultActionTrait;
    use ValidatableComponentTrait; // <-- expose validate() et gère _errors

    #[LiveProp(writable: true)]
    #[Assert\NotBlank(message: 'L’e-mail est requis.')]
    #[Assert\Email(message: 'Adresse e-mail invalide.')]
    public ?string $email = null;

    #[LiveProp(writable: true)]
    #[Assert\NotBlank(message: 'Le mot de passe est requis.')]
    #[Assert\Length(min: 8, minMessage: 'Au moins {{ limit }} caractères.')]
    public ?string $password = null;

    #[LiveAction]
    public function submit(): void
    {
        $this->validate(); // Alimente _errors en cas de violations
        if ($this->isValid()) {
            // ... traitement / persistance ...
        }
    }
}
```

```twig
{# templates/components/signup_step.html.twig #}
<div {{ attributes }}>
  <label>
    E-mail
    <input type="email" data-model="debounce(300)|email" />
  </label>
  {% if _errors.email is defined %}
    <div class="error">{{ _errors.email|first }}</div>
  {% endif %}

  <label>
    Mot de passe
    <input type="password" data-model="debounce(300)|password" />
  </label>
  {% if _errors.password is defined %}
    <div class="error">{{ _errors.password|first }}</div>
  {% endif %}

  <button {{ live_action('submit') }}>Créer le compte</button>
</div>
```

> Le trait permet de valider des `LiveProp` directement, puis d’afficher les erreurs via `_errors` dans le template.


---

## Tableau récapitulatif

| Besoin | Trait |
|---|---|
| Rendre un composant *live* (socle minimal) | `DefaultActionTrait` |
| Communiquer entre composants / émettre des événements navigateur | `ComponentToolsTrait` |
| Gérer un **Form Symfony** dans un composant | `ComponentWithFormTrait` |
| Ajouter/supprimer des éléments d’une **collection** | `LiveCollectionTrait` |
| **Valider** des propriétés (avec ou sans Form) | `ValidatableComponentTrait` |

---

## Notes pratiques
- Les traits sont **cumulables** selon les besoins du composant.  
- `ComponentToolsTrait` est optionnel si aucune émission d’événements n’est nécessaire.  
- Pour rediriger depuis une `LiveAction`, il est possible de retourner un `RedirectResponse` ou d’étendre `AbstractController` afin d’utiliser `redirectToRoute()` / `addFlash()` directement.  
- En cas de dépendances à des fonctionnalités plus récentes (ex. `mapPath`), vérifier la correspondance avec la version utilisée (ici v2.31).

---

> Référence : documentation officielle Symfony UX Live Components (v2.x).

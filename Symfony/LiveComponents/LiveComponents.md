# Symfony UX Live Components — Cheatsheet des *Traits* (v2.31) avec exemples

Ce document récapitule les principaux traits Live Components, propose des exemples de code et inclut une section détaillée sur la **mise à jour de l’URL lors d’un changement de LiveProp** (URL bindings).

---

## Sommaire

### 1. Traits & Exemples
- [1.1 DefaultActionTrait](#11-defaultactiontrait)
- [1.2 ComponentToolsTrait](#12-componenttoolstrait)
- [1.3 ComponentWithFormTrait](#13-componentwithformtrait)
- [1.4 LiveCollectionTrait](#14-livecollectiontrait)
- [1.5 ValidatableComponentTrait](#15-validatablecomponenttrait)

### 2. URL Bindings (mise à jour de l’URL avec les LiveProps)
- [2.1 Principe & base](#21-principe--base)
- [2.2 Types supportés](#22-types-supportés)
- [2.3 Multiples bindings (plusieurs LiveProps)](#23-multiples-bindings-plusieurs-liveprops)
- [2.4 Contrôler le nom du paramètre (`as`, `modifier`)](#24-contrôler-le-nom-du-paramètre-as-modifier)
- [2.5 `modifier` avec nom de propriété (≥ 2.26)](#25-modifier-avec-nom-de-propriété--226)
- [2.6 Mapper dans le *path* (`mapPath`, ≥ 2.28)](#26-mapper-dans-le-path-mappath--228)
- [2.7 Valider la valeur initiale (PostMount + validation)](#27-valider-la-valeur-initiale-postmount--validation)

### 3. Tableau récapitulatif & Notes
- [3.1 Tableau récapitulatif des traits](#31-tableau-récapitulatif-des-traits)
- [3.2 Notes pratiques](#32-notes-pratiques)

### 4. Serializer & Profondeur (`max_depth`)
- [4.1 Contexte](#41-contexte)
- [4.2 Configurer le Serializer](#42-configurer-le-serializer)
- [4.3 Définir un handler pour les cycles](#43-définir-un-handler-pour-les-cycles)
- [4.4 Limiter la profondeur de sérialisation](#44-limiter-la-profondeur-de-sérialisation)
- [4.5 Personnaliser la sérialisation d’une LiveProp](#45-personnaliser-la-sérialisation-dune-liveprop)
- [4.6 Bonnes pratiques](#46-bonnes-pratiques)

### 5. Cycle de vie d’un Live Component (v2.31)
- [5.1 Vue d’ensemble](#51-vue-densemble)
- [5.2 Schéma temporel simplifié](#52-schéma-temporel-simplifié)
- [5.3 Bonnes pratiques](#53-bonnes-pratiques)
- [5.4 Exemple complet](#54-exemple-complet)
- [5.5 Synthèse pratique](#55-synthèse-pratique)

### 6. Loading States (affichage / classes / attributs / ciblage)
- [6.1 Afficher ou masquer selon l’état de chargement](#61-afficher-ou-masquer-selon-létat-de-chargement)
- [6.2 Ajouter / retirer des classes](#62-ajouter--retirer-des-classes)
- [6.3 Ajouter / retirer un attribut HTML](#63-ajouter--retirer-un-attribut-html)
- [6.4 Combiner plusieurs directives](#64-combiner-plusieurs-directives)
- [6.5 Utiliser le `delay`](#65-utiliser-le-delay)
- [6.6 Cibler une action spécifique](#66-cibler-une-action-spécifique)
- [6.7 Cibler un modèle spécifique (`data-model`)](#67-cibler-un-modèle-spécifique-data-model)
- [6.8 Exemples pratiques (loader bouton / barre / points)](#68-exemples-pratiques-loader-bouton--barre--points)

---

## 1. Traits & Exemples

### 1.1 `DefaultActionTrait`

**Objet** : fournit l’« action par défaut » pour le rendu/re-rendu d’un composant côté client.  
**Usage** : à inclure systématiquement pour un composant `#[AsLiveComponent]`.

**Exemple minimal**

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

---

### 1.2 `ComponentToolsTrait`

**Objet** : helpers d’événements entre composants et d’événements navigateur.  
- `emit()`, `emitSelf()` : communication entre composants.  
- `dispatchBrowserEvent()` : déclencher un event JS sur l’élément racine.

**Exemple**

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
    use ComponentToolsTrait;

    #[LiveProp(writable: true)]
    public string $message = 'Prêt.';

    #[LiveAction]
    public function notify(): void
    {
        $this->emit('notification:created', ['message' => $this->message]);
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
document.addEventListener('toast', (e) => {
  console.log('[toast]', e.detail?.text);
});
</script>
```

---

### 1.3 `ComponentWithFormTrait`

**Objet** : intégration de **Form Symfony** dans un composant live (instanciation, soumission Ajax, reset).

**Exemple**

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
    use ComponentWithFormTrait;

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
        $this->submitForm();
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

---

### 1.4 `LiveCollectionTrait`

**Objet** : manipulation de **collections** de formulaires (`CollectionType`) en live, avec `LiveCollectionType`.

**Exemple**

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
    use LiveCollectionTrait;

    public function __construct(private FormFactoryInterface $formFactory) {}

    protected function instantiateForm(): FormInterface
    {
        return $this->formFactory->createBuilder()
            ->add('items', LiveCollectionType::class, [
                'entry_type' => TextType::class,
                'allow_add' => true,
                'allow_delete' => true,
                'prototype' => true,
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

---

### 1.5 `ValidatableComponentTrait`

**Objet** : validation sur les propriétés d’un composant, avec ou sans **Form Symfony**.

**Exemple**

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
    use ValidatableComponentTrait;

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
        $this->validate();
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

---

## 2. URL Bindings (mise à jour de l’URL avec les LiveProps)

### 2.1 Principe & base

Pour mettre à jour l’URL lorsqu’une `LiveProp` change, ajouter l’option `url` sur la propriété.

```php
<?php
namespace App\Twig\Components;

use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\LiveComponent\DefaultActionTrait;

#[AsLiveComponent]
class SearchModule
{
    use DefaultActionTrait;

    #[LiveProp(writable: true, url: true)]
    public string $query = '';
}
```

- Lorsqu’une saisie met à jour `query`, l’URL devient par exemple :  
  `https://my.domain/search?query=my+search+string`
- Charger cette URL initialise la valeur de la LiveProp depuis la querystring.  
- La mise à jour se fait via `history.replaceState()` (aucune nouvelle entrée d’historique).

### 2.2 Types supportés

Représentations côté URL pour différents types :

| Valeur JS | Représentation URL |
|---|---|
| `'some search string'` | `prop=some+search+string` |
| `42` | `prop=42` |
| `['foo', 'bar']` | `prop[0]=foo&prop[1]=bar` |
| `{ foo: 'bar', baz: 42 }` | `prop[foo]=bar&prop[baz]=42` |

> À l’arrivée (chargement de page), la valeur passe par l’**hydratation**. Si la valeur ne peut pas être hydratée, elle est **ignorée**.

### 2.3 Multiples bindings (plusieurs LiveProps)

Il est possible de lier **plusieurs** propriétés à l’URL. Pour que l’état soit **entièrement** représenté, **toutes** les props liées sont **écrites** comme paramètres, même si leurs valeurs n’ont pas changé.

```php
#[AsLiveComponent]
class SearchModule
{
    #[LiveProp(writable: true, url: true)]
    public string $query = '';

    #[LiveProp(writable: true, url: true)]
    public string $mode = 'fulltext';
}
```
Si seule `query` est modifiée, l’URL devient :  
`/search?query=my+query+string&mode=fulltext`

### 2.4 Contrôler le nom du paramètre (`as`, `modifier`)

> **Depuis 2.17** : option `as` pour définir un nom de paramètre différent du nom de la propriété.

```php
use Symfony\UX\LiveComponent\Metadata\UrlMapping;

#[AsLiveComponent]
class SearchModule
{
    #[LiveProp(writable: true, url: new UrlMapping(as: 'q'))]
    public string $query = '';
}
```
Résultat : `/search?q=my+query+string`

**Personnalisation côté composant via `modifier`** :

```php
use Symfony\UX\LiveComponent\Metadata\UrlMapping;

#[AsLiveComponent]
class SearchModule
{
    #[LiveProp(writable: true, url: true, modifier: 'modifyQueryProp')]
    public string $query = '';

    #[LiveProp]
    public ?string $alias = null;

    public function modifyQueryProp(LiveProp $liveProp): LiveProp
    {
        if ($this->alias) {
            $liveProp = $liveProp->withUrl(new UrlMapping(as: $this->alias));
        }
        return $liveProp;
    }
}
```
Dans le template, passer l’alias pour éviter les collisions :
```twig
<twig:SearchModule alias="q" />
```

Plusieurs instances sur la même page :
```twig
<twig:SearchModule alias="q1" />
<twig:SearchModule alias="q2" />
```

### 2.5 `modifier` avec nom de propriété (≥ 2.26)

> **Depuis 2.26** : la fonction `modifier` peut recevoir le **nom de la propriété** en second paramètre.

```php
abstract class AbstractSearchModule
{
    #[LiveProp(writable: true, url: true, modifier: 'modifyQueryProp')]
    public string $query = '';

    protected string $urlPrefix = '';

    public function modifyQueryProp(LiveProp $liveProp, string $propName): LiveProp
    {
        if ($this->urlPrefix) {
            return $liveProp->withUrl(new UrlMapping(as: $this->urlPrefix.'-'.$propName));
        }
        return $liveProp;
    }
}

#[AsLiveComponent]
class ImportantSearchModule extends AbstractSearchModule {}

#[AsLiveComponent]
class SecondarySearchModule extends AbstractSearchModule
{
    protected string $urlPrefix = 'secondary';
}
```
```twig
<twig:ImportantSearchModule />
<twig:SecondarySearchModule />
```
Résultat :  
`/search?query=my+important+query&secondary-query=my+secondary+query`

### 2.6 Mapper dans le *path* (`mapPath`, ≥ 2.28)

> **Depuis 2.28** : option `mapPath` pour mapper la valeur **dans le path** plutôt qu’en querystring.

```php
use Symfony\UX\LiveComponent\Metadata\UrlMapping;

#[AsLiveComponent]
class SearchModule
{
    #[LiveProp(writable: true, url: new UrlMapping(mapPath: true))]
    public string $query = '';
}
```

Route correspondante :
```php
#[Route('/search/{query}')]
public function __invoke(string $query): Response
{
    // ... rendu d’un template qui utilise SearchModule ...
}
```
Résultat : `/search/my+query+string`

- Si le nom du paramètre de route diffère de la propriété, utiliser `as`.  
- Si aucun paramètre de route n’existe, `mapPath` est **ignoré** et la valeur revient en querystring.

### 2.7 Valider la valeur initiale (PostMount + validation)

Comme toute `LiveProp` writable, une entrée utilisateur doit être **validée**. La valeur initiale issue de l’URL **n’est pas validée automatiquement**. Mettre en place un hook `PostMount` et la **validation** (éventuellement avec des **groupes**).

```php
use Symfony\Component\Validator\Constraints as Assert;
use Symfony\UX\LiveComponent\ValidatableComponentTrait;
use Symfony\UX\TwigComponent\Attribute\PostMount;

#[AsLiveComponent]
class SearchModule
{
    use ValidatableComponentTrait;

    #[LiveProp(writable: true, url: true)]
    public string $query = '';

    #[LiveProp(writable: true, url: true)]
    #[Assert\NotBlank]
    public string $mode = 'fulltext';

    #[PostMount]
    public function postMount(): void
    {
        // Valide "mode" sans jeter d’exception : le composant monte quand même
        if (!$this->validateField('mode', false)) {
            // gestion en cas d’échec (journalisation, valeur par défaut, etc.)
        }
    }
}
```

---

## 3. Tableau récapitulatif & Notes

### 3.1 Tableau récapitulatif des traits

| Besoin | Trait |
|---|---|
| Rendre un composant *live* (socle minimal) | `DefaultActionTrait` |
| Événements inter-composants / navigateur | `ComponentToolsTrait` |
| Gérer un **Form Symfony** dans un composant | `ComponentWithFormTrait` |
| Collections dynamiques dans un formulaire | `LiveCollectionTrait` |
| **Validation** (avec ou sans Form) | `ValidatableComponentTrait` |

### 3.2 Notes pratiques
- Les traits sont **cumulables** selon les besoins.  
- `replaceState()` est utilisé pour la mise à jour d’URL (pas d’entrée d’historique).  
- En présence de plusieurs URL bindings, tous les paramètres sont écrits pour refléter l’état complet.  
- Pour mapper des états complexes et garder une URL propre, envisager un **paramètre unique** et/ou `mapPath` avec un segment optionnel dans la route.  
- Vérifier la compatibilité des options dépendantes de la version (ex. `as` ≥ 2.17, `modifier` avec `propName` ≥ 2.26, `mapPath` ≥ 2.28).

---

## 4. Serializer & Profondeur (`max_depth`)

### 4.1 Contexte

Les composants Live utilisent le `Serializer` Symfony pour **hydrater** et **déshydrater** les `LiveProp`.  
Quand une propriété contient un objet ou une arborescence complexe (ex. entités Doctrine, objets imbriqués), la normalisation peut produire :

- des **récursions** (objets qui se référencent mutuellement),
- ou une **profondeur excessive** (`"Over 9 levels deep, aborting normalization"`).

Ces erreurs apparaissent généralement avec `useSerializerForHydration: true`.

---

### 4.2 Configurer le Serializer

```yaml
framework:
    serializer:
        enabled: true
        enable_attributes: true
        default_context:
            enable_max_depth: true
            circular_reference_handler: '@App\Serializer\CircularReferenceHandler'
```

- `enable_attributes` : active les attributs PHP 8 (`#[Groups]`, `#[MaxDepth]`, etc.)
- `enable_max_depth` : active la prise en compte de `#[MaxDepth]`
- `circular_reference_handler` : gère les cycles d’objets

---

### 4.3 Définir un handler pour les cycles

```php
<?php
namespace App\Serializer;

class CircularReferenceHandler
{
    public function __invoke(object $object): string
    {
        return sprintf('CircularRef(%s)', get_class($object));
    }
}
```

Ce handler empêche les boucles infinies en remplaçant les objets cycliques par une valeur descriptive.

---

### 4.4 Limiter la profondeur de sérialisation

Utilise l’attribut `#[MaxDepth]` sur les propriétés susceptibles d’être trop profondes :

```php
use Symfony\Component\Serializer\Annotation\MaxDepth;

class ProductMainDatas
{
    #[MaxDepth(2)]
    public ?Availability $availability = null;
}
```

➡️ Nécessite `enable_max_depth: true` dans la config.

---

### 4.5 Personnaliser la sérialisation d’une LiveProp

Pour les objets complexes :

```php
#[LiveProp(hydrateWith: 'hydrateOrder', dehydrateWith: 'dehydrateOrder')]
public ?OrderMainDatas $simulatedOrder = null;

public function dehydrateOrder(?OrderMainDatas $order): ?array
{
    return $order
        ? $this->serializer->normalize($order, 'json', ['max_depth' => 3])
        : null;
}

public function hydrateOrder(?array $data): ?OrderMainDatas
{
    return $data
        ? $this->serializer->denormalize($data, OrderMainDatas::class, 'json')
        : null;
}
```

Cela donne un contrôle total sur la profondeur et la structure.

---

### 4.6 Bonnes pratiques

| Cas | Recommandation |
|-----|----------------|
| Objets simples / DTO | `useSerializerForHydration: true` |
| Entités Doctrine ou relations multiples | Méthodes `hydrateWith` / `dehydrateWith` |
| Graphes récursifs | `circular_reference_handler` |
| Arbres profonds | `#[MaxDepth]` + `enable_max_depth: true` |
| Débogage | Loguer `$serializer->normalize($data, 'json')` avant rendu |

---

> 💡 **Astuce Monolog**
> 
> Pour tracer les problèmes de sérialisation :
> 
> ```yaml
> monolog:
>   channels: [serializer]
>   handlers:
>     serializer:
>       type: stream
>       path: '%kernel.logs_dir%/serializer.log'
>       level: debug
>       channels: [serializer]
> ```
> 
> Puis injecte `LoggerInterface $serializerLogger` dans tes composants pour observer la structure normalisée.

## 5. Cycle de vie d’un Live Component (v2.31)

### 5.1 Vue d’ensemble

Un composant Live Symfony suit un **cycle de vie précis** depuis sa création jusqu’à chaque re-rendu.  
Bien comprendre ce cycle permet de choisir le bon **hook d’exécution** (`#[PostMount]`, `#[PostHydrate]`, etc.) selon le moment où les propriétés (`LiveProp`) sont disponibles.

| Étape | Description | LiveProps hydratées ? | Hook / Exemple |
|--------|--------------|-----------------------|----------------|
| **`__construct()`** | Appelée lors de la création du composant (injection de dépendances). | ❌ Non | `__construct(ProductRepository $repo)` |
| **`mount()`** | Exécutée après `__construct()`, mais **avant** hydratation des LiveProps. | ❌ Non | Sert uniquement pour initialiser des valeurs de base. |
| **`#[PostMount]`** | Appelée **après hydratation complète** des LiveProps, juste avant le premier rendu. | ✅ Oui | `#[PostMount] public function initAfterMount()` |
| **`#[PostHydrate]`** | Appelée après chaque requête Live (interaction AJAX). | ✅ Oui | `#[PostHydrate] public function afterHydrate()` |
| **Rendu initial** | Le template Twig est rendu avec l’état hydraté. | ✅ Oui | `<div {{ attributes }}>...</div>` |

---

### 5.2 Schéma temporel simplifié

```
┌────────────────────┐
│ __construct()      │  → services & dépendances
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ mount()            │  → initialisation simple
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ Hydratation des    │
│ LiveProps (depuis  │
│ Twig, URL, etc.)   │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ #[PostMount] hook  │  → logique dépendant des props
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ Rendu initial Twig │
└────────┬───────────┘
         │
         ▼
┌────────────────────┐
│ Interaction Live   │
│ (actions, models)  │
│ → #[PostHydrate]   │
└────────────────────┘
```

---

### 5.3 Bonnes pratiques

- ⚠️ **Ne jamais accéder à une LiveProp dans `mount()`** → elles ne sont pas encore hydratées.  
  👉 Utilise plutôt `#[PostMount]` pour tout chargement dépendant d’un paramètre (ex. `$slug`, `$filters`).

- ✅ **`#[PostMount]`** : initialiser les données qui dépendent des LiveProps (slug, id, etc.).  
- ✅ **`#[PostHydrate]`** : exécuter une logique après chaque action Live (AJAX).  
- ✅ **`priority`** : tous les hooks peuvent recevoir une priorité (`#[PostHydrate(priority: 10)]`).

---

### 5.4 Exemple complet

```php
<?php
namespace App\Twig\Components;

use App\Repository\ProductRepository;
use Symfony\UX\LiveComponent\Attribute\AsLiveComponent;
use Symfony\UX\LiveComponent\Attribute\LiveProp;
use Symfony\UX\TwigComponent\Attribute\PostMount;
use Symfony\UX\LiveComponent\Attribute\PostHydrate;
use Symfony\UX\LiveComponent\DefaultActionTrait;

#[AsLiveComponent('product_filter')]
class ProductFilter
{
    use DefaultActionTrait;

    public function __construct(private ProductRepository $repo) {}

    #[LiveProp(writable: true, url: true)]
    public string $slug = '';

    public array $filters = [];

    #[PostMount]
    public function initFilters(): void
    {
        // ✅ $slug est hydraté ici
        $this->filters = $this->repo->findFiltersByCategorySlug($this->slug);
    }

    #[PostHydrate]
    public function afterLiveRequest(): void
    {
        // ✅ exécuté après chaque action Live
        $this->filters = $this->repo->findFiltersByCategorySlug($this->slug);
    }
}
```

### 5.5 Synthèse pratique

| Situation | Hook recommandé |
|------------|----------------|
| Initialisation via services (constructeur) | `__construct()` |
| Définition de valeurs par défaut simples | `mount()` |
| Chargement de données dépendantes d’un LiveProp (`slug`, `id`, `query`, etc.) | `#[PostMount]` |
| Exécution de logique après chaque interaction Live | `#[PostHydrate]` |
| Nettoyage avant envoi au client | `#[PreDehydrate]` |
| Ajustement juste avant re-rendu | `#[PreReRender]` |

---

## 6. Loading States (v2.31)

Les *loading states* permettent d'afficher un élément, le masquer, ou modifier son apparence **pendant qu’un composant est en cours de re-render ou qu’une LiveAction est en train d’être exécutée**.

Ces comportements s’obtiennent via l’attribut :

```twig
data-loading="..."
```

---

### 6.1 Afficher ou masquer selon l’état de chargement

Afficher pendant le chargement :

```twig
<span data-loading>Loading...</span>

<!-- équivalent -->
<span data-loading="show">Loading...</span>
```

Masquer pendant le chargement :

```twig
<span data-loading="hide">Saved!</span>
```

---

### 6.2 Ajouter / retirer des classes

Ajouter une classe :

```twig
<div data-loading="addClass(opacity-50)">...</div>
```

Retirer une classe :

```twig
<div data-loading="removeClass(opacity-50)">...</div>
```

Ajouter plusieurs classes :

```twig
<div data-loading="addClass(opacity-50 text-muted)">...</div>
```

---

### 6.3 Ajouter / retirer un attribut HTML

Ajouter un attribut HTML vide (`disabled`, `readonly`, etc.) :

```twig
<button data-loading="addAttribute(disabled)">
    Save
</button>
```

⚠️ Limite :  
Ne fonctionne **que** pour les attributs *vides*, pas pour :

```twig
<!-- ❌ ne fonctionne pas -->
<div data-loading="addAttribute(style='color:red')">
```

---

### 6.4 Combiner plusieurs directives

```twig
<div data-loading="addClass(opacity-50) addAttribute(disabled)">
    ...
</div>
```

---

### 6.5 Utiliser le `delay`

Le loader n’apparaît qu’après un délai (évite les clignotements) :

```twig
<!-- délai 200ms -->
<div data-loading="delay|show">Loading...</div>

<!-- délai personnalisé -->
<div data-loading="delay(500)|addClass(opacity-50)">...</div>
```

---

### 6.6 Cibler une action spécifique

```twig
<span data-loading="action(saveForm)|show">
    Saving...
</span>

<div data-loading="action(saveForm)|delay|addClass(opacity-50)">
    ...
</div>
```

---

### 6.7 Cibler un modèle spécifique (`data-model`)

```twig
<input data-model="email" />

<span data-loading="model(email)|show">
    Checking availability...
</span>
```

Fonctionne aussi avec des propriétés imbriquées :

```twig
<span data-loading="model(user.email)|delay|addClass(opacity-50)">
```

---

### 6.8 Exemples pratiques (loader bouton / barre / points)

#### Bouton + loader

```twig
<div class="actions">
    <button class="btn" {{ live_action('reset') }}>Reset</button>

    <div class="loading-zone" data-loading="action(reset)|addClass(loading-active)">
        <div class="spinner"></div>
    </div>
</div>
```

#### Ligne de chargement animée

```twig
<div class="loading-line" data-loading="show"></div>
```

#### Points animés

```twig
<span class="dots" data-loading="show">
    <span></span><span></span><span></span>
</span>
```


> Référence : documentation officielle Symfony UX Live Components (v2.x, notamment 2.17/2.26/2.28/2.31).

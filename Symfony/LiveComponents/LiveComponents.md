# Symfony UX Live Components — Cheatsheet des *Traits* (v2.31)

Ce document récapitule les principaux traits fournis par Symfony UX Live Components et leur usage recommandé. Style neutre, prêt à intégrer dans une documentation technique.

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
**Objet** : fournit l’« action par défaut » permettant le rendu et le re‑rendu d’un composant côté client.  
**Usage** : à inclure systématiquement dans un composant annoté `#[AsLiveComponent]`, même sans actions personnalisées.  
**Bénéfice** : rend le composant *live* avec un minimum de code.

### Points clés
- Nécessaire pour activer le cycle de rendu live.
- Compatible avec d’autres traits (voir ci‑dessous).

---

## `ComponentToolsTrait`
**Objet** : expose des utilitaires de communication et d’intégration front.  
**Fonctions clés** :
- `emit('event', payload)` / `emitSelf('event', payload)` : communication entre composants (émission d’événements).  
- `dispatchBrowserEvent('event', detail)` : déclenchement d’un événement JavaScript sur le nœud DOM racine du composant (fermeture de modale, focus, notifications, etc.).

**Quand l’utiliser** : dès qu’une interaction inter‑composants est nécessaire ou qu’un comportement navigateur doit être piloté sans JavaScript personnalisé.

### Points clés
- N’est pas requis si aucun événement n’est émis ni dispatché.
- Fonctionne indépendamment de l’usage d’un `FormType`.

---

## `ComponentWithFormTrait`
**Objet** : facilite l’intégration d’un **Form Symfony** dans un composant live (instanciation, soumission Ajax, réinitialisation).

**Méthodes typiques** :
- `instantiateForm(): FormInterface` (à implémenter dans le composant).  
- `getForm(): FormInterface`  
- `submitForm(?FormInterface $form = null): void`  
- `resetForm(?FormInterface $form = null): void`

**Quand l’utiliser** : gestion de la soumission/validation d’un formulaire sans rechargement de page, directement dans le cycle de vie du composant.

### Points clés
- S’intègre naturellement avec la validation serveur.
- Réduit la quantité de JavaScript dédié pour les formulaires.

---

## `LiveCollectionTrait`
**Objet** : ajoute des aides pour manipuler des **collections** de formulaires (p. ex. `CollectionType`) en live, en particulier conjointement avec `LiveCollectionType`.

**Cas d’usage** : ajout/suppression dynamique d’éléments d’une collection (lignes, items, variantes) sans JavaScript personnalisé.

### Points clés
- Simplifie les formulaires dynamiques complexes.
- S’emploie dans les composants qui gèrent des listes éditables.

---

## `ValidatableComponentTrait`
**Objet** : active la **validation** sur les propriétés d’un composant, avec ou sans **Form Symfony**.

**Pattern courant** :
1. Ajouter des contraintes sur les `#[LiveProp]` (et `#[Assert\Valid]` pour les objets imbriqués).  
2. Appeler `validate()` dans une `LiveAction` pour déclencher la validation.  
3. Rendre les erreurs côté template via la variable spéciale `_errors`.

**Quand l’utiliser** : validation d’entrées utilisateur en direct (live) sans recourir systématiquement à un `FormType` complet.

### Points clés
- S’intègre avec le composant Validator de Symfony.
- Permet des retours d’erreur granulaires côté template.

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
- En cas de dépendances à des fonctionnalités plus récentes (ex. `mapPath`), vérifier la correspondance avec la version de Symfony UX Live Components employée (ici v2.31).

---

> Référence : documentation officielle Symfony UX Live Components (v2.x).


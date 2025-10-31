# 🌳 Comprendre les algorithmes d'arbre (Tree Algorithms)

Les **arbres** sont des **structures de données hiérarchiques** où
chaque élément (appelé **nœud**) peut avoir **zéro ou plusieurs
enfants**.\
Ils permettent de représenter des relations parent-enfant, d'organiser
l'information efficacement, et de faciliter la recherche, l'insertion ou
la suppression d'éléments.

------------------------------------------------------------------------

## 🧩 Structure d'un arbre

Un **arbre** est composé de : - **Racine (Root)** : le nœud de départ. -
**Nœuds (Nodes)** : les éléments de l'arbre. - **Arêtes (Edges)** : les
liens entre les nœuds. - **Feuilles (Leaves)** : les nœuds sans
enfants. - **Hauteur (Height)** : la distance maximale entre la racine
et une feuille.

### Exemple visuel :

            A
           /       B   C
         / \       D   E   F

Ici : - `A` est la racine.\
- `B` et `C` sont les enfants de `A`.\
- `D`, `E` et `F` sont des **feuilles**.

------------------------------------------------------------------------

## 🌿 Types d'arbres

### 1. **Arbre général**

Chaque nœud peut avoir un nombre **illimité d'enfants**.\
Utilisé pour représenter des structures hiérarchiques comme des
dossiers, des menus, etc.

### 2. **Arbre binaire**

Chaque nœud a **au maximum deux enfants** :\
- un **fils gauche (left)**\
- un **fils droit (right)**

C'est le type le plus étudié car il sert de base à de nombreux
algorithmes.

### 3. **Arbre binaire de recherche (Binary Search Tree -- BST)**

Les valeurs y sont organisées selon une règle : - Tout élément du
**sous-arbre gauche** est **inférieur** à la valeur du nœud.\
- Tout élément du **sous-arbre droit** est **supérieur**.

Cela permet une **recherche très rapide (O(log n))**.

### 4. **Arbre équilibré (Balanced Tree)**

C'est un arbre où la **hauteur des sous-arbres** reste équilibrée pour
éviter les recherches inefficaces.\
Exemples : **AVL**, **Red-Black Tree**, **B-Tree** (utilisé dans les
bases de données).

### 5. **Arbre de décision (Decision Tree)**

Utilisé en **intelligence artificielle et machine learning**.\
Chaque nœud représente une **question**, et chaque branche une **réponse
possible** menant à une décision.

------------------------------------------------------------------------

## ⚙️ Algorithmes fondamentaux sur les arbres

### 1. **Parcours (Traversal)**

Le **parcours** consiste à visiter tous les nœuds d'un arbre selon un
certain ordre.

#### 🔹 Parcours en profondeur (DFS -- Depth First Search)

On explore **le plus profondément possible** avant de revenir en
arrière.

Trois variantes : - **Préfixe (Preorder)** : `Nœud → Gauche → Droit` -
**Infixe (Inorder)** : `Gauche → Nœud → Droit` (très utile pour les
arbres de recherche) - **Suffixe (Postorder)** : `Gauche → Droit → Nœud`

#### Exemple en PHP :

``` php
function inorderTraversal(?Node $node): void {
    if ($node === null) {
        return;
    }
    inorderTraversal($node->left);
    echo $node->value . " ";
    inorderTraversal($node->right);
}
```

------------------------------------------------------------------------

#### 🔹 Parcours en largeur (BFS -- Breadth First Search)

On visite les nœuds **niveau par niveau**, en commençant par la racine.

#### Exemple :

``` php
function breadthFirstTraversal(?Node $root): void {
    if ($root === null) return;

    $queue = [$root];
    while ($queue) {
        $node = array_shift($queue);
        echo $node->value . " ";
        if ($node->left) $queue[] = $node->left;
        if ($node->right) $queue[] = $node->right;
    }
}
```

------------------------------------------------------------------------

### 2. **Insertion dans un arbre binaire de recherche**

L'insertion respecte la propriété du BST : - Si la valeur est plus
petite, on va à gauche. - Si elle est plus grande, on va à droite.

``` php
function insertNode(?Node $root, int $value): Node {
    if ($root === null) {
        return new Node($value);
    }
    if ($value < $root->value) {
        $root->left = insertNode($root->left, $value);
    } else {
        $root->right = insertNode($root->right, $value);
    }
    return $root;
}
```

------------------------------------------------------------------------

### 3. **Recherche dans un arbre binaire de recherche**

On suit la logique de l'insertion :

``` php
function searchNode(?Node $root, int $value): ?Node {
    if ($root === null || $root->value === $value) {
        return $root;
    }
    if ($value < $root->value) {
        return searchNode($root->left, $value);
    }
    return searchNode($root->right, $value);
}
```

------------------------------------------------------------------------

### 4. **Suppression d'un nœud**

Cas à gérer : 1. Le nœud est une feuille → on le supprime directement.\
2. Le nœud a un seul enfant → on le remplace par cet enfant.\
3. Le nœud a deux enfants → on le remplace par le **successeur** (le
plus petit du sous-arbre droit).

------------------------------------------------------------------------

## 🌲 Arbres avancés

### 🧱 AVL Tree

Un **arbre binaire auto-équilibré**.\
Après chaque insertion/suppression, il **rééquilibre** la hauteur pour
garder des performances optimales.

### ⚖️ Red-Black Tree

Un autre type d'arbre équilibré, plus souple que l'AVL.\
Chaque nœud est coloré **rouge** ou **noir**, et ces couleurs
garantissent un équilibre global.

### 📚 B-Tree & B+Tree

Utilisés dans les **bases de données** et **systèmes de fichiers**.\
Chaque nœud peut contenir plusieurs clés et enfants.\
Très efficaces pour les accès disque.

------------------------------------------------------------------------

## 🧮 Complexités typiques

  Opération          Arbre équilibré   Arbre non équilibré
  ------------------ ----------------- ---------------------
  Recherche          O(log n)          O(n)
  Insertion          O(log n)          O(n)
  Suppression        O(log n)          O(n)
  Parcours complet   O(n)              O(n)

------------------------------------------------------------------------

## 💡 Utilisations courantes

-   Arbres DOM dans le web\
-   Systèmes de fichiers (dossiers, sous-dossiers)\
-   Arbres syntaxiques (compilateurs, parseurs)\
-   Arbres de décision (IA, ML)\
-   Structures d'index (B-Trees dans MySQL, MongoDB...)

------------------------------------------------------------------------

## 🚀 Résumé

-   🌱 Les **arbres** structurent les données hiérarchiquement.\
-   🔍 Les **algorithmes d'arbre** (DFS, BFS) permettent de les explorer
    efficacement.\
-   ⚖️ Les **arbres équilibrés** assurent des performances optimales.\
-   💾 Les **arbres multi-branches** (B-Trees) sont essentiels pour les
    bases de données.

# ⚙️ Symfony
## 🎯 Objectif
Rappels rapides sur les commandes et concepts clés.

## 💡 Commandes utiles
```bash
symfony console make:entity
symfony console make:controller
symfony console doctrine:migrations:migrate
symfony server:start
```

## 🧱 Exemple Controller
```php
#[Route('/products', name: 'product_index')]
public function index(ProductRepository $repo): Response
{
    $products = $repo->findAll();
    return $this->render('product/index.html.twig', ['products' => $products]);
}
```

## 🧠 Bonnes pratiques
- ✅ Toujours typer les services et arguments
- ✅ Préférer l’injection par constructeur
- ✅ Utiliser les DTO / Value Objects pour le domaine

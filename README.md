# Documentation checklogs.dev

Système de documentation moderne pour checklogs.dev avec support Markdown et design cohérent.

## 🚀 Structure

```
/
├── index.html              # Page d'accueil de la documentation
├── read.html               # Lecteur d'articles
├── vercel.json             # Configuration Vercel
├── README.md               # Ce fichier
└── pages/
    ├── api.md              # Documentation API REST
    ├── node-sdk.md         # Documentation SDK Node.js
    ├── quick-start.md      # Guide de démarrage rapide
    └── test.md             # Article de test
```

## 🔧 Comment ça fonctionne

### URLs automatiques
- `/` → Page d'accueil avec liste des articles
- `/read/api` → Documentation API
- `/read/node-sdk` → SDK Node.js
- `/read/quick-start` → Guide rapide

### Auto-découverte
Le système découvre automatiquement les articles dans `/pages/` et génère :
- Métadonnées (titre, description, catégorie)
- Table des matières
- Navigation entre articles
- Temps de lecture estimé

### Fallback intégré
Si les fichiers `.md` ne sont pas accessibles, le système utilise du contenu de fallback intégré dans le JavaScript.

## 📝 Ajouter un article

1. **Créer** `/pages/mon-article.md`
2. **Écrire** en Markdown standard :
   ```markdown
   # Titre de l'article
   
   Description de l'article ici.
   
   ## Section 1
   
   Contenu...
   ```
3. **Ajouter** le slug dans la liste des articles connus dans `index.html` et `read.html`
4. **Déployer** sur Vercel

## 🎨 Design

Le design utilise exactement les mêmes styles que checklogs.dev :
- Variables CSS cohérentes
- Logo avec triangles animés
- Couleurs et dégradés identiques
- Animations fluides

## 🐛 Debug

Pour débugger un problème :
1. Ouvrir les DevTools (F12)
2. Aller sur `/read/nom-article`
3. Vérifier la console pour les logs de debug
4. Vérifier que les fichiers `.md` sont accessibles

## 🚀 Déploiement

### Sur Vercel
1. Connecter le repo GitHub
2. Vercel détecte automatiquement la configuration
3. Les rewrites dans `vercel.json` gèrent les URLs propres

### Configuration requise
- `vercel.json` pour les rewrites
- Headers CORS pour les fichiers `.md`
- Fallback JavaScript pour la robustesse

## 🔍 URLs de test

Après déploiement, tester :
- `/` - Page d'accueil
- `/read/api` - Documentation API  
- `/read/node-sdk` - SDK Node.js
- `/read/quick-start` - Guide rapide
- `/read/test` - Article de test

## 📋 Troubleshooting

### "Article non trouvé"
- Vérifier que le fichier `.md` existe dans `/pages/`
- Vérifier l'URL (sensible à la casse)
- Le fallback devrait fonctionner même sans fichier

### Styles cassés
- Vérifier que `global.css` et les CDN sont accessibles
- Problème probablement de cache du navigateur

### Navigation ne fonctionne pas
- Vérifier `vercel.json` déployé
- Tester les URLs directement

## 💡 Fonctionnalités

✅ **Auto-découverte** des articles  
✅ **Fallback** intégré  
✅ **Design cohérent** avec checklogs.dev  
✅ **URLs propres** avec Vercel  
✅ **Responsive** mobile/desktop  
✅ **Table des matières** automatique  
✅ **Navigation** entre articles  
✅ **Syntax highlighting** pour le code  
✅ **Estimation** temps de lecture
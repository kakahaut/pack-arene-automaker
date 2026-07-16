# Pack Arène AutoMaker

Site statique, **100 % côté navigateur**, aucun serveur ni backend. Tu déposes une archive
de pack de textures (`.zip`, `.rar`, `.7z`, `.tar`, `.tar.gz`, `.tgz`...), le site :

1. **Repère le dossier `assets/`**, qu'il soit à la racine de l'archive ou dans un premier
   sous-dossier (le sous-dossier "wrapper" est ignoré dans le résultat).
2. **Duplique** les 9 textures concernées sous leur nouveau nom (le fichier d'origine reste
   en place, rien n'est renommé ni déplacé). Si un fichier attendu manque dans le pack, il est
   remplacé par la version de base stockée dans `fallback/`.
3. **Ajoute un crédit** dans `pack.mcmeta` (`description`), en conservant ce qu'il y avait avant.
4. **Nettoie le nom du fichier de sortie** : pas de `!` ni d'espace en début de nom, même si
   l'archive d'origine en avait.
5. Renvoie un `.zip` où `assets/`, `pack.mcmeta` etc. sont **directement à la racine** —
   à l'extraction, pas de sous-dossier parasite.

## ⚙️ Ça tourne où ?

**GitHub Pages suffit très bien.** Tout le traitement (y compris décompresser du `.rar`/`.7z`/
`.tar.gz`) se fait dans le navigateur de la personne qui utilise le site, via WebAssembly.
Il n'y a besoin ni de backend, ni de base de données, ni de fonction serverless — juste
d'un hébergement de fichiers statiques. Ça marche donc aussi bien sur :

- **GitHub Pages** (gratuit, recommandé, c'est ce que ce projet est fait pour)
- N'importe quel "mini hébergeur" statique : Netlify, Vercel, Cloudflare Pages, Surge, ou même
  un simple hébergement mutualisé où tu déposes les fichiers par FTP.

La seule contrainte : sers le dossier **tel quel**, avec sa structure de fichiers intacte
(voir plus bas), parce que le site va chercher `vendor/libarchive/worker-bundle.js` et
`vendor/libarchive/libarchive.wasm` à des chemins relatifs précis.

Pour tester en local avant de déployer, ouvrir `index.html` directement avec un double-clic
**ne fonctionnera pas** (les navigateurs bloquent les requêtes `fetch` en `file://`). Lance un
petit serveur local à la racine du projet, par exemple :

```
python3 -m http.server 8000
```

puis ouvre `http://localhost:8000`.

## 📁 Structure du projet

```
.
├── index.html                     ← le site (une seule page)
├── .nojekyll                      ← empêche GitHub Pages de traiter le site avec Jekyll
├── README.md
├── fallback/                      ← tes 9 textures de base (à remplir toi-même)
│   ├── LISEZMOI.txt
│   ├── demonic_sword.png
│   ├── eco_boots.png
│   ├── eco_helmet.png
│   ├── eco_layer_1.png
│   ├── eco_layer_2.png
│   ├── desh_2.png
│   ├── desh_3.png
│   ├── deshChestplate.png
│   └── deshLeggings.png
└── vendor/
    └── libarchive/                ← libarchive.js (MIT), pour lire .rar/.7z/.tar.gz/...
        ├── libarchive.js
        ├── worker-bundle.js
        ├── libarchive.wasm
        └── LICENSE.txt
```

`vendor/libarchive/` est la bibliothèque [libarchive.js](https://github.com/nika-begiashvili/libarchivejs)
(licence MIT, incluse) compilée en WebAssembly. Elle gère la lecture de `.rar`, `.7z`, `.tar`,
`.tar.gz` etc. Les `.zip` sont lus directement avec JSZip (plus rapide, pas de WASM à charger).

## 🚀 Déployer sur GitHub Pages

1. Crée un repo GitHub public, par ex. `pack-arene-automaker`.
2. Mets **tout le contenu de ce dossier** (avec la structure ci-dessus) à la racine du repo —
   n'oublie pas `fallback/` rempli avec tes 9 vrais fichiers, ni le dossier `vendor/`.
3. Repo GitHub → **Settings → Pages** → Source : **Deploy from a branch** →
   Branch : `main`, dossier : `/ (root)` → **Save**.
4. Après ~1 minute, le site est en ligne à :
   `https://<ton-pseudo>.github.io/<nom-du-repo>/`

## 🛠️ Personnaliser

Tout se modifie dans `index.html`, dans la balise `<script type="module">` :

- **Les 9 règles de duplication** : tableau `RULES` en haut du script (chemin source, chemin
  destination, nom du fichier de repli).
- **Le texte de crédit** dans `pack.mcmeta` : constantes `CREDIT_TEXT` et `CREDIT_COLOR`
  juste après `RULES`. `CREDIT_COLOR` ne sert que si `description` était déjà un composant de
  texte riche (objet/array) — sinon c'est juste ajouté en texte brut à la ligne du dessous.
- **Formats acceptés** : l'attribut `accept` du `<input type="file">` (c'est juste indicatif,
  la logique de lecture s'adapte de toute façon au contenu réel du fichier).

## ⚠️ Limites

- Fonctionne pour des packs de taille raisonnable : tout est traité en mémoire dans l'onglet.
  Un très gros pack (plusieurs centaines de Mo) peut ramer selon l'appareil.
- Le premier traitement d'un fichier `.rar`/`.7z`/`.tar.gz` télécharge le module WebAssembly
  de libarchive.js (~1 Mo) — un peu plus lent que pour un `.zip`, une seule fois par session.
- Les fichiers de `fallback/` sont publics dans le repo (visibles par quiconque connaît l'URL).
- Le crédit est ajouté avec des règles simples (chaîne, tableau ou objet de composant de texte) ;
  un `pack.mcmeta` avec un JSON invalide ne sera pas modifié (le reste du traitement continue).

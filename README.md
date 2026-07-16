# Pack Arène AutoMaker

Site statique, **100 % côté navigateur**, aucun serveur ni backend. Tu déposes une archive
de pack de textures (`.zip`, `.rar`, `.7z`, `.tar`, `.tar.gz`, `.tgz`...), le site :

1. **Repère le dossier `assets/`**,
2. **Duplique** les 9 textures concernées sous leur nouveau nom (le fichier d'origine reste
   en place, rien n'est renommé ni déplacé). Si une texture attendue manque dans le pack, celle par défaut est utilisée.
3. Renvoie un `.zip` modifié, a juste mettre dans le dossier des packs.

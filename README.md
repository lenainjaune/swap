# swap
Tout ce qui concerne le swap

# Quantité allouée
https://opensource.com/article/18/9/swap-space-linux-systems

## Modifier partition de swap
Cas d'étude : sur un notebook (Samsung NP-NC10) j'avais 1GB de RAM et alloué 2GB de swap à l'installation sur une partition dédiée (le système Emmabuntüs était installé sur l'autre partition). Les 2 partitions sont contigûes sur un même disque : la 1ère est la partition système, la 2ème est la partition swap. 

Pour améliorer ses performances, j'ai upgradé la RAM à 2GB.
=> la partition de swap est désormais trop petite MAIS il n'y a plus d'espace non alloué sur le disque, il faut que je récupère de l'espace ailleurs. 
La partition système est faiblement occupée (~10GB/141GB), mais quand le système est démarré, je ne peux pas la modifier
=> il faut faire l'opération hors ligne (par exemple depuis un support Live CD ou USB)

Procédure
1. Booter sur un support Live (testé : Emmabuntüs)
1. Ouvrir **GParted**
1. S'assurer qu'on peut bien récupérer de l'espace sur la partition système, sinon ce n'est pas la peine de poursuivre
1. Redimmensionner la partition système en lui demandant de libérer 2048MB (= 2GB) à la fin
1. Désactiver puis supprimer la partition swap => désormais l'espace libéré de la partition système et celui anciennement occupé par le swap sont fusionnés)
1. Créer une nouvelle **partition primaire** depuis l'espace libre et lui donner comme système de fichier **linux-swap**
Nota : il n'est pas nécessaire de réactiver le swap dans la session Live avec **swapon -a**

Remarque : au redémarrage, j'ai eu un message indiquant que j'avais upgradé ma RAM et si je voulais appliquer les changements, ce que j'ai accepté puis j'ai redémarré pour vérifier si le démarrage était plus rapide, un message Kernel Panic s'est affiché, j'ai redémarré => plus rien depuis...

## swap sur une partition ou un fichier ? Nécessité ? Performance ?
intéressant : https://serverfault.com/questions/25653/swap-partition-vs-file-for-performance

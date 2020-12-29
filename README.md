# swap
Tout ce qui concerne le swap

# Quantité allouée
https://opensource.com/article/18/9/swap-space-linux-systems

## Modifier partition de swap (Live GUI)
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
1. Créer une nouvelle **partition étendue** depuis l'espace libre, ajouter une nouvelle **partition logique** et lui donner comme système de fichier **linux-swap**
Nota : il n'est pas nécessaire de réactiver le swap dans la session Live avec **swapon -a**

Remarque : au redémarrage, j'ai eu un message indiquant que j'avais upgradé ma RAM et si je voulais appliquer les changements, ce que j'ai accepté puis j'ai redémarré pour vérifier si le démarrage était plus rapide, un message Kernel Panic s'est affiché, j'ai redémarré => plus rien depuis...

## swap sur une partition ou un fichier ? Nécessité ? Performance ?
intéressant : https://serverfault.com/questions/25653/swap-partition-vs-file-for-performance

## ATTENTION Modifier partition de swap (Live CLI) ATTENTION N'A PAS FONCTIONNE (partition système non accessible) => voir dessus la subtilité d'ajouter une partition étendue et une partition logique dedans
On redimensionne la partition 1 (contient l'espace non utilisé) et on supprime la partition de swap

Nota : on supprime aussi dans le cas présent la partition étendue qui contient uniquement la partition swap
```sh
root@host:~# parted
(parted) print                                                            
...
Number  Start   End    Size    Type      File system     Flags
 1      1049kB  159GB  159GB   primary   ext4            boot
 2      159GB   160GB  1062MB  extended
 5      159GB   160GB  1062MB  logical   linux-swap(v1)
(parted) resizepart                                                       
Partition number? 1                                                       
End?  [159GB]? 156GB
Warning: Shrinking a partition can cause data loss, are you sure you want to continue?
Yes/No? Yes
(parted) print                                                            
...
Number  Start   End    Size    Type      File system     Flags
 1      1049kB  156GB  156GB   primary   ext4            boot
 2      159GB   160GB  1062MB  extended
 5      159GB   160GB  1062MB  logical   linux-swap(v1)
(parted) rm 5
(parted) rm 2
(parted) print free
...
Number  Start   End     Size    Type     File system  Flags
        32.3kB  1049kB  1016kB           Free Space
 1      1049kB  156GB   156GB   primary  ext4         boot
        156GB   160GB   4042MB           Free Space
(parted) unit s
(parted) print free
...
Number  Start       End         Size        Type     File system  Flags
        63s         2047s       1985s                Free Space
 1      2048s       304687500s  304685453s  primary  ext4         boot
        304687501s  312581807s  7894307s             Free Space

(parted) quit
```
Calculs des frontières alignées de l'espace SWAP (sinon les performances ne seraient pas optimales selon l'avertissement qui s'affiche) :

On voit après l'espace système que l'espace libre démarre à 304687501s et finit à 312581807s et il faut que cette espace aligné soit entre ces bornes. 
Selon [ce lien](https://wiki.archlinux.org/index.php/Parted#Alignment), il faut un alignement à 2048 octets.

304687501 / 2048 = 148773,193847656 arrondi supérieur à 148774 et 148774 x 2048 = 304689152 (> 304687501)

312581807 / 2048 = 152627,835449219 arrondi inférieur à 152627 et 148774 x 2048 = 312580096 (< 312581807)

Création de cet espace aligné depuis cet espace libre
```sh
root@host:~# parted
(parted) print free
...
Number  Start       End         Size        Type     File system  Flags
        63s         2047s       1985s                Free Space
 1      2048s       304687500s  304685453s  primary  ext4         boot
        304687501s  312581807s  7894307s             Free Space

(parted) mkpart primary linux-swap(v1) 304689152s 312580096s
(parted) align-check optimal 1                                            
1 aligned
```
=> partition SWAP créée et alignée

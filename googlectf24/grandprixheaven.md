# GRAND PRIX HEAVEN - web

chall : [https://grandprixheaven-web.2024.ctfcompetition.com/](https://)
code : [https://github.com/google/google-ctf/tree/main/2024/quals/web-grandprixheaven/challenge](https://)

Lorsqu'on poste une voiture /new-fave, on a la possibilité de rajouter un champ "custom" à la main (avec burpsuite par exemple) qui va influencer le template.

Dans le serveur de template, on a un mediaparser qui est déprécié, il a été remplacé par apiparser. mediaparser est vulnérable aux XSS alors que apiparser ne l'est pas.

Une vérification est faite pour ne pas injecter le mediaparser, on doit bypass cette vérification.

Le bypass s'effectue avec le payload suivant
```{"1": "retrieve", "2": "faves", "4 \"\n\nfooter\n--GP_HEAVEN\nContent-Disposition: form-data;name=\"5\"\r\n\r\nmediaparser\r\n--GP_HEAVEN\nContent-Disposition: form-data;name=\"6": "footer"}```

On effectue ici 2 bypass en 1, le premier bypass c'est le check que la clé est un nombre, facilement bypassablement en javascript en introduisant un espace. Le deuxième bypass se fait au niveau du template serveur, on introduit une confusion sur les "chunks", on rajoute un chunk à l'intérieur d'un chunk ce qui permet d'introduire le template mediaparser.

Ce n'est pas fini, le mediaparser lit ses données sur un chemin différent de apiparseur. On voudrait faire un directory traversal mais ../ est bloqué par une regex. Cependant dans cette regex, il y a un range non standard A-z, si l'on regarde la table ascii, on a le droit au caractère "\\". On peut ainsi pointer sur notre image avec ?F1=\media\<mediaID>.

Pour finir, un petit point relou, la modification des metadonnées exiftool. J'ai été obligé de passer par la lib python exif pour modifier "ICC Profile Date" et par le binaire exiftool pour modifier "UserComment" et "ImageDescription".

Ouf c'est fini, il nous reste plus qu'à injecter notre XSS dans UserComment ou ImageDescription, sous forme ```<img src=x onerror="<javascript>">```, ce qui permet au javascript de s'executer au chargement. Et pour finir, on report l'url à /report pour récupérer le flag qui est dans les cookies du bot.

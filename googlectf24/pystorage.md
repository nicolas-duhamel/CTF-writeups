# PYSTORAGE - misc

```nc py-storage.2024.ctfcompetition.com 1337```

code : [https://github.com/google/google-ctf/blob/main/2024/quals/misc-py-storage/challenge/chal.py](https://)

Le flag est stockée dans un fichier sous forme de key:value
On a la possibilité d'ajouter et de récupérer des key:value, il y a des restrictions sur le prefixe "secret_" qui est réservé pour le password et le flag

Un check est effectué sur "\n" qui aurait comme effet d'écrire 2 entrée dans le fichier et de bypass la restriction du prefixe

Cependant on peut utiliser "\x0D" comme une new line, on envoie une première requête
```add tata toto\x0Dsecret_password:toto```

Pour réécrire le password dans la base de donnée (il faut envoyer la requête en hexadécimal)

On s'authentifie pour récupérer le flag
```auth toto get secret_flag```

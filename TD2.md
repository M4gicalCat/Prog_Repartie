# CLIENT/SERVEUR : PROBLÈMES CLASSIQUES

## Détection de fin d'envoi
- Pour une même requête, les données envoyées peuvent être de taille variable.
    - Exemple : serveur de fichier ou le client demande le contenu d'un fichier plus ou moins gros.
- Principalement 3 solutions :
    - Le serveur coupe la connexion après l'envoi des derniers octets.
      -> le client sait donc qu'il n'y a plus rien à recevoir.
    - Le serveur envoie une séquene d'octets distinctive pour signifier la terminaison -> le client doit détecter cette suite d'octet.
    - Le serveur prévient à l'avance du volue de données qu'il va envoyer

- Problèmes : 
    1) Le client doit se reconnecter constamment
    2) Fonctionne ssi la séquence n'apparaît jamais dans les données
    3) Plus simple, universelle. Il faut tout de même être capable de connaître la taille des donneés à envoyer au préalable.

- Pour envoyer beaucoup de donnnées, il faut les envoyer en plusieurs envois de préférence.

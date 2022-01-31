# Principes généraux sur les sockets

## Les domaines

Sous Unix, il existe différents domaines de sokets qui fixent le contexte de leur utilisation. Par exemple : 
- Domaine Internet : ces sockets utilisent des périphériques réseaux pour communiquer, par exemple entre deux applications.
- domaine Unix : Permettent de communiquer entre 2 applications s'éxectutant sur une même machine.

En C, le domaine est spécifié à l'appel de soocket().<br>
En Java, il n'existe que des classes pour le domaine Internet.

## Les protocoles des domaines Internet
- Une socket internet est obligatoirement associée à un protocole de communication qui va fixer les possibilités fonctionnelles de la socket et comment s'en servir
- Certains protocoles nécessitent une phase de connexion explicite entre sockets avant communication, d'autres non.

En Java, il est uniqument possible de créer des sockets TCP et UDP.

## Principes des modes connecté et non connecté
Soient A et B deux applications désirant communiquer via des sockets TCP. 

En mode connecté, A et B ont des rôles différents : 
### Application A
- créer une socket serveur
- associer la socket à un port
- attendre une demande de connexion
- accepter une connexion<br>
-> création automatique d'une socket de connexion
- communication

### Application B
- créer une socket de communication
- connexion via la soocket à l'IP de la machine exécutant A + port associé à A
- communication

En mode non connecté, il n'y a pas vraiment de différence entre A et B

### Application A
- créer une socket datagram
- lier la socket à un port
- créer un paquet datagram
- envoi du datagram via la socket à l'IP de la machine exécutant B + port de B

### Application B
- créer une socket datagram
- associer la socket à un port
- attendre la réception  d'un datagram (via la socket)

Dans ce cas, B peut recevoir de n'import quelle autre application et peut elle même envoyer des datagrammes.
C'est donc plutôt les fonctionnalités codées dans A et B qui vont définir les rôles client/serveur.

# Les classes de sockets en Java
En Java, le mode connecté utilise les classes `Socket` (communication) et `ServerSocket` (attente de connexion).
Le mode non connecté utilise `DatagramSocket` et `DatagramPacket`.<br>
En mode connecté, une socket de communication ne permet pas directement d'envoyer / recevoir des données. Autrement dit, il n'y a pas de méthode d'envoi / réception dans la classe `Socket`.

Pour communiquer il faut utiliser les classes de flux.

La classe `Socket` contient deux méthodes, mermettant de récupérer une instance de flux d'octets entrant et sortant de la socket : `getInputStream()` et `getOutputStream()`.

Comme pour les fichiers ordinaires, il est possible d'encapsuler ces flux d'octets dans des classes fournissant des fonctionnalités plus avancées pour envoyer / recevoir des tableaux de caractères, des lignes de textes, des types primaires, des objets etc.

## Règles basiques
Si une application C / S doit communiquer :

- **uniquement du texte** : on encapsule les flux d'octets dans des instances de `BufferedReader` et `PrintStream` afin de lire / écrire du texte.
- **dans toute autre situation** : on encapsule les flux d'octets dans des instances de `ObjectInputStream` et `ObjectOutputStream`.

En mode non connecté, les données sont empaquetées dans un objet `DatagramPacket` et c'est la classe `DatagramSocket` qui contient les méthodes d'envoi / réception de paquets

Il est impossible d'utiliser les flux et seuls des tableaux d'octets peuvent être échangés.

# Exemple basique

--> Client / serveur texte : Le serveur renvoie ce qu'il a reçu et se termine immédiatement

### Serveur
```java
import java.io.*;
import java.net.*;

class MyServer {
    public static void main(String[] args) {
        ServerSocket sockeConn = null; //attente de connexion
        Sockett sockComm = null;       //socket de communication
        int port = -1;
        PrintStream ps;
        BufferedReader br;

        try {
            port = Integer.parseInt(args[0]);
            sockConn = new ServerSocket(port);
            sockComm = sockConn.accept();
            ps = new PrintStream(sockComm.getOutputStream());
            br = new BufferedReader(new InputStreamReader(sockComm.getInputStream()));

            String msg = br.readLine(); //réception bloquante
            ps.println(msg); //envoi
        } catch (IOException e) {
            System.err.println("Pb socket : " + e.getMessage());
            System.exit(1);
        }
    }
}
```

### Client
```java
import java.io.*;
import java.net.*;

class  MyClient {
    public static void main(String[] args) {
        Socket sockComm = null; //communication
        String ipServ;
        int port = -1;
        PrintStream ps;
        BufferedReader br;

        try {
            ipServ = args[0];
            port = Integer.ParseInt(args[1]);
            sockConn = new Socket(ipServ, port);
            ps = new PrinstStream(sockConn.getOutputStream());
            br = new BufferedReader(new InputStreamReader(sockConn.getInputStream()));
            System.out.println("I say " + args[2]);
            String msg = br.reaLine();
            System.out.println("Server said " + msg);
        } catch (IOException e) {
            System.err.println("Pb socket : " + e.getMessage());
            System.exit(1);
        }
    }
}
```

# Remarques
- La façon d'établir la connexion est toujours la même, quelle que soit l'application. La partie communication changera selon les besoins.
- Il est inutile de mettre l'adresse et le port du serveur dans dans les paramètres du programme (plutôt que codés en dur)
- L'adresse du serveur peut $etre numérique au format A.B.C.D ou bien être un nom symbolique
- Normalement, un serveur ne se termine pas après la première socket, comme dans cet exemple de merde

# Définir un protocole de communication à base de requêtes

## Pourquoi un protocole ?
- Si un serveur propose différents services, il faut différencier la façon d'y accéder pour  un client.
- une demande du client est appelée une __requête__
- Pour que le serveur sache quoi faire quand il reçoit une requête, celle-ci est associée à un identifiant que le client envoie.
- Une requête peut avoir des paramètres, permettant au client de paramétrer le trantement que le serveur doit effectuer.
- Les requêtes doivent être structurées rigoureusement (programme =/= humain)
- Pour que le servue rpuisse être implémenté, il faut que :
    - L'identifiant de requête soit toujours de même type (int, string, etc)
    - L'identifiant soit la première chose que le client envoie au serveur
    - Le type (voire le format), l'ordre et le nombre de paramètres soient fixes pour une requête donnée.
    - La structure de la réponse soit fixe pour une requête donnée
- Généralement, on a pour une requête : envoi identifiant + paramètres côté client, puis envoi du résultat côté serveur.
- Il est cependant possible qu'il y ait plusieurs allers-retours, donc plusieurs sous-traitements côté serveur pour traiter entièrement la requête
- Définir un protocole de communication consiste à fixer : 
    - Le type de l'identifiant, et sa valeur pour chaque requête
    - La structure des informations échangées lors du traitement de la requête (type, nombre et ordre)
- Cela peut sembler simple, mais en pratique les cnoses se compliquent dès que l'on veut un serveur "poli", c'est à dire qui signale au client les erreurs dans ses requêtes 
- un solution simple est un serveur qui ne répond pas aux requêtes incorrectes -> rend le déboguage d'unc lient très difficile.
- Si le serveur doit signaler les erreurs au client, il faut prévoir ces messages dans le protocole tout en s'assurant qu'il reste cohérent quand il n'y a pas d'erreur

## Exemple : un serveur horaire
- Un serveur propose deux services : 
    - Mettre à jour l'heure courante du serveur
    - Donner l'heure actuelle pour un réseau horaire donné

### Exemple de protocole
- identifiant de requête : 1 octet
    - `1` pour la requête de misa à jour
    - `2` pour la requête heure fuseau.
- requête 1:
    - message 1 (client -> serveur) 
        - Identifiant | 1 octet
        - Heure       | 1 octet
        - Minute      | 1 octet
        - Seconde     | 1 octet
    - message 2 (serveur -> client)
        - `ack = 0` : tout est valide
        - `ack = 1` : heure invalide
        - `ack = 2` : minute invalide
        - `ack = 4` : seconde invalide
        - Si plusieurs invalides, on additione les valeurs
- requête 2:
    - message 1 (client -> serveur)
        - identifiant | 1 octet
        - fuseau | 1 entier sur 16 bits
    - message 2 (serveur -> client)
        - `ack = 0` : requête valide
        - `ack = 1` : requête invvalide
    - message 3 (serveur -> client) 
    <span style="color: red">seulement si `ock == 0`</span>
        - taille réponse | 1 octet
        - heure courante | string de longueur `taille réponse`
### Remarques
- La requête 2 illustre une situation classique de communication conditionnée : pas d'envoi de résultat en cas d'erreur
- Cette requête illustre aussi le principe des messages à taille variable

Ce protocole spécifie les structures avec des types primaires (octets etc). Cela permet d'être indépendant du langage de programmation et du système.
Le protocole est imparfait : 
- entier sur 16 bits => signé ou non ? complément à deux ?
- codage des caractères => ASCII ? iso ? utf ?
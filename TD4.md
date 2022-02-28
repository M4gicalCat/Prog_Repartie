## 2.4 Régler les conflits d’accès : mutex et attente d’évènement

- Dans l’exemple de la boîte, l’ordre d’éxécution des threads n’a pas vraiment d’importance.
- Pourtant, il semblerait logique que les instructions du for() ne puissent pas être interrompues
- On appelle cela une section critique
- Ce n’est pas réellement faisable au niveau applicatif (ça pet l(’être au niveau du noyau du système)
- En fait l’ordonnanceur des threads fixe un temps d’éxécution par thread, et un thread ne peut rien y changer.
- On ne peut donc jamais garantir qu’un thread puisse terminer une suite d’instructions assembleur avant de passer la main à un autre thread
- En revanche, on peut assurer qu’il n’y a qu’un seul thread à la fois qui exécute cette suite d’instructions, peu importe qu’il le fasse d’un seul tenant ou non

Pour mettre en place ce type de section critique, on utilise un mécanisme de programmation appellé `Mutex` (**mut**ual **ex**clusion).

D'autres pplications nécessitent de mettre en place un ordre d'exécution.
Par exemple, certains threads doivent attendre que d'autres aient terminé un certain traitement. Dans ce cas on utilise l'attente d'événement.

## Mutex
### Principe général
- On créé une variable de type mutex
- Juste avant la première instruction de la section critique, on "verrouille" le mutex.
- après l'opération critique, on le "déverrouille"

### Explication
- Quand un thread A appelle la méthode pour verrouiller le mutex, le système d'exploitation garde en mémoire quel thread l'a fait.
- Si A est interrompu au milieu de la section critique, il garde le verrou du mutex
- Si par la suite, un autre thread B essaie de verrouiller le mutex, le système le lui refuse et le met en attente : B perd la main et un autre thread C (ou A) s'exécute.
- Chaque fois que B reprend la main, il essaie de verrouiller le mutex, échoue et reperd la main.
- ainsi de suite jusqu'à ce que A finisse l'exécution de sa section critique, et déverrouille le mutex.
- Quand B reprend la main, il peut enfin verrouiller le mutex et s'exécuter.

## Remarque
Un mutex ne garanti en rien l'ordre d'exécution
- Quand A déverrouille le mutex, c'est le premier thread qui le demande qui l'obtient, et on ne peut pas le prévoir.

En Java, la mise en place du mutex est un peu spéciale.
Elle a pour but d'éviter au programmeur  d'appeler explicitement des méthodes de (dé)verrouillage .... et d'oublier de le faire.


Elle repose sur le fait que tout objet Java contient, de manière cachée, une variable mutex. Il n'existe pas de getter pou cette variable.
On ne peut manipuler cette variable qu'au travers du mot clé `synchronized`, qui a deux principes d'utilisation.

Pour l'illustrer, on suppose une classe A avec trois méthodes : meth1(), meth2() et meth3().

meth2() constitue à elle seule une section critique alors que seules quelques instructions de meth3() en forment une.
Supposons également que des threads t1, t2, ... aient tous accès à un objet objA instance de A;

### Rappel
verrouillage de l'objet courant grâce à une methode `syncronized`

```java
public void meth1() {}
public synchronized void meth2() {}
public synchronized void meth3() {}
```

Dans ce cas, si un thread essaie d'appeler une des méthodes `syncronized`, la JVM vérifie le mutex de `objA`

- S'il est libre, alors le thread peut exécuter le code, et le mutex lui appartient.
- Si le mutex est déjà verrouillé, le thread doit attendre.

### Remarque
Cela se produit même si les 2 threads appellent une méthode `synchronized` diifférente (le mutex est le même).
En revanche, `meth1()` n'est pas déclaré `synchronized`, donc un thread l'appelant n'est jamais bloqué.


```java
public void meth3() {
    synchronized(myMut) {
        //section critique
    }
}
```
- Cette fois, on utilise 2 mutex : celui de l'objet `objA` et celui de `objA.myMut`
- Il n'existe pas de syntaxe ppour spécifier que mutex utilise une méthode.
- En revanche, il existe un syntaxe permettant de déclarer un blac d'instructions comme étant protégé par le mutex de son choix
    - c'est la syntaxe utilisé dans meth3()

+ Deux avantages des blocs `synchronized` :
    - On peut utiliser plusieurs mutex et donc ne pas bloquer tous les threads qui veulent appeler des méthodes différentes.
    - On n'est pas obligé de mettre toute la méthode en section critique, juste certaines instructions.
+ Inconvénient :
    - en phase de développement, quand on change le code d'une méthode, il est facile d'oublie d'ajouter des instructions critiques dans un bloc synchronized.


## Attente d'événement
L'attente d'événement ne peut être dissociée des mutex.
En effet, pour mettre en place cette attente, on a besoin de mutex, car il protège les instructions dqui peut permettre à un thread de tester si ce qu'il attend s'est réellement passé. Reste donc à ajouter ces instructions permettant de spécifier qu'est ce qu'il attend, et de l'avertir quand l'évènement a eu lieu.

En Java, on utilise les instructions :
`wait()`, `notify()` et `notifyAll()`.

On n'attend pas directement l'attente causée par un mutex mais un deuxième type d'attente.
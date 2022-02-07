# LES THREADS
## Théorie
### Problématique
- Pour qu'une application fasse plusieurs traitements concurrents, il faut créer plusieurs processus.

- Le système d'exploitation se charge de donner du temps processeur à tous les processus qui s'exécutent, de façon plus ou moins équitable selon leur priorité

- Passer de l'exécution d'un processus à un autre s'appelle la __commutation de contexte__. Il s'agit en gros de prendre une photo de l'état d'un processus et de la conserver en mémoirre jusqu'à ce que du temps processeurr soit à nouveau donné au processus pour s'exécuter.

- Le problème est que le temps passé à prendre la photo dépend du processeur et du système d'exploitation, mais il est toujours long à l'échelle de la machine : quelques centaines de µsec.

C'est ce problème qui a été à l'origine des threads. Ils permettent de faire une application multitâche sans utiliser des processus classiques.

### Principes
- Un thread est un processus dit léger, car la commutation de contexte entre threads est beaucoup moins coûteuse : qques µsec (ou dizaines)

- Pour cela, il suffit de créer un deuxième niveau de découpage du temps processeur.

- En fait, un thread est une unité d'exécution interne à un processus classique.Le temps d'exécution alloué au processus va être partagé entre chaque thread du processus.

- Cela nécessite de créer un mécanisme d'ordonnancement des threads au sein du processus, qui va choisir quel thread s'exécute et pendant combien de temps.

- Comme pour les processus, on dit qu'un thread "perd / passe la main" lorsqu'il perd le contrôle du processeur, et "reprend la main" lorsqu'il reprend le contrôle.

- Tout cela est possible en utilisant des bibliothèques dédiées, par exemple la lib `pthread` sous Linux.

- Comme les threads s'exécutent au sein d'un processus, ils se partagent les ressources du processus et notamment son espace mémoire.

- Cela veut direque deux threads vont pouvoir accéder aux mêmes emplacements mémoire, et ce de façon naturelle, sans avoir à mettre en place des mécanismes logiciels complexes. Par exemple, en Java, deux threads vont pouvoir accéder à un même objet à condition d'en avoir une référence.

- Mais qui dit partage dit conflit potentiel : comme on ne sait jamais à quel moment tel ou tel thread s'exécute et pour combien de temps, on peut aboutir à des situations problématiques quand il s'agit de mettre à jour une zone mémoire.

Principes :
- Exclusion mutuelle
- Attente sur évenement

## Mise en œuvre en Java
### Créer des threads  
- L'API Java propose une classe Thread dont on peut hériter pour créer ses propres classes de threads (NB : il existe aussi la possibilité d'implémenter l'interface `Runnable`)
- La seule contrainte est qu'il faut définir une méthode `run()` qui constitue le `main()` du thread.

**Exemple :**
```java
class MyThread extends Thread {
    int value;
    public MyThread(int value) {
        this.value = value;
    }
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(value++);
        }
    }
}
```

NB : Le prototype de `run()` est invariable : pas de paramètres, pas de valeur de retour.
De ce fait, si le thread doit "recevoir" des paramètres, on le fait par le constructeur.

### Instanciation et exécution
- Quand on lance un programme Java, la JVM créé plusieurs threads, dont un pour le programme principal.
- Pour que ce dernier créé lui-même d'autres threads, il suffit d'instancier une classe de threads avec `new`.
- Pour démarrer le thread, on utilise sa méthode `start()`, qui est héritée de `Thread`.

**Exemple :** 
```java
class TestThread {
    public static void main(String[] args) {
        MyThread t1 = new MyThread(10);
        MyThread t2 = new MyThread(20);
        t1.start();
        t2.start();
    }
}
```

Si on se trompe et qu'on appelle `run()` à la place de `start()`, on ne crée pas de thread qui s'exécute en parallèle.

### Partage mémoire
- Pour que plusieurs threads aient accès à un même objet, la façon la plus simple est de fournir cet objet via le constructeur du Thread, et/ou via un setter.
- Dans ces deux cas, l'objet est référencé par un atrribt du thread.
- En général, on créé des objets partagés avant les threads, mais on peut le faire aprèès si on dispose d'un setter.

**Exemple :**

```java
class Box {
    int value;
    public Box() {
        value = 0;
    }
    public void inc (int v) {
        value += v;
    }
    public int getValue() {
        return value;
    }
}

class MyIyncThread extends Thread {
    Box b;
    int incr;

    public MyIyncThread(Box b, int incr){
        this.b = b;
        this.incr = incr;
    }

    public void run() {
        for (int i = 0; i < 100; i++) {
            b.inc(incr);
            System.out.println(b.getValue());
        }
    }

    class IncTest {
        public static void main(String[] args) {
            Box b = new Box();
            MyIncThread t1 = new MyIncThread(b, 2);
            MyIncThread t2 = new MyIncThread(b, 5);
            t1.start();
            t2.start();
        }
    }
}
```

Si on exécute plein de fois cet exemple, il peut y avoir des résultats différents. Il faut absolument considérer que l'ordre d'affichage peut être quelconque.
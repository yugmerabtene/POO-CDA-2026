# Polymorphisme en Java

Le mot **polymorphisme** signifie littéralement : **plusieurs formes**.

En programmation orientée objet, cela veut dire qu’une **même référence** ou une **même méthode** peut produire des comportements différents selon le **type réel de l’objet** ou selon les **paramètres reçus**.

Autrement dit, on écrit du code plus général, mais le programme sait s’adapter au bon comportement.

---

# 1. Pourquoi le polymorphisme est important

Le polymorphisme permet de :

* manipuler plusieurs objets différents avec une logique commune
* éviter de dupliquer le code
* rendre le programme plus souple
* faciliter l’évolution du projet

Exemple d’idée simple :

* un `Chien` peut aboyer
* un `Chat` peut miauler
* mais les deux sont des `Animal`

Donc on peut écrire du code qui manipule un `Animal`, tout en laissant chaque objet faire son comportement propre.

---

# 2. Les principaux types de polymorphisme en Java

Les types les plus importants sont :

## 2.1 Polymorphisme d’héritage

Aussi appelé :

* **polymorphisme par redéfinition**
* **polymorphisme dynamique**
* **runtime polymorphism**

## 2.2 Polymorphisme de surcharge

Aussi appelé :

* **polymorphisme statique**
* **compile-time polymorphism**

## 2.3 Polymorphisme via les interfaces

Techniquement, il repose aussi sur le polymorphisme dynamique, mais on le présente souvent à part car il est très important en Java.

---

# 3. Polymorphisme d’héritage

## 3.1 Principe

Une classe enfant hérite d’une classe parent et **redéfinit** une méthode.

Quand on appelle cette méthode via une référence du parent, Java exécute la version de la classe réelle de l’objet.

C’est le polymorphisme le plus important en POO.

## 3.2 Exemple simple

```java
public class Animal {
    public void parler() {
        System.out.println("L'animal fait un son.");
    }
}
```

```java
public class Chien extends Animal {
    @Override
    public void parler() {
        System.out.println("Le chien aboie.");
    }
}
```

```java
public class Chat extends Animal {
    @Override
    public void parler() {
        System.out.println("Le chat miaule.");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Animal a1 = new Chien();
        Animal a2 = new Chat();

        a1.parler();
        a2.parler();
    }
}
```

## 3.3 Résultat

```java
Le chien aboie.
Le chat miaule.
```

## 3.4 Ce qu’il faut comprendre

Dans :

```java
Animal a1 = new Chien();
```

* `Animal` est le **type de la référence**
* `Chien` est le **type réel de l’objet**

Au moment de l’appel :

```java
a1.parler();
```

Java regarde l’objet réel, donc ici `Chien`, et appelle :

```java
public void parler() {
    System.out.println("Le chien aboie.");
}
```

Donc le comportement dépend de l’objet réel, pas seulement du type écrit à gauche.

---

# 4. Pourquoi on parle de polymorphisme dynamique

On dit **dynamique** car le choix de la méthode est fait **à l’exécution**.

Le compilateur sait que `a1` est un `Animal`, donc il vérifie que la méthode `parler()` existe bien dans `Animal`.

Mais la version exacte à appeler est déterminée pendant l’exécution selon l’objet réel :

* si c’est un `Chien`, version `Chien`
* si c’est un `Chat`, version `Chat`

---

# 5. Redéfinition de méthode et polymorphisme

Le polymorphisme d’héritage repose sur la **redéfinition** de méthode.

## 5.1 Redéfinir une méthode

Cela signifie qu’une classe enfant réécrit une méthode déjà présente dans la classe parent avec :

* le même nom
* les mêmes paramètres
* un type de retour compatible

Exemple :

```java
@Override
public void parler() {
    System.out.println("Le chien aboie.");
}
```

L’annotation `@Override` n’est pas obligatoire, mais elle est fortement recommandée.

Elle permet à Java de vérifier qu’on redéfinit bien une méthode existante.

---

# 6. Exemple plus réaliste

## 6.1 Classe parent

```java
public class Paiement {
    public void payer() {
        System.out.println("Paiement générique.");
    }
}
```

## 6.2 Classes enfants

```java
public class PaiementCarte extends Paiement {
    @Override
    public void payer() {
        System.out.println("Paiement par carte bancaire.");
    }
}
```

```java
public class PaiementPaypal extends Paiement {
    @Override
    public void payer() {
        System.out.println("Paiement via PayPal.");
    }
}
```

## 6.3 Utilisation polymorphique

```java
public class Main {
    public static void main(String[] args) {
        Paiement p1 = new PaiementCarte();
        Paiement p2 = new PaiementPaypal();

        p1.payer();
        p2.payer();
    }
}
```

## 6.4 Intérêt

On peut manipuler tous les paiements comme des `Paiement`, sans réécrire une logique différente pour chaque classe.

---

# 7. Polymorphisme avec tableau ou collection

Le polymorphisme devient très puissant quand on manipule plusieurs objets ensemble.

```java
public class Main {
    public static void main(String[] args) {
        Animal[] animaux = {
            new Chien(),
            new Chat(),
            new Chien()
        };

        for (Animal animal : animaux) {
            animal.parler();
        }
    }
}
```

Ici :

* le tableau contient des références de type `Animal`
* chaque case contient un objet réel différent
* chaque objet exécute sa propre version de `parler()`

C’est un usage très classique du polymorphisme.

---

# 8. Polymorphisme de surcharge

## 8.1 Principe

La **surcharge** consiste à créer plusieurs méthodes avec le **même nom**, mais avec des **paramètres différents**.

Ici, ce n’est pas l’objet réel qui change le comportement, mais la **signature de la méthode**.

Exemple :

```java
public class Calculatrice {

    public int additionner(int a, int b) {
        return a + b;
    }

    public double additionner(double a, double b) {
        return a + b;
    }

    public int additionner(int a, int b, int c) {
        return a + b + c;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Calculatrice calculatrice = new Calculatrice();

        System.out.println(calculatrice.additionner(2, 3));
        System.out.println(calculatrice.additionner(2.5, 1.5));
        System.out.println(calculatrice.additionner(1, 2, 3));
    }
}
```

## 8.2 Résultat

```java
5
4.0
6
```

## 8.3 Pourquoi on parle de polymorphisme statique

On dit **statique** car le choix de la méthode est fait **à la compilation**.

Le compilateur regarde les paramètres :

* `additionner(int, int)`
* `additionner(double, double)`
* `additionner(int, int, int)`

Il sait immédiatement quelle version appeler.

---

# 9. Différence entre surcharge et redéfinition

C’est une confusion très fréquente.

## 9.1 Surcharge

Même classe, ou classe enfant, mais :

* même nom
* paramètres différents

Exemple :

```java
public void afficher(String texte)
public void afficher(int nombre)
```

## 9.2 Redéfinition

Classe enfant qui réécrit la méthode du parent :

* même nom
* mêmes paramètres
* comportement différent

Exemple :

```java
class Animal {
    public void parler() { ... }
}

class Chien extends Animal {
    @Override
    public void parler() { ... }
}
```

## 9.3 Résumé clair

* **surcharge** = plusieurs méthodes semblables, paramètres différents
* **redéfinition** = une méthode héritée est réécrite dans la classe enfant

---

# 10. Polymorphisme avec interface

## 10.1 Principe

Une interface définit un contrat.

Plusieurs classes peuvent l’implémenter, et on peut manipuler tous les objets via le type de l’interface.

## 10.2 Exemple

```java
public interface Notification {
    void envoyer();
}
```

```java
public class EmailNotification implements Notification {
    @Override
    public void envoyer() {
        System.out.println("Envoi d'un email.");
    }
}
```

```java
public class SmsNotification implements Notification {
    @Override
    public void envoyer() {
        System.out.println("Envoi d'un SMS.");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Notification n1 = new EmailNotification();
        Notification n2 = new SmsNotification();

        n1.envoyer();
        n2.envoyer();
    }
}
```

## 10.3 Ce qu’il faut comprendre

Ici, `Notification` ne sait pas comment envoyer.

Elle impose seulement une méthode :

```java
void envoyer();
```

Chaque classe décide de son comportement.

C’est un polymorphisme très utilisé dans les applications Java professionnelles, car il permet de découpler le code.

---

# 11. Polymorphisme et upcasting

## 11.1 Upcasting

Quand on fait :

```java
Animal animal = new Chien();
```

on transforme implicitement un objet plus spécifique en référence plus générale.

C’est un **upcasting**.

Java l’autorise automatiquement car un `Chien` est un `Animal`.

## 11.2 Intérêt

Cela permet d’écrire un code générique :

```java
public void faireParler(Animal animal) {
    animal.parler();
}
```

Puis :

```java
faireParler(new Chien());
faireParler(new Chat());
```

La même méthode fonctionne pour plusieurs types.

---

# 12. Polymorphisme et downcasting

## 12.1 Principe

Le **downcasting** consiste à convertir une référence générale vers un type plus spécifique.

Exemple :

```java
Animal animal = new Chien();
Chien chien = (Chien) animal;
```

## 12.2 Pourquoi faire attention

Cela peut provoquer une erreur si le type réel ne correspond pas.

Exemple dangereux :

```java
Animal animal = new Chat();
Chien chien = (Chien) animal;
```

Cela provoque une `ClassCastException`.

## 12.3 Version plus sûre

```java
Animal animal = new Chat();

if (animal instanceof Chien) {
    Chien chien = (Chien) animal;
    chien.parler();
}
```

---

# 13. Exemple complet mélangeant tout

```java
public class Employe {
    public void travailler() {
        System.out.println("L'employé travaille.");
    }
}
```

```java
public class Developpeur extends Employe {
    @Override
    public void travailler() {
        System.out.println("Le développeur écrit du code.");
    }
}
```

```java
public class Designer extends Employe {
    @Override
    public void travailler() {
        System.out.println("Le designer crée une interface.");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Employe[] equipe = {
            new Developpeur(),
            new Designer()
        };

        for (Employe employe : equipe) {
            employe.travailler();
        }
    }
}
```

Ici :

* `Employe` est le type commun
* `Developpeur` et `Designer` redéfinissent la méthode
* chaque objet exécute son propre comportement

C’est exactement le cœur du polymorphisme.

---

# 14. Ce que le polymorphisme n’est pas

Le polymorphisme ne veut pas dire :

* qu’une classe change de nature
* qu’un objet devient réellement une autre classe
* qu’on peut appeler n’importe quelle méthode sur une référence générale

Exemple :

```java
Animal animal = new Chien();
```

Même si l’objet réel est un `Chien`, avec la référence `animal`, on ne peut appeler que les méthodes visibles dans `Animal`.

Si `Chien` possède une méthode propre :

```java
public void garderMaison() {
    System.out.println("Le chien garde la maison.");
}
```

alors ceci ne compile pas :

```java
animal.garderMaison();
```

Pourquoi ?

Parce que la référence est de type `Animal`, et `Animal` ne connaît pas cette méthode.

---

# 15. Avantages du polymorphisme

## 15.1 Code plus propre

On évite les gros `if` ou `switch` pour tester le type des objets.

## 15.2 Code extensible

On peut ajouter une nouvelle classe enfant sans casser le code existant.

## 15.3 Code plus générique

On programme contre une classe parent ou une interface, pas contre une implémentation trop spécifique.

## 15.4 Maintenance facilitée

Le comportement spécifique reste dans la bonne classe.

---

# 16. Inconvénients ou points d’attention

## 16.1 Compréhension parfois plus difficile

Pour un débutant, il faut bien distinguer :

* type de la référence
* type réel de l’objet

## 16.2 Mauvais cast possible

Un mauvais downcasting peut provoquer une erreur à l’exécution.

## 16.3 Hiérarchie mal conçue

Si l’héritage est mal pensé, le polymorphisme devient confus.

---

# 17. Résumé final des types de polymorphisme en Java

## Polymorphisme statique

C’est la **surcharge**.

Le choix est fait à la compilation.

Exemple :

```java
additionner(int a, int b)
additionner(double a, double b)
```

## Polymorphisme dynamique

C’est la **redéfinition** via héritage ou interface.

Le choix est fait à l’exécution selon l’objet réel.

Exemple :

```java
Animal a = new Chien();
a.parler();
```

## Polymorphisme via interface

C’est une forme très utilisée du polymorphisme dynamique.

Exemple :

```java
Notification n = new EmailNotification();
n.envoyer();
```

---

# 18. Schéma mental simple à retenir

Tu peux retenir cela :

* **surcharge** : même nom, paramètres différents
* **redéfinition** : même méthode, comportement différent dans la classe enfant
* **polymorphisme** : une référence générale peut manipuler plusieurs objets réels différents

---

# 19. Mini tableau de synthèse

| Type         | Principe                                     | Moment du choix | Exemple                                  |
| ------------ | -------------------------------------------- | --------------- | ---------------------------------------- |
| Surcharge    | Même nom, paramètres différents              | Compilation     | `additionner(int, int)`                  |
| Redéfinition | Classe enfant réécrit la méthode du parent   | Exécution       | `Animal a = new Chien()`                 |
| Interface    | Plusieurs classes respectent le même contrat | Exécution       | `Notification n = new SmsNotification()` |

---

# 20. Formule simple pour examen ou cours

Tu peux dire :

> Le polymorphisme en Java est la capacité pour une même référence de manipuler des objets de types différents, tout en déclenchant le comportement adapté. Il existe principalement le polymorphisme statique par surcharge et le polymorphisme dynamique par redéfinition, souvent utilisé avec l’héritage et les interfaces.

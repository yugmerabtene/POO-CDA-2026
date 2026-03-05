# Cours — Généricité en Java (POO) avec `User`

## Code très commenté, explications après chaque fichier, résultat attendu et justification

---

## Introduction — Contexte et objectifs du cours

La généricité (generics) est un mécanisme du langage Java qui permet d’écrire du code **réutilisable** tout en conservant un typage **strict**. Elle intervient dès qu’un programme manipule des structures qui contiennent des objets (listes, stockages, retours de méthodes), et qu’il devient nécessaire d’exprimer clairement “ce que ces structures contiennent”.

### Problème que la généricité résout

Sans généricité, il est possible d’utiliser des collections sans préciser leur type. Dans ce cas :

* une collection peut contenir des objets de types différents
* lors de la lecture, la valeur revient souvent en `Object`
* pour utiliser cette valeur comme un `User`, il faut un cast
* le cast peut échouer à l’exécution (`ClassCastException`)

En pratique, le programme compile, mais l’erreur apparaît tard, au mauvais moment.

### Ce que la généricité apporte

Avec la généricité, le type contenu est déclaré :

* `List<User>` signifie : “cette liste contient uniquement des `User`”
* `Storage<User>` signifie : “ce stockage manipule uniquement des `User`”

Conséquences :

* le compilateur refuse l’ajout d’un type incompatible
* les casts deviennent inutiles
* les erreurs de type sont détectées à la compilation

### Notions incluses dans ce cours

* `<T>` : type paramétré, rend une classe ou interface réutilisable.
* `<>` (diamond operator) : Java déduit le type paramétré lors du `new`.
* `T extends HasId` : contrainte sur le type paramétré, impose un contrat.
* `Optional<T>` : valeur présente ou absente, sans utiliser `null`.

### Préambule essentiel

La généricité s’appuie sur un concept fondamental : **une classe définit un type, et les objets sont des instances de ce type**.
Ainsi, `List<User>` signifie “liste d’instances de `User`”.

---

## Étape 1 — Contrat `HasId` (prépare la contrainte `T extends HasId`)

### `HasId.java`

```java
/**
 * Interface (contrat) : tout type qui "a un identifiant" doit fournir getId().
 *
 * Pourquoi ce fichier existe ?
 * - Plus loin, on veut écrire une classe générique avec contrainte :
 *   IdStorage<T extends HasId>
 * - Cette contrainte exige que T possède la méthode getId().
 */
public interface HasId {

    /**
     * Retourne l'identifiant de l'objet.
     * On choisit Long pour rester simple dans le cours.
     */
    Long getId();
}
```

**Contexte**
Ce contrat sert à imposer une règle à un type paramétré : si un type veut être utilisé comme `T`, il doit fournir `getId()`.

**Résultat attendu à l’exécution**
Aucun : ce fichier ne contient pas de `main`.

**Pourquoi**
C’est une pièce de conception : elle permet au compilateur de garantir que `getId()` existe sur `T`.

---

## Étape 2 — Définir le type métier `User`

### `User.java`

```java
/**
 * Classe métier : User.
 *
 * Points importants :
 * - Une classe définit un TYPE.
 * - Un objet (instance) est créé avec new User(...).
 * - User implémente HasId pour pouvoir être utilisé dans T extends HasId.
 */
public class User implements HasId {

    // Attributs : état interne d'un utilisateur
    private Long id;
    private String email;

    /**
     * Constructeur : initialise l'objet lors de sa création.
     * new User(1L, "alice@mail.com")
     */
    public User(Long id, String email) {
        this.id = id;
        this.email = email;
    }

    /**
     * Implémentation du contrat HasId.
     * Grâce à cette méthode, User respecte l'interface HasId.
     */
    @Override
    public Long getId() {
        return id;
    }

    /**
     * Méthode métier simple : retourne l'email.
     */
    public String getEmail() {
        return email;
    }

    /**
     * Représentation texte utile en démonstration.
     */
    @Override
    public String toString() {
        return "User{id=" + id + ", email='" + email + "'}";
    }
}
```

**Contexte**
Le cours utilise `User` comme exemple concret. C’est un type métier (comme “Client”, “Produit”, etc.).
La classe existe pour que le compilateur sache ce que signifie `User`.

**Résultat attendu à l’exécution**
Aucun : ce fichier ne contient pas de `main`.

**Pourquoi**
On ne peut pas utiliser `User` comme type (`List<User>`, `Storage<User>`) si la classe n’est pas déclarée.

---

## Étape 3 — Classe (type) et objet (instance)

### `Step1_UserCreation.java`

```java
public class Step1_UserCreation {

    public static void main(String[] args) {

        /*
         * User = TYPE (défini par la classe User).
         *
         * new User(...) = création d'un OBJET en mémoire.
         *
         * alice = référence typée :
         * - elle référence un objet User
         * - elle ne peut pas référencer un objet d'un autre type
         */
        User alice = new User(1L, "alice@mail.com");

        /*
         * Appel d'une méthode :
         * - getEmail() existe dans User, donc le compilateur autorise cet appel.
         */
        System.out.println("Email : " + alice.getEmail());

        /*
         * Affichage de l'objet :
         * - System.out.println(alice) appelle alice.toString().
         */
        System.out.println("Objet : " + alice);
    }
}
```

**Contexte**
Avant la généricité, il faut comprendre ce que signifie “type” et “instance”.
La généricité réutilise exactement cette notion : `List<User>` signifie “liste d’instances de User”.

**Résultat attendu**

```text
Email : alice@mail.com
Objet : User{id=1, email='alice@mail.com'}
```

**Pourquoi**
On établit la base : `User` est un type, un `User` concret est un objet (instance).

---

## Étape 4 — Problème : liste non typée (raw type)

### `Step2_RawListProblem.java`

```java
import java.util.ArrayList;
import java.util.List;

public class Step2_RawListProblem {

    /*
     * Exemple concret :
     * On veut envoyer un email à un utilisateur.
     * Cette méthode exige un User.
     */
    private static void sendWelcomeEmail(User user) {
        System.out.println("Bienvenue envoyé à : " + user.getEmail());
    }

    public static void main(String[] args) {

        /*
         * List non typée :
         * - on n'écrit pas List<User>
         * - donc la liste peut contenir n'importe quel type d'objet
         */
        List users = new ArrayList();

        // Ajout correct : un User
        users.add(new User(1L, "alice@mail.com"));

        /*
         * Ajout incorrect :
         * On ajoute une String par erreur.
         * La liste non typée l'accepte, car elle n'impose aucun type.
         */
        users.add("ADMIN");

        /*
         * Plus tard :
         * On parcourt la liste en supposant que tous les éléments sont des User.
         */
        for (int i = 0; i < users.size(); i++) {

            /*
             * get(...) renvoie Object :
             * - Java ne sait pas quel type est dans la liste
             * - donc il renvoie le type le plus général : Object
             */
            Object element = users.get(i);

            /*
             * Cast forcé :
             * On tente de transformer l'Object en User.
             * - Si element est un User : OK
             * - Si element est une String : ClassCastException à l'exécution
             */
            User user = (User) element; // crash lorsque i = 1

            // Traitement métier
            sendWelcomeEmail(user);
        }
    }
}
```

**Contexte**
Cette étape illustre le vrai problème : une collection non typée permet des erreurs silencieuses.

**Résultat attendu**
Le programme affiche :

```text
Bienvenue envoyé à : alice@mail.com
```

Puis il échoue avec `ClassCastException`.

**Pourquoi**
Sans `List<User>`, la liste accepte des types incohérents. L’erreur n’est détectée qu’au moment du cast.

---

## Étape 5 — Solution : `List<User>` (généricité)

### `Step3_GenericListSafe.java`

```java
import java.util.ArrayList;
import java.util.List;

public class Step3_GenericListSafe {

    private static void sendWelcomeEmail(User user) {
        System.out.println("Bienvenue envoyé à : " + user.getEmail());
    }

    public static void main(String[] args) {

        /*
         * List<User> :
         * - la liste est typée
         * - elle ne peut contenir que des User
         * - le compilateur impose cette règle
         */
        List<User> users = new ArrayList<>();

        users.add(new User(1L, "alice@mail.com"));
        users.add(new User(2L, "bob@mail.com"));

        /*
         * Interdit :
         * Décommenter la ligne suivante provoque une erreur de compilation.
         * users.add("ADMIN");
         */

        for (int i = 0; i < users.size(); i++) {

            /*
             * get(...) renvoie directement un User, pas Object.
             * - plus besoin de cast
             */
            User user = users.get(i);

            sendWelcomeEmail(user);
        }
    }
}
```

**Contexte**
La généricité exprime un contrat : “cette liste contient uniquement des `User`”.

**Résultat attendu**

```text
Bienvenue envoyé à : alice@mail.com
Bienvenue envoyé à : bob@mail.com
```

**Pourquoi**
Le compilateur garantit le type des éléments. L’erreur est bloquée avant l’exécution.

---

## Étape 6 — `<T>` : classe générique réutilisable

### `Storage.java`

```java
/**
 * Classe générique :
 * - <T> signifie : type paramétré
 * - T sera remplacé par un type réel lors de l'utilisation
 *
 * Exemple :
 * - Storage<User>   => T devient User
 * - Storage<String> => T devient String
 */
public class Storage<T> {

    /*
     * Attribut de type T :
     * - si Storage<User>, alors value est un User
     * - si Storage<String>, alors value est un String
     */
    private T value;

    /**
     * Stocke une valeur de type T.
     */
    public void set(T value) {
        this.value = value;
    }

    /**
     * Retourne la valeur stockée, de type T.
     */
    public T get() {
        return value;
    }
}
```

**Contexte**
On montre la généricité au niveau des classes : ce n’est pas réservé aux collections.

**Résultat attendu**
Aucun : pas de `main`.

**Pourquoi**
La classe est écrite une seule fois et spécialisée ensuite par `Storage<User>`, `Storage<Integer>`, etc.

---

## Étape 7 — `<>` : diamond operator (déduction du type au `new`)

### `Step4_StorageUser.java`

```java
public class Step4_StorageUser {

    public static void main(String[] args) {

        /*
         * À gauche : Storage<User> fixe le type paramétré.
         *
         * À droite : new Storage<>() utilise le diamond operator.
         * Java déduit automatiquement que c'est new Storage<User>().
         */
        Storage<User> storage = new Storage<>();

        // set attend un User car T = User
        storage.set(new User(1L, "alice@mail.com"));

        /*
         * Interdit :
         * Décommenter provoque une erreur de compilation car T = User.
         * storage.set("ADMIN");
         */

        // get renvoie un User car T = User
        User u = storage.get();
        System.out.println("Email : " + u.getEmail());
    }
}
```

**Contexte**
Le diamond operator évite de répéter le type et conserve le typage strict.

**Résultat attendu**

```text
Email : alice@mail.com
```

**Pourquoi**
Le type est déduit à partir de `Storage<User>` à gauche.

---

## Étape 8 — `T extends HasId` : imposer une règle sur le type paramétré

### `IdStorage.java`

```java
/**
 * Classe générique contrainte :
 * - T extends HasId signifie :
 *   "T doit implémenter HasId"
 *
 * Conséquence :
 * - on peut appeler value.getId() sans risque,
 *   car le compilateur garantit l'existence de getId().
 */
public class IdStorage<T extends HasId> {

    private T value;

    public void set(T value) {
        this.value = value;
    }

    /**
     * Méthode possible uniquement grâce à la contrainte :
     * getId() existe forcément sur T.
     */
    public Long storedId() {
        return value.getId();
    }

    public T get() {
        return value;
    }
}
```

**Contexte**
On veut garantir qu’un type paramétré respecte un contrat minimal.

**Résultat attendu**
Aucun : pas de `main`.

**Pourquoi**
La contrainte rend certains appels possibles (`getId()`), sinon le compilateur refuserait.

---

### `Step5_IdStorageUser.java`

```java
public class Step5_IdStorageUser {

    public static void main(String[] args) {

        /*
         * User implémente HasId, donc il est accepté par IdStorage<T extends HasId>.
         */
        IdStorage<User> storage = new IdStorage<>();

        storage.set(new User(1L, "alice@mail.com"));

        System.out.println("Id stocké : " + storage.storedId());
        System.out.println("Email : " + storage.get().getEmail());
    }
}
```

**Contexte**
Démonstration concrète de la contrainte : `User` est compatible avec `T extends HasId`.

**Résultat attendu**

```text
Id stocké : 1
Email : alice@mail.com
```

**Pourquoi**
`storedId()` fonctionne car `getId()` est garanti par la contrainte générique.

---

## Étape 9 — `Optional<T>` : présent ou absent sans `null`

### `UserFinder.java`

```java
import java.util.List;
import java.util.Optional;

/**
 * Recherche d'un utilisateur par email.
 *
 * Retour :
 * - Optional<User> au lieu de null
 * - Optional.of(user) si trouvé
 * - Optional.empty() si absent
 */
public class UserFinder {

    public static Optional<User> findByEmail(List<User> users, String email) {

        // Défense simple : si paramètres invalides, on retourne "absent"
        if (users == null || email == null) {
            return Optional.empty();
        }

        // Parcours classique
        for (int i = 0; i < users.size(); i++) {
            User u = users.get(i);

            // Comparaison d'email
            if (email.equals(u.getEmail())) {
                // Valeur présente
                return Optional.of(u);
            }
        }

        // Valeur absente
        return Optional.empty();
    }
}
```

**Contexte**
Au lieu de retourner `null` quand l’utilisateur n’existe pas, on retourne `Optional.empty()`.

**Résultat attendu**
Aucun : pas de `main`.

**Pourquoi**
Le type de retour force le consommateur à gérer l’absence de valeur.

---

### `Step6_OptionalDemo.java`

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

public class Step6_OptionalDemo {

    public static void main(String[] args) {

        // Liste typée de User
        List<User> users = new ArrayList<>();
        users.add(new User(1L, "alice@mail.com"));
        users.add(new User(2L, "bob@mail.com"));

        // Recherche : présent
        Optional<User> found = UserFinder.findByEmail(users, "alice@mail.com");

        // Recherche : absent
        Optional<User> missing = UserFinder.findByEmail(users, "nobody@mail.com");

        // Gestion du cas présent
        if (found.isPresent()) {
            System.out.println("Trouvé : " + found.get().getEmail());
        } else {
            System.out.println("Absent");
        }

        // Gestion du cas absent
        if (missing.isPresent()) {
            System.out.println("Trouvé : " + missing.get().getEmail());
        } else {
            System.out.println("Absent");
        }

        /*
         * orElse(...) :
         * - si missing est vide => renvoie l'utilisateur par défaut
         * - sinon => renvoie la valeur présente
         */
        User defaultUser = missing.orElse(new User(-1L, "default@mail.com"));
        System.out.println("Par défaut : " + defaultUser.getEmail());
    }
}
```

**Contexte**
Démonstration de `Optional<T>` : une valeur trouvée et une valeur absente.

**Résultat attendu**

```text
Trouvé : alice@mail.com
Absent
Par défaut : default@mail.com
```

**Pourquoi**
`Optional.empty()` représente l’absence explicitement et évite l’utilisation de `null`.

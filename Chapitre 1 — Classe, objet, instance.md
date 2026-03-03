## Chapitre 1 — Classe, objet, instance (Java)

### Objectifs

* Comprendre la différence entre une classe et un objet
* Savoir créer des instances
* Comprendre la notion de référence en Java

---

### 1) Classe

```java id="ch1_user"
public class User {
    // Identifiant de l'utilisateur
    long id;

    // Nom de l'utilisateur
    String name;

    // Email de l'utilisateur
    String email;
}
```

Explications

* `public class User` : définition d’une classe. Elle décrit le modèle d’un utilisateur.
* `id`, `name`, `email` : champs (données) qui décrivent l’état d’un utilisateur.
* Une classe ne représente pas un utilisateur concret : elle sert à créer des objets.

---

### 2) Objet et instance

```java id="ch1_instances"
public class Main {
    public static void main(String[] args) {

        // Création d'un premier objet (instance) de la classe User
        User u1 = new User();
        u1.id = 1;
        u1.name = "Alice";
        u1.email = "alice@example.com";

        // Création d'un second objet (instance) différent
        User u2 = new User();
        u2.id = 2;
        u2.name = "Bob";
        u2.email = "bob@example.com";

        // Affichage simple
        System.out.println(u1.name);
        System.out.println(u2.name);
    }
}
```

Explications

* `new User()` : création d’un objet en mémoire, donc une instance de `User`.
* `u1` et `u2` : deux objets différents, chacun avec son propre état.
* Modifier `u1.name` ne modifie pas `u2.name` car ce sont deux instances distinctes.

---

### 3) Référence en Java

```java id="ch1_reference"
public class Main {
    public static void main(String[] args) {

        // Création d'un objet User
        User a = new User();
        a.name = "Alice";

        // Copie de référence : aucun nouvel objet n'est créé
        User b = a;

        // Modification via b
        b.name = "Alice2";

        // Les deux variables pointent vers le même objet
        System.out.println(a.name); // Alice2
        System.out.println(b.name); // Alice2
    }
}
```

Explications

* `User b = a` copie la référence : `a` et `b` pointent vers le même objet.
* Il n’y a pas de `new` dans cette ligne, donc pas de nouvelle instance.
* Modifier l’objet via `b` impacte ce que voit `a`, car c’est le même objet.

---

### Points clés à retenir

* Une classe est un modèle
* Un objet est une instance créée avec `new`
* En Java, les objets sont manipulés via des références

---

### Exercice

1. Créer la classe `User` avec `id`, `name`, `email`
2. Créer trois instances : `admin`, `client`, `guest`
3. Afficher uniquement le `name` de chaque utilisateur

---

### Complément — Valeurs simples et références dans les champs

En Java, il faut distinguer deux cas quand un objet possède des champs :

* un champ de type simple (comme `long`) stocke directement une valeur
* un champ de type objet (comme `String`) stocke une référence vers un objet

Exemple : effet “référence” sur l’objet `User`

```java id="ch1_ref_user"
public class Main {
    public static void main(String[] args) {
        User a = new User();
        a.id = 1;
        a.name = "Alice";

        User b = a;   // b pointe vers le même User
        b.id = 2;     // modification via b

        System.out.println(a.id); // 2
    }
}
```

Explications

* `id` est un champ de type `long` : on y stocke directement un nombre.
* `b = a` copie la référence de l’objet `User`, donc `a` et `b` désignent le même utilisateur.
* `b.id = 2` modifie l’unique objet, donc `a.id` affiche aussi `2`.

Remarque sur `String`

* `name` est un champ de type `String` : il contient une référence vers un objet `String`.
* Quand on fait `a.name = "Alice2"`, on ne “modifie pas” le texte `"Alice"` : on fait simplement pointer `name` vers une autre valeur `String`.

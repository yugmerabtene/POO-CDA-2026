## Chapitre 17 — `static` : attributs de classe, méthodes de classe, constantes

### Objectifs

* Comprendre la différence entre ce qui appartient à un objet et ce qui appartient à la classe
* Utiliser `static` pour créer un attribut partagé entre tous les objets
* Utiliser `static` pour créer une méthode utilitaire
* Utiliser `static final` pour créer une constante
* Garder la structure par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch17_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
```

Explications

* Ce chapitre se concentre sur `static`, donc on garde un projet minimal.
* `model` : modèle utilisateur.
* `app` : démonstration.

---

## Code complet du chapitre

### `model/UserModel.java`

```java id="ch17_user_model"
package model;

public class UserModel {

    // Attribut de classe (shared) : compteur global pour tous les utilisateurs
    private static long nextId = 1;

    // Constante de classe : valeur fixe, partagée, non modifiable
    public static final int MIN_PASSWORD_LENGTH = 8;

    // Attributs d'objet : chaque utilisateur a ses propres valeurs
    private final long id;
    private final String email;

    // Constructeur : chaque création consomme un id unique
    public UserModel(String email) {
        this.id = nextId;
        nextId++;
        this.email = email;
    }

    public long getId() {
        return id;
    }

    public String getEmail() {
        return email;
    }

    // Méthode de classe : utilitaire, ne dépend pas d'un objet
    public static boolean isPasswordValid(String password) {
        if (password == null) {
            return false;
        }
        return password.length() >= MIN_PASSWORD_LENGTH;
    }
}
```

Explications

* `private static long nextId` :

  * `static` signifie que l’attribut appartient à la classe, pas à un objet.
  * il est partagé par tous les `UserModel`.
  * il sert ici de compteur global pour générer des identifiants.
* `public static final int MIN_PASSWORD_LENGTH` :

  * `static final` : constante de classe.
  * `final` signifie que la valeur ne peut pas changer.
* `private final long id` et `private final String email` :

  * ce sont des attributs d’objet : chaque utilisateur a les siens.
* `public static boolean isPasswordValid(...)` :

  * méthode de classe : on peut l’appeler sans créer d’objet (`UserModel.isPasswordValid(...)`).
  * elle utilise la constante `MIN_PASSWORD_LENGTH`.

---

### `app/Main.java`

```java id="ch17_main"
package app;

import model.UserModel;

public class Main {
    public static void main(String[] args) {

        // Création de deux utilisateurs : nextId est partagé, donc les id diffèrent
        UserModel u1 = new UserModel("alice@example.com");
        UserModel u2 = new UserModel("bob@example.com");

        System.out.println("u1 id=" + u1.getId() + " email=" + u1.getEmail());
        System.out.println("u2 id=" + u2.getId() + " email=" + u2.getEmail());

        // Appel d'une méthode static sans objet
        System.out.println("pass 'abc' valid = " + UserModel.isPasswordValid("abc"));
        System.out.println("pass 'abcdefgh' valid = " + UserModel.isPasswordValid("abcdefgh"));

        // Accès à une constante static final
        System.out.println("MIN_PASSWORD_LENGTH = " + UserModel.MIN_PASSWORD_LENGTH);
    }
}
```

Explications

* `UserModel.isPasswordValid(...)` :

  * appel direct de la méthode de classe, sans objet.
* `UserModel.MIN_PASSWORD_LENGTH` :

  * accès direct à une constante de classe.
* `u1` et `u2` reçoivent des identifiants différents car `nextId` est partagé.

---

### Points clés à retenir

* `static` : appartient à la classe, partagé entre tous les objets
* attribut d’objet : appartient à un objet, chaque objet a sa valeur
* méthode `static` : utilitaire, appelable sans objet
* `static final` : constante de classe, non modifiable

## Chapitre 9 — Surcharge et redéfinition

### Objectifs

* Comprendre la surcharge (overloading) : même nom de méthode, paramètres différents
* Comprendre la redéfinition (override) : une classe enfant remplace le comportement d’une méthode héritée
* Continuer la gestion d’utilisateurs et préparer l’authentification

---

### Structure de projet utilisée

```text id="ch9_tree_full"
src/
  app/
    Main.java
  model/
    UserModel.java
    AdminUserModel.java
  service/
    AuthenticatorService.java
    PasswordAuthenticatorService.java
```

Explications

* `model` : modèles utilisateurs
* `service` : logique d’authentification
* `app` : exécution et tests

---

## Code complet du chapitre

### `model/UserModel.java`

```java id="ch9_user_model_full"
package model;

public class UserModel {

    // Attributs communs
    private long id;
    private String name;
    private String email;

    public UserModel(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    // Méthode commune : sera redéfinie dans une classe enfant
    public void display() {
        System.out.println("USER id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `id`, `name`, `email` sont des attributs : ils représentent l’état d’un utilisateur.
* `display()` est une méthode du parent : une classe enfant pourra la redéfinir (override) pour adapter le comportement.

---

### `model/AdminUserModel.java`

```java id="ch9_admin_user_model_full"
package model;

public class AdminUserModel extends UserModel {

    // Attribut spécifique aux administrateurs
    private int accessLevel;

    public AdminUserModel(long id, String name, String email, int accessLevel) {
        super(id, name, email); // initialise la partie héritée
        this.accessLevel = accessLevel;
    }

    public int getAccessLevel() {
        return accessLevel;
    }

    // Redéfinition : même signature que display() dans UserModel
    @Override
    public void display() {
        // Réutilisation de l'affichage commun
        super.display();

        // Ajout de l'information spécifique
        System.out.println("ADMIN accessLevel=" + accessLevel);
    }
}
```

Explications

* `extends UserModel` : `AdminUserModel` hérite des attributs et méthodes publiques du parent.
* `@Override` : indique que la méthode `display()` remplace celle du parent pour un admin.
* `super.display()` : appelle la méthode du parent avant d’ajouter la partie spécifique.

---

### `service/AuthenticatorService.java`

```java id="ch9_authenticator_service_full"
package service;

import model.UserModel;

public interface AuthenticatorService {

    // Contrat : tenter d'authentifier un utilisateur avec un secret
    boolean authenticate(UserModel user, String secret);
}
```

Explications

* `interface` : définit un contrat.
* `boolean` : résultat simple `true` (succès) ou `false` (échec).
* `UserModel user` : paramètre typé avec un type que nous avons créé (`UserModel`).

---

### `service/PasswordAuthenticatorService.java`

```java id="ch9_password_authenticator_service_full"
package service;

import model.UserModel;

public class PasswordAuthenticatorService implements AuthenticatorService {

    // Attribut interne du service : secret attendu (démonstration)
    private String expectedSecret;

    public PasswordAuthenticatorService(String expectedSecret) {
        this.expectedSecret = expectedSecret;
    }

    // Méthode imposée par l'interface (signature exacte)
    @Override
    public boolean authenticate(UserModel user, String secret) {
        if (user == null) {
            return false;
        }
        if (secret == null) {
            return false;
        }
        return expectedSecret.equals(secret);
    }

    // Surcharge : même nom de méthode, paramètres différents (char[] au lieu de String)
    public boolean authenticate(UserModel user, char[] secretChars) {
        if (user == null) {
            return false;
        }
        if (secretChars == null) {
            return false;
        }

        // Conversion char[] -> String uniquement pour la démonstration
        String secret = new String(secretChars);

        // Réutilisation de la méthode authenticate(UserModel, String)
        return authenticate(user, secret);
    }
}
```

Explications

* **Surcharge (overloading)** : ici, il y a deux méthodes `authenticate` :

  * `authenticate(UserModel, String)`
  * `authenticate(UserModel, char[])`
    Elles coexistent, Java choisit selon le type de l’argument.
* La méthode `authenticate(UserModel, String)` est celle du contrat (interface).
* La méthode `authenticate(UserModel, char[])` est une méthode supplémentaire qui illustre la surcharge.

---

### `app/Main.java`

```java id="ch9_main_full"
package app;

import model.AdminUserModel;
import model.UserModel;
import service.PasswordAuthenticatorService;

public class Main {
    public static void main(String[] args) {

        // 1) Test de la redéfinition (override)
        UserModel admin = new AdminUserModel(1, "Admin", "admin@example.com", 10);
        admin.display();

        // 2) Test de la surcharge (overloading)
        UserModel user = new UserModel(2, "Alice", "alice@example.com");
        PasswordAuthenticatorService auth = new PasswordAuthenticatorService("secret123");

        boolean ok1 = auth.authenticate(user, "secret123");

        char[] secretChars = new char[] {'s','e','c','r','e','t','1','2','3'};
        boolean ok2 = auth.authenticate(user, secretChars);

        System.out.println("ok1=" + ok1);
        System.out.println("ok2=" + ok2);
    }
}
```

Explications

* `UserModel admin = new AdminUserModel(...)` : un objet enfant est référencé par un type parent.
* `admin.display()` : exécute la version redéfinie dans `AdminUserModel` (override).
* `auth.authenticate(user, "secret123")` appelle la méthode avec `String`.
* `auth.authenticate(user, secretChars)` appelle la méthode surchargée avec `char[]`.

---

### Points clés à retenir

* **Surcharge** : même nom de méthode, paramètres différents, plusieurs méthodes coexistent
* **Redéfinition** : même signature, comportement remplacé dans la classe enfant
* `@Override` sécurise la redéfinition
* `super.display()` réutilise le comportement commun avant d’ajouter le spécifique

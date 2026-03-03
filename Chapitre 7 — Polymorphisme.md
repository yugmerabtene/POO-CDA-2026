## Chapitre 7 — Polymorphisme

### Objectifs

* Comprendre le polymorphisme : manipuler plusieurs types d’objets via un type commun
* Observer que le comportement exécuté dépend de l’objet réel (admin, client, etc.)
* Continuer la gestion d’utilisateurs en restant structuré par packages

---

### Structure de projet utilisée

```text id="ch7_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
    AdminUserModel.java
    ClientUserModel.java
```

Explications

* `model` contient les modèles utilisateurs.
* `app` contient le point d’entrée pour tester.

---

### 1) Base commune : `UserModel`

```java id="ch7_user_model"
package model;

public class UserModel {

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

    // Méthode commune : elle pourra être utilisée sur tous les utilisateurs
    public void display() {
        System.out.println("USER id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `display()` est une méthode commune disponible sur tous les utilisateurs.
* Elle sert de base pour montrer le polymorphisme : on va appeler `display()` via le type `UserModel`, mais l’objet réel pourra être un admin ou un client.

---

### 2) Spécialisation : `AdminUserModel`

```java id="ch7_admin_user_model"
package model;

public class AdminUserModel extends UserModel {

    private int accessLevel;

    public AdminUserModel(long id, String name, String email, int accessLevel) {
        super(id, name, email);
        this.accessLevel = accessLevel;
    }

    public int getAccessLevel() {
        return accessLevel;
    }

    // Redéfinition de la méthode display() pour un admin
    @Override
    public void display() {
        // On réutilise l'affichage commun du parent
        super.display();

        // Puis on affiche la partie spécifique admin
        System.out.println("ADMIN accessLevel=" + accessLevel);
    }
}
```

Explications

* `@Override` indique que la méthode `display()` existe déjà dans `UserModel` et qu’on la redéfinit ici pour `AdminUserModel`.
* `super.display()` appelle la version du parent, puis on ajoute l’information spécifique admin.
* Résultat : un admin a un affichage plus complet, sans dupliquer l’affichage commun.

---

### 3) Spécialisation : `ClientUserModel`

```java id="ch7_client_user_model"
package model;

public class ClientUserModel extends UserModel {

    private String customerId;

    public ClientUserModel(long id, String name, String email, String customerId) {
        super(id, name, email);
        this.customerId = customerId;
    }

    public String getCustomerId() {
        return customerId;
    }

    // Redéfinition de la méthode display() pour un client
    @Override
    public void display() {
        // On réutilise l'affichage commun du parent
        super.display();

        // Puis on affiche la partie spécifique client
        System.out.println("CLIENT customerId=" + customerId);
    }
}
```

Explications

* Même logique que pour l’admin : on redéfinit `display()` pour ajouter la partie spécifique client.
* On garde un comportement commun (parent) et on ajoute uniquement ce qui est propre au type.

---

### 4) Test du polymorphisme : `Main`

```java id="ch7_main"
package app;

import model.AdminUserModel;
import model.ClientUserModel;
import model.UserModel;

public class Main {
    public static void main(String[] args) {

        // Polymorphisme : une variable de type UserModel peut référencer
        // un AdminUserModel ou un ClientUserModel car ils héritent de UserModel.
        UserModel u1 = new AdminUserModel(1, "Admin", "admin@example.com", 10);
        UserModel u2 = new ClientUserModel(2, "Client", "client@example.com", "C-100");

        // Appel de la même méthode sur le même type (UserModel)
        // mais le comportement exécuté dépend de l'objet réel.
        u1.display(); // affichage admin (version redéfinie)
        u2.display(); // affichage client (version redéfinie)

        // Exemple avec plusieurs utilisateurs dans un tableau de type commun
        UserModel[] users = new UserModel[2];
        users[0] = u1;
        users[1] = u2;

        for (int i = 0; i < users.length; i++) {
            users[i].display();
        }
    }
}
```

Explications

* `UserModel u1 = new AdminUserModel(...)` : variable de type parent qui référence un objet enfant. C’est le point clé du polymorphisme.
* `u1.display()` : même si la variable est de type `UserModel`, Java exécute la méthode correspondant à l’objet réel (ici `AdminUserModel`).
* Même chose pour `u2` qui est un `ClientUserModel`.
* `UserModel[] users` : un tableau du type commun permet de stocker plusieurs types d’utilisateurs ensemble et d’appeler la même méthode sur chacun.

---

### Points clés à retenir

* Le polymorphisme permet de manipuler plusieurs types d’objets via un type commun (`UserModel`)
* On appelle la même méthode (`display()`) mais le comportement exécuté dépend de l’objet réel (admin, client)
* `@Override` et `super.display()` permettent d’adapter le comportement d’une classe enfant tout en réutilisant le comportement commun

---

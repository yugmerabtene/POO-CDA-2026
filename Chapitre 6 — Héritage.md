## Chapitre 6 — Héritage

### Objectifs

* Comprendre l’héritage : créer une classe enfant à partir d’une classe parent
* Comprendre la relation “est-un”
* Réutiliser du code commun sans duplication
* Continuer la gestion d’utilisateurs avec des types d’utilisateurs

---

### Structure de projet utilisée

```text id="ch6_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
    AdminUserModel.java
    ClientUserModel.java
```

Explications

* `model` contient les classes représentant les utilisateurs.
* `app` contient le point d’entrée pour tester et exécuter.

---

### 1) Classe parent : `UserModel`

```java id="ch6_user_model"
package model;

public class UserModel {

    // Champs communs à tous les utilisateurs
    private long id;
    private String name;
    private String email;

    // Constructeur : initialise les informations communes
    public UserModel(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Accès en lecture
    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    // Affichage simple des informations communes
    public void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `package model;` place la classe dans le dossier `model`.
* `UserModel` contient le commun : `id`, `name`, `email`.
* `private` empêche l’accès direct aux champs depuis l’extérieur.
* Le constructeur initialise l’objet dès sa création.
* Les classes enfants hériteront des méthodes publiques (`get...`, `display`).

---

### 2) Classe enfant : `AdminUserModel`

```java id="ch6_admin_user_model"
package model;

public class AdminUserModel extends UserModel {

    // Champ spécifique aux administrateurs
    private int accessLevel;

    // Constructeur enfant : initialise d'abord la partie parent avec super(...)
    public AdminUserModel(long id, String name, String email, int accessLevel) {
        super(id, name, email);
        this.accessLevel = accessLevel;
    }

    public int getAccessLevel() {
        return accessLevel;
    }

    // Affichage spécifique admin
    public void displayAdminInfo() {
        System.out.println("admin accessLevel=" + accessLevel);
    }
}
```

Explications

* `extends UserModel` : `AdminUserModel` hérite de `UserModel`.
* `super(id, name, email)` appelle le constructeur du parent pour initialiser la partie commune.
* `accessLevel` n’existe que pour les administrateurs.
* Un admin possède aussi `display()` grâce à l’héritage.

---

### 3) Classe enfant : `ClientUserModel`

```java id="ch6_client_user_model"
package model;

public class ClientUserModel extends UserModel {

    // Champ spécifique aux clients
    private String customerId;

    // Constructeur enfant : initialise d'abord la partie parent avec super(...)
    public ClientUserModel(long id, String name, String email, String customerId) {
        super(id, name, email);
        this.customerId = customerId;
    }

    public String getCustomerId() {
        return customerId;
    }

    // Affichage spécifique client
    public void displayClientInfo() {
        System.out.println("client customerId=" + customerId);
    }
}
```

Explications

* `ClientUserModel` réutilise tout ce qui est commun via `UserModel`.
* `customerId` est une donnée spécifique aux clients.
* Le code commun n’est pas dupliqué : il reste dans la classe parent.

---

### 4) Programme de test : `Main`

```java id="ch6_main"
package app;

import model.AdminUserModel;
import model.ClientUserModel;

public class Main {
    public static void main(String[] args) {

        // Création d'un admin et d'un client
        AdminUserModel admin = new AdminUserModel(1, "Admin", "admin@example.com", 10);
        ClientUserModel client = new ClientUserModel(2, "Client", "client@example.com", "C-100");

        // Méthode héritée (définie dans UserModel)
        admin.display();
        client.display();

        // Méthodes spécifiques
        admin.displayAdminInfo();
        client.displayClientInfo();
    }
}
```

Explications

* `package app;` place le point d’entrée dans `app`.
* `import ...` est nécessaire car `Main` n’est pas dans le même package que les modèles.
* `admin.display()` et `client.display()` fonctionnent car `display()` est hérité de `UserModel`.
* Les méthodes spécifiques (`displayAdminInfo`, `displayClientInfo`) existent uniquement sur leurs types respectifs.

---

### Points clés à retenir

* L’héritage sert à regrouper le commun dans une classe parent
* Les classes enfants ajoutent leurs spécificités
* `extends` crée la relation “est-un”
* `super(...)` initialise la partie héritée sans duplication de code

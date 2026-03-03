## Chapitre 2 — Attributs, méthodes et constructeurs (Java)

### Objectifs

* Comprendre le rôle des attributs et des méthodes dans une classe
* Initialiser un objet correctement dès sa création avec un constructeur
* Faire évoluer la gestion d’utilisateurs avec des actions simples (afficher, mettre à jour)

---

### 1) Attributs (état de l’objet)

```java id="ch2_user_fields"
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

* Les attributs (champs) représentent l’état de l’objet.
* Chaque instance de `User` possède ses propres valeurs pour `id`, `name`, `email`.
* À ce stade, ces champs peuvent encore être modifiés librement. On améliorera ça plus tard.

---

### 2) Méthodes (comportement de l’objet)

On ajoute des méthodes simples pour “faire quelque chose” avec un utilisateur : afficher ses informations et mettre à jour son email.

```java id="ch2_user_methods"
public class User {
    long id;
    String name;
    String email;

    // Affiche les informations de l'utilisateur
    void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }

    // Met à jour l'email de l'utilisateur
    void updateEmail(String newEmail) {
        email = newEmail;
    }
}
```

Explications

* Une méthode est une action liée à l’objet.
* `display()` utilise les champs de l’objet courant (celui sur lequel on appelle la méthode).
* `updateEmail(...)` modifie l’état de l’objet en changeant `email`.
* L’avantage : on commence à centraliser des actions dans la classe, au lieu de tout faire dans `main`.

---

### 3) Constructeur (initialisation dès la création)

Problème fréquent sans constructeur : on peut créer un utilisateur “vide” puis oublier de remplir un champ, ce qui donne un objet incohérent.

Le constructeur permet d’imposer une initialisation dès `new User(...)`.

```java id="ch2_user_constructor"
public class User {
    long id;
    String name;
    String email;

    // Constructeur : s'exécute automatiquement à chaque new User(...)
    User(long id, String name, String email) {
        this.id = id;       // this.id = champ de l'objet / id = paramètre
        this.name = name;
        this.email = email;
    }

    void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }

    void updateEmail(String newEmail) {
        email = newEmail;
    }
}
```

Explications

* Un constructeur porte le même nom que la classe et n’a pas de type de retour.
* Il s’exécute automatiquement à la création : `new User(...)`.
* `this` désigne l’objet courant.

  * `this.id` = le champ de l’objet
  * `id` = le paramètre reçu
* Grâce au constructeur, on crée des utilisateurs déjà “prêts”.

---

### 4) Utilisation dans le programme (gestion d’utilisateurs)

```java id="ch2_main_usage"
public class Main {
    public static void main(String[] args) {

        // Création d'utilisateurs avec le constructeur (objets initialisés immédiatement)
        User admin = new User(1, "Admin", "admin@example.com");
        User client = new User(2, "Client", "client@example.com");

        // Affichage via une méthode
        admin.display();
        client.display();

        // Mise à jour via une méthode
        client.updateEmail("client.new@example.com");

        // Vérification
        client.display();
    }
}
```

Explications

* On n’a plus besoin de créer `new User()` puis remplir champ par champ : le constructeur le fait directement.
* Les actions sont appelées sur l’objet : `client.display()` / `client.updateEmail(...)`.
* `updateEmail` modifie l’état de l’objet `client`, pas celui de `admin`.

---

### Points clés à retenir

* Les attributs représentent l’état d’un objet
* Les méthodes représentent les actions possibles sur l’objet
* Le constructeur garantit une initialisation dès la création

---

### Exercice

1. Ajouter une méthode `updateName(String newName)` dans `User`
2. Dans `main`, créer un utilisateur `guest` avec le constructeur
3. Afficher `guest`, modifier son nom, puis réafficher `guest`

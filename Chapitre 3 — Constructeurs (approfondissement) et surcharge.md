## Chapitre 3 — Constructeurs (approfondissement) et surcharge

### Objectifs

* Comprendre précisément le rôle d’un constructeur
* Savoir utiliser `this` correctement
* Introduire la surcharge de constructeurs pour créer un utilisateur de plusieurs façons
* Continuer la gestion d’utilisateurs progressivement

---

### 1) Pourquoi un constructeur est important

Sans constructeur, on peut créer un utilisateur “vide” et oublier de remplir un champ.
Avec un constructeur, on force une initialisation minimale dès la création de l’objet.

---

### 2) Constructeur “complet” (initialisation directe)

```java id="ch3_user_full_constructor"
public class User {
    long id;
    String name;
    String email;

    // Constructeur complet : on fournit toutes les informations
    User(long id, String name, String email) {
        // this.id correspond au champ de l'objet
        // id correspond au paramètre reçu
        this.id = id;
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

* Un constructeur s’exécute automatiquement quand on fait `new User(...)`.
* Il permet de garantir un objet “prêt” immédiatement.
* `this` sert à désigner les champs de l’objet courant quand ils ont le même nom que les paramètres.

---

### 3) Surcharge de constructeurs (plusieurs façons de créer un User)

Parfois, on ne connaît pas toutes les informations au moment de la création.
Exemple : on connaît le nom et l’email, mais l’id sera attribué plus tard.

On va donc créer un second constructeur.

```java id="ch3_user_overloaded_constructors"
public class User {
    long id;
    String name;
    String email;

    // Constructeur complet
    User(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Constructeur surchargé : id inconnu au départ
    User(String name, String email) {
        // On initialise id à 0 pour signifier "pas encore attribué"
        this.id = 0;
        this.name = name;
        this.email = email;
    }

    void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* La surcharge signifie : même nom `User(...)` mais paramètres différents.
* Java choisit le constructeur selon les paramètres fournis.
* Ici, `User("Alice", "alice@example.com")` appelle le constructeur à 2 paramètres.
* On choisit une convention simple : `id = 0` signifie “id non attribué”. On améliorera plus tard.

---

### 4) Utiliser `this(...)` pour éviter la duplication

Quand deux constructeurs font presque la même chose, on évite de répéter le code.
On peut appeler un constructeur depuis un autre avec `this(...)`.

```java id="ch3_user_this_constructor"
public class User {
    long id;
    String name;
    String email;

    // Constructeur complet
    User(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Constructeur surchargé : appelle le constructeur complet
    User(String name, String email) {
        this(0, name, email); // appelle User(long, String, String)
    }

    void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `this(0, name, email)` appelle un autre constructeur de la même classe.
* Ça évite les erreurs et rend le code plus propre.
* Règle Java : l’appel à `this(...)` doit être la première ligne du constructeur.

---

### 5) Programme de test (gestion d’utilisateurs)

```java id="ch3_main_usage"
public class Main {
    public static void main(String[] args) {

        // Création avec id connu
        User admin = new User(1, "Admin", "admin@example.com");

        // Création avec id inconnu
        User guest = new User("Guest", "guest@example.com");

        admin.display();
        guest.display();
    }
}
```

Explications

* Deux façons de créer un `User`, selon ce qu’on sait au moment de l’instanciation.
* Le programme reste simple, mais on commence à concevoir une gestion d’utilisateurs réaliste.

---

### Points clés à retenir

* Le constructeur initialise l’objet dès `new`
* `this` permet de distinguer champ et paramètre
* La surcharge permet plusieurs façons de créer un objet
* `this(...)` évite de dupliquer le code entre constructeurs

---

### Exercice

1. Ajouter un troisième constructeur `User(String name)` qui met :

   * `id = 0`
   * `email = "unknown@example.com"`
2. Créer dans `main` un utilisateur avec ce constructeur et l’afficher

## Chapitre 4 — Encapsulation

### Objectifs

* Comprendre pourquoi on cache les champs d’un objet
* Empêcher les modifications incohérentes depuis l’extérieur
* Donner des méthodes simples pour lire et modifier l’utilisateur de façon contrôlée

---

### 1) Problème sans encapsulation

Jusqu’ici, les champs sont accessibles directement :

* n’importe quel code peut faire `user.email = "nimportequoi"`
* n’importe quel code peut faire `user.id = -5`
* l’objet peut finir dans un état incohérent

L’encapsulation consiste à :

* rendre les champs **privés**
* obliger le reste du programme à passer par des **méthodes** pour lire ou modifier

---

### 2) Encapsulation : champs `private` et méthodes d’accès

```java id="ch4_user_private_fields"
public class User {
    // Champs privés : accessibles uniquement à l'intérieur de la classe
    private long id;
    private String name;
    private String email;

    // Constructeur : initialise l'objet correctement dès la création
    public User(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    // Méthodes de lecture (getters) : permettent d'accéder aux valeurs
    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    // Méthode de modification contrôlée : on ne change pas email directement
    public void updateEmail(String newEmail) {
        email = newEmail;
    }

    public void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `private` : empêche l’accès direct depuis `Main` ou toute autre classe.
* Les “getters” (`getId`, `getName`, `getEmail`) donnent un accès en lecture.
* `updateEmail` est une méthode prévue pour modifier l’email : c’est la classe `User` qui contrôle comment ça se fait.
* À partir de maintenant, on ne fait plus `user.email = ...` depuis l’extérieur.

---

### 3) Exemple d’utilisation (ce qui change dans `Main`)

```java id="ch4_main_usage"
public class Main {
    public static void main(String[] args) {

        User user = new User(1, "Alice", "alice@example.com");

        // Lecture via méthodes
        System.out.println(user.getEmail());

        // Modification via méthode dédiée
        user.updateEmail("alice.new@example.com");

        user.display();

        // Interdit : ne compile pas car email est private
        // user.email = "hack@example.com";
    }
}
```

Explications

* On accède aux données via les getters : `user.getEmail()`
* On modifie via une méthode : `user.updateEmail(...)`
* Le commentaire montre l’intérêt : l’accès direct est bloqué, donc on évite des modifications sauvages.

---

### 4) Encapsulation et cohérence (première règle simple)

Maintenant qu’on a une méthode `updateEmail`, on peut commencer à mettre des règles.

Exemple de règle simple : refuser un email vide.

```java id="ch4_user_email_rule"
public class User {
    private long id;
    private String name;
    private String email;

    public User(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    public String getEmail() {
        return email;
    }

    public void updateEmail(String newEmail) {
        if (newEmail == null || newEmail.isEmpty()) {
            return; // on refuse la modification
        }
        email = newEmail;
    }

    public void display() {
        System.out.println("id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* On contrôle maintenant les changements de `email`.
* Même si un autre développeur fait n’importe quoi dans `Main`, il ne peut pas forcer `email` à une valeur vide.
* L’objet reste plus cohérent.

---

### Points clés à retenir

* Encapsulation = champs privés + accès contrôlé via méthodes
* L’extérieur ne doit plus modifier l’état directement
* Ça permet d’ajouter des règles au bon endroit : dans la classe

---

### Exercice

1. Rendre `name` et `email` privés (si pas déjà fait)
2. Ajouter une méthode `updateName(String newName)` qui refuse `null` ou vide
3. Dans `main`, tester avec un nom vide puis un nom correct, et afficher le résultat

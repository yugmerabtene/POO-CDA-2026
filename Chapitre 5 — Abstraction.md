## Chapitre 5 — Abstraction

### Objectifs

* Comprendre l’idée d’abstraction : séparer le “quoi” du “comment”
* Utiliser une interface pour définir un contrat simple
* Continuer la gestion d’utilisateurs en préparant le futur système d’authentification
* Comprendre une notion indispensable qui apparaît ici : utiliser un type déjà créé (`User`) comme type d’une variable ou d’un paramètre

---

### 0) Notion indispensable avant de commencer : utiliser une classe comme type

Jusqu’ici, on a vu que `User` est une **classe** et qu’on peut créer des objets avec `new User(...)`.

Ici, on fait une étape de plus : on utilise `User` comme **type** dans une méthode, par exemple :

* `void save(User user)`

Ça veut dire :

* la méthode `save` reçoit **un objet** de type `User`
* la variable `user` est une variable qui référence un objet `User`

Même logique que :

* `long id` (variable de type `long`)
* `String email` (variable de type `String`)
* `User user` (variable de type `User`)

`User` est un type, exactement comme `String`, sauf que c’est un type que nous avons créé.

---

### 1) Contrat abstrait : `UserRepository`

```java id="ch5_user_repository_interface"
public interface UserRepository {

    // Cette méthode reçoit un objet User et le sauvegarde.
    // "void" signifie : la méthode ne renvoie aucune valeur.
    void save(User user);

    // Cette méthode reçoit un email (String) et renvoie un User si trouvé.
    // Si non trouvé, on renvoie null (pour l'instant).
    User findByEmail(String email);
}
```

Explications

* `interface` : on définit un contrat, pas une implémentation.
* `void` : nouveauté importante à rappeler : “ne renvoie rien”.
* `User user` : nouveauté importante : `User` est un type créé par nous.

  * `user` est juste le nom de la variable (le paramètre).
  * La méthode attend donc “un utilisateur” en entrée.
* `User findByEmail(...)` : la méthode renvoie un `User` (un objet). Si elle ne trouve pas, elle renvoie `null` pour l’instant.

---

### 2) Implémentation simple en mémoire : `InMemoryUserRepository`

```java id="ch5_in_memory_user_repository"
public class InMemoryUserRepository implements UserRepository {

    // Tableau d'objets User (plusieurs utilisateurs)
    private User[] users = new User[10];

    // Nombre d'utilisateurs réellement stockés dans le tableau
    private int count = 0;

    @Override
    public void save(User user) {
        // On cherche si un utilisateur avec le même email existe déjà
        int index = indexOfEmail(user.getEmail());

        // Si trouvé, on remplace l'utilisateur (mise à jour)
        if (index != -1) {
            users[index] = user;
            return;
        }

        // Sinon, on ajoute un nouvel utilisateur
        users[count] = user;
        count++;
    }

    @Override
    public User findByEmail(String email) {
        // Recherche de l'utilisateur par email
        int index = indexOfEmail(email);

        // Si non trouvé, on renvoie null
        if (index == -1) {
            return null;
        }

        // Sinon, on renvoie l'utilisateur trouvé
        return users[index];
    }

    // Méthode interne pour trouver l'indice d'un utilisateur à partir de son email
    private int indexOfEmail(String email) {
        for (int i = 0; i < count; i++) {
            if (users[i].getEmail().equals(email)) {
                return i;
            }
        }
        return -1;
    }
}
```

Explications

* `implements UserRepository` : la classe respecte le contrat, donc elle doit écrire `save` et `findByEmail`.
* `User[] users` : tableau d’objets `User`. C’est la première fois qu’on manipule “plusieurs utilisateurs” au même endroit.
* `int count` : permet de savoir combien d’utilisateurs on a vraiment stockés.
* `@Override` : indique que les méthodes correspondent bien au contrat de l’interface.
* `return;` dans `save` : quitte la méthode immédiatement (utile ici après remplacement).
* `return null;` dans `findByEmail` : signifie “pas trouvé” (on remplacera plus tard par une solution plus propre).

---

### 3) Service métier basé sur l’abstraction : `UserService`

```java id="ch5_user_service"
public class UserService {

    // Le service ne dépend pas d'une classe précise.
    // Il dépend du contrat UserRepository.
    private UserRepository repo;

    public UserService(UserRepository repo) {
        this.repo = repo;
    }

    // Inscription : on demande au repository de sauvegarder l'utilisateur
    public void register(User user) {
        repo.save(user);
    }

    // Recherche : on demande au repository de trouver l'utilisateur
    public User getByEmail(String email) {
        return repo.findByEmail(email);
    }
}
```

Explications

* `private UserRepository repo;` : le service connaît seulement le contrat, pas la manière de stocker.
* `register(User user)` : nouvelle apparition de `User` comme type de paramètre : la méthode attend un objet `User`.
* Le service appelle `repo.save(...)` et `repo.findByEmail(...)` sans savoir si derrière c’est un tableau, une base SQL, etc.

---

### 4) Programme de test

```java id="ch5_main"
public class Main {
    public static void main(String[] args) {

        // On choisit une implémentation concrète du contrat
        UserRepository repo = new InMemoryUserRepository();

        // Le service reçoit le contrat
        UserService service = new UserService(repo);

        // Création d'objets User
        User alice = new User(1, "Alice", "alice@example.com");
        User bob = new User(2, "Bob", "bob@example.com");

        // Sauvegarde via le service
        service.register(alice);
        service.register(bob);

        // Recherche via le service
        User result = service.getByEmail("bob@example.com");

        // Gestion du cas non trouvé (null)
        if (result != null) {
            result.display();
        }
    }
}
```

Explications

* `UserRepository repo = ...` : on déclare une variable avec le type de l’interface (contrat).
* `new InMemoryUserRepository()` : on choisit une implémentation concrète.
* `User result = ...` : la méthode renvoie un objet `User` si trouvé, sinon `null`.

---

### Points clés à retenir

* Une classe créée (`User`) devient un type utilisable partout : variable, paramètre, retour de méthode
* Abstraction : le code métier dépend d’un contrat (interface), pas d’une technique
* Interface : définit ce qu’on veut faire (`save`, `findByEmail`)
* Implémentation : explique comment on le fait (ici un tableau en mémoire)

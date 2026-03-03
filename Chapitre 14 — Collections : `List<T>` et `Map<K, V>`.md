## Chapitre 14 — Collections : `List<T>` et `Map<K, V>`

### Objectifs

* Comprendre pourquoi on utilise des collections au lieu de tableaux fixes
* Utiliser `List<UserModel>` pour stocker des utilisateurs sans taille fixe
* Utiliser `Map<Long, UserModel>` pour retrouver rapidement un utilisateur par identifiant
* Garder le code simple, progressif, et complet pour exécuter le chapitre

---

### Structure de projet utilisée

```text id="ch14_tree"
src/
  app/
    Main.java
  model/
    RoleModel.java
    CredentialModel.java
    UserModel.java
  repository/
    UserRepository.java
    InMemoryUserRepository.java
```

Explications

* `model` : modèles (utilisateur, rôle, identifiants).
* `repository` : stockage en mémoire.
* `app` : démonstration.

---

## Code complet du chapitre

### `model/RoleModel.java`

```java id="ch14_role_model"
package model;

public enum RoleModel {
    ADMIN,
    CLIENT,
    GUEST
}
```

Explications

* Un `enum` représente une liste fermée de valeurs possibles.
* Ici, le rôle d’un utilisateur est obligatoirement l’une de ces valeurs.

---

### `model/CredentialModel.java`

```java id="ch14_credential_model"
package model;

public class CredentialModel {

    private String username;
    private String password;

    public CredentialModel(String username, String password) {
        this.username = username;
        this.password = password;
    }

    public String getUsername() {
        return username;
    }

    public String getPassword() {
        return password;
    }
}
```

Explications

* `CredentialModel` contient les identifiants nécessaires pour une connexion.
* On garde `password` en clair ici uniquement pour rester concentré sur les collections.

---

### `model/UserModel.java`

```java id="ch14_user_model"
package model;

public class UserModel {

    private long id;
    private String name;
    private String email;

    private RoleModel role;
    private CredentialModel credential;

    public UserModel(long id, String name, String email, RoleModel role, String username, String password) {
        this.id = id;
        this.name = name;
        this.email = email;

        this.role = role;

        // L'utilisateur crée ses identifiants : on garde cette relation comme avant
        this.credential = new CredentialModel(username, password);
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

    public RoleModel getRole() {
        return role;
    }

    public CredentialModel getCredential() {
        return credential;
    }
}
```

Explications

* `role` est un attribut de type `RoleModel`.
* `credential` est un attribut de type `CredentialModel`.
* Ce chapitre n’ajoute rien dans `UserModel` : on le garde stable pour se concentrer sur `List` et `Map`.

---

### `repository/UserRepository.java`

```java id="ch14_user_repository"
package repository;

import java.util.List;
import java.util.Optional;
import model.UserModel;

public interface UserRepository {

    // Enregistre ou met à jour un utilisateur
    void save(UserModel user);

    // Recherche un utilisateur par identifiant
    Optional<UserModel> findById(long id);

    // Retourne tous les utilisateurs
    List<UserModel> findAll();
}
```

Explications

* Nouveauté du chapitre : `List<UserModel>` et `Optional<UserModel>`.
* `List<UserModel>` signifie “liste d’utilisateurs” dont la taille peut changer.
* `Optional<UserModel>` signifie “utilisateur présent ou absent” sans `null`.

---

### `repository/InMemoryUserRepository.java`

```java id="ch14_in_memory_user_repository"
package repository;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import model.UserModel;

public class InMemoryUserRepository implements UserRepository {

    // List<UserModel> : stocke tous les utilisateurs, sans taille fixe
    private List<UserModel> users = new ArrayList<>();

    // Map<Long, UserModel> : clé = id, valeur = utilisateur
    private Map<Long, UserModel> usersById = new HashMap<>();

    @Override
    public void save(UserModel user) {
        if (user == null) {
            return;
        }

        long id = user.getId();

        // 1) On regarde si l'utilisateur existe déjà grâce à la Map
        boolean alreadyExists = usersById.containsKey(id);

        // 2) On met à jour la Map (ajout ou remplacement)
        usersById.put(id, user);

        // 3) Si c'est un nouvel utilisateur, on l'ajoute dans la List
        if (!alreadyExists) {
            users.add(user);
            return;
        }

        // 4) Si c'est une mise à jour, on remplace dans la List
        for (int i = 0; i < users.size(); i++) {
            if (users.get(i).getId() == id) {
                users.set(i, user);
                return;
            }
        }
    }

    @Override
    public Optional<UserModel> findById(long id) {
        // Map.get(id) renvoie l'utilisateur si présent, sinon null
        UserModel found = usersById.get(id);

        if (found == null) {
            return Optional.empty();
        }

        return Optional.of(found);
    }

    @Override
    public List<UserModel> findAll() {
        // On renvoie une copie pour éviter la modification directe des données internes
        return new ArrayList<>(users);
    }
}
```

Explications
Nouveautés introduites ici

* `List<UserModel> users = new ArrayList<>();`

  * `List` est une collection ordonnée.
  * `ArrayList` est une implémentation de `List`.
  * `users.add(user)` ajoute un élément.
  * `users.size()` donne le nombre d’éléments.
  * `users.get(i)` lit un élément à un index.
  * `users.set(i, user)` remplace un élément à un index.

* `Map<Long, UserModel> usersById = new HashMap<>();`

  * `Map` associe une clé à une valeur.
  * `HashMap` est une implémentation de `Map`.
  * `containsKey(id)` vérifie si une clé existe.
  * `put(id, user)` ajoute ou remplace la valeur pour une clé.
  * `get(id)` récupère la valeur associée à une clé (ou `null` si absent).

Pourquoi on garde `List` et `Map`

* La `Map` permet de retrouver vite un utilisateur par `id`.
* La `List` permet de récupérer tous les utilisateurs dans l’ordre d’ajout.

---

### `app/Main.java`

```java id="ch14_main"
package app;

import java.util.List;
import java.util.Optional;
import model.RoleModel;
import model.UserModel;
import repository.InMemoryUserRepository;
import repository.UserRepository;

public class Main {
    public static void main(String[] args) {

        UserRepository repo = new InMemoryUserRepository();

        // Ajout d'utilisateurs
        repo.save(new UserModel(1, "Alice", "alice@example.com", RoleModel.CLIENT, "alice", "pass1"));
        repo.save(new UserModel(2, "Bob", "bob@example.com", RoleModel.ADMIN, "bob", "pass2"));
        repo.save(new UserModel(3, "Guest", "guest@example.com", RoleModel.GUEST, "guest", "pass3"));

        // Lecture de tous les utilisateurs (List)
        List<UserModel> all = repo.findAll();
        System.out.println("Nombre d'utilisateurs=" + all.size());

        for (int i = 0; i < all.size(); i++) {
            UserModel user = all.get(i);
            System.out.println(" - " + user.getId() + " " + user.getEmail() + " role=" + user.getRole());
        }

        // Recherche par id (Map) via Optional
        Optional<UserModel> result = repo.findById(2);

        if (result.isPresent()) {
            UserModel user = result.get();
            System.out.println("Trouvé id=2 email=" + user.getEmail());
        }

        Optional<UserModel> absent = repo.findById(99);
        if (absent.isEmpty()) {
            System.out.println("Aucun utilisateur avec id=99");
        }
    }
}
```

Explications

* `List<UserModel> all = repo.findAll();` récupère tous les utilisateurs.
* `all.size()` donne le nombre d’utilisateurs.
* `all.get(i)` permet d’accéder à un utilisateur par index.
* `Optional<UserModel> result = repo.findById(2);` représente “présent ou absent”.
* `isPresent()` et `isEmpty()` permettent de tester sans utiliser `null`.

---

### Points clés à retenir

* `List<T>` sert à stocker une liste de taille variable
* `Map<K, V>` sert à retrouver rapidement une valeur à partir d’une clé
* `Optional<T>` permet de représenter l’absence sans `null`
* Dans ce chapitre :

  * `List` sert à parcourir tous les utilisateurs
  * `Map` sert à retrouver un utilisateur par identifiant

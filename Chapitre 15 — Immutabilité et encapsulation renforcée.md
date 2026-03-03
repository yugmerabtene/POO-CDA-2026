## Chapitre 15 — Immutabilité et encapsulation renforcée

### Objectifs

* Comprendre l’immutabilité : un objet dont l’état ne change pas après création
* Renforcer l’encapsulation : limiter les modifications depuis l’extérieur
* Appliquer ces idées à la gestion d’utilisateurs, sans ajouter trop de notions à la fois
* Garder la structure par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch15_tree"
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

* `model` : utilisateur, rôle, identifiants.
* `repository` : stockage en mémoire (List et Map).
* `app` : démonstration.

---

## Code complet du chapitre

### `model/RoleModel.java`

```java id="ch15_role_model"
package model;

public enum RoleModel {
    ADMIN,
    CLIENT,
    GUEST
}
```

Explications

* Rôles possibles pour un utilisateur.

---

### `model/CredentialModel.java`

```java id="ch15_credential_model"
package model;

public class CredentialModel {

    // Attributs rendus immuables avec final
    private final String username;
    private final String password;

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

* Nouveauté : `final` sur les attributs.

  * Cela signifie que ces attributs reçoivent une valeur une seule fois (dans le constructeur).
  * Après, on ne peut plus les modifier.
* `CredentialModel` devient immuable : une fois créé, il ne change plus.

---

### `model/UserModel.java`

```java id="ch15_user_model"
package model;

public class UserModel {

    // Attributs immuables
    private final long id;
    private final String name;
    private final String email;
    private final RoleModel role;
    private final CredentialModel credential;

    public UserModel(long id, String name, String email, RoleModel role, String username, String password) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.role = role;

        // L'utilisateur crée ses identifiants : on garde cette relation
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

* Nouveauté : `final` sur tous les attributs.

  * `UserModel` devient immuable : après création, on ne peut plus changer `email`, `role`, etc.
* Cela renforce l’encapsulation : personne ne peut modifier l’état via des setters, car il n’y en a pas.

---

### `repository/UserRepository.java`

```java id="ch15_user_repository"
package repository;

import java.util.List;
import java.util.Optional;
import model.UserModel;

public interface UserRepository {

    void save(UserModel user);

    Optional<UserModel> findById(long id);

    List<UserModel> findAll();
}
```

Explications

* Contrat identique : ce chapitre se concentre sur l’immutabilité des modèles.

---

### `repository/InMemoryUserRepository.java`

```java id="ch15_in_memory_user_repository"
package repository;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Optional;
import model.UserModel;

public class InMemoryUserRepository implements UserRepository {

    private final List<UserModel> users = new ArrayList<>();
    private final Map<Long, UserModel> usersById = new HashMap<>();

    @Override
    public void save(UserModel user) {
        if (user == null) {
            return;
        }

        long id = user.getId();
        boolean alreadyExists = usersById.containsKey(id);

        usersById.put(id, user);

        if (!alreadyExists) {
            users.add(user);
            return;
        }

        for (int i = 0; i < users.size(); i++) {
            if (users.get(i).getId() == id) {
                users.set(i, user);
                return;
            }
        }
    }

    @Override
    public Optional<UserModel> findById(long id) {
        UserModel found = usersById.get(id);
        if (found == null) {
            return Optional.empty();
        }
        return Optional.of(found);
    }

    @Override
    public List<UserModel> findAll() {
        return new ArrayList<>(users);
    }
}
```

Explications

* Nouveauté : `final` sur `users` et `usersById`.

  * Cela signifie que les références ne changent pas (on garde toujours la même liste et la même map).
  * On peut encore ajouter des éléments dans la liste, mais on ne peut pas réassigner `users` vers une autre liste.

---

### `app/Main.java`

```java id="ch15_main"
package app;

import java.util.List;
import model.RoleModel;
import model.UserModel;
import repository.InMemoryUserRepository;
import repository.UserRepository;

public class Main {
    public static void main(String[] args) {

        UserRepository repo = new InMemoryUserRepository();

        repo.save(new UserModel(1, "Alice", "alice@example.com", RoleModel.CLIENT, "alice", "pass1"));
        repo.save(new UserModel(2, "Bob", "bob@example.com", RoleModel.ADMIN, "bob", "pass2"));

        List<UserModel> all = repo.findAll();
        for (int i = 0; i < all.size(); i++) {
            UserModel user = all.get(i);

            System.out.println("id=" + user.getId() + " email=" + user.getEmail() + " role=" + user.getRole());
        }

        // Démonstration : on ne peut pas modifier un attribut final
        // Par exemple, ceci n'existe pas car il n'y a pas de méthode setEmail :
        // user.setEmail("new@example.com");
    }
}
```

Explications

* `UserModel` n’a pas de méthodes de modification (pas de setter).
* Avec `final`, l’état est fixé dans le constructeur : c’est une forme simple d’immutabilité.
* L’immutabilité rend le code plus prévisible : un utilisateur ne “change pas de forme” sans création d’un nouvel objet.

---

### Points clés à retenir

* `final` sur un attribut signifie : assigné une seule fois
* Une classe sans setters et avec des attributs `final` est immuable
* L’immutabilité renforce l’encapsulation et réduit les effets de bord

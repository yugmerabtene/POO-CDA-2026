## Chapitre 13 — Enum (rôles) et contrôle d’accès simple (RBAC)

### Objectifs

* Comprendre ce qu’est un `enum` en Java
* Modéliser des rôles d’utilisateurs (ADMIN, CLIENT, GUEST)
* Mettre en place un contrôle d’accès simple dans le service d’authentification
* Garder une structure claire par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch13_tree"
src/
  app/
    Main.java
  model/
    RoleModel.java
    CredentialModel.java
    UserModel.java
  repository/
    Repository.java
    InMemoryRepository.java
  service/
    AuthenticationFailedException.java
    AccessDeniedException.java
    AuthenticatorService.java
    PasswordAuthenticatorService.java
    AuthService.java
```

Explications

* `model` : rôle, utilisateur, identifiants.
* `repository` : stockage en mémoire.
* `service` : authentification et contrôle d’accès.
* `app` : démonstration.

---

## Code complet du chapitre

### `model/RoleModel.java`

```java id="ch13_role_model"
package model;

public enum RoleModel {
    ADMIN,
    CLIENT,
    GUEST
}
```

Explications

* `enum` définit un ensemble fermé de valeurs possibles.
* Ici, un utilisateur ne peut avoir que ces rôles, pas autre chose.
* Un `enum` remplace des chaînes du type `"ADMIN"` qui sont source d’erreurs de frappe.

---

### `model/CredentialModel.java`

```java id="ch13_credential_model"
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

* `CredentialModel` contient les identifiants utilisés pour l’authentification.

---

### `model/UserModel.java`

```java id="ch13_user_model"
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

* Nouveauté : `RoleModel role` est un attribut de type `enum`.
* L’utilisateur possède un rôle, utilisé ensuite pour autoriser ou refuser des actions.
* `credential` reste une composition : l’utilisateur contient ses identifiants.

---

### `repository/Repository.java`

```java id="ch13_repository"
package repository;

import java.util.Optional;

public interface Repository<T> {

    void save(T item);

    Optional<T> findById(long id);
}
```

Explications

* Contrat de stockage générique.

---

### `repository/InMemoryRepository.java`

```java id="ch13_in_memory_repository"
package repository;

import java.util.Optional;

public class InMemoryRepository<T> implements Repository<T> {

    private Object[] items = new Object[10];
    private int count = 0;

    @Override
    public void save(T item) {
        items[count] = item;
        count++;
    }

    @Override
    public Optional<T> findById(long id) {
        if (id < 0 || id >= count) {
            return Optional.empty();
        }
        T found = cast(items[(int) id]);
        return Optional.of(found);
    }

    private T cast(Object obj) {
        return (T) obj;
    }
}
```

Explications

* Stockage minimal en mémoire.

---

### `service/AuthenticationFailedException.java`

```java id="ch13_auth_failed_exception"
package service;

public class AuthenticationFailedException extends RuntimeException {

    public AuthenticationFailedException(String message) {
        super(message);
    }
}
```

Explications

* Exception métier : échec d’authentification.

---

### `service/AccessDeniedException.java`

```java id="ch13_access_denied_exception"
package service;

public class AccessDeniedException extends RuntimeException {

    public AccessDeniedException(String message) {
        super(message);
    }
}
```

Explications

* Exception métier : accès refusé, l’utilisateur est bien connecté mais n’a pas le rôle requis.

---

### `service/AuthenticatorService.java`

```java id="ch13_authenticator_service"
package service;

import model.UserModel;

public interface AuthenticatorService {

    boolean authenticate(UserModel user, String secret);
}
```

Explications

* Contrat d’authentification.

---

### `service/PasswordAuthenticatorService.java`

```java id="ch13_password_authenticator"
package service;

import model.UserModel;

public class PasswordAuthenticatorService implements AuthenticatorService {

    @Override
    public boolean authenticate(UserModel user, String secret) {
        if (user == null) {
            return false;
        }
        if (secret == null) {
            return false;
        }
        if (user.getCredential() == null) {
            return false;
        }
        return user.getCredential().getPassword().equals(secret);
    }
}
```

Explications

* Vérification simple du secret.

---

### `service/AuthService.java`

```java id="ch13_auth_service"
package service;

import java.util.Optional;
import model.RoleModel;
import model.UserModel;
import repository.Repository;

public class AuthService {

    private Repository<UserModel> userRepo;
    private AuthenticatorService authenticator;

    public AuthService(Repository<UserModel> userRepo, AuthenticatorService authenticator) {
        this.userRepo = userRepo;
        this.authenticator = authenticator;
    }

    public UserModel login(long userId, String secret) {
        Optional<UserModel> found = userRepo.findById(userId);
        if (found.isEmpty()) {
            throw new AuthenticationFailedException("Utilisateur introuvable");
        }

        UserModel user = found.get();
        boolean ok = authenticator.authenticate(user, secret);
        if (!ok) {
            throw new AuthenticationFailedException("Secret invalide");
        }

        return user;
    }

    // Contrôle d'accès simple : seuls les admins peuvent appeler cette méthode
    public void adminAction(UserModel user) {
        if (user == null) {
            throw new AccessDeniedException("Accès refusé : utilisateur manquant");
        }

        if (user.getRole() != RoleModel.ADMIN) {
            throw new AccessDeniedException("Accès refusé : rôle requis ADMIN");
        }
    }
}
```

Explications

* Nouveauté : contrôle d’accès par rôle (RBAC simplifié).
* `user.getRole() != RoleModel.ADMIN` : comparaison d’un enum.
* `AccessDeniedException` est différente de `AuthenticationFailedException` :

  * authentification = qui tu es (connexion)
  * autorisation = ce que tu as le droit de faire (rôle)

---

### `app/Main.java`

```java id="ch13_main"
package app;

import model.RoleModel;
import model.UserModel;
import repository.InMemoryRepository;
import repository.Repository;
import service.AccessDeniedException;
import service.AuthService;
import service.AuthenticationFailedException;
import service.PasswordAuthenticatorService;

public class Main {
    public static void main(String[] args) {

        Repository<UserModel> repo = new InMemoryRepository<>();
        AuthService authService = new AuthService(repo, new PasswordAuthenticatorService());

        // id = index pour cette démonstration
        repo.save(new UserModel(0, "Alice", "alice@example.com", RoleModel.CLIENT, "alice", "pass1"));
        repo.save(new UserModel(1, "Bob", "bob@example.com", RoleModel.ADMIN, "bob", "pass2"));

        // Connexion admin
        try {
            UserModel admin = authService.login(1, "pass2");
            System.out.println("Connecté : " + admin.getEmail() + " role=" + admin.getRole());

            authService.adminAction(admin);
            System.out.println("Action admin autorisée");
        } catch (AuthenticationFailedException ex) {
            System.out.println("Connexion refusée : " + ex.getMessage());
        } catch (AccessDeniedException ex) {
            System.out.println("Accès refusé : " + ex.getMessage());
        }

        // Connexion client (puis tentative action admin)
        try {
            UserModel client = authService.login(0, "pass1");
            System.out.println("Connecté : " + client.getEmail() + " role=" + client.getRole());

            authService.adminAction(client);
            System.out.println("Action admin autorisée");
        } catch (AuthenticationFailedException ex) {
            System.out.println("Connexion refusée : " + ex.getMessage());
        } catch (AccessDeniedException ex) {
            System.out.println("Accès refusé : " + ex.getMessage());
        }
    }
}
```

Explications

* Premier scénario : un admin se connecte puis passe le contrôle d’accès.
* Deuxième scénario : un client se connecte, mais se fait refuser l’action admin.
* `try/catch` différencie bien “erreur de connexion” et “accès refusé”.

---

### Points clés à retenir

* `enum` fournit des valeurs contrôlées et évite les erreurs de chaînes
* RBAC simplifié : on autorise une action en vérifiant le rôle
* Authentification et autorisation sont deux problèmes différents
* Exceptions métier rendent les erreurs compréhensibles et gérables

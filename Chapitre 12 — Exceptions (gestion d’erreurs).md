## Chapitre 12 — Exceptions (gestion d’erreurs)

### Objectifs

* Comprendre le rôle des exceptions : signaler une erreur proprement
* Distinguer une erreur métier et une erreur technique
* Créer une exception personnalisée pour l’authentification
* Utiliser `try` / `catch` pour gérer une erreur sans faire planter l’application
* Garder une structure claire par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch12_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
    CredentialModel.java
  repository/
    Repository.java
    InMemoryRepository.java
  service/
    AuthenticationFailedException.java
    AuthenticatorService.java
    PasswordAuthenticatorService.java
    AuthService.java
```

Explications

* `model` : utilisateur et identifiants.
* `repository` : stockage générique en mémoire.
* `service` : logique d’authentification et exceptions.
* `app` : exécution et démonstration.

---

## Code complet du chapitre

### `model/CredentialModel.java`

```java id="ch12_credential_model"
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

* `CredentialModel` regroupe les informations utilisées pour l’authentification.
* On garde `password` en clair uniquement pour illustrer les exceptions. La sécurisation viendra plus tard.

---

### `model/UserModel.java`

```java id="ch12_user_model"
package model;

public class UserModel {

    private long id;
    private String name;
    private String email;

    // Composition : l'utilisateur contient ses identifiants
    private CredentialModel credential;

    public UserModel(long id, String name, String email, String username, String password) {
        this.id = id;
        this.name = name;
        this.email = email;
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

    public CredentialModel getCredential() {
        return credential;
    }

    public void display() {
        System.out.println("USER id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `credential` est un attribut de type `CredentialModel`.
* L’utilisateur crée ses identifiants dans le constructeur : c’est cohérent avec la composition.

---

### `repository/Repository.java`

```java id="ch12_repository"
package repository;

import java.util.Optional;

public interface Repository<T> {

    void save(T item);

    Optional<T> findById(long id);
}
```

Explications

* Interface générique de stockage.
* `Optional<T>` représente un résultat présent ou absent sans utiliser `null`.
* Ici, on ne met pas de contrainte `extends ...` pour rester concentré sur les exceptions.

---

### `repository/InMemoryRepository.java`

```java id="ch12_in_memory_repository"
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
        // Démonstration simple : on considère que l'id correspond à l'index (id = 0..)
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

* Stockage minimal en mémoire pour éviter de rajouter trop de notions.
* `findById` retourne `Optional.empty()` si l’id est hors limites.

---

### `service/AuthenticationFailedException.java`

```java id="ch12_auth_failed_exception"
package service;

public class AuthenticationFailedException extends RuntimeException {

    public AuthenticationFailedException(String message) {
        super(message);
    }
}
```

Explications

* Une exception personnalisée représente une erreur métier : “authentification refusée”.
* `extends RuntimeException` :

  * ce type d’exception n’oblige pas à écrire `throws` partout
  * on peut la lancer quand une règle métier est violée
* `super(message)` stocke un message lisible pour expliquer la cause.

---

### `service/AuthenticatorService.java`

```java id="ch12_authenticator_service"
package service;

import model.UserModel;

public interface AuthenticatorService {

    // Contrat : authentifier l'utilisateur avec un secret (mot de passe)
    boolean authenticate(UserModel user, String secret);
}
```

Explications

* Contrat minimal pour séparer l’authentification du reste.
* Ici on retourne `boolean` pour garder la logique simple dans ce chapitre.

---

### `service/PasswordAuthenticatorService.java`

```java id="ch12_password_authenticator"
package service;

import model.CredentialModel;
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

        CredentialModel credential = user.getCredential();
        if (credential == null) {
            return false;
        }

        // Vérification simple du mot de passe
        return credential.getPassword().equals(secret);
    }
}
```

Explications

* Cette classe contient la règle de vérification du mot de passe.
* On utilise `CredentialModel` via `user.getCredential()` pour lire le mot de passe attendu.

---

### `service/AuthService.java`

```java id="ch12_auth_service"
package service;

import java.util.Optional;
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
        // Recherche de l'utilisateur
        Optional<UserModel> found = userRepo.findById(userId);

        // Si absent, erreur métier
        if (found.isEmpty()) {
            throw new AuthenticationFailedException("Utilisateur introuvable");
        }

        UserModel user = found.get();

        // Vérification du secret
        boolean ok = authenticator.authenticate(user, secret);

        // Si échec, erreur métier
        if (!ok) {
            throw new AuthenticationFailedException("Secret invalide");
        }

        // Succès : on renvoie l'utilisateur authentifié
        return user;
    }
}
```

Explications

* `throw new AuthenticationFailedException(...)` :

  * on signale une erreur métier claire
  * on arrête le flux normal de la méthode
* `Optional<UserModel>` :

  * `isEmpty()` évite le `null`
  * `get()` récupère l’utilisateur seulement si présent
* Cette méthode `login` renvoie un `UserModel` en cas de succès, sinon elle lance une exception.

---

### `app/Main.java`

```java id="ch12_main"
package app;

import model.UserModel;
import repository.InMemoryRepository;
import repository.Repository;
import service.AuthService;
import service.AuthenticationFailedException;
import service.PasswordAuthenticatorService;

public class Main {
    public static void main(String[] args) {

        // Dépendances
        Repository<UserModel> repo = new InMemoryRepository<>();
        PasswordAuthenticatorService authenticator = new PasswordAuthenticatorService();
        AuthService authService = new AuthService(repo, authenticator);

        // Ajout d'utilisateurs dans le repository
        repo.save(new UserModel(0, "Alice", "alice@example.com", "alice", "pass1"));
        repo.save(new UserModel(1, "Bob", "bob@example.com", "bob", "pass2"));

        // Cas 1 : succès
        try {
            UserModel user = authService.login(1, "pass2");
            System.out.println("Connexion réussie : " + user.getEmail());
        } catch (AuthenticationFailedException ex) {
            System.out.println("Connexion refusée : " + ex.getMessage());
        }

        // Cas 2 : échec (mot de passe incorrect)
        try {
            UserModel user = authService.login(1, "wrong");
            System.out.println("Connexion réussie : " + user.getEmail());
        } catch (AuthenticationFailedException ex) {
            System.out.println("Connexion refusée : " + ex.getMessage());
        }

        // Cas 3 : échec (utilisateur introuvable)
        try {
            UserModel user = authService.login(99, "pass");
            System.out.println("Connexion réussie : " + user.getEmail());
        } catch (AuthenticationFailedException ex) {
            System.out.println("Connexion refusée : " + ex.getMessage());
        }
    }
}
```

Explications

* `try { ... } catch (AuthenticationFailedException ex) { ... }` :

  * `try` contient le code qui peut déclencher une exception
  * `catch` récupère l’exception et évite que l’application s’arrête
* `ex.getMessage()` récupère le message fourni au moment du `throw`.
* On teste 3 cas : succès, secret invalide, utilisateur introuvable.

---

### Points clés à retenir

* Une exception sert à signaler une erreur et interrompre le flux normal
* Une exception personnalisée rend l’erreur plus lisible et plus métier
* `throw` lance une exception
* `try/catch` permet de gérer l’erreur proprement sans faire planter l’application

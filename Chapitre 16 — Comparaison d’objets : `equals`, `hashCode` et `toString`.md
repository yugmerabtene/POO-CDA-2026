## Chapitre 16 — Comparaison d’objets : `equals`, `hashCode` et `toString`

### Objectifs

* Comprendre la différence entre comparaison de références et comparaison de contenu
* Implémenter `equals` pour comparer deux utilisateurs par contenu
* Implémenter `hashCode` pour rester cohérent avec `equals`
* Implémenter `toString` pour afficher un objet proprement
* Garder la structure par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch16_tree"
src/
  app/
    Main.java
  model/
    RoleModel.java
    CredentialModel.java
    UserModel.java
```

Explications

* Ce chapitre se concentre sur la comparaison et l’affichage des objets, donc pas besoin de repository ici.
* `app` : démonstration.
* `model` : modèles.

---

## Code complet du chapitre

### `model/RoleModel.java`

```java id="ch16_role_model"
package model;

public enum RoleModel {
    ADMIN,
    CLIENT,
    GUEST
}
```

Explications

* Rôles possibles.

---

### `model/CredentialModel.java`

```java id="ch16_credential_model"
package model;

public class CredentialModel {

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

* Identifiants simples (toujours en clair pour le cours, la sécurité viendra après).

---

### `model/UserModel.java`

```java id="ch16_user_model"
package model;

import java.util.Objects;

public class UserModel {

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

    @Override
    public String toString() {
        return "UserModel{id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", role=" + role +
                ", username='" + credential.getUsername() + '\'' +
                '}';
    }

    @Override
    public boolean equals(Object obj) {
        // 1) Même référence : c'est forcément égal
        if (this == obj) {
            return true;
        }

        // 2) Si obj est null, ce n'est pas égal
        if (obj == null) {
            return false;
        }

        // 3) Si ce n'est pas le même type, ce n'est pas égal
        if (getClass() != obj.getClass()) {
            return false;
        }

        // 4) Conversion vers le bon type pour comparer le contenu
        UserModel other = (UserModel) obj;

        // 5) Comparaison du contenu
        return id == other.id
                && Objects.equals(email, other.email);
    }

    @Override
    public int hashCode() {
        // Doit être cohérent avec equals : mêmes champs utilisés
        return Objects.hash(id, email);
    }
}
```

Explications

* `toString()` :

  * sert à obtenir un affichage lisible d’un objet.
  * ici, on affiche l’id, le nom, l’email, le rôle et le username.
* `equals(Object obj)` :

  * compare le contenu, pas juste la référence.
  * ici, on décide qu’un utilisateur est “égal” à un autre si `id` et `email` sont identiques.
* `hashCode()` :

  * doit utiliser les mêmes éléments que `equals`.
  * nécessaire si l’objet est utilisé dans des structures comme `HashMap` ou `HashSet`.
* `Objects.equals(...)` évite une erreur si `email` est `null`.

---

### `app/Main.java`

```java id="ch16_main"
package app;

import model.RoleModel;
import model.UserModel;

public class Main {
    public static void main(String[] args) {

        UserModel u1 = new UserModel(1, "Alice", "alice@example.com", RoleModel.CLIENT, "alice", "pass1");
        UserModel u2 = new UserModel(1, "Alice", "alice@example.com", RoleModel.CLIENT, "alice", "pass1");
        UserModel u3 = new UserModel(2, "Bob", "bob@example.com", RoleModel.ADMIN, "bob", "pass2");

        // Comparaison de contenu via equals
        System.out.println("u1.equals(u2) = " + u1.equals(u2));
        System.out.println("u1.equals(u3) = " + u1.equals(u3));

        // Affichage via toString
        System.out.println("u1 = " + u1);
        System.out.println("u3 = " + u3);

        // hashCode cohérent
        System.out.println("u1.hashCode() = " + u1.hashCode());
        System.out.println("u2.hashCode() = " + u2.hashCode());
        System.out.println("u3.hashCode() = " + u3.hashCode());
    }
}
```

Explications

* `u1` et `u2` sont deux objets différents, mais avec le même contenu (`id` et `email`).
* `equals` renvoie donc `true`.
* `hashCode` de `u1` et `u2` doit être identique si `equals` est `true`.
* `toString` permet d’afficher proprement l’objet dans la console.

---

### Points clés à retenir

* `==` compare des références, pas le contenu
* `equals` compare le contenu selon les règles que tu définis
* `hashCode` doit rester cohérent avec `equals`
* `toString` améliore la lisibilité du debug et des logs

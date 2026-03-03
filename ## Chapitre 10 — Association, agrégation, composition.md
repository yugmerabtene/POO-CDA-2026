## Chapitre 10 — Association, agrégation, composition

### Objectifs

* Comprendre les trois types de relations entre objets
* Savoir les reconnaître dans une gestion d’utilisateurs
* Poser une base cohérente pour le système d’authentification
* Garder la structure par packages et fournir tout le code complet du chapitre

---

### Structure de projet utilisée

```text id="ch10_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
    CredentialModel.java
    TeamModel.java
  service/
    AuditLoggerService.java
    ConsoleAuditLoggerService.java
    UserService.java
```

Explications

* `model` : modèles (utilisateur, identifiant, équipe).
* `service` : services (journalisation, gestion utilisateur).
* `app` : exécution et démonstration.

---

## Code complet du chapitre

### `model/CredentialModel.java`

```java id="ch10_credential_model"
package model;

public class CredentialModel {

    // Attributs d'identification (démonstration simple)
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

* `CredentialModel` représente des informations d’identification.
* Pour l’instant, on garde `password` en clair uniquement pour illustrer les relations. La sécurisation viendra dans un chapitre dédié.

---

### `model/UserModel.java`

```java id="ch10_user_model"
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

        // Création à l'intérieur : c'est l'idée de la composition
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
        System.out.println("CREDENTIAL username=" + credential.getUsername());
    }
}
```

Explications

* `credential` est un attribut de type `CredentialModel`.
* L’objet `CredentialModel` est créé dans le constructeur de `UserModel`, ce qui illustre la composition : l’utilisateur “possède” ses identifiants.
* `getCredential()` permet de consulter les identifiants (on améliorera plus tard pour éviter d’exposer des informations sensibles).

---

### `model/TeamModel.java`

```java id="ch10_team_model"
package model;

public class TeamModel {

    private String name;

    // Agrégation : l'équipe référence des utilisateurs existants
    private UserModel[] members;

    public TeamModel(String name, UserModel[] members) {
        this.name = name;
        this.members = members;
    }

    public String getName() {
        return name;
    }

    public UserModel[] getMembers() {
        return members;
    }

    public void display() {
        System.out.println("TEAM name=" + name);
        for (int i = 0; i < members.length; i++) {
            System.out.println(" - member: " + members[i].getName());
        }
    }
}
```

Explications

* `members` est un attribut qui contient des références vers des utilisateurs.
* Les utilisateurs sont créés ailleurs, puis fournis à l’équipe : c’est l’agrégation.
* L’équipe peut exister sans créer ses membres elle-même.

---

### `service/AuditLoggerService.java`

```java id="ch10_audit_logger_service"
package service;

public interface AuditLoggerService {

    // Contrat : enregistrer un message d'audit
    void log(String message);
}
```

Explications

* Une interface permet de définir un contrat de journalisation.
* Le reste du code pourra utiliser ce contrat sans dépendre d’une implémentation précise.

---

### `service/ConsoleAuditLoggerService.java`

```java id="ch10_console_audit_logger"
package service;

public class ConsoleAuditLoggerService implements AuditLoggerService {

    @Override
    public void log(String message) {
        System.out.println("[AUDIT] " + message);
    }
}
```

Explications

* Implémentation simple : écrit dans la console.
* `@Override` indique que `log` respecte le contrat défini par l’interface.

---

### `service/UserService.java`

```java id="ch10_user_service"
package service;

import model.UserModel;

public class UserService {

    // Association : le service utilise un autre service (logger)
    private AuditLoggerService auditLogger;

    public UserService(AuditLoggerService auditLogger) {
        this.auditLogger = auditLogger;
    }

    public void register(UserModel user) {
        auditLogger.log("register user email=" + user.getEmail());
    }

    public void login(UserModel user) {
        auditLogger.log("login user email=" + user.getEmail());
    }
}
```

Explications

* `auditLogger` est un attribut de type `AuditLoggerService`.
* Le logger est fourni au constructeur : `UserService` ne le crée pas lui-même.
* Cela illustre l’association : `UserService` connaît et utilise `AuditLoggerService`.

---

### `app/Main.java`

```java id="ch10_main"
package app;

import model.TeamModel;
import model.UserModel;
import service.ConsoleAuditLoggerService;
import service.UserService;

public class Main {
    public static void main(String[] args) {

        // Composition : l'utilisateur crée ses identifiants à l'intérieur
        UserModel alice = new UserModel(1, "Alice", "alice@example.com", "alice", "pass1");
        UserModel bob = new UserModel(2, "Bob", "bob@example.com", "bob", "pass2");

        alice.display();
        bob.display();

        // Agrégation : l'équipe reçoit des utilisateurs déjà créés
        UserModel[] members = new UserModel[] { alice, bob };
        TeamModel team = new TeamModel("SecurityTeam", members);
        team.display();

        // Association : UserService utilise un logger via un contrat
        UserService userService = new UserService(new ConsoleAuditLoggerService());
        userService.register(alice);
        userService.login(bob);
    }
}
```

Explications

* Composition : `UserModel` crée `CredentialModel` dans son constructeur, lien fort.
* Agrégation : `TeamModel` regroupe des utilisateurs existants, lien plus faible.
* Association : `UserService` utilise `AuditLoggerService` pour journaliser, relation “utilise”.

---

### Points clés à retenir

* Composition : l’objet contient et crée un autre objet, lien fort
* Agrégation : l’objet référence des objets existants, lien plus faible
* Association : un objet utilise un autre objet via un contrat, relation “connaît/emploie”

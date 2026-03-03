## Chapitre 8 — Interfaces

### Objectifs

* Comprendre ce qu’est une interface et à quoi elle sert
* Utiliser une interface comme contrat commun pour plusieurs implémentations
* Préparer le système d’authentification en restant simple et progressif
* Garder une séparation claire par packages

---

### Structure de projet utilisée

```text id="ch8_tree"
src/
  app/
    Main.java
  model/
    UserModel.java
    AdminUserModel.java
    ClientUserModel.java
  service/
    AuthenticatorService.java
    PasswordAuthenticatorService.java
    AlwaysFailAuthenticatorService.java
```

Explications

* `model` : les utilisateurs.
* `service` : la logique, ici l’authentification.
* `app` : exécution et tests.

---

### 1) Contrat d’authentification : `AuthenticatorService`

```java id="ch8_authenticator_interface"
package service;

import model.UserModel;

public interface AuthenticatorService {

    // Contrat : tenter d'authentifier un utilisateur.
    // true signifie succès, false signifie échec.
    boolean authenticate(UserModel user, String secret);
}
```

Explications

* `interface` : définit un contrat. Elle décrit ce que doit savoir faire une implémentation.
* `boolean` : type simple qui vaut soit `true`, soit `false`. Ici on l’utilise pour dire succès ou échec.
* `UserModel user` : paramètre de type `UserModel`. Cela signifie que la méthode reçoit un objet utilisateur.
* `String secret` : une information fournie par l’utilisateur, par exemple un mot de passe.
* À ce stade, on ne gère pas encore la sécurité réelle du mot de passe. On pose juste l’architecture.

---

### 2) Première implémentation : `PasswordAuthenticatorService`

Cette classe respecte le contrat et fournit un comportement concret.

```java id="ch8_password_authenticator"
package service;

import model.UserModel;

public class PasswordAuthenticatorService implements AuthenticatorService {

    // Mot de passe attendu pour la démonstration.
    // Ce choix est volontairement simple pour le cours.
    private String expectedSecret;

    // Constructeur : on fixe le secret attendu au moment de créer le service
    public PasswordAuthenticatorService(String expectedSecret) {
        this.expectedSecret = expectedSecret;
    }

    @Override
    public boolean authenticate(UserModel user, String secret) {
        // Vérification minimale pour éviter les erreurs évidentes
        if (user == null) {
            return false;
        }
        if (secret == null) {
            return false;
        }

        // Vérification simple : si le secret fourni correspond au secret attendu, succès
        return expectedSecret.equals(secret);
    }
}
```

Explications

* `implements AuthenticatorService` : la classe s’engage à respecter le contrat de l’interface.
* `private String expectedSecret;` : champ interne du service, utilisé pour la comparaison.
* `public PasswordAuthenticatorService(...)` : constructeur qui initialise l’état du service.
* `@Override` : garantit que la méthode correspond bien à celle de l’interface.
* `return expectedSecret.equals(secret);` : comparaison de chaînes via `equals`, pas via `==`.
* Ce service est volontairement simple : on prépare le terrain, la vraie sécurité viendra plus tard.

---

### 3) Deuxième implémentation : `AlwaysFailAuthenticatorService`

Une autre classe qui respecte le même contrat, mais avec un comportement différent.

```java id="ch8_always_fail_authenticator"
package service;

import model.UserModel;

public class AlwaysFailAuthenticatorService implements AuthenticatorService {

    @Override
    public boolean authenticate(UserModel user, String secret) {
        // Implémentation volontairement simple : échec systématique
        return false;
    }
}
```

Explications

* Même contrat, autre comportement.
* L’intérêt pédagogique : montrer qu’une interface permet de remplacer une implémentation sans changer le code qui l’utilise.

---

### 4) Test dans `Main`

On utilise uniquement le type de l’interface dans le code d’exécution.

```java id="ch8_main"
package app;

import model.UserModel;
import service.AuthenticatorService;
import service.PasswordAuthenticatorService;
import service.AlwaysFailAuthenticatorService;

public class Main {
    public static void main(String[] args) {

        // Création d'un utilisateur simple
        UserModel user = new UserModel(1, "Alice", "alice@example.com");

        // On déclare avec le type de l'interface
        AuthenticatorService auth1 = new PasswordAuthenticatorService("secret123");
        AuthenticatorService auth2 = new AlwaysFailAuthenticatorService();

        // Même appel, résultat différent selon l'implémentation
        boolean ok1 = auth1.authenticate(user, "secret123");
        boolean ok2 = auth2.authenticate(user, "secret123");

        System.out.println("auth1 result=" + ok1);
        System.out.println("auth2 result=" + ok2);
    }
}
```

Explications

* `AuthenticatorService auth1 = ...` : variable de type interface qui référence une implémentation concrète.
* `auth1.authenticate(...)` : le code appelle la même méthode du contrat. Le comportement dépend de l’objet réel.
* `auth2` respecte le même contrat, donc il peut remplacer `auth1` sans changer la logique d’appel.
* `boolean ok1` et `boolean ok2` stockent le résultat de l’authentification.

---

### Points clés à retenir

* Une interface définit un contrat, sans expliquer comment c’est fait
* Une classe qui `implements` une interface doit fournir les méthodes du contrat
* Le code peut manipuler le type interface et remplacer facilement les implémentations
* Cette structure prépare directement l’authentification du projet final, étape par étape

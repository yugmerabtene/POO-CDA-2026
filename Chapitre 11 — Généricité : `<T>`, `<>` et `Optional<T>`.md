## Chapitre 11 — Généricité : `<T>`, `<>` et `Optional<T>`

### Objectifs

* Comprendre ce que signifie `<T>` : un type paramétré
* Comprendre `<>` : le diamond operator
* Utiliser `Optional<T>` pour représenter “présent ou absent” sans utiliser `null`
* Appliquer la généricité à un stockage en mémoire d’utilisateurs

---

### Structure de projet utilisée

```text id="ch11_tree"
src/
  app/
    Main.java
  model/
    HasId.java
    UserModel.java
  repository/
    Repository.java
    InMemoryRepository.java
```

Explications

* `model` : modèles et contrats simples.
* `repository` : stockage générique.
* `app` : exécution et démonstration.

---

## Code complet du chapitre

### `model/HasId.java`

```java id="ch11_has_id"
package model;

public interface HasId {

    // Contrat minimal pour identifier un objet
    long getId();
}
```

Explications

* Cette interface impose une méthode `getId()`.
* Elle sert à permettre au repository générique de retrouver un objet par identifiant.

---

### `model/UserModel.java`

```java id="ch11_user_model"
package model;

public class UserModel implements HasId {

    private long id;
    private String name;
    private String email;

    public UserModel(long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }

    @Override
    public long getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public void display() {
        System.out.println("USER id=" + id + " name=" + name + " email=" + email);
    }
}
```

Explications

* `implements HasId` : `UserModel` respecte le contrat `getId()`.
* `@Override` garantit que `getId()` correspond bien à la méthode de l’interface.

---

### `repository/Repository.java`

```java id="ch11_repository"
package repository;

import java.util.Optional;
import model.HasId;

public interface Repository<T extends HasId> {

    // Stocke un objet du type T
    void save(T item);

    // Recherche un objet par identifiant
    Optional<T> findById(long id);
}
```

Explications

* `Repository<T extends HasId>` :

  * `<T>` introduit un type paramétré.
  * `T extends HasId` impose que `T` possède `getId()`.
* `Optional<T>` :

  * représente un résultat présent ou absent, sans `null`.

---

### `repository/InMemoryRepository.java`

```java id="ch11_in_memory_repository"
package repository;

import java.util.Optional;
import model.HasId;

public class InMemoryRepository<T extends HasId> implements Repository<T> {

    // Stockage en mémoire : tableau d'objets
    private Object[] items = new Object[10];

    // Nombre réel d'éléments stockés
    private int count = 0;

    @Override
    public void save(T item) {
        // Si un élément avec le même id existe, on le remplace
        int index = indexOfId(item.getId());
        if (index != -1) {
            items[index] = item;
            return;
        }

        // Sinon, on ajoute à la fin
        items[count] = item;
        count++;
    }

    @Override
    public Optional<T> findById(long id) {
        int index = indexOfId(id);
        if (index == -1) {
            return Optional.empty();
        }

        T found = cast(items[index]);
        return Optional.of(found);
    }

    // Recherche interne : renvoie l'indice, ou -1 si absent
    private int indexOfId(long id) {
        for (int i = 0; i < count; i++) {
            T current = cast(items[i]);
            if (current.getId() == id) {
                return i;
            }
        }
        return -1;
    }

    // Conversion Object -> T centralisée
    private T cast(Object obj) {
        return (T) obj;
    }
}
```

Explications

* `InMemoryRepository<T extends HasId>` : la classe est générique et impose la contrainte `HasId`.
* `Object[] items` :

  * limitation Java : on ne peut pas créer directement un tableau de `T` (`new T[10]`).
  * on stocke donc des `Object` puis on convertit en `T`.
* `Optional.empty()` : aucun résultat.
* `Optional.of(found)` : résultat présent.

---

### `app/Main.java`

```java id="ch11_main"
package app;

import java.util.Optional;
import model.UserModel;
import repository.InMemoryRepository;
import repository.Repository;

public class Main {
    public static void main(String[] args) {

        // Repository paramétré avec UserModel
        Repository<UserModel> repo = new InMemoryRepository<>();

        // <> : Java déduit le type paramétré (UserModel)
        repo.save(new UserModel(1, "Alice", "alice@example.com"));
        repo.save(new UserModel(2, "Bob", "bob@example.com"));

        Optional<UserModel> result1 = repo.findById(2);
        Optional<UserModel> result2 = repo.findById(99);

        if (result1.isPresent()) {
            UserModel user = result1.get();
            user.display();
        }

        if (result2.isEmpty()) {
            System.out.println("Aucun utilisateur avec id=99");
        }
    }
}
```

Explications

* `Repository<UserModel>` : on utilise le repository générique avec un type concret (`UserModel`).
* `new InMemoryRepository<>()` : `<>` permet à Java de déduire le type paramétré.
* `Optional<UserModel>` :

  * `isPresent()` indique que la valeur existe
  * `get()` récupère la valeur si elle existe
  * `isEmpty()` indique qu’il n’y a pas de valeur

---

### Points clés à retenir

* `<T>` rend une classe ou une interface réutilisable avec plusieurs types
* `T extends HasId` impose une règle au type paramétré
* `<>` laisse Java déduire le type paramétré lors de la création
* `Optional<T>` représente une valeur présente ou absente sans utiliser `null`

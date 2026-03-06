# L’encapsulation en Java

## Introduction

L’encapsulation est un principe fondamental de la programmation orientée objet.
Son objectif est de **protéger l’état interne d’un objet** en contrôlant la manière dont on lit ou modifie ses attributs.

En Java, cela passe principalement par les **modificateurs d’accès** :

* `public`
* `private`
* `protected`
* accès par défaut, aussi appelé **package-private**

L’idée est simple :

* certains éléments doivent être accessibles partout
* d’autres seulement dans la classe
* d’autres encore seulement dans le même package ou dans les classes enfants

L’encapsulation permet donc :

* d’éviter les modifications incohérentes
* de sécuriser les données
* d’imposer des règles métier
* de rendre le code plus propre et plus maintenable

---

# 1. Définition simple

Encapsuler un objet, c’est :

* **cacher directement ses attributs**
* **passer par des méthodes contrôlées** pour lire ou modifier ces attributs

Exemple d’idée :

* un compte bancaire possède un solde
* on ne veut pas que n’importe quel code fasse `solde = -5000`
* on préfère obliger le passage par une méthode comme `retirer(double montant)`

Ainsi, la classe protège sa propre logique.

---

# 2. Les modificateurs d’accès en Java

## `public`

Accessible depuis partout.

## `private`

Accessible uniquement dans la classe elle-même.

## `protected`

Accessible :

* dans la classe elle-même
* dans le même package
* dans les classes enfants, même si elles sont dans un autre package

## accès par défaut

Quand on n’écrit rien, l’élément est accessible uniquement dans le même package.

---

# 3. Pourquoi `private` est au cœur de l’encapsulation

Dans une bonne conception orientée objet :

* les attributs sont souvent `private`
* les méthodes publiques permettent de travailler avec l’objet

Cela évite qu’un code extérieur manipule directement les données sans contrôle.

---

# 4. Premier exemple concret : sans encapsulation

## Code

```java
public class CompteBancaire {

    // Attribut public : accessible partout
    public double solde;
}
```

```java
public class Main {
    public static void main(String[] args) {
        CompteBancaire compte = new CompteBancaire();

        // N'importe quel code peut modifier directement le solde
        compte.solde = 1000;
        compte.solde = -500;

        System.out.println("Solde : " + compte.solde);
    }
}
```

## Contexte

Ici, on modélise un compte bancaire très simple avec un attribut `solde`.

## Résultat attendu

Le programme affiche :

```java
Solde : -500.0
```

## Pourquoi ce code est mauvais

Le problème est que `solde` est `public`.

Donc :

* n’importe quelle classe peut le modifier
* aucune vérification n’est faite
* on peut mettre une valeur incohérente
* l’objet ne protège pas son propre état

Dans un vrai système, ce serait dangereux.

---

# 5. Même exemple avec encapsulation correcte

## Code

```java
public class CompteBancaire {

    // Attribut privé : inaccessible directement depuis l'extérieur
    private double solde;

    // Constructeur
    public CompteBancaire(double soldeInitial) {
        if (soldeInitial >= 0) {
            this.solde = soldeInitial;
        } else {
            this.solde = 0;
        }
    }

    // Méthode publique pour lire le solde
    public double getSolde() {
        return solde;
    }

    // Méthode publique pour déposer de l'argent
    public void deposer(double montant) {
        if (montant > 0) {
            solde = solde + montant;
        }
    }

    // Méthode publique pour retirer de l'argent
    public void retirer(double montant) {
        if (montant > 0 && montant <= solde) {
            solde = solde - montant;
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        CompteBancaire compte = new CompteBancaire(1000);

        compte.deposer(200);
        compte.retirer(150);

        System.out.println("Solde final : " + compte.getSolde());
    }
}
```

## Contexte

On reprend le cas du compte bancaire, mais cette fois le solde est protégé.

## Résultat attendu

Le programme affiche :

```java
Solde final : 1050.0
```

## Pourquoi ce code est meilleur

Ici :

* `solde` est `private`
* on ne peut pas modifier `solde` directement depuis `Main`
* les méthodes `deposer` et `retirer` imposent des règles
* l’objet garde la maîtrise de son état interne

C’est exactement le rôle de l’encapsulation.

---

# 6. Rôle des getters et setters

Les méthodes souvent utilisées avec l’encapsulation sont :

* **getter** : lire un attribut
* **setter** : modifier un attribut

Exemple classique :

* `getNom()`
* `setNom(String nom)`

Mais il faut comprendre un point important :

**encapsulation ne veut pas dire mettre automatiquement un getter et un setter sur tous les attributs**.

Un setter ne doit exister que si la modification a du sens.

---

# 7. Exemple concret avec validation dans un setter

## Code

```java
public class Utilisateur {

    // Attributs privés
    private String nom;
    private int age;

    // Constructeur
    public Utilisateur(String nom, int age) {
        this.nom = nom;
        setAge(age); // On passe par la méthode pour réutiliser la validation
    }

    // Getter sur le nom
    public String getNom() {
        return nom;
    }

    // Setter sur le nom
    public void setNom(String nom) {
        if (nom != null && !nom.isBlank()) {
            this.nom = nom;
        }
    }

    // Getter sur l'âge
    public int getAge() {
        return age;
    }

    // Setter avec validation
    public void setAge(int age) {
        if (age >= 0 && age <= 120) {
            this.age = age;
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Utilisateur user = new Utilisateur("Yanis", 25);

        user.setAge(30);
        user.setAge(-10); // Refusé
        user.setNom("Sara");

        System.out.println("Nom : " + user.getNom());
        System.out.println("Âge : " + user.getAge());
    }
}
```

## Contexte

On modélise un utilisateur avec un nom et un âge.

## Résultat attendu

Le programme affiche :

```java
Nom : Sara
Âge : 30
```

## Pourquoi

* l’âge `30` est accepté
* l’âge `-10` est refusé car invalide
* le nom `"Sara"` remplace `"Yanis"`

Grâce à l’encapsulation, l’objet empêche les valeurs absurdes.

---

# 8. Exemple d’erreur si un attribut est privé

## Code

```java
public class Utilisateur {
    private String nom;
}
```

```java
public class Main {
    public static void main(String[] args) {
        Utilisateur user = new Utilisateur();

        user.nom = "Karim"; // Erreur de compilation
    }
}
```

## Contexte

On essaie d’accéder directement à un attribut privé depuis une autre classe.

## Résultat attendu

Le code ne compile pas.

## Pourquoi

L’attribut `nom` est `private`.

Cela signifie :

* accessible seulement dans la classe `Utilisateur`
* inaccessible dans `Main`

C’est une protection voulue.

---

# 9. Exemple simple de `public`

## Code

```java
public class MessageService {

    // Méthode publique : accessible partout
    public void envoyerMessage() {
        System.out.println("Message envoyé.");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        MessageService service = new MessageService();
        service.envoyerMessage();
    }
}
```

## Contexte

On a un service avec une méthode qui doit être appelée depuis l’extérieur.

## Résultat attendu

Le programme affiche :

```java
Message envoyé.
```

## Pourquoi

La méthode est `public`, donc :

* n’importe quelle autre classe peut l’appeler
* c’est normal ici, car cette méthode représente une action publique de l’objet

Une règle simple à retenir :

* les **attributs** sont souvent `private`
* les **méthodes utiles à l’extérieur** sont souvent `public`

---

# 10. Exemple simple de `private` sur une méthode

## Code

```java
public class CommandeService {

    public void validerCommande() {
        verifierStock();
        System.out.println("Commande validée.");
    }

    // Méthode interne à la classe
    private void verifierStock() {
        System.out.println("Vérification du stock...");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        CommandeService service = new CommandeService();
        service.validerCommande();

        // service.verifierStock(); // Erreur de compilation
    }
}
```

## Contexte

La vérification du stock fait partie du fonctionnement interne du service.

## Résultat attendu

Le programme affiche :

```java
Vérification du stock...
Commande validée.
```

## Pourquoi

La méthode `verifierStock()` est `private` car :

* elle ne doit pas être appelée directement depuis l’extérieur
* elle appartient à l’implémentation interne
* seule la méthode `validerCommande()` expose l’action complète

C’est aussi une forme d’encapsulation :
on cache non seulement les données, mais aussi certains détails techniques.

---

# 11. Exemple de `protected`

Le `protected` est surtout utile avec l’héritage.

## Code

```java
public class Employe {

    // Accessible dans la classe fille
    protected String nom;

    public Employe(String nom) {
        this.nom = nom;
    }
}
```

```java
public class Developpeur extends Employe {

    public Developpeur(String nom) {
        super(nom);
    }

    public void afficherNom() {
        System.out.println("Nom du développeur : " + nom);
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Developpeur dev = new Developpeur("Youghourta");
        dev.afficherNom();
    }
}
```

## Contexte

On a une classe parent `Employe` et une classe enfant `Developpeur`.

## Résultat attendu

Le programme affiche :

```java
Nom du développeur : Youghourta
```

## Pourquoi

L’attribut `nom` est `protected`, donc il est accessible :

* dans `Employe`
* dans `Developpeur`

Cela facilite certains cas d’héritage.

## Point important

En conception moderne, on évite souvent de mettre les attributs en `protected` sans raison forte.
On préfère souvent :

* attribut `private`
* getter protégé ou public si nécessaire

Car `protected` expose davantage l’état interne aux classes enfants.

---

# 12. Exemple d’accès par défaut

Quand on n’écrit aucun mot-clé d’accès, on est en accès **package-private**.

## Code

```java
class Produit {

    String nom;

    void afficherNom() {
        System.out.println("Produit : " + nom);
    }
}
```

## Contexte

Ici, ni la classe, ni l’attribut, ni la méthode ne sont précédés par `public`, `private` ou `protected`.

## Résultat attendu

* ils sont accessibles uniquement dans le même package
* ils ne sont pas accessibles depuis un autre package

## Pourquoi

C’est le comportement par défaut de Java.

Cela peut être utile pour limiter l’accès à l’échelle d’un package, mais pour débuter il faut surtout retenir que :

* sans mot-clé, l’accès n’est pas public
* il est limité au package

---

# 13. Exemple très concret : encapsulation métier

Ici, on va voir un cas plus réaliste.
On ne veut pas seulement protéger un attribut : on veut aussi protéger une règle métier.

## Code

```java
public class Produit {

    private String nom;
    private double prix;

    public Produit(String nom, double prix) {
        this.nom = nom;
        setPrix(prix);
    }

    public String getNom() {
        return nom;
    }

    public double getPrix() {
        return prix;
    }

    public void setPrix(double prix) {
        if (prix >= 0) {
            this.prix = prix;
        }
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Produit produit = new Produit("Clavier", 49.99);

        produit.setPrix(59.99);
        produit.setPrix(-10); // Refusé

        System.out.println("Produit : " + produit.getNom());
        System.out.println("Prix : " + produit.getPrix());
    }
}
```

## Contexte

On modélise un produit avec un prix.

## Résultat attendu

Le programme affiche :

```java
Produit : Clavier
Prix : 59.99
```

## Pourquoi

Le prix négatif est refusé.
Donc l’objet reste cohérent.

Sans encapsulation, un autre code pourrait forcer un prix négatif, ce qui casserait la logique métier.

---

# 14. Encapsulation et objet immuable

Parfois, on va encore plus loin : on ne permet plus de modifier les attributs après la création.

C’est une très bonne approche dans beaucoup de cas.

## Code

```java
public class Client {

    private final String nom;
    private final String email;

    public Client(String nom, String email) {
        this.nom = nom;
        this.email = email;
    }

    public String getNom() {
        return nom;
    }

    public String getEmail() {
        return email;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Client client = new Client("Nadia", "nadia@email.com");

        System.out.println("Nom : " + client.getNom());
        System.out.println("Email : " + client.getEmail());
    }
}
```

## Contexte

On crée un client dont les données ne doivent pas changer après création.

## Résultat attendu

Le programme affiche :

```java
Nom : Nadia
Email : nadia@email.com
```

## Pourquoi

* les attributs sont `private final`
* il n’y a pas de setter
* l’objet est en lecture seule après construction

C’est une forme forte d’encapsulation.

---

# 15. Différence entre protéger et cacher

L’encapsulation ne sert pas seulement à “cacher”.

Elle sert surtout à **contrôler**.

Exemple :

* cacher complètement un attribut avec `private`
* autoriser sa lecture avec un getter
* refuser certaines modifications avec un setter
* imposer une méthode métier à la place d’un setter

Exemple bancaire :

au lieu de faire :

```java
setSolde(2000)
```

on préfère parfois :

```java
deposer(2000)
retirer(500)
```

C’est plus logique métier, plus sûr, et plus propre.

---

# 16. Tableau simple de synthèse

| Modificateur     | Accessible dans la classe | Même package |       Classe enfant | Partout |
| ---------------- | ------------------------: | -----------: | ------------------: | ------: |
| `private`        |                       Oui |          Non |     Non directement |     Non |
| accès par défaut |                       Oui |          Oui | Oui si même package |     Non |
| `protected`      |                       Oui |          Oui |                 Oui |     Non |
| `public`         |                       Oui |          Oui |                 Oui |     Oui |

---

# 17. Bonnes pratiques en Java

## À retenir

### 1. Mettre les attributs en `private`

C’est la pratique la plus courante.

### 2. Exposer seulement ce qui est nécessaire

Ne pas rendre `public` un élément inutilement.

### 3. Valider les données dans les setters ou dans le constructeur

Un objet doit rester cohérent.

### 4. Préférer des méthodes métier à des setters génériques

Exemple :

* préférable : `deposer()`, `retirer()`
* moins bon : `setSolde()`

### 5. Ne pas créer automatiquement tous les getters et setters

Chaque accès doit avoir une vraie utilité.

---

# 18. Exemple final complet

## Code

```java
public class Voiture {

    // Attributs privés : protégés contre l'accès direct
    private String marque;
    private int vitesse;

    // Constructeur
    public Voiture(String marque) {
        this.marque = marque;
        this.vitesse = 0;
    }

    // Getter sur la marque
    public String getMarque() {
        return marque;
    }

    // Getter sur la vitesse
    public int getVitesse() {
        return vitesse;
    }

    // Méthode métier pour accélérer
    public void accelerer(int valeur) {
        if (valeur > 0) {
            vitesse = vitesse + valeur;
        }
    }

    // Méthode métier pour freiner
    public void freiner(int valeur) {
        if (valeur > 0) {
            vitesse = vitesse - valeur;

            // On empêche une vitesse négative
            if (vitesse < 0) {
                vitesse = 0;
            }
        }
    }

    // Méthode publique d'affichage
    public void afficherEtat() {
        System.out.println("Marque : " + marque);
        System.out.println("Vitesse : " + vitesse + " km/h");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Voiture voiture = new Voiture("Toyota");

        voiture.accelerer(50);
        voiture.freiner(20);
        voiture.freiner(100);

        voiture.afficherEtat();
    }
}
```

## Contexte

On modélise une voiture avec une marque et une vitesse.

## Résultat attendu

Le programme affiche :

```java
Marque : Toyota
Vitesse : 0 km/h
```

## Pourquoi

Déroulement :

* la voiture démarre à `0`
* `accelerer(50)` donne `50`
* `freiner(20)` donne `30`
* `freiner(100)` ferait `-70`, mais la classe corrige à `0`

Donc l’objet reste toujours dans un état valide.

C’est un très bon exemple d’encapsulation, car :

* les attributs sont protégés
* les modifications passent par des méthodes contrôlées
* la logique métier est centralisée dans la classe

---

# 19. Phrase d’examen simple

Tu peux retenir cette formulation :

> L’encapsulation en Java consiste à protéger les attributs et certains détails internes d’une classe en limitant leur accès grâce aux modificateurs comme `private`, `protected` et `public`, puis en contrôlant les lectures et modifications à l’aide de méthodes.

---

# 20. Résumé final

L’essentiel à retenir est le suivant :

* `private` protège l’intérieur de la classe
* `public` expose ce qui doit être accessible de l’extérieur
* `protected` sert surtout dans l’héritage
* sans mot-clé, l’accès est limité au package
* une bonne encapsulation protège les attributs et impose des règles de cohérence

Pour bien coder en Java, la base saine est souvent :

* attributs `private`
* méthodes `public` pour les actions utiles
* validation dans les constructeurs et méthodes
* éviter de laisser l’extérieur modifier librement l’état de l’objet

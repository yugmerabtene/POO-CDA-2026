* **Abstraction** est un **principe** de conception : exposer le “quoi” et masquer le “comment”. Elle peut être faite avec une interface, une classe abstraite, ou même une classe classique bien conçue.
* **Classe abstraite** est un **mécanisme du langage Java** : un type non instanciable qui peut contenir du code commun et imposer des méthodes à implémenter.

Donc tu peux les **fusionner dans un syllabus** pour alléger, à condition d’être clair que :

* “Abstraction” = principe
* “classe abstraite” = une façon (parmi d’autres) de mettre ce principe en pratique

Voici le syllabus fusionné, propre et cohérent.

## Chapitre-01 — Classe, objet, instance

* Définition et rôle
* Références et identité d’objet
* Cycle de vie et création d’objets

## Chapitre-02 — Attributs, méthodes et constructeurs

* État vs comportement
* Visibilités (public, private, protected)
* Constructeur, invariants, surcharge de constructeurs

## Chapitre-03 — Encapsulation

* Masquage des données
* Validation et règles métier
* Cohérence et invariants d’objet

## Chapitre-04 — Abstraction

* Principe “quoi vs comment”
* Contrats (interfaces)
* Classes abstraites : rôle, cas d’usage, limites
* Différences interface vs classe abstraite

## Chapitre-05 — Héritage

* Relation “est-un”
* Spécialisation et réutilisation
* Limites et bonnes pratiques (éviter hiérarchies profondes)

## Chapitre-06 — Polymorphisme

* Substitution et comportement dynamique
* Extensibilité du code
* Cas d’usage typiques en architecture

## Chapitre-07 — Interfaces

* Responsabilités et contrats
* Segmentation des interfaces
* Implémentations multiples et découplage

## Chapitre-08 — Surcharge et redéfinition

* Surcharge (overloading) : signatures, résolution
* Redéfinition (override) : règles, cohérence, contrat

## Chapitre-09 — Relations entre objets

* Association
* Agrégation
* Composition

## Chapitre-10 — Généricité

* Types paramétrés
* Contraintes de type
* Sécurité de compilation, collections typées

## Chapitre-11 — Exceptions et gestion d’erreurs

* Exceptions métier vs techniques
* Propagation contrôlée
* Bonnes pratiques de robustesse

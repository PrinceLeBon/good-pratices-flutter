# Good Practices — Flutter/Dart

Standards que j'applique sur tous mes projets. Chaque dossier est un standard
autonome : son README pose les invariants, les chapitres numérotés expliquent
le pourquoi et donnent les vérifications.

## Standards

### [Offline-first](offline-first/README.md)

Couches **données** (providers remote/local) et **Repository** d'une app
offline-first, indépendantes du state management.

`ApiHttp` · `HttpException` · `Either<Failure, T>` · `readOnlineFirst` ·
file de synchronisation · cache Hive

### [Publication App Store — iOS](publication-app-store/README.md)

Ce qu'il faut décider, coder et préparer pour passer la revue Apple au premier
essai. Centré sur le point qui coince le plus : encaisser hors de l'achat
intégré, et savoir dans quels cas Apple le permet.

modèle de monétisation · anti-steering · métadonnées et pages publiques ·
comptes de démonstration · signature et build · réponses au reviewer

## Convention de chaque standard

```
README.md            invariants non négociables + checklist
01-….md … NN-….md    un sujet par fichier, numérotés dans l'ordre de lecture
<retour>-<app>.md    retour d'expérience issu d'un projet réel
```

Les **invariants** du README sont la partie non négociable — ce qui se vérifie
en revue de code. Les chapitres sont là pour expliquer, pas pour être récités.

Les fichiers de retour d'expérience gardent la trace de ce qu'un projet donné a
coûté : ils alimentent les standards, et ne se confondent pas avec eux.

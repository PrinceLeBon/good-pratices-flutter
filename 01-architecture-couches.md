# 1. Architecture en couches

## Sens des dépendances (strict)

```
UI (screens / widgets)
      │
      ▼
Controller (état réactif)        ← ne connaît NI le réseau NI le cache
      │
      ▼
Repository                       ← SEULE couche qui connaît l'API ET le cache
      │
      ├─► Provider (HTTP via ApiHttp)   → renvoie des modèles OU lève
      └─► Cache local (Hive / SQLite…)  → lecture/écriture encapsulées
```

## Règles

- **Un écran / un controller n'accède jamais au réseau ni au cache directement.**
  Il passe par le Repository.
- **Le Repository est la seule couche qui décide online/offline** et applique
  les effets de bord (écriture cache, file de sync).
- **Provider** : encapsule un endpoint. Renvoie un **modèle typé** en cas de
  succès, **lève une exception** en cas d'erreur (voir
  [02](02-reseau-et-erreurs.md)). Ne connaît pas le cache.
- **Repository** : renvoie **`Either<String, T>`** (dartz) — `Left` = message
  d'erreur, `Right` = succès. Il orchestre Provider + cache.

## Pourquoi

- Testabilité : la logique (calculs, agrégations) vit dans des **fonctions
  pures** / builders sans dépendance réseau ni cache → testables seuls.
- Un seul endroit à auditer pour la cohérence online/offline.
- Le controller reste « bête » : il appelle le repo et expose l'état.

## Anti-patterns à bannir

- Un controller qui écrit dans le cache ou appelle `http` directement.
- Un provider qui lit/écrit le cache.
- De la logique métier (calcul) enfouie dans la couche d'accès aux données.

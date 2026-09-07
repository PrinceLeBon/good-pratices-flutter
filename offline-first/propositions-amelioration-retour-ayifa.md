# Propositions d'amélioration de ce standard — issues de l'expérience AYIFA

> **Ce document ne fait pas partie du standard : ce sont des propositions.**
> Rien de ce qui suit n'est appliqué aux fichiers `01` à `09`, et rien ne doit
> l'être sans arbitrage.
>
> **D'où ça vient.** AYIFA est une plateforme de sécurisation et de certification
> des **transactions foncières** au Bénin. Le dépôt concerné est une suite mobile
> Flutter (monorepo de trois paquets) branchée sur un backend Spring Boot de huit
> microservices, avec des conseillers qui saisissent des biens sur le terrain,
> souvent sans réseau. Le domaine change la donne par rapport à GestFerme : ici
> une donnée périmée n'est pas gênante, elle est **dangereuse** — un certificat
> révoqué qui s'affiche comme valide engage un acheteur sur un titre qui ne vaut
> plus rien.
>
> **Sur quoi elles s'appuient.** Chaque point part d'un incident réellement
> rencontré, daté, et corrigé. Aucun ne vient d'une préférence de style.
> Rédigé le **2026-08-22**.
>
> **Convention.** Reprise de `propositions-amelioration-retour-gestferme.md`, pour
> que les deux retours se lisent de la même façon : **A** = ajout,
> **C** = correction, **S** = allègement, et un **§0** pour ce qui demande un
> arbitrage avant tout le reste. Les numéros sont préfixés `AY-` afin de ne pas
> entrer en collision avec ceux de GestFerme, cités ici sous la forme
> « GestFerme A1 ».

---

## 0. Ce qui demande un arbitrage

### 0.1 `Result<T>` — une troisième issue au point ouvert par GestFerme

GestFerme constate que le projet fondateur n'applique pas l'invariant central :
`Either<Failure, T>` compté **0 fois** contre **279** `Either<String, …>`. Deux
issues sont proposées — assumer la cible, ou renoncer au typage.

**Il en existe une troisième, et AYIFA tourne dessus depuis le premier jour.**

```dart
sealed class Result<T> { const Result(); }
final class Ok<T>  extends Result<T> { const Ok(this.value);  final T value; }
final class Err<T> extends Result<T> { const Err(this.failure); final Failure failure; }
```

Dart 3, **aucune dépendance** à `dartz`, et surtout un `switch` exhaustif
**vérifié par le compilateur** : oublier un cas d'erreur ne compile pas.

```dart
emit(switch (failure) {
  NotFoundFailure() => CertificateScanUnknown(reference),
  NetworkFailure()  => CertificateScanOffline(reference),
  _                 => CertificateScanFailed(failure),
});
```

C'est exactement ce que le §8 demande — « réagir par type » — et ce qu'un
`fold((l) => …)` sur une `String` rend impossible.

**Pourquoi ça pèse sur l'arbitrage de GestFerme.** L'argument « 279 sites à
migrer » ne plaide pas contre le typage, il plaide pour la forme la moins chère
à adopter. Une `sealed class` n'a rien à installer, et l'exhaustivité vérifiée
donne une garantie que `Either` ne donne pas : avec `dartz`, rien n'empêche
d'avaler le `Left`.

**Proposition.** Faire de `Result<T>` l'invariant n°7, et mentionner `Either`
comme la variante historique acceptable. Répercuter dans les §2.4, §5.1 et §8.

### 0.2 Le repli sur cache n'est pas toujours souhaitable ⭐ priorité haute

Le §3 ouvre sur « **toute** lecture passe par un seul helper », avec une table de
décision où réseau KO → cache, systématiquement, en `Right`.

**Vécu (22/08/2026).** Écran de vérification d'un certificat foncier. Servir un
verdict mis en cache afficherait « certificat authentique » sur un titre
**révoqué depuis** — précisément le cas que la révocation existe pour couvrir.
Le dépôt refuse donc de répondre hors ligne, et l'écran dit qu'il ne sait pas :

```dart
Future<Result<PublicCertificate>> verifyById(String id) async {
  if (await _connectivity.isOffline()) {
    return const Err<PublicCertificate>(NetworkFailure());
  }
  return _api.verifyById(id);
}
```

**La règle qu'on en tire.** *Une donnée dont la péremption est le sujet ne se met
pas en cache.* Certificat, solde, droit d'accès, statut de validation : tout ce
dont la valeur d'hier peut nuire aujourd'hui. Le repli sur cache protège d'un
écran vide ; il ne doit pas protéger d'une information devenue fausse.

**Ce que ça change dans le standard.** Le §3.1 devient une politique **par
défaut**, pas une politique universelle, avec une ligne de plus dans la table :

| Situation | Retour |
|---|---|
| Lecture **critique de fraîcheur**, hors ligne ou réseau KO | **`Err(NetworkFailure)`** — jamais de cache |

Et trois lignes pour reconnaître le cas : *la donnée peut-elle être révoquée,
expirée ou retirée entre deux consultations ? Si oui, elle ne se sert pas de
mémoire.*

**Rapport à GestFerme A1.** Les deux points disent la même chose vue de deux
côtés : le repli n'est pas gratuit. GestFerme veut le **compter**, AYIFA ajoute
qu'il faut parfois l'**interdire**. Ils se renforcent.

### 0.3 `Failure.message` encourage exactement ce qu'il ne faut pas faire

Le §2.4 conclut : « `Failure.message` donne toujours un texte affichable →
l'affichage simple reste trivial ». C'est une invitation à montrer le message du
serveur.

**Vécu.** Sur ce backend, les messages sont **génériques par construction** :
« Une erreur s'est produite. », « Vous ne detenez pas la permission requise pour
cette action. » Les afficher n'apprend rien à l'utilisateur, et surtout **les
afficher empêche de traduire et de nuancer**. AYIFA en a fait un interdit : on se
branche sur le `code` de l'enveloppe, on écrit ses propres phrases.

**Le coût du contraire, mesuré le 21/08/2026.** La visionneuse affirmait « Le
lien d'accès expire au bout de 15 minutes » pour *n'importe quel* échec de
lecture vidéo. La vraie cause était un refus Android du HTTP en clair. Le message
a envoyé chercher côté serveur un problème qui était dans le manifeste. Le
commentaire du code justifiait même le raccourci en affirmant que la cause
n'était pas discernable — le log d'ExoPlayer la nommait précisément.

**Proposition.** Deux règles jumelles dans le §2.4 :

1. `Failure.message` sert au **journal et au rapport d'erreur**, jamais à
   l'affichage. L'écran choisit ses mots à partir du **type** de `Failure` et du
   `code` métier.
2. **Ne jamais affirmer une cause qu'on ne connaît pas.** Une explication qui
   devine envoie chercher au mauvais endroit — ce qui coûte plus cher qu'un
   « le chargement a échoué » honnête.

**Rapport à GestFerme A2.** Aucune contradiction, une précision utile : GestFerme
veut **joindre le message du serveur au rapport d'erreur** (c'est lui qui nomme
le champ fautif), AYIFA veut **ne jamais le montrer à l'utilisateur**. Les deux
usages sont opposés, et le standard doit le dire explicitement sous peine qu'on
lise l'un pour l'autre.

---

## 1. Ajouts

### AY-A1 — L'enveloppe de réponse, déballée une seule fois ⭐ priorité haute

**Constat.** Le standard suppose qu'un `ApiProvider` rend un modèle. Il ne dit
rien du cas — très répandu dès qu'il y a une passerelle — où **toute** réponse
est encapsulée :

```jsonc
{ "success": true, "status": 200, "code": "CERTIFICATE_VERIFIED",
  "message": "…", "data": { /* le contenu utile */ },
  "details": [], "meta": { … } }
```

**Vécu.** Sans règle, chaque provider réinvente le déballage, l'enveloppe fuit
jusque dans les tests, et le jour où `meta` gagne un champ, dix fichiers bougent.

**La règle AYIFA.** **Le client HTTP la déballe une fois ; aucun repository ne la
connaît.** Le provider reçoit déjà `data`, et le `code` sert au mapping vers
`Failure`.

**Le piège qui va avec, et qui mérite d'être écrit.** Il y a toujours une route
qui n'est pas enveloppée. Ici, `GET /certificates/{id}/pdf` rend le fichier — mais
**un échec sur cette même route reste enveloppé**. Le client a donc besoin d'un
chemin « brut » qui lit les octets *et* sait décoder une enveloppe d'erreur, sans
quoi un `403` parfaitement nommé retombe sur « panne inconnue ».

**Cible.** `02-reseau-et-erreurs.md`, nouveau §2.7.

### AY-A2 — Vérifier qu'un filtre filtre vraiment ⭐ priorité haute

**Constat.** Absent du standard, et c'est le point le plus transférable de ce
retour.

**Vécu.** Le backend expose des filtres Spring `Criteria` (`?type.equals=`). **Un
nom de filtre inconnu est ignoré en silence** — la requête réussit et rend
*toute* la collection. Une recherche ignorée ressemble donc trait pour trait à
une recherche qui trouve tout le monde.

**La discipline.** Trois requêtes avant de câbler un filtre, jamais une :

| Requête | Attendu |
|---|---|
| `email.contains=gerard1` | 1 résultat |
| `email.contains=zzzzinexistant` | **0** — il restreint donc réellement |
| `emailXXX.contains=gerard1` | 5 — le **témoin**, ignoré comme prévu |

Sans la deuxième, on ne sait pas s'il filtre. Sans la troisième, on ne sait pas
si le nom est bon. Appliqué deux fois le 22/08/2026, sur la recherche par e-mail
et sur `certificates?propertyId.equals=` : les deux étaient valides, mais le
contrôle a coûté deux minutes contre une demi-journée de doute.

**Cible.** `05-conventions.md`, nouveau §5.7 « Vérifier un contrat de filtrage ».

### AY-A3 — Les permissions sont un sujet de la couche données

**Constat.** Le standard n'en parle jamais.

**Vécu.** Dix rôles, et une matrice qui bouge à chaque migration backend. Deux
règles se sont imposées :

1. **Le serveur reste l'autorité.** Le gating côté client est de l'**ergonomie**,
   pas de la sécurité — l'app ne décide rien, elle évite un aller-retour perdu.
2. **Sans le droit, le contrôle n'est pas rendu** — pas seulement désactivé. Un
   bouton grisé laisse croire que le droit s'obtient en insistant.

**Corollaire.** Ça ajoute un état d'écran que le standard ne liste pas :
**« sans permission »**, distinct de « vide » et de « erreur ».

**Cible.** `01-architecture-couches.md` (responsabilités) et une ligne dans le §8.

### AY-A4 — Les registres de valeurs : ne jamais inventer un statut

**Constat.** Le standard ne dit rien des enums venus du serveur.

**Vécu.** Un backend en cours de livraison ajoute des valeurs. Une `enum` Dart
stricte plante au parsing ; une `enum` complétée « à la devinette » affiche des
statuts qui n'existent pas.

**La règle AYIFA.** Un **registre** — des constantes, une table de libellés, et
une **dégradation sur la valeur brute** :

```dart
static String label(String raw) => _labels[raw] ?? raw;
static bool isKnown(String raw) => _labels.containsKey(raw);
```

Si le backend ajoute `SUSPENDED`, on affiche `SUSPENDED`. Pas « valide », et pas
un plantage. On complète **en observant l'API**, pas en lisant un `record` Java.

**Cible.** `05-conventions.md`, nouveau §5.8.

### AY-A5 — Un chapitre « méthode d'observation »

**Constat.** Le standard décrit une architecture. Il ne dit pas **comment on
établit le contrat** qu'on va coder — et c'est là que se perdent les journées.

Trois règles, chacune payée :

**Observer avant de typer.** Une forme déduite d'un `record` Java n'est pas
vérifiée. On appelle l'endpoint, on consigne la forme observée, puis on écrit le
modèle.

**Reconstruire avant d'observer (22/08/2026).** Mes conteneurs exécutaient du
code plus vieux que les sources : une route rendait `500` avec
`NoResourceFoundException: No static resource …` — c'est-à-dire *cette route
n'existe pas ici*. Sans reconstruction, j'aurais documenté le contrat
d'avant-hier.

**Un constat tiré du code seul peut être faux (22/08/2026).** J'avais écrit une
remontée affirmant qu'aucune route ne menait d'un bien à son certificat. Elle
existait, répondait `200`, et filtrait correctement — je ne l'avais pas essayée.
*« Je ne vois pas de route » n'est pas « il n'y a pas de route ».*

**Cible.** Nouveau `10-methode-observation.md`. Court, trois sections.

### AY-A6 — Le `clientId` naît avec le brouillon, pas avec l'opération de sync

**Constat.** Le §6.2 génère l'uuid **au moment d'enfiler**.

**Vécu.** Un brouillon existe **avant** d'être enfilé, se modifie plusieurs fois,
et peut ne jamais partir. S'il n'a pas d'identité dès sa création, on ne sait ni
le retrouver, ni le dédupliquer, ni rattacher ses photos.

**La règle AYIFA.** Le `clientId` (UUID) se génère **à la création du brouillon**,
jamais au moment de la synchronisation — et c'est lui qui porte l'idempotence
serveur, sans en-tête `Idempotency-Key`.

**Cible.** `06-file-de-sync.md` §6.2 et §6.6.

---

## 2. Corrections

### AY-C1 — Une valeur qui signifie deux choses

**Constat.** Ce n'est pas une règle absente du standard, c'est un **motif de bug**
qui mérite d'être nommé, parce qu'on le reconnaît une fois qu'on l'a vu.

Quatre occurrences sur ce seul projet :

| Où | La confusion | Le symptôme |
|---|---|---|
| `price` → `sellingPrice` | même champ, deux noms | **quatre jours** de biens synchronisés avec un `200` et un prix perdu |
| Filtre `null` | « aucun filtre » **et** « ne change rien » | le filtre « Tous » restait coincé |
| `isPhoto` / `isMedia` | photo **et** vidéo comptées ensemble | une vidéo comptait pour une photo dans le minimum requis |
| Badge de synchro | brouillons locaux **et** file d'attente | un badge « 1 » que l'onglet ne justifiait jamais |

**La règle.** Quand un compteur, un badge ou un filtre paraît « figé » ou
« faux », chercher d'abord **s'il ne mesure pas autre chose que ce qu'il annonce**.
Et corriger la source, pas l'affichage : le badge et son écran doivent lire **la
même** donnée, sinon l'écart réapparaîtra.

**Cible.** `06-file-de-sync.md` §6.7 « Pièges connus », qui est déjà l'endroit
prévu pour ce genre de savoir.

### AY-C2 — Un test qui ne tombe jamais ne prouve rien

**Constat.** Le §5.5 parle de couverture, jamais de **pouvoir de détection**.

**Vécu.** AYIFA vérifie chaque garantie par **mutation** : on retire la
protection, on s'assure que le bon test échoue — et lui seul — puis on restaure.
Appliqué trois fois le 22/08/2026 (blocage hors-ligne, verdict déduit du statut,
garde-fou anti-rafale). Sans ce contrôle, plusieurs tests écrits ce jour-là
seraient passés quoi qu'il arrive.

**Corollaire.** **Les écrans se montent dans un arbre Flutter**, pas seulement
leur Cubit. AYIFA a payé un écran blanc le 07/08/2026 pour l'apprendre : l'état
était correct, l'écran ne se construisait pas.

**Cible.** `05-conventions.md` §5.5, deux paragraphes.

---

## 3. Pièges courts, mais coûteux

À ranger là où ils tombent, sans chapitre dédié.

**`dart:io` ignore la politique réseau d'Android, les greffons natifs non.**
Nos appels d'API passaient en HTTP clair sans rien déclarer, parce que le client
Dart ouvre ses sockets lui-même. ExoPlayer, pile native, se faisait refuser :
`CleartextNotPermittedException`. D'où une application qui parle au backend et
refuse de lire une vidéo — parfaitement déroutant tant qu'on n'a pas le log.
Remède : un `network_security_config.xml` dans les **variantes de développement
seulement**. → `07-connectivite.md`.

**Ne jamais suivre une URL venue d'un contenu scanné.** Le QR d'un certificat
contient une URL. On en extrait l'identifiant et on interroge **sa propre**
passerelle : la suivre laisserait un code contrefait pointer l'application vers un
serveur complaisant qui répondrait « valide ». → `02-reseau-et-erreurs.md`.

**Le jeton vit dans le stockage sécurisé, nulle part ailleurs.** Ni en base
locale, ni en `SharedPreferences`, ni dans un singleton en mémoire qui survit à
la déconnexion. Le standard ne dit rien du stockage des identifiants alors qu'il
décrit toute la couche données. → `09` généralisé (voir AY-S1).

**Les montants ne passent jamais par un `double`.** Le franc CFA n'a pas de
sous-unité, et le backend rend pourtant `3000000.0`. Garder le texte tel quel
jusqu'à l'affichage évite d'hériter des arrondis de quelqu'un d'autre.
→ `05-conventions.md`, à côté du §5.6 sur les dates.

---

## 4. Allègements

### AY-S1 — Le chapitre 9 devrait devenir agnostique du moteur de stockage

Le §9 s'appelle « Cache local **Hive** » et son contenu est spécifique. AYIFA
utilise **Drift + SQLCipher**, et le guide de travail du projet note déjà que
« les chapitres 02, 03, 04 et 08 restent valables, le 09 est à adapter ».

Ce qui compte n'est pas Hive : c'est le **contrat `LocalProvider`** — une source
locale typée, aux opérations granulaires, qui ne laisse fuir aucun type de son
moteur. Une `Box<X>` et une table Drift sont deux implémentations du même
contrat.

**Proposition.** Renommer en « Cache local », garder le contrat et les règles
(§9.1 à §9.3), et pousser les spécificités Hive dans une annexe — au même titre
qu'une annexe Drift. **Ajouter le chiffrement au repos** : ni GestFerme ni AYIFA
ne le mentionnent dans le standard, alors qu'AYIFA stocke des titres fonciers sur
des téléphones qui circulent.

### AY-S2 — Sur GestFerme S1, je diverge

GestFerme propose d'alléger le §8 à deux adaptateurs. **Je ne le ferais pas
maintenant.** Le §8 est le chapitre qui *justifie* l'invariant de contrat typé.
L'amputer pendant que le §0 est ouvert affaiblirait l'argument au moment précis
où il en a le plus besoin. À reconsidérer une fois §0.1 tranché.

Sur **S2, S3 et S4**, aucune objection.

---

## 5. Ordre de traitement proposé

1. **Trancher §0.1 (`Result` vs `Either`)** — conditionne les §2.4, §5.1 et §8,
   et débloque l'arbitrage laissé ouvert par GestFerme.
2. **§0.2 et §0.3** — deux corrections qui touchent des invariants énoncés, donc
   à faire avant que d'autres projets s'appuient dessus.
3. **AY-A1, AY-A2** — les deux ajouts au plus fort rendement, courts à écrire.
4. **AY-C1, AY-C2** — tirés de bugs déjà corrigés, faciles à documenter.
5. **AY-A3, AY-A4, AY-A6** — sections courtes dans des chapitres existants.
6. **§3 (pièges courts)** — à disperser d'un bloc.
7. **AY-A5 et AY-S1** — vrais chapitres à écrire, à planifier à part.

## 6. Ce que ce retour ne remet pas en cause

Le découpage en couches, la place du Repository comme seul décideur online /
offline, l'interdiction de toucher le stockage local hors du `LocalProvider`, le
client HTTP unique avec timeout, et l'écriture optimiste suivie d'une file de
rejeu : **tout cela a tenu sur un projet foncier, offline-first, à huit
microservices**, sans qu'on ait eu à contourner la règle une seule fois. Les
propositions ci-dessus portent sur ce que le standard **ne dit pas encore**, pas
sur ce qu'il dit.

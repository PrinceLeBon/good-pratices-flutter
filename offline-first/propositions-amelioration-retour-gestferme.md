# Propositions d'amélioration de ce standard — issues de l'expérience GestFerme

> **Ce document ne fait pas partie du standard : ce sont des propositions.**
> Rien de ce qui suit n'est appliqué aux fichiers `01` à `09`, et rien ne doit
> l'être sans arbitrage.
>
> **D'où ça vient.** Ces propositions naissent de l'usage réel du standard sur
> **GestFerme** — application Flutter + backend NestJS de gestion de fermes
> avicoles au Bénin, plusieurs employés par ferme, réseau intermittent. Elles
> disent ce que la confrontation au terrain suggère d'**ajouter**, de
> **corriger** ou d'**alléger** dans le standard.
>
> **Sur quoi elles s'appuient.** Chaque point part d'un incident réellement
> rencontré sur ce projet — bug corrigé, angle mort découvert, règle qui s'est
> révélée impraticable — et non d'une préférence de style. Les propositions sont
> donc datées : rédigées le **2026-08-19**, après une série de correctifs et
> l'intégration d'un outil de rapport d'erreurs.
>
> Convention de lecture : **A** = ajout, **C** = correction, **S** = allègement.

---

## 0. Le point à trancher avant tout le reste — `Failure`

Le standard fait de `Either<Failure, T>` un invariant (n°7), et trois chapitres
en dépendent : 2.4 (taxonomie), 5.1 (contrat), 8 (réagir par type).

Dans GestFerme :

| | |
|---|---|
| `Either<String, …>` | **279** occurrences |
| `Either<Failure, …>` | **0** |
| `sealed class Failure` | n'existe pas |

Le projet qui a nourri ce standard n'applique donc pas sa règle centrale. Tant
que ce n'est pas tranché, le document se lit comme une description de l'existant
et il est faux sur ce point.

**Deux issues possibles :**

1. **C'est une cible.** Le dire explicitement dans le README (« état visé, non
   encore atteint »), et brancher le §5.4 (expand / migrate / contract) sur cette
   migration précise.
2. **C'est irréaliste.** Assumer `Either<String, T>` et déplacer la typologie
   d'erreur au niveau du client HTTP — approche retenue de fait dans GestFerme
   avec `ApiHttp._inspect`, qui classe les réponses sans toucher aux 279 sites.

**Coût observé du statu quo :** les `fold((l) => …)` avalent des messages en
`String`, donc aucun code ne peut réagir par type d'erreur. Consigné comme point
ouvert dans `docs/SENTRY.md` du dépôt mobile.

---

## 1. Ajouts

### A1 — Replier sur le cache n'est pas ignorer l'échec ⭐ priorité haute

**Constat.** Le §3.1 prescrit : réseau KO → cache, 5xx → cache, et renvoie
`Right`. Du point de vue de l'app, **tout va bien** alors que le serveur est en
panne. Ni l'utilisateur ni l'équipe n'en savent rien.

**Le trou.** Le standard décrit un mécanisme qui **rend les incidents
invisibles**, sans jamais dire qu'il faut les compter. C'est sa faiblesse
conceptuelle principale.

**Proposition.** Un chapitre *observabilité*, et une règle dans le §3 :
*tout repli est un événement à tracer*. Avec la contrepartie indispensable — une
panne réseau chez un utilisateur hors couverture est **normale** : filtrer, sinon
le bruit noie tout et épuise le quota de l'outil.

**Cible.** Nouveau `10-observabilite.md`, et un encart dans `03-lecture-offline-first.md`.

### A2 — En Dart, un 4xx ne lève aucune exception ⭐ priorité haute

**Constat.** `http.post` rend une `Response` avec `statusCode: 400`. Le provider
ne lève que s'il teste le code ; le repository renvoie un `Left` ; l'utilisateur
voit « une opération qui ne passe pas ». **Rien ne remonte.**

**Vécu.** Cas anticipé sur GestFerme : un déploiement backend rend un champ
obligatoire, l'app ne l'envoie pas, toutes les créations échouent en silence.
Sans instrumentation, le seul signal est un fermier qui téléphone.

**Proposition.** Le dire noir sur blanc dans le §2, et documenter le remède :
inspecter les réponses `>= 400` au niveau du client HTTP unique, en joignant
**le message du serveur** (c'est lui qui nomme le champ fautif) et **les noms des
champs envoyés, sans leurs valeurs** — le message dit ce qui manque, jamais ce
qu'on a réellement transmis, et c'est cet écart qui coûte une heure sur un
renommage.

**Cible.** `02-reseau-et-erreurs.md`, nouveau §2.6.

### A3 — Cloisonner le cache par locataire

**Constat.** Le standard parle de `Box<X>` typée, jamais de la clé par
`fermeId`.

**Vécu.** Bug réel : les types de transaction de **toutes** les fermes se
retrouvaient mélangés dans une seule box, et s'affichaient en double dans les
formulaires. `HydrationService` applique déjà la bonne pratique
(`lastFullSyncAt:$fermeId`) mais elle n'est écrite nulle part.

**Corollaire absent.** **Purger le cache à la déconnexion** : le téléphone passe
d'un employé à l'autre, et sans purge le suivant voit les données du précédent.

**Cible.** `09-cache-hive.md`, nouveau §9.6.

### A4 — N'annoncer un état qu'une fois les effets écrits

**Constat.** Règle d'ordonnancement absente du standard.

**Vécu.** Crash `type 'Null' is not a subtype of type 'Ferme'` à la connexion :
`emit(LoginSuccess)` partait **avant** `fermeBox.put('fermes', …)`. L'émission
passe par un stream, donc l'écran réagissait dès le premier `await` suivant et
lisait une box encore vide.

**Pourquoi ça vise l'offline-first.** L'écran lit le **local** juste après un
changement d'état : l'ordre « persister puis annoncer » y est bien plus critique
qu'en architecture online-only.

**Cible.** `01-architecture-couches.md` ou `05-conventions.md`.

### A5 — Le multi-utilisateur et le droit d'écrire hors ligne

**Constat.** Le standard suppose implicitement **un** utilisateur par jeu de
données. La question « trois employés partagent une ferme, qui peut écrire hors
ligne ? » n'est jamais posée.

**Vécu.** GestFerme a dû construire un verrou écrivain unique
(`OfflineOwnerGuard` côté backend, mode hors-ligne détenu par une seule personne,
abandon de file orpheline quand le détenteur change). Sans ce verrou, deux
saisies hors ligne concurrentes divergent sans réconciliation possible.

**Cible.** Nouveau chapitre, ou §6.7 étendu.

### A6 — Le temps réel

**Constat.** Aucune mention des websockets, alors qu'ils changent la donne :
les données bougent sans que l'utilisateur agisse.

**Vécu.** Reconnexions fantômes, URL morte, écran figé quand un collègue ajoutait
une transaction. La règle finalement retenue : **une mise à jour venue d'un tiers
se recharge depuis le local, sans loader bloquant** — sinon l'écran d'un employé
se fige à chaque saisie d'un autre.

**Cible.** Nouveau chapitre, ou encart dans `07-connectivite.md`.

---

## 2. Corrections

### C1 — `isNetworkError` contredit l'interdit du §5.3

Le §5.3 interdit `.contains('SocketException')`. Le §2.3 propose une fonction qui
**ne fait que** des `.contains` sur le message d'erreur. C'est le même procédé,
simplement centralisé — fragile aux formulations et à la locale.

**Proposition.** Tester les **types** plutôt que le texte :

```dart
bool isNetworkError(Object e) =>
    e is SocketException ||
    e is TimeoutException ||
    e is HandshakeException ||
    e is http.ClientException;
```

L'interdit doit porter sur le **procédé**, pas sur l'endroit où on l'écrit.
Garder éventuellement le repli textuel en dernier recours, en le signalant comme
tel.

**Cible.** `02-reseau-et-erreurs.md` §2.3 et `05-conventions.md` §5.3.

### C2 — `hasServerConnection` est plus restrictif que le code réel

Le §7.2 renvoie `true` si `statusCode < 500`. L'implémentation de GestFerme est
meilleure : **n'importe quelle réponse HTTP prouve que l'hôte répond**, 500
compris — seules les erreurs réseau et les timeouts signifient « injoignable ».
Un backend qui renvoie 500 est joignable ; c'est une information différente.

**Cible.** `07-connectivite.md` §7.2.

---

## 3. Allègements

### S1 — Deux adaptateurs au lieu de quatre (§8)

Cubit et GetX suffisent (ce sont ceux utilisés). Riverpod et ChangeNotifier
ajoutent une quarantaine de lignes que personne ne lit. La règle — un contrat,
N adaptateurs — tient en trois lignes et deux exemples.

### S2 — La variante « Liste complète » (§9.4)

Le verdict du chapitre est « ops granulaires 95 % du temps ». Documenter
l'anti-pattern sur trente lignes avec du code prêt à copier travaille contre ce
verdict. Trois lignes suffisent : *à éviter, sauf petites listes stables*.

### S3 — Les tableaux à étoiles

⭐⭐⭐⭐⭐ contre ⭐⭐ n'apporte rien qu'une phrase ne dise mieux, et donne un
faux air de mesure à un jugement.

### S4 — L'hydratation (§6.5) : développer ou renvoyer

Trois lignes pour un service qui gère 42 datasets, un verrou anti-concurrence,
une fraîcheur par ferme et un écran bloquant : trop peu pour servir à quelqu'un
qui doit l'implémenter. Soit un vrai chapitre, soit un renvoi assumé vers
`docs/HYDRATION_SERVICE.md` du dépôt mobile.

---

## 4. Ordre de traitement proposé

1. **Trancher §0 (`Failure`)** — conditionne les chapitres 2, 5 et 8.
2. **A1 + A2 (observabilité)** — le manque le plus coûteux aujourd'hui, et le
   plus lié à la nature offline-first du modèle.
3. **A3, A4** — courts, tirés de bugs déjà corrigés, faciles à écrire.
4. **C1, C2** — corrections ponctuelles, sans risque.
5. **S1 → S4** — allègements rapides, à faire d'un bloc.
6. **A5, A6** — vrais chapitres à écrire, à planifier à part.

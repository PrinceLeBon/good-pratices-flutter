# 2. Réseau, erreurs & `Failure`

Trois niveaux : le **client HTTP** (timeout), l'**exception HTTP typée** (côté
provider), et la **taxonomie `Failure`** (contrat Repository → State).

## 2.1 Un seul client HTTP avec timeout — `ApiHttp`

**Tous** les appels passent par un client unique avec **timeout**. Sinon, quand
l'interface est up mais le serveur injoignable, l'appel pend jusqu'au timeout TCP
de l'OS et **gèle l'UI**.

```dart
import 'package:http/http.dart' as http;

class ApiHttp {
  ApiHttp._();
  static const Duration defaultTimeout = Duration(seconds: 10);

  static Future<http.Response> get(Uri url,
          {Map<String, String>? headers, Duration? timeout}) =>
      http.get(url, headers: headers).timeout(timeout ?? defaultTimeout);
  // post, put, patch, delete : idem.
}
```

## 2.2 Exception HTTP typée — `HttpException` (côté provider)

Le provider lève **cette** classe sur non-2xx : elle porte le **code HTTP**.

```dart
class HttpException implements Exception {
  final int statusCode;
  final String message;
  final String? body;
  HttpException({required this.statusCode, required this.message, this.body});

  bool get isServerError => statusCode >= 500 && statusCode < 600;
  bool get isClientError => statusCode >= 400 && statusCode < 500;

  @override
  String toString() => 'HttpException: $message (Status Code: $statusCode)';
}
```

Provider (lecture) :

```dart
Future<List<Aliment>> getAliments(String fermeId, String token) async {
  final res = await ApiHttp.get(Uri.parse('$_baseUrl/$fermeId'),
      headers: {'Authorization': 'Bearer $token'});
  if (res.statusCode >= 200 && res.statusCode < 300) {
    return (json.decode(res.body) as List).map((e) => Aliment.fromMap(e)).toList();
  }
  throw HttpException(
      statusCode: res.statusCode, message: 'Failed to load aliments', body: res.body);
}
```

**Piège du `catch` qui écrase le type** : rattraper d'abord `HttpException`.

```dart
try { ... } on HttpException { rethrow; } catch (e) { throw Exception('Réseau: $e'); }
```

**Interdits** : `throw Exception('Failed: ${res.body}')` (perd le code) ; classe
`HttpException` dupliquée.

## 2.3 Détection réseau — `isNetworkError`

Une **seule** fonction, jamais recopiée, jamais de `.contains('SocketException')`.

```dart
bool isNetworkError(Object e) {
  final m = e.toString().toLowerCase();
  return m.contains('failed host lookup') || m.contains('network') ||
      m.contains('connection') || m.contains('socket') ||
      m.contains('timeout') || m.contains('unreachable') ||
      m.contains('no address associated');
}
```

## 2.4 Taxonomie `Failure` (contrat Repository → State)

Le Repository ne renvoie **jamais** un `String` d'erreur nu (le state layer
devrait faire du string-matching). Il renvoie un **`Failure` typé** — petit,
4-5 cas :

```dart
sealed class Failure {
  final String message;
  const Failure(this.message);
}
class NetworkFailure extends Failure { const NetworkFailure([super.m = 'Pas de connexion']); }
class ServerFailure  extends Failure { final int? code; const ServerFailure(super.m, {this.code}); } // 5xx
class AuthFailure    extends Failure { const AuthFailure([super.m = 'Session expirée']); }           // 401/403
class ClientFailure  extends Failure { final int? code; const ClientFailure(super.m, {this.code}); } // autres 4xx
class CacheFailure   extends Failure { const CacheFailure([super.m = 'Erreur locale']); }
class UnknownFailure extends Failure { const UnknownFailure(super.m); }
```

**Mapping erreur → `Failure`** (fait dans le Repository / `readOnlineFirst`) :

```dart
Failure toFailure(Object e) {
  if (isNetworkError(e)) return const NetworkFailure();
  if (e is HttpException) {
    if (e.isServerError) return ServerFailure(e.message, code: e.statusCode);
    if (e.statusCode == 401 || e.statusCode == 403) return AuthFailure(e.message);
    if (e.isClientError) return ClientFailure(e.message, code: e.statusCode);
  }
  return UnknownFailure(e.toString());
}
```

> `Failure.message` donne toujours un texte affichable → l'affichage simple reste
> trivial, tout en permettant au state layer de **réagir par type** (voir
> [08](08-etat-agnostique.md)).

## 2.5 Les 3 couches de détection réseau

1. **Check interface** (`connectivity_plus`) — rapide, mais peut mentir.
2. **Tenter l'appel + timeout** (`ApiHttp`) — le signal fiable.
3. **Sonde serveur** (ping court) — uniquement à la reconnexion (voir [07](07-connectivite.md)).

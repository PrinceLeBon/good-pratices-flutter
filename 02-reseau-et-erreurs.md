# 2. Réseau & erreurs

## 2.1 Un seul client HTTP avec timeout — `ApiHttp`

**Tous** les appels réseau passent par un client unique qui impose un **timeout**.

Pourquoi : quand l'interface réseau est active mais le serveur injoignable
(backend éteint, route morte, portail captif), un appel `http` sans timeout pend
jusqu'au timeout TCP de l'OS (très long) et **gèle l'UI**. Avec un timeout,
l'appel lève une `TimeoutException` rapidement → classée comme erreur réseau →
repli offline.

```dart
import 'package:http/http.dart' as http;

class ApiHttp {
  ApiHttp._();
  static const Duration defaultTimeout = Duration(seconds: 10);

  static Future<http.Response> get(Uri url,
          {Map<String, String>? headers, Duration? timeout}) =>
      http.get(url, headers: headers).timeout(timeout ?? defaultTimeout);

  static Future<http.Response> post(Uri url,
          {Map<String, String>? headers, Object? body, Duration? timeout}) =>
      http.post(url, headers: headers, body: body).timeout(timeout ?? defaultTimeout);

  // put, patch, delete : idem.
}
```

> Règle : `grep "http\.(get|post|put|patch|delete)"` ne doit rien renvoyer hors
> de `ApiHttp`.

## 2.2 Une seule exception HTTP typée — `HttpException`

Les providers lèvent **cette** classe sur non-2xx. Elle porte le **code HTTP**,
indispensable pour distinguer « serveur en panne » (5xx) de « erreur client »
(4xx).

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

**Provider — modèle de lecture :**

```dart
Future<List<Aliment>> getAliments(String fermeId, String token) async {
  final res = await ApiHttp.get(Uri.parse('$_baseUrl/$fermeId'),
      headers: {'Authorization': 'Bearer $token'});
  if (res.statusCode >= 200 && res.statusCode < 300) {
    final data = json.decode(res.body) as List;
    return data.map((e) => Aliment.fromMap(e)).toList();
  }
  throw HttpException(
      statusCode: res.statusCode, message: 'Failed to load aliments', body: res.body);
}
```

**Piège — le `catch` qui écrase le type.** Si un provider enveloppe tout en
« erreur réseau » dans un `catch`, la `HttpException` (donc le code) est perdue.
Rattraper d'abord le type :

```dart
try {
  ...
  throw HttpException(statusCode: res.statusCode, message: '...', body: res.body);
} on HttpException {
  rethrow; // conserver le code HTTP
} catch (e) {
  throw Exception('Erreur réseau: $e'); // wrap uniquement les vraies erreurs réseau
}
```

**Interdits :** `throw Exception('Failed: ${res.body}')` (perd le code) ; une
classe `HttpException` dupliquée par fichier.

## 2.3 Une seule détection réseau — `isNetworkError`

Une **seule** fonction, jamais recopiée, jamais de bloc `.contains(...)` inline.

```dart
bool isNetworkError(Object error) {
  final msg = error.toString().toLowerCase();
  return msg.contains('failed host lookup') ||
      msg.contains('network') ||
      msg.contains('connection') ||
      msg.contains('socket') ||
      msg.contains('timeout') ||
      msg.contains('unreachable') ||
      msg.contains('no address associated');
}

bool isServerError(Object error) =>
    error is HttpException && error.isServerError; // 5xx
```

## 2.4 Comment on détecte réellement une coupure — 3 couches

1. **Check d'interface** (`connectivity_plus`) : y a-t-il du wifi/data ?
   Rapide **mais peut mentir** (interface up ≠ serveur joignable). Ne jamais s'y
   fier seul.
2. **Tenter l'appel + timeout** (`ApiHttp`) : le signal **fiable**. On a la vraie
   réponse en essayant ; un serveur muet → `TimeoutException`.
3. **Sonde serveur** (ping HTTP court) : uniquement **à la reconnexion** pour
   décider quand relancer la sync — pas avant chaque appel (latence + course).

> La combinaison « check interface → tenter avec timeout → lire l'erreur »
> rend le repli local fiable et le faux positif du check d'interface inoffensif.

# 7. Service de connectivité

Deux sondes de nature différente. Ne pas confondre.

## 7.1 `checkConnection()` — interface seulement (rapide)

Répond « y a-t-il une interface réseau active ? » (wifi/data). **Rapide mais peut
mentir** : wifi allumé ≠ internet/serveur joignable (portail captif, serveur
éteint).

Utilisé comme **1er filtre** avant chaque appel. Le faux positif est rendu
inoffensif par le timeout + repli local (cf. [02](02-reseau-et-erreurs.md)).

```dart
class ConnectivityService {
  final Connectivity _connectivity = Connectivity(); // connectivity_plus

  Future<bool> checkConnection() async {
    final results = await _connectivity.checkConnectivity();
    return _isConnected(results);
  }

  bool _isConnected(List<ConnectivityResult> r) =>
      r.any((x) => x != ConnectivityResult.none);
}
```

## 7.2 `hasServerConnection()` — serveur réellement joignable (sonde HTTP)

Répond « le backend répond-il vraiment ? » via un petit appel HTTP avec timeout
court. Plus fiable mais plus lent → **à ne pas faire avant chaque appel**.

```dart
Future<bool> hasServerConnection() async {
  try {
    final res = await ApiHttp.get(Uri.parse('$baseUrl/health'),
        timeout: const Duration(seconds: 4));
    return res.statusCode >= 200 && res.statusCode < 500;
  } catch (_) {
    return false;
  }
}
```

## 7.3 Reconnexion → déclencher la sync (au bon moment)

À la remontée de l'interface, DNS/route ne sont pas forcément prêts. **Sonder le
serveur** avant de signaler « online » pour la sync, sinon on lance un replay qui
échouera (faux succès de reconnexion).

```dart
_connectivity.onConnectivityChanged.listen((results) async {
  final isOnline = _isConnected(results);
  if (_wasOffline && isOnline) {
    _wasOffline = false;
    // sonde le serveur, puis, s'il répond, notifie → SyncService.performSync()
    if (await hasServerConnection()) _notifyServerReachable();
  }
  if (!isOnline) _wasOffline = true;
});
```

## 7.4 Règle d'or

- **Lectures/écritures** : `checkConnection()` (rapide) + timeout `ApiHttp` +
  `isNetworkError` → repli local fiable.
- **Décider de rejouer la file de sync** : `hasServerConnection()` (sonde) pour
  ne pas déclencher un replay prématuré.

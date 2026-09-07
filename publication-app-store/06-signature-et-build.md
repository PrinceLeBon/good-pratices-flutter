# 6. Signature et build

Ces problèmes n'ont rien à voir avec la revue, mais ils coûtent des heures parce
qu'ils **échouent sans bruit**.

## 6.1 Le certificat Distribution

Un export App Store exige un certificat **Apple Distribution**. Une identité
*Apple Development* laisse l'archive se construire, puis échoue à l'export :

```
error: exportArchive No Accounts
error: exportArchive No signing certificate "iOS Distribution" found
```

Vérification **avant** de lancer un build de dix minutes :

```
security find-identity -v -p codesigning
```

La sortie doit lister une identité `Apple Distribution: …`. Si seule
`Apple Development` apparaît, c'est que le compte n'est pas connecté dans Xcode :
*Settings → Accounts → +*, puis *Manage Certificates → + → Apple Distribution*.

> Ne te fie pas à la clé `IDEProvisioningTeams` des préférences Xcode : elle est
> absente sur les versions récentes même lorsqu'un compte est bien connecté. Le
> seul signal fiable est la présence du certificat.

## 6.2 Le code de sortie ment

```
flutter build ipa --release
```

**retourne `0` même quand l'export échoue.** L'archive Xcode est produite,
l'`.ipa` non. Ne te fie jamais au code de sortie :

```
ls build/ios/ipa/*.ipa   → doit exister
```

Conséquence directe : un `cmd_ipa && cmd_aab` enchaîne le build Android sur un
export iOS raté sans que rien ne le signale.

## 6.3 Trois réflexes

- **Commiter `DEVELOPMENT_TEAM`** dans `ios/Runner.xcodeproj/project.pbxproj`.
  Xcode l'ajoute lors de la connexion du compte ; c'est ce réglage qui débloque
  la signature, et sans lui le build n'est pas reproductible sur une autre
  machine.
- **`flutter clean` supprime `build/`**, donc l'IPA et l'AAB qu'on vient de
  produire. Les mettre de côté avant, ou ne pas nettoyer entre deux cibles.
- **Un numéro de build ne se réutilise jamais.** L'App Store refuse un doublon.
  Incrémenter à chaque upload, même pour corriger une virgule.

## 6.4 Vérifier la version réellement embarquée

La version affichée par le log de build n'est pas une preuve. La source de
vérité est l'archive :

```
plutil -p build/ios/archive/Runner.xcarchive/Products/Applications/*.app/Info.plist \
  | grep -iE "CFBundleShortVersionString|CFBundleVersion"
```

Utile quand le courriel d'Apple annonce une version qui ne correspond pas à ce
qu'on croit avoir envoyé.

## 6.5 Cible de déploiement

Relever `IPHONEOS_DEPLOYMENT_TARGET` se répercute à trois endroits :
`ios/Podfile` (`platform :ios, 'X.0'`), le projet Xcode, et les lockfiles
CocoaPods. Les trois se commitent ensemble, sinon le build diverge d'une machine
à l'autre.

## 6.6 Uploader

Deux voies :

- **Transporter** (app macOS) — glisser l'`.ipa`, authentification par le compte
  déjà connecté à Xcode. Le plus simple pour un envoi ponctuel.
- **`xcrun altool --upload-app`** avec une clé API App Store Connect
  (*Users and Access → Integrations*). Le `.p8` se dépose dans
  `~/.appstoreconnect/private_keys/` et ne se télécharge **qu'une seule fois**.
  À privilégier dès qu'on automatise.

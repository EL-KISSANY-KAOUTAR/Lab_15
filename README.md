# LAB 15 : Analyse Dynamique Android : Inspection TLS/HTTPS et Gestion du SSL Pinning


## 1. Contexte

Ce laboratoire a pour objectif d'analyser dynamiquement une application Android afin d'inspecter le trafic HTTPS, comprendre le mécanisme de SSL Pinning et mettre en œuvre des techniques de bypass pour observer les communications chiffrées. 

## Objectifs du lab

* Installer et configurer Frida
* Déployer Frida Server sur Android
* Configurer un proxy TLS (Burp Suite ou mitmproxy)
* Installer un certificat CA sur l'appareil
* Comprendre le fonctionnement du SSL Pinning
* Réaliser un bypass du SSL Pinning
* Capturer et analyser le trafic HTTPS


## Prérequis

### Matériel

* PC Windows/Linux/macOS
* Émulateur Android ou appareil réel

### Logiciels

* Python 3.8+
* ADB (Android Platform Tools)
* Frida
* Frida Server
* Burp Suite Community/Professional
* Android Studio 

---
1.1 Installer Frida (PC) et démarrer frida-server (Android)

<img width="644" height="326" alt="image" src="https://github.com/user-attachments/assets/a5184d4b-f563-4dcb-9aab-70bd55ba7d41" />

<img width="625" height="97" alt="image" src="https://github.com/user-attachments/assets/3bff5f54-4a70-4d0a-8f40-abfaffaa1255" />

1.2 Préparer ADB et l’appareil

<img width="599" height="97" alt="image" src="https://github.com/user-attachments/assets/3ea265c3-5209-4a94-8911-e39bce4d73d0" />

## 1.3 Déployer et lancer Frida Server

### a) Identifier l'architecture CPU de l'appareil

Avant de télécharger Frida Server, il est nécessaire d'identifier l'architecture CPU de l'émulateur ou de l'appareil Android.

Commande :

```bash
adb shell getprop ro.product.cpu.abi
```

Exemple :

![Architecture CPU](https://github.com/user-attachments/assets/41adf1fa-074c-4411-8131-e4ed638fec99)

Le résultat obtenu permettra de choisir la version appropriée de **Frida Server** (arm64-v8a, armeabi-v7a, x86 ou x86_64).

---

### b) Télécharger Frida Server

Télécharger la version correspondant :

- à la version de Frida installée sur le poste de travail ;
- à l'architecture CPU de l'appareil Android.

Vérification de la version installée :

```bash
frida --version
```

Téléchargement :

👉 https://github.com/frida/frida/releases

Sélectionner le fichier :

```text
frida-server-<version>-android-<architecture>.xz
```

Exemple :

```text
frida-server-17.2.17-android-x86_64.xz
```

---

### c) Décompresser l'archive

Sous Windows, utiliser un outil comme :

- 7-Zip
- WinRAR

Décompresser l'archive `.xz` afin d'obtenir le binaire :

```text
frida-server
```

---

### d) Transférer Frida Server vers l'appareil

Copier le binaire sur l'appareil Android :

```bash
adb push frida-server /data/local/tmp/
```

Attribuer les permissions d'exécution :

```bash
adb shell chmod 755 /data/local/tmp/frida-server
```

Lancer Frida Server :

```bash
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"
```

---

### e) Redirection des ports (optionnel)

Dans certains environnements, il peut être nécessaire de rediriger les ports utilisés par Frida :

```bash
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
```

---

### f) Vérification du fonctionnement

Vérifier que Frida communique correctement avec l'appareil :

```bash
frida-ps -Uai
```

Exemple de résultat :

```text
PID     Name
----    --------------------
1234    com.android.chrome
5678    com.example.app
```

---

### ⚠️ Remarque importante

La version de **Frida Client** installée sur le PC doit être **strictement identique** à la version de **Frida Server** déployée sur l'appareil Android.

Vérification :

```bash
frida --version
```

Une incompatibilité de versions provoquera des erreurs de connexion entre le client et le serveur.

---

## 1.4 Configuration du Proxy TLS et installation du certificat CA

Afin d'intercepter et d'analyser le trafic HTTPS de l'application Android, un proxy TLS doit être configuré.

### Configuration de Burp Suite

Ouvrir **Burp Suite** puis :

1. Aller dans l'onglet **Proxy**
2. Vérifier que le Proxy Listener est actif
3. Vérifier l'adresse d'écoute

Valeur par défaut :

```text
127.0.0.1:8080
```

L'option **Intercept** permet d'activer ou désactiver l'interception des requêtes HTTP/HTTPS.

---

### Installation du certificat CA

Depuis le navigateur Android :

Accéder à l'une des adresses suivantes :

```text
http://burp
```

ou

```text
http://mitm.it
```

Télécharger le certificat CA correspondant à Android.

Installer ensuite le certificat comme :

```text
Certificat CA utilisateur
```

Illustration :

![Installation certificat](https://github.com/user-attachments/assets/c6c03391-2e44-40d6-ae22-7d527e265180)

---

### Vérification

Après installation :

- Configurer le proxy Wi-Fi de l'appareil.
- Ouvrir un navigateur Android.
- Accéder à un site HTTPS.

Si tout est correctement configuré, les requêtes apparaissent dans Burp Suite.

---

## 1.5 Bypass du SSL Pinning avec Frida

Certaines applications implémentent un mécanisme de **SSL Pinning** afin d'empêcher l'interception du trafic HTTPS même lorsqu'un certificat CA utilisateur est installé.

Pour contourner cette protection, un script Frida peut être injecté dans l'application cible.

---

### Script universel Java

Le script suivant applique plusieurs hooks afin de neutraliser les mécanismes de validation TLS les plus courants :

- SSLContext
- TrustManager
- Conscrypt
- OkHttp
- WebView

Illustration :

![SSL Pinning Script](https://github.com/user-attachments/assets/8c76422f-de34-4d7f-af8e-82556d2c076a)

---

### Injection du script

Identifier le PID de l'application :

```bash
frida-ps -Uai
```

Injecter ensuite le script :

```bash
frida -U -p <PID> -l sslpin_bypass_universal.js
```

Exemple :

```bash
frida -U -p 13686 -l sslpin_bypass_universal.js
```

---

### Résultat attendu

Après injection, Frida affiche les hooks appliqués :

```text
[+] SSL bypass: SSLContext.init patched
[+] SSL bypass: X509TrustManager patches attempted
[+] SSL bypass: TrustManagerImpl patched
[+] Universal SSL pinning bypass installed
```

Illustration :

![Bypass SSL Pinning](https://github.com/user-attachments/assets/19263d7b-cb0e-4cab-9d1f-9f5d7275bb28)

---

### Validation

Une fois le bypass appliqué :

✅ Les connexions HTTPS ne sont plus bloquées.

✅ Les requêtes apparaissent dans Burp Suite.

✅ Le contenu HTTP/HTTPS peut être inspecté et analysé.

Cette étape valide le bon fonctionnement de l'analyse dynamique TLS/HTTPS de l'application Android.


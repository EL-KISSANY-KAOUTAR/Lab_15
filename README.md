# LAB 15 : Analyse Dynamique Android : Inspection TLS/HTTPS et Gestion du SSL Pinning

## Objectifs du lab

Installer et vérifier Frida côté PC et frida-server sur Android.
Mettre en place un proxy (Burp Suite ou mitmproxy) et un certificat CA sur l’appareil.
Neutraliser l’SSL pinning via hooks Java (TrustManager/Conscrypt/OkHttp/WebView).
Diagnostiquer et compléter le bypass si le pinning est natif (BoringSSL/OpenSSL).
Valider en capturant le trafic HTTPS déchiffré.

## Prérequis

Avant de commencer, assurez-vous de disposer des éléments suivants :

Un ordinateur sous Windows (avec PowerShell), macOS ou Linux.
Python 3.8 ou une version plus récente, ainsi que l’outil pip pour la gestion des paquets Python.
Les Android Platform Tools (ADB) installés sur la machine.
Frida installé sur le poste de travail et frida-server déployé sur l’appareil Android, avec des versions strictement identiques.
Un appareil Android 8.0 ou supérieur sur lequel l’option Débogage USB est activée.
Un outil de proxy TLS, tel que Burp Suite Community, Burp Suite Professional ou mitmproxy, installé et configuré sur l’ordinateur afin d’analyser le trafic réseau.

#### Installer Frida (PC) et démarrer frida-server (Android)

<img width="644" height="326" alt="image" src="https://github.com/user-attachments/assets/a5184d4b-f563-4dcb-9aab-70bd55ba7d41" />

<img width="625" height="97" alt="image" src="https://github.com/user-attachments/assets/3bff5f54-4a70-4d0a-8f40-abfaffaa1255" />

1.2 Préparer ADB et l’appareil

<img width="599" height="97" alt="image" src="https://github.com/user-attachments/assets/3ea265c3-5209-4a94-8911-e39bce4d73d0" />

1.3 Déployer et lancer frida-server  

1. Identifier l’architecture CPU de l’appareil :
   
<img width="648" height="78" alt="image" src="https://github.com/user-attachments/assets/41adf1fa-074c-4411-8131-e4ed638fec99" />

2. Télécharger Frida-server :
Récupérer depuis la page officielle :
https://github.com/frida/frida/releases

Choisir le fichier :
frida-server-<version>-android-<arch>.xz
en fonction de la version installée de Frida sur le PC (frida --version) et de l’architecture CPU.

3. Décompression :
Décompresser l’archive (par exemple avec 7-Zip sous Windows), puis obtenir le binaire frida-server.

4. Transfert et exécution sur l’appareil :

adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"

5. (Optionnel) Redirection des ports :

adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

6. Vérification du bon fonctionnement :

frida-ps -Uai

2. Télécharger Frida-server :
Récupérer depuis la page officielle :
https://github.com/frida/frida/releases

Choisir le fichier :
frida-server-<version>-android-<arch>.xz
en fonction de la version installée de Frida sur le PC (frida --version) et de l’architecture CPU.

3. Décompression :
Décompresser l’archive (par exemple avec 7-Zip sous Windows), puis obtenir le binaire frida-server.

4. Transfert et exécution sur l’appareil :

adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server -l 0.0.0.0"

5. (Optionnel) Redirection des ports :

adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043

6. Vérification du bon fonctionnement :

frida-ps -Uai

Remarque importante :
La version de frida-client (PC) doit être strictement identique à celle de frida-server, sinon la connexion échoue.

#### Mettre en place le proxy et le certificat CA

Burp Suite : aller dans l’onglet Proxy et activer/désactiver l’option Intercept.
Vérifier l’adresse et le port utilisés (par défaut : 127.0.0.1:8080).

Installer la CA proxy sur l’appareil
Depuis le navigateur du téléphone : 

Accéder à :
http://burp (Burp Suite)
ou http://mitm.it (mitmproxy)
Télécharger le certificat CA.
Installer le certificat en tant que certificat CA utilisateur.

<img width="202" height="182" alt="image" src="https://github.com/user-attachments/assets/c6c03391-2e44-40d6-ae22-7d527e265180" />

#### Script « universel » Java pour bypass SSL pinning

<img width="862" height="437" alt="image" src="https://github.com/user-attachments/assets/8c76422f-de34-4d7f-af8e-82556d2c076a" />

<img width="1539" height="1022" alt="image" src="https://github.com/user-attachments/assets/ea53fff4-dacc-4286-8108-c5d3b4330a88" />







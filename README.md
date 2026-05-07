# Contournement de SSL Pinning sur Android
> **Auteur :** Ahmed  
> **Outils utilisés :** Frida, Objection, Burp Suite, ADB.

##  Objectif
L'objectif de ce lab est de mettre en place un environnement d'interception de trafic HTTPS pour une application Android protégée par le **SSL Pinning**, en utilisant l'instrumentation dynamique.

---

##  Étape 1 : Préparation du PC
Installation des outils nécessaires via Python/Pip.

```powershell
# Installation des outils
pip install --upgrade objection frida frida-tools
````
# Vérification des versions
objection --version
frida --version

## Étape 2 : Préparation de l'appareil (Frida-Server)
Configuration du "moteur" Frida sur le téléphone pour permettre l'injection de scripts.
Identification de l'architecture :
code
````Powershell
adb shell getprop ro.product.cpu.abi
````
Installation du serveur :
Téléchargement de frida-server-16.2.1-android-[arch].xz.
Décompression et transfert sur l'appareil.

```` Powershell
adb push frida-server /data/local/tmp/
adb shell chmod 755 /data/local/tmp/frida-server
adb shell "/data/local/tmp/frida-server"
````
Vérification de la liaison :

````Powershell
frida-ps -Uai
````
<img width="1000" height="671" alt="Capture d&#39;écran 2026-05-07 192949" src="https://github.com/user-attachments/assets/5a37c87f-d725-4733-9da0-af7a4da8569a" />

## Étape 3 : Configuration du Proxy (MITM)
Interception du trafic réseau via Burp Suite.
Configuration Wi-Fi : Mise en place du proxy manuel sur le téléphone (IP du PC, Port 8080).
Installation de la CA : Téléchargement du certificat via http://burp et installation dans les certificats de confiance de l'appareil.
Test : Validation du flux HTTP via le navigateur du téléphone.
<img width="1097" height="270" alt="Capture d&#39;écran 2026-05-07 192744" src="https://github.com/user-attachments/assets/75ed18e3-9fd4-46b1-ab97-61834677ad5d" />
<img width="305" height="485" alt="Capture d&#39;écran 2026-05-07 192756" src="https://github.com/user-attachments/assets/b8783d37-3b68-4b44-892a-5de0d90de8e8" />

Étape 4 : Injection et Bypass du SSL Pinning
Utilisation d'Objection pour désactiver les vérifications de certificats au démarrage de l'application.

````Powershell
# Commande de lancement (Spawn)
objection -g  com.android.chrome explore --startup-command "android sslpinning disable"
````
Résultats obtenus :
.L'application est lancée avec succès.

.Objection a injecté des "hooks" sur les classes OkHttp, TrustManager et WebView.

.Le SSL Pinning est neutralisé dynamiquement en mémoire sans modifier le fichier APK.

## Étape 5 : Validation et Analyse
Le trafic HTTPS de l'application cible est désormais visible en clair dans l'onglet HTTP History de Burp Suite.
Livrables validés :
.Communication Frida PC <-> Mobile établie.

.Certificat Burp Suite installé.

.Hooking SSL réussi via Objection.

.Requêtes API interceptées (JSON/Headers visibles).

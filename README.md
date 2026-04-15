# FireStorm-R-solution-d-taill-e-tape-par-tape
## 1. Introduction
Ce projet présente la résolution du challenge **Firestorm**. L'objectif était d'extraire un "Flag" sécurisé dans une base de données Firebase. La sécurité de l'application repose sur un mot de passe dynamique généré par une combinaison de ressources Java et de bibliothèques natives.

---

## 2. Analyse du Vecteur d'Attaque

### Analyse Statique (JADX-GUI)
En analysant l'APK avec JADX, nous avons identifié une méthode critique dans `MainActivity` :
* **Méthode :** `public String Password()`
* **Fonctionnement :** Elle concatène des chaînes de caractères issues de `strings.xml` (`s1`, `s2`, etc.) avec un segment généré par une fonction native `generateRandomStrings()` située dans `libfirestorm.so`.
* **Problème :** Cette méthode n'est jamais appelée dans le flux d'exécution standard de l'application. Elle est "dormante".

### Analyse des Ressources
Le fichier `strings.xml` contient les configurations Firebase nécessaires :
- `firebase_api_key` : Clé d'accès à l'API.
- `firebase_email` : Identifiant de l'utilisateur technique.
- `firebase_database_url` : Point de terminaison de la base de données.

---

## 3. Phase d'Instrumentation (Frida)

Puisque la méthode `Password()` n'est pas appelée par l'application, nous utilisons **Frida** pour forcer son exécution en mémoire (Hooking).

### Explication du Script (`frida_firestorm.js`)

Java.perform(function() {
    // On attend que l'application soit totalement chargée
    setTimeout(function() {
        // Java.choose permet de scanner la mémoire pour trouver l'instance active de MainActivity
        Java.choose('com.pwnsec.firestorm.MainActivity', {
            onMatch: function(instance) {
                // Appel manuel de la méthode Password() sur l'instance trouvée
                var pass = instance.Password();
                console.log("[***] PASSWORD OBTENU : " + pass);
            },
            onComplete: function() {}
        });
    }, 5000); // Délai de 5 secondes pour éviter les crashs (ANR)
});
Résultat : Le script nous a permis d'extraire le mot de passe dynamique suivant :

C7_dotpsC7t7f_._In_i.IdttpaofoaIIdIdnndIfC

4. Exploitation Finale (Python)
Une fois le mot de passe récupéré, nous avons automatisé la connexion à la base de données Firebase via un script Python utilisant pyrebase4.

Fonctionnement du script get_flag.py :
Initialisation : Utilise l'API Key et l'URL trouvées dans l'APK.

Authentification : Envoie l'email statique et le mot de passe dynamique (extrait via Frida).

Extraction : Une fois authentifié, le script interroge la Realtime Database pour lire la valeur du Flag.

5. Analyse du Flag Obtenu
Flag : PWNSEC{C0ngr4ts_Th4t_w45_4N_345y_P4$$w0rd_t0_G3t!!!_0R_!5_!t???}

Signification technique :
Le flag lui-même est un message de félicitations ironique. Il confirme que :

L'authentification Firebase a été réussie avec succès.

La logique de protection native (le secret dans .so) a été contournée grâce à l'appel de fonction direct (Method Invocation) via Frida.

Le développeur pensait sécuriser l'accès en ne reliant pas la fonction au bouton, mais l'instrumentation dynamique permet de passer outre cette absence de lien logique.

6. Outils Utilisés
JADX-GUI : Analyse statique du code Java.

Frida & Frida-Server : Hooking et manipulation de la mémoire.

ADB (Android Debug Bridge) : Installation et communication avec l'émulateur.

Python 3 : Scripting pour l'extraction finale des données.
#les captures
<img width="821" height="659" alt="Screenshot 2026-04-15 200700" src="https://github.com/user-attachments/assets/3fdd60c6-7c44-45e4-b495-403e0a71f689" />
<img width="868" height="398" alt="Screenshot 2026-04-15 201336" src="https://github.com/user-attachments/assets/1cc715b6-3e54-4beb-831a-1e3fec2d0b60" />
<img width="956" height="146" alt="Screenshot 2026-04-15 203537" src="https://github.com/user-attachments/assets/b068c5fc-676a-474d-9c74-3d2720014c7a" />
<img width="762" height="215" alt="Screenshot 2026-04-15 204653" src="https://github.com/user-attachments/assets/5411bf15-b95f-40b0-832c-58df6817d9cc" />






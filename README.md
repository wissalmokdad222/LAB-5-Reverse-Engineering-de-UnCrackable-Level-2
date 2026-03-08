# LAB-5-Analyse de l’application Android UnCrackable-Level2

**Etudiante :** Wissal MOKDAD  
**Date :** 08/03/2026  
**Outils :** ADB, JADX, Ghidra, Python, PowerShell

---

# Partie 1 — Découverte de l’application

## Étape 1 — Installer et lancer l’APK

### Action
![](https://github.com/user-attachments/assets/68012337-2d4d-49fd-bfe2-50d6b303507c)
![](https://github.com/user-attachments/assets/faa4b735-be73-4287-baa2-5455d61cb42f)
![](https://github.com/user-attachments/assets/9299e49d-eed9-495a-97a8-20cca11b43a7)
![](https://github.com/user-attachments/assets/d19352ac-136b-4780-abcf-1086ac17afa2)
# Partie 2 — Trouver où commence la vérification

## Étape 3 — Décompiler l’APK avec JADX

Pour comprendre la logique Java, il faut ouvrir l’APK dans un décompilateur comme **JADX**.

### Action

Lancer JADX :

```bash
jadx-gui UnCrackable-Level2.apk
```
![](https://github.com/user-attachments/assets/93af3994-079c-4009-87af-b3332b77d82d)
## Étape 4 — Repérer l’appel de validation dans MainActivity
Code extrait de MainActivity
![](https://github.com/user-attachments/assets/4954a4f2-5d7a-4060-a9f6-e175f3b2150d)
# Partie 3 — Comprendre le rôle de CodeCheck
## Étape 5 — Identifier la classe qui effectue la vérification
![](https://github.com/user-attachments/assets/9753d22d-a19b-4cbd-a230-5cecd34986fb)
# Partie 4 — Retrouver la bibliothèque native
## Étape 6 — Extraire le contenu de l’APK
Action PowerShell
![](https://github.com/user-attachments/assets/ccb676d6-e574-45c7-b784-835eb161fb63)
### Conclusion
La bibliothèque native libfoo.so est prête pour l’analyse statique
# Partie 5 — Analyser le code natif avec Ghidra
## Étape 7 — Importer libfoo.so dans Ghidra
Action
### Lancer Ghidra :
![](https://github.com/user-attachments/assets/043fdd64-5c3a-46ca-8ea6-e91c2c77cee3)

### Créer un nouveau projet
![](https://github.com/user-attachments/assets/b54f3c2a-829a-4d77-a54f-45e95d85f367)
### Importer :libfoo.so
![](https://github.com/user-attachments/assets/a7514fca-d454-481c-b46d-e7a31c7cd676)
### Lancer Auto Analysis
Observation
Ghidra génère :
- le pseudo-code
- la liste des fonctions exportées
## Étape 8 — Chercher la fonction JNI liée à bar
Recherche dans Ghidra
Symbole : Java_sg_vantagepoint_uncrackable2_CodeCheck_bar
![](https://github.com/user-attachments/assets/820b1d66-ae3d-4199-953a-6a1cae8a0ba5)
Observation
Cette fonction correspond à l’implémentation native de :CodeCheck.bar()
# Partie 6 — Comprendre la comparaison avec strncmp
## Étape 9 — Lire le pseudo-code de la fonction native
![](https://github.com/user-attachments/assets/3d1f3cf1-8459-4ee7-8d06-4c3e7352d01f)
### Observation :

La fonction JNI Java_sg_vantagepoint_uncrackable2_CodeCheck_bar prend en paramètre l’entrée utilisateur.

À l’intérieur, une variable local_30 est initialisée avec la chaîne :

"Thanks for all the fish"

La comparaison est faite via strncmp :

strncmp(__s1, local_30, 0x17);

Ici, __s1 correspond à l’entrée utilisateur convertie en chaîne C.

Si les 23 caractères correspondent, la fonction retourne 1 → succès. Sinon, elle retourne 0 → échec.

### Analyse :

La chaîne secrète est stockée dans la bibliothèque native, pas dans le code Java.

La logique de vérification n’est donc pas visible directement dans MainActivity.

Dans ce cas précis, la chaîne est en clair, donc il n’est pas nécessaire de décoder un hexadécimal.

### Résultat / Conclusion :

Le secret attendu par l’application est :

Thanks for all the fish

Cette information permet de passer à l’étape suivante : valider le secret dans l’application pour obtenir le message de succès.
## Partie 6 — Comprendre la comparaison avec `strncmp`

### Étape 9 — Lire le pseudo-code de la fonction native

**Objectif :**  
Identifier comment la fonction native `bar` compare l’entrée utilisateur au secret.

**Action :**  
- Analyser le pseudo-code dans Ghidra.  
- Rechercher l’appel à `strncmp`.

**Observation :**  
- La fonction `bar` utilise `strncmp` pour comparer l’entrée à une chaîne stockée localement (`local_30`).  
- La comparaison se fait sur 23 caractères.

**Explication :**  
- `strncmp` compare deux chaînes sur une longueur donnée.  
- Si elles correspondent, la vérification réussit, ce qui indique que la chaîne secrète est définie dans le code natif.
![](https://github.com/user-attachments/assets/305389b9-aa45-4a59-9f09-4510903f5683)
## Partie 7 — Décoder le secret

**Objectif :** Décoder la chaîne secrète stockée dans la bibliothèque native et obtenir le secret final.

**Étapes :**  
1. Convertir l’hexadécimal en ASCII avec Python :

```python
hex_data = "6873696620656874206c6c6120726f6620736b6e616854"
ascii_str = bytes.fromhex(hex_data).decode("ascii")
print(ascii_str)  # hsif eht lla rof sknahT
```
2. Inverser la chaîne pour obtenir le secret lisible :
```python
secret = ascii_str[::-1]
print(secret)  # Thanks for all the fish
```
![](https://github.com/user-attachments/assets/b2a4807e-510f-4254-b02d-55c7cbf0851e)
Explication :

Chaque paire hex correspond à un caractère ASCII.

La chaîne obtenue est inversée dans le code natif pour cacher le secret.

L’inversion permet de retrouver la phrase finale à entrer dans l’application.

Résultat final : Thanks for all the fish

# 📖 Résumé d’utilisation de DTL-X

## 🔎 Contexte

**DTL-X** est un outil en Python qui permet de **décompiler, modifier puis recompiler** un APK.  
Chaque option applique un patch spécifique (suppression de pubs, trackers, restrictions, etc.).  

Le workflow est toujours le même :

`
APK → Décompilation (apktool + smali) → Modifications → Recompilation → APK patché
`

---

# example

```python
python3 dtlx.py --rmads1 --rmads3 --rmads4 --rmtrackers --paidkw --patch README_PATCH.md --fixinstall lab.apk
```


- `--rmads*` : coupe les pubs
 
- `--rmtrackers` : enlève les SDK analytics
 
- `--paidkw` : scanne les options payantes
 
- `--patch` : applique le contournement premium/pro
 
- `--fixinstall` : assure l’installation correcte

---

## ⚙️ Options principales

| Option | Action technique | Résultat attendu | À combiner avec… |
|--------|-----------------|------------------|------------------|
| `--rmads1` | Remplace dans le manifeste/librairies les références pub (ex: `com.google.android.gms.ad`) | Supprime une partie des SDK de pubs | `--rmads3`, `--rmads4` |
| `--rmads2` | Retire la permission `INTERNET` du manifeste | Empêche toute pub réseau (⚠️ casse parfois des fonctions en ligne) | — |
| `--rmads3` | Remplace les identifiants AdMob (`ca-app-pub`) par une valeur neutre | Bloque le chargement des pubs AdMob | `--rmads1`, `--rmads4` |
| `--rmads4` | Désactive via un dictionnaire les loaders de pubs (puissant) | Coupe la majorité des pubs multi-SDK | Les trois autres `--rmads` |
| `--rmtrackers` | Supprime/neutralise les SDK de tracking (analytics, etc.) | Réduction du suivi utilisateur | `--rmads4` |
| `--paidkw` | Recherche des chaînes liées aux achats in-app (SKU, billing, premium, pro) | Repérage des mécanismes payants pour analyse | `--findstring` |
| `--findstring` | Recherche une chaîne personnalisée (regex ou mot) dans le code smali/resources | Détection ciblée (utile pour rapport) | Toujours utile avant patch |
| `--rmnop` | Supprime toutes les instructions smali `nop` | Nettoie le code (souvent ajouté par obfuscateurs) | Avec patchs forts |
| `--rmunknown` | Supprime fichiers inconnus (ex: `.properties`) | Réduction de la taille, suppression bruit | `--rmprop` |
| `--rmprop` | Supprime uniquement les fichiers `.properties` | Nettoyage ciblé | `--rmunknown` |
| `--rmcopy` | Retire la protection *AppCloner Copy Protection* | Permet d’analyser un APK cloné | — |
| `--rmpairip` | Supprime une ancienne protection Google Pairip | Contournement basique | `--bppairip` |
| `--bppairip` | Bypass Google Pairip (2024, plus efficace) | Passe les vérifs Pairip modernes | `--fixinstall` |
| `--rmvpndet` | Neutralise la détection de VPN | Permet l’usage sous VPN | `--rmxposedvpn` |
| `--rmusbdebug` | Supprime la détection du mode debug USB | Autorise le débogage | — |
| `--rmssrestrict` | Supprime les restrictions de capture d’écran | Autorise screenshots | — |
| `--rmxposedvpn` | Supprime la détection ROOT/Xposed/VPN packages | Évite crashs sur devices rootés | `--rmvpndet` |
| `--sslbypass` | Injecte un bypass SSL Pinning | Permet d’analyser le trafic HTTPS | Avec Frida/mitmproxy |
| `--rmexportdata` | Supprime notification export de données (AppCloner) | Nettoyage interface | — |
| `--fixinstall` | Répare problème d’installation d’APK modifié | Facilite la réinstallation | Après tout patch |
| `--cleanrun` | Supprime le projet décompilé après patch | Gain de place (⚠️ pas de traces) | En fin de run |
| `--noc` | *No compile* (analyse sans reconstruire APK) | Plus rapide, sûr pour repérage | `--findstring`, `--paidkw` |
| `--patch <file>` | Applique un patch défini dans un fichier (README_PATCH.md) | Personnalisation avancée | Selon patch fourni |
| `--cloneapk` | Clone APK (nouveau package name) | Installer plusieurs fois la même app | `--changepkgname` |

---

## 🔗 Combinaisons utiles (exemples pédagogiques)

### Analyse préalable (sans patcher)

```ruby
python3 dtlx.py --paidkw --findstring "premium,pro,license" --noc lab.apk
```

👉 Repère les chaînes sensibles sans toucher à l’APK.
 
### Suppression publicités

```ruby
python3 dtlx.py --rmads1 --rmads3 --rmads4 --rmtrackers lab.apk
```

👉 Neutralise pubs AdMob + autres SDK + trackers.
 
### Neutralisation restrictions device

```ruby
python3 dtlx.py --rmvpndet --rmusbdebug --rmxposedvpn lab.apk
```

👉 L’appli ne détecte plus VPN/ROOT/USB Debug.
 
### Nettoyage APK

```ruby
python3 dtlx.py --rmunknown --rmprop --rmnop lab.apk
```

👉 Allège et simplifie le code.
  
## 📝 À retenir
 
 
- Chaque option cible **un mécanisme particulier** (pubs, trackers, restrictions, protections).
 
- Les options peuvent se **compléter** (ex: pubs + trackers + restrictions).
 
- Toujours commencer par **analyse** (`--paidkw`, `--findstring`, `--noc`) avant un patch.
 
- Utiliser `--fixinstall` si l’APK patché refuse de s’installer.
 

# Analyse accélérométrique de la marche : sujet sain vs sujet avec prothèse

Calcul de la **RMS (valeur efficace) de l'accélération sur les 3 axes** pendant la marche, pour plusieurs positions du tronc (cou, épaules, lombaires), afin de comparer un sujet sain et un sujet avec prothèse et d'identifier d'éventuelles compensations.

Le traitement est réalisé dans un notebook Jupyter (`analyse_rms_marche.ipynb`) qui lit les 4 fichiers CSV bruts, isole automatiquement les phases de marche grâce à l'horodatage, puis calcule et compare les RMS.

## Contenu du dépôt

```
.
├── analyse_rms_marche.ipynb   # notebook d'analyse (à exécuter)
├── README.md
└── data/
    ├── input/                 # fichiers CSV bruts des accéléromètres
    └── output/                # résultats générés par le notebook
```

## Protocole de mesure

- **Capteurs** : accéléromètres triaxiaux, environ 100 Hz, accélérations en g.
- **Sujets** : un sujet sain et un sujet avec prothèse.
- **Enregistrement** : un enregistrement continu par capteur pendant toute la séance. Le capteur est déplacé d'une position à l'autre entre les phases de marche.
- **Marche** : à chaque position, deux allers-retours, repérés par leur heure de début et de fin.

| Fichier | Sujet | Positions successives du capteur |
|---|---|---|
| `sain_A.csv` | Sain | Cou, épaule droite, lombaire droite |
| `prothese_A.csv` | Avec prothèse | Cou, épaule droite, lombaire droite |
| `sain_B.csv` | Sain | Lombaires (centre), épaule gauche, lombaire gauche |
| `prothese_B.csv` | Avec prothèse | Lombaires (centre), épaule gauche, lombaire gauche |

Plages horaires de marche (identiques pour les 4 fichiers) :

| Position | Aller-retour 1 | Aller-retour 2 |
|---|---|---|
| 1re position (cou ou lombaires centre) | 10:02:40 – 10:04:25 | 10:07:25 – 10:09:02 |
| 2e position (épaule droite ou gauche) | 10:15:32 – 10:17:08 | 10:18:22 – 10:19:56 |
| 3e position (lombaire droite ou gauche) | 10:26:40 – 10:28:15 | 10:32:46 – 10:34:13 |

## Format des données d'entrée

Fichiers CSV **sans en-tête**, une ligne par échantillon, 4 colonnes :

```
2026-11-11 09:56:28.409,0.234375,0.328125,-0.937500
2026-11-11 09:56:28.420,0.187500,0.312500,-0.859375
```

Colonnes : date-heure, accélération x, y, z (en g).

## Méthode

1. **Lecture** du CSV et conversion de l'horodatage en secondes depuis minuit (la date est ignorée).
2. **Fréquence d'échantillonnage** estimée à partir de la médiane des écarts entre horodatages.
3. **Isolement de la marche** : seuls les échantillons situés dans les plages horaires du protocole sont conservés.
4. **Prétraitement** de chaque fenêtre : retrait de la moyenne de chaque axe (composante statique) ou, avec `FILTRE = True`, filtre passe-bande de Butterworth 0,5–20 Hz (ordre 4, phase nulle).
5. **RMS** par axe :

   `RMS = √( (1/N) × Σ (aᵢ − ā)² )`

   **RMS résultante** : `√(RMS_x² + RMS_y² + RMS_z²)`
6. **Agrégation** : RMS de chaque aller-retour, puis RMS de l'ensemble des deux allers-retours d'une position (ligne `total`).
7. **Comparaison** : rapport prothèse / sain pour chaque position.

## Installation

Prérequis : Python 3.10 ou plus récent.

```bash
# 1. Récupérer le projet
git clone https://github.com/<utilisateur>/<depot>.git
cd <depot>

# 2. Créer et activer un environnement virtuel
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS / Linux

# 3. Installer les dépendances
pip install -r requirements.txt
```

## Utilisation

1. Placer les 4 fichiers CSV dans `data/input/` avec les noms `sain_A.csv`, `sain_B.csv`, `prothese_A.csv` et `prothese_B.csv` (ou adapter le dictionnaire `FICHIERS` dans la section *Configuration* du notebook).
2. Lancer Jupyter depuis la racine du projet :

   ```bash
   jupyter notebook
   ```

   Il est aussi possible d'ouvrir le notebook directement dans VS Code en choisissant l'environnement `.venv` comme noyau.
3. Ouvrir `analyse_rms_marche.ipynb` puis exécuter toutes les cellules (*Run All*).

Seule la section *Configuration* est à adapter : noms de fichiers, colonnes, plages horaires de marche, choix du filtre.

## Résultats (`data/output/`)

| Fichier | Contenu |
|---|---|
| `rms_resultats.csv` | RMS x, y, z et résultante pour chaque sujet, position et fenêtre de marche (et `total` par position) |
| `rms_comparaison.csv` | RMS totale par position pour chaque sujet, et rapport prothèse / sain |
| `rms_comparaison.png` | Graphique comparatif sain / prothèse, un panneau par position |



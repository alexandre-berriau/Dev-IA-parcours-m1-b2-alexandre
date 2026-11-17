# Escale Atlantique — Classifieur de motifs d'insatisfaction



## Le contexte

Le service qualité veut connaître le **motif** de chaque avis négatif. L'équipe data a fait étiqueter un historique d'avis par un grand modèle de langue (LLM), une fois. Ce modèle est trop lent et trop lourd pour tourner sur chaque nouvel avis : il faut un petit classifieur, entraîné sur ces étiquettes.

## Structure

```
data/
  avis_motifs.csv     # 1 200 avis négatifs de 2016, étiquetés par le LLM : pour entraîner
  avis_recents.csv    # 180 avis de 2017, étiquetés à la main par l'équipe qualité : pour évaluer
  cache/              # tes vecteurs sauvegardés (non versionné)
models/               # le modèle de motifs que tu retiens (à versionner)
notebooks/
  01_motifs.ipynb     # point de départ
requirements.txt
```

## Installation

```bash
python -m venv .venv
```

```bash
.venv\Scripts\Activate.ps1
```

(macOS / Linux : `source .venv/bin/activate`.) Puis, **dans cet ordre** :

```bash
pip install torch==2.5.1 --index-url https://download.pytorch.org/whl/cpu --extra-index-url https://pypi.org/simple
```

```bash
pip install -r requirements.txt
```

⚠️ Installe PyTorch **avant** `requirements.txt` : sinon pip installe la version GPU, plusieurs Go. Sous Windows, place le projet dans un dossier au chemin court (par exemple `C:\dev\M1-B2-motifs`) : PyTorch contient des fichiers aux noms très longs, et un chemin trop profond fait échouer l'installation (`No such file or directory … Long Path`).

Ouvre ensuite `notebooks/01_motifs.ipynb` et **sélectionne le noyau `.venv`** en haut à droite. Si VS Code propose d'installer `ipykernel` dans un autre Python, c'est que le mauvais noyau est sélectionné.

Au premier lancement, le modèle d'embeddings (environ 470 Mo) est téléchargé depuis Hugging Face, puis gardé en cache sur ta machine.

## Les données

| Colonne | Contenu |
|---|---|
| `avis_id` | identifiant de l'avis |
| `hotel` | `City Hotel` ou `Resort Hotel` |
| `source` | Booking, Google, TripAdvisor ou Questionnaire |
| `date_avis` | date de publication |
| `note` | note donnée par le client (1 ou 2 sur 5) — `avis_motifs.csv` seulement |
| `texte` | l'avis |
| `motif` | le motif **principal** de l'insatisfaction |

## Les motifs

| Motif | Couvre |
|---|---|
| `proprete` | ménage, hygiène, odeurs |
| `bruit` | nuisances sonores, isolation |
| `accueil` | attitude et disponibilité du personnel |
| `restauration` | petit-déjeuner, restaurant, room service |
| `chambre` | confort, literie, taille, vue, état du mobilier |
| `equipements` | wifi, climatisation, eau chaude, ascenseur, piscine, spa |
| `prix` | tarifs, suppléments, rapport qualité-prix |
| `emplacement` | situation, accès, quartier, transports |
| `reservation` | réservation, arrivée, facturation, remboursements |

## Le modèle d'embeddings

[`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`](https://huggingface.co/sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2) : un modèle multilingue qui transforme une phrase en un vecteur de 384 nombres. Deux phrases de sens proche donnent des vecteurs proches, même avec des mots différents.

## Keras

Keras 3 tourne ici avec **PyTorch** comme moteur, déjà installé, ce qui évite TensorFlow. La première cellule du notebook le règle :

```python
os.environ["KERAS_BACKEND"] = "torch"   # AVANT tout « import keras »
```

## Données

Avis fictifs, rédigés pour la formation.

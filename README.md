# TP2 — Conteneurisation du modèle Adult Income

Le projet entraîne un classifieur à partir de `data/adult.csv`, enregistre le pipeline dans `/artifacts/model.joblib`, puis expose ce modèle avec FastAPI.

## Environnement local

Le fichier `requirements.txt` cible Python 3.12. Dans PowerShell, depuis la racine du projet :

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
python -m src.train --data data/adult.csv --out artifacts/model.joblib
python -m src.predict --model artifacts/model.joblib --input data/sample.csv
```

Si `py -3.12` n'est pas disponible, installez Python 3.12 puis recréez `.venv` avec cet interpréteur. Le dossier `.venv` existant peut être recréé si son interpréteur d'origine a été déplacé.

## Docker Compose

Docker Desktop doit être démarré. Depuis la racine du projet :

```powershell
docker compose up --build
```

Compose exécute d'abord le service `train`. Il lit le CSV monté en lecture seule et écrit le modèle dans le volume `model-artifacts`. Le service `predict` démarre après la réussite de l'entraînement et sert l'API sur `http://localhost:8000`.

Vérification et exemple de prédiction :

```powershell
Invoke-RestMethod http://localhost:8000/ready
Invoke-RestMethod -Method Post -Uri http://localhost:8000/v1/predict `
  -ContentType 'application/json' `
  -Body '{"age":35,"education_num":13,"hours_per_week":40}'
```

## Comparaison des images

`Dockerfile.serve` est la version de référence. `Dockerfile.serve.opt` utilise une image slim, une construction multi-étape, un environnement d'exécution séparé et un utilisateur non privilégié. Mesurez les deux depuis Bash ou Git Bash :

```bash
bash benchmark.sh naive Dockerfile.serve
bash benchmark.sh optimise Dockerfile.serve.opt
```

| Image | Taille (Mo) | Construction froide (s) | Reconstruction après changement de code (s) |
|---|---:|---:|---:|
| Référence (`Dockerfile.serve`) | À mesurer | À mesurer | À mesurer |
| Optimisée (`Dockerfile.serve.opt`) | À mesurer | À mesurer | À mesurer |

Les mesures dépendent de la machine et du cache Docker ; recopiez les valeurs affichées par `benchmark.sh` après exécution.

## Analyse Trivy

Après construction des images, analysez-les avec Trivy :

```powershell
trivy image --severity HIGH,CRITICAL adult-serve:naive
trivy image --severity HIGH,CRITICAL adult-serve:optimise
```

Consignez les vulnérabilités détectées et leur correction ici après le scan. L'image optimisée réduit la surface d'attaque en supprimant les outils de construction et en exécutant le service avec un compte non privilégié.

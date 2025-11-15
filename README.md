# VRP Or-Tools — Frontend (Vite/React) + Backend (FastAPI)

Application complète pour générer des matrices de distances/temps via OSRM et résoudre un problème de tournées de véhicules (VRP) avec Google OR-Tools. Frontend moderne avec Vite/React + Tailwind (shadcn/ui).


## Sommaire
- [Architecture du projet](#architecture-du-projet)
- [Prérequis](#prérequis)
- [Installation rapide](#installation-rapide)
- [Lancer le Backend (FastAPI)](#lancer-le-backend-fastapi)
- [Lancer le Frontend (Vite/React)](#lancer-le-frontend-vitereact)
- [Commandes utiles](#commandes-utiles)
- [API Backend](#api-backend)
- [Flux d’utilisation recommandé](#flux-dutilisation-recommandé)
- [Dépannage](#dépannage)


## Architecture du projet
```
Projet/
├─ Backend/
│  ├─ code/
│  │  └─ main.py            # Application FastAPI (endpoints VRP)
│  └─ requirements.txt      # Dépendances Python
├─ Frontend/
│  ├─ src/                  # Code React + UI
│  ├─ package.json          # Scripts et dépendances
│  └─ tailwind.config.ts    # Config Tailwind
└─ README.md
```


## Prérequis
- Node.js (recommandé: LTS) et un gestionnaire de paquets (npm, pnpm, yarn ou bun)
- Python 3 (avec venv)
- OSRM backend accessible en local sur `http://localhost:5001` (profil driving) pour la génération des matrices

Exemple de démarrage rapide d’OSRM avec Docker (exige un extrait de carte `.osm.pbf`):
```bash
# 1) Télécharger un extrait de carte (ex: maroc-latest.osm.pbf) depuis Geofabrik
# 2) Préparer et lancer OSRM (profil car/driving)
docker run -t -v $(pwd):/data osrm/osrm-backend osrm-extract -p /opt/car.lua /data/maroc-latest.osm.pbf

# Contract (optionnel mais recommandé)
docker run -t -v $(pwd):/data osrm/osrm-backend osrm-contract /data/maroc-latest.osrm

# Lancer le serveur OSRM sur le port 5001
docker run -t -i -p 5001:5000 -v $(pwd):/data osrm/osrm-backend osrm-routed /data/maroc-latest.osrm
```


## Installation rapide
```bash
# Cloner le repo (si ce n'est pas déjà fait)
# git clone <url>
# cd Projet

# 1) Backend: créer un venv et installer les deps
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\\Scripts\\activate
pip install -r Backend/requirements.txt

# 2) Frontend: installer les deps
cd Frontend
npm install                 # ou: pnpm i | yarn | bun install
```


## Lancer le Backend (FastAPI)
Depuis la racine du projet (avec le venv activé):
```bash
# Option A: via uvicorn (module)
python -m uvicorn Backend.code.main:app --reload --port 8000

# Option B: uvicorn direct (si installé en global)
uvicorn Backend.code.main:app --reload --port 8000
```
- L’API sera disponible par défaut sur: `http://localhost:8000`
- Le backend requiert OSRM sur `http://localhost:5001` pour les endpoints de matrices.


## Lancer le Frontend (Vite/React)
Depuis `Frontend/`:
```bash
npm run dev   # ou: pnpm dev | yarn dev | bun dev
```
- Par défaut, Vite lance l’app sur `http://localhost:5173`.
- Assurez-vous que l’API FastAPI tourne (port 8000) et qu’OSRM est accessible (port 5001).


## Commandes utiles
- Frontend (`Frontend/package.json`):
  - `npm run dev` — lancer le serveur de dev Vite
  - `npm run build` — build production
  - `npm run build:dev` — build en mode développement
  - `npm run preview` — prévisualiser le build
  - `npm run lint` — lint du code

- Backend:
  - Installer les deps: `pip install -r Backend/requirements.txt`
  - Lancer l’API: `python -m uvicorn Backend.code.main:app --reload --port 8000`


## API Backend
Fichier principal: `Backend/code/main.py`

- `GET /`
  - Ping simple: `{ "message": "..." }`

- `POST /upload-dataset` (multipart/form-data)
  - Paramètre: `file` (xlsx ou csv)
  - Stocke le fichier sous `uploads/latest.xlsx` ou `uploads/latest.csv`
  - Vérifie la présence des colonnes: `PARTNER_CODE`, `LATITUDE`, `LONGITUDE`, `WEIGHT`

- `POST /generate-matrices` (JSON)
  - Corps (par défaut):
    ```json
    {
      "depot_lat": 33.604427,
      "depot_lng": -7.558631
    }
    ```
  - Produit: `matrices/distance_matrix.csv`, `matrices/time_matrix.csv`, `matrices/clients_processed.csv` et `matrices/summary.json`
  - S’appuie sur OSRM (`http://localhost:5001/table/v1/driving/...`)

- `POST /solve-vrp` (JSON)
  - Corps (exemple par défaut):
    ```json
    {
      "num_vehicles": 36,
      "vehicle_capacity": 4000.0,
      "service_time": 5,
      "time_limit": 300,
      "vehicle_time_limit": 480,
      "interval_seconds": 60
    }
    ```
  - Utilise OR-Tools pour optimiser les tournées à partir des matrices CSV générées


## Flux d’utilisation recommandé
1. Lancer OSRM (port 5001)
2. Lancer le Backend (port 8000)
3. Lancer le Frontend (Vite)
4. Depuis l’UI:
   - Importer un dataset via l’interface (ou appeler `POST /upload-dataset`)
   - Générer les matrices (`POST /generate-matrices`)
   - Lancer la résolution (`POST /solve-vrp`)
5. Visualiser les résultats dans le composant `ResultsViewer`


## Dépannage
- Erreur Tailwind dans `tailwind.config.ts` liée à `require`: le projet utilise ESM/TypeScript. Le fix appliqué importe le plugin ainsi:
  ```ts
  import animate from "tailwindcss-animate";
  export default { plugins: [animate] };
  ```
- OSRM « connection refused »: assurez-vous que le conteneur/serveur OSRM est actif sur `localhost:5001` et que les fichiers `.osrm` correspondent à la zone visée.
- Problèmes Python/venv: recréez l’environnement et réinstallez les dépendances.
  ```bash
  rm -rf .venv
  python3 -m venv .venv && source .venv/bin/activate
  pip install -r Backend/requirements.txt
  ```
- Ports occupés: changez les ports (`--port`) de Vite/uvicorn si nécessaire.

---
Si vous souhaitez ajouter un Docker Compose pour automatiser OSRM + Backend + Frontend, je peux le fournir.

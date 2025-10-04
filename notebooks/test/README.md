# Frelon — Entraînement SSD MobileNet V3 Small (classe unique)

Ce projet prépare l'entraînement d'un détecteur **SSD MobileNet V3 Small FPNLite 320x320** sur la seule classe **frelon** à partir du dataset Roboflow fourni.

## Contenu

- `00_environment_check.ipynb` — Vérifie l'environnement et versions.
- `01_download_and_prepare_data.ipynb` — Télécharge le dataset Roboflow, filtre la classe **frelon**, prépare COCO + TFRecords.
- `02_install_tfod_api.ipynb` — Installe la **TensorFlow Object Detection API**.
- `03_configure_ssd_mobilenet_v3_small.ipynb` — Met à jour `pipeline.config` pour 1 classe et chemins locaux.
- `04_train_and_eval.ipynb` — Lance l'entraînement et l'évaluation.
- `05_export_tflite.ipynb` — Exporte en SavedModel/TFLite (quantification optionnelle).
- `requirements.txt` — Dépendances Python.

## Démarrage rapide

1. Créez un environnement et installez les dépendances :
   ```bash
   python -m venv .venv && source .venv/bin/activate  # (Windows: .venv\Scripts\activate)
   pip install --upgrade pip
   pip install -r requirements.txt
   ```

2. Ouvrez Jupyter :
   ```bash
   jupyter notebook
   ```

3. Exécutez les notebooks dans l'ordre 00 → 05.
   - Dans `01_...`, définissez `RF_API_KEY` (variable d'environnement ou directement dans la cellule).
   - Après `02_...`, générez les TFRecords avec la commande indiquée en sortie de `01_...`.
   - Dans `03_...`, téléchargez et placez le modèle `ssd_mobilenet_v3_small_fpnlite_320x320_coco17_tpu-8` dans `training/`.

> Remarque : si vous préférez CPU-only, ignorez les paquets NVIDIA. Sur GPU, installez les versions CUDA/cuDNN compatibles avec votre setup.

Bon entraînement 🐝

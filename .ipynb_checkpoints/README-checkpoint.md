# Choice Disease

Outil d'annotation manuelle de maladies rares pour des patients porteurs de variants génétiques associés à plusieurs pathologies possibles (Orphanet).

## Contexte

Certains patients eHOP portent un variant sur un gène associé à plusieurs maladies rares dans la base Orphanet. Ce projet permet à un annotateur de consulter les données cliniques de chaque patient (termes HPO, comptes rendus médicaux) et de sélectionner la maladie la plus probable parmi les candidats proposés.

---

## Structure du projet

```
choice_disease/
├── data_processing.ipynb          # Pipeline de préparation des données
├── choice_disease.ipynb           # Application d'annotation interactive
├── convert_genes_to_diseases.ipynb
├── genes_associated_with_rare_diseases.xml   # Référentiel Orphanet
├── mapping_genes_maladies_orphanet_13_05_2026.csv  # Généré par data_processing
├── patients_multi_diseases.csv    # Généré par data_processing (input de l'appli)
└── choice_output.csv              # Généré par l'appli (annotations sauvegardées)
```

Données sources (non versionnées, accès sécurisé) :
```
data/raw_data/clinical_side/ehop_one_gene_export_with_keys_without_na.tsv
data/raw_data/cr_genetique_exomes_with_text.csv
data/modified_data/for_phenoBERT/full_table_agg_patient_level_phenobert03032025.csv
```

---

## Pipeline

### Étape 1 — data_processing.ipynb

1. Parse le fichier XML Orphanet → table gène/maladie (mappingGTD)
2. Charge les patients eHOP avec leur gène causal (rdPatData)
3. Joint les deux tables sur gene_symbol → dfMerged
4. Filtre les patients ayant plusieurs maladies possibles (NB_DISEASES_TOTAL > 1) → log_data
5. Ajoute les termes HPO depuis le fichier PhenoBERT (jointure ID_PAT_ETUDE ↔ PATIENT_ID)
6. Sauvegarde → patients_multi_diseases.csv

Les comptes rendus médicaux ne sont pas intégrés ici — ils sont lus directement depuis le fichier source par l'application d'annotation.

### Étape 2 — choice_disease.ipynb

Application interactive en cellule Jupyter :
- Charge patients_multi_diseases.csv et les comptes rendus bruts (cr_genetique_exomes_with_text.csv)
- Pour chaque patient non encore annoté, affiche :
  - Identifiants (Patient ID, IPP)
  - Gène et Gene ID
  - Termes HPO
  - Comptes rendus médicaux navigables (touches n / p)
  - Liste des maladies candidates (Orphanet)
- L'annotateur saisit un numéro pour choisir une maladie, s pour passer, q pour arrêter
- La progression est sauvegardée en continu dans choice_output.csv
- La session peut être reprise à tout moment sans perdre les annotations déjà faites

---

## Utilisation

### Prérequis

```bash
pip install pandas
```

### Lancer la préparation des données

Ouvrir et exécuter toutes les cellules de data_processing.ipynb.

### Lancer l'annotation

Ouvrir et exécuter toutes les cellules de choice_disease.ipynb.

Commandes disponibles pendant l'annotation :

| Touche | Action |
|--------|--------|
| 1, 2... | Choisir la maladie correspondante |
| n | Compte rendu suivant |
| p | Compte rendu précédent |
| s | Passer ce patient |
| q | Arrêter (reprise possible) |

### Exporter les résultats

Exécuter la dernière cellule de choice_disease.ipynb pour générer patients_one_disease.csv — une ligne par patient avec la maladie annotée.

---

## Colonnes de choice_output.csv

| Colonne | Description |
|---------|-------------|
| ID_PAT_ETUDE | Identifiant patient |
| real_disease_name | Nom de la maladie sélectionnée |
| real_disease_code | Code Orphanet de la maladie sélectionnée |

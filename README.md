# Choice Disease

Outil d'annotation manuelle de maladies rares pour des patients porteurs de variants génétiques associés à plusieurs pathologies possibles (Orphanet).

## Contexte

Certains patients eHOP portent un variant sur un gène associé à plusieurs maladies rares dans la base Orphanet. Ce repo permet de consulter les données cliniques de chaque patient (termes HPO, comptes rendus médicaux) et de sélectionner la maladie la plus probable parmi les candidats proposés.

---

## Structure du projet

```
choice_disease/
├── data_processing.ipynb          # Pipeline de préparation des données
├── choice_disease.ipynb           # mini application d'annotation
├── genes_associated_with_rare_diseases.xml   # Référentiel Orphanet
├── mapping_genes_maladies_orphanet_13_05_2026.csv  # Généré par data_processing
├── patients_multi_diseases.csv    # Généré par data_processing (input de l'appli)
└── choice_output.csv              # Généré par l'appli (annotations sauvegardées)
└── patients_one_disease.csv              # Fichier final avec 1 maladie par patient

```

Données sources sur session Hugo-RD :
```
data/raw_data/clinical_side/ehop_one_gene_export_with_keys_without_na.tsv (données avec les gènes)
data/raw_data/cr_genetique_exomes_with_text.csv (données pour les comptes rendus)
data/modified_data/for_phenoBERT/full_table_agg_patient_level_phenobert03032025.csv (données avec les termes HPO)
```

---

## Pipeline

### Étape 1 — data_processing.ipynb

1. Parse le fichier XML Orphanet → table gène/maladie 
2. Charge les patients eHOP avec leur gène causal 
3. Joint les deux dataframes
4. Filtre les patients ayant plusieurs maladies possibles 
5. Ajoute les termes HPO depuis le fichier full_table_agg_patient_level_phenobert03032025
6. Sauvegarde → patients_multi_diseases.csv

Les comptes rendus médicaux ne sont pas intégrés ici — ils sont lus directement depuis le fichier source par l'application d'annotation pour éviter des problèmes de format. 

### Étape 2 — choice_disease.ipynb

Application simple en cellule Jupyter (pas besoin de librairies spéciales) :
- Charge patients_multi_diseases.csv et les comptes rendus bruts (cr_genetique_exomes_with_text.csv)
- Pour chaque patient non encore annoté, affiche :
  - Identifiants (Patient ID, clé IPP)
  - Gène symbole et Gene ID
  - Termes HPO
  - Comptes rendus médicaux navigables (touches n / p)
  - Liste des maladies candidates (Orphanet)
- Le médecin saisit un numéro pour choisir une maladie, s pour passer, q pour arrêter
- La progression est sauvegardée en continu dans choice_output.csv
- La session peut être reprise à tout moment sans perdre les annotations déjà faites
- Exécuter la dernière cellule de choice_disease.ipynb pour générer patients_one_disease.csv — une ligne par patient avec la maladie annotée.


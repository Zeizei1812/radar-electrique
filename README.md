# ⚡ Radar Électrique

Outil qui suit la **transition électrique en France**, département par département :

- il compare la croissance du **parc de voitures électriques** à celle des **bornes de recharge**
  (où l'infrastructure prend-elle du retard ?) ;
- il teste si le **prix des carburants** pousse les ménages vers l'électrique.

Projet en binôme — Master MOSEF, cours d'ingénierie du code. Rendu avant Noël.

> **État du dépôt :** cette branche `main` ne contient pour l'instant que la **structure** du
> projet (dossiers et fichiers vides). Le code arrive par des pull requests relues par
> le binôme.

---

## 1. Les étapes du projet

Le projet suit une chaîne en cinq étapes. Chaque étape a sa place dans le dépôt.

| # | Étape | Où dans le dépôt |
|---|---|---|
| 1 | **Récupérer les données** automatiquement (problématique n°1) | `config/sources.yaml`, `src/radar/collect/` |
| 2 | **Préparer les données** (nettoyage, rattachement au département, alignement des dates) | `src/radar/preprocess.py` |
| 3 | **Analyser** (tendances, prévisions, effet du prix des carburants) | `src/radar/analyze.py` |
| 4 | **Mettre à disposition** : interface en ligne de commande (bash) et dashboard | `src/radar/cli.py`, `app/streamlit_app.py` |
| 5 | **Expliquer** : document d'explication remis au professeur | `docs/explication.md` |

Et, transversal à toutes les étapes : **l'automatisation** (collecte planifiée, tests à chaque
modification), dans `.github/workflows/`.

```
   Sources open data          Étape 1             Étape 2              Étape 3
 (data.gouv, SDES, Insee,  ─► collect/  ─► data/raw ─► preprocess ─► data/clean ─► analyze
  prix des carburants)                                                     │
                                                                           ▼  Étape 4
                                                              cli.py (bash) · dashboard
                                                                           │
                                                                           ▼  Étape 5
                                                               docs/explication.md
   ⟳ Automatisation : GitHub Actions lance la collecte chaque jour et les tests à chaque PR
```

## 2. Les données

Toutes en open data, téléchargeables par lien direct, **sans clé API**.

| Source | Producteur | Fréquence |
|---|---|---|
| Base consolidée des bornes de recharge (IRVE) | data.gouv.fr | quotidienne |
| Parc et immatriculations de véhicules électriques par département | SDES | trimestrielle / annuelle |
| Prix des carburants (flux quotidien + historique annuel) | Ministères économiques et financiers | quotidienne |
| Population par département | Insee | annuelle |

Les liens exacts iront dans `config/sources.yaml`.

## 3. Organisation du dépôt

```
radar-electrique/
├── config/
│   └── sources.yaml          # URL et paramètres de chaque source
├── src/radar/
│   ├── collect/              # ÉTAPE 1 — un module par source
│   │   ├── base.py           #   outils communs : téléchargement, nouvelles tentatives, logs
│   │   ├── irve.py           #   bornes de recharge
│   │   ├── sdes.py           #   véhicules électriques
│   │   ├── carburants.py     #   prix des carburants
│   │   └── insee.py          #   population
│   ├── preprocess.py         # ÉTAPE 2 — nettoyage et alignement
│   ├── analyze.py            # ÉTAPE 3 — Mann-Kendall, pente de Sen, prévision, Granger
│   ├── storage.py            # lecture/écriture des fichiers, détection « fichier inchangé »
│   └── cli.py                # ÉTAPE 4 — commandes `radar ...` dans le terminal
├── app/
│   └── streamlit_app.py      # ÉTAPE 4 — dashboard interactif
├── docs/
│   └── explication.md        # ÉTAPE 5 — document d'explication
├── tests/                    # tests automatiques (au moins un par module)
├── data/raw/, data/clean/    # données téléchargées / nettoyées (jamais envoyées sur GitHub)
├── logs/                     # journaux de collecte (jamais envoyés sur GitHub)
├── .github/
│   ├── workflows/            # AUTOMATISATION — tests à chaque PR, collecte quotidienne
│   ├── ISSUE_TEMPLATE/       # modèles pour créer une tâche ou signaler un bug
│   ├── pull_request_template.md
│   └── CODEOWNERS            # relecteurs automatiques des PR
├── pyproject.toml            # dépendances Python (gérées avec uv)
├── Makefile                  # raccourcis : make install, make collect, make test...
├── CONTRIBUTING.md           # notre façon de travailler à deux
└── .env.example              # exemple de réglages locaux (aucun secret nécessaire)
```

## 4. Outils

- **Python** avec **[uv](https://docs.astral.sh/uv/)** pour gérer l'environnement et les dépendances.
- **pandas / pyarrow** pour les données, **statsmodels / pymannkendall** pour les statistiques.
- **Typer + Rich** pour l'interface en ligne de commande.
- **Streamlit + Plotly** pour le dashboard.
- **pytest + ruff** pour les tests et la qualité du code.
- **GitHub Actions** pour l'automatisation.

Commandes prévues (une fois le code fusionné) :

```bash
make install     # installe l'environnement
make collect     # récupère les données
make test        # lance les tests
make lint        # vérifie le style du code
make app         # ouvre le dashboard
radar --help     # liste les commandes : collect, rank, analyze <departement>, carburant, report
```

## 5. Travailler à deux

La branche `main` est **protégée** : on ne peut pas y pousser directement.

1. Partir de `main` à jour : `git checkout main && git pull`
2. Créer sa branche : `git checkout -b feat/nom-de-la-tache`
3. Coder, tester (`make test`), enregistrer (`git add` puis `git commit -m "..."`)
4. Envoyer sa branche : `git push -u origin feat/nom-de-la-tache`
5. Ouvrir une pull request ; **l'autre relit et approuve**, la CI doit être verte, puis on fusionne
6. Revenir sur `main` et supprimer la branche locale

Les tâches à faire sont listées dans les
[issues](https://github.com/Zeizei1812/radar-electrique/issues).

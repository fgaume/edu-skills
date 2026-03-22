---
name: affelnet-score-scolaire
description: Use when the user wants to calculate the Affelnet school score (score scolaire) from a student's trimestrial or semestrial grades. Covers subject averaging, disciplinary field grouping, academic harmonization, weighting, and final score computation.
---

# Calcul du score scolaire Affelnet

## Overview

Calcul du score scolaire Affelnet a partir des moyennes trimestrielles ou semestrielles d'un eleve, en 5 etapes : moyenne par matiere, regroupement en champs disciplinaires, harmonisation academique, ponderation, et coefficient final.

## When to Use

- L'utilisateur fournit des bulletins scolaires (notes trimestrielles/semestrielles) et veut calculer son score Affelnet
- L'utilisateur veut comprendre comment Affelnet calcule le score scolaire
- L'utilisateur fournit une image ou un tableau de notes

## Matieres prises en compte

Uniquement ces 12 matieres : Francais, Mathematiques, Histoire-Geo, LV1, LV2, EPS, Arts Plastiques, Education Musicale, SVT, Technologie, Physique-Chimie, EMC.

**Si EMC est absente** : la remplacer par la moyenne d'Histoire-Geo.

**Attention** : prendre la note de l'eleve, pas celle de la classe. Si les 3 trimestres sont dans le meme tableau, bien identifier les colonnes.

## Etape 1 : Moyenne annuelle par matiere

Calculer la moyenne des 3 trimestres (ou 2 semestres) pour chaque matiere.

## Etape 2 : Regroupement en 7 champs disciplinaires

| Champ disciplinaire | Calcul |
|---------------------|--------|
| Francais | tel quel |
| Mathematiques | tel quel |
| EPS | tel quel |
| ARTS | moyenne(Arts Plastiques, Education Musicale) |
| HISTOIRE-GEO | moyenne(Histoire-Geo, EMC) |
| LANGUES VIVANTES | moyenne(LV1, LV2) |
| SCIENCES-TECHNO-DP | moyenne(Physique-Chimie, SVT, Technologie) |

## Etape 3 : Harmonisation academique

Pour chaque champ disciplinaire, on harmonise la note T de l'eleve avec la formule :

```
H = 10 x [10 + (T - mu) / sigma]
```

Ou `mu` = moyenne academique et `sigma` = ecart-type academique du champ disciplinaire.

**Recuperer mu et sigma** via l'API Hugging Face :

```
GET https://datasets-server.huggingface.co/rows?dataset=fgaume%2Faffelnet-paris-statistiques-champs-disciplinaires&config=default&split=train&offset=0&length=100
```

Filtrer les rows avec l'annee la plus recente.

## Etape 4 : Ponderation des champs disciplinaires

| Champ | Coefficient |
|-------|-------------|
| Francais | 5 |
| Mathematiques | 5 |
| EPS | 4 |
| ARTS | 4 |
| HISTOIRE-GEO | 4 |
| LANGUES VIVANTES | 4 |
| SCIENCES-TECHNO-DP | 4 |

Score bilan periodique = somme des 7 valeurs (H x coefficient).

## Etape 5 : Coefficient de ponderation scolaire

```
Score scolaire final = Score bilan periodique x 2
```

## Quick Reference

| Etape | Operation | Sortie |
|-------|-----------|--------|
| 1 | Moyenne trimestres/semestres par matiere | 12 moyennes annuelles |
| 2 | Regroupement en champs disciplinaires | 7 notes intermediaires (T) |
| 3 | Harmonisation : H = 10 x [10 + (T-mu)/sigma] | 7 notes harmonisees (~100) |
| 4 | Ponderation (x5 FR/Maths, x4 autres) | Score bilan periodique |
| 5 | x2 | Score scolaire final |

## Common Mistakes

- **Prendre la moyenne de classe** au lieu de la note de l'eleve
- **Oublier EMC** : si absente, utiliser la moyenne d'Histoire-Geo
- **Confondre trimestres et semestres** : adapter le calcul de moyenne (diviser par 3 ou par 2)
- **Utiliser des stats academiques perimees** : toujours filtrer par l'annee la plus recente dans le dataset HF
- **Oublier le x2 final** de l'etape 5

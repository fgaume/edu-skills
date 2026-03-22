---
name: affelnet-paris
description: Use when the user asks about Affelnet Paris - calculating a lycee admission score (bareme), understanding the admission process, analyzing a fiche bareme from the Rectorat, or estimating chances for a specific lycee. Orchestrates sub-skills for sectors, IPS, UAI lookup, and school score calculation.
---

# Affelnet Paris - Calcul du bareme d'affectation en lycee

## Overview

Affelnet Paris est la procedure d'affectation automatique (Gale & Shapley) des collegiens de Paris aux lycees publics (2nde GT uniquement). Ce skill orchestre le calcul du score (bareme) d'un voeu de lycee.

## When to Use

- L'utilisateur veut calculer ou estimer son score Affelnet pour un lycee
- L'utilisateur fournit des bulletins scolaires et veut une estimation
- L'utilisateur a une fiche bareme du Rectorat a analyser/verifier
- L'utilisateur pose des questions sur le fonctionnement d'Affelnet Paris

## Informations a recueillir aupres du collegien

Si non precisees, demander :
1. **Adresse** : pour determiner le college de secteur (via skill `affelnet-secteurs-paris`)
2. **College de scolarisation** : s'il est different du college de secteur (derogation)
3. **Boursier ou non** : bonus de 600 points si oui
4. **Notes/bulletins** : moyennes trimestrielles ou semestrielles

## Calcul du score d'un voeu

```
Score = Bonus geographique + Bonus IPS + Bonus boursier + Score scolaire
```

### 1. Bonus geographique

Depend du **college de secteur** (pas de scolarisation) et du lycee demande.

| Secteur du lycee | Bonus |
|------------------|-------|
| Secteur 1 (5 lycees) | 32 640 |
| Secteur 2 | 16 800 |
| Secteur 3 | 17 760 |
| Hors secteur | 0 |

**Lycees particuliers** (secteur 1 pour TOUS les colleges de Paris) : PIERRE GILLES DE GENNES, HENRI IV, LOUIS LE GRAND.

College prive : on utilise le college de secteur lie a l'adresse postale.
College non parisien : aucun bonus geographique.

Utiliser le skill `affelnet-secteurs-paris` pour determiner college de secteur et lycees de secteur 1/2/3.

### 2. Bonus IPS

Lie au **college de scolarisation** (pas de secteur). Valeurs possibles : 1200, 800, 400 ou 0.

Utiliser le skill `affelnet-ips-colleges` pour obtenir le bonus.

### 3. Bonus boursier

600 points si boursier, 0 sinon. Demander au collegien si non precise.

### 4. Score scolaire

Depuis 2026, seul le bilan periodique compte (le bilan de fin de cycle sur 4800 points a disparu).

Utiliser le skill `affelnet-score-scolaire` pour calculer ce score a partir des notes.

### Admissibilite

Si score >= seuil d'admission du lycee demande, l'eleve est admissible.
L'algorithme tourne 2 fois : d'abord les boursiers, puis les non-boursiers sur les places restantes.

## Analyse de la fiche bareme du Rectorat

La fiche bareme est fournie apres l'execution d'Affelnet pour verification. Elle contient 3 tableaux :

### Tableau 1 : Notes par matiere (etape 1 du bilan periodique)

Abbreviations : A-PLA = Arts Plastiques, EDMUS = Education Musicale, PH-CH = Physique-Chimie, HI-GE = Histoire-Geo.

### Tableau 2 : Notes harmonisees par champs disciplinaires

Affiche note brute et note harmonisee pour les 7 champs (fin de l'etape 3).

### Tableau 3 : Total du bareme et comparaison aux derniers entrants

Colonnes : Rang du voeu, secteur (bonus geo), Boursier, I.P.S., Disciplines, bareme, Etablissement demande, Formation, Decision, Bareme dernier entrant.

**En-tete** : college de scolarisation (+ UAI + PU/PR) a gauche, statut boursier a droite.

### Retrouver le college de secteur depuis la fiche

La fiche n'indique pas le college de secteur. Pour le retrouver :
1. Reperer les voeux avec bonus geographique = 32 640 (hors PIERRE GILLES DE GENNES)
2. Ces lycees sont les lycees de secteur 1
3. Utiliser le skill `affelnet-secteurs-paris` (recherche inverse) pour trouver quel college a ces lycees en secteur 1

## Sub-skills utilises

| Besoin | Skill |
|--------|-------|
| College de secteur / lycees de secteur | `affelnet-secteurs-paris` |
| Bonus IPS d'un college | `affelnet-ips-colleges` |
| Correspondance UAI / nom | `affelnet-uai-paris` |
| Score scolaire | `affelnet-score-scolaire` |

## Common Mistakes

- **Confondre college de secteur et de scolarisation** : le bonus geo depend du college de secteur, le bonus IPS du college de scolarisation
- **Oublier les 3 lycees particuliers** : PGDG, Henri IV, Louis Le Grand sont secteur 1 pour tous
- **Inclure le bilan de fin de cycle** : supprime depuis 2026, ne reste que le bilan periodique
- **Oublier de demander le statut boursier** : 600 points de difference

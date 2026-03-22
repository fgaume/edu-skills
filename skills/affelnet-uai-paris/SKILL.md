---
name: affelnet-uai-paris
description: Use when the user needs to find the UAI code of a Parisian college from its name, or the name of a college from its UAI code. Uses the ArcGIS Rectorat API for Paris school identification.
---

# Lien UAI / Nom des colleges et lycees Paris

## Overview

Correspondance entre UAI et nom des colleges parisiens via l'API ArcGIS du Rectorat de Paris.

## When to Use

- L'utilisateur cherche l'UAI d'un college a partir de son nom
- L'utilisateur cherche le nom d'un college a partir de son UAI
- Besoin de l'UAI en entree d'un autre skill Affelnet (secteurs, IPS, etc.)

## Semantique des champs ArcGIS

**Attention**, les noms de champs sont trompeurs :

| Champ ArcGIS | Signification reelle |
|--------------|---------------------|
| UAI | UAI du **lycee** |
| Nom | Nom du **lycee** |
| Reseau | UAI du **college** |
| Nom_Tete | Nom du **college** |

## Nom du college -> UAI

```
GET https://services9.arcgis.com/ekT8MJFiVh8nvlV5/arcgis/rest/services/Affectation_Lyc%C3%A9es/FeatureServer/0/query
  ?outFields=Réseau,Nom_Tete
  &returnGeometry=false
  &f=pjson
  &returnDistinctValues=true
  &where=Nom_Tete like '%{nom_college}'
```

Le champ `Réseau` dans la reponse contient l'UAI du college.

## UAI du college -> Nom

```
GET https://services9.arcgis.com/ekT8MJFiVh8nvlV5/arcgis/rest/services/Affectation_Lyc%C3%A9es/FeatureServer/0/query
  ?outFields=Nom_Tete
  &returnGeometry=false
  &f=pjson
  &returnDistinctValues=true
  &where=Réseau='{uai_college}'
```

Le champ `Nom_Tete` dans la reponse contient le nom du college.

## Common Mistakes

- **Confondre les champs** : `UAI`/`Nom` = lycee, `Réseau`/`Nom_Tete` = college
- **Oublier le wildcard** : utiliser `like '%NOM'` pour la recherche par nom (pas d'egalite stricte)

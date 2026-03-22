---
name: affelnet-secteurs-paris
description: Use when the user asks about Affelnet Paris school sectors - finding a college from a Paris address, listing lycees for a college, or finding colleges feeding into a lycee. Covers geocoding, coordinate conversion, and ArcGIS API calls for the Paris school map (carte scolaire).
---

# Affelnet - Secteurs scolaires parisiens

## Overview

Skill de reference pour determiner les colleges et lycees de secteur Affelnet dans l'Academie de Paris, via des APIs publiques (CapGeo Paris et ArcGIS Rectorat).

## When to Use

- L'utilisateur cherche son **college de secteur** a partir d'une adresse parisienne
- L'utilisateur veut les **lycees de secteur** (1, 2, 3) d'un college donne
- L'utilisateur veut savoir quels **colleges ont un lycee donne en secteur 1**
- Uniquement pour l'**Academie de Paris**

## Use Case 1 : College de secteur a partir d'une adresse parisienne

Trois etapes sequentielles. Toujours Paris, pas besoin de preciser la ville.

### Step 1 : Geocodage (adresse -> coordonnees Lambert-93)

URL-encoder l'adresse et l'inserer dans le parametre `SingleLine` :

```
GET https://capgeo2.paris.fr/datastore/rest/services/GEOCODAGE/Localisateur_Capgeo_Paris/GeocodeServer/findAddressCandidates?outFields=Score&outSR=%7B%22latestWkid%22%3A2154%2C%22wkid%22%3A102110%7D&f=json&SingleLine={adresse_url_encodee}
```

Reponse : prendre `candidates[0].location` (x, y) avec le score le plus eleve. Coordonnees en EPSG:2154 (Lambert-93).

### Step 2 : Conversion Lambert-93 -> Web Mercator (EPSG:3857)

La carte scolaire utilise le systeme Web Mercator. Convertir avec pyproj :

```python
from pyproj import Transformer, CRS

transformer = Transformer.from_crs(CRS("EPSG:2154"), CRS("EPSG:3857"), always_xy=True)
x_mercator, y_mercator = transformer.transform(x_lambert, y_lambert)
```

### Step 3 : Requete carte scolaire

URL-encoder les coordonnees en JSON `{"x":..., "y":...}` et appeler :

```
GET https://capgeo2.paris.fr/public/rest/services/DASCO/DASCO_Carte_scolaire/MapServer/0/query?f=json&resultOffset=0&resultRecordCount=2000&where=annee_scol%3D%272026-2027%27%20AND%20type_etabl%3D%27COL%27&outFields=libelle%2Ctype_etabl&returnZ=true&spatialRel=esriSpatialRelIntersects&geometryType=esriGeometryPoint&inSR=102100&outSR=102100&returnGeometry=false&geometry={coordonnees_url_encodees}
```

**Important :** mettre a jour `annee_scol` selon l'annee scolaire en cours.

Reponse : chercher dans `features[].attributes` l'item ou `type_etabl == "COL"`. Le champ `libelle` donne le nom du college.

## Use Case 2 : Lycees de secteur d'un college

API ArcGIS du Rectorat. Necessite l'UAI du college (ex: `0752523K`).

```
GET https://services9.arcgis.com/ekT8MJFiVh8nvlV5/arcgis/rest/services/Affectation_Lyc%C3%A9es/FeatureServer/0/query
  ?outFields=UAI,Nom,secteur
  &returnGeometry=false
  &f=pjson
  &orderByFields=secteur
  &where=secteur<>'Tete' and Réseau='{uai_college}'
```

Reponse : liste de lycees avec leur UAI, nom, et secteur (1, 2 ou 3), triee par secteur croissant.

## Use Case 3 : Colleges ayant un lycee donne en secteur 1

Meme API ArcGIS, requete inversee. Necessite l'UAI du lycee.

```
GET https://services9.arcgis.com/ekT8MJFiVh8nvlV5/arcgis/rest/services/Affectation_Lyc%C3%A9es/FeatureServer/0/query
  ?outFields=Réseau,Nom_Tete
  &returnGeometry=false
  &f=pjson
  &orderByFields=Nom_Tete
  &where=secteur='1' and UAI='{uai_lycee}'
```

Reponse : le champ `Réseau` contient l'UAI du college, `Nom_Tete` le nom du college tete de reseau.

## Quick Reference

| Action | API | Cle d'entree |
|--------|-----|-------------|
| Adresse -> coordonnees | CapGeo Paris (geocodage) | Adresse parisienne |
| Coordonnees -> college secteur | CapGeo Paris (carte scolaire) | Coordonnees Web Mercator |
| College -> lycees secteur | ArcGIS Rectorat | UAI du college |
| Lycee -> colleges secteur 1 | ArcGIS Rectorat | UAI du lycee |

## Common Mistakes

- **Oublier la conversion de coordonnees** : le geocodage renvoie du Lambert-93, la carte scolaire attend du Web Mercator
- **Ne pas URL-encoder** les coordonnees JSON et l'adresse
- **annee_scol perimee** : verifier que l'annee scolaire dans la requete carte scolaire est a jour
- **Confondre UAI college/lycee** : dans l'API ArcGIS, `Réseau` = UAI du college, `UAI` = UAI du lycee

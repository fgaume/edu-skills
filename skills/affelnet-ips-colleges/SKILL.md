---
name: affelnet-ips-colleges
description: Use when the user asks about the IPS (Indice de Position Sociale) or IPS bonus of a Parisian college for Affelnet. Covers retrieving IPS data and associated Affelnet bonuses from a public Hugging Face dataset.
---

# IPS et bonus IPS des colleges de Paris

## Overview

Determination de l'IPS moyen et du bonus IPS Affelnet d'un college parisien, via un dataset Hugging Face public.

## When to Use

- L'utilisateur demande l'IPS d'un college parisien
- L'utilisateur veut connaitre le bonus IPS Affelnet d'un college
- L'utilisateur compare les IPS de plusieurs colleges parisiens

## Source de donnees

Toutes les donnees (IPS + bonus Affelnet) depuis 2021 sont dans ce dataset :

```
GET https://huggingface.co/datasets/fgaume/affelnet-paris-bonus-ips-colleges/raw/main/affelnet-paris-bonus-ips-colleges.json
```

## Regles

- **Sans precision sur l'annee** : toujours prendre l'annee la plus recente du dataset
- Si l'utilisateur precise une annee, filtrer en consequence

## Common Mistakes

- **Utiliser une annee perimee** : par defaut, toujours l'annee la plus recente
- **Confondre IPS et bonus IPS** : l'IPS est l'indice brut, le bonus est le nombre de points Affelnet associe

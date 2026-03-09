# Niveau scolaire des établissments et effectifs des admis au DNB

## Description
Un indicateur du niveau scolaire d'un collège est la note moyenne à l'écrit du DNB pour les élèves de 3e de ce collège.

## Détails
Un indicateur du niveau scolaire d'un collège s'obtient directement avec cette requête HTTP, qui retourne également le nombre de candidats et le taux de réussite (qui permet de déduire le nombre d'élèves admis au DNB) par exemple pour l'académie de PARIS pour la session 2024 :

https://data.education.gouv.fr/api/explore/v2.1/catalog/datasets/fr-en-indicateurs-valeur-ajoutee-colleges/exports/json?select=uai%2Cnote_a_l_ecrit_g%2Cnb_candidats_g%2Ctaux_de_reussite_g&lang=fr&refine=academie%3A%22PARIS%22&refine=session%3A%222024%22&facet=facet(name%3D%22academie%22%2C%20disjunctive%3Dtrue)&timezone=Europe%2FBerlin

Il est possible de filtrer public/privé avec le paramètre 'secteur' (secteur = 'PU' ou 'PR'), par exemple pour les collèges publiques parisiens, la requête devient :

https://data.education.gouv.fr/api/explore/v2.1/catalog/datasets/fr-en-indicateurs-valeur-ajoutee-colleges/exports/json?select=uai%2Cnote_a_l_ecrit_g%2Cnb_candidats_g%2Ctaux_de_reussite_g&lang=fr&refine=academie%3A%22PARIS%22&refine=session%3A%222024%22&facet=facet(name%3D%22academie%22%2C%20disjunctive%3Dtrue)&timezone=Europe%2FBerlin&refine=secteur%3A%22PU%22

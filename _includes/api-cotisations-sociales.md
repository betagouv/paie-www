API cotisations sociales
===================

[OpenFisca](http://www.openfisca.fr/) est un logiciel de simulation du système socio-fiscal français. C'est un projet *libre*,  aujourd'hui développé principalement par [Etalab](http://etalab.gouv.fr/), l'[IPP](http://ipp.eu/) et le l'[incubateur](https://beta.gouv.fr) de services numériques de l’État.

Particulièrement, le moteur permet de calculer les cotisations sociales (et les aides qui les réduisent) qui doivent être aujourd'hui payées par un employeur quand il verse un salaire. C'est à quelques différences près (dans les deux sens) ce qui constitue la fiche de paie.

OpenFisca est écrit en [Python](https://fr.wikipedia.org/wiki/Python_%28langage%29), mais surtout **rendu accessible directement sous la forme d'une API** complète couvrant toute la complexité des simulations possibles. **Pour le calcul des cotisations sociales, un point d'entrée simplifié, nommé `formula` a été créé** : il constitue l'API "cotisations sociales" et nous allons l'expliciter ici.

**Pour aller plus loin** dans la connaissance de cette API `/formula`, aidez-vous de ces ressources essentielles :

- la [description complète et intéractive](http://embauche.beta.gouv.fr/api/doc) de l'API cotisations sociales.
- la [documentation générale](http://doc.openfisca.fr/) d'OpenFisca (couvrant notamment le lancement d'une instance du moteur).
- [legislation.openfisca.fr](http://legislation.openfisca.fr/) pour explorer les règles et les paramètres d'OpenFisca.

## Les instances utilisables

L'instance principale et officielle est `api.openfisca.fr`.
L'instance utilisée et gérée par l'équipe du [simulateur](http://sgmap.github.io/cout-embauche/) de coût d'embauche est **`embauche.beta.gouv.fr/openfisca`**.

Cette dernière est privilégiée pour ce projet car elle nous permet de proposer des nouveautés toujours en attente de validation dans la branche principale (qui peuvent peuvent très bien marcher pour nous, mais impacter d'autres parties du moteur).

## Le point d'entrée principal

> A savoir : un des éléments les plus importants d'OpenFisca est la *variable*. Certaines sont des variables d'entrée (nous fournissons une valeur au moteur de calcul), d'autres des variables de sortie (elles sont calculées et renvoyées).


**Le point d'entrée qui nous concerne est très simple :**

 **`GET`**  `/api/2/formula/DATE/VARIABLES-DE-SORTIE ? VALEURS-D’ENTRÉE`


- DATE : la date de la simulation, sous la forme `YYYY-MM`
- VARIABLES-DE-SORTIE : une liste d'identifiants de variables que nous voulons recevoir en retour séparées par +.
- VALEURS-D’ENTRÉE :  une liste de couples `variable=valeur` séparés par &.

> Toutes les variables d'OpenFisca sont listées sur le très utile site d'exploration http://legislation.openfisca.fr/variables

#### Exemples :

Pour obtenir la valeur du plafond de la sécurité sociale (un exemple de variable) :

**`GET`** `/api/2/formula/2015-11/plafond_securite_sociale`

[essayer](https://embauche.beta.gouv.fr/openfisca/api/2/formula/2015-11/plafond_securite_sociale)

Obtenir la valeur du SMIC :

**`GET`** `/api/2/formula/2015-11/smic_proratise`

[essayer](https://embauche.beta.gouv.fr/openfisca/api/2/formula/2015-11/smic_proratise)

Pour obtenir les deux :

**`GET`** `/api/2/formula/2015-11/plafond_securite_sociale+smic_proratise`

[essayer](https://embauche.beta.gouv.fr/openfisca/api/2/formula/2015-11/plafond_securite_sociale+smic_proratise)

Ces exemples ne sollicitent aucun calcul du moteur : ces variables proviennent directement des `paramètres`, qui sont un peu comme une base de donnée des chiffres de la loi. Et en effet, nous n'avons pas renseigné de valeur d'entrée.

Voici un exemple plus complet :

**`GET`**

 `/api/2/formula/2015-11/vieillesse_plafonnee_salarie+vieillesse_plafonnee_employeur?effectif_entreprise=1&salaire_de_base=1458`

[essayer](https://embauche.beta.gouv.fr/openfisca/api/2/formula/2015-11/vieillesse_plafonnee_salarie+vieillesse_plafonnee_employeur?effectif_entreprise=1&salaire_de_base=1458)

Nous avons demandé le calcul des deux cotisations de vieillesse plafonnée, pour une entreprise de 1 salarié rémunéré 1458 euros.

> La vieillesse plafonnée est une cotisation que nous payons tous,  calculée sur la base du plafond de la sécurité sociale (notre premier exemple), et non sur l'ensemble du salaire : voila un exemple de `règle` de calcul interne à OpenFisca. Elles est complétée par la vieillesse déplafonnée.

Un dernier exemple, qui constitue une **simulation presque complète** des cotisations sociales et aides d'un salarié rémunéré au salaire médian :

[Exemple complet](https://embauche.beta.gouv.fr/openfisca/api/2/formula/accident_du_travail+famille+fnal+versement_transport+agff_salarie+agirc_salarie+agirc_gmp_salarie+apec_salarie+arrco_salarie+chomage_salarie+cotisation_exceptionnelle_temporaire_salarie+vieillesse_plafonnee_salarie+vieillesse_deplafonnee_salarie+mmid_salarie+csg_deductible_salaire+csg_imposable_salaire+crds_salaire+salaire_net_a_payer+salaire_super_brut+ags+agff_employeur+apec_employeur+arrco_employeur+chomage_employeur+cotisation_exceptionnelle_temporaire_employeur+vieillesse_deplafonnee_employeur+vieillesse_plafonnee_employeur+mmid_employeur+contribution_supplementaire_apprentissage+contribution_solidarite_autonomie+formation_professionnelle+participation_effort_construction+taxe_apprentissage+taxe_salaires+agirc_employeur+agirc_gmp_employeur+allegement_fillon+allegement_cotisation_allocations_familiales+exoneration_cotisations_employeur_apprenti+exoneration_cotisations_employeur_stagiaire+exoneration_cotisations_employeur_jei+credit_impot_competitivite_emploi+financement_organisations_syndicales+prevoyance_obligatoire_cadre+cout_du_travail+aide_premier_salarie+smic_proratise?effectif_entreprise=1&type_sal=prive_non_cadre&salaire_de_base=2300&code_postal_entreprise=&depcom_entreprise=&allegement_fillon_mode_recouvrement=anticipe_regularisation_fin_de_periode&allegement_cotisation_allocations_familiales_mode_recouvrement=anticipe_regularisation_fin_de_periode&jeune_entreprise_innovante=false&contrat_de_travail_debut=2016-2)


> Pour aller plus loin et voir un exemple d'utilisation exhaustif et à jour, il suffit d'analyser les requêtes que le [simulateur](http://sgmap.github.io/cout-embauche/) de coût d'embauche envoie à l'API.


## Les périodes

OpenFisca est prévu pour faire des calculs sur la période de l'année ou du mois. **Le point d'entrée `/formula` se concentre lui exclusivement sur le mois** (ce qui correspond aux besoins actuels du simulateur de coût d'embauche). Le méchanisme est simple : nous simulons la situation d'un salarié à salaire constant.

Certaines variables *doivent* être calculées à l'année, particulièrement les éxonérations applicables à des salaires inférieurs à une limite annuelle, par exemple allègement général sur les bas salaires ou l'allègement sur la cotisation d'allocations familiales. Pour demander leur calcul mensuel au moyen de ce point d'entrée, des paramètres supplémentaires sont à fournir : voir cette [note](https://github.com/sgmap/cout-embauche/wiki/Note-sur-le-calcul-des-all%C3%A8gements-%28Fillon-g%C3%A9n%C3%A9ral-et-allocations-familiales%29).

Pour l'instant, **le calcul des cotisations sociales n'est pas géré sur l'année**.

## Intégrer des extensions / réformes

Il est possible de modifier les variables et paramètres d'OpenFisca de façon modulaire grâce aux extensions. Il sera bientôt possible d'utiliser les extensions dans le point d'entrée `/formula`

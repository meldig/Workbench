# FICHE TECHNIQUE : Fusion des polygones

* **Objectif du Workbench :** fusionner tous les polygones simples et valides composant un plan d'eau, séparés par les limites communales, sur le territoire de la MEL.
* **Objets utilisés pour la fusion :** polygones simple (SDO_GTYPE = 2003), valides (GEO_ON_VALIDE = 0), s'intersectant ou se trouvant à 40 cm au plus les uns des autres.
* **Objets insérés dans la base :** tout objet résultant de la fusion de polygones
* **Objets mis à jour dans la base :** tout objet ayant servi à la fusion (la valeur du champ GEO_ON_VALIDE passant de 0 à 1 et champs GEO_DM et GEO_NMN étant mis à jour).
* **Durée du processus de fusion :** 22 secondes

## Explication du processus de traitement :

1. Chargement des données de plans d'eau valides (filtrage des données);
1. Création d'un identifiant unique par objet qui se mettra à jour automatiquement en cas d'insertion ou de modification d'objet ;
1. Comblement des espaces de 40 cm maximum entre les polygones adjacents ;
1. Fusion des polygones adjacents ;
1. Validation ou réparation des géométries créées ;
1. Distinction entre les polygones créés, ceux ayant servi à la fusion et ceux qui sont restés intouchés ;
1. Mise à jour des données attributaires des objets ayant servis à la fusion ;
1. Insertion des objets issus de la fusion dans la base ;

## Explications du processus de fusion :

1. **Chargement de la couche dans FME**

	![Reader](./images/fusion_polygones_plans_eau/reader.jpg)
	1. **Objectif :** charger uniquement les objets valides de type Eau ;
	1. **Reader** ;
	1. **Clause where :** "CLA_INU" = 325 AND "GEO_ON_VALIDE" = 0 -> La condition sur le champ CLA_INU permet de ne sélectionner que les polygones de type Eau et celle sur le champ GEO_ON_VALIDE les polygones dont la géométrie est valide ;
	1. **Spatial Column :** GEOM ;

1. **Création d'un identifiant unique par objet**

	![CRCCalculator](./images/fusion_polygones_plans_eau/CRCCalculator.jpg)

	1. **Objectif :** créer à l'aide d'un algorithme de cryptage un identifiant unique par objet, qui nous servira à distinguer les objets que l'on veut insérer de ceux à mettre à jour et des autres objets intouchés ;
	1. **Transformer :** CRCCALCULATOR ;
	1. **Raison de son placement :** obtenir un identifiant unique des objets avant fusion et des objets créés par la fusion (création automatique pour ces derniers dû à l'utilisation d'un agorithme de cryptage) ;
	1. **CRC Algorithm :** MD5 -> Type de cryptage ;
	1. **Calculate CRC on :** Coordinates and Selected Attributes -> afin d'avoir un identifiant unique, uniquement créé à partir de leur géométrie, pour les objets issus de la fusion ;
	1. **CRC Output Attribute :** crc -> nom du nouveau champ contenant les identifiants des objets ;

1. **Suppression des espaces entre polygones adjacents**

	![AreaGapAndOverlapCleaner](./images/fusion_polygones_plans_eau/AreaGapAndOverlapCleaner.jpg)

	1. **Objectif :** Combler les espaces entre les polygones, s'il y en a, afin de pouvoir les fusionner ;
	1. **Transformer :** AreaGapAndOverlapCleaner ;
	1. **Group By Mode :** Process at End (Blocking) -> le transformer ne fonctionnera qu'une fois tous les objets chargés ;
	1. **Tolerance :** 0.4 m ;
	1. **Aggregate Handling :** Deaggregate -> afin de n'avoir que des polygones simples non agrégés ;
	1. **Conect Z mode :** Ignore -> l'altitude n'est pas prise en compte ;
	1. **Measures / Z Conflict value :** compute ;
	1. **Treat Measures as :** Continuous ;
	1. **Fill All Gaps :** No -> seuls les trous dont le diamètre n'excède pas la tolérance seront comblés ;
	1. **Repair Method :** Priority (Edge Preserving) -> l'objectif est de conserver la forme d'origine en comblant les trous, mais pas de déformer la géométrie. Par contre, cela implique que le polygone dont le côté est le plus large verra sa géométrie appliquée au polygone avec lequel il fusionne. Ce dernier sera donc élargit de quelques millimètres. Pour plus d'informations consultez l'aide depuis le transformer ;
	1. **Priority Attribute :** OBJECTID ;

1. **Fusion des polygones**

	![Dissolver](./images/fusion_polygones_plans_eau/Dissolver.jpg)

	1. **Objectif :** Fusionner les polygones adjacents, afin d'obtenir un seul polygone par plan d'eau (et non un plan d'eau composé de plusieurs polygones et divisé par les limites communales) ;
	1. **Transformer :** Dissolver ;
	1. **Group By Mode :** Process at End (Blocking) -> le transformer ne fonctionnera qu'une fois tous les objets chargés ;
	1. **Tolerance :** 0 (puisque tous les espaces entre polygones adjacents ont été comblés par le transformer précédent) ;
	1. **Connect Z Mode :** Ignore -> l'altitude n'est pas prise en compte ;
	1. **Aggregate Handling :** Deaggregate -> désaggège tous les polygones qui seraient restés aggrégés ;
	1. **Accumulation Mode :** Use Attributes From One Feature (pas d'utilité particulière dans notre cas) ;

1. **Vérification de la validité des géométries créées**
![GeometryValidator](./images/fusion_polygones_plans_eau/GeometryValidator.jpg)

 1. **Objectif :** Vérifier que les géométries produites sont valides ;
 1. **Transformer :** GeometryValidator ;
 1. **Paramètres :** 	
	* Degenerate or Corrupt Geometries + Edit - Connect Z mode : Ignore ET Segment EndPoint Accuracy Mode : Auto ;
	* Surface Orientation ;
	* Contains Null Geometry Parts -> Vérifie qu'il n'y a pas de géométrie nulle ;
	* Self-Intersections in 2D + Edit - Check Self-Touching Polygon : No (afin de ne pas combler les trous des donuts dont les bords intérieurs touchent les bords extérieurs du polygone) ET Connect Z mode : Ignore;
	* Tolerance : Automatic ;
 1. **Attempt Repair :** Yes -> Les réparations sont visualisables dans l'aperçu de FME. Il faudra donc d'abord visualiser les réparations effectuées afin d'évaluer leur pertinence, pour ensuite les insérer dans la base ou un shape. Le chaînage actuel ne sauvegarde pas les données réparées ;

1. **Séparation des objets créés des objets ayant servis à la fusion et des objets inchangés**

![ChangeDetector](./images/fusion_polygones_plans_eau/ChangeDetector.jpg)

 1. **Objectif :** Différencier les nouveaux polygones issus de la fusion de ceux qui ont servis à la faire et des autres polygones de la table chargée ;
 1. **Transformer :** ChangeDetector ;
 1. **Check Attributes :**
  *  Attribute Matching Strategy : Match Selected Attributes
  * Selected Attributes : crc ;
 1. **Attribute Comparisons :**
  * Differentiate Empty, Missing, and NULL Attributes : No ;  
  * Check Attribute Types and Encodings : No ;
 1. **Check Geometry :**
  * Match Geometry : 2D ;
  * Lenient Geometry Matching : Yes ;
  * Check Coordinate Systems : No ;
  * Vector Tolerance : 0 ;
 1. **Advanced Output Options : **
  * Unchanged Output Ports : Output Original ;

1. **Mise à jour des champs des polygones fusionnés :**

![MAJ_Polygones_fusionnés](./images/fusion_polygones_plans_eau/MAJ_Polygones_fusionnés.jpg)

 1. **Objectif :** Mise à jour attributaire des polygones fusionnés ;
 1. **Transformer :** AttributeManager ;
 1. **Parameters :**
  * CLA_INU = 325 ;
  * GEO_ON_VALIDE = 0 ;
  * GEO_NMS : 'nom_de_la_personne_utilisant_le_workbench' **A rentrer à la main AVANT la fusion** -> Exemple : bjacq ;
  * GEO_DS : DateTime(local) -> insérer la date du jour de création de la donnée ;

1. **Mise à jour de la base Oracle**

![DatabaseUpdater](./images/fusion_polygones_plans_eau/DatabaseUpdater.jpg)

 1. **Objectif :** Mise à jour attributaire des polygones de la base Oracle ayant servi à la fusion, afin de les désactiver ;
 1. **Transformer :** DatabaseUpdater ;
 1. **Database To Update :**
  * Format : Oracle Spatial Object ;
  * Connexion : GEO (CUDL) ;
 1. **Parameters :**
  * Table : GEO.TA_SUR_TOPO_G ;
  * Condition : Match Column(s) ;
  * Match On : OBJECTID (de la base) = OBJECTID (des données à MAJ) | CLA_INU = 325 ;
  * Columns to Update : GEO_ON_VALIDE = 1 | GEO_DM = DateTime(local) | GEO_NMN = 'nom_de_la_personne_utilisant_le_workbench' **A rentrer à la main AVANT la fusion**-> la mise à jour des champs GEO_DM et GEO_NMN permet de sélectionner tous les polygones ayant servi à la fusion, une fois celle-ci effectuée ;

1. **Insertion des polygones fusionnés dans la base**

![Writer](./images/fusion_polygones_plans_eau/Writer.jpg)

  1. **Objectif :** Insérer uniquement les polygones issus de la fusion dans la table TA_SUR_TOPO_G d'Oracle ;
  1. **Writer** ;
  1. **General :**
   * Table Name : TA_SUR_TOPO_G ;
   * Table Qualifier : GEO (CUDL) ;
   * Writer : GEO [ORACLE_SPATIAL] - 2 ;
  1. **Table - General :**
   * Feature Operation : Insert ;
   * Table Handling : Use Existing ;
  1. **Table - Spatial :**
   * Spatial Column : GEOM ;
   * Spatial SRID : 2154 ;

## Après la fusion - Visualisation des données dans SQL Developper:

### Sélection des objets de plans d'eau valides (objets fusionnés compris) :

```sql
SELECT *
FROM
	GEO.TA_SUR_TOPO_G a
WHERE
	a.CLA_INU = 325
	AND a.GEO_ON_VALIDE = 0;
```

### Sélection des objets issus de la fusion :

```sql
SELECT *
FROM
	GEO.TA_SUR_TOPO_G a
WHERE
	a.CLA_INU = 325
	AND a.GEO_ON_VALIDE = 0
	AND a.GEO_NMS = 'bjacq'
	-- Valeur renseignée dans le workbench (transformers : MAJ_Polygones_fusionnés | DatabaseUpdater )
	AND a.GEO_DS BETWEEN '07/07/19' AND '09/07/19'
	-- Le champ étant de type date, il faut utiliser une fouchette de dates pour sélectionner la date de modification, c'est-à-dire ici le 08/07/19 ;
```

### Sélection des objets ayant servis à la fusion :

```sql
SELECT *
FROM
	GEO.TA_SUR_TOPO_G a
WHERE
	a.CLA_INU = 325
	AND a.GEO_ON_VALIDE = 1
	AND a.GEO_NMN = 'bjacq'
	AND a.GEO_DM BETWEEN '07/07/19' AND '09/07/19';
```

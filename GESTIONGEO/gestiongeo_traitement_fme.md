# Présentation des chaines FME mise à jour dans le cadre du projet GESTIONGEO.

Dans le cadre de la refonte du projet GESTIONGEO la chaine FME utilisée pour intégrer en base les éléments d'un dossier a été mise à jour. Deux autres chaines ont été écrites pour mieux gérer les dossiers.

## Traitement gestion_fme2020_deplacement_recol_ic.fmw.

Ce traitement fme est une mise à jour de la chaine __*acad_recV13_20161026.fmw*__ développé initialement pour le projet *gestiongeo* fonctionnant avec dynmap. Cette chaine fme peut être utilisée par l'intermédiaire de GTF.


### Objectif de la chaine de traitement.

L'objectif de la chaine, est d'insérer en base les éléments topographiques contenus dans le fichier *autocad* d'un dossier. Ce traitement va réaliser plusieurs actions.


#### Si le dossier est un récolement.

1. Les éléments *autocad* vont être reprojetés en LAMBERT-93.
2. La chaine va vérifier que les éléments sont situés à l'intérieur du périmètre de la MEL. Utilisation de la couche __G_REFERENTIEL.ADMIN_PERIMETRE_MEL__.
3. Insérer les éléments dans les tables:
	* __GEO.PTTOPO__.
	* __GEO.TA_LIG_TOPO_GPS__.
	* __GEO.TA_POINT_TOPO_GPS__.
4. Calculer l'enveloppe concave des éléments.
5. Mise à jour ou insertion de la géométrie du dossier dans la table G_GESTIONGEO.TA_GG_GEO.
6. Mise à jour de la table G_GESTIONGEO.TA_GG_DOSSIER: colonne ETAT_ID.
7. Gestion des fichiers attachés au dossier:
	* Mise à jour de la table G_GESTIONGEO.TA_GG_URL_FILE.
	* Déplacement des fichiers situés dans le même répertoire que le fichier *autocad* intégré dans la base dans le répertoire du dossier situé dans *Appli_GG*.


#### Si le dossier est une investigation complémentaire.

1. Les éléments *autocad* vont être reprojetés en LAMBERT-93.
2. La chaine va vérifier que les éléments sont situés à l'intérieur du périmètre de la MEL. Utilisation de la couche __G_REFERENTIEL.ADMIN_PERIMETRE_MEL__.
3. Calculer l'enveloppe concave des éléments.
4. Mise à jour ou insertion de la géométrie du dossier dans la table __G_GESTIONGEO.TA_GG_GEO__.
5. Mise à jour de la table __G_GESTIONGEO.TA_GG_DOSSIER__: colonne ETAT_ID.
6. Gestion des fichiers attachés au dossier:
	* Mise à jour de la table __G_GESTIONGEO.TA_GG_URL_FILE__.
	* Déplacement des fichiers situés dans le même répertoire que le fichier *autocad* intégré dans la base dans le répertoire du dossier situé dans *Appli_GG*.


### Utiliser la chaine Intégration des relevés géomètres, récolement et IC.

Pour utiliser, la chaine il est nécessaire:

1. De se rendre sur l'application GTF à l'adresse suivante: https://gtf.lillemetropole.fr/extraction/login.
2. cliquer sur __demande__.
3. Cliquer sur __ajouter__.
4. sélectionner la chaine *Intégration des relevés géomètres, récolement et IC*.


#### Paramètres nécessaires pour lancer le traitement

1. Numéro de dossier: Le numéro de dossier est obtenu suite à la création d'un dossier avec *QGIS*.
2. Fichier à intégrer: ajouter le fichier *autocad* lié au dossier.
3. Préciser si le dossier est un *Recolement* ou une *Investigation complémentaire*.
4. Préciser le système de projection du fichier *autocad* .


## Traitement gestion_fme2020_deplacement_recol_ic.fmw.

### Objectif de la chaine de traitement.

Ce traitement *FME* permet de déplacer une fois le dossier créé et ses éléments intégrés en base, des fichiers qui lui sont liés (compte rendu *pdf*, word) dans le répertoire du dossier créé sur *Appli_GG*.


### Action réalisé par le traitement.

Ce traitement permet de:

1. Déplacer un fichier attaché à un dossier dans son répertoire dossier sur *Appli_GG*.
2. Mettre à jour la table G_GESTIONGEO.TA_GG_URL_FILE.


### Utiliser la chaine Ajout de fichier dans le répertoire d'un dossier de APPLI_GG.

Pour utiliser la chaine il est nécessaire:

1. De se rendre sur l'application GTF à l'adresse suivante: https://gtf.lillemetropole.fr/extraction/login.
2. cliquer sur __demande__.
3. Cliquer sur __ajouter__.
4. selectionner la chaine *Ajout de fichier dans le répertoire d'un dossier de APPLI_GG*.


### Paramètres nécessaires pour lancer le traitement

1. Numéro de dossier associé au fichier: le numéro de dossier est obtenu suite à la création d'un dossier avec *QGIS*.
2. Fichier à déplacer: ajouter le fichier lié au dossier à déplacer.


## Traitement gestion_fme2020_suppression_recol_ic.fmw.

### Objectif de la chaine de traitement.

Le but de cette chaine de traitement est de reproduire la fonction *INVALIDER UN DOSSIER* qui était présente dans l'application *DYNMAP GESTIONGEO*. Cette chaine va supprimer un dossier en base, tous les éléments qui lui sont attachés ainsi que le répertoire du dossier présent sur *Appli_GG*.

Il faut préciser qu'une fois le traitement exécuté, le dossier et ses éléments sont supprimés. Aucun retour en arrière n'est possible. Pour réintégrer les éléments il sera nécessaire de créer un nouveau dossier.


#### Actions réalisées 

Ce traitement permet:

1. De supprimer l'ensemble des éléments d'un dossier dans les tables:
	* GEO.PTTOPO.
	* GEO.TA_LIG_TOPO_GPS.
	* GEO.TA_POINT_TOPO_GPS.
	* G_GESTIONGEO.TA_GG_DOSSIER.
		* La contrainte de clé primaire __ON CASCADE__ présente sur la colonne ID_DOS de la table G_GESTIONGEO.TA_GG_DOSSIER supprimera les enregistrement du dossier présents dans les tables:
			* G_GESTIONGE.TA_GG_GEO.
			* G_GESTIONGEO.TA_GG_URL_FILE.
2. Supprime le répertoire du dossier présent sur *Appli_GG*.


### Utiliser la chaine Ajout de fichier dans le répertoire d'un dossier de APPLI_GG.

Pour utiliser la chaine il est nécessaire:

1. De se rendre sur l'application GTF à l'adresse suivante: https://gtf.lillemetropole.fr/extraction/login.
2. cliquer sur __demande__
3. Cliquer sur __ajouter__
4. selectionner la chaine *Invalider un dossier*


### Paramètres nécessaires pour lancer le traitement

Numéro du dossier a supprimer: Le numéro de dossier est obtenu suite à la création d'un dossier avec *QGIS*.
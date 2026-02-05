# projet-demo-gestion-commerciale
Projet de démonstration WinDev – gestion commerciale (clients, produits, factures) utilisé comme support pour un POC de génération de documentation assistée par IA.

## Documentation
La documentation détaillée du projet se trouve dans le dossier `/docs` :
- `PROJECT_OVERVIEW.md` : vue d’ensemble du projet
- `SCREENS.md` : description des écrans
- `DATA_MODEL.md` : modèle de données
- `WORKFLOWS.md` : parcours fonctionnels

## IA & Documentation
Le code WinDev n’est pas directement analysable par les outils IA actuels.
La documentation est donc **générée de manière assistée par IA** à partir
des fichiers textuels présents dans le dossier `/docs`.

Cette approche permet :
- une documentation claire et maintenable
- une compatibilité avec les outils IA
- un versionnement cohérent avec le code

##  Technologies
- Langage : WinDev (WLangage)
- Base de données : HFSQL
- Gestion de version : Git / GitHub

##  Statut
Projet de démonstration / Proof of Concept (POC)

##  Documentation fonctionnelle du ProjetDemo_GestionCommerciale — (WinDev)

Documentation fonctionnelle orientée MOA / métier, générée à partir des fichiers WinDev fournis sans interpréter les propriétés internes encodées (ex. internal_properties).
Elle s’appuie uniquement sur : noms de fichiers, noms des fenêtres/volets/tables/boutons, libellés visibles, code WLangage lisible.

Table des matières

1) Vue d’ensemble métier

2) Navigation & ergonomie générale

3) Référentiels & constantes (projet)

4) Écrans “Clients”

4.1 Liste / Table clients

4.2 Fiche client

5) Écrans “Produits”

5.1 Liste / Table produits

5.2 Fiche produit

6) Flux “Commande” → “Facture”

6.1 Fiche commande

6.2 Fiche facture

7) États / impressions

7.1 État clients

7.2 État facture

8) Parcours utilisateur (synthèse)

9) Points d’attention (qualité fonctionnelle)

Sources analysées

1) Vue d’ensemble métier

Le projet est une démo de gestion commerciale centrée sur 4 objets :

Client

Produit (avec déclinaisons / références)

Commande

Facture

États (impressions) : liste clients, facture

Le fonctionnement “écran” est basé sur une fenêtre à onglets dynamiques : lorsqu’un utilisateur ouvre une fiche (client/commande/facture/produit), elle s’ouvre dans un volet (onglet) et peut être réactivée si elle existe déjà.

2) Navigation & ergonomie générale
Fenêtre principale : FEN_Onglet

Rôle fonctionnel :

Sert de conteneur d’affichage (onglets/volets) via ONG_Menu.

Gère l’ouverture/activation d’écrans internes dans des volets.

Comportement clé (procédure) :

OuvreOuActiveVolet(sNomFenetre, sLibelle, *)

Construit un alias (basé sur le nom de fenêtre + libellé).

Si le volet n’existe pas : ouvre un nouvel onglet.

Sinon : active l’onglet existant.

Gestion d’un cas “CTRL enfoncé” (comportement d’activation temporaire).

Conséquence côté utilisateur :

Ouvrir une fiche depuis une liste n’ouvre pas forcément un doublon : l’application peut réutiliser l’onglet existant.

3) Référentiels & constantes (projet)
Projet : ProjetDemo_GestionCommerciale.tkp

Constantes visibles :

CST_TOUTAFFICHER = 1

CST_FILTRE = 2

REP_PHOTOS_PRODUITS = "produits"

CST_CLÉGOOGLE = "" (clé Google Maps)

Licence Google Maps initialisable via CarteLicenceGgl(CST_CLÉGOOGLE) si la clé est renseignée.

Implication métier :

Le projet peut afficher une carte / adresse (fonctionnellement lié aux fiches client et à l’adresse).

Mode “Tout afficher” vs “Filtré” utilisé dans les tables (clients/produits).

4) Écrans “Clients”
4.1 Liste / Table clients — FI_Volet_Table_Client

Objectif :

Afficher et rechercher des clients.

Ouvrir la fiche client.

Actions de gestion : nouveau / suppression (selon boutons présents).

Contrôles repérables (par nom) :

Table : TABLE_Client (colonnes techniques repérées : COL_ID, et une colonne calculée COL_NomComplet)

Boutons : BTN_Nouveau (au moins), et un mécanisme de suppression (dialogue confirmé).

Fonctionnalités déduites du code :

Nom complet calculé :

COL_NomComplet = FormateNomComplet(COL_Civilité, COL_Nom, COL_Prénom)

Ouverture de la fiche client :

Ouvre un volet via FEN_Onglet.OuvreOuActiveVolet(FI_Volet_Fiche_Saisie_Client, "Client " + TABLE_Client.COL_NomComplet, TABLE_Client.COL_ID)

Recherche / filtre

Si tous les champs de recherche sont renseignés (Nom, Prénom, Téléphone, Code postal, Ville, Email) :

Mode CST_TOUTAFFICHER, table alimentée par le fichier client.

Sinon :

Exécution d’une requête REQ_RechercheClient avec paramètres (Nom/Prénom/CP/Ville/Tel/Email)

Si aucun résultat : toast “Aucun résultat trouvé”

Sinon : table alimentée par le résultat de requête.

Suppression (confirmation)

Message visible :

“Êtes-vous sûr de vouloir supprimer le client ?”

Boutons : “Supprimer” / “Ne pas supprimer”

4.2 Fiche client — FI_Volet_Fiche_Saisie_Client

Objectif :

Créer / modifier un client

Gérer l’adresse du client

Visualiser documents liés (commandes / factures) via une table

Ouvrir une commande ou une facture depuis la liste associée

Chargement de la fiche (mode création vs modification)

Si un identifiant client (gId de type UUID) est fourni :

Recherche du client HLitRecherche(Client, UUIDClient, gId)

FichierVersEcran()

Affichage d’une info “date de création” :

Libellé visible du type : “Créé le …”

Chargement de l’adresse défaut (via Adresse + index [gid,1]) et alimentation des champs SAI_Adresse...

Chargement des factures/commandes du client : ChargerFacturesCommandes(gid)

Sinon (nouveau client) :

RAZ client HRAZ(Client)

Cache la date de création

Désactive des actions :

BTN_Supprimer grisé

BTN_Commande grisé

BTN_devis grisé (un flux “devis” semble prévu côté UI)

Gestion des modifications (fermeture)

À la fermeture si écran modifié :

Demande de confirmation (dialogue) :

“Voulez-vous fermer sans sauver vos modifications ?”

“Voulez-vous enregistrer vos modifications ?”

Choix visibles :

“Oui, enregistrer et fermer” (déclenche BTN_Sauver)

“Non, fermer sans enregistrer”

“Annuler”

Contrôle de validité email

Vérification par expression régulière :

En cas d’erreur : affichage “Saisissez une adresse email valide” + mise en forme du champ (stylo rouge).

Carte / affichage adresse

Un composant CFI_Carte est rendu visible et affiche l’adresse concaténée (voie, complément, CP/ville, province, pays).

Accès aux documents (commande / facture)

Depuis une table (ex : TABLE_CdesFactures) :

Si type doc = “Commande” → ouvre FI_Volet_Fiche_Saisie_Commande

Si type doc = “Facture” → ouvre FI_Volet_Fiche_saisie_Facture

Rafraîchissement automatique

Notifications :

DeclareProcedureNotification("Modif.Document", ...) → rafraîchit la table courante

DeclareProcedureNotification("Ajout.Document", ...) → recharge la liste des documents

5) Écrans “Produits”
5.1 Liste / Table produits — FI_Volet_Table_Produit

Objectif :

Afficher et rechercher des produits

Accéder à la fiche produit

(Suppression via confirmation — attention incohérence texte)

Recherche / filtre

Si libellé et référence sont renseignés :

Mode CST_TOUTAFFICHER, table alimentée par le fichier produit

Sinon :

Exécute REQ_RechercheProduit (ParamLibelleProduit / ParamReference)

Toast “Aucun résultat trouvé” si vide

Sinon table sur requête.

Suppression (confirmation)

⚠️ Texte visible copié/collé :

“Êtes-vous sûr de vouloir supprimer le client ?” (dans l’écran produit)
→ à corriger fonctionnellement : devrait parler de “produit”.

5.2 Fiche produit — FI_Volet_Fiche_Saisie_Produit

Objectif :

Créer / modifier un produit

Gérer/afficher ses déclinaisons (références)

Déclinaisons produit

La table/liste de déclinaisons repose sur la requête paramétrée :

REQ_DéclinaisonsProduitDunProduit

Paramètre : ParamIDProduit = gId

Gestion fermeture avec modifications

Même logique que les fiches : confirmation si modifié, possibilité de sauver via BTN_Sauver.

6) Flux “Commande” → “Facture”
6.1 Fiche commande — FI_Volet_Fiche_Saisie_Commande

Objectif :

Saisir une commande (client + lignes)

Calculer totaux HT/TVA/TTC

Générer une facture à partir d’une commande

Cycle de vie / états

Présence d’une combo COMBO_EtatCommande.

Un scénario force un état (ex : CDE_ATTENTEREGLEMENT) puis sauve et ferme.

Lignes de commande

Table : TABLE_LigneCde (colonnes repérées : référence, libellé, prix, quantité, sous-total, taux taxe…)

Fonctions visibles :

Contrôle de référence :

Vérifie existence de la déclinaison produit via DéclinaisonProduit + champ Reference.

Si référence invalide : propose de supprimer la ligne.

Calcul sous-total :

COL_SousTotalHT = COL_Quantite * COL_PrixUnitaireHT

Suppression ligne :

TableSupprime(TABLE_LigneCde, TableSelect(...))

Marque table modifiée.

Ajout ligne (contrôles de saisie)

Quantité doit être > 0 sinon message : “Vous devez saisir une quantité valide.”

Référence doit être valide sinon : “Vous devez saisir une référence valide.”

Totaux

Procédure dédiée : CalculeTotaux()

Parcourt toutes les lignes de TABLE_LigneCde

Cumule :

Total HT

Total TVA (à partir taux taxe par ligne)

Total TTC

Saisie assistée (aide à la saisie)

Procédure : SaisieAssistée()

Fonction de type “auto-complétion” sur référence / libellé.

Utilise une zone répétée ZR_DéclinaisonProduit :

Sélection remplit les champs d’ajout : référence + libellé

Ferme le plan de saisie assistée (Plan = 0)

Facturation (création facture)

Avant facturation : resauve la commande.

Création facture via factureCrée()

Si échec : “Erreur ! Facturation impossible !”

La facture reprend :

UUIDClient

NumCommande

DateCommande (selon champs commande)

Les lignes / totaux sont ensuite exploités côté facture.

6.2 Fiche facture — FI_Volet_Fiche_Saisie_Facture

Objectif :

Afficher une facture (et ses lignes)

Afficher le client lié

Préparer l’impression (via état)

Chargement

Recherche facture par gnIDFacture.

Affiche :

Client (via fonction ClientNomComplet(facture.UUIDClient))

Informations facture via FichierVersEcran()

Recherche commande associée (via NumCommande) :

Exécute REQ_LignesCdeDuneCde.ParamIDCommande

Ajoute les lignes dans TABLE_LigneCde (référence, libellé, prix HT, quantité, sous-total, taux taxe…)

Libellé total

Mise à jour d’un libellé : LIB_Total = <préfixe> + SAI_TotalTTC

Fermeture avec modifications

Confirmation si modifié (sauver / ne pas sauver / annuler).

7) États / impressions
7.1 État clients — ETAT_Clients

Objectif :

Imprimer/exporter une liste de clients.

Éléments visibles (noms de champs) :

En-tête document : TITREDOC, HAUT_DE_PAGE

Colonnes/libellés :
LIB_COL_NomComplet, LIB_COL_TelFixe, LIB_COL_TelPortable, LIB_COL_Email, LIB_COL_CodePostal, LIB_COL_Ville

Corps : sections correspondantes (ex : LIB_COL_NomComplet1, etc.)

7.2 État facture — ETAT_Facture

Objectif :

Imprimer/exporter une facture.

Procédure d’alimentation :

monetat(gnIDFacture, gnIDCommande)

Initialise les paramètres de requête : REQ_Facture.paramnumfacture = gnIDFacture

Prépare des variables (ex : gsDateLimiteRèglement)

Éléments visibles :

Sections : DEBUT_DOCUMENT, CORPS

Champs RTF (contenu riche) : RTF_SansNom, RTF_SansNom1

Rubrique “à payer” : RUB_APayer

Plusieurs libellés “SansNom” (à renommer idéalement pour maintenance fonctionnelle).

8) Parcours utilisateur (synthèse)
Parcours “Gestion client”

Ouvrir FI_Volet_Table_Client

Rechercher un client (filtres) ou afficher tout

Double-clic / action → ouvre FI_Volet_Fiche_Saisie_Client dans un onglet

Modifier infos + adresse (validation email, affichage carte)

Sauver

Depuis la fiche client, consulter commandes/factures et ouvrir le détail

Parcours “Commande”

Depuis un client, ouvrir une commande (FI_Volet_Fiche_Saisie_Commande)

Ajouter lignes (référence/quantité, saisie assistée)

Vérification références

Calcul automatique des totaux

Sauver

Générer facture (bouton/traitement) → création facture liée

Parcours “Facture”

Ouvrir FI_Volet_Fiche_Saisie_Facture

Visualiser client, entête facture, lignes issues de la commande

Imprimer via ETAT_Facture

9) Points d’attention (qualité fonctionnelle)

Message de suppression sur l’écran produit : texte “supprimer le client” (incohérent) → à corriger.

Beaucoup de champs “SansNom” dans l’état facture → complique la lecture métier.

Les fiches reposent fortement sur la logique “fenêtre modifiée” → bon point UX (confirmation de sortie).

Sources analysées

Sources analysées (sans décoder internal_properties) :
ProjetDemo_GestionCommerciale.tkp, FEN_Onglet.bkw,
FI_Volet_Table_Client.bkw, FI_Volet_Fiche_Saisie_Client.bkw,
FI_Volet_Table_Produit.bkw, FI_Volet_Fiche_Saisie_Produit.bkw,
FI_Volet_Fiche_Saisie_Commande.bkw, FI_Volet_Fiche_Saisie_Facture.bkw,
ETAT_Clients.bke, ETAT_Facture.bke.

Si tu veux, je peux aussi te générer une structure de repo (ex. docs/functional/README.md, docs/screens/clients.md, etc.) tout en gardant exactement le même contenu, juste découpé proprement pour GitHub.

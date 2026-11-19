# Historique des versions — Mémo MSDGN

## v2.1 (novembre 2026) — base reconfigurée : code personnel unique

Même application, même banque (409 questions, `409q-v1`), même barème (`B1`), **même format de résultats** (schéma 2, 23 colonnes + somme de contrôle). Seule l'identification change ; `version_app` vaut `2.1` dans les lignes de résultats.

### Ce qui change par rapport à la v2.0
- **Un seul code par élève** : « code personnel » de la forme `mot-mot-nn` (deux mots et deux chiffres). Plus de code de groupe : l'équipe est **déduite du code** par `config.json`. Saisie tolérante : majuscules, tirets, espaces et accents sont ignorés (`Lune-Pont 42` = `lunepont42`).
- **Vérification** : PBKDF2-SHA256 (WebCrypto, 100 000 itérations, sel public `msdgn2|`, 128 bits en hexadécimal) comparé à `codes_eleves` dans `config.json`. Le code lui-même n'est pas dans l'application. Un code inconnu est refusé ; sans configuration en cache, l'élève doit se connecter une fois à internet.
- **Colonne `eleve`** des résultats : le code normalisé (ex. `lunepont42`).
- **`config.json` schéma 2** : disparaît `codes_equipe` ; apparaissent `codes_eleves` (empreinte → équipe), `kdf` (paramètres du hachage), `contact_resultats` (adresse d'envoi, modifiable sans redéployer) et `conservation_jusqu_au` (date maximale, AAAA-MM-JJ). Un schéma 1 est encore lu (identification impossible tant que `codes_eleves` est absent).
- **Écran « Données et vie privée »** : texte validé (document `05_Textes_eleves`), adresse et date insérées depuis `config.json` ; adresse affichée aussi sous « Envoyer mes résultats ».
- **Pied de page** : « v2.1 » seulement pour les élèves (plus de mention de phase) ; nouvelle formule : « Les données restent sur ton appareil ; l'application n'en envoie aucune. » ; la ligne de droits est inchangée.
- **Textes au masculin** (« ton enseignant »).
- **Mode test** : code personnel libre + liste d'équipes simulées (plus de code de groupe).
- **Migration** : une identité enregistrée par la v2.0 (code élève + code de groupe) est effacée au premier lancement de la v2.1 ; les réponses déjà enregistrées sont conservées. La v1 déployée n'enregistrait rien sur les téléphones : aucun élève n'est concerné.
- **Service worker** `msdgn-v21`.

### Limites connues
- Pas de protection contre l'usurpation : un élève qui connaît le code d'un camarade peut l'utiliser.
- Le `config.json` livré contient **14 codes d'exemple publics** : à remplacer par les codes du générateur (étape 4) avant tout usage réel.
- Vérification du code impossible hors WebCrypto (navigateur ancien ou page non sécurisée).

## v2.0 (novembre 2026) — première version « jeu »

Dossier : `quiz-mobile-jeu/`. Une seule page `index.html` ; la **phase** (`base` ou `ludique`) est lue dans `config.json`. Passer de l'une à l'autre = modifier une ligne de `config.json` sur GitHub, sans toucher à l'application.

### Ce qui change par rapport à la v1 (version déployée : `msdgn-v1`)
- **Identité** : plus de prénom ni de nom. L'élève saisit un *code élève* et un *code de groupe* (équipe). Le code de groupe est vérifié par un hash SHA-256 (le code lui-même n'est pas dans l'application).
- **Banque** : 409 questions, identifiant stable `ID` (ex. `C2-014`) en première colonne. Source : `banque_questions_modele_corrige.csv`. Écart avec la banque embarquée dans la v1 déployée : 8 lignes (libellé « Notion » de C4-050, 052, 054, 055, 056, 057 ; « Notion » et explication de C4-053 ; choix A de C5-024) ; la v2 reprend la version corrigée.
- **Format de résultats v2** : une ligne par réponse (23 colonnes + somme de contrôle). Voir « Colonnes » ci-dessous.
- **Export** : bouton « Envoyer mes résultats » (partage natif sur téléphone, téléchargement sinon). Plus de téléchargement automatique sur téléphone ; il reste automatique sur ordinateur. Le fichier est le **journal cumulé** de l'appareil ; l'outil de fusion élimine les doublons.
- **Correctif** : questions à choix multiples — toucher le texte d'une proposition cochait puis décochait la case (défaut présent dans toutes les versions précédentes).
- **Mode test caché** (identifiant enseignante saisi dans « Code élève ») : phase forcée, date simulée, hors ligne simulé, configuration périmée simulée, codes simulés. Données de test dans un espace séparé, lignes marquées `TEST`, ignorées par l'outil de fusion.
- **Service worker** `msdgn-v2` : `config.json` et `classement.json` ne sont jamais interceptés (l'application garde sa propre copie hors ligne).
- **Pas de date codée en dur** : changement de phase manuel.

### Révision 2.0b — effets « Arcade » (phase ludique uniquement)
Explosions de particules plein écran (canvas, sans bibliothèque), feux d'artifice à chaque bonne réponse, combos x3/x5/x10 (« COMBO », « INARRÊTABLE »), flash coloré unique (jamais de clignotement rapide), lueur de flammes en bas d'écran pendant les séries, secousse + éclats rouges sur erreur, vibration brève, bruitages arcade (son coupé par défaut), plus gros effets pour niveau, badge et défi du jour. Réglage « ✨ Effets » : Calme / Arcade (défaut) / Max, mémorisé sur le téléphone. `config.json` peut imposer un niveau avec la clé facultative `"effets_force": "calme"|"arcade"|"max"` (le bouton disparaît alors). Animations coupées si le téléphone demande de réduire les animations. Le numéro de version des résultats reste `2.0`.

### Phase « ludique »
Points par format (QCM 10, VF 10, TROUS 12, QCM_MULTI 15, ASSO 15) ; bonus de série +5/+10/+20 à 3/5/10 bonnes réponses de suite **dans une séance** ; plafond 15 bonnes réponses rapportant des points par jour, une question ne rapporte qu'une fois par jour, 450 points par semaine (semaine ISO, lundi) ; mode correction = 0 point. Défi du jour (10 bonnes réponses, sans point). 10 niveaux. 6 badges de régularité. Score d'équipe = (Σ points plafonnés ÷ inscrits) × (0,5 + 0,5 × part d'actifs) ; actif = ≥ 20 bonnes réponses sur ≥ 2 jours dans la semaine. Équipes (thème A) : Renard #D55E00, Hibou #0072B2, Ours #CC79A7, Requin #56B4E9, Faucon #F0E442, Cerf #009E73, Tortue #E69F00. Son coupé par défaut ; `prefers-reduced-motion` respecté.

### `config.json` (schéma 1)
`phase`, `version_application`, `version_banque`, `bareme_version`, `mis_a_jour_le`, `valide_jusqu_au`, `bareme{…}`, `equipes[{id,nom,couleur,embleme,inscrits}]`, `codes_equipe{hash:idEquipe}` *(objet : remplace l'idée initiale d'une liste)*, `codes_seance[{date,hash}]`, `challenge_fin`. Configuration absente : phase `base`. Configuration périmée (`valide_jusqu_au` dépassé) : utilisable, signalée `config_perimee=1` dans les lignes.

### `classement.json` (schéma 1)
`genere_le`, `semaine`, `bareme_version`, `equipes[{id,score,actifs,inscrits}]`, `cumul{id:score}`, `manches[{semaine,gagnante}]`, `challenge_fin`, `gagnante`. Aucun code élève.

### Colonnes du fichier de résultats v2
`schema` (=2) · `eleve` · `equipe` · `session_id` · `session_debut` · `session_fin` · `horodatage` (ISO avec décalage horaire) · `jour` · `question_id` · `format` · `correct` (1/0) · `tentative` · `duree_s` · `mode` (`normal`, `correction`, `TEST`, `TEST-correction`) · `phase` · `version_app` · `version_banque` · `bareme_version` · `config_date` · `config_perimee` · `contexte_seance` (`en_classe`/`hors_classe`) · `participation_ludique` · `points_jeu` · `controle` (somme de contrôle à l'export : détecte une modification accidentelle, **pas** une protection contre la fraude).

### Outils
- `fusion_resultats.html` (v2) : lit les fichiers v1 et v2, recalcule les points avec le même algorithme que l'application, classe les équipes par semaine, exporte `classement.json` et `suivi_hebdo.csv`, signale les anomalies (réponses < 1 s répétées, points au-dessus du calcul, somme de contrôle invalide, doublons, fichier chargé deux fois, phase/version inattendue, horodatage futur ou hors période), enregistre les décisions (annulation de points d'un jour, absence justifiée, contexte de séance) dans `decisions.json`.
- `generateur_codes.html` : codes élèves, codes de groupe, tirage stratifié en 7 équipes, codes de séance, blocs à coller dans `config.json`.

## v1 (`msdgn-v1`)
Version actuellement déployée : quiz sans équipes ni points, résultats nominatifs, une ligne par réponse de la séance.

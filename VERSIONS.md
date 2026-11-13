# Historique des versions — Mémo MSDGN

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

# Mémo MSDGN — version 3.0 « univers » (21/11/2026)
Copie de la v2.1 (inchangée dans `jeu_v2\`) avec un nouvel habillage de la **phase ludique** uniquement.
- 3 univers au choix de l'élève : B Rush arcade (défaut), C Zone finale, E Briques. Écran de choix au 1er lancement + bouton « Univers » sur l'accueil. Choix stocké sur le téléphone (clé `x:th`), jamais dans le CSV.
- Effets Calme / Arcade / Max : réglage séparé (défaut Arcade). Son coupé par défaut. « Réduire les animations » du système => Calme imposé.
- Décors animés, mascotte (HUD et accueil), particules, +N qui vole vers le score, bannières de série ×3/×5/×10, niveau, badges, défi, confettis de fin.
- Mesure, points, journal et CSV : code inchangé. `version_app` du CSV reste « 2.1 » (accepté par l'outil de fusion v2.1) ; le pied de page affiche « v3.0 · mesure 2.1 ».
- Phase base : aucune modification visible (captures identiques à la v2.1 à 18 pixels près, écart 1/255).
- Ressources (Kenney CC0, polices SIL OFL) intégrées au fichier : index.html ~493 Ko, un seul fichier + fichiers PWA.
- Non fait : décompte 3-2-1 au départ, fiche d'explication en feuille basse (l'explication reste dans le cadre de retour).
- Reconstruction : `sources_v3_0\build_app.py` (ux.css, ux.js + v2.1).

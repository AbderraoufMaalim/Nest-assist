# Nest Assist

Deux outils d'atelier, une page chacun, servis depuis le même hébergement
statique et reliés par un bouton dans l'en-tête :

| Page | Outil |
|---|---|
| `index.html` | **Nest Assist** — répartition matière → JobBOSS |
| `fraisage.html` | **Fraisage** — calcul du cône de fraisure en tôle mince |

---

## Nest Assist — répartition matière

Répartition de la consommation de matière première par job, après imbrication
TruTops, pour saisie dans JobBOSS.

Remplace la méthode papier : un tableau job × tôle où chaque cellule est la
fraction de cette tôle qui revient à ce job. Mode **semi-automatique** —
l'opérateur voit le calcul se construire et garde la main sur l'estimation.

## Utilisation

Une page, aucune installation. `index.html` s'ouvre directement dans un
navigateur ou se sert tel quel depuis un hébergement statique.

1. **Importer un plan (PDF)** — le plan de travail TruTops. Remplit les tôles,
   leurs formats, leurs répétitions, les pièces et une répartition proposée au
   prorata des surfaces nettes.
2. **Importer des jobs** — les lignes copiées du tableau « Nest à faire » du
   dashboard JobBOSS. Apporte les numéros de job, les révisions et les quantités
   ERP, en se rattachant aux pièces du plan. L'ordre des deux imports est libre.
3. **Saisir les retailles récupérées** par tôle (12″ et 24″). C'est la réalité
   du plancher : le PDF ne peut pas la connaître.
4. **Ajuster la répartition** à la glissière ou en tirant les frontières de la
   barre segmentée.
5. **Reporter dans JobBOSS** la Valeur JobBOSS (pièces/feuille) de chaque job.

## Modèle de calcul

```
multiplicateur = (longueur_de_départ − 12×n12 − 24×n24) / longueur_standard
poids d'une tôle = multiplicateur × nombre de feuilles
consommation d'un job = Σ sur les tôles (fraction × poids)
Valeur JobBOSS = quantité / consommation, tronquée vers le bas à 2 décimales
```

Les fractions sont des **entiers de 0 à 100 dont la somme fait exactement 100**
par colonne : la contrainte est garantie par construction, pas vérifiée après
coup. Les consommations sont ensuite forcées à 2 décimales de façon à ce que
leur somme retombe exactement sur la matière engagée (« blindage anti-JobBOSS »),
avec un minimum de 0,01 pour éviter une division par zéro.

## Lecture du PDF

Extraction en local par pdf.js, à partir des coordonnées du texte. **Aucun appel
d'API, aucun token, aucune donnée qui sort du poste.** Le parseur se repère sur
des ancres de mise en page et se vérifie lui-même avec deux contrôles internes
au rapport :

- somme des répétitions = « Nombre de tôles »
- « Quantité sur tôle » × répétition = colonne « Pièce »

Si l'un des deux échoue, l'écran d'aperçu le signale au lieu de remplir le
tableau avec des chiffres douteux.

## Dépendance

pdf.js 3.11.174, chargé depuis un CDN. Tout le reste est dans le fichier.

## Données

Aucune donnée n'est transmise nulle part : tout reste dans le navigateur.
Les plans de travail et listes de jobs réels ne doivent pas être versionnés
(voir `.gitignore`) — ils contiennent des noms de clients et des références
de production.

---

## Fraisage — calcul du cône de fraisure

`fraisage.html`. Une seule relation, lue dans les quatre sens :

```
Ø extérieur = Ø intérieur + 2 · p · tan(θ/2)      p = épaisseur − bord droit
```

`θ` est l'angle **inclus total**, pas le demi-angle. `p` est la profondeur du
cône, qui n'est **pas** l'épaisseur de la tôle dès qu'il reste une paroi droite
sous le fraisage — c'est le piège habituel, d'où le champ « bord droit » séparé
et le bouton « Pleine profondeur » pour le cas h = 0.

Le point cible devant un champ (Ø ext, Ø int, angle, bord droit) désigne
l'inconnue ; les autres sont les mesures. L'épaisseur est toujours une entrée.
Changer d'inconnue ne perd rien : la valeur calculée est figée dans l'état avant
la bascule.

- Vue en coupe SVG redessinée en direct : matière hachurée, axe de symétrie,
  parois prolongées en pointillé jusqu'à l'apex virtuel, cotes fléchées. C'est
  là qu'on voit pourquoi un 82° creuse plus profond qu'un 100° à Ø égal.
- Angles normalisés 82 / 90 / 100 / 110 / 120 — et rien d'autre. Un angle
  bâtard est signalé avec le normalisé le plus proche et son écart.
- Forets proposés, le meilleur de chaque jeu : numérotés #1–#60, lettres A–Z,
  fractions au 64e, métriques 1,0–13,0 mm. Un clic prend le foret comme mesure.
- Jauges de tôle (acier, inox, aluminium), bascule pouces / mm, virgule
  décimale et fractions tapées telles quelles (« 15/64 », « 1 3/8 »).

L'état est en pouces à l'interne ; l'unité n'est qu'un habillage. Le thème
(clair / sombre) est partagé avec Nest Assist.

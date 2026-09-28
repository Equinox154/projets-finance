# Notions techniques — Finance & Python

Chaque notion : **définition** → **formule / code** → **à retenir**.

---

## 1. Données de marché

### Ticker
Code qui identifie un titre sur une place de cotation.
- `SAN.PA` = Sanofi sur Euronext **Pa**ris. Le suffixe indique la place (`.SW` = Suisse, pas de suffixe = US).
- **À retenir :** même entreprise, places différentes = tickers différents, devises différentes.

### DataFrame
Tableau pandas : des lignes (ici une par jour) et des colonnes (ici les prix).
- `san.head()` / `san.tail()` : 5 premières / dernières lignes
- `san.shape` : (nombre de lignes, nombre de colonnes)
- **À retenir :** toujours regarder ses données avant de calculer quoi que ce soit.

### Index
L'étiquette des lignes. Pour des cours de bourse, c'est la **date**.
- `san.index.min()` / `san.index.max()` : première et dernière date

### OHLCV
Les colonnes standard renvoyées par yfinance :
- **O**pen (ouverture), **H**igh (plus haut), **L**ow (plus bas), **C**lose (clôture), **V**olume (nombre de titres échangés)

### Cours de clôture ajusté
Prix de clôture corrigé des **dividendes** et des **splits** (division d'actions).
- yfinance l'applique par défaut (`auto_adjust=True`).
- **À retenir :** sans ajustement, le jour du versement d'un dividende, le prix baisse mécaniquement → on croirait à une perte qui n'en est pas une.

### Jours de cotation (252)
La bourse est fermée les week-ends et jours fériés → **~252 jours de cotation par an**, pas 365.
- 5 ans ≈ 1 260 à 1 280 lignes, pas 1 825.
- **À retenir :** 252 est le chiffre utilisé pour tout annualiser.

### NaN (valeur manquante)
*Not a Number* : case vide.
- `san.isna().sum()` : compte les valeurs manquantes par colonne
- `ffill()` (*forward fill*) : remplace un vide par la dernière valeur connue → « marché fermé, le prix n'a pas bougé »
- **À retenir :** plusieurs places = plusieurs calendriers de jours fériés = des NaN à traiter.

---

## 2. Rendement

### Rendement quotidien
Variation en pourcentage du prix d'un jour à l'autre.
- Formule : `(prix du jour − prix de la veille) / prix de la veille`
- Code : `san["Close"].pct_change()`
- **À retenir :** la première ligne vaut NaN (pas de veille). On compare des **rendements**, jamais des niveaux de prix (un titre à 100 € n'est pas « plus cher » qu'un titre à 10 €).

---

## 3. Risque

### Écart-type
Mesure de la dispersion des valeurs autour de leur moyenne. Plus il est grand, plus les rendements s'éloignent de la moyenne.
- Code : `.std()`

### Volatilité
Écart-type des rendements = amplitude typique des variations. C'est la mesure de risque la plus utilisée.
- Quotidienne : `san["rendement"].std()` → Sanofi ≈ **1,47 %**
- Annualisée : `vol_quotidienne * (252 ** 0.5)` → Sanofi ≈ **23,3 %**
- **À retenir :** on multiplie par **√252**, pas par 252 : la volatilité croît avec la racine du temps.
- Repères : grande capitalisation européenne 15–25 %, tech US 40–60 %.

### Clustering de volatilité
Les périodes agitées se regroupent : une grosse journée est souvent suivie d'autres grosses journées.
- **À retenir :** le risque arrive par vagues, pas de façon régulière.

### Asymétrie / queues épaisses
Les chutes extrêmes sont plus violentes et plus fréquentes que ne le prévoit la loi normale.
- Sanofi, 27/10/2023 : −19 % en une séance ≈ 13 écarts-types — « impossible » selon une loi normale.
- **À retenir :** la volatilité sous-estime le risque de krach.

### Plus haut historique courant
Le maximum atteint depuis le début de la période, jour après jour.
- Code : `san["Close"].cummax()`

### Drawdown
Perte depuis le dernier sommet atteint.
- Formule : `prix / plus haut courant − 1`
- Code : `san["Close"] / san["Close"].cummax() - 1`
- Vaut 0 quand le titre est à son sommet, négatif sinon.

### Drawdown maximum (max drawdown)
La pire perte depuis un sommet sur toute la période.
- Code : `san["drawdown"].min()` et sa date `san["drawdown"].idxmin()`
- Sanofi : **−27,8 % le 9 mars 2026**
- **À retenir :** c'est ce que l'investisseur *vit* réellement — la perte maximale qu'il aurait subie en achetant au pire moment.

### Volatilité vs drawdown
| | Volatilité | Drawdown |
|---|---|---|
| Voit bien | les chocs brutaux (oct. 2023) | l'érosion lente et durable (2025–2026) |
| Voit mal | la baisse lente | l'agitation sans tendance |

- **À retenir :** aucune mesure de risque ne suffit seule.

---

## 4. Lecture de marché

### Écart au consensus
Le **consensus** = la moyenne des prévisions des analystes. Le marché réagit à l'écart entre le résultat publié et ce qui était attendu, pas au résultat lui-même.
- Sanofi, 27/10/2023 : prévisions 2024 en baisse alors que le consensus attendait une hausse → −19 %.
- **À retenir :** un bon résultat peut faire chuter le cours s'il était moins bon qu'attendu.

---

## 5. Réflexes Jupyter

- `Shift + Entrée` : exécuter la cellule
- `fonction?` : afficher la documentation (ex. `yf.download?`)
- Touche `M` sur une cellule : la passer en Markdown (texte)
- `Kernel → Restart Kernel and Run All Cells` : tout relancer dans l'ordre si les résultats deviennent incohérents
- `NameError` : variable inconnue → kernel redémarré, cellule non exécutée, ou faute de frappe
- Lire une erreur **de bas en haut** : la dernière ligne donne le diagnostic, la flèche `---->` la ligne fautive
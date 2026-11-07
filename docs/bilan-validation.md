# Validation indépendante des socles P4 sur Nexus 5 — 6 novembre 2026

## Données et correction opérateur

- Modèle figé issu de `mp30-training12-1793977203066.json`, couvrant P401–P405, P407–P411, P414–P415 (12 codes). P406, P412 et P413 ne sont pas disponibles.
- 48 essais de validation enregistrés sur le Nexus, quatre poses consécutives par code. Le fichier brut est conservé tel quel dans `raw-mp30-validation-1793982130255.json`.
- P405 pose 1, essai 17 : l'opérateur confirme avoir posé l'ancien socle. Exclu **uniquement de l'analyse**, sans modifier la capture originale. Le modèle prédit P404 pour cet essai (RMS 0,352 mm, marge 0,842), ce qui est cohérent avec cette déclaration, sans prouver à lui seul quel socle a été posé.

## Résultats observés

| Mesure | Avant apprentissage | Modèle appris |
|---|---:|---:|
| Essais fiables après exclusion de P405-1 | 47 | 47 |
| Captures à quatre contacts retenues | 33 | 33 |
| Identités correctes parmi les captures | 28/33 | **33/33** |
| Fausses identités acceptées | 1 | **0** |
| Abstentions du classifieur | 4 | **0** |
| Échecs de capture (10 nombres de contacts, 4 retraits précoces) | 14 | **14** |

Le taux de bout en bout observé est **33/47 = 70,2 %**. Le 33/33 conditionnel montre que les géométries sont séparables *quand quatre contacts sont capturés*. Il ne démontre pas 100 % en usage réel. P401 : 0/4 captures ; P411 : 1/4. Ces deux socles demandent une attention prioritaire.

## Récupération temporelle simulée

Règle définie sans regarder l'étiquette attendue : examiner les événements Android ayant exactement quatre points (mouvement ou quatrième point ajouté), classer chaque trame avec le modèle appris, accepter une identité seulement avec au moins trois votes, au moins 80 % des votes pour elle et un intervalle d'au moins 150 ms entre le premier et le dernier vote concordant.

- Sur les 47 essais fiables : **38 corrects, 0 faux, 9 sans décision** (80,9 % de réussite de bout en bout).
- Cinq essais que l'application avait rejetés sont récupérés : P401 essai 1, P402 essai 6, P403 essai 10, P408 essai 26, P411 essai 40.
- Ce résultat est **rétrospectif**, calculé sur les mêmes 47 essais. Il faut intégrer l'algorithme, figer ses paramètres, puis le retester sur de nouvelles poses avant de parler de gain confirmé.
- Plusieurs échecs `wrong_contact_count` ont brièvement vu quatre points, mais le dernier événement était à trois ou cinq. L'application ne conservait que la dernière configuration complète au terme des deux secondes : c'est une cause logicielle plausible des rejets récupérables. D'autres essais n'ont jamais eu de configuration exploitable ; le logiciel seul ne les récupère pas.

## Localisation

Sur les 33 captures fiables : erreur du centre par rapport à la croix affichée, médiane **4,34 mm**, P95 **6,58 mm** (conversion selon le xdpi/ydpi Android). Un décalage médian de **−0,85 mm en X, −4,23 mm en Y** apparaît. Une correction de translation estimée par validation croisée leave-one-out ramène les écarts résiduels à une médiane de **1,88 mm** et un P95 de **3,93 mm**. Ces nombres mêlent erreur du capteur et précision de pose manuelle ; ils ne constituent pas une mesure de précision absolue ni une calibration transférable à une tablette.

## Suite technique

1. Corriger la capture : conserver des trames à quatre contacts stables pendant la fenêtre de mesure, puis voter dans le temps ; ne pas exiger que **la dernière** trame soit à quatre points. Garder une abstention en cas d'ambiguïté.
2. Refaire un court test indépendant ciblant P401 et P411, avec ces règles figées et des poses nouvelles. Mesurer séparément disponibilité de quatre points, exactitude d'identité, position et orientation.
3. Tester ensuite sur une tablette non rootée. Le Nexus est un banc Android standard pour cette expérience, mais la réponse tactile peut changer d'un appareil à l'autre.

Calculs reproductibles : `analyze_validation.py`, `validation-analysis.json`, `temporal_recovery.py`, `temporal-recovery-analysis.json`. Les données d'origine restent conservées.

## Mise en œuvre préparée

L'application 0.7 a été construite, signée, installée et ouverte sur le Nexus. Le modèle et les anciennes séries y sont toujours présents. Le parcours court `mp30-validation-1793985829647.json` attend P401, pose 1/4 ; huit essais sont prévus, groupés P401 puis P411. Aucune nouvelle mesure n'était encore enregistrée lors de cette vérification. Cette version applique le vote temporel ci-dessus aux nouvelles validations, ce qui permet de vérifier réellement le gain annoncé par la simulation.

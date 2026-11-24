# MeetRecord — Cahier des charges fonctionnel et technique

> **Document de référence destiné à Claude (interface web) pour concevoir et développer MeetRecord.**  
> Domaine cible : `https://meetrecord.cltobadminton.fr`  
> Organisation utilisatrice : CLTO Badminton  
> Langue de l’application : français  
> Date de rédaction : 24 novembre 2026  
> Statut : spécification initiale — les décisions explicitement marquées « à confirmer » doivent être validées avant l’implémentation concernée.

---

## 0. Instructions impératives pour Claude

Tu interviens comme **architecte logiciel, développeur full-stack senior et responsable qualité**. Tu dois développer une application réellement exploitable, sécurisée, maintenable et documentée, pas une simple démonstration.

### 0.1 Méthode de travail

1. Commence par analyser ce cahier des charges et produire un plan d’implémentation découpé en lots courts.
2. Ne génère jamais toute l’application en une seule réponse ou un seul gros changement.
3. Avant chaque lot :
   - rappelle son objectif ;
   - liste les fichiers qui seront créés ou modifiés ;
   - identifie les risques et dépendances ;
   - annonce les commandes de vérification prévues.
4. Implémente un lot à la fois.
5. Après chaque lot :
   - exécute les tests et contrôles disponibles ;
   - corrige les erreurs ;
   - mets à jour la documentation ;
   - indique ce qui fonctionne, ce qui reste à faire et les éventuelles limites.
6. Ne prétends jamais qu’une commande, un test, un build ou un déploiement a réussi s’il n’a pas réellement été exécuté.
7. Ne remplace pas une fonctionnalité réelle par un faux bouton, une donnée codée en dur ou une simulation non signalée.
8. Si une information manque, propose une hypothèse raisonnable, indique-la clairement et demande confirmation si elle modifie une décision structurante. Ne bloque pas pour des détails mineurs.
9. N’ajoute pas de dépendance sans justification. Vérifie la maintenance, la licence, la compatibilité et la version stable disponible.
10. Privilégie les API officielles Google et les bibliothèques maintenues. N’utilise pas de scraping de Google Docs.
11. Ne stocke jamais de secret dans Git, dans le frontend ou dans les logs.
12. Ne désactive jamais les contrôles TLS, les protections de sécurité ou les tests pour faire passer une fonctionnalité.

### 0.2 Contraintes liées à l’environnement Claude gratuit

Le propriétaire du projet utilise l’interface web gratuite de Claude, et non Claude Code CLI avec un abonnement payant.

- Produis des changements petits et autonomes, faciles à appliquer et à vérifier.
- Donne des instructions de commandes précises pour Windows/PowerShell et, lorsque pertinent, Debian/Linux.
- Ne suppose pas que tu peux accéder au terminal, au VPS, au dépôt Git ou aux secrets du propriétaire.
- Demande les sorties de commandes nécessaires au lieu d’inventer leur résultat.
- Lorsqu’un fichier est trop volumineux, travaille par étapes et fournis des patchs ciblés.
- À la fin de chaque lot, donne un prompt de reprise concis pour la session suivante.
- Ne demande jamais à l’utilisateur de coller une clé API, un mot de passe ou un jeton dans la conversation.

### 0.3 Priorités

Ordre de priorité absolu :

1. Sécurité et confidentialité.
2. Fiabilité de la capture audio et absence de perte de données.
3. Respect strict du document Google Docs existant.
4. Classement correct des échanges dans les sections du compte rendu.
5. Qualité rédactionnelle et traçabilité des propositions de l’IA.
6. Simplicité d’utilisation pendant une réunion.
7. Optimisation des coûts et des ressources du VPS.

---

# 1. Présentation du projet

## 1.1 Nom et identité

- **Nom :** MeetRecord
- **URL de production :** `https://meetrecord.cltobadminton.fr`
- **Organisation initiale :** CLTO Badminton
- **Type :** application web privée de transcription et d’assistance à la rédaction de comptes rendus de réunions physiques.
- **Public :** membres autorisés de l’équipe / du Conseil d’administration.
- **Langue principale :** français.

L’application ne doit pas être référencée par les moteurs de recherche. Elle doit être utilisable uniquement via son URL et après authentification. L’URL non référencée n’est pas une mesure de sécurité : toutes les routes et données doivent être protégées côté serveur.

## 1.2 Problème à résoudre

Les réunions ont lieu en présentiel, généralement avec environ 10 personnes dans une salle d’environ 10 m². Elles durent souvent plus de deux heures et se tiennent environ une fois par mois.

Une secrétaire rédige actuellement le compte rendu dans un document Google Docs partagé. Le document comporte une structure précise, des rubriques et des sous-rubriques. Les participants annoncent souvent oralement le changement de sujet ou de commission, par exemple :

> « On passe à la Commission Informatique. »

L’objectif est que MeetRecord :

1. capture le son depuis un microphone USB branché à l’ordinateur Windows de la réunion ;
2. transcrive les échanges en français ;
3. détecte le sujet ou la commission en cours, notamment grâce aux annonces orales ;
4. transforme les échanges en notes synthétiques, fidèles et exploitables ;
5. insère ces notes **directement dans la bonne section du document Google Docs partagé** ;
6. permette à un utilisateur de vérifier, corriger, compléter, accepter ou supprimer les propositions de l’IA ;
7. ne déplace, n’efface et ne réécrive jamais le contenu existant de la secrétaire sans action explicite.

Il ne s’agit pas uniquement de générer une transcription ou un résumé global après la réunion. Le classement contextuel par section et l’insertion dans le document existant sont les fonctionnalités centrales.

## 1.3 Utilisateurs et rôles

Plusieurs comptes seront utilisés.

- Le propriétaire du projet est le **Super administrateur** et utilisateur principal.
- D’autres membres de l’équipe pourront avoir un compte individuel.
- Les comptes sont créés ou invités par un administrateur. **Pas d’inscription publique.**
- Le système doit permettre de désactiver un compte sans supprimer l’historique des réunions auquel il est associé.

Rôles minimum :

| Rôle                 | Droits                                                                                                              |
| -------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Super administrateur | Tous les droits, gestion des comptes, paramètres, intégration Google, historique, suppression et configuration      |
| Administrateur       | Gestion des réunions et des utilisateurs selon les permissions accordées ; pas de transfert de propriété du système |
| Utilisateur          | Démarrer/consulter les réunions autorisées, contrôler les notes et utiliser les documents autorisés                 |

Prévoir un modèle de permissions extensible, sans multiplier inutilement les rôles dans le MVP.

---

# 2. Analyse du document de référence fourni

## 2.1 Document analysé

Le document joint a été lu : `2026-10-29_CR_CA_CLTO_Badminton.docx`, compte rendu du Conseil d’administration du 29 octobre 2026, 18 pages.

Il sert de **référence de structure et de style rédactionnel**. Le document de production réel sera un Google Docs dans un Drive partagé Google Workspace.

Ne pas traiter le DOCX d’exemple comme un document à modifier automatiquement. Il sert à comprendre le modèle rédactionnel et les rubriques.

## 2.2 Structure observée

Le compte rendu comporte :

- un en-tête institutionnel « Cercle Laïque des Tourelles d’Orléans BADMINTON » ;
- le nom du président ;
- le titre « Compte rendu du Conseil d’Administration » ;
- la date, l’heure et le lieu ;
- les présents et absents excusés ;
- une numérotation hiérarchique des grands points ;
- des titres de sujets et des paragraphes explicatifs ;
- des listes à puces ;
- des tableaux, notamment pour les répartitions de dépenses et les responsables ;
- des décisions clairement identifiées ;
- des résultats de votes avec nombre de voix ;
- des responsables et échéances lorsqu’ils sont connus ;
- une formule de rédaction et des espaces de signature du président et de la secrétaire.

Le style est professionnel, factuel et rédigé en français. Les échanges oraux sont reformulés, dédoublonnés et regroupés par thème. Le compte rendu n’est pas une transcription mot à mot.

## 2.3 Rubriques réellement observées dans le document

Les grandes rubriques de l’exemple sont les suivantes (les intitulés doivent rester configurables, car l’ordre du jour peut évoluer d’une réunion à l’autre) :

1. Validation du compte-rendu précédent
2. Revue des décisions
3. Actualités de la Présidence (Steve)
4. Vie Administrative (Steve)
5. Vie sportive (Mathieu)
6. Trésorerie (Philippe)
7. Secrétariat et organisation interne (Élodie)
8. Commission Partenariats & Entreprises (Benoit & Benjamin)
9. Commission Compétitions - Gestion ICN (Martin & Bastien) — le document présente aussi cette rubrique plus loin dans sa structure ; vérifier le plan de réunion réellement utilisé au moment de l’import.
10. Commission Jeunes (Mathilde)
11. Commission événements (Mathilde & Benjamin)
12. Commission Communication (Élodie)
13. Commission Formations & Officiels techniques (Laurent)
14. Commission Eco-responsabilité & Labellisation (Élodie)
15. Commission Informatique (Steve et Maxime)
16. Points divers
17. Prochain Conseil d’administration

**Important :** le document comporte des rubriques qui ne sont pas toutes des « commissions ». Il faut donc parler de **section active / rubrique active**, et non coder uniquement une liste fixe de commissions.

Le document présente également des sous-rubriques thématiques, par exemple :

- « Licences et inscriptions 2026-2027 » ;
- « Relations avec la Ville d’Orléans » ;
- « Situation financière et accompagnement du club » ;
- « Organisation des créneaux pendant les vacances scolaires » ;
- « Préparation de l’Assemblée générale » ;
- « Partenariat avec Solibad » ;
- « Renouvellement du matériel informatique ».

Ces sous-rubriques doivent pouvoir être proposées par l’IA ou définies par le modèle du document.

## 2.4 Style rédactionnel attendu

L’IA doit produire un compte rendu comparable à l’exemple :

- français correct, professionnel et neutre ;
- paragraphes courts et listes lorsque cela améliore la lisibilité ;
- suppression des répétitions et digressions sans intérêt ;
- conservation des noms propres, dates, montants, pourcentages, volumes et résultats de vote ;
- distinction stricte entre information, hypothèse, proposition, action et décision ;
- aucune invention pour compléter une information manquante ;
- pas de conclusion ou de décision implicite qui n’a pas été formulée pendant la réunion.

Exemples de formulations à privilégier :

- « Le CA constate que… »
- « Plusieurs pistes sont envisagées… »
- « Il est proposé de… »
- « La décision est validée/rejetée… » uniquement si le résultat est clairement établi ;
- « Un retour est attendu… »
- « X est chargé de… » uniquement si le responsable a été explicitement désigné.

### 2.5 Décisions et votes : règle impérative

Le document d’exemple distingue parfois :

- la proposition soumise ;
- le résultat du vote ;
- la décision du CA ;
- les modalités concrètes qui en découlent.

MeetRecord doit préserver cette distinction.

Ne jamais transformer :

- une idée en décision ;
- une discussion en validation ;
- un accord de principe en engagement définitif ;
- une échéance envisagée en échéance confirmée ;
- une personne évoquée en responsable désigné.

Si le résultat d’un vote est incomplet ou ambigu, créer un élément « À vérifier » et ne pas conclure à l’adoption ou au rejet.

### 2.6 Particularités à prendre en charge

- Certaines rubriques peuvent être vides ou indiquer « Aucun point particulier ».
- Un sujet peut être abordé dans une autre rubrique que celle où il sera finalement rédigé.
- Une discussion peut revenir sur une commission précédente.
- Une même personne peut intervenir sur plusieurs rubriques.
- Une commission peut avoir plusieurs responsables.
- Les noms et intitulés changent d’une réunion à l’autre.
- Les informations chiffrées doivent être conservées avec leurs unités.
- Les actions doivent être repérées même si elles sont formulées oralement sans le mot « action ».

---

# 3. Objectifs et périmètre

## 3.1 Objectifs du MVP

Le MVP doit permettre de réaliser un cycle complet :

1. Connexion sécurisée.
2. Création d’une réunion.
3. Connexion à Google Workspace / Drive partagé.
4. Sélection d’un Google Docs existant ou création d’une copie d’un modèle.
5. Lecture de la structure du document et détection des rubriques.
6. Sélection du microphone USB.
7. Démarrage, pause et reprise de l’enregistrement.
8. Transcription par blocs avec conservation des horodatages.
9. Détection de la section active à partir des annonces et du contexte.
10. Génération de notes synthétiques par section.
11. Prévisualisation et correction des notes.
12. Insertion contrôlée dans la section correspondante du Google Docs.
13. Vérification que l’insertion a réussi.
14. Arrêt et clôture de la réunion.
15. Accès à l’historique et aux journaux de traitement.

## 3.2 Hors périmètre du MVP

Ne pas développer dans le premier lot :

- visioconférence ;
- identification biométrique des personnes ;
- reconnaissance vocale nominative obligatoire ;
- application mobile native ;
- partage public de compte rendu ;
- génération automatique de procès-verbal juridiquement certifié ;
- signature électronique ;
- synchronisation avec d’autres suites bureautiques que Google Workspace ;
- gestion avancée des votes avec authentification des votants ;
- entraînement d’un modèle IA personnalisé ;
- intégration à Asana ou WhatsApp ;
- envoi automatique de mails aux participants.

Ces éléments pourront être étudiés après validation du MVP.

---

# 4. Architecture fonctionnelle cible

## 4.1 Vue d’ensemble

Architecture recommandée :

```text
MICRO USB
   |
   v
Navigateur Windows (React)
- getUserMedia / MediaRecorder
- sélection du périphérique audio
- capture et découpage audio
- interface de contrôle
   |
   | HTTPS / WebSocket sécurisé
   v
API MeetRecord (Node.js / TypeScript)
- authentification et autorisations
- gestion des réunions
- réception des segments audio
- orchestration des traitements
- gestion des sections et notes
- intégration Google
   |
   +---------------------> Google Gemini API
   |                        - transcription / compréhension audio
   |                        - classification de section
   |                        - synthèse structurée
   |
   +---------------------> Google Drive API
   |                        - recherche / sélection des documents
   |                        - accès au Drive partagé
   |
   +---------------------> Google Docs API
   |                        - lecture de la structure
   |                        - insertion dans les sections
   |                        - vérification des modifications
   |
   +---------------------> MariaDB
   |                        - comptes et rôles
   |                        - réunions et états
   |                        - sections et événements
   |                        - notes, validations et audits
   |
   +---------------------> Stockage audio temporaire
                            - segments chiffrés si possible
                            - politique de rétention stricte
```

## 4.2 Décision d’architecture audio et IA

Le microphone est connecté à l’ordinateur Windows de la salle. Le navigateur doit capturer directement ce microphone.

**Ne pas dépendre de Meetily comme composant obligatoire du système.** MeetRecord doit maîtriser son propre flux audio et son pipeline de traitement. Meetily peut rester un outil de comparaison ou de test, mais ne doit pas être une dépendance de production.

Pour le MVP, prévoir une architecture par adaptateurs :

- `AudioCaptureAdapter` côté navigateur ;
- `AudioTranscriptionProvider` côté backend ;
- `MeetingAnalysisProvider` côté backend ;
- `GoogleDocsProvider` côté backend.

Le premier fournisseur d’analyse sera Gemini API.

### Point de vigilance sur Gemini

Un abonnement **Google AI Pro / Gemini Pro dans l’application Gemini ne signifie pas que l’utilisation de Gemini API est incluse gratuitement**. Gemini API est un service distinct, avec ses propres quotas, modèles, conditions et éventuelle facturation.

Avant d’activer la production :

- créer/configurer un projet Google Cloud / AI Studio adapté ;
- activer la facturation si le modèle ou le volume l’exige ;
- vérifier le tarif actuel du modèle choisi ;
- vérifier les quotas et limites de taille/durée audio ;
- vérifier les conditions de traitement et de conservation des données ;
- afficher une estimation ou un indicateur de consommation lorsque possible.

Ne jamais annoncer que l’application est gratuite sans vérifier le coût réel des appels API.

### 4.3 Stratégie de traitement audio recommandée

Les réunions durent plus de deux heures. Il ne faut donc jamais envoyer un seul fichier audio géant en fin de réunion.

Le navigateur doit :

- capturer l’audio en continu ;
- utiliser `MediaRecorder` avec un timeslice configurable ;
- produire des segments de taille/durée raisonnable (valeur initiale à tester : 30 à 60 secondes) ;
- transmettre les segments dans l’ordre avec un identifiant de réunion et un numéro de séquence ;
- conserver temporairement les segments non acquittés en cas de coupure réseau ;
- réessayer les transferts de manière idempotente ;
- afficher le retard de traitement et l’état de synchronisation.

Le backend doit :

- valider le type MIME, la taille, la durée et l’ordre ;
- empêcher les doublons ;
- stocker temporairement le segment ;
- l’envoyer au fournisseur de transcription/analyse ;
- enregistrer le résultat et ses horodatages ;
- marquer le segment comme traité ou en erreur ;
- permettre une reprise après redémarrage.

Ne pas supposer que le navigateur produit toujours du `audio/webm;codecs=opus` : détecter les formats pris en charge via `MediaRecorder.isTypeSupported()` et tester Chrome/Edge sous Windows.

### 4.4 Transcription : choix technique à valider par un POC

Le premier POC doit comparer au minimum :

1. transcription audio via Gemini API ;
2. transcription Whisper / faster-whisper, si le coût et les performances sur la machine de capture ou le VPS sont acceptables.

La solution retenue doit être évaluée sur :

- français oral ;
- noms propres et noms de commissions ;
- chiffres, dates et montants ;
- conversations à 10 personnes dans une petite salle ;
- bruit ambiant et chevauchement de voix ;
- durée de deux heures ou plus ;
- coût par réunion ;
- latence de traitement.

Ne pas faire tourner par défaut un modèle lourd sur le VPS sans benchmark. Le VPS de production est un VPS Debian 13 aux ressources limitées ; la charge CPU/RAM doit être mesurée.

**Architecture cible :** le fournisseur de transcription est interchangeable. Le MVP peut commencer par Gemini API pour limiter la complexité d’installation, puis ajouter un adaptateur Whisper local si les coûts ou la confidentialité le justifient.

Si Gemini traite l’audio directement, conserver une transcription horodatée exploitable. Si le modèle ne fournit pas des timestamps suffisamment précis, ajouter une stratégie de segmentation et des timestamps côté application. Ne pas inventer de timecodes.

---

# 5. Gestion des sections et classement intelligent

## 5.1 Principe central : section active

MeetRecord doit maintenir un état de réunion appelé `activeSectionId`.

Exemple :

1. L’animateur dit : « On passe à la Commission Informatique. »
2. Le système détecte cette annonce.
3. Il associe l’annonce à une section existante du document.
4. Il met à jour la section active.
5. Les échanges suivants sont classés dans cette section, sauf détection d’un changement ou correction manuelle.

La section active doit être visible dans l’interface en permanence.

## 5.2 Détection d’un changement de section

Utiliser une combinaison de :

- détection d’expressions d’annonce (« on passe à », « point suivant », « nous abordons », « commission », « rubrique », « revenons sur », etc.) ;
- comparaison avec les titres des sections du Google Docs ;
- contexte des échanges ;
- confirmation manuelle possible.

Ne pas se limiter à une liste de phrases codées en dur.

Le modèle IA doit recevoir :

- les sections disponibles et leurs identifiants ;
- la section active actuelle ;
- les derniers segments de transcription ;
- les éventuels alias configurés ;
- la consigne de ne changer de section que si les indices sont suffisamment fiables.

## 5.3 Confiance et ambiguïté

Chaque proposition de changement de section doit avoir un niveau de confiance ou une catégorie :

- `high` : changement explicite et correspondance nette ;
- `medium` : correspondance plausible, mais ambiguë ;
- `low` : incertain.

Comportement :

- confiance haute : changement automatique possible, avec événement visible dans l’interface ;
- confiance moyenne : demander confirmation ou afficher une suggestion ;
- confiance faible : conserver la section active et signaler une incertitude.

Le seuil doit être configurable. Ne pas laisser une IA basculer silencieusement dans une mauvaise rubrique.

## 5.4 Retour sur un sujet précédent

Une réunion peut revenir à une rubrique déjà abordée. Il faut donc :

- autoriser le retour vers une section précédente ;
- conserver les notes précédentes ;
- ajouter les nouvelles notes à la même section ou à un sous-titre cohérent ;
- ne jamais écraser les notes déjà insérées ;
- afficher un événement « retour à la section… ».

Si le sujet est seulement lié à une autre commission sans que la réunion change réellement de section, l’IA peut proposer une référence croisée, mais ne doit pas déplacer automatiquement le contenu.

## 5.5 Sections non reconnues

Si l’annonce ne correspond à aucun titre :

- proposer une section existante proche ;
- proposer une nouvelle sous-rubrique ou « Points divers » ;
- demander une validation utilisateur ;
- ne jamais créer automatiquement une grande section sans confirmation.

## 5.6 Configuration des alias

Prévoir une interface d’administration pour associer des alias à une section.

Exemple :

```json
{
  "sectionTitle": "Commission Informatique",
  "aliases": [
    "informatique",
    "commission informatique",
    "point informatique",
    "site internet et outils numériques"
  ]
}
```

Les alias sont configurables par réunion ou par modèle de réunion.

---

# 6. Génération des notes

## 6.1 Pas de transcription brute dans le compte rendu

La transcription complète peut être conservée temporairement ou selon les paramètres, mais le Google Docs final doit recevoir des notes synthétiques et rédigées.

L’IA doit :

- regrouper les phrases qui parlent du même sujet ;
- supprimer les hésitations et répétitions ;
- conserver les informations utiles ;
- structurer les sujets avec des sous-titres lorsque nécessaire ;
- produire des paragraphes et listes lisibles ;
- conserver les faits et les nuances ;
- signaler les informations incertaines.

## 6.2 Schéma de sortie structuré

Demander à Gemini une réponse structurée JSON validée côté backend, par exemple :

```json
{
  "sectionId": "commission-informatique",
  "sectionTitle": "Commission Informatique",
  "items": [
    {
      "type": "topic",
      "title": "Renouvellement du matériel informatique",
      "summary": "Le club poursuit sa réflexion sur le renouvellement de plusieurs équipements, notamment l’acquisition de quatre tablettes.",
      "sourceTimeStartMs": 125000,
      "sourceTimeEndMs": 181000,
      "confidence": 0.94,
      "status": "draft"
    },
    {
      "type": "action",
      "title": "Étudier les solutions d’acquisition",
      "summary": "Étudier les possibilités de matériel reconditionné, de dons ou d’autres solutions économiques.",
      "assignee": null,
      "dueDate": null,
      "sourceTimeStartMs": 182000,
      "sourceTimeEndMs": 215000,
      "confidence": 0.87,
      "status": "draft"
    }
  ],
  "uncertainties": [],
  "decisionCandidates": []
}
```

Le schéma définitif devra être validé avec une bibliothèque de validation (Zod ou équivalent). Ne jamais faire confiance directement au JSON renvoyé par un modèle.

## 6.3 Types d’éléments

Prévoir au minimum :

- `information` : information factuelle ;
- `topic` : sous-sujet avec titre et synthèse ;
- `proposal` : proposition non validée ;
- `decision` : décision explicitement actée ;
- `action` : tâche à réaliser ;
- `vote` : résultat de vote ;
- `question` : question en suspens ;
- `uncertainty` : élément nécessitant une vérification.

Pour une action, champs possibles :

- intitulé ;
- responsable (nullable) ;
- échéance (nullable) ;
- statut ;
- source audio / horodatage.

## 6.4 Règles de fidélité

Instructions système du modèle :

- Ne jamais inventer un nom, une date, un montant, un responsable ou une décision.
- Ne jamais déduire un vote d’un simple accord verbal.
- Ne pas attribuer une action à la personne qui parle si elle n’a pas été désignée.
- Conserver les réserves et conditions (« sous réserve de », « à confirmer », « à étudier »).
- Ne pas transformer un futur conditionnel en engagement ferme.
- Préserver les unités et devises.
- Si deux informations se contredisent, signaler le conflit au lieu d’en choisir une arbitrairement.
- Si la transcription est inaudible ou ambiguë, demander une vérification.
- Ne pas écrire de données sensibles inutiles dans le compte rendu.

## 6.5 Fréquence de synthèse

Valeur initiale recommandée : génération d’un lot de notes toutes les 30 à 60 secondes de transcription utile, ou après un seuil de texte configurable.

Éviter :

- un appel Gemini par phrase ;
- un résumé complet de toute la réunion à chaque segment ;
- la réécriture systématique des notes déjà validées.

Utiliser un mécanisme incrémental :

1. transcrire les nouveaux segments ;
2. déterminer leur section ;
3. analyser uniquement le nouveau contexte pertinent avec une fenêtre de contexte limitée ;
4. produire des propositions de notes ;
5. dédoublonner avec les propositions récentes ;
6. soumettre les notes à la règle d’insertion choisie.

Une synthèse finale de contrôle peut être proposée à la fin, sans remplacer les notes existantes.

---

# 7. Intégration Google Workspace : Drive partagé et Google Docs

## 7.1 Contexte

Les comptes rendus sont stockés dans un **véritable Drive partagé Google Workspace**, et non uniquement dans un dossier personnel partagé.

Le document de référence est un DOCX exporté pour cette analyse. En production, l’application doit travailler avec des documents Google Docs natifs.

## 7.2 APIs à utiliser

Utiliser les API officielles :

- Google Drive API v3 ;
- Google Docs API ;
- OAuth 2.0 pour l’autorisation ;
- éventuellement Google Picker pour faciliter la sélection du document si pertinent.

Documentation officielle à consulter pendant le développement :

- Drive API : `https://developers.google.com/drive/api/guides/about-sdk`
- Shared Drives : `https://developers.google.com/drive/api/guides/enable-shareddrives`
- Google Docs API : `https://developers.google.com/docs/api`
- OAuth web server : `https://developers.google.com/identity/protocols/oauth2/web-server`

## 7.3 Authentification Google

Distinguer deux authentifications :

1. **Authentification MeetRecord** : identifiant/mot de passe propre à l’application.
2. **Autorisation Google Workspace** : connexion OAuth permettant à MeetRecord d’accéder au Drive partagé et de modifier les documents autorisés.

Ne pas confondre les deux.

Approche recommandée pour le MVP :

- le Super administrateur connecte le compte Google Workspace qui possède les droits nécessaires sur le Drive partagé ;
- le backend conserve les jetons OAuth côté serveur, chiffrés au repos ;
- les utilisateurs MeetRecord autorisés utilisent cette connexion Google centrale ;
- les actions Google sont journalisées avec l’utilisateur MeetRecord à l’origine de l’action.

Scopes OAuth : demander le minimum nécessaire. Évaluer si `drive.file` suffit au parcours retenu ; sinon justifier précisément le scope plus large. Ne pas demander un accès complet au Drive par défaut.

Si un compte de service est envisagé, vérifier les contraintes Google Workspace et l’accès au Shared Drive. Ne pas supposer qu’un compte de service peut accéder au Drive partagé sans ajout explicite comme membre ou configuration administrateur.

## 7.4 Shared Drives

Les requêtes Drive doivent prendre en charge les Shared Drives :

- `supportsAllDrives=true` lorsque requis ;
- `includeItemsFromAllDrives=true` lorsque requis ;
- utiliser les identifiants de fichiers et de Drives, pas seulement leurs noms ;
- gérer les permissions héritées et les erreurs d’accès ;
- ne pas exposer à un utilisateur un document auquel il ne devrait pas accéder.

Ne pas parcourir ou indexer tout le Drive sans nécessité. Limiter la recherche au Shared Drive et aux dossiers explicitement configurés.

## 7.5 Choix du document de réunion

L’utilisateur doit pouvoir :

- créer une réunion dans MeetRecord ;
- sélectionner un document Google Docs existant dans le Drive partagé ;
- ou créer une copie d’un modèle de compte rendu ;
- confirmer le document cible avant de démarrer l’enregistrement ;
- ouvrir le document Google Docs dans un nouvel onglet.

**Règle de sécurité :** ne jamais modifier le document modèle original. Lorsqu’un modèle est utilisé, créer une copie et travailler sur cette copie.

Si l’utilisateur choisit un compte rendu déjà créé et commencé par la secrétaire, MeetRecord travaille sur ce document existant, sans effacer les notes manuelles.

## 7.6 Analyse de la structure Google Docs

Au démarrage :

1. récupérer le document via Google Docs API ;
2. analyser les paragraphes et les styles ;
3. repérer les titres et sous-titres ;
4. construire une liste de sections ;
5. présenter cette liste à l’utilisateur ;
6. permettre de corriger les correspondances ou les alias avant l’enregistrement.

Privilégier les styles Google Docs (`HEADING_1`, `HEADING_2`, etc.) plutôt que la seule comparaison de chaînes de caractères.

Si le document utilise des titres visuellement mis en forme sans styles de titre, proposer un mode de configuration manuelle des sections.

Ne pas déduire la hiérarchie uniquement de numéros écrits dans le texte. Les titres peuvent changer d’ordre ou être renumérotés.

## 7.7 Insertion dans le document

C’est une fonctionnalité critique.

Le backend doit utiliser `documents.get` puis `documents.batchUpdate` de Google Docs API.

Approche :

- identifier l’intervalle d’index de chaque section à partir de la structure actuelle ;
- insérer les nouveaux paragraphes à l’intérieur de la bonne section, avant le titre de niveau égal ou supérieur suivant ;
- recalculer les index à partir d’une lecture fraîche du document avant chaque groupe d’insertion ;
- sérialiser les écritures d’une même réunion pour éviter les collisions ;
- préserver les paragraphes et tableaux existants ;
- utiliser des opérations `insertText` et `updateTextStyle` / `updateParagraphStyle` si nécessaire ;
- ne jamais remplacer l’intégralité du contenu du document pour ajouter une note.

Les index Google Docs changent après insertion. Ne jamais réutiliser aveuglément des index calculés avant une modification précédente.

### Protection contre les modifications concurrentes

La secrétaire peut continuer à modifier le document pendant la réunion.

Le système doit :

- relire la structure juste avant l’insertion ;
- détecter les modifications externes lorsque l’API ou les métadonnées le permettent ;
- recalculer les positions ;
- éviter les insertions sur des index périmés ;
- journaliser les erreurs de conflit ;
- proposer une reprise manuelle si la structure a changé de manière inattendue.

Ne jamais supprimer ou écraser une modification manuelle pour résoudre un conflit.

### Idempotence

Chaque proposition de note possède un identifiant stable. Une note déjà insérée ne doit pas être ajoutée une seconde fois lors d’une reprise réseau ou d’un retry.

Stocker :

- l’ID de la note ;
- l’ID du document ;
- la section cible ;
- le statut d’insertion ;
- l’horodatage ;
- l’identifiant de requête Google si disponible ;
- le résultat de l’opération.

Ne pas se baser uniquement sur le texte pour détecter les doublons.

## 7.8 Présentation des ajouts dans Google Docs

Les notes insérées doivent respecter la hiérarchie et le style du document :

- sous-titres de sujet en style cohérent ;
- paragraphes simples ;
- listes à puces pour les éléments multiples ;
- libellés explicites pour les décisions, votes et actions lorsque pertinent.

Les ajouts automatiques doivent pouvoir être identifiés visuellement pendant la phase de relecture, sans dégrader le document final.

Option recommandée :

- préfixe ou style discret « Note proposée par MeetRecord » pendant la réunion ;
- action de validation permettant de retirer ce marqueur ou d’appliquer le style définitif.

Ne pas ajouter un énorme bloc de transcription brute.

---

# 8. Modes d’insertion et validation humaine

## 8.1 Mode par défaut

Le mode par défaut du MVP est **validation avant insertion** :

- l’IA prépare une note ;
- l’interface affiche la note et sa section cible ;
- l’utilisateur peut modifier, accepter ou rejeter ;
- après acceptation, l’application insère la note dans Google Docs.

Ce choix réduit le risque de polluer le compte rendu avec une erreur de transcription ou de classement.

## 8.2 Mode insertion automatique

Prévoir une option configurable « insertion automatique pendant la réunion », désactivée par défaut.

Dans ce mode :

- seules les notes avec confiance suffisante sont insérées ;
- les changements de section ambigus restent en attente ;
- les notes restent modifiables dans MeetRecord ;
- les insertions sont tracées ;
- l’utilisateur peut annuler une insertion récente si cela ne risque pas d’effacer une modification manuelle ultérieure.

Ne pas mettre en place une suppression automatique risquée dans Google Docs. En cas d’erreur, privilégier une correction manuelle ou une opération compensatoire précisément localisée.

## 8.3 File de validation

L’interface doit afficher :

- notes en attente ;
- notes validées ;
- notes insérées ;
- notes rejetées ;
- erreurs d’insertion ;
- éléments incertains.

Actions :

- accepter ;
- modifier ;
- rejeter ;
- changer de section ;
- fusionner avec une note existante ;
- marquer comme décision/action/proposition ;
- réessayer l’insertion.

---

# 9. Interface utilisateur

## 9.1 Principes UX

- Interface en français.
- Utilisation prioritaire sur ordinateur Windows.
- Interface claire et professionnelle.
- Peu d’actions nécessaires pendant la réunion.
- État de capture toujours visible.
- Prévenir toute fermeture accidentelle pendant un enregistrement.
- Ne pas afficher des informations techniques inutiles aux participants.

## 9.2 Écrans minimum

### A. Connexion

- identifiant ou adresse e-mail ;
- mot de passe ;
- bouton de connexion ;
- message d’erreur générique ;
- lien de réinitialisation si un mécanisme sûr est implémenté ;
- aucune inscription publique.

### B. Tableau de bord

- bouton « Nouvelle réunion » ;
- réunions récentes ;
- état de connexion Google ;
- état du service audio/IA ;
- accès à l’administration selon le rôle.

### C. Préparation de réunion

- titre de la réunion ;
- date et heure ;
- type de réunion ;
- document Google Docs cible ;
- sélection d’un modèle ou d’un document existant ;
- aperçu des sections détectées ;
- configuration du microphone ;
- test audio ;
- choix du mode d’insertion ;
- confirmation des informations avant démarrage.

### D. Écran de réunion en direct

Disposition recommandée en trois zones :

1. **Contrôles** : démarrer, pause, reprendre, terminer ; chronomètre ; état du microphone.
2. **Contexte** : section active, annonce détectée, bouton de changement manuel de section.
3. **Notes IA** : notes proposées, statut, confiance, boutons accepter/modifier/rejeter.

Prévoir un indicateur de qualité/activité audio, sans prétendre mesurer la qualité de transcription avec une précision inexistante.

### E. Historique d’une réunion

- transcription disponible selon la rétention configurée ;
- notes par section ;
- historique des changements de section ;
- insertions Google Docs ;
- erreurs et reprises ;
- actions de nettoyage ou suppression selon les droits.

### F. Administration

- utilisateurs et rôles ;
- connexion Google ;
- configuration du Shared Drive et des dossiers autorisés ;
- modèles de réunion ;
- alias de sections ;
- paramètres Gemini ;
- politique de rétention ;
- journaux d’audit ;
- paramètres de sécurité.

## 9.3 Accessibilité et ergonomie

- boutons suffisamment grands ;
- raccourcis clavier facultatifs ;
- états visibles sans dépendre uniquement des couleurs ;
- confirmations pour les actions destructrices ;
- affichage responsive, mais priorité desktop ;
- contraste lisible ;
- prise en charge de Chrome et Edge récents.

---

# 10. Authentification et sécurité applicative

## 10.1 Authentification locale MeetRecord

L’application possède son propre système d’authentification.

Exigences :

- comptes créés par le Super administrateur ;
- mot de passe stocké uniquement sous forme de hash ;
- algorithme recommandé : Argon2id avec paramètres adaptés à l’environnement ;
- jamais de mot de passe en clair ;
- message identique si l’identifiant n’existe pas ou si le mot de passe est incorrect ;
- limitation des tentatives de connexion ;
- délai progressif ou verrouillage temporaire après plusieurs échecs ;
- cookies de session `HttpOnly`, `Secure`, `SameSite` adapté ;
- rotation et invalidation des sessions ;
- déconnexion effective ;
- expiration des sessions ;
- protection CSRF si authentification par cookie ;
- validation stricte des entrées ;
- journalisation des événements de sécurité sans stocker les secrets.

Ne pas stocker les jetons d’authentification dans `localStorage`.

## 10.2 Gestion des comptes

- pas de route d’inscription publique ;
- création par Super administrateur ;
- invitation par lien à usage unique et durée limitée, ou création initiale par l’administrateur ;
- possibilité de réinitialiser un mot de passe sans révéler si un compte existe ;
- désactivation d’un compte ;
- révocation des sessions actives ;
- journal des créations, changements de rôle et désactivations.

Le premier compte Super administrateur doit être créé par une commande CLI ou un mécanisme d’initialisation protégé, jamais avec un mot de passe par défaut documenté dans le dépôt.

## 10.3 Sécurité HTTP et réseau

- HTTPS obligatoire en production ;
- redirection HTTP vers HTTPS ;
- HSTS après validation du HTTPS ;
- CSP stricte et adaptée à Google OAuth ;
- `X-Content-Type-Options: nosniff` ;
- `Referrer-Policy` restrictive ;
- protection contre clickjacking (`frame-ancestors`) ;
- CORS limité au domaine MeetRecord ;
- rate limiting sur login et endpoints sensibles ;
- limites de taille sur les fichiers audio ;
- validation MIME réelle, pas seulement extension ;
- protection contre path traversal ;
- requêtes SQL paramétrées via ORM ;
- dépendances auditées ;
- secrets dans `.env` non versionné ou secrets Docker ;
- sauvegardes chiffrées et accès limité.

## 10.4 Non-indexation et accès privé

Exigences :

- `robots.txt` avec `Disallow: /` ;
- balise `noindex, nofollow, noarchive` ;
- en-tête `X-Robots-Tag: noindex, nofollow, noarchive` ;
- aucune page métier accessible sans authentification ;
- aucun contenu privé rendu dans le HTML initial ;
- routes API protégées côté serveur ;
- aucun document Google ou lien privé exposé publiquement ;
- aucune route de debug en production.

**Rappel :** robots.txt et noindex ne sécurisent pas une application. La sécurité repose sur l’authentification et l’autorisation serveur.

---

# 11. Confidentialité, données et conformité

Les réunions peuvent contenir des informations personnelles, des informations financières, des décisions internes et des noms de personnes.

Prévoir :

- information préalable des participants sur l’enregistrement et la transcription ;
- validation des règles internes du CLTO ;
- accès limité aux utilisateurs autorisés ;
- minimisation des données ;
- durée de conservation configurable ;
- suppression des fichiers audio après traitement selon la politique choisie ;
- possibilité de supprimer une réunion et ses données associées selon les droits ;
- journalisation des accès et modifications ;
- absence de données audio dans les logs applicatifs ;
- absence de données sensibles dans les outils d’analyse tiers ;
- documentation des flux de données envoyés à Gemini API.

Le fait d’utiliser une API Google ne dispense pas de vérifier les conditions contractuelles, la région de traitement, les paramètres de conservation et les règles applicables aux données envoyées.

Prévoir une configuration de rétention :

- audio brut : durée courte configurable, valeur initiale proposée 7 jours ;
- transcription : conservation selon politique de l’organisation ;
- notes et historique d’insertion : conservation liée au compte rendu ;
- logs techniques : durée limitée et sans contenu audio/transcription inutile.

La valeur de 7 jours est une proposition technique, pas une décision juridique. Elle doit être validée par le responsable du traitement.

---

# 12. Stack technique recommandée

Les versions doivent être **les dernières versions stables compatibles réellement disponibles au moment du développement**, vérifiées sur les registres officiels. Ne pas inventer de numéro de version. Verrouiller les versions effectivement installées dans le lockfile.

## 12.1 Frontend

- React 19 ou dernière version stable compatible.
- Vite 8 ou dernière version stable.
- TypeScript strict.
- Tailwind CSS **v4.3.5 demandé**, si cette version est effectivement publiée et installable au moment de l’implémentation. Si elle ne l’est pas, signaler le blocage et proposer la dernière v4 stable sans prétendre avoir installé la 4.3.5.
- React Router pour le routage.
- TanStack Query pour les données serveur.
- Zustand ou Context pour l’état local global, uniquement si nécessaire.
- Zod pour validation partagée.
- Lucide React pour les icônes.
- React Hook Form pour les formulaires si utile.

Ne pas multiplier les bibliothèques d’état.

## 12.2 Backend

Recommandation :

- Node.js LTS compatible avec les versions choisies ;
- TypeScript strict ;
- Fastify pour l’API HTTP, avec plugins maintenus ;
- WebSocket pour l’état de réunion et les événements en direct ;
- Zod pour valider les entrées et sorties ;
- aucun Prisma, Drizzle, TypeORM, Sequelize ou autre ORM/query builder ne doit être introduit.
- MariaDB comme base principale (requêtes préparées `mysql2`).

Le backend doit rester modulaire : auth, users, meetings, audio, transcription, analysis, sections, Google Drive, Google Docs, audit, settings.

## 12.3 Base de données

MariaDB.

Prévoir des migrations versionnées et reproductibles. Ne jamais modifier manuellement la base de production sans migration.

## 12.4 Traitements asynchrones

Pour le MVP :

- une file de jobs persistée en base peut suffire ;
- éviter d’ajouter Redis dès le premier jour si aucun besoin mesuré ne le justifie ;
- si une file robuste est nécessaire, évaluer BullMQ + Redis.

Les tâches audio/IA ne doivent pas bloquer les requêtes HTTP.

## 12.5 Audio

- API navigateur `getUserMedia`;
- `MediaRecorder`;
- WebSocket pour les événements d’état ;
- upload HTTP segmenté ou WebSocket binaire selon le POC ;
- codec supporté détecté dynamiquement ;
- buffer local limité pour segments non transmis ;
- reprise sur erreur réseau.

Ne jamais conserver un flux audio indéfiniment en mémoire.

## 12.6 Google

- bibliothèques officielles Google pour Node.js ;
- OAuth 2.0 ;
- Drive API v3 ;
- Google Docs API ;
- Gemini API via SDK officiel maintenu si disponible, sinon API REST documentée.

Ne pas appeler Google API directement depuis le navigateur avec un secret serveur.

## 12.7 Qualité et tests

- Vitest pour tests unitaires frontend/backend selon configuration ;
- Playwright pour tests end-to-end ;
- tests d’intégration MariaDB ;
- ESLint ;
- Prettier ;
- TypeScript `strict: true` ;
- audit npm ;
- tests de migration ;
- tests de sécurité des routes.

---

# 13. Structure de projet proposée

Monorepo TypeScript recommandé :

```text
meetrecord/
├── apps/
│   ├── web/                     # React + Vite + Tailwind
│   │   ├── src/
│   │   │   ├── app/
│   │   │   ├── components/
│   │   │   ├── features/
│   │   │   │   ├── auth/
│   │   │   │   ├── dashboard/
│   │   │   │   ├── meetings/
│   │   │   │   ├── audio/
│   │   │   │   ├── notes/
│   │   │   │   ├── google-drive/
│   │   │   │   └── admin/
│   │   │   ├── hooks/
│   │   │   ├── lib/
│   │   │   ├── routes/
│   │   │   └── styles/
│   │   └── ...
│   └── api/                     # Fastify + TypeScript
│       ├── src/
│       │   ├── modules/
│       │   │   ├── auth/
│       │   │   ├── users/
│       │   │   ├── meetings/
│       │   │   ├── audio/
│       │   │   ├── transcription/
│       │   │   ├── analysis/
│       │   │   ├── sections/
│       │   │   ├── google/
│       │   │   │   ├── drive/
│       │   │   │   └── docs/
│       │   │   ├── audit/
│       │   │   └── settings/
│       │   ├── plugins/
│       │   ├── workers/
│       │   ├── config/
│       │   └── server.ts
│       └── ...
├── packages/
│   ├── shared/                  # Types, schemas, enums partagés
│   └── config/                  # Configurations partagées
├── infra/
│   ├── docker/
│   ├── nginx/
│   └── scripts/
├── docs/
│   ├── architecture.md
│   ├── google-workspace-setup.md
│   ├── deployment-debian.md
│   ├── security.md
│   └── decisions/
├── docker-compose.yml
├── .env.example
├── .gitignore
├── package.json
├── pnpm-workspace.yaml
└── README.md
```

Cette arborescence est une proposition. Ajuster si un choix technique le justifie, sans mélanger frontend, backend et logique métier.

Gestionnaire de paquets recommandé : pnpm avec lockfile versionné.

---

# 14. Modèle de données initial

Schéma conceptuel à affiner avant migrations.

## 14.1 User

- `id`
- `email` unique, normalisé
- `displayName`
- `passwordHash`
- `role`
- `isActive`
- `createdAt`
- `updatedAt`
- `lastLoginAt`

## 14.2 Session

- `id`
- `userId`
- `tokenHash` ou identifiant de session opaque
- `expiresAt`
- `createdAt`
- `revokedAt`
- métadonnées minimales de sécurité

Le cookie contient un identifiant aléatoire opaque ; le stockage serveur ne doit pas conserver le jeton brut si une approche hashée est utilisée.

## 14.3 Meeting

- `id`
- `title`
- `meetingType`
- `status` : `draft`, `ready`, `recording`, `paused`, `processing`, `completed`, `error`, `cancelled`
- `startedAt`
- `endedAt`
- `createdById`
- `googleDriveId`
- `googleDocumentId`
- `googleDocumentUrl`
- `insertionMode`
- `retentionPolicyId`
- `createdAt`
- `updatedAt`

## 14.4 MeetingSection

- `id`
- `meetingId`
- `googleHeadingText`
- `normalizedTitle`
- `headingLevel`
- `documentStartIndex` (cache temporaire uniquement, jamais source de vérité durable)
- `aliases` JSON
- `isEnabled`
- `sortOrder`

Les index Google Docs doivent être recalculés à partir du document réel avant écriture.

## 14.5 AudioSegment

- `id`
- `meetingId`
- `sequenceNumber`
- `mimeType`
- `durationMs`
- `storageKey`
- `checksum`
- `status`
- `receivedAt`
- `processedAt`
- `errorCode` sans données sensibles

Contrainte unique `(meetingId, sequenceNumber)`.

## 14.6 TranscriptSegment

- `id`
- `meetingId`
- `audioSegmentId`
- `sequenceNumber`
- `text`
- `startMs`
- `endMs`
- `language`
- `provider`
- `createdAt`

## 14.7 SectionEvent

- `id`
- `meetingId`
- `fromSectionId`
- `toSectionId`
- `triggerText`
- `confidence`
- `sourceStartMs`
- `sourceEndMs`
- `decisionMode` : automatic / confirmed / manual
- `createdById` nullable
- `createdAt`

## 14.8 NoteItem

- `id`
- `meetingId`
- `sectionId`
- `type`
- `title`
- `content`
- `assignee`
- `dueDate`
- `confidence`
- `sourceStartMs`
- `sourceEndMs`
- `status` : draft / approved / rejected / inserted / insertion_error
- `createdBy` : ai / user
- `googleInsertionReference` nullable
- `createdAt`
- `updatedAt`

## 14.9 GoogleConnection

- `id`
- `provider`
- `googleAccountEmail`
- `encryptedAccessToken`
- `encryptedRefreshToken`
- `scopes`
- `expiresAt`
- `connectedById`
- `createdAt`
- `updatedAt`

Chiffrer les jetons avec une clé externe à la base, fournie par secret d’environnement. Prévoir une stratégie de rotation.

## 14.10 AuditLog

- `id`
- `actorUserId`
- `action`
- `resourceType`
- `resourceId`
- `metadata` JSON filtré
- `createdAt`

Ne jamais enregistrer le contenu audio, le mot de passe, les tokens OAuth ou les clés API dans l’audit log.

---

# 15. API applicative indicative

Les routes exactes peuvent évoluer, mais conserver une API cohérente et versionnée (`/api/v1`).

## Auth

- `POST /api/v1/auth/login`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/me`
- `POST /api/v1/auth/password/change`
- `POST /api/v1/auth/password/reset` si le flux de réinitialisation est configuré

## Users (admin)

- `GET /api/v1/users`
- `POST /api/v1/users`
- `PATCH /api/v1/users/:id`
- `POST /api/v1/users/:id/disable`
- `POST /api/v1/users/:id/reset-password`

## Google

- `GET /api/v1/google/connect`
- `GET /api/v1/google/callback`
- `GET /api/v1/google/status`
- `POST /api/v1/google/disconnect`
- `GET /api/v1/google/shared-drives`
- `GET /api/v1/google/files`
- `GET /api/v1/google/documents/:documentId/structure`

## Meetings

- `GET /api/v1/meetings`
- `POST /api/v1/meetings`
- `GET /api/v1/meetings/:id`
- `PATCH /api/v1/meetings/:id`
- `POST /api/v1/meetings/:id/start`
- `POST /api/v1/meetings/:id/pause`
- `POST /api/v1/meetings/:id/resume`
- `POST /api/v1/meetings/:id/stop`
- `GET /api/v1/meetings/:id/events`

## Audio

- `POST /api/v1/meetings/:id/audio-segments`
- `GET /api/v1/meetings/:id/audio-status`
- `POST /api/v1/meetings/:id/audio/retry-failed`

## Sections and notes

- `GET /api/v1/meetings/:id/sections`
- `PATCH /api/v1/meetings/:id/sections/:sectionId`
- `POST /api/v1/meetings/:id/sections/:sectionId/activate`
- `GET /api/v1/meetings/:id/notes`
- `PATCH /api/v1/notes/:noteId`
- `POST /api/v1/notes/:noteId/approve`
- `POST /api/v1/notes/:noteId/reject`
- `POST /api/v1/notes/:noteId/insert`
- `POST /api/v1/notes/:noteId/retry-insertion`

Toutes les routes doivent vérifier :

- session valide ;
- rôle/permission ;
- appartenance ou accès à la réunion ;
- validation des paramètres ;
- limites de débit et de taille si nécessaire.

---

# 16. Docker et déploiement VPS Debian 13

## 16.1 Environnement cible

- VPS OVH sous Debian 13 ;
- application accessible sur `https://meetrecord.cltobadminton.fr` ;
- DNS du sous-domaine configuré vers le VPS ;
- application dans Docker ;
- HTTPS via le reverse proxy déjà présent ou un proxy dédié ;
- données persistantes hors du cycle de vie des conteneurs.

Le VPS héberge déjà d’autres projets. **Ne pas modifier globalement le serveur, les règles firewall, le reverse proxy ou les réseaux Docker sans analyser l’existant et demander confirmation.**

## 16.2 Services Docker envisagés

MVP :

- `meetrecord-web` : frontend buildé et servi par Nginx ou serveur statique ;
- `meetrecord-api` : backend Node.js ;
- `meetrecord-db` : MariaDB ;
- éventuellement `meetrecord-worker` si les tâches audio sont séparées ;
- éventuellement Redis plus tard, si le besoin est démontré.

Autres infos :

- Le conteneur `meetrecord-db` utilise MariaDB.
- Le port interne est `3306`.
- La base de données est accessible uniquement depuis le réseau Docker interne.
- Les données persistantes sont conservées dans un volume Docker.

Le reverse proxy public existant peut router vers le conteneur web. Ne pas exposer MariaDB ni l’API interne directement sur Internet.

## 16.3 Docker

- images multi-stage ;
- utilisateurs non-root ;
- healthchecks ;
- limites CPU/RAM raisonnables ;
- volumes nommés pour MariaDB et stockage temporaire ;
- réseau Docker interne ;
- secrets fournis au runtime ;
- redémarrage automatique contrôlé ;
- logs structurés ;
- pas de `latest` en production sans verrouillage ;
- builds reproductibles.

## 16.4 Déploiement

Fournir `docs/deployment-debian.md` avec :

- prérequis ;
- DNS ;
- variables d’environnement ;
- configuration reverse proxy ;
- certificat TLS ;
- création des dossiers et permissions ;
- installation / mise à jour Docker ;
- build et lancement ;
- migrations ;
- création du premier Super administrateur ;
- connexion Google OAuth ;
- tests de santé ;
- sauvegarde ;
- restauration ;
- rollback.

## 16.5 Sauvegardes

Prévoir :

- sauvegarde MariaDB ;
- sauvegarde des configurations non secrètes ;
- sauvegarde chiffrée des données persistantes nécessaires ;
- politique de rétention ;
- test de restauration documenté.

Les fichiers audio temporaires ne doivent pas être sauvegardés automatiquement si la politique de rétention prévoit leur suppression.

---

# 17. Variables d’environnement

Créer un `.env.example` complet, sans valeur secrète réelle.

Exemple indicatif :

```dotenv
NODE_ENV=production
APP_NAME=MeetRecord
APP_URL=https://meetrecord.cltobadminton.fr
API_URL=https://meetrecord.cltobadminton.fr/api
APP_TIMEZONE=Europe/Paris

DATABASE_URL=mariadb://meetrecord:CHANGE_ME@db:3306/meetrecord

SESSION_SECRET=CHANGE_ME_LONG_RANDOM_SECRET
ENCRYPTION_KEY=CHANGE_ME_32_BYTE_KEY
COOKIE_SECURE=true

GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
GOOGLE_REDIRECT_URI=https://meetrecord.cltobadminton.fr/api/v1/google/callback
GOOGLE_OAUTH_SCOPES=

GOOGLE_SHARED_DRIVE_ID=
GOOGLE_DEFAULT_FOLDER_ID=
GOOGLE_TEMPLATE_DOCUMENT_ID=

GEMINI_API_KEY=
GEMINI_MODEL=
GEMINI_MAX_OUTPUT_TOKENS=

AUDIO_STORAGE_PATH=/data/audio
AUDIO_SEGMENT_DURATION_SECONDS=45
AUDIO_MAX_SEGMENT_SIZE_MB=
AUDIO_RETENTION_DAYS=7

LOG_LEVEL=info
```

Adapter les variables au code réel. Ne pas conserver des variables inutilisées. Les secrets ne doivent jamais être inclus dans `VITE_*`, car ces variables sont exposées au frontend.

---

# 18. Observabilité et gestion des erreurs

L’utilisateur doit comprendre l’état de l’application sans lire les logs.

États à afficher :

- microphone non autorisé ;
- microphone prêt ;
- enregistrement actif ;
- pause ;
- envoi audio en cours ;
- retard de traitement ;
- transcription en cours ;
- analyse IA en cours ;
- note en attente ;
- insertion Google réussie ;
- erreur Google ;
- erreur Gemini ;
- perte réseau ;
- reprise automatique.

Backend :

- logs structurés JSON ;
- correlation ID par requête ;
- meeting ID et segment ID dans les logs ;
- masquage des données personnelles ;
- métriques simples : durée de traitement, taille audio, erreurs API, retard de traitement.

Ne jamais loguer :

- audio brut ;
- token OAuth ;
- clé Gemini ;
- mot de passe ;
- cookie de session ;
- transcription intégrale par défaut.

---

# 19. Tests et critères d’acceptation

## 19.1 Authentification

- Un utilisateur valide peut se connecter.
- Un mot de passe erroné et un e-mail inconnu produisent le même message.
- Un compte désactivé ne peut pas se connecter.
- Les tentatives répétées sont limitées.
- Le hash du mot de passe est Argon2id.
- Les routes privées refusent les requêtes non authentifiées.
- Les cookies ont les attributs de sécurité attendus.
- Aucun secret n’est présent dans le bundle frontend.

## 19.2 Capture audio

- Le navigateur liste les microphones disponibles.
- L’utilisateur peut sélectionner le microphone USB.
- L’utilisateur peut tester le niveau audio avant la réunion.
- Le refus d’autorisation est expliqué clairement.
- L’enregistrement fonctionne pendant au moins 2 h 30 lors d’un test.
- Une perte réseau temporaire ne fait pas perdre silencieusement les segments.
- Les segments reçus en double ne sont pas traités deux fois.
- Pause/reprise fonctionne.
- L’arrêt final ferme correctement les ressources audio.

## 19.3 Transcription et IA

Créer un jeu de test audio représentatif (avec consentement) contenant :

- annonces de changement de commission ;
- noms propres ;
- montants et dates ;
- votes ;
- propositions non validées ;
- décisions ;
- discussions qui reviennent sur une rubrique précédente ;
- phrases ambiguës et chevauchements de voix.

Critères :

- la transcription est en français ;
- les timestamps sont cohérents ;
- les chiffres ne sont pas supprimés ;
- une incertitude est signalée plutôt qu’inventée ;
- une proposition n’est pas convertie en décision ;
- la section active change lors d’une annonce explicite ;
- une section ambiguë demande confirmation ;
- une section précédente peut être réactivée.

## 19.4 Google Docs

- La connexion OAuth fonctionne.
- Le Shared Drive est accessible avec les permissions attendues.
- L’application peut lister les documents du dossier autorisé.
- L’application peut lire les titres et sous-titres du document.
- L’utilisateur peut sélectionner un document cible.
- Une note approuvée est insérée dans la bonne section.
- Le texte existant n’est pas supprimé.
- Un tableau existant est conservé.
- Une modification manuelle concurrente n’est pas écrasée.
- Un retry ne duplique pas la note.
- Une erreur de permission est clairement affichée.
- Le document modèle original n’est jamais modifié lorsqu’une copie est demandée.

## 19.5 Tests end-to-end

Scénario principal :

1. Connexion en tant qu’utilisateur autorisé.
2. Création d’une réunion.
3. Sélection du document Google Docs.
4. Détection des sections.
5. Test du microphone.
6. Démarrage de l’enregistrement.
7. Phrase : « On passe à la Commission Informatique ».
8. Discussion sur le renouvellement de quatre tablettes.
9. Génération d’une note.
10. Validation de la note.
11. Insertion dans la section Informatique.
12. Annonce d’une autre section.
13. Vérification de la nouvelle section active.
14. Arrêt de la réunion.
15. Vérification du document et de l’historique.

---

# 20. Phasage de développement imposé

Claude doit respecter ce découpage. Ne pas passer au lot suivant si les critères du lot courant ne sont pas atteints.

## Lot 0 — Étude et décisions d’architecture

Livrables :

- architecture détaillée ;
- choix des versions réellement disponibles ;
- comparaison Gemini audio vs Whisper ;
- vérification des limites Gemini API et du coût ;
- vérification des scopes Google ;
- plan de sécurité ;
- diagramme des flux audio et Google Docs ;
- liste des décisions à confirmer.

Aucun développement fonctionnel important avant validation de l’architecture.

## Lot 1 — Socle technique

- monorepo ;
- pnpm ;
- React/Vite/Tailwind ;
- API Fastify ;
- MariaDB ;
- mysql2 (requêtes préparées) ;
- Docker Compose local ;
- lint, format, tests ;
- healthcheck ;
- documentation de démarrage.

Critère : frontend et backend démarrent en local et communiquent.

## Lot 2 — Authentification et administration

- modèle User/Session ;
- hash Argon2id ;
- login/logout ;
- message générique ;
- rate limiting ;
- création du Super administrateur ;
- CRUD utilisateurs ;
- rôles et protections serveur.

Critère : toutes les routes privées sont protégées et testées.

## Lot 3 — Connexion Google Workspace

- OAuth ;
- stockage chiffré des tokens ;
- état de connexion ;
- lecture du Shared Drive ;
- sélection d’un document ;
- lecture de la structure Google Docs ;
- affichage des rubriques détectées.

Critère : l’utilisateur peut sélectionner un document de son Drive partagé et voir ses sections.

## Lot 4 — Écriture Google Docs sécurisée

- insertion de texte dans une section ;
- gestion des titres et index ;
- sérialisation des écritures ;
- idempotence ;
- gestion des conflits ;
- audit des insertions.

Critère : un texte de test est ajouté à la bonne section sans altérer le reste du document.

## Lot 5 — Capture audio

- sélection du microphone ;
- test audio ;
- MediaRecorder ;
- segmentation ;
- upload ;
- reprise réseau ;
- pause/reprise/arrêt ;
- persistance de l’état.

Critère : enregistrement de test de 2 h 30 sans perte silencieuse.

## Lot 6 — Transcription

- POC des fournisseurs ;
- adaptateur de transcription ;
- segments horodatés ;
- reprise après erreur ;
- affichage de la transcription ;
- rétention et suppression.

Critère : transcription exploitable d’un enregistrement représentatif.

## Lot 7 — Classification des sections

- détection d’annonces ;
- alias ;
- section active ;
- score de confiance ;
- confirmation manuelle ;
- retour à une section précédente ;
- journal des changements.

Critère : scénario de changement de section validé avec un audio de test.

## Lot 8 — Synthèse et notes

- prompts versionnés ;
- schéma JSON ;
- validation Zod ;
- dédoublonnage ;
- types de notes ;
- file de validation ;
- correction manuelle.

Critère : notes structurées fidèles à la transcription et à la section.

## Lot 9 — Intégration de bout en bout

- capture → transcription → classification → synthèse → validation → insertion Google Docs ;
- gestion des erreurs ;
- indicateurs en direct ;
- test complet sur une réunion simulée.

Critère : le parcours complet fonctionne sans intervention technique.

## Lot 10 — Durcissement et production

- audit sécurité ;
- tests E2E ;
- optimisation mémoire/CPU ;
- politique de rétention ;
- sauvegardes ;
- reverse proxy et HTTPS ;
- déploiement Debian 13 ;
- test de restauration ;
- documentation utilisateur et administrateur.

Critère : application exploitable en réunion réelle, avec procédure de secours.

---

# 21. Scénarios fonctionnels détaillés

## Scénario A — Réunion normale

- Maxime se connecte à MeetRecord.
- Il crée une réunion « Conseil d’administration – 24/11/2026 ».
- Il sélectionne le compte rendu préparé par la secrétaire dans le Drive partagé.
- MeetRecord détecte les rubriques.
- Il sélectionne le microphone USB et réalise un test.
- Il démarre l’enregistrement.
- Les participants discutent.
- L’IA génère une proposition de note dans la rubrique active.
- Maxime valide ou corrige.
- MeetRecord insère le contenu dans Google Docs.
- La secrétaire peut continuer à travailler dans le document.
- En fin de réunion, l’enregistrement est arrêté et les éléments en attente sont traités.

## Scénario B — Changement de commission

Transcription :

> « Nous avons terminé la partie trésorerie. On passe à la Commission Informatique. »

Résultat attendu :

- l’IA reconnaît la fin de la section Trésorerie ;
- elle identifie Commission Informatique ;
- l’interface affiche la nouvelle section ;
- les notes suivantes sont affectées à cette section ;
- un événement de changement est conservé.

## Scénario C — Retour sur une section antérieure

Transcription :

> « Avant de passer au point suivant, je voudrais revenir sur la trésorerie. »

Résultat attendu :

- MeetRecord propose le retour à Trésorerie ;
- si la confiance est élevée, l’interface affiche le changement ;
- les nouvelles notes sont ajoutées sans supprimer les précédentes ;
- l’historique conserve le retour.

## Scénario D — Discussion ambiguë

Transcription :

> « Pour le site, il faudrait voir avec l’équipe communication. »

Résultat attendu :

- ne pas basculer automatiquement vers Commission Communication si aucune annonce de changement n’est faite ;
- garder la section active ;
- éventuellement proposer une référence croisée ou une suggestion.

## Scénario E — Décision non confirmée

Transcription :

> « On pourrait peut-être acheter quatre tablettes reconditionnées. Il faudra regarder les devis. »

Résultat attendu :

- enregistrer une proposition ou une action d’étude ;
- ne pas écrire « Le CA décide d’acheter quatre tablettes ».

## Scénario F — Problème Google Docs

- Google API renvoie une erreur temporaire.
- La note reste stockée dans MeetRecord avec le statut `insertion_error`.
- L’interface indique que la note n’a pas été insérée.
- L’utilisateur peut réessayer.
- Le retry ne crée pas de doublon si la première requête a finalement été appliquée.

---

# 22. Règles de développement et qualité du code

- TypeScript strict, éviter `any`.
- Fonctions courtes et responsabilité unique.
- Architecture modulaire, séparation contrôleurs/services/repositories/providers.
- Aucun appel Gemini ou Google depuis le frontend avec une clé secrète.
- Toutes les entrées externes sont validées.
- Erreurs typées et réponses API cohérentes.
- Pas de logique métier dans les composants React.
- Pas de requêtes SQL construites par concaténation.
- Transactions DB pour les opérations multi-étapes.
- Idempotency keys pour les traitements audio et insertions.
- Timeouts et retries avec backoff pour les APIs externes.
- Limiter les retries sur les erreurs non récupérables (401, 403, données invalides).
- Tests unitaires pour les fonctions de classement et de génération de notes.
- Tests d’intégration pour Google Docs via mocks puis environnement de test réel.
- Aucun secret dans le dépôt.
- `.env.example` maintenu à jour.
- README et documentation mis à jour à chaque changement structurant.
- Toute décision d’architecture importante doit être consignée dans `docs/decisions/` sous forme d’ADR.

---

# 23. Points à confirmer avant le développement

Ces points ne doivent pas être inventés :

1. **Compte Google de connexion :** compte Workspace qui autorisera l’application à accéder au Shared Drive.
2. **Identifiant du Shared Drive :** à renseigner dans la configuration.
3. **Dossier de destination :** dossier dans lequel les copies de comptes rendus doivent être créées, si le mode modèle est utilisé.
4. **Modèle Google Docs :** identifier le document modèle natif Google Docs. Le DOCX fourni est une référence de style, pas nécessairement le modèle de production.
5. **Mode de travail :** document déjà créé par la secrétaire ou copie d’un modèle créée par MeetRecord. Les deux parcours sont à prendre en charge si possible.
6. **Insertion par défaut :** validation avant insertion (recommandé) ou insertion automatique avec validation après coup.
7. **Rétention audio :** valeur proposée 7 jours, à valider par l’organisation.
8. **Fournisseur de transcription :** choisir après POC et vérification du coût Gemini API.
9. **Configuration du VPS :** mémoire, CPU, espace disque, reverse proxy, réseaux Docker et ports déjà utilisés à inventorier avant déploiement.
10. **Information des participants :** texte d’information et procédure de consentement/validation interne à définir par l’organisation.

---

# 24. Définition de « terminé »

MeetRecord ne sera considéré comme livré que lorsque :

- un utilisateur autorisé peut se connecter ;
- le Super administrateur peut gérer les comptes ;
- l’application est inaccessible sans authentification ;
- le domaine `meetrecord.cltobadminton.fr` sert l’application en HTTPS ;
- l’application n’est pas indexable ;
- le microphone USB est sélectionnable ;
- une réunion de plus de deux heures peut être enregistrée ;
- les segments audio sont traités sans perte silencieuse ;
- la transcription française est exploitable ;
- les annonces de sections sont reconnues ;
- les changements de rubrique sont visibles et corrigeables ;
- les notes sont synthétiques et fidèles ;
- les propositions et décisions sont distinguées ;
- les notes sont insérées dans la bonne rubrique du Google Docs du Shared Drive ;
- le contenu manuel existant est préservé ;
- les insertions sont idempotentes ;
- les erreurs Google/Gemini sont récupérables ;
- l’historique est consultable ;
- les données audio sont supprimées selon la politique définie ;
- les tests principaux passent ;
- la procédure de sauvegarde/restauration est documentée ;
- la documentation d’installation, de configuration et d’utilisation est complète.

---

# 25. Première mission à exécuter par Claude

**Ne commence pas immédiatement à coder.**

Commence par :

1. reformuler les exigences principales ;
2. relever les risques techniques ;
3. vérifier les versions stables réellement disponibles des technologies ;
4. vérifier la documentation actuelle des API Google Drive, Google Docs et Gemini ;
5. expliquer la différence entre l’abonnement Gemini grand public et la facturation Gemini API ;
6. proposer le POC minimal permettant de valider la capture audio, la transcription française et l’insertion dans une section Google Docs ;
7. produire le diagramme d’architecture ;
8. proposer les ADR initiaux ;
9. lister les informations à obtenir du propriétaire ;
10. attendre validation avant de commencer le Lot 1.

Le premier POC doit prouver le point le plus risqué du projet : **une annonce orale de changement de rubrique doit conduire à l’insertion d’une note synthétique dans la bonne section d’un Google Docs situé dans un Drive partagé, sans altérer le contenu déjà présent.**

Fin du cahier des charges.

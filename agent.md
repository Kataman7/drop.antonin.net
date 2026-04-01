Agent Guidelines (Architecture + Clean Code)

1. Objectif global
   Le projet suit une Clean Architecture / Hexagonal Architecture stricte.
   Le but est de produire du code maintenable, testable et lisible.
   La logique métier doit rester pure et indépendante.
   Aucune règle d’architecture ne doit être violée.

2. Architecture du projet

2.1 Règle principale : sens des dépendances
Les dépendances doivent toujours pointer vers l’intérieur :
presentation → application → domain
L’infrastructure peut dépendre de application/domain, mais domain ne dépend jamais de infrastructure.

2.2 Responsabilité des dossiers

src/domain/
Contient uniquement le métier : entités, value objects, domain services, events, erreurs métier.
Interdictions : aucun import de framework, aucune DB, aucun appel HTTP, aucune logique UI, aucun service externe.
Ce code doit pouvoir être testé sans mocks.

src/application/
Contient les workflows : les use cases.
Les use cases orchestrent la logique domain et utilisent uniquement des interfaces (ports) pour parler au monde extérieur.
Interdictions : pas d’accès direct DB, pas d’appel API externe direct, pas de dépendance vers infrastructure.

src/application/ports/
Contient uniquement des interfaces : repositories, mail/sms providers, payment gateways, logger, auth, etc.

src/infrastructure/
Contient les implémentations techniques : Prisma/Mongo/SQL, implémentations de repositories, Stripe/Twilio/Mailer, logs.
Règles : infrastructure implémente les ports, mais ne doit jamais exposer ses types vers domain/application.

src/presentation/
Contient tout ce qui est interface utilisateur ou API : controllers, routes, middlewares, validation, UI pages/components.
Règles : aucune logique métier ici. Les controllers appellent uniquement les use cases.
presentation ne doit jamais appeler infrastructure directement.

src/main/
Contient uniquement le wiring : injection de dépendances, bootstrap, server startup.
Interdiction : pas de logique métier.

src/shared/
Contient uniquement des utilitaires vraiment génériques (types, helpers date/string, constantes).
Interdiction : ne pas y mettre du métier déguisé.

3. Règles strictes Clean Code

3.1 Fonctions
Chaque fonction doit être très courte (idéalement < 20 lignes).
Chaque fonction doit faire une seule chose.
Si une fonction est difficile à nommer clairement, elle fait trop de choses.
Aucune fonction ne doit avoir plusieurs responsabilités.

3.2 Nommage
Le code doit être compréhensible sans commentaire.
Les noms doivent être explicites, même s’ils sont longs.
Éviter les abréviations floues.

Mauvais : data, tmp, res, handleThing()
Bon : appointmentStartDate, calculateInvoiceTotal(), validatePatientPhoneNumber()

3.3 Interdiction des booléens en paramètre
Aucun booléen ne doit être passé en paramètre d’une fonction.

Interdit : sendEmail(user, true)
Correct : sendInvoiceEmail(user), sendReminderEmail(user)

Interdit : createInvoice(patient, false)
Correct : createDraftInvoice(patient), createFinalInvoice(patient)

3.4 Conditions
Éviter l’imbrication profonde.
Utiliser des early returns.
Extraire les conditions complexes dans des fonctions bien nommées.

3.5 Gestion des erreurs
Ne jamais retourner null ou undefined silencieusement.
Utiliser des erreurs explicites et typées.
Les erreurs doivent être précises : PatientNotFoundError, AppointmentConflictError, InvoiceAlreadyPaidError.

3.6 Types et Value Objects
Préférer les Value Objects aux primitives quand ça apporte de la clarté.
Exemples : Email, Money, PhoneNumber, AppointmentDate.

3.7 Valeurs magiques interdites
Aucune valeur hardcodée sans explication.
Utiliser des constantes explicites.

3.8 Commentaires
Éviter les commentaires qui répètent le code.
Les commentaires sont autorisés uniquement pour expliquer une règle métier ou une contrainte importante.

4. Pattern obligatoire des Use Cases
   Chaque use case doit suivre ce schéma :

* input DTO
* validation basique
* exécution de logique domain
* persistance via repository port
* output DTO

Un use case ne doit jamais contenir : requêtes DB directes, logique HTTP, logique UI.

5. Repositories
   Les repositories doivent exposer des objets domain, pas des modèles Prisma.
   Ils doivent cacher totalement la structure DB.
   Ils doivent avoir des méthodes explicites : findById, findByEmail, save, delete.

Interdit : repository.query("SELECT ...")

6. Tests
   Domain : un maximum de tests unitaires.
   Application : tests de use cases avec ports mockés.
   Infrastructure : tests d’intégration.
   Presentation : tests API/e2e.

7. Patterns interdits
   Pas de god class, pas de god service.
   Pas de fichiers énormes (>300 lignes).
   Pas de fichier “helpers.ts” fourre-tout.
   Pas de dépendances circulaires.

8. Règles de génération de code
   Toujours respecter les frontières d’architecture.
   Créer uniquement le code nécessaire.
   Ne pas sur-architecturer.
   Favoriser la clarté plutôt que l’astuce.

Résumé final
Le projet doit rester clean, modulaire, prévisible et testable.
L’architecture est non négociable.
Les règles Clean Code sont obligatoires.

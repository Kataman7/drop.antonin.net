# Plan type – Étude d'impact RGPD + IA Act

---

## I. Le RGPD s'applique-t-il ? (Champ d'application)

**D'abord**, identifier si on a affaire à des **données personnelles** (= toute info qui permet d'identifier une personne physique, directement ou indirectement).

**Ensuite**, vérifier que les opérations effectuées constituent bien des **traitements** au sens du RGPD (= toute opération sur ces données : collecte, transmission, stockage...).

**→ Si oui aux deux : le RGPD s'applique, on continue l'analyse.**

---

## II. Les bases légales sont-elles valides ?

**Pour chaque traitement**, se demander : sur quoi l'entreprise s'appuie-t-elle pour justifier la collecte ?

**Le consentement** → vérifier qu'il est : libre, spécifique, éclairé, univoque, et qu'il résulte d'un **acte positif**. ⚠️ Si les données sont obligatoires pour accéder au service, ce n'est plus du consentement → utiliser plutôt l'exécution du contrat.

**L'exécution du contrat** → les données doivent être **strictement nécessaires** à la prestation. Si on peut faire autrement (moyens moins intrusifs), cette base ne tient pas.

**L'intérêt légitime** → faire une **balance des intérêts** : l'intérêt de l'entreprise est-il proportionné aux droits des personnes ? Les utilisateurs pouvaient-ils s'y attendre raisonnablement ?

---

## III. Les autres principes du RGPD sont-ils respectés ?

Même si la base légale est valide, **ce n'est pas suffisant**. Il faut aussi vérifier :

**La transparence** → les utilisateurs doivent être informés de façon précise : qui collecte, pourquoi, et **à qui les données sont transmises** (les partenaires doivent être nommément identifiés).

**La finalité** → chaque traitement doit avoir un **but précis et clairement défini**.

**La minimisation** → on ne collecte **que ce qui est nécessaire** pour atteindre ce but. Si on peut faire autrement avec moins de données, les données supplémentaires sont en trop.

**La durée de conservation** → les données ne doivent pas être gardées plus longtemps que nécessaire.

**La sécurité** → les données stockées doivent être **protégées** (chiffrement, etc.). Sans protection, l'entreprise engage sa responsabilité en cas de fuite.

---

## IV. Obligations liées à l'IA Act (si un système d'IA est en jeu)

**D'abord**, classifier le système selon son niveau de risque :
- Risque inacceptable → interdit
- Risque élevé → obligations lourdes
- **Risque limité (ex : génération d'images/textes)** → obligations de transparence
- Risque minimal → presque rien

**Ensuite**, identifier le rôle de l'entreprise :
- **Fournisseur** (elle a développé le système) → doit s'assurer que les contenus générés sont **marqués comme tels** (watermark)
- **Déployeur** (elle l'utilise dans son appli) → doit **informer les utilisateurs** que le contenu est généré par IA

⚠️ Une même entreprise peut être **les deux à la fois**.

---

# Anex :

## Les 4 niveaux de risque de l'IA Act

**Risque inacceptable (interdit)** → IA qui manipule les gens à leur insu, notation sociale par les gouvernements, reconnaissance faciale en temps réel dans les espaces publics (sauf exceptions).

**Risque élevé** → IA qui prend des décisions importantes sur ta vie : recrutement, crédit bancaire, justice, médecine, sécurité des infrastructures.

**Risque limité** → IA qui génère ou imite du contenu : chatbots, deepfakes, génération d'images/textes. Le danger c'est que tu ne saches pas que c'est une IA → obligation de te le dire.

**Risque minimal** → tout le reste : filtres anti-spam, recommandations Netflix, jeux vidéo... Pas d'obligations particulières.

---

## L'intérêt légitime, c'est quoi concrètement ?

C'est une base légale qui dit : *"je n'ai pas ton consentement, mais j'ai un intérêt valable à utiliser tes données."*

Le problème c'est que ça peut devenir un prétexte, donc la loi impose de faire une **balance** :

> **Mon intérêt à moi** (faire de la pub, améliorer mon service...) **est-il plus fort que l'atteinte à la vie privée de l'utilisateur ?**

Et surtout : **l'utilisateur pouvait-il s'y attendre ?** Si tu commandes un kebab en ligne, tu t'attends peut-être à recevoir des pubs kebab. Tu t'attends beaucoup moins à ce que ta géolocalisation exacte soit revendue à un réseau social.

→ Plus l'atteinte est intrusive et surprenante, moins l'intérêt légitime peut être invoqué.

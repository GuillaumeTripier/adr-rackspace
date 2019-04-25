# adr-rackspace

# Contexte global :
Mailtrust est une subdivision de rackspace. Les techniciens de l'assistance technique doivent examiner les logs de messagerie pour résoudre les problèmes des clients. Des recherches sont en générale effectuées des centaines de fois par jour. Il est donc nécessaire de réaliser ces recherches très rapidement et précisément. Les outils fournissant cette fonctionnalité doivent donc être rapides et précis. D'après les dernières informations, il y a 600 serveurs de messagerie et des centaines de giga-octets de données de logs générées chaque jour. Il faut donc des performances à la hauteur.


## V1
* #### Contexte 1
  Les fichiers de log sont stockés dans des fichiers texte sur le disque local de chaque serveur de mail. Ces logs sont conservés pendant 14 jours. Les techniciens du support n'ont pas accès aux serveurs mail. Par conséquent, ils doivent faire appel aux ingénieurs à l'aide de tickets. Les ingénieurs doivent, dès la réception d'un ticket, rechercher les logs sur chaque serveur en se connectant en ssh en utilisant les commandes grep / var / log / maillog.
* #### Problématique 1
  Avec plus de 12 serveurs, ce processus manuel de connexion à chaque serveur prend trop de temps pour les ingénieurs qui ne parviennent plus à traiter toutes les demandes.
* #### Contexte 2
    Les ingénieurs utilisent maintenant un script pour ne plus avoir à rechercher sur chaque machine manuellement et une par une.
* #### Problématique 2
  Avec le grand nombre de tickets, les recherches associées ralentissent les machines.
  
* #### Cas d’utilisation
  Un technicien du support reçoit une demande de la part d'un client. Le technicien envoi donc un ticket à un ingénieur qui va lancer le script de recherche et va renvoyer les résultats au technicien. Le technicien réagit en fonction des résultats obtenus.

* #### Le scénario d’attribut de qualité : la performance
  * Stimulus : lancement du script de recherche
  * Source : l'ingénieur
  * Réponse : résultats de la recherche
  * Mesure : temps de réponse
  * Environnement : plusieurs machines serveurs distinctes
  * Artefact : fichiers de logs
  
* #### Vue de la structure architecturale
![image_vue_de_la_structure_architecturale_V1](https://raw.githubusercontent.com/GuillaumeTripier/adr-rackspace/master/Structure_V1.png)

## V2
* #### Contexte 1
  L’équipe de support peut utiliser un outil Web qui leur permet de faire des recherches dans les logs sans avoir à passer par les ingénieurs. Seules les recherches non-génériques sont autorisées pour des raisons de performance due au volume très important des données. Afin de faciliter la suppression des logs anciens, chaque jour, les logs sont stockés dans une nouvelle table de la base de données. Pour limiter la taille totale de la base de données, les logs ne sont conservés que pendant 3 jours. Cette version du système de logs n'a jamais été utilisée en production.
* #### Problématique 1
  Au fur et à mesure que les tables dans la base de données grandissent, l'indexation de chaque entrée insérée est de plus en plus lente. Après un certain temps d'exécution, la base de données ne parvient pas à traité les données qu'elle reçoit suffisamment vite pour traiter la réception suivante.
* #### Contexte 2
    Le problème de la version précédente a été corrigé en mettant une file d'attente pour les entrées des logs dans des fichiers textes locaux sur le serveur de logs centralisé afin qu'elles régulièrement chargées. Cela permet de charger beaucoup de données d'un coup et ainsi gagner du temps. Cette version est assez rapide pour être mise en production.
* #### Problématique 2
  * Le chargement dans la base de données est de plus en plus lent à mesure que les entrées de logs sont nombreuses.
  * Dans cet état, le système n'est pas évolutif sans travail supplémentaire.
* #### Contexte 3
    Les entrées des logs ne sont plus directement ajouté dans une seule base de données. Elles sont stockées dans la table de la base de données qui est nouvelle toutes les 10 minutes.
* #### Problématique 3
  * Cette version du système de journalisation a fonctionné de manière fiable pendant environ un an. Mais il y a des problèmes qui surviennent due au nombre de clients, serveurs. Des problèmes étranges tels que des erreurs aléatoires lors de la création de nouvelles tables dans la base de données. Ces erreurs deviennent de plus en plus fréquentes ce qui cause des pertes de logs.
  * L'équipe du support perd confiance en la précision du système. Il arrive à plusieurs reprises que les ingénieurs effectuent une mise à niveau logicielle vers une application particulière. Cela modifie le format des logs ce qui provoque un disfonctionnement des scripts. Comme le nombre de clients augmente, le système se doit d'être évolutif.
  
* #### Cas d’utilisation
  Un ingénieur effectue une mise à jour logicielle. Le système de logs redémarre normalement.

* #### Le scénario d’attribut de qualité : la fiabilité
  * Stimulus : mise à niveau logiciel
  * Source : l'ingénieur
  * Réponse : création de logs toujours active
  * Mesure : temps de relancement de la création des logs
  * Environnement : une seule machine, une seule base de données, un très grand nombre de tables
  * Artefact : fichiers de logs
  
* #### Vue de la structure architecturale
![image_vue_de_la_structure_architecturale_V2](https://raw.githubusercontent.com/GuillaumeTripier/adr-rackspace/master/Structure_V2.png)


## V3
* #### Contexte
  Après un examen de plusieurs applications commerciales qui permettent de traiter des logs, Splunk s'est démarqué et a fait à peu près tout ce que nous voulions. Cependant, l'utilisation de ce type de produit pourrait limiter la création de nouvelles fonctionnalités. D'autres entreprises utilisaient Hadoop pour leur propre traitement de logs à très grande échelle. C'est alors la solution choisie, car Hadoop est un projet dynamique  et open source ce qui permet d'avoir une grande confiance dans ces évolutions avenir.
* #### Problématique
  Les seuls problèmes rencontrés sont les bogues en interne qui sont immédiatement corrigés.
  
* # Cas d’utilisation
  Un technicien effectue une recherche dans les logs à l'aide de Hadoop et reçoit la réponse.

* # Le scénario d’attribut de qualité : la performance
  * Stimulus : lancement de la recherche sur Hadoop
  * Source : le technicien
  * Réponse : résultats de la recherche
  * Mesure : temps de réponse
  * Environnement : une machine qui contient le système Hadoop
  * Artefact : fichiers de logs, Hadoop
  
* #### Vue de la structure architecturale
![image_vue_de_la_structure_architecturale_V3](https://raw.githubusercontent.com/GuillaumeTripier/adr-rackspace/master/Structure_V3.png)
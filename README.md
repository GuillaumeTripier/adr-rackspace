# adr-rackspace

# Contexte global :
Mailtrust est une subdivision de rackspace. Les techniciens de l'assistance technique doivent examiner les logs de messagerie pour résoudre les problèmes des clients. Des recherches sont en générale effectuées des centaines de fois par jour. Il est donc nécessaire de réaliser ces recherches très rapidement et précisément.Les outils fournissant cette fonctionnalité doivent donc être rapides et précis. D'après les dernières informations, il y a 600 serveurs de messagerie et des centaines de giga-octets de données de logs générées chaque jour. Il faut donc des performances à la hauteur.


##V1
* #### Contexte 1
  Les fichiers de log sont stockés dans des fichiers texte sur le disque local de chaque serveur de mail. Ces logs sont conservés pendant 14 jours. Les techniciens du support n'ont pas accès aux serveurs mail. Par conséquent, ils doivent faire appel aux ingénieurs à l'aide de tickets. Les ingénieurs doivent, dès la réception d'un ticket, rechercher les logs sur chaque serveur en se connectant en ssh en utilisant les commandes grep / var / log / maillog.
* #### Problématique 1
  Avec plus de 12 serveurs, ce processus manuel de connexion à chaque serveur prend trop de temps pour pour les ingénieurs qui ne parviennent plus à traiter toutes les demandes.
* #### Contexte 2
    Les ingénieurs utilisent maintenant un script pour ne plus avoir à rechercher sur chaque machine manuellement et une par une.
* #### Problématique 2
  Avec le grand nombre de tickets, les recherches associées ralentissent les machines.
  
* #### Cas d’utilisation
  Un technicien du support reçoit une demande de la part d'un client. Le technicien envoit donc un ticket à un ingénieur qui va lancer le script de recherche et va renvoyer les résultats au technicien. Le technicien réajit en fonction des résultats obtenus.

* #### Le scénario d’attribut de qualité : la performance
  * Stimulus : lancement du script de recherche
  * Source : l'ingénieur
  * Réponse : résultats de la recherche
  * Mesure : temp de réponse
  * Environnement : plusieurs machines serveurs distinctes
  * Artefact : fichiers de logs
  
* #### Vue de la structures architecturale
![image_vue_de_la_structure_architecturale_V1](https://raw.githubusercontent.com/GuillaumeTripier/adr-rackspace/master/Structure_V1.png)

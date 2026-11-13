# Compte rendu TP PetitPanier — ZAK / NOM2
Systèmes utilisés : Linux

## Palier 1
1.1 : 
Client = c'est lui qui reçoit mes requêtes et les transmet au serveur qui agit comme un service qui tourne en fond.
Server = c'est lui qui gère et exécute les requêtes.
Un bloc serveur absent peut signifier que le service n'est pas lancé ou que les droits ne sont pas appropriés.

1.2 : 
Dans "-p 8080:80" le port 8080 est celui de mon PC et le port 80 celui du serveur.

1.3 :
L'image c'est le système d'exploitation utilisé par un conteneur et elle est indépendante de ce dernier.

Défi 1 :
Il est impossible d'avoir deux fois le même port actif sur un équipement.

## Palier 2
2.2 :
Non, "EXPOSE 8080" ne suffit pas car il indique uniquement le port du conteneur vers lequel il écoute et pas le port côté client.

2.3 :
Le "." sert à indiquer le chemin que doit utiliser le daemon docker pour générer l'image (le dockerfile entre autre).
Ici il s'agit du port 8080 et non pas juste 80 car lors de la génération de l'image nous avons indiqué que le port d'écoute serait le 8080.

2.4 :
Après le build la page affichait toujours 1.0 car l'image d'un conteneur n'est utilisé que lors de sa création et sa modification n'impact pas les conteneurs déjà lancés qui l'utilisent.
Pour passer en 2.0 j'ai éteint le conteneur, supprimé le conteneur et créé un autre conteneur avec la nouvelle image.
Docker stop petitpanier
docker rm petitpanier
docker run --name petitpanier -d -p 8080:8080 petitpanier:2.0 .

Défi 2 : 
La seule étape en cached c'est le copy index.html car le fichier n'a pas été modifié depuis le dernier build. Le build a donc pris une à deux secondes.

## Palier 3
3.2 : 
UP = Le service est en cours de démarrage.
UP Healthy = Le sevice est démarré et prêt.
C'est la section "healthcheck" du fichier compose qui produit ça.

3.3 :
La seconde commande échoue car je n'ai pas lancé le conteneur avec apache depuis localhost mais depuis le docker compose, de ce fait il est dans un bridge isolé.
"app" c'est le nom dns du container nginx, on aurait pu mettre l'IP ça aurait fonctionner aussi.

Défi 3 :
La commande affiche bien la page index.html ce qui prouve que le container associé à "stats" arrive bien à joindre celui associé à "app".

## Palier 4
4.1 :
"kind load" est nécessaire car sans ça kube ne sait pas si j'ai des images et où elles sont, je les upload donc dans mon cluster.
En entreprise on utilise des dépôts d'images plutôt que d'envoyer les images à la main comme ça kube peut aller les chercher tout seul.

4.2 :
Les labels "app:" sont comme des groupes pour que kube sache quelles actions faire sur quels pods.
Si le selector était différent soit il n'existe pas dans le fichier deployment donc il ne fait rien avec soit il existe pour un autre pod et dirige les flux au mauvais endroit.

4.3 :
L'adresse dans endpoints correspond à celle du conteneur lancé par kube.
Nous avons besoin d'un port forwarding car mon PC est externe au réseaux interne de kube, par conséquent il faut que je lui dise de transmettre mes demande sur le port 8080 vers petitpanier sur son propre port 8080.

Défi 4 :
je n'ai rien dans la section Events "<none>"

## Palier 5

5.1 :
Non, je n'ai actuellement qu'un seul noeud "petitpanier" et ils tournent tous dessus.
On ne peut pas parler de HA car les pods sont sur un seul noeud, si le noeud tombe, tous les pods tombent. 

5.2 :
C'est mon noeud "petitpanier" qui a recréé un pod car on lui a dit qu'il fallait 3 replicas, lorsqu'on en a supprimé un il s'est rendu compte qu'il n'y en avait plus que deux alors il en a recréé un autre avec un ID différent.

5.3 :
"Web" correspond au nom du conteneur renseigné dans le fichier "deployment" et le rollout history affiche 2 révisions.

Défi 5 :
Il vaut mieux modifier le fichier plutôt que de faire un scale car c'est une bonne pratique permettant l'uniformité du système.

## Palier 6 — tableau de diagnostic

| Panne | Symptôme | Hypothèse | Commande de preuve et résultat | Correction
|
|---|---|---|---|---|
| promo-1 | ImagePullBackOff | L'image n'est pas sur le noeud ni sur un depot distant | Commande "kubectl describe pod promo... --> Dans le section Event l'erreur est la suivante "Back-off pulling image "petitpanier:1.0"" | |
| promo-2 | | | | |
| promo-3 | | | | |
6.2 (--previous) :
6.3 (pourquoi Running mais inaccessible) :
## Palier 7
| Étape | Commande | Si échec |
|---|---|---|
| build | | |
| test | | |
| publish | | |
| deploy | | |
7.1 :
Workflow n8n :
## Défis réalisés

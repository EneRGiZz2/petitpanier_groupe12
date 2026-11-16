# Compte rendu TP PetitPanier — ZAK / OSSIBI / BOUZIANE / BEN-AKOUCHE / HAMOUTEN
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

| promo-1 | ImagePullBackOff avec la commande get pod | L'image n'est pas sur le noeud ni sur un depot distant | Commande "kubectl describe pod promo... --> Dans le section Event l'erreur est la suivante "Back-off pulling image "petitpanier:1.0"" | kubectl set image deployment/promo-1 web=petitpanier:1.0 |

| promo-2 | CrashLoopBackOff/error avec la commande get pod 6 fois en 8min| | commande de preuve : kubectl describe pod promo-2-5fb657dc47-chgk6 dans la section Event : Beckoff x70 18 min backoff restarting failed container web in pod | Ajout de la variable PROMO_CODE "kubectl set env deployment/promo-2 PROMO_CODE=PROMO10" puis "kubectl patch deployment promo-2 --type=json -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/command"}]'" pour le mettre à jour sans passer par le .yaml puis kubectl get pod running pour le promo-2|

| promo-3 | This site can't be reached + timeout du port forwarding| problème réseau entre ma machine et le site web sur le pod | Tout est OK sauf kubectl get endpoints qui renvoi <none> de ce fait le pod ne connait aucun réseau et je ne peux pas joindre sa page web| kubectl patch service promo-3 --type=json -p='[{"op": "replace", "path": "/spec/selector/app", "value": "promo-3"}]', il y a maintenant deux endpoints pour les deux replicas et le site web est joignable


## Palier 7
| Étape | Commande | Si échec |

| build | docker build -t petitpanier:1.0 . | L'image n'est pas construite et le pipeline s'arrête.

| test | kubectl apply -f pannes/panne-2.yaml | Le manifeste présente une erreur de syntaxe et bloque le deploiement

| publish | kind load docker-image petitpanier:1.0 | L'image n'est pas chargée dans le pod, il générera une erreur en boucle ne connaissant aucune image pour démarrer

| deploy | kubectl set image deployment/promo-1 web=petitpanier:1.0 | Le cluster ne prend pas en comtpe la MAJ et reste avec une version antérieure "fonctionnelle".

7.1 :
Une pipeline tague l'image avec un identifiant de commit plutôt qu'avec latest car cela permet une meilleure traçabilité sur l'image exacte utilisée que ce soit pour l'admin ou pour kube pour lui permettre de savoir s'il doit télécharger une nouvelle image ou si l'ancienne est toujours d'actualité.

7.2 :
Noeud 1 : Déclencheur (Webhook GitLab)
--> Réceptionne la notification envoyée par le pipeline GitLab dès qu'un job ou un déploiement se termine.

Noeud 2 : Optionnel / Filtrage (Code)
--> Extrait proprement le statut du déploiement depuis le JSON reçu.

Noeud 3 : Condition (If)
--> Vérifie si le statut = en échec (statut == failed). Si c'est true, le workflow continue vers l'alerte, si c'est false, il s'arrête.

Noeud 4 : Action (Slack)
--> Envoie un message d'alerte textuel formaté sur le canal #petitpanier-ops pour prévenir l'équipe technique de l'échec du déploiement.

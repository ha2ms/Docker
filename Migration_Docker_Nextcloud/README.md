
# Paramétrage de NextCloud
Ce dossier comprends l'ensemble des éléments nécessaire à la configuration de NextCloud pour un usage collaboratif entre plusieurs utilisateurs.

> Nous y verrons les éléments suivants:
> - Première configuration de Nextcloud
> - Ajouts de nouveaux utilisateurs
> - Définissions de leurs droits et limitations
> - Création d'un nouveau répertoire de Partage
> - Ajout des utilisateurs au Dossier partagé
> - Définissions des droits et limitations au Dossier
> ---
> 
## Première configuration de NextCloud
Nextcloud a été installé via son image docker pour permettre sa flexibilité multi plateforme, ainsi nous avons mis en place un fichier docker-compose complet, séparant Nextcloud de sa base de données et en partageant les données soumis à évolution comme les configurations de l'interface web ou encore les modifications faites dans la base données via un volume que les conteneurs partagerons avec notre hôte.

> La configuration du Docker-Compose comporte:
> 
> - La configuration du Network
> - La configuration du service db (Database)
> - La configuration du service app (NextCloud)
> 
> ## Configuration du Network 
> 
> Pour faciliter l'automatisation ainsi que les futures optimisations, nous avons préconfiguré un réseau statique propre aux conteneurs utilisé par NextCloud. Cela permet une meilleure prédictibilité ainsi qu'une plus grande sécurisation en isolant les conteneurs au stricte nécessaire.
> 
> ![](https://i.postimg.cc/VLk4GJ9J/network.png) 
> 
> ---
> 
> ## Configuration du service DB (Database)
>  
>  Le service DB se repose sur une image MariaDB auquel on vient ajouter les configuration supplémentaires nécessaires non seulement au fonctionnement initial de NextCloud mais également à sa réplication parfaite sur différents environnement sans perdre aucune données lors de migrations.
>  Pour se faire nous avons mis en place un volume partagé qui entre l'hôte et le conteneur permettant la mise à jour instantanée du contenu de la base de donnée et de sa configuration.
>  
>   ![](https://i.postimg.cc/1z5jwx6L/db.png)
> 
> ---
> ## Configuration du service App (Nextcloud)
>  
>  Le service App lui repose sur l'image de NextCloud, auquel on ajoute également les variables d'environnement liées à notre base de données lui permettant de s'y connecter.
>  L'on prend également la peine d'ajouter le paramètre "- depends_on: db" lui spécifiant que le service ne doit démarrer qu'après la mise en route de la base de données, cela permet d'éviter son plantage si le service tente de l'atteindre avant qu'elle ne soit disponible.
>  L'on partage aussi le contenu du site web via un volume partagé permettant de récupérer systématiquement toutes les modifications apporté en sein lors de ces différentes configurations.
>  
>   ![](https://i.postimg.cc/Qdj4ZL6H/app.png)
>    
>    ---
>    ## Sécurisation
>    Comme on peut le remarquer les données sensibles ne sont pas directement accessible depuis le fichier docker-compose.yml, il sont stocké dans un fichier .env servant de variables d'environnement pour nos conteneurs.
>    ![](https://i.postimg.cc/fytLbgBd/env.png)
>    
>    ### Cela permet 2 choses:
>    - **Sécurisation de données sensibles**
>      - L'accès aux données sensibles n'étant pas directement possible via le docker-compose, sa mise en disponibilité sur le réseau d'une entreprise afin que plusieurs employées travail dessus en collaboration devient plus simple. En effet même si l'entreprise subissait une attaque réseau de type **"ManInTheMiddle"** pour intercepter le flux de données et donc le fichier disponible en réseau, il n'aura aucunement accès aux données sensibles de type identifiant / mot de passe car le fichier .env devra toujours rester en local.
>      De plus le fichier .env peut à son tour être chiffré par clé GPG ou encore via Vault afin d'accroitre davantage la sécurité de nos précieuses données.
>   - **Migration simplifiée et automatisée**
>     - Le fait de normer ses variables d'environnements permet également de simplifier et d'automatiser la migration sur différents environnements sans nécessitez de modifier manuellement les identifiants du docker-compose, si un environnement à un des identifiants différents d'un autre environnement, le fichier docker-compose s'adaptera automatiquement aux valeurs qu'elle recevra par le fichier .env et ainsi permet d'accélérer le processus de déploiement de manière significatif.
>      
>      ---
>    ## Composition du répertoire de Dockerisation
>    ### Affichage des conteneurs
>    Une fois les conteneurs lancées (docker compose up -d) l'on observe son état de fonctionnement:
>    
>    ![](https://i.postimg.cc/ZqZpGsBK/docker-up.png)
>    
>    ---
>    ### Affichage des répertoires (Volumes partagés)
>    On observe également si les répertoires (volumes partagés) ont bien réussi à récupérer les données des conteneurs correspondants:
>    
>    ![](https://i.postimg.cc/zB3C7dPt/ls-db-nextcloud-full.png)
>    
>    Maintenant, chaque nouvelles modifications apportées à la base de données ou à l'interface web sera systématiquement exporté vers ses 2 répertoires au sein de notre répertoire hôte de conteneurisation, facilitant ainsi sa réplication sur d'autres environnement par la suite.
>    
>   ---

## Mesures de précautions 
> ### Config.php
> Lorsque vous migrez une interface web vers un autre environnement, il y'a toujours un fichier dont il faut porter une attention particulière.
> En général (pour les sites PHP), il s'agit du fichier [ config.php ]
> Si nous ouvrons ce fichier nous voyons ceci:
> 
> ![](https://i.postimg.cc/mkcph3xq/screen1.png)
>   
>  ---
>  Comme vous pouvez le voir l'adresse IP n'est pas celle du conteneur mais bien celui de l'hôte, cela signifie qu'au moment de la migration vers un autre serveur, NextCloud empêchera le lancement de son interface Web car l'IP du nouvel hôte ne fera pas parti des hôtes de confiances de configuré dans NextCloud.
>  
>   ![](https://i.postimg.cc/kgPYXtT1/screen2.png)
>    
>   ---
>   Cela signifie qu'il faudrait manuellement reconfigurer le fichier [ config.php ] à chaque fois que celui-ci soit transféré d'un serveur à un autre ou encore à chaque fois que l'hôte change d'IP. Cela pourrait devenir vite pénible dans un environnement de Dev changeant régulièrement.
>   Une solution à cela serait d'automatiser directement au sein du fichier config le bon hôte en fonction de son environnement. C'est possible en récupérant l'hôte actuel dans une variable via la ligne : " $domain = $_SERVER['HTTP_HOST']; ", et en remplaçant les hôtes statiques par la variable contenant l'hôte actuel.
>   
>   ![](https://i.postimg.cc/c1QXkJxf/screen3.png)
>   
> ---
> Cette solution reste toutefois à privilégier en phase de développement où les adresses IP peuvent évoluer continuellement, il sera préférable de fixer cette configuration une fois le projet prêt pour le développement pour des raisons de sécurité évidentes. Ou simplement d'attribuer une configuration par Nom de Domaine pour simplifier le paramétrage à venir.
> 
> ---

## Premiers pas avec l'Interface Graphique
 La configuration initiale de NextCloud est rudimentaire, il ne s'agira là que de définir le compte Administrateur de la plateforme ainsi que des outils supplémentaires que l'on souhaite installer.
 > ![](https://i.postimg.cc/QCfVzs9c/admin-install-first-page.png)
 > 
 > ---
 > ![](https://i.postimg.cc/qv3Ms2Ls/appli-recommande.png)
 > 
 > ---
 > 
 ## Créations de nouveaux Utilisateurs
 > Nous voilà maintenant sur l'interface principale de NextCloud, pour ajouter ou modifier un utilisateur, cliquer sur l’icône ronde correspondante en haut à droite, puis sélectionner l'élément [Utilisateurs]
 > 
 > ![](https://i.postimg.cc/pXDPyPXC/screen-01.png)
 > 
 > ---
 >  
 > Ensuite, cliquez sur [ + Nouvel Utilisateur ] en haut à gauche de l'interface. 
 > 
 > ![](https://i.postimg.cc/DyD1fd0L/screen-02.png)
 >  
 >  ---
 >  Définissez votre utilisateur et ses différents droits/limitations.
 >  
 >  ![](https://i.postimg.cc/Jh4L0JGc/screen-03.png)
 >  
 >  ---
 >   Faites de même avec tous les autres utilisateurs:
 >   
 >   ![](https://i.postimg.cc/zXvjpG4D/screen-04.png)
 >    
 >    ---
 
 ## Création d'un répertoire Partagé
 >
 > Retournons sur l'interface principal et sélectionnez l'élément [ Fichier ] en haut à gauche 
 > 
 > ![](https://i.postimg.cc/x84ZfxcB/screen-01.png)
 > 
 > ---
 >  Choisissez ensuite [ Nouveau ], puis [ Nouveau dossier ]
 >  
 >  ![](https://i.postimg.cc/4NRL59p6/screen-02.png)
 >   
 >   ---
 >    Nommez le Dossier
 >    
 >    ![](https://i.postimg.cc/TPKJGdr2/screen-03.png)
 >     
 >  ---
 >   Une fois le Dossier créé, cliquez l'icône à sa droite pour ajouter les utilisateurs pouvant y accéder
 >   
 >   ![](https://i.postimg.cc/sD1XTMy2/screen-04.png)
 >    
 >    ---
 > Dans l'onglet partage vous pouvez si vous le souhaitez ajouter directement les utilisateurs pouvant accéder au Répertoire.
 > 
 >    ![](https://i.postimg.cc/g04b9KHb/screen-05.png)  
 >   
 >   ---
 >   Une fois l'utilisateur sélectionné vous avez là possibilité de définir ses différents droits/limitations. 
 >   
 >    ![](https://i.postimg.cc/JhCjwN4x/screen-05-1.png)    
 >     
 >    ---
 >    Ou sinon vous pouvez également créer des liens d'accès au Dossier Partagé avec différents droits/limitations pour chaque liens générés 
 >    
 >    ![](https://i.postimg.cc/5NMH24M9/screen-06.png)
 >   
 >   ---
 >   En ce qui concerne les liens d'accès au partage vous avez également la possibilité de définir des règles de sécurités supplémentaire comme l'accès du répertoire par mot de passe et/ou définir également une date d'expiration du lien.
 >   
 >   ![](https://i.postimg.cc/8z6nBm0m/screen-07.png)
 >    
 >   ---
 >   Vous pouvez dès à présent partagez vos différents entre utilisateurs 
 >   
 >   ![](https://i.postimg.cc/Y9hX0NYF/screen-08.png)
 >    
 >   ---

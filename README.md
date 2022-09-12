# The Cadors company website
L'objetif ici, est d'héberger un site web sur une ec2 derriére un équilibreur de charge, tout cela hébergé dans un VPC sécurisé sur plusieurs Zones de disponibilités et divisées en sous-réseaux pertinents : sous-réseaux publics, sous-réseaux privés, sous-réseaux de données,...etc

Pour mettre tout ca en place, l'image ci-dessous montre la conception architecturale du modèle CloudFormation.

  ![diagram project](https://github.com/SunlogCloud/hachem/blob/master/The%20cadors_diag.jpg) 

 
### Étape 1 : Création des modèles CloudFormation
Pour faire cette tâche, j'ai creé donc **sept modéles** comme suit : 

  ![Stacks project](https://github.com/SunlogCloud/hachem/blob/master/Stacks%20CF.PNG)

> **NB:** Vous trouverez les codes yaml de ces modéles "commenter" dans le dossier **CF_templates**.

* Un modéle __"vpc"__  qui consiste à :
    *   Créer un VPC 
    *   Créer une Internet Gateway
    *   Attacher l'Internet Gateway au VPC
    *   Créer des sous-réseaux publics
    *   Créer une table de routage publique
    *   Ajouter une route publique à la table de routage publique
    *   Associer les sous-réseaux publics à la table de routage publique
    *   Créer des sous-réseaux privés
    *   Créer des groupes de sécurité
     
     ![diagram vpc](https://github.com/SunlogCloud/hachem/blob/master/diag%20vpc.PNG) 


* Un modéle __"nat-gateway"__ qui consiste à :
    * Attribuer une adresse IP élastique 
    * Créer une Nat-Gateway dans chaque sous-réseau public
    * Créer une table de routage privée
    * Ajouter une route pour pointer le trafic lié à Internet vers Nat-Gateway
    * Associer les sous-réseaux privés à une table de routage privée
     
      ![diagram natgateway](https://github.com/SunlogCloud/hachem/blob/master/Diag%20NatGatway.PNG) 


* Un modéle __"Acm"__ pour automatiser la validation du certificat SSL ACM à l’aide du service DNS AWS Route53.

  Dès que nous essayons de créer la pile à partir de ce modèle, nous verrons immédiatement que dans la section __"Outputs"__ de notre pile, il existe un champ "Value", qui contient l'ARN de notre certificat, qu'on peut aussi visibement le trouver dans le service aws certificate manager :   
 
  
  ![Acm ](https://github.com/SunlogCloud/hachem/blob/master/acm.PNG)

Ce ARN nous sera utile pour créer notre prochain modéle "alb".

* Un modéle __"alb"__ pour créer un loadbalancer avec deux instances.
  
  L’utilisation d’un équilibreur de charge ici, permet d’acheminer le trafic vers nos deux serveurs principaux. En cas de défaillance d’un serveur, le trafic sera acheminé vers l'autre serveur. 

  le modéle "alb" consiste donc à : 

    * Créer un équilibreur de charge d'application
    * Créer un écouteur sur le port 80 et rediriger le trafic vers 443
    * Créer un écouteur sur le port 443
    * Créer un Target group
    * Créer deux Ec2 avec une ami dans la région europ-eu3

      ![Listener alb](https://github.com/SunlogCloud/hachem/blob/master/Listeners.PNG)

Étant donné que les ports 80 et 443 sont ouverts sur le groupe de sécurité de l’équilibreur de charge " voir le modéle 'vpc' " , le modèle 'alb' ici, crée les deux écouteurs pour l’équilibreur de charge. Un écouteur sur le port 80 (HTTP) et l’autre sur le port 443 (HTTPS). Ensuite, pour l’écouteur HTTP, le modèle est configuré pour créer une action par défaut pour transférer toutes les demandes à l’écouteur HTTPS par défaut.

Pour l’écouteur HTTPS, j'ai inclus un certificat qui a été généré pour notre nom de domaine. L’utilisateur doit remplacer l’ARN du certificat par son propre ID ARN de certificat.




* Un modéle __route53__ pour configurer un jeu d’enregistrements de ressources d’alias nommé qui achemine le trafic vers l'équilibreur de charge ALB. La propriété "AliasTarget" spécifie l’ID de la zone hébergée et le nom DNS de l'ALB à l’aide de la fonction Fn::ImoprtValue qui récupère ces différentes propriétés de la ressource ALB.
  
   ![route53](https://github.com/SunlogCloud/hachem/blob/master/route53screen.PNG)


  

* Un autoscaling groupe
* Un RDS

##### En cours de rédaction ...
### Étape 2 : Créer CodePipeline pour déployer le modèle
##### En cours de rédaction ...
  ![Pipeline project](https://github.com/Hachemzeine/website_project/blob/main/Pipeline.PNG)
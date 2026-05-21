# Env-Hautement-disponible-AWS-
### création d'un environnement hautement disponible
Présentation de l'atelier et des objectifs
Les systèmes métier critiques doivent être déployés en tant qu'applications hautement disponibles. Cela signifie que les applications restent opérationnelles, même en cas de défaillance de certains composants. Pour garantir une haute disponibilité dans Amazon Web Services (AWS), vous allez exécuter les services dans plusieurs zones de disponibilité.

De nombreux services AWS sont intrinsèquement hautement disponibles, notamment les équilibreurs de charge. Vous pouvez également configurer de nombreux autres services AWS pour qu'ils soient hautement disponibles, tels que le déploiement d'instances Amazon Elastic Compute Cloud (Amazon EC2) dans plusieurs zones de disponibilité.

Au cours de cet atelier, vous allez commencer avec une application qui s'exécute sur une seule instance EC2, puis la rendre hautement disponible.
À la fin de cet atelier, vous serez en mesure d'effectuer les tâches suivantes :
inspecter un cloud privé virtuel (VPC) fourni ;
créer un Application Load Balancer ;
créer un groupe Auto Scaling ;
tester la haute disponibilité de l'application.
À la fin de cet atelier, votre architecture ressemblera à l'exemple suivant :


<----------------->


<img width="816" height="455" alt="image" src="https://github.com/user-attachments/assets/afe719f6-faed-4a3e-9446-1b2c98d9b00f" />


<----------------->


# Tâche 1 : inspection de votre VPC
Cet atelier commence dans un environnement déjà déployé à l'aide d'AWS CloudFormation. Cet environnement inclut les éléments suivants :
un VPC ;
des sous-réseaux publics et privés dans deux zones de disponibilité ;
une passerelle Internet associée aux sous-réseaux publics ;
une passerelle NAT dans l'un des sous-réseaux publics ;
une instance Amazon Relational Database Service (Amazon RDS) dans l'un des sous-réseaux privés.


<img width="817" height="470" alt="image" src="https://github.com/user-attachments/assets/f44ee4f9-0ec3-4f2b-aac6-80ac4687293e" />


<-------------->



Au cours de cette tâche, vous allez passer en revue la configuration du VPC déjà créé pour cet atelier.
Dans la barre de recherche de la console de gestion AWS, saisissez et sélectionnez VPC pour ouvrir la console Amazon VPC.
Dans le volet de navigation de gauche, dans la liste déroulante Filtrer par VPC, choisissez Lab VPC (VPC de l'atelier).
Ce paramètre restreint la console à l'affichage des ressources qui sont associées au VPC de l'atelier.
Dans le volet de navigation de gauche, choisissez Vos VPC.
Ici, vous pouvez accéder aux informations du VPC de l'atelier qui a été créé pour vous. 
Sélectionnez Lab VPC (VPC de l'atelier).
Dans l'onglet Détails, notez que le champ IPv4 CIDR (CIDR IPv4) contient la valeur 10.0.0.0/16, ce qui signifie que ce VPC inclut toutes les adresses IP qui commencent par 10.0.x.x.
Dans le volet de navigation de gauche, choisissez Sous-réseaux.
Ici, vous pouvez accéder aux informations du sous-réseau public 1. Dans la liste des sous-réseaux, notez ce qui suit :
La colonne VPC du sous-réseau public 1 montre que ce sous-réseau existe dans Lab VPC (VPC de l'atelier).
La colonne IPv4 CIDR (CIDR IPv4) contient la valeur 10.0.0.0/24, ce qui signifie que ce sous-réseau inclut les 256 adresses IP comprises entre 10.0.0.0 et 10.0.0.255. Cinq de ces adresses sont réservées et inutilisables.
Dans la colonne Zone de disponibilité, vous pouvez voir la zone de disponibilité dans laquelle se trouve ce sous-réseau.
Pour plus de détails, sélectionnez Sous-réseau public 1.
Conseil : pour régler la taille du volet inférieur, faites glisser le séparateur.
Dans la moitié inférieure de la page, sélectionnez l'onglet Table de routage.
Cet onglet répertorie les détails du routage de ce sous-réseau :
La première entrée indique que le trafic destiné à la plage CIDR du VPC (10.0.0.0/16) est acheminé au sein du VPC (local).
La deuxième entrée spécifie que le trafic destiné à Internet (0.0.0.0/0) est acheminé vers la passerelle internet (igw-) du VPC de l'atelier. Ce paramétrage en fait un sous-réseau public.
Cliquez sur l'onglet Liste ACL réseau.
Cet onglet présente des informations sur la liste de contrôle d'accès au réseau (ACL réseau) associée au sous-réseau. Les règles sont actuellement définies par défaut. Elles autorisent tout le trafic à entrer dans le sous-réseau et à en sortir. Cependant, les règles peuvent être restreintes à l'aide de groupes de sécurité.
Dans le volet de navigation de gauche, sélectionnez Passerelles Internet.
Notez qu'une passerelle Internet portant le nom Lab IG est déjà attachée à Lab VPC (VPC de l'atelier).
Dans le volet de navigation de gauche, sélectionnez Groupes de sécurité.

Sélectionnez Inventory-DB.
Ce groupe de sécurité sert à contrôler le trafic entrant à destination de la base de données.

Dans la moitié inférieure de la page, sélectionnez l'onglet Règles entrantes.
La règle définie dans ce groupe de sécurité autorise le trafic MySQL ou Aurora entrant (port 3306) depuis n'importe où dans le VPC (10.0.0.0/16). Vous modifierez ultérieurement ce paramètre pour qu'il n'accepte que le trafic provenant des serveurs d'application.

Sélectionnez l'onglet Règles sortantes.
Par défaut, les groupes de sécurité autorisent la totalité du trafic sortant. Vous pouvez toutefois modifier ces paramètres selon vos besoins.


<img width="1118" height="330" alt="image" src="https://github.com/user-attachments/assets/165b53a6-00f0-4ec9-b3fa-6a6db37029fd" />



<------------------->


<img width="1106" height="144" alt="image" src="https://github.com/user-attachments/assets/68bde7cd-59c6-4edb-923f-313ab2b02b6c" />


<----------------->



<img width="1522" height="505" alt="image" src="https://github.com/user-attachments/assets/41025b15-ca68-4a5a-ad7e-2d6480c2fa33" />



<----------------->


<img width="1514" height="267" alt="image" src="https://github.com/user-attachments/assets/0d0843ac-9ec1-46f1-9ee2-5b6cdbe89603" />


<----------------->


<img width="1514" height="267" alt="image" src="https://github.com/user-attachments/assets/4fba3667-54fa-4d57-9b57-adfee4117dab" />



<----------------->


<img width="1011" height="216" alt="image" src="https://github.com/user-attachments/assets/6a410ea0-628f-4896-8cbe-a7797315032e" />



<--------------->



<img width="1203" height="140" alt="image" src="https://github.com/user-attachments/assets/ad9356ae-1e38-48d7-a1a7-f8c6f557bff7" />


<---------------->

Dans le volet de navigation de gauche, sélectionnez Groupes de sécurité.
Sélectionnez Inventory-DB.


<img width="1104" height="181" alt="image" src="https://github.com/user-attachments/assets/7f4015b0-9bf2-46f7-8175-9a114de6ce2e" />


<------------------>



<img width="1496" height="402" alt="image" src="https://github.com/user-attachments/assets/bfa2440a-9953-4502-b316-8519ac605892" />



# Tâche 2 : création d'un Application Load Balancer

Pour concevoir une application hautement disponible, il est recommandé de lancer les ressources dans plusieurs zones de disponibilité. Les zones de disponibilité sont des centres de données (ou des groupes de centres de données) physiquement séparés dans la même région. Si vous exécutez vos applications dans plusieurs zones de disponibilité, vous bénéficiez d'une plus grande disponibilité en cas de défaillance d'un centre de données.

Comme l'application s'exécute sur plusieurs serveurs d'application, vous devez trouver un moyen de répartir le trafic entre ces serveurs. Pour cela, vous pouvez utiliser un équilibreur de charge. Cet équilibreur de charge surveille également l'état des instances et n'envoie des demandes qu'aux instances saines.

<img width="819" height="487" alt="image" src="https://github.com/user-attachments/assets/1a5e1f13-4a89-45c2-bf50-95a7012c3a28" />


<------------------->

Dans la barre de recherche de la console de gestion AWS, saisissez et sélectionnez EC2 pour ouvrir la console Amazon EC2.
Dans le volet de navigation de gauche, sélectionnez Équilibreurs de charge. (Vous devez peut-être faire défiler l'écran vers le bas voir cette option.)
Choisissez Créer un équilibreur de charge.
Plusieurs types d'équilibreurs de charge s'affichent. Lisez les descriptions de chacun d'eux pour comprendre leurs capacités.
Pour Application Load Balancer, choisissez Créer.
Dans la section Basic Configuration (Configuration de base), pour Nom de l'équilibreur de charge, saisissez Inventory-LB.
Dans la section Mappage réseau, configurez les options suivantes :
Pour VPC, sélectionnez Lab VPC (VPC de l'atelier).
Important : veillez à sélectionner Lab VPC (VPC de l'atelier). Ce n'est probablement pas l'option sélectionnée par défaut.
À présent, vous allez indiquer les sous-réseaux que l'équilibreur de charge doit utiliser. Il s'agit d'un équilibreur de charge public. Par conséquent, sélectionnez les deux sous-réseaux publics.

Pour Mappings (Mappages), sélectionnez la première zone de disponibilité, puis le sous-réseau public qui s'affiche.
Pour Mappings (Mappages), sélectionnez la deuxième zone de disponibilité, puis le sous-réseau public qui s'affiche.
Vous devriez maintenant avoir deux sous-réseaux : Sous-réseau public 1 et Sous-réseau public 2. Si ce n'est pas le cas, revenez en arrière et réessayez la configuration.
Dans la section Groupes de sécurité, sélectionnez l'hyperlien Créer un groupe de sécurité. Ce lien ouvre un nouvel onglet de navigateur. 
Sur la page Créer un groupe de sécurité, dans la section Détails de base, configurez les options suivantes pour créer le groupe de sécurité :

Nom du groupe de sécurité : saisissez Inventory-LB.
Description : saisissez Enable web access to load balancer.
VPC : sélectionnez Lab VPC (VPC de l'atelier) dans la liste déroulante.
Dans la section Règles entrantes, sélectionnez Ajouter une règle et configurez les options suivantes :
Type : HTTP
Source : Anywhere-IPv4
Pour Règles entrantes, choisissez de nouveau Ajouter une règle et configurez les options suivantes :
Type : HTTPS

Source : Anywhere-IPv4
Sélectionnez Créer un groupe de sécurité.
Vous allez ensuite assigner le groupe de sécurité à l'équilibreur de charge.
Revenez à l'onglet du navigateur dans lequel vous êtes toujours en train de configurer l'équilibreur de charge et configurez les options suivantes :
Dans la section Groupes de sécurité, cliquez sur l'icône d'actualisation .
Dans la liste déroulante Groupes de sécurité, sélectionnez le groupe de sécurité Inventory-LB que vous venez de créer.
Sélectionnez ensuite le X du groupe de sécurité par défaut afin qu'Inventory-LB soit le seul groupe de sécurité choisi.
Dans la section Écouteurs et routage, sélectionnez le lien Créer un groupe cible.
Un nouvel onglet de navigateur s'ouvre. 

Analyse : les groupes cibles définissent où le trafic entrant dans l'équilibreur de charge doit être envoyé. L'Application Load Balancer peut envoyer le trafic vers plusieurs groupes cibles en fonction de l'URL de la demande entrante. Par exemple, il peut envoyer les demandes provenant d'applications mobiles vers un autre ensemble de serveurs. Votre application web utilise un seul groupe cible.
Pour l'étape 1 : Specify group details (Spécifier les détails du groupe), configurez les options ci-dessous.
Dans la section Basic configuration (Configuration de base), configurez les options suivantes :
Choose a target type (Choisir un type de cible) : sélectionnez Instances.
Target group name (Nom du groupe cible) : saisissez Inventory-App.
VPC : vérifiez que l'option Lab VPC (VPC de l'atelier) est sélectionnée.
Dans la section Vérifications de l'état, développez  Advanced health check settings (Paramètres avancés de vérification de l'état) et configurez les options suivantes :
Remarque : l'Application Load Balancer réalise automatiquement des vérifications de l'état pour toutes les instances, afin de s'assurer qu'elles répondent aux demandes. Il est recommandé de conserver les paramètres par défaut, mais vous allez les rendre légèrement plus rapides dans le cadre de cet atelier.
Healthy threshold (Seuil de bonne santé) : saisissez 2.
Interval (Intervalle) : saisissez 10 (secondes).
Les configurations que vous avez choisies entraînent l'exécution d'une vérification de l'état toutes les 10 secondes. Si l'instance répond correctement deux fois de suite, elle est considérée comme saine.

Choisissez Suivant. 
L'étape 2 : Register targets (Enregistrement des cibles) apparaît.
Remarque : les cibles sont les instances individuelles qui répondent aux demandes de l'équilibreur de charge.
Vous ne disposez pas encore d'instances d'application web, vous pouvez donc passer cette étape.
Passez en revue les paramètres et sélectionnez Create target group (Créer un groupe cible).
Le message Successfully created the target group (Groupe cible créé avec succès) s'affiche.
Revenez dans l'onglet du navigateur où vous avez commencé à configurer l'équilibreur de charge.
Dans la section Écouteurs et routage, cliquez sur l'icône d'actualisation .
Dans la liste déroulante Default action (Action par défaut), choisissez le groupe cible Inventory-App que vous venez de créer.
Faites défiler l'écran jusqu'au bas de la page, puis sélectionnez Créer un équilibreur de charge.
L'équilibreur de charge a été créé avec succès.
Sélectionnez View load balancer (Afficher l'équilibreur de charge).



<img width="915" height="369" alt="image" src="https://github.com/user-attachments/assets/1da50020-4203-45b8-982e-fed72aa09e4e" />


<------------------>



<img width="1288" height="433" alt="image" src="https://github.com/user-attachments/assets/08cc4798-c9c3-4f38-bc70-8ef1aba97d23" />



<------------------>




<img width="1001" height="607" alt="image" src="https://github.com/user-attachments/assets/48251042-9fa0-4bab-8de4-3519f0a2f85f" />



<----------------->




<img width="939" height="236" alt="image" src="https://github.com/user-attachments/assets/5c6edd24-d91b-4a13-b021-9bea30cd327c" />



<------------------------->




<img width="821" height="206" alt="image" src="https://github.com/user-attachments/assets/1f87b6d5-8dcd-4515-9e94-cd4a1ef00210" />




<-------------------------->



# Tâche 4 : mise à jour des groupes de sécurité
L'application que vous avez déployée possède une architecture à trois niveaux. Vous allez maintenant configurer les groupes de sécurité pour appliquer ces niveaux :



<img width="526" height="207" alt="image" src="https://github.com/user-attachments/assets/6bf40a07-cb59-4d80-95ae-936669f3e816" />


<------------------>


Tâche 4.1 : configuration du groupe de sécurité de l'équilibreur de charge
Vous avez déjà configuré le groupe de sécurité de l'équilibreur de charge lorsque vous avez créé ce dernier. Il accepte tout le trafic HTTP et HTTPS entrant.

L'équilibreur de charge a été configuré pour transférer les demandes entrantes à un groupe cible. Quand Auto Scaling lance de nouvelles instances, il ajoute automatiquement ces instances au groupe cible.


 

# Tâche 4.2 : configuration du groupe de sécurité de l'application
Le groupe de sécurité de l'application a été fourni dans le cadre de la configuration de l'atelier. Vous allez maintenant le configurer pour qu'il accepte uniquement le trafic entrant de l'équilibreur de charge.

Dans le volet de navigation de gauche, sélectionnez Groupes de sécurité.

Sélectionnez Inventory-App.

Sélectionnez l'onglet Règles entrantes.

Le groupe de sécurité est actuellement vide. Vous allez maintenant ajouter une règle pour accepter le trafic HTTP entrant de l'équilibreur de charge. Il n'est pas nécessaire de configurer le trafic HTTPS, car l'équilibreur de charge a été configuré pour transférer les demandes HTTPS via HTTP. Cette pratique décharge la sécurité sur l'équilibreur de charge, ce qui réduit la somme de travail requise par les serveurs d'application individuels.

Sélectionnez Modifier les règles entrantes.
Sur la page Modifier les règles entrantes, sélectionnez Ajouter une règle et configurez les options suivantes :
Pour Type, choisissez HTTP.
Pour Port, saisissez 80.
Pour Source, configurez les options suivantes :
Sélectionnez la barre de recherche à droite de l'option Personnalisé.
Supprimez le contenu actuel.
Saisissez sg.
Dans la liste qui s'affiche, sélectionnez Inventory-LB.
Pour Description, saisissez Traffic from load balancer.
Choisissez Enregistrer les règles.
Les serveurs d'application peuvent désormais recevoir le trafic de l'équilibreur de charge. Cela inclut les vérifications de l'état que l'équilibreur de charge effectue automatiquement.




<img width="821" height="218" alt="image" src="https://github.com/user-attachments/assets/3ccda90b-c85a-47e7-8cbd-7f9d6ccb9e1c" />



<------------------------------------>



# Tâche 4.3 : configuration du groupe de sécurité de la base de données

Vous allez maintenant configurer le groupe de sécurité de la base de données pour qu'il n'accepte que le trafic entrant en provenance des serveurs d'application.
Dans la liste Groupes de sécurité, sélectionnez Inventory-DB et veillez à ce qu'aucun autre groupe de sécurité ne soit sélectionné.
La règle existante autorise le trafic sur le port 3306 (utilisé par MySQL) à partir de n'importe quelle adresse IP au sein du VPC. Il s'agit d'une bonne règle, mais la sécurité peut encore être renforcée.

Dans l'onglet Règles entrantes, sélectionnez Modifier les règles entrantes et configurez les options suivantes :
Pour supprimer la règle existante, sélectionnez Supprimer.
Sélectionnez Ajouter une règle.
Pour Type, sélectionnez MYSQL/Aurora.
Pour Source, configurez les options suivantes :
Sélectionnez la barre de recherche à droite de l'option Personnalisé.
Saisissez sg.
Dans la liste déroulante, sélectionnez Inventory-App.
Pour Description, saisissez Traffic from application servers.
Choisissez Enregistrer les règles.
Vous avez maintenant configuré la sécurité à trois niveaux. Chaque élément du niveau accepte le trafic du niveau supérieur uniquement.
En outre, l'utilisation de sous-réseaux privés signifie qu'il existe deux barrières de sécurité entre Internet et les ressources de votre application. Cette architecture correspond à la bonne pratique consistant à appliquer plusieurs couches de sécurité.




<img width="808" height="239" alt="image" src="https://github.com/user-attachments/assets/c9c0cefd-15c1-43bf-9a40-eafa5eeb2a9d" />




Vous avez maintenant configuré la sécurité à trois niveaux. Chaque élément du niveau accepte le trafic du niveau supérieur uniquement.
En outre, l'utilisation de sous-réseaux privés signifie qu'il existe deux barrières de sécurité entre Internet et les ressources de votre application. Cette architecture correspond à la bonne pratique consistant à appliquer plusieurs couches de sécurité.



<----------------->


# Tâche 5 : test de l'application

L'application est maintenant prête à être testée.
Au cours de cette tâche, vous allez vérifier que votre application web est en cours d'exécution. Vous allez également vérifier qu'elle est hautement disponible.
Dans le volet de navigation de gauche, sélectionnez Groupes cibles.
Sélectionnez Inventory-App.
Sélectionnez l'onglet Cibles.
Il doit répertorier deux cibles enregistrées. La colonne État de santé montre le résultat de la vérification de l'état de l'équilibreur de charge qui a été réalisée sur les instances.
Dans la zone Registered targets (Cibles enregistrées), cliquez sur l'icône d'actualisation  jusqu'à ce que l'état des deux instances soit Sain.
Si l'état ne devient pas Sain, demandez à votre formateur de vous aider à diagnostiquer la configuration.
Vous allez tester l'application en la connectant à l'équilibreur de charge, qui enverra ensuite votre demande à l'une des instances EC2. Vous allez d'abord extraire le nom DNS de l'équilibreur de charge.
Dans le volet de navigation de gauche, choisissez Équilibreurs de charge, puis sur Inventory-LB.
Dans l'onglet Détails situé dans la moitié inférieure de la fenêtre, copiez le nom DNS dans le presse-papiers.
Le nom doit ressembler à Inventory-LB-xxxx.elb.amazonaws.com.
Dans un nouvel onglet de navigateur web, collez le nom DNS du presse-papiers et appuyez sur Entrée.
L'équilibreur de charge a transféré votre demande à l'une des instances EC2. L'ID de l'instance et la zone de disponibilité sont visibles en bas de la page web.
Rechargez la page  dans le navigateur web. Vous devriez remarquer que l'ID de l'instance et la zone de disponibilité changent parfois entre deux instances.
Lorsque cette application web s'affiche, le flux de données sur le réseau est le suivant :




<img width="816" height="324" alt="image" src="https://github.com/user-attachments/assets/27d0e7a6-3836-470c-b814-890ea8502b5a" />


<--------------------->






<img width="942" height="287" alt="image" src="https://github.com/user-attachments/assets/afe3f4cb-e789-4ac3-96e8-0831d8e43439" />




<-------------------->




<img width="942" height="287" alt="image" src="https://github.com/user-attachments/assets/8531fdf0-7a36-4866-8650-8c19c147686c" />





<------------------>


Dans un nouvel onglet de navigateur web, collez le nom DNS du presse-papiers et appuyez sur Entrée.
L'équilibreur de charge a transféré votre demande à l'une des instances EC2. L'ID de l'instance et la zone de disponibilité sont visibles en bas de la page web.


<img width="800" height="324" alt="image" src="https://github.com/user-attachments/assets/3106a740-cc0b-4d2f-abd9-9c198de21d08" />




<------------------>


Rechargez la page  dans le navigateur web. Vous devriez remarquer que l'ID de l'instance et la zone de disponibilité changent parfois entre deux instances.



<img width="915" height="402" alt="image" src="https://github.com/user-attachments/assets/78836919-7c91-4c7c-9596-b006d7975438" />



<--------------------->



Changement de Zone 


<img width="843" height="406" alt="image" src="https://github.com/user-attachments/assets/547126e3-ea63-4725-bcc1-70f51be0e05c" />




<---------------------->



<img width="776" height="409" alt="image" src="https://github.com/user-attachments/assets/cfe51869-b860-4bdb-a8a0-8a17b504871b" />




<----------------->



<img width="698" height="368" alt="image" src="https://github.com/user-attachments/assets/ff96e9b5-1e21-4f90-861d-797703474e46" />



<----------------------->



# Tâche 6 : test de la haute disponibilité
Votre application est configurée pour être hautement disponible. Vous pouvez vérifier sa haute disponibilité en résiliant l'une des instances EC2.
Revenez dans l'onglet Console EC2 de votre navigateur Web. Ne fermez pas l'onglet de l'application web. Vous y reviendrez bientôt.
Dans le volet de navigation de gauche, choisissez Instances.
Vous allez maintenant résilier l'une des instances d'application web pour simuler une défaillance.
Sélectionnez l'une des instances Inventory-App. Vous pouvez sélectionner n'importe laquelle.
Sélectionnez État de l'instance > Résilier l'instance.

Dans la fenêtre Terminate instance? (Résilier l'instance ?), choisissez Résilier.
D'ici peu, les vérifications de l'état de l'équilibreur de charge vont remarquer que l'instance ne répond pas. L'équilibreur de charge va automatiquement acheminer toutes les demandes vers l'autre instance.
Revenez à l'onglet de l'application web dans le navigateur web. Rechargez la page  plusieurs fois.
Vous noterez que la zone de disponibilité affichée en bas de la page est restée la même. Même en cas de défaillance d'une instance, votre application reste disponible.
Après quelques minutes, Amazon EC2 Auto Scaling détecte à son tour la défaillance de l'instance. Amazon EC2 Auto Scaling a été configuré pour continuer à exécuter deux instances, si bien qu'il va automatiquement lancer une instance de remplacement.
Revenez à l'onglet de la console Amazon EC2 qui répertorie les instances. Dans la zone en haut à droite, sélectionnez l'icône d'actualisation  toutes les 30 secondes ou jusqu'à ce qu'une nouvelle instance EC2 s'affiche.
Après quelques minutes, la vérification de l'état de la nouvelle instance devrait devenir saine. L'équilibreur de charge va recommencer à répartir le trafic entre les deux zones de disponibilité. Vous pouvez recharger l'onglet de l'application web pour voir ce qu'il se passe.
Cette tâche démontre que votre application est désormais hautement disponible.





<img width="806" height="237" alt="image" src="https://github.com/user-attachments/assets/7b82178d-0389-4812-93b7-a452fce66a44" />



<-------------------->



<img width="706" height="450" alt="image" src="https://github.com/user-attachments/assets/10b08e6f-8786-4305-90de-217ebc8c9e52" />





<---------------------->

initialisation d,une nouvelle  instance  en cours 



<img width="818" height="249" alt="image" src="https://github.com/user-attachments/assets/2df11ec5-bcd1-4390-879f-158ff5d02e76" />



<------------------->



<img width="819" height="198" alt="image" src="https://github.com/user-attachments/assets/a71923c9-4237-469b-b45f-889827969582" />



<---------------->



Note >.

On n'a pas  fais la derniere partie  .... manque de temps dans Lab 

Merci 


<img width="259" height="273" alt="image" src="https://github.com/user-attachments/assets/1cd846ee-8210-4db9-a600-2c8282c3df4b" />












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



<img width="915" height="369" alt="image" src="https://github.com/user-attachments/assets/1da50020-4203-45b8-982e-fed72aa09e4e" />


<------------------>




















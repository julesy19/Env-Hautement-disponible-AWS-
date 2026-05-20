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

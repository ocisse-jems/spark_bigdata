-  **Spark**
  utilise intensivement la mémoire (RAM)  pour faire ses calculs, à l'inverse de spremières versions d'Hadoop
  qui utilisaient les disques durs. Cependant, RAM largement plus chère que le disque dur et on peut en mettre
  bcp moins sur un ordinateur. 
  D'où continuer à utiliser Hadoop pour gérer de gros volumes qu'il ne serait plus rentable de gérervia de la RAM.
  On met alors en place un architectue qui contient les 2 technologies, sur dusque et sur RAM = **lambda architectures**


- **DAG** (Directed Acyclic Graph)
  Du point de vue Spark les données ne sont pas nécessairement sur l'ordinateur local, ce qui fait que les calculs
  se font à distance.
  DAG est l'ens. des opérations mathématiques que l'on veut effectuer.
  
  Spark = framework de programmation **lazy**: les fonctions ne s'executent que lorsqu'on leur demande explicitement.
  Avant, seule la syntaxe est vérifiée.
  Les données étant censées être disponibles sur chaque ordinateur (notamment grâce à HDFS), il est possible d'effectuer
  un calcul partiel sur chaque ordinateur du cluster, par exemple: 
  lire le 1er quart du ficher sur le 1er ordinateur, le 2eme quart sur le 2nd, le3eme quart sur le 3eme et 
  le 4eme quart sur le 4eme rodinateur.
  Ainsi, gain de temps en accès disque acr les lectures sont parallélisées.

- **RDD** (Resilent Distributed Datasets)
  Objet au coeur de Spark qui effectue tous les calculs
  Resilient = les RDD résistent à la panne d'un noeud. Si une machine du cluster tombe en panne, le calcul continue
  sur une autre machine, avec un recalcul du DAG si besoin
  Distributed = les données sont distribuées sur le cluster. Spark se charge de trouver et placer les calculs sur 
  les differents noeuds du cluster de maniere autonome
  Datasets = ens. de données, de types distincts

  Le DAG résume les programmes Spark à des manipulations de RDD
  Le Dataframe est une surcouche pour manipuler des RDD avec facilité et rapidité

  RDD = ens. de données partitionnés et distribués sur un cluster d'ordinateur, sur lesquels on peut lancer
  des calculs effectués en parallèle. Elles sont :
  -- immuables : on ne peut pas change rle contenu d'un RDD, faut en crée une nouvelle
  -- partitionnées : données peuvent être splittées sur differents noeuds du cluster
  -- en memoire : jusqu'à la limite de celle-ci sinon Spark swape sur le disque
  -- lazy : les calculs ne sont effectués qu'à la fin de la definition du DAG suite à une action


- **PAIRERDD**: pour les calculs de type MapReduce
  Pour faire des oerations avec l'algo MapReduce, il est important de ravailler avec des objets repondant
  à lalogique clef-valeur (comme JSON, ou les dictionnaires)
  Pour cela, Spark transforme nativement des RDD en paired-RDD lorsque la fonction lambda en parametre
  d'un Map renvoie un tuple de donnees
  La 1ere valeur du tuple sera condideree comme la clef et la 2nde valeur comme la valeur de cette nouvelle 
  paired-RDD


- **DATAFRAMES**
  ou table de données. Les dataframes s'appuient sur les RDD. C'est un RDD ave un schema caraterisant les types
  de donnes contenues permettant d'utiliser les fonctions de Spark SQL.
  RDD et dataframe sont les 2 objets de base de Spark.


- **Application SPARK vs Cluster Spark**
  Cluster Spark = ens. d'ordinateurs reliés entre eux de maniere perenne. Constitué de 2 types de machines :
  un master et des workers. ce sont des processus de JVM.
  master et workers sont chacun sur des serveurs differents, relies entre eux par un reseau et communiquent de 
  façon cryptee e sans mot de passe grace à lamise en place de clefs ssh.
  L'ens. constitue le cluster, sur lequel se créeront les applications.

  Application Spark = execution de DAG sur le cluster. Les apps. demarrent et s'arretent sur le cluster
  qui est, lui permanent.
  Les **workers** envoient au **master** leur nombre de CPU et de RAM disponible. Ainsi, le master connait les ressources
  disponibles et va gerer la distribution de la puissance de calcul et de l'espace meoire aux applications.
  Lors de la creation d'une app le master cree une JVM appelee **driver** et demande ensuite aux workers de creer
  des JVM appelees **executors** dans lesquels se calculeront les operations demandees par l'application

  Cette partie de creation de JVM et d'attribution de la puissance de calcul est appelée **Cluster Ressource Scheduling**
  Une fois le driver cree, ce dernier communique directement avec les executors pour distribuer les operations du DAG
  au sein des **task slot** gérés par chaque executor.
  Ces task slots representent des **threads** dans lesquels le scheduler du driver positionne les operations du DAG.
  Cte etapes s'apelle **Spark Ressource Scheduling**


- **Composants du cluster**
  -- le client (généré par le script **spark-submit** ou **spark-shell**)
    . demande à demarrer le driver
    . initialise les variables de configuration de l'app.
    . initialise les classPath
  -- le master 
    . connait les ressources disponibles au niveau master
    . execute la partie "Cluster Ressource Scheduling" (demande aux workers de crer des executors)
    . gere les options de deploiement par defaut
  -- le driver
    . un seul par application
    . prend en charge 4 elements clefs:
        .. **Spark Context**
            ... demande ds ressources au cluster
            ... cree les RDD, charge le donnees ...
        .. **Scheduler**
            ... construit la logique des taches à effectuer par l'application
            ... distribue les taches sur les executors alloués (Spark Ressource Scheduling)
            ... gere la data-locality (HDFS et RDD cache)
        .. **JVM heap**
            ... limitée à celle du client
            ... ajuste la taille memoire allouee (option -Xmx)
        .. Code de l'app. Saprk
            ... code source (ex: .jar si spark-submit)
  -- l'executor
    . reçoit les taches du Scheduler
    . execute les taches dans des task slots
        .. ce sont des threads, le nb de threads peut etre 2 à 3 fois superieur au nb de CPU
    . retourne les resultats au driver
    . JVM heap
        .. memoire geree localement selon les regles memoires parametrees pour l'application


- **Etapes d'une application Spark**
  Les applications ont pour vocation à faire des calculs sur les donnees.
  Chaque calcul est appelé **job**
  Spark est lazy/paresseux: il demande que soit explicitée lalogique des calculs à effectuer (DAG)
  avec differentes operations de transformations (select, filter, agg ...) avant de lancer ces calculs
  via une operation d'action (collect, take, show...)
  
  Les jobs sont terminés ou finis par les operations d'action. Mais ils sont eux memes subdivisés en **stages**
  ou étapes, et **transformations**

  Les stages sont creees par des transformations speciales qui necessitent un reordonnancement de la donnee
  à travers le reseau => le passage d'une stage à une autre fait appel à bcp de ressources reseau et est assez couteux.
  C'est le cas pour la plupart des transformatins qui appliquent un regourpement par clef (reduceByKey, countByKey, ...)
  Pour ces operations, il est necessaire que toutes les lignes de nos donnes soient regroupees sur la meme machine
  afin par exemple de pouvoir les comptes exhaustivement

  -- **parallelisme**
    les taches sont effectuees au niveau de chaque partition, chacune par un task slot (thread) different
    Le niveau de parallelisme a donc un impact direct sur la vitesse d'excution des calculs mais doit aussi 
    être optimisé en fonction des donnees que l'on a.
    Un niveau de parallelisme par defaut est parametré pour chaque application.
    Par defaut en mode **stand alone** Spark utilisera tous les coeurs/task slots présents sur la machine locale
    Il est possible de forcer le parametrage avec la variable d'env. **spark.default.parallelism**
    Un certain nombre de fonctions permettent de preciser le niveau de parallelisme attendu en sortie de leurs
    transformations ( ex : sc.textFile(chemin, nb_de_partition) // df.coalesce(nb_de_partition))
    Trouver le bonequilibre dans le partitionnement des donnes est une tache importante d'optimisation dans les calculs:
        . troppeu de partition entraine une sous-utilisation des task slots
        . trop de partition rajoute un poids excessif pour manipuler et gerer toutes les partitions

  -- **analyse basee sur les clefs**
    Dans l'algo MapReduce, la phase de tri est prise en charge par le systeme lui meme.
    Ce tri des donnees passe par un accès reseau et est aussi materiellement couteaux
    Les fonctions de tri **ByKey** entrainent quasiment systematiquement ce shuffle des donnes.
    Ce sera la frontiere entre 2 stages


  -- **Débuggage via des interfaces graphiques web**
    Spark fournit des IHM web pour surveiller le fonctionnement du cluster.
    Chaque element du cluster fournit sa propre interface, sur des numeros de ports connus par defaut:
        . master : 8080 (la page du master est dispo sur le port 8080 de lamachine-maitre)
        . slave :  8081
        . app :    4040

  
  -- **Types de calculs possibles**
    Spark propose plusieurs paradigmes d'analyse :
        . mode batch: utile pour les grosses données, la distribution des calculs permettant une vitesse 
          de transfert ideale du disque vers la memoire
        . temps reel: pour analyser les flux de donnees et effectuer des calculs en continu
        . iteratif: Spark est très bon pour les descentes de grdient et les operations d'optimisation convexe
        . interactif: la capacité à donner àl'utilisateur des reponses en qq secondes permet une facilité d'exploration
          de donnes notament grace aux calculs effectues en memoire et grace à une tres faiblelatence

    Spark a aussi 2 modes operatoires distincts, pour lesquels le memecode source peut etre utilisé :
        . mode local: sur une seule machine
        . mode distribué: sur un cluster, en lien avec 4 ressources manager possibles: Standalone, Yarn,
          Mesos, Kubernetes
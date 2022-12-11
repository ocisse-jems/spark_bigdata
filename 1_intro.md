# Intro



- **Où conserver les données ?**
  Données stockées sur disques durs. Points faibles des ordinateurs car très lents.  
  Problème lorsque l'on a bcp de données à lire e à stocker  

- **MTBF** (Mean Time Before Faillure)  
  Problème lorsque l'on a plusieurs disques durs : risque de panne à l'installation ou après qq années.  
  Coûts et risques : matériel, arrêt de services, instabilité perte d'information

- **Stockage districué et HDFS**
  Hadoop résout le problème lié au MTBF en dupliquant la donnée (par défaut 3 copies)  
  **HDFS** est un outil qui s'occupe de copier les informationsà enregister d'un ordianateur (ou **_noeud_**)  
  vers d'autres, automatiquement

  Avantage : si un disque dur tombe en panne, on peut récupérer l'info dans d'autres disques  
  Les fichiers sont dupliqués sur HDFS en étant tout d'abord divisés en blocs.  
  HDFS permet de faire une lecture parallélisée des fichiers  


  Exemple : un cluster HDFS de 4 machines (avec adresses IP) et un fichierde 4 lignes enregistré sur ce cluster  
  -- le fichier serait donc divisé en 4 blocs : B1, B2, B3, B4  
  -- chaque fichier serait dupliqué parle facteur de réplication, par défaut 3 fois, sur le cluster  
  -- le **NameNode** (noeud central) enregistre l'emplacement de chaque bloc (dans son fichier **FS Image**)  
  -- chaque **DataNode** enregistre des blocs de fichiers  

- **Algorithme MapReduce**  
  propose de penser les ogiciels comme devant fonctionner sur plusieurs ordinateurs  

  Exemple : compter le nombre d'occurences des mots dans un fichier en Map Reduce  
  -- Map : pour chaque mot du fichier, renvoyer lemot associé à la valeur 1  
           Mode "clef:valeur"  / clef = un mot  / valeur = 1  
           Nb de "clef:valeur" = nb de mots dans le fichier  

  -- Sort : regrouper ensemble les paires "clef:valeur" qui ont la même clef  
            faire une liste par clef  
            Nb de liste = nb de mots différents dans le fichier  

  -- Reducce : pour chaque liste, additionner toutes les valeurs  

    La phase de Map est distribuée sur les différentes machines du cluster, chaque machien renvoyant  
    au master une liste de résultats intermédiaires. C'est cette liste que le master va réduire en  
    un seul résultat via la fonction Reduce qui n'est exécutée que de manière centrale  

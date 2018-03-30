### Table des matières
* Contexte
* Accès à SAUS
* Création d'un espace de stockage de type NAS (accès fichiers)
* Création d'un espace de stockage de type SAN (accès blocs)
* Création d'un miroir pour tout type d'espace de stockage existant
* Ajout de client dans la liste d'accès d'un export NFS
* Modifier le fichier de configuration

### Contexte
Pour simplifier et assurer le service en toute circonstance, SAUS peut être utiliser pour faciliter les tâches récurrentes et quotidiennes avec consistance.
Cela permet à toute personne n'ayant pas une connaissance approfondie de l'environnement de stockage NetApp de pouvoir intervenir sur l'infrastructure avec confiance.

### Accès à SAUS
L'accès à SAUS se fait via la machine infrastructure de scripting: <strong>si5215v</strong> via le compte <strong>m15lot2</strong>. Pour rappel, les informations de connexion se trouve dans le keepass dans la section NetApp. 

Une fois connecté, dans le chemin courant se trouve un répertoire "saus":
```
[m15lot2@si5215v ~]$ ls -l
total 4
drwxrwxr-x 7 m15lot2 m15lot2 4096 Feb  9 12:39 saus
[m15lot2@si5215v ~]$ cd saus/
[m15lot2@si5215v saus]$ ls
build  LICENCE  log  NAS  NetApp  saus.pl  TODO
```

Une fois dans ce répertoire, il suffit d'exécuter la commande suivante afin de lancer le script:
```
[m15lot2@si5215v saus]$ ./saus.pl
```
Cela lancera le menu comme ci-dessous:
```
Storage Automation Menu
    Current available flows:
    1   Create a volume & share for CIFS
    2   Create a volume & export for NFS
    3   Create a volume & LUN for VMWare (SAN)
    4   Create a qtree in an existing volume
    5   Create a mirror of an existing volume
    6   Add NFS clients to existing NFS export
    h   Help
    0   Exit
    Please select an option:
```

### Création d'un espace de stockage de type NAS (accès fichiers)
à écrire

### Création d'un espace de stockage de type SAN (accès blocs)
à écrire

### Création d'un miroir pour tout type d'espace de stockage existant
à écrire

### Ajout de client dans la liste d'accès d'un export NFS
à écrire

### Modifier le fichier de configuration
à écrire
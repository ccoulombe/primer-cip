
# Prérequis
Afin d'utiliser les systèmes de calculs, vous devez avoir un compte à [l'Alliance](https://alliancecan.ca/fr/services/calcul-informatique-de-pointe/gestion-de-compte/demander-un-compte).

# Alliance Canada et Calcul Québec
L'alliance de recherche numérique du Canada châpeaute les consortiums régionnaux tel que Calcul Québec.
Calcul Québec offre différent services (formation, soutien technique) à ces usagers.

![image](images/DRAC.png)

Avec votre compte, vous avez accès aux différents systèmes au travers le pays.

# Documentation technique
Le wiki regorge d'informations pertinente. Vous trouverez certainement réponse à vos questions : https://docs.alliancecan.ca/mediawiki/index.php?title=Special:Search

* [Documentation technique](https://docs.alliancecan.ca/wiki/Technical_documentation)

# Utilisation d'un système de calcul
Vous vous connectez à l'aide d'un terminal et vous attérissez sur un noeud de connexion. Ce noeud est partagé avec tous les autres usagers, et vous pouvez y travailler, configurer, faire de courts tests et préparer vos tâches de calcul.

À l'aide d'un script de soumission, vous pourrez soumettre votre tâche à l'ordonnanceur, qui allouera celle-ci sur un noeud disponible pour l'accueillir.

Tous les noeuds ont accès aux différents espaces de stockage (`/home`, `/project`, `/scratch`).

![image](images/hpc.png)

L'ordonnanceur utilise le principe de [fair-share](https://docs.alliancecan.ca/wiki/Job_scheduling_policies#Priority_and_fair-share), soit plus vous calculer (utiliser de ressources), plus votre priorité diminue, et moins vous calculer, plus votre priorité se rétablie.


## Transfert de données
Vous pouvez transférer vos données depuis votre ordinateur vers le système de calcul à l'aide de `scp` ou un client `sftp` (tel que `FileZilla`).

Depuis le noeud de connexion, vous pouvez télécharger vos jeux de données (au besoin) à l'aide de `wget` par exemple.

Note: Les noeuds de calculs n'ont pas accès à internet.

![image](images/transfert-donnees.png)

# Exemples
## Charger un module
Vous pouvez visualiser la liste de logiciel déjà disponible sur les systèmes: https://docs.alliancecan.ca/wiki/Available_software

Vous pouvez également chercher un mot-clé directement à l'aide de la commande `module spider`:
```raw
module spider python
```
Vous obtiendrez une liste des versions disponible ou une erreur si le module n'existe pas.

Si le logiciel existe, vous pouvez le charger avec la commande `module load`. Par exemple:
```bash
module load python/3.13
```

Ainsi, `python` est accessible et vous pouvez l'utiliser:
```
python --version
```

Commandes `module` courantes:
```bash
# Lister tous les modules chargés
module list

# Décharger un module spécifique
module unload python/3.13

# Décharger tous les modules
module purge
```

## Installation wheel Python

[Référence](https://docs.alliancecan.ca/wiki/Python)

Il est fortement suggérer de créer un fichier de requis. Cela évite une panoplie d'erreurs et assure que l'environnement virtuel créé est identique, assurant ainsi des tâches reproductibles d'une fois à l'autre : [créer son env. virtuel dans notre tâche](https://docs.alliancecan.ca/wiki/Python#Creating_virtual_environments_inside_of_your_jobs).

1. Charger un module Python
```bash
module load python/3.13
```
2. Créer un environnemnt virtuel
```bash
virtualenv --no-download ~/ENV
```
3. Activez celui-ci:
```bash
source ~/ENV/bin/activate
```
4. Mettre à jour `pip`
```bash
pip install --no-index --upgrade pip
```
5. Installer un wheel

Note: les noeuds de calculs n'ont pas accès à internet et donc pas à PyPI!

Depuis la wheelhouse:
```bash
pip install --no-index numpy
```
`--no-index` assure que les wheels de la wheelhouse sont utilisé et n'accède pas à PyPI.

Depuis un noeud de connexion/login:
```
pip install setuptools numpy
```
Ici, les wheels de la wheelhouse seront utilisés, mais `pip` utilisera également PyPI.

6. Test si l'installation est ok
```bash
python -c 'import numpy'
```

## Installation paquet R

Les noeuds de calculs n'ayant pas accès à internet, il est primordial d'installer les paquets R depuis un noeud de connexion.

[Référence](https://docs.alliancecan.ca/wiki/R)

1. Charger le module
```bash
module load r/4.4
```

2. Installer une librairie

Dans le dossier par défaut (~/R/gnu)
```bash
# installer
R -e 'install.packages("sp", repos="https://cloud.r-project.org/")'
```
où `sp` est le nom du paquet à installer.

Dans un dossier spécifique:
```bash
# créer un dossier d'installation
mkdir -p ~/.local/R/$EBVERSIONR/
# spécifier où trouver les librairies
export R_LIBS=~/.local/R/$EBVERSIONR/
# installer
R -e 'install.packages("sp", repos="https://cloud.r-project.org/")'
```

3. Test simple

Tester que la librarie est bien accessible:
```bash
R -e 'library(sp)'
```

4. Ainsi, dans votre script de soumission:
```bash
....

module load r/4.4

R -f mon_script.R
```

## Soumission d'une tâche
Rappel: les noeuds de calcul n'ont pas accès à internet.

Pour soumettre une tâche de calcul, il faut créer un fichier de soumission et utiliser la commande `sbatch` afin de soumettre notre script.
1. Créer un fichier de soumission

Créer le fichier (ex. `job.sh`) avec les ressources nécessaire et les commandes à exécuter:
```bash
#!/bin/bash

#SBATCH --account=<def-username>    # Remplacez par votre compte
#SBATCH --time=01:00:00             # Format de temps : HH:MM:SS
#SBATCH --cpus-per-task=1           # Nombre de cœurs CPU par tâche
#SBATCH --mem=4G                    # Mémoire pour la tâche
#SBATCH --job-name=mon_job          # Nom du travail
#SBATCH --mail-user=votre.email@example.com  # Email pour les notifications
#SBATCH --mail-type=ALL             # Quand envoyer un email (BEGIN, END, FAIL, ALL)

# Charger les modules nécessaires
module load python/3.13

# Exécuter vos commandes
python mon_script_python.py
```

2. Soumettre vôtre tâche
Soumettre le fichier de soumission à l'ordonnanceur (Slurm):
```bash
sbatch job.sh
```
Vous obtiendrez un numéro de tâche (jobid).

Les sorties de votre tâche se trouveront dans le répertoire de soumission par défaut, soit où vous avez soumis votre tâche, sous le nom (par défaut) : `slurm-<jobid>.out`.

# Annexe
## CPU vs GPU
Le CPU est distinct du GPU. Ce dernier est un accélérateur graphique, et nous devons transférer les données de la mémoire système (CPU) vers le GPU, et vice-versa afin de récupérer les résultats.

![image](images/cpu-gpu.png)

## Parallélisme - Multi-coeurs

Certains programmes sont en mesure d'utiliser plus d'un cpu, par exemple 4 cpus sur 1 noeud.
![image](images/multi-coeurs.png)

## Parallélisme - Multi-noeuds Multi-coeurs
Certains programmes sont en mesure d'utiliser plusieurs cpus, distribués sur 1 ou plusieurs noeuds.

![image](images/mc-mn.png)

## Ordonnancement (Slurm)
L'ordonnanceur (Slurm) s'occupe d'allouer les tâches de la file d'attente, sur les noeuds, lorsque les ressources sont disponible.

Par exemple, dans l'exemple ci-bas, la tâche #4 ne peut être allouée, mais la 5e, si elle est soumise, pourra être allouée et ce même si d'autres tâches sont en attente.
![image](images/tetris.png)

L'allocation des tâches de calcul, est similaire à la file d'attente au restaurant. Il y a des personnes devant et derrière vous. Si vous êtes seul, mais attendez qu'une table de 4 se libère, il se peut que le groupe de 2 (2 places) derrière vous se fasse assigner une table avant vous.

# FAQ
## Estimation des ressources ?

### Bonnes pratiques
* Commencez petit pour calibrer vos besoins
* Ajustez progressivement les ressources demandées

Une bonne estimation des ressources pour votre tâche est essentielle pour:
* Réduire le temps d'attente dans la file;
* Minimiser le gaspillage des ressources;
* Éviter les échecs de tâches par manque de ressources

L'estimation des ressources requises pour une tâche dépend du contexte et de chaque logiciel.
Ainsi, il n'existe pas de recette magique. Pour estimer au mieux les ressources, il faut faire
quelques tests essais-erreurs.

Il est aussi pertinent de lire la documentation des outils utilisés afin de connaître les ressources utilisé.
Par exemple, si l'outil est capable d'utilisé plusieurs cpus/threads, un GPU ou non, etc.

Dans le doute, débuter avec un minimum de ressources.

1. Débuter avec un ensemble de données petit, 1 cpu, 20GB de mémoire et 8 heures.
Puis surveillez votre utilisation de ressources:
- avec htop
- portail (Narval et Béluga)
- avec nvtop (GPU)

2. Ajuster les ressources en conséquence.
Si la mémoire a manquée, il faudra augmenter celle-ci, par exemple à 50GB.
Si au contraire la mémoire a été suffisante et que seulement 5GB ont été utilisé, il est pertinent de diminuer la demande (ex. ~8GB).

3. Augmenter l'ensemble de données au besoin, et ajuster les ressources.

# Liens utiles
* [documentation technique](https://docs.alliancecan.ca)
* [suivi tâches; portail de Narval](https://portail.narval.calculquebec.ca/)
* [suivi tâches; portail de Béluga](https://portail.beluga.calculquebec.ca/)
* [soumission de tâches](https://docs.alliancecan.ca/wiki/Running_jobs)
* [modules disponibles](https://docs.alliancecan.ca/wiki/Available_software)
* [wheels disponibles](https://docs.alliancecan.ca/wiki/Available_Python_wheels)
* [formations Calcul Québec](https://calculquebec.eventbrite.ca/)

# Glossaire
* Noeud de connexion (login):
Noeud principal sur lequel nous nous connectons, afin d'effectuer diverses tâches (configuration, téléchargement de données, soumission de tâche).
* Noeud de calcul (compute): Noeud où une tâche est allouée par l'ordonnanceur, où s'exécute le contenu du script de soumission.
* Ordannceur / Scheduler (Slurm): Logiciel permettant d'ordonner les tâches dans la file d'attente. Connaît les tâches actuelles et celles en attentes. (Pensez à un hôte au restaurant.)

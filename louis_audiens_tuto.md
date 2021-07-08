# Introduction
Présentation des projets Audiens à destination de Louis Martin


# Présentation
Il y a 5 principaux projets Audiens : 
 - Plateforme retraite : https://github.com/guillaume-guerdoux/plateforme-retraite / https://accompagnement-retraite-audiens.org/
 - Plateforme fussat (ministere) : https://github.com/guillaume-guerdoux/aide-audiens-ministere / https://fussat-audiens.org/
 - Plateforme aide exceptionnelle : https://github.com/guillaume-guerdoux/audiens-aide-exceptionnelle / http://aide-exceptionnelle-audiens.org/
 - Plateforme d'aide à la garde d'enfants : https://github.com/guillaume-guerdoux/agedati / http://garde-enfant-fonpeps-audiens.org/ 
 - Plateforme des bourses d'études :  https://github.com/guillaume-guerdoux/audiens-bourse-etudes / https://audiens-bourses-etudes.org/

Il y a aussi une plateforme qui est juste un site vitrine : 
- Audas pro : https://github.com/guillaume-guerdoux/audas-pro-vitrine


# Workflow d'une plateforme
Sur chaque plateforme c'est à peu près toujours le même workflow : 
1. Un bénéficiaire fait un test d'eligibilité
2. Si il passe le test il s'inscrit
3. Il accède ensuite à son tableau de bord où il peut charger ses documents, renseigner ses informations et cliquer sur "Demander la validation de mon dossier"
4. Les conseillers Audiens ont un compte (avec un profil CONSEILLER), il accèdent à la liste des demandes des bénéficiaires. Il y a plein de filtres (par statut de l'aide, par type d'aide, recherche par nom prénom, etc.)
5. Le conseiller étudie le dossier et peut le valider (ou le refuser, ou  le mettre en incomplet avec un message)
6. Les manager Audiens ont un compte (avec un profil MANAGER), et pareil, ils peuvent valider les dossiers déjà validés par les conseiller (ou refusé, etc.)
7. Puis les conseillers peuvent mettre le dossier en "Mis en paiement" quand ils ont payé le bénéficiaire via leur logiciel de paiement

# Technologies utilisées
Les plateformes sont toutes développées sous les mêmes technos :
- Vue.js (vue cli version 2) en front
- Django (v2) et django rest api en back
- Celery et redis pour les tâches asynchrones

# Architecture des applications
Toujours la même architecture de dossier : monorepo et à la racine un dossier :
- nomduprojet-front-end/ : c'est tout le code du frontend
- nomduprojetBackEnd/ : c'est tout le code du backend

Sur le frontend, les pages sont en gros dans views/ et les composants dans components/
Sur le backend, c'est du django donc tu as les app en gros avec les models, les views, les serializers.

Attention, il y a juste deux fichiers qui ne sont pas dans le repo : 
- localsettings.py qui doit être dans nomduprojetBackEnd/nomduprojetBackEnd
- localVariables.js qui doit être dans src/variables/

Pour les récupérer, le mieux est d'aller sur le serveur et de download ces fichiers puis de les adapter pour le local.

# Démarrage en local
Pour démarrer en local, il faut installer vue.js bien sûr https://cli.vuejs.org/guide/installation.html --> Version 2
Puis tu crées ton virtualenv à la racine du projet avec : 
```
virtualenv .venv -p python3
source .venv/bin/activate
```
Tu dois ensuite installer les requirements.txt
```
pip install -r requirements.txt
```
Pour lancer le backend 
```
cd nomduprojetBackEnd/
python manage.py runserver
```

Pour lancer le frontend, via un autre terminal
```
cd nomduprojet-front-end/
npm run serve
```

L'application devrait fonctionner ! 

Il faut aussi lancer celery si tu veux utiliser les tâches asynchrones (attention il faut d'abord avoir redis installé et starté)
```
celery worker -A projectBackEnd --loglevel=INFO
```

Il y a aussi le scheduler (qui lancer des tâches à des heures précises). C'est défini dans le fichier celery.py qui est dans nomduprojetBackEnd/nomduprojetBackEnd
Pour lancer le scheduler 
```
celery beat -A projectBackEnd
```

Ok tout fonctionne !

# Déployer sur le serveur
Alors tout d'abord, toute la procédure de déploiement d'un nouveau projet de ce type est dans le repo wiki/deploy_django_vue_on_ubuntu.md
C'est générique mais c'est tout la procédure du coup tu as quasi toutes les commandes.

Lorsque tu clones le repo, si tu veux déployer il faut que tu ajoutes un git remote : 
```
git remote add production guillaume@XX.XX.XX.XX:name-project.git
```
Le dossier name-project.git est sur le serveur à /home/guillaume
Et ensuite si tu veux deployer sur le serveur tu fais :
```
git push production master
```
Ca va pusher le repo sur le serveur et declencher le hook post-receive qui va updater les fichiers et relancer tous les applicatifs. Le hook post-receive est situé dans /home/guillaume/name-project.git/hooks/post-receive
Si jamais ca te rejette --> git push production master --force
Si jamais tu veux comprendre, tu peux le mettre sur le wiki

# Explications du serveur
C'est un vps ovh.
Tout le déploiement est vraiment dans le fichier wiki/deploy_django_vue_on_ubuntu.md
Front end : 
Très simple, à chaque fois qu'on push, tu rebuild le front (npn run build), et ca te le met dans un dossier qui est servi par nginx sur le port 443 ssl (le port 80 redirige sur le port 443).

Backend : 
base de données c'est du postgres. Si tu veux faire des modifs en fait tu fais juste
```
sudo su - postgres
```

Ensuite le backend est servi par un UWSGI et nginx sur le port 8443 en ssl. 
Si jamais tu veux accéder à l'interface de django via ton navigateur c'est : 
fussat-audiens.org:8443/<URLADMIN>

Les celery sont sous supervisor. 

Toutes les commandes pour tout relancer, starter elles sont toutes dans le post-receive.


# Certificat SSL 
Faut aller sous gandi, certificats SSL.
Ensuite un génères un CSR  en local sous linux
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
Tu achètes un certiciats SSL en copiant collant le csr (bien exlpliqué sous gandi)
Une fois que c'est acheté et validé, tu as trois ficheirs :
 - le .csr
 - le .key
 - le .crt 

Faut copier coller le key et crt dans le dossier /etc/ssl du serveur. Redémarrer nginx.

# Les documents
Ils sont tous sur un serveur AWS S3. Django est configuré pour automatiquement balancer les documents sur s3. 

Du coup en local attention, si tu prends la base de prod tu vas avoir des bugs pck les ficheirs existent pas et que le local est pas fait pour synchro avec S3. 
Si vraiment c'est important pour toi d'avoir les documents, tu peux juste setup DEBUG a FALSE dans localsettings.py et rajouter les identifiatns s3 de l'API --> Attention, du coup tu es interfacé avec les documents de prod. 



# Contact audiens
claire.gentil@audiens.org
julien.da_silva@audiens.org


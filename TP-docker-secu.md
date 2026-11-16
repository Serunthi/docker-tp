Mission 1 

● Construisez votre image initiale sous le nom quiz-docker:initial.

```docker build -t quiz-docker:initial .```
docker build : construit une image à partir du Dockerfile.
-t : attribue un nom et un tag à l'image.
quiz-docker : nom de l'image.
initial : version avant sécurisation.
. : indique que le contexte de construction est le dossier actuel


● Vérifiez que l'application fonctionne.

- Vérifier l'image
```docker images quizz-docker```
- Vérifier l'appli 
```docker run -d --name quiz-initial -p 8080:3000 quiz-docker:initial```

-d	Exécute le conteneur en arrière-plan
--name	Donne le nom quiz-initial au conteneur
-p 8080:3000	Relie le port 8080 du PC au port 3000 du conteneur
quiz-docker:initial	Image utilisée

- Vérifier l'état du conteneur 
```docker ps```


● Identifiez l'image de base utilisée.
```FROM node:22-alpine```

● Vérifiez sous quel utilisateur s'exécute le processus applicatif.

Un attaquant qui compromet l'application pourrait disposer de privilèges élevés dans le conteneur. Cela augmente les risques, notamment si le conteneur est mal configuré.
Exécution avec un utilisateur non-root :

Le processus dispose de moins de privilèges, ce qui limite les conséquences potentielles d'une compromission.

● Relevez la taille de l'image.
Content Size = 74.3MB
Disk Usage = 285 MB

● Analysez les différentes couches qui la composent.
```docker history quiz-docker:initial```

Installation de Node.js,
Installation des dépendances : des dépendances de développement ou des packages vulnérables peuvent être présents dans l'image.
Absence de user, (exécution avec root)
COPY . .	Copie éventuelle de fichiers sensibles si .dockerignore est insuffisant
npm run dev	Utilisation d'un serveur de développement non adapté à la production

● Réalisez une première analyse des vulnérabilités.
● Conservez les résultats de cette analyse.
```mkdir audit```
```docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest image --format json quiz-docker:initial > audit\trivy-initial.json```


Questions :
● L'image utilisée est-elle adaptée à une utilisation en production ?
Non, elle n'est pas adaptée à une utilisation en production car : l'application est lancée avec npm run dev, qui démarre le serveur de développement Vite, on utilise les droits root, Trivy détecte également des vulnérabilités.

● Quels composants sont réellement nécessaires pour exécuter l'application ?
Vite et Node.js sont utiles pour construire l'application. Mais une fois celle-ci compilée avec npm run build, les fichiers statiques générés dans dist/ peuvent être servis par Nginx, sans conserver Node.js dans l'image finale.

● Quels risques de sécurité identifiez-vous ?
- Exécution avec des droits privilégiés (root), risque de compromissions.
- Fichiers inutiles ou sensibles : COPY . . peut inclure des fichiers non nécessaires si .dockerignore est insuffisant.
- Surface d'attaque importante : conserver Node.js, npm et les outils de développement en production ajoute des composants qui ne sont pas indispensables.
- Dépendances vulnérables : Trivy a identifié des vulnérabilités dans Vite, PostCSS, Rollup et esbuild.

● Quelles améliorations recommanderiez-vous en priorité ?
Utilisation d'un Docker Multi-Stage pour séparer la compilation et l'exécution. 
Mettre à jour les dépendances vulnérables et utiliser des versions d'images maintenues.
Ajouter un .dockerignore pour exclure les fichiers inutiles et sensibles.
Utilisateur d'un compte non-root avec USER


Mission 2 : Sécuriser l'image et le conteneur Docker

Mission 3 : Comparer les résultats avant et après sécurisation
Après avoir appliqué les recommandations :
● Construisez une nouvelle image sous le nom de quiz-docker:secure
● Démarrez un conteneur à partir de cette image et vérifiez le bon fonctionnement de l'application.
```docker image ls```
quiz-docker:initial                       f41424d91b55        285MB         74.3MB    U   
quiz-docker:secure                        ffe8c4bb552c       81.6MB         23.1MB


PS C:\Users\serun\docker-introduction\dockerfile-exercice2> docker run --rm quiz-docker:initial whoami
root
PS C:\Users\serun\docker-introduction\dockerfile-exercice2> docker run --rm quiz-docker:secure whoami
nginx

● Publiez l’image dans votre registry dockerHub
● Complétez un tableau comparatif de l’image initial et sécurisé avec les critères comme présence d'outils de
développement, utilisation de .dockerignore, multi-stage build, limitation des ressources, restrictions des privilèges,
accès au socket Docker, CVE critiques, CVE élevées, CVE moyennes, CVE faibles etc…


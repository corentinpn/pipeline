Ce pipeline montre comment utiliser un conteneur multi-stage dans un workflow DevOps pour construire, tester et déployer une application.

Contenu du fichier .github/workflows/docker-ci-cd.yml

# Nom du workflow, affiché dans l'interface.
name: CI/CD Pipeline for Multi-Stage Docker

# Définition des événements déclencheurs du pipeline.
on:
  push: # Le workflow s'exécute lorsqu'il y a un "push".
    branches: # Limité à la branche principale pour éviter les déclenchements sur toutes les branches.
      - main

# Définition des tâches (jobs) du pipeline.
jobs:
  build-and-deploy: # Nom du job principal.
    runs-on: ubuntu-latest # Le pipeline s'exécute sur une machine virtuelle Ubuntu.

    # Définition des étapes du job.
    steps:
      # Étape 1 : Récupération du code source depuis le dépôt GitHub.
      - name: Checkout code
        uses: actions/checkout@v3 # Action GitHub officielle pour cloner le code.

      # Étape 2 : Construire l'image Docker multi-stage.
      - name: Build Docker Image
        run: docker build -t my-app:latest . # Commande pour construire une image Docker.
        # - `-t my-app:latest` : Tag de l'image pour l'identifier.
        # - `.` : Chemin où se trouve le Dockerfile (dans le répertoire racine du projet).

      # Étape 3 : Authentification au registre DockerHub.
      - name: Login to DockerHub
        run: echo "${{ secrets.DOCKER_PASSWORD }}" | docker login --username ${{ secrets.DOCKER_USERNAME }} --password-stdin
        # - `${{ secrets.DOCKER_PASSWORD }}` : Mot de passe sécurisé stocké dans GitHub Secrets.
        # - `${{ secrets.DOCKER_USERNAME }}` : Nom d'utilisateur DockerHub stocké dans GitHub Secrets.

      # Étape 4 : Pousser l'image Docker vers DockerHub.
      - name: Push Docker Image
        run: |
          docker tag my-app:latest my-dockerhub-username/my-app:latest # Ajoute un tag contenant le nom utilisateur.
          docker push my-dockerhub-username/my-app:latest # Envoie l'image vers le registre DockerHub.

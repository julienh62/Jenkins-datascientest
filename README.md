http://54.216.97.114:8080/login?from=%2Fenv-vars.html
http://adresseip:8080/login?from=%2Fenv-vars.html pour se connecetr à jenkins


### 1. Modification de `main.py`

Chaque fois que vous apportez des modifications au fichier `main.py` (ou à tout autre fichier de votre application), il est nécessaire de reconstruire l'image Docker et de la déployer pour que les changements soient pris en compte.

### 2. Construction de la nouvelle image Docker
#Utilisez la commande suivante pour construire une nouvelle image Docker, en spécifiant une nouvelle version (ex : `v.7.0`) :
docker build -t julh62/datascientestapi:v.7.0 .

#3. Connexion à Docker Hub
#Pour pouvoir pousser l'image sur Docker Hub, vous devez être connecté à votre compte Docker Hub.
docker login

Suivez les instructions pour entrer votre identifiant et votre mot de passe Docker Hub.
#4. Pousser la nouvelle image sur Docker Hub
#Une fois connecté, poussez la nouvelle image vers Docker Hub avec la commande suivante :

docker push julh62/datascientestapi:v.7.0

#5. Déploiement sur Kubernetes avec Helm
#Avant de lancer le déploiement, assurez-vous que le fichier values.yaml contient le bon pullPolicy (réglé sur Always pour récupérer la dernière version de l'image).
#Pour appliquer les changements dans Kubernetes, utilisez la commande Helm suivante :

helm upgrade --install app ./fastapi --values=fastapi/values.yaml --namespace dev

#Cette commande :
 #   Installe (ou met à jour) l’application FastAPI dans le namespace dev.
  #  Charge les valeurs de configuration depuis le fichier values.yaml.
#Vérification du déploiement
#Pour vérifier que les pods fonctionnent correctement, vous pouvez exécuter la commande suivante :

kubectl get pods -n dev

#Si un pod rencontre des erreurs d'image, utilisez la commande suivante pour inspecter les logs et diagnostiquer le problème :
kubectl describe pod <pod-name> -n dev




export NODE_PORT=$(kubectl get --namespace dev -o jsonpath="{.spec.ports[0].nodePort}" services app-fastapi)
export NODE_IP=$(kubectl get nodes --namespace dev -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT

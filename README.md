cat README.md

#Pour vérifier que Java est installé sur notre système, exécutons la commande :
java --version

#Vérifions que Jenkins est bien installé sur notre système et que le service est actif :
sudo systemctl status jenkins
#Puisque Jenkins est basé sur Java, nous devons installer OpenJDK. Pour cela, exécutons la commande :
sudo apt update
sudo apt install fontconfig openjdk-17-jre -y
sudo apt update -y
#Exécutons ensuite :
sudo apt upgrade -y
#Installons à présent Jenkins. Nous ajoutons la clé du référentiel Jenkins à notre système :

curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins -y

http://54.216.97.114:8083/login?from=%2Fenv-vars.html
http://adresseip:8083/login?from=%2Fenv-vars.html pour se connecetr à jenkins

sudo systemctl status jenkins
sudo systemctl start jenkins
Pour activer Jenkins au démarrage, exécutons la commande :

sudo systemctl enable --now jenkins
Nous exécuterons la commande suivante afin de révéler le mot de passe:

sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Affichage en sortie:

943eb6a8472b4e929945a5cb65745f24

Copions et collons le mot de passe dans le champ de texte « Administrator Password », comme indiqué. Une fois collé, cliquons sur le bouton « Continuer »:


Une fois que nous avons recopié le mot de passe généré, nous arriverons sur la page suivante :

Sélectionnons le bouton « Install suggested plugin »,

À présent, cliquons sur Manage plugins. Nous remarquons plusieurs onglets, mais nous allons nous intéresser à l'onglet Manage Plugin. Il s'agit d'une fonctionnalité de Jenkins qui permet d'améliorer son usage.


    Dans la page suivante, cliquons sur l'onglet Available. Les plugins sont regroupés dans 4 onglets :

Dans le champ de recherche, entrons github integration et cochons sur la checkbox afin de sélectionner le plugin github integration:
#ICI; identifiant  =   admin   mot de passse = azerty

installation de kubernetes
k3s kubectl get nodes

Nous devons ensuite ajouter l'utilisateur Jenkins au groupe Docker afin que Jenkins puisse piloter le Docker engine :

sudo usermod -aG docker jenkins

Nous créerons ensuite les Namespaces dans lesquelles nous ferons nos déploiements au sein de Kubernetes depuis jenkins. Nous allons en créer 03, dev, staging et prod :

sudo chmod 755 /etc/rancher/k3s/k3s.yaml
kubectl create namespace dev
kubectl create namespace staging
kubectl create namespace prod

installer docker
sudo apt-get install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
sudo systemctl enable --now docker

Nous devons ensuite ajouter l'utilisateur Jenkins au groupe Docker afin que Jenkins puisse piloter le Docker engine.

Nous le ferons de la façon suivante :

sudo usermod -aG docker jenkins

A - Configurer les tâches de build

Nous allons nous lancer dans la pratique de Jenkins.

Nous devons au préalable installer git (si celui-ci n'est pas installé sur votre machine) avant de commencer à configurer nos projets Jenkins :

sudo apt install git -y

Il est essentiel de créer une tâche de build Jenkins avant de lancer celui-ci. Dans cette section, nous allons créer notre premier travail Jenkins et terminer la configuration initiale.


#Une liste de variable est disponible au sein de Jenkins en ouvrant l'url du serveur 
#master sur un navigateur et en y ajoutant env-vars.html. L'url finale est donc sous ce format :

#http://ip_de_votre_masterjenkins:8083/env-vars.html


Nous pouvons générer depuis notre serveur un fichier de configuration Kubernetes grâce à la commande suivante :

mkdir ~/.kube
sudo kubectl config view --raw > ~/.kube/config

Nous pouvons afficher le contenu de notre fichier de configuration Kubernetes :

cat ~/.kube/config


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

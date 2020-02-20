# gitlab-ci-examples
# Configuration d'un pipeline Gitlab sous Windows

## Pré-requis

* Installer Docker-desktop
* Télécharger Gitlab-runner pour Windows

## Lancer une instance Gitlab

* Créer un conteneur gitlab en lançant la commande suivante:

```
docker run -d --name gitlabce -p80:80 -p22:22 -p443:443 --hostname localgitlab gitlab/gitlab-ce
```

* Ajouter le nom d'hôte du conteneur dans le fichier host de Windows pour pouvoir y avoir accès depuis votre machine

## Enregistrer un Gitlab runner

* Créer un projet de test
* Configurer dans l'onglet settings -> CI/CD -> specific runner en récupérant le token généré
* Dans une invite de commande Windows, lancer la commande:

```
gitlab-runner register
```

Ici spécifier l'URL, le token, le contexte d'exécution, l'image par défaut dans le cas de docker  ainsi que le tag associé à ce runner.

* Modifier la configuration du gitlab-runner __config.toml__ pour que les conteneurs lancés aient accès au réseau de l'hôte

```
  [runners.docker]
    tls_verify = false
    image = "alpine"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    network_mode = "host"
```

* Lancer le runner:

```
gitlab-runner run
```

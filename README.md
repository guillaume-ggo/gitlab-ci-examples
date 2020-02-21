# gitlab-ci-examples
# Configuration d'un pipeline Gitlab sous Windows

## Pré-requis

* Installer Docker-desktop
* Télécharger Gitlab-runner pour Windows
* Installer le binaire __jq__

## Lancer une instance Gitlab

* Créer un conteneur gitlab en lançant la commande suivante:

```
docker run -d --name gitlabce -v etc_gitlab:/etc/gitlab -v var_log_gitlab:/var/log/gitlab -v var_opt_gitlab:/var/opt/gitlab -p80:80 -p22:22 -p443:443 --hostname localgitlab gitlab/gitlab-ce
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

## Sauvegarder l'instance

* Arrêter l'instance

```
docker stop gitlabce
```

* Identifier les volumes de l'instance __gitlabce__

```
docker inspect gitlabce | jq .[].Mounts[].Destination
```

* Créer un conteneur temporaire en attachant les volumes de __gitlabce__

```
docker run --name tmp_gitlab_backup --volumes-from gitlabce -v gitlabbackup:/backup -it debian /bin/bash
```

* Créer sauvegarder les volumes de l'instance Gitlab

```
cd /etc/gitlab
tar cpzf etc_gitlab.tar.gz . && cp etc_gitlab.tar.gz /backup
cd /var/log/gitlab
tar cpzf var_log_gitlab.tar.gz . && cp var_log_gitlab.tar.gz /backup
cd /var/opt/gitlab
tar cpzf var_opt_gitlab.tar.gz . && cp var_opt_gitlab.tar.gz /backup
```

* Arrêter et supprimer le conteneur temporaire

```
docker rm --force tmp_gitlab_backup
```

A ce stade, notre instance Gitlab est sauvegardée dans trois archives __tarball__ compressées.

## Restaurer une instance Gitlab

* Créer un conteneur temporaire en attachant le volume gitlabbackup et les futures volumes de l'instance Gitlab

```
docker run --name tmp_gitlab_restore gitlabbackup:/backup -v etc_gitlab:/etc/gitlab -v var_log_gitlab:/var/log/gitlab -v var_opt_gitlab:/var/opt/gitlab -it debian /bin/bash
```

* Restaurer le contenu des volumes de l'instance Gitlab

```
cd /etc/gitlab
tar xpzf /backup/etc_gitlab.tar.gz
cd /var/log/gitlab
tar xpzf var_log_gitlab.tar.gz
cd /var/opt/gitlab
tar xpzf var_opt_gitlab.tar.gz
```

* Arrêter et supprimer le conteneur temporaire

```
docker rm --force tmp_gitlab_restore
```

* Lancer l'instance Gitlab

```
docker run -d --name gitlabce -v etc_gitlab:/etc/gitlab -v var_log_gitlab:/var/log/gitlab -v var_opt_gitlab:/var/opt/gitlab -p80:80 -p22:22 -p443:443 --hostname localgitlab gitlab/gitlab-ce
```

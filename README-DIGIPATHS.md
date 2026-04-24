# Digipaths notes

This is the OpenMRS O3 reference distribution with customisations on the digipaths branch.

## Customisations

* The list of locations is changed to "Kondavil, Clinic 2, Clinic 3"
* The Tests Orderability concept includes the "serum creatinine mg/dL" instead of "serum creatinine mmol/L"

## Git tips

The workflow is to commit the customisations in this repo onto the digipaths branch and to keep in sync with the original openmrs distro on a regular basis.

### Getting started

Clone the main repo for this project, i.e.

```bash
git clone https://github.com/YOUR_USERNAME/openmrs-distro-referenceapplication
cd openmrs-distro-referenceapplication
```

Then add the original as an upstream remote, i.e.

```bash
git remote add upstream https://github.com/openmrs/openmrs-distro-referenceapplication
```

So that you should see:

```
you@your-machine:~/Software/openmrs-distro-referenceapplication$ git remote -v
origin	git@github.com:mattsouth/openmrs-distro-referenceapplication.git (fetch)
origin	git@github.com:mattsouth/openmrs-distro-referenceapplication.git (push)
upstream	https://github.com/openmrs/openmrs-distro-referenceapplication (fetch)
```

Customisations are on the `digipaths` branch.

### Syncing with upstream

When you want to pull in upstream changes from the original openmrs distro:

```bash
bash# Fetch latest from upstream
git fetch upstream

# Update your main branch to match upstream
git checkout main
git merge upstream/main   # or: git rebase upstream/main

# Rebase your config branch on top of the updated main
git checkout digipaths
git rebase main
```

Using rebase (rather than merge) on the `digipaths` branch keeps history linear and your local commits always sitting cleanly on top of whatever upstream has done.

### Handling conflicts

If upstream changes something you've also configured, rebase will pause and ask you to resolve the conflict file-by-file. Once resolved:

```bash
git add <conflicted-file>
git rebase --continue
```

## Docker tips

### Getting started

To get started, clone this project and then run ``docker compose up -d``, which on ubuntu should look something like this:

```
you@yourmachine:~/Software/openmrs-distro-referenceapplication$ docker -v
Docker version 24.0.5, build 24.0.5-0ubuntu1~22.04.1
you@yourmachine:~/Software/openmrs-distro-referenceapplication$ docker compose up -d
WARN[0000] /home/you/Software/openmrs-distro-referenceapplication/docker-compose.yml: `version` is obsolete
WARN[0000] /home/you/Software/openmrs-distro-referenceapplication/docker-compose.override.yml: `version` is obsolete
[+] Running 9/9
 ✔ db 8 layers [⣿⣿⣿⣿⣿⣿⣿⣿]      0B/0B      Pulled                                                                                  27.0s
   ✔ bccd10f490ab Pull complete                                                                                                     12.8s
   ✔ d9d8e1823c6f Pull complete                                                                                                      0.5s
   ✔ 84f2e2edb76d Pull complete                                                                                                      1.9s
   ✔ 4df97d18a516 Pull complete                                                                                                     13.8s
   ✔ 79fe85183306 Pull complete                                                                                                      2.1s
   ✔ b891b67a5cf8 Pull complete                                                                                                     23.7s
   ✔ ac1d0cb433aa Pull complete                                                                                                     25.3s
   ✔ c29a5135f832 Pull complete                                                                                                      4.4s
[+] Running 4/5
 ⠹ Network openmrs-distro-referenceapplication_default     Created                                                                   1.2s
 ✔ Container openmrs-distro-referenceapplication-db        Started                                                                   0.4s
 ✔ Container openmrs-distro-referenceapplication-backend   Started                                                                   0.5s
 ✔ Container openmrs-distro-referenceapplication-frontend  Started                                                                   0.7s
 ✔ Container openmrs-distro-referenceapplication-gateway   Started                                                                   0.9s
```

It will take some time to download the container images and startup and then even once docker compose is started, the app will still take some time to settle.  In this time you can see logs with ``docker compose logs`` and follow logs with ``docker compose logs -f``.  Once you see a log entry similar to this:

```
backend   | 11-Apr-2024 14:25:41.363 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in 30007 ms
```

You should be able to:
* see the openmrs backend at http://localhost/openmrs - on a fresh install this will take sometime (5 mins or so) to setup the demo data
* see your client at http://localhost/ (which auto redirects to http://localhost/openmrs/spa)
* succesfully shutdown your local version with ``docker compose down``

### restarting after making changes

You have two options

```bash
docker compose down
docker compose build --no-cache backend
docker compose up -d
```

If you want a fully clean slate (drops the MariaDB volume so Initializer starts from scratch):

```bash
docker compose down -v
docker compose up --build -d
```

### resetting docker completely

If you wish to purge everything docker and start from fresh you must run several steps.  First check that nothing is running with `docker ps` and run `docker compose down` if containers from this project are running.

WARNING: if you have other docker / docker compose projects on this machine you will lose all of their data!

* remove containers: `docker rm $(docker ps -aq)`
* remove networks: `docker network rm $(docker network ls -q)`
* remove storage volumes: `docker volume rm $(docker volume ls -q)`
* remove all images: `docker image rm $(docker image ls -aq)`
* prune system artefacts: `docker system prune`

Useful guide: https://thelinuxcode.com/how-to-clean-up-docker-compose/

TODO: prune this guide so that it only affects the docker elements in this project

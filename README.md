# Deployment af produktionsserver 1

## Diagram over services

<img src="https://github.com/frederiksberg/prod-app1-deployment/blob/master/figures/tree.svg" width="900px">

## Opdeling

Projektet er opdelt i 4 logiske kasser.

* [GIS](gis/README.md)
* [IOT](iot/README.md)
* [meta](meta/README.md)
* [Reverse proxy](proxy/README.md)

## Kom igang

1. Klon dette repository `git clone https://github.com/frederiksberg/prod-app1-deployment.git`
2. Tilret `*.env`under de forskellige projekter (brug evt. `init_env.sh` script)
3. Container specifik init (se [dette afsnit](#konfiguration))
4. Kør `make` i root folderen
5. Kør `/proxy/init.sh` for at sætte SSL op
6. Genstart med `make restart`

### Make

Til at styrer docker-compose filerne bruges [Make](https://www.gnu.org/software/make/), som gør det muligt at bygge, fjerne,  starte eller stoppe produktionsserveren eller dele af den afhængigt af hvor man står i foldertræet. Som udgangpunkt har alle services en `docker-compose.yml` og en tilhørnede `Makefile` som kalder docker-compose filen med forskellige kommandoer.

* `make deploy` - Starter containerne i detatch mode (`docker-compose up -d`)
* `make run` - Starter containerne med live logging (`docker-compose up`)
* `make build` - Bygger images fra `Dockerfile` (`docker build -t frbsc/img_name:latest -f ./Dockerfile .`)
* `make kill` - Lukker containere(`docker-compose down`)
* `make clean` - Fjerner containere (`docker-compose rm -f`)

Ovennævnte build targets har interne afhængigheder for at lette workflowet.

Deploy er standard target, dvs. at køres make uden at specificere et target, køres deploy.

Dette kombineret betyder at, der kan startes en service ved at køre `make` fra servicens rod.
Og at servicen kan tages ned, bygges om og spinnes op igen ved at køre `make`.

Når en applikation debugges kand et være en fordel at køre `make run`. Servicen vil nu løbende printe log output til terminalen. Vær dog opmærksom på at servicen ikke længere køres som en daemon, og at dens life-cycle derfor er tied ttil terminalen.

### Make i roden

Makefilen i roden af repositoriet er sat op til at styre deployment af hele serveren.
Her er specificeret de samme targets som i de individuelle projekt, samt et `restart` target.

## Opgradering

Se guiden for [Blue-green deployment](tutorials/blue-green.md) og [Live opgradering](tutorials/upgrade-service.md).

## Konfiguration

Flere af containerne er 

## Sikkerhed

## God stil

* Commit ikke kode fra serveren til Github

## Digital Ocean

Vi kører setuppet i DigitalOcean.

Produktionsdelen af setuppet består af en produktions server og en produktionsdatabase.

Produktionserveren har en floating ip, som dns'en peger på. Dette gør at produktionsserveren kan flyttes ved plot at pege denne ip over på en ny droplet.

Produktionsdatabasen db1 er hostet i DO's managed database cluster engine. Det vil sige at vi ikke selv administrerer lav-praktisk tuning og vedligeholdelse.

Det er muligt at tilføje standby og read-only nodes til clusteren, hvis nedetid bliver et problem.

## Videre udvikling

Ønskes der at tilføjes services til stacken 
* lav `docker-compose.yml` og tilhørende `Makefile`
* Hvis servicen er en webapplikation tilføjes docker netværket til `docker-compose.yml`:
```yml
services:
    ...
    networks:
      - front-facing
networks:
  front-facing:
    external: true
    name: proxy
```
* Tilføj den nye service i parent `Makefile`. Se [eksempel](https://github.com/frederiksberg/prod-app1-deployment/blob/master/gis/Makefile)
* Opret nginx konfiguration under [`proxy/confs/`](https://github.com/frederiksberg/prod-app1-deployment/tree/master/proxy/confs)
* Tilføj services til [`proxy/init.sh`](https://github.com/frederiksberg/prod-app1-deployment/blob/master/proxy/init.sh)
* Gentart servicen 

## Requirements

For at benytte setup kræves flgn. installeret på serveren.

* Docker
* docker-compose
* cmake

De enkelte projekter kan have yderligere dependencies.


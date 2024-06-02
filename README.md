# Nextcloud Implementatie met Ansible en Docker

*Door: Turgay Yasar*

Dit project automatiseert de implementatie van Nextcloud met behulp van Ansible en Docker. De playbook configureert het systeem, zal de noodzakelijke software installeren en zal er voor zorgen dat Nextcloud bereikbaar is via Traefik met SSL-ondersteuning voor domeinnamen.

## Vereisten

- Een werkende installatie van Ansible
- Toegang tot een machine met root-rechten
- Een werkende internetverbinding voor het downloaden van paketten, Docker-images

## Structuur van de map

```bash
Nextcloud/
├── hosts.ini
├── playbook.yml
├── README.md
├── roles/
│   ├── change_hostname/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── firewall_role/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── install_docker/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── install_nextcloud/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── templates/
│   │   │   ├── docker-compose.yml.j2
│   │   │   └── traefik.yml.j2
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── set_static_ip/
│   │   ├── handlers/
│   │   │   └── main.yml
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── traefik_requirements/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   ├── tests/
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars/
│   ├── uninstall_nginx/
│   │   ├── tasks/
│   │   │   └── main.yml
│   │   └── tests/
│   │       ├── inventory
│   │       └── test.yml
│   ├── update_system_packages/
│       ├── tasks/
│       │   └── main.yml
│       └── tests/
│           ├── inventory
│           └── test.yml
├── template.env.j2
├── template_traefik.env.j2
├── traefik/
│   ├── acme.json
│   └── docker-compose.yml
└── vars.yml
```

## Installatie instructies

1) Voer de twee volgende commando's uit om SHH-toegang in te stellen op de doelmachine:

```bash
ssh [user]@[IP-adres] 'mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R 700 ~/.ssh'
cat ~/.ssh/id_rsa.pub | ssh [user]@[IP-adres] 'cat >> ~/.ssh/authorized_keys'
```

2) Bewerk het hosts.ini bestand om het IP-adres van de doelmachine in te vullen:

```bash
[testuser]
[IP-address] #ansible_user=root
```

3) bewerk de variabelen in het centraal beheerde vars.yml bestand naar de gewenste waarden:

```yml
change_hostname_desired_hostname: ""
network_interface: "ens160" #standard for Rocky Linux
ip_address: ""
subnet_mask: ""
gateway: ""
dns_servers: ""
domain_name: ""
traefik_email: ""
mysql_root_password: ""
mysql_password: ""
mysql_database: ""
mysql_user: ""
domain_name_traefik: ""
```

4) voer het playbook uit met het volgende commando

```bash
ansible-playbook -i hosts.ini playbook.yml
```

## Beschrijving van de rollen

- change_hostname: Wijzigt de hostnaam van de doelmachine.
- firewall_role: Configureert de firewall om HTTP en HTTPS verkeer toe te staan.
- install_docker: Installeert Docker en Docker Compose.
- install_nextcloud: Installeert Nextcloud met behulp van Docker Compose.
- set_static_ip: Stelt een statisch IP-adres in op de doelmachine.
- traefik_requirements: Configureert Traefik en zorgt ervoor dat Nextcloud via Traefik wordt aangeboden.
- uninstall_nginx: Verwijdert NGINX om conflicten met Traefik te voorkomen.
- update_system_packages: Werk systeem pakketten bij naar de nieuwste versies.

## Aangepaste Templates

- template.env.j2: Bevat omgevingsvariabelen voor Nextcloud.
- template_traefik.env.j2: Bevat omgevingsvariabelen voor Traefik.

deze worden nog steeds centraal beheerd door de vars.yml en dus hoeft de gebruiker geen rekening te houden met deze bestanden.

## Bekend Probleem

Af en toe kan het gebeuren dat de website niet bereikbaar is, zelfs als alles correct draait via Docker. Om dit op te lossen, volg je deze stappen:

1) Ga naar de ansible-Nextcloud folder.
2) Herstart Traefik met het volgende commando:
```bash
cd /traefik
docker compose down -v
docker compose up -d --build
```
3) Herstart ook de andere twee services die verbonden zijn met de website.
```bash
cd ..
docker compose down -v
docker compose up -d --build
```

## Extra's

- Traefik Setup: Traefik is ingesteld als een reverse proxy om verkeer naar de Nextcloud-containers te leiden.
- SSL Certificaat: Traefik zorgt voor SSL-certificaten voor de website, waardoor veilige HTTPS-verbindingen mogelijk zijn. Dit wordt geregeld via de acme.json configuratie in de traefik map.
# Steps:


## 1. Install Docker Desktop

```
https://docs.docker.com/desktop/install/windows-install/
```

## 2. Download and compile

```bash
git clone https://github.com/eabris/symfony-6-docker-skeleton.git
cd symfony-6-docker-skeleton
docker-compose up -d
```

## 3. Run

Open in browser: 

```
http://localhost:8812
```

Run terminal commands in the container:

```bash
docker exec -it testserver-symfony bash
# run in bash
composer install
php bin/console doctrine:migrations:migrate
```

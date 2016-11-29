# Ghost + Nginx + Lets Encrypt (optional)

![ghost.png](https://github.com/hadd0ck/GhostBlogDocker/raw/master/ghost.png)

Setup to deploy quickly your blogging platform:
- Ghost: Easy way to publish blog posts
- docker-compose: portable and powerful tool used to deploy, run multiple docker containers in one command-line. 
- Nginx: a web proxy with a free SSL certificate via LetsEncrypt (or import your own Certificate).

Notes: 
- Ghost official docker image for developers is ready for test environment but not for production.
This Docker image simplify the use in production.


### 1. Prerequisite
- Linux (Ubuntu in this case)
- Docker & Docker-compose
- A Domain (Blogciso.com in this example) pointing to the server IP

### 2. Build and run

Copy code from repo:

    git clone https://github.com/hadd0ck/GhostBlogDocker blog && cd blog
    
    
Setup SSL:

    mkdir -p /etc/ssl/private /etc/ssl/certs && openssl dhparam -out /etc/ssl/private/dhparams_4096.pem 4096     <-- Diffie Hellman Key Exchange to improve security (this might take a while to generate)
    
Before generating the certificate with letsencrypt, make sure the Domain Name is pointing to the new server builded

Use letsencrypt to generate and get the certificate (replace domain and email with your own)

    docker run -it --rm -p 443:443 -p 80:80 --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "/var/lib/letsencrypt:/var/lib/letsencrypt" quay.io/letsencrypt/letsencrypt:latest certonly --standalone --domain blogciso.com --email romain.braud@me.com --quiet --noninteractive --rsa-key-size 4096 --agree-tos --standalone-supported-challenges http-01

Or you can import your own certificate in the folder below:

    /etc/ssl/certs/your_domain.crt /etc/ssl/private/your_domain.key

Or create a selfsigned for test purpose:

    sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/selfsigned.key -out /etc/ssl/certs/selfsigned.crt 

Edit configs with your settings:
     
    nano ghost/config.js <-- replace blog url & email details
    nano nginx/blog.conf <-- replace server_name & ssl_certificate & ssl_certificate_key
    nano docker-compose.yml <-- replace in the paths with your domain name

Run the Docker containers

    docker-compose up -d --build

### 3. Administration

Open the link in the browser and setup the admin user 

    https://blogciso.com/ghost/setup/one/

To add theme you can use the GUI under settings/general 

or

it's possible from the command line directly from the server like the example below

    git clone https://github.com/phongtruongg/Cle templates/Cle

Copy in ghost & restart
 
    docker cp templates/Cle blog_ghost_1:/var/lib/ghost/themes/

Now template Cle is available under settings/general


### 4. Backup and restore

Simply backup the folder /var/lib/ghost while the ghost container is stopped (for data persistency).

With script:

    scripts/backup.sh

With crontab:

```
# Backup Ghost Blog: daily at 12:00 (noon)
00 12 * * * /bin/bash -c "docker stop blog_ghost_1 && tar -zcvf /root/backup/ghost/ghost-$(date +\%A).tar.gz -C /var/lib/docker/volumes/blog_ghost/_data/ . && docker start blog_ghost_1"

# Backup Ghost Blog: weely, monday at 01:00
00 01 * * 1 /bin/bash -c "docker stop blog_ghost_1 && tar -zcvf /root/backup/ghost/ghost-$(date -I).tar.gz -C /var/lib/docker/volumes/blog_ghost/_data/ . && docker start blog_ghost_1"
```

### 5. LetsEncrypt

LetsEncrypt give you for free a certificate valid for 3 months. Many company migrated to this open CA because of price and security. The only way you can get a certificate for a domain is if this domain targets the ip where you run the command. So it means that you control the DNS and the server, that's the requirement for certificate delivery.

To update your certificate, you can do manually:

    docker-compose stop
    
    docker run -it --rm -p 443:443 -p 80:80 --name certbot -v "/etc/letsencrypt:/etc/letsencrypt" -v "/var/lib/letsencrypt:/var/lib/letsencrypt" quay.io/letsencrypt/letsencrypt:latest certonly --standalone --domain blogciso.com --email romain.braud@me.com --quiet --noninteractive --rsa-key-size 4096 --agree-tos --standalone-supported-challenges http-01
    
    docker-compose up -d --build

Or a script in a crontab.

The email you registered the certificate will be warn few weeks before the vertification will expired.

To force renew, a way before the expiration date, use the flag: --force-renewal

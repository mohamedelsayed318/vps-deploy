# Connecting to the VPS

To connect your VPS server, you can use your server IP, you can create a root password and enter the server with your IP address and password credentials. But the more secure way is using an SSH key.

## Creating SSH Key

### For MAC OS / Linux / Windows 10 (with openssh)

1. Launch the Terminal app.
2. ```ssh-keygen -t rsa```
3. Press ```ENTER``` to store the key in the default folder /Users/lamadev/.ssh/id_rsa).

4. Type a passphrase (characters will not appear in the terminal).

5. Confirm your passphrase to finish SSH Keygen. You should get an output that looks something like this:

``` Your identification has been saved in /Users/lamadev/.ssh/id_rsa.
Your public key has been saved in /Users/lamadev/.ssh/id_rsa.pub.
The key fingerprint is:
ae:89:72:0b:85:da:5a:f4:7c:1f:c2:43:fd:c6:44:30 lamadev@mac.local
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|         .       |
|        E .      |
|   .   . o       |
|  o . . S .      |
| + + o . +       |
|. + o = o +      |
| o...o * o       |
|.  oo.o .        |
+-----------------+ 
```
6. Copy your public SSH Key to your clipboard using the following code:
```
pbcopy < ~/.ssh/id_rsa.pub
```

7. make ssh key for github

```
ssh-keygen -t ed25519 -C "mohamed@prop.support"
```

8. copy this to github 
```
clip < ~/.ssh/id_ed25519.pub
```

## First Configuration

### Deleting apache server

```
systemctl stop apache2
```

```
systemctl disable apache2
```

```
apt remove apache2
```

to delete related dependencies:
```
apt autoremove
```

### Cleaning and updating server
```
apt clean all && sudo apt update && sudo apt dist-upgrade
```

```
rm -rf /var/www/html
```

### Installing Nginx

```
apt install nginx
```

### Installing and configure Firewall

```
apt install ufw
```

```
ufw enable
```

```
ufw allow "Nginx Full"
```

## First Page

#### Delete the default server configuration

```
 rm /etc/nginx/sites-available/default
```

```
 rm /etc/nginx/sites-enabled/default
```

#### First configuration
```
 nano /etc/nginx/sites-available/netflix
```
```
server {
  listen 80;

  location / {
        root /var/www/netflix;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }
}

```

```
ln -s /etc/nginx/sites-available/netflix /etc/nginx/sites-enabled/netflix

```

##### Write your fist message
```
nano /var/www/netflix/index.html

```

##### Start Nginx and check the page

```
systemctl start nginx
```

##### Restart Nginx
```
systemctl reload nginx
```

##### Nginx Status 
```
service nginx status
```

##### Nginx syntax test 
```
nginx -t
```

## Uploading Apps Using Git

```
apt install git
```

```
mkdir netflix
```
```
cd netflix
```

```
git clone <your repository>
```

## Nginx Configuration for new apps
```
nano /etc/nginx/sites-available/netflix
```
```
location /api {
        proxy_pass http://45.90.108.107:8800;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
  }
```

##### If you check the location /api you are going to get "502" error which is good. Our configuration works. The only thing we need to is running our app

```
apt install nodejs
```

```
apt install npm
```

```
cd api
```
```
npm install
```
```
nano .env
```
##### Copy and paste your env file
```
node index.js
```

#### But if you close your ssh session here. It's gonna kill this process. To prevent this we are going to need a package which is called ```pm2```
```
npm i -g pm2
```
Let's create a new pm2 instance

```
pm2 start --name api index.js   
```
```
pm2 startup ubuntu 
```

## React App Deployment

```
cd ../client
```

```
nano .env
```
Paste your env file.

```
npm i
```
Let's create the build file

```
npm run build
```

Right now, we should move this build file into the main web file

```
rm -rf /var/www/netflix/*
```
```
mkdir /var/www/netflix/client
```

```
cp -r build/* /var/www/netflix/client
```

Let's make some server configuration
```
 location / {
        root /var/www/netflix/client/;
        index  index.html index.htm;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        try_files $uri $uri/ /index.html;
  }

```
### Adding Domain
1 - Make sure that you created your A records on your domain provider website.

2 - Change your pathname from Router

3 - Change your env files and add the new API address 

4 - Add the following server config
```
server {
 listen 80;
 server_name safakkocaoglu.com www.safakkocaoglu.com;

location / {
 root /var/www/netflix/client;
 index  index.html index.htm;
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection 'upgrade';
 proxy_set_header Host $host;
 proxy_cache_bypass $http_upgrade;
 try_files $uri $uri/ /index.html;
}
}

server {
  listen 80;
  server_name api.safakkocaoglu.com;
  location / {
    proxy_pass http://45.90.108.107:8800;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }
}

server {
  listen 80;
  server_name admin.safakkocaoglu.com;
  location / {
    root /var/www/netflix/admin;
    index  index.html index.htm;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    try_files $uri $uri/ /index.html;
  }
}
```

## SSL Certification
```
apt install certbot python3-certbot-nginx
```

Make sure that Nginx Full rule is available
```
ufw status
```

```
certbot --nginx -d example.com -d www.example.com
```

Let’s Encrypt’s certificates are only valid for ninety days. To set a timer to validate automatically:
```
systemctl status certbot.timer
```

## Add next js to pm2 
```
pm2 start npm --name "myapp" -- start
```



##### Sample of nginx config file that have 4 things (Server, Frontend with nextjs, Dashboard with nextjs, Test or Dev with nextjs) with cerbot(ssl certificate)


```
server {
  server_name alsmsar.com www.alsmsar.com;
  location / {
    proxy_pass http://195.26.251.157:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/alsmsar.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/alsmsar.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}

server {
  server_name api.alsmsar.com www.api.alsmsar.com;
  location / {
    proxy_pass http://195.26.251.157:4000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/alsmsar.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/alsmsar.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}

server {
  server_name dashboard.alsmsar.com www.dashboard.alsmsar.com;
  location / {
    proxy_pass http://195.26.251.157:5000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
    }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/alsmsar.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/alsmsar.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}
server {
    if ($host = www.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  server_name alsmsar.com www.alsmsar.com;
    listen 80;
    return 404; # managed by Certbot




}

server {
    if ($host = www.api.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = api.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  listen 80;
  server_name api.alsmsar.com www.api.alsmsar.com;
    return 404; # managed by Certbot




}

server {
    if ($host = www.dashboard.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = dashboard.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  server_name dashboard.alsmsar.com www.dashboard.alsmsar.com;
    listen 80;
    return 404; # managed by Certbot




}


server {
  server_name dev.alsmsar.com www.dev.alsmsar.com;
  location / {
    proxy_pass http://195.26.251.157:7000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }


    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/dev.alsmsar.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/dev.alsmsar.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot


}


server {
    if ($host = www.dev.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = dev.alsmsar.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


  server_name dev.alsmsar.com www.dev.alsmsar.com;
    listen 80;
    return 404; # managed by Certbot




}
```

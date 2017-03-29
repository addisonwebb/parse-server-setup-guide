# Setup [Parse Dashboard](https://github.com/ParsePlatform/parse-dashboard)

### Setup VPS for Parse Dashboard
1. Create new droplet for parse dashboard
  - Ubuntu 16.04.2
  - 512 MB
  - 20 GB SSD
  - NY1/2/3
  - name: dashboard.goexploremichigan.com
2. Wait for server to be setup
3. Copy server ip address

### Secure Server
*https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-16-04*

1. `ssh root@[server ip address]`
2. Get password for new server from email sent by Digital Ocean
3. Create a new root password (Make sure you save this in a password manager!)
4. `adduser michigander`
5. Create a password for the new user (Make sure you save this in a password manager as well!)
  - skip through (hit enter) the rest of the steps, they are not required
6. `usermod -aG sudo michigander`

**Add SSH Key**
*Note: I assume you have a ssh key setup on your computer.*
1. `su - michigander`
2. `mkdir ~/.ssh`
3. `chmod 700 ~/.ssh`
4. `vim ~/.ssh/authorized_keys`
5. On your local machine: `pbcopy < ~/.ssh/id_rsa.pub`
6. Insert mode: `i`
7. Paste the your public key
8. Save: `esc`, `:wq`, `enter`
9. `chmod 600 ~/.ssh/authorized_keys`
10. `exit`

**Disable Password Authentication**
1. `sudo vim /etc/ssh/sshd_config`
2. Insert mode: `i`
3. Verify:
  - PasswordAuthentication no (this should be the only one you have to change)
  - PubkeyAuthentication yes
  - ChallengeResponseAuthentication no
4. Save: `esc`, `:wq`, `enter`
5. `sudo systemctl reload sshd`

**Test SSH Login**

In a new terminal tab do the following:
1. `ssh michigander@[server IP address]`

### Point Domain DNS at Server (optional)
1. Create a new A record in your domain name's DNS settings and set it to your server's IP address
  - Example: Record Type: A, Name: test, Value: [server's ip address], TTL: 600 seconds
2. Now we can test logging into our server with our custom domain name `ssh michigander@dashboard.goexploremichigan.com`

*Note: It may take some time for the DNS settings to update.*

### Install Let's Encrypt SSL Certificate (optional)
1. `sudo apt-get install letsencrypt`
2. `sudo letsencrypt certonly --standalone -d dashboard.goexploremichigan.com`
3. Enter in email address
4. Agree to terms

### Install NPM
- `sudo apt-get update`
- `sudo apt install npm`

### [Install Node with PPA](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-16-04)
1. `curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh`
2. `sudo bash nodesource_setup.sh`
3. `sudo apt-get install nodejs`
4. `sudo apt-get install build-essential`

### Install Parse Dashboard
1. `sudo npm install -g parse-dashboard`

### Setup Parse Dashboard Config file
1. `touch parse-dashboard-config.json`
2. `vim parse-dashboard-config.json`
3. Insert mode: `i`
4. Copy and paste the example config contents into the new file
5. Save: `esc`, `:wq`, `enter`

**Plain Text Password Example**
```
{
  "apps": [{
      "serverURL": "https://test.goexploremichigan.com/parse",
      "appId": "exploremichigan",
      "masterKey": "9B04B992-D07A-4CEC-A6BF-49018D556929",
      "appName": "Explore Michigan",
      "production": false
    }],
  "users": [
    {
      "user":"admin1",
      "pass":"super-not-secure-password1"
    },
    {
      "user":"admin2",
      "pass":"super-not-secure-password2"
    }
  ],
  "useEncryptedPasswords": false
}
```

**Hashed Password Example**
```
{
  "apps": [{
      "serverURL": "https://test.goexploremichigan.com/parse",
      "appId": "exploremichigan",
      "masterKey": "9B04B992-D07A-4CEC-A6BF-49018D556929",
      "appName": "Explore Michigan",
      "production": false
    }],
  "users": [
    {
      "user":"admin1",
      "pass":"$0y$00$G9ZWiVC/8TpOaAogVzYJ2aVf5fsuArGRDYsjPpRb2sQ1C.7gwJr1e"
    },
    {
      "user":"admin2",
      "pass":"$0y$00$d.d66KnIvX0NcWL9J5PumufLrCsbJYxI7rkOsoyikSgfTlNWmnUTG"
    }
  ],
  "useEncryptedPasswords": true
}
```

*Tip: You can use https://www.bcrypt-generator.com to generate a hash of your password to store in the config file.*

### [Install Nginx](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04)
1. `sudo apt-get install nginx`

### Update Nginx Sites Enabled
1. `sudo vim /etc/nginx/sites-enabled/default`
2. Delete the contents of the file: `dG`
3. Insert mode: `i`
4. Copy and paste in example below
5. Save: `esc`, `:wq`, `enter`
6. Reload nginx: `sudo service nginx reload`

```
# HTTP - redirect all requests to HTTPS
server {
    listen 80;
    listen [::]:80 default_server ipv6only=on;
    return 301 https://$host$request_uri;
}

# HTTPS - serve HTML from /usr/share/nginx/html, proxy requests to /parse/
# through to Parse Server
server {
        listen 443;
        server_name dashboard.goexploremichigan.com;

        root /usr/share/nginx/html;
        index index.html index.htm;

        ssl on;
        # Use certificate and key provided by Let's Encrypt:
        ssl_certificate /etc/letsencrypt/live/dashboard.goexploremichigan.com/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/dashboard.goexploremichigan.com/privkey.pem;
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;
        ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

        # Pass requests for /parse/ to Parse Server instance at localhost:1337
        location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:4040/;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
                proxy_redirect off;
        }

        #location / {
         #       try_files $uri $uri/ =404;
        #}
}
```

### Start Parse Dashboard
1. Start: `parse-dashboard --config parse-dashboard-config.json`
2. Verify dashboard is working: dashboard.goexploremichigan.com

[<- Previous: Setup Parse Server](./setup-parse-server.md) --- [Home](./readme.md)

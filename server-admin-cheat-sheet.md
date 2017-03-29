# Parse Server Admin Cheat Sheet
This cheat sheet provides a quick reference for common tasks that you might need to perform while maintaining your Parse server.

### Start Parse Server (Example)
- Start: `parse-server`
- Install/Update: `sudo npm install -g parse-server`
- Version: `npm list parse-server`

### Parse Dashboard
- Start parse dashboard: `parse-dashboard --config parse-dashboard-config.json`
- Install/Update: `sudo npm install -g parse-dashboard`
- Version: `npm list parse-dashboard`

### TMUX
- open tmux window: `tmux attach`
- detach: `ctrl` + `b`, `d`

### Generating Master/API Keys
`uuidgen`

Example output: `1BF7EED7-5F8E-4A31-B9B6-5D0E8726C074`

### Updating Server Packages
SSH into servers and run the following commands.

1. `sudo apt-get update`
2. `sudo apt-get upgrade`
3. `sudo apt-get dist-upgrade`

Could also use:

> `sudo apt-get update && sudo apt-get upgrade && sudo apt-get dist-upgrade`

Updating will likely require user interaction to verify updates should be installed. e.g.

> `Need to get 68.6 MB of archives.
After this operation, 296 MB of additional disk space will be used.
Do you want to continue? [Y/n]`

After running the update commands restart the server. (See Server Reboot)

**Notes**
- http://askubuntu.com/questions/196768/how-to-install-updates-via-command-line

### Renewing Letsencrypt Certs
1. Stop Nginx - `sudo service nginx stop`
2. Renew cert - `sudo letsencrypt renew`
3. Start Nginx - `sudo service nginx start`

**Notes**
- https://github.com/certbot/certbot/issues/2750#issuecomment-238983818

### Server Reboot
`sudo reboot`

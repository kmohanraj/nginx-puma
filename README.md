## Rails Puma and Nginx Setup
#### 1.Nginx Setup:

install the following key,
```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys ABF5BD827BD9BF62
```
THEN, add teh line to end of the file ``/etc/apt/sources.list ``
```
deb http://nginx.org/packages/ubuntu/ precise nginx
```

NOTE, if you have already installed `` Nginx `` before, make remove it.
```
sudo apt-get purge nginx*
```
THEN, update and install the package,
```
sudo apt-get update

sudo apt-get install nginx
```
THEN, check your nginx version,
```
nginx -v
```

THEN, start the daemon using Upstart,
```
sudo service nginx start
```

#### 2.Configuration:

FIRTS, remove the `default` conf in both sites-available and sites-enabled.
```
sudo rm /etc/nginx/sites-available/default

sudo rm /etc/nginx/sites-enabled/default
```

THEN, create new app conf in ` sites-availabe/app_name`
```
sudo vi /etc/nginx/sites-available/app_name or app_name.conf
```

THEN, paste the following config,
```

upstream app_name {
  server unix:///var/run/app_name.sock;
}

server {
  listen 80;
  server_name app_name_url.com or ip; # change to match your URL
  root /var/www/my_app/public; # I assume your app is located at this location

  location / {
    proxy_pass http://my_app; # match the name of upstream directive which is defined above
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  location ~* ^/assets/ {
    # Per RFC2616 - 1 year maximum expiry
    expires 1y;
    add_header Cache-Control public;

    # Some browsers still send conditional-GET requests if there's a
    # Last-Modified header or an ETag header even if they haven't
    # reached the expiry date sent in the Expires header.
    add_header Last-Modified "";
    add_header ETag "";
    break;
  }
}
```

THEN, need to enable it by creating	symlink in `/etc/nginx/site-enabled`
```
sudo ln -sf /etc/nginx/sites-available/app_name /etc/nginx/sites-enabled/app_name
```
THEN. start nginx web server,
```
sudo service nginx restart
```

THEN, start your app server, before inside the application folder, if use `daemon` use -d 
```
bundle exec puma -e production -d 
 (or) 
RAILS_ENV=production bundle exec puma -d 
```

###### http://ruby-journal.com/how-to-setup-rails-app-with-puma-and-nginx/

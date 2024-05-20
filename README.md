# tools.ramvasuthevan.ca

I want to start posting more of the tools that I use. 

I decided to start with [JSON Crack](https://github.com/AykutSarac/jsoncrack.com), which I am hosting at [jsoncrack.tools.ramvasuthevan.ca](http://jsoncrack.tools.ramvasuthevan.ca).

1) I need to write down what I did to setup this server

dogsheep.ramvasuthevan.ca

Set up [Dogsheep](https://simonwillison.net/2021/Aug/22/weeknotes-dogsheep/) using these [instructions](https://simonwillison.net/2021/Aug/22/weeknotes-dogsheep/)

1. Get up Reserve IP and add A record to CloudFlare DNS
2. `apt-get install python3 python3-venv nginx sqlite -y` (I am running Ubuntu 24 and these instructions are for Ubuntu 20) (Appreantly, [sqlite](https://packages.ubuntu.com/search?keywords=sqlite&searchon=names&suite=focal&section=all) is for sqlite2)
3. apt install certbot python3-certbot-nginx -y, then certbot --nginx -d dogsheep.ramvasuthevan.ca
4. I don't think I need to do step 4
5. Ran `useradd -s /bin/bash -d /home/dogsheep/ -m -G users dogsheep` (Need to add to -G, so just added to users)
6. `apt install python3-pip` (Do I need pip?), `apt install pipenv`. `mkdir datasette`, `cd datasette`, `pipenv install wheel`, `pipenv install datasette datasette-auth-passwords`
7. Created a /etc/systemd/system/datasette.service file with this [contents](https://gist.github.com/simonw/0653b6177c6f12caa16530da4c56646f). Use [python3 -c 'import secrets; print(secrets.token_hex(32))'](https://docs.datasette.io/en/0.60/settings.html#:~:text=%24%20python3%20-c%20'import%20secrets%3B%20print(secrets.token_hex(32))'%20) to generate `DATASETTE_SECRET`. I had to make change to the confs.  I updated to create just test.db
8. I had to reinstall sqlite3 ?? `sqlite3 test.db 'VACUUM;'`
9. service datasette start
10. Configured NGINX to proxy to localhost port 8001, using this configuration. Needed to remove daemon off; `sudo nginx -t` `sudo systemctl restart nginx`
/etc/systemd/system/datasette.service

[Unit]
Description=Datasette
After=network.target

[Service]
Type=simple
User=root
Environment=DATASETTE_SECRET=07286d504947909ee8bd9d2ef998f8bbfbac2ba21dcc8335a7aaf26c0b1f2445
WorkingDirectory=/root/datasette
ExecStart=/usr/bin/pipenv run datasette serve -h 127.0.0.1 -p 8000 \
  -m /datasette/metadata.yml \
  --setting sql_time_limit_ms 20000 \
  --setting facet_time_limit_ms 5000 \
  --setting template_debug 1 \
  --setting force_https_urls 1 \
  /root/datasette/test.db
Restart=on-failure

[Install]
WantedBy=multi-user.target


/etc/nginx/sites-available/datasette 
server {
    listen 80;
    server_name 159.203.50.142;

    location / {
        proxy_pass http://127.0.0.1:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
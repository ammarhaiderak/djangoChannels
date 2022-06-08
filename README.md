# Production Env Configurations for Django Channels with Daphne, gunicorn and Nginx


There are not many straight forward solutions out there on how to properly use **wss** websockets. 
Everything would work on simple **ws** but you might be getting "connection failed" when trying to use same websockets over https. 

So here is how I managed to use wss.

1. Use gunicorn for http requests i.e. use gunicorn to serve wsgi. <br />
e.g. ```gunicorn yourproject.wsgi:application --bind 0.0.0.0:8000```

2. Use daphne for websockets i.e use daphne to serve asgi. I am using letsencrypt certificates, you can use certbot for that. <br />
    i) you need to make sure you install the Twisted http2 and tls extras: <br />
    ```pip install -U 'Twisted[tls,http2]'```
   
    command: <br />
   ```sudo path_to_your_project_dir/venv/bin/python path_to_your_project_dir/venv/bin/daphne -e  ssl:8001:privateKey=/etc/letsencrypt/live/yourwebsite.com/privkey.pem:certKey=/etc/letsencrypt/live/yourwebsite.com/fullchain.pem yourproject.asgi:application```

  ps: you may use unixsockets for this
  
3. nginx config <br />
   command: <ins> sudo nano /etc/nginx/sites-enabled/default or yourwebsite.com </ins>
   use reverse proxy for webscocket endpoint
   update your server part like this: 
   ```
    server {
     ...
     location /ws/ {
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          proxy_redirect off;
          proxy_pass http://127.0.0.1:8001;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header X-Forwarded-Host $server_name;
     } 
     ...
   }
   ```
4. connection at frontend:
  ```js
  const socketUrl = `wss://yourwebsite.com:8001/ws/notify/${notifyToken}/`;
  console.log('going to connect to socket')
  const socket = new WebSocket(socketUrl);
  ```


  
please feel free reach out in case of any issues / questions. <br />
ammarkhaliq@gmail.com

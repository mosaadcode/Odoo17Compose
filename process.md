sudo apt update && sudo apt upgrade -y
# Add Docker's official GPG key:
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

docker compose version

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker
docker run hello-world

sudo systemctl enable docker
sudo systemctl start docker

docker network create --driver bridge --subnet=10.1.1.0/24 ourweb


docker volume create portainer_data

docker run -d \
  -p 9000:9000 \
  --name portainer \
  --restart=always \
  --network ourweb \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v portainer_data:/data \
  portainer/portainer-ce:2.20.1

# and this is for windows ///////
docker run -d ^
  -p 9000:9000 ^
  --name portainer ^
  --restart=always ^
  --network ourweb ^
  -v //var/run/docker.sock:/var/run/docker.sock ^
  -v portainer_data:/data ^
  portainer/portainer-ce:2.20.1
# ///////////////////////////////////////////
sudo mkdir -p /opt/ourcode/npm/{data,letsencrypt}
<!-- sudo chown -R $USER:$USER /opt/ourcode/npm/{data,letsencrypt}
sudo chown -R $USER:$USER /opt/ourcode -->

version: "3.9"
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: n-p-m
    restart: always
    ports:
      - 80:80
      - 81:81
      - 443:443
    environment:
      - TZ=Africa/Cairo

    volumes:
      - /opt/ourcode/npm/data:/data
      - /opt/ourcode/npm/letsencrypt:/etc/letsencrypt
    networks:
      ourweb:

networks:
  ourweb:
    external: true
    name: ourweb



# for Django applications we do the follwoing steps

# 1 - Create voulmes for static files and media

docker volume create static_volume
docker volume create media_volume

# 2 - nginx proxy manager stack with static and media volumes like this

version: "3.9"
services:
  npm:
    image: jc21/nginx-proxy-manager:latest
    container_name: n-p-m
    restart: always
    ports:
      - 80:80
      - 81:81
      - 443:443
    environment:
      - TZ=Africa/Cairo
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
      - /opt/ourcode/npm/data:/data
      - /opt/ourcode/npm/letsencrypt:/etc/letsencrypt
    networks:
      ourweb:
networks:
  ourweb:
    external: true
    name: ourweb
volumes:
  static_volume:
    external: true
  media_volume:
    external: true


# 3- make dockerfile in App directory like this 

FROM python:3.11-alpine                    
ENV PYTHONBUFFERED=1                    
ENV PORT 8000                          
WORKDIR /app
COPY . /app/                            
RUN pip install --upgrade pip
RUN pip install -r requirements.txt     
# Run app with gunicorn command
CMD gunicorn smart_mfis.wsgi:application --bind 0.0.0.0:"${PORT}"

EXPOSE ${PORT}

# 4 - biuld an image with this command

docker build -t IMAGENAME:latest .



# 5 - run stack for applications in portainer like this 

version: '2'
services:
  smart:
    image: mfis:latest
    container_name: smart
    # user: root (not recommended for production)
    networks:
      - ourweb
    ports:
      - 8000:8000
    # environment variables can be uncommented if needed
    # - POSTGRES_USER=odoo
    # - POSTGRES_PASSWORD=odoo17@2023
    # - POSTGRES_DB=postgres
    restart: always  # run as a service
    volumes:
      - static_volume:/app/staticfiles
      - media_volume:/app/media
networks:
  ourweb:
    external: true
    name: ourweb
volumes:
  static_volume:
    external: true
  media_volume:
    external: true


# 6 - Custom Nginx Configuration for Serving Static and Media Files in Advanced Tab:

location /static/ {
    alias /app/staticfiles/;
}

location /media/ {
    alias /app/media/;
}








<!-- for exctract file -->

tar -xzf odoo-17.0+e.20240528.gz

<!-- cd /opt/ourcode/odoo/medad/addons -->
mv addons/* .

rmdir addons




sudo mkdir -p /opt/ourcode/odoo/medad/{db,db-backup,config,data,addons}
sudo chown -R $USER:$USER /opt/ourcode/odoo/medad/{db,db-backup,config,data,addons}
sudo chown -R $USER:$USER /opt/ourcode/odoo/medad



# for odoo on windows ////////
version: '2'
services:
  db:
    image: postgres:14.1-alpine
    container_name: grage-db
    # user: root
    networks:
      - ourweb
    ports:
      - 5432:5432
    environment:
      - POSTGRES_USER=odoo
      - POSTGRES_PASSWORD=odoo17@2023
      - POSTGRES_DB=postgres
    restart: always
    volumes:
      - C:/ourcode/odoo17/postgresql:/var/lib/postgresql/data

  odoo17:
    image: odoo:17.0-20240624
    container_name: grage-odoo
    # user: root
    networks:
      - ourweb
    depends_on:
      - db
    ports:
      - "10017:8069"
      - "20017:8072" # live chat
    tty: true
    command: --
    environment:
      - HOST=db
      - USER=odoo
      - PASSWORD=odoo17@2023
    volumes:
      #- /etc/timezone:/etc/timezone:ro
      #- /etc/localtime:/etc/localtime:ro
      # - ./entrypoint.sh:/entrypoint.sh   # if you want to install additional Python packages, uncomment this line!
      - C:/ourcode/odoo17/addons:/mnt/extra-addons
      - C:/ourcode/odoo17/etc:/etc/odoo
    restart: always

networks:
  ourweb:
    external: true
    name: ourweb
# ///////////////////////////////////////////////////////////////////////
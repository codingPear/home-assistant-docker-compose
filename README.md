# home-assistant-docker-compose

Already configured setup for Home Assistant with Zigbee2MQTT, Mosquitto-MQTT, and MariaDB.
Some changes are required, but outside of editing ".env" file, it should work out of the box. (it does work even if you don't change anything in the file, but the default passwords are in no way safe).

# Usage

    git clone https://github.com/codingPear/home-assistant-docker-compose.git
    cd home-assistant-docker-compose
    nano -l .env #or whatever editor you prefer
    (change default settings & save)
    docker-compose up -d
    
    
Home Assistant URL: http://< serverIP >:8123 (or http://localhost:8123 if deployed locally)
Password is the one you set up in .env file

# Config file details

# .env file

# docker-compose.yaml

  # HomeAssistant
  Setting container name and using the latest official image
  
      homeassistant:
        container_name: home-assistant
        image: homeassistant/home-assistant:latest
  
  Mounting the host config directory within the container.
  This is done in order to avoid Home Assistant reconfiguration every time there is a new container version
  
      volumes:
        # Local path where your home assistant config will be stored
        - ./config/home-assistant:/config
        - /etc/localtime:/etc/localtime:ro
  
  Setting dependencies so that those containers will start before Home Assistant
  
      depends_on:
        - mariadb # MariaDB is optional (only if you would like to use a different database for HA).
        - zigbee2mqtt  # zigbee2mqtt is optional (only if you want to add Zigbee devices and have a zigbee sniffer attached).
        - eclipse-mosquitto # aka mosquitto-mqtt is optional (only if you want to add Zigbee devices and have a zigbee sniffer attached).


  # MariaDb
  Setting up the root and Home Assistant password as they are define in .env file 

    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD}"
      MYSQL_DATABASE: ha_db
      MYSQL_USER: homeassistant
      MYSQL_PASSWORD: "${HA_MYSQL_PASSWORD}"
    
  Mounting the host config directory within the container. Similar to what's been done for Home Assistant.

    volumes:
      # Local path where the database will be stored.
      - ./store/mariadb:/var/lib/mysql
    ports:
      - 3306:3306
  
  # eclipse-mosquitto (MQTT broker)
  Mounting the config file
    
    volumes:
      - ./config/mosquitto/mosquitto.conf:/mosquitto/config/mosquitto.conf
  

  # zigbee2mqtt
  Setting timezone to the Home Assistant Default
    
    environment:
      - TZ=Europe/Amsterdam
    
    
  Mounting the host config directory within the container. Similar to what's been done for Home Assistant.
    
    volumes:
      - ./config/zigbee2mqtt:/app/data
      - /run/udev:/run/udev:ro 
    
    
    devices:
      - "${ZIGBEE_ADAPTER_TTY}:/dev/ttyZigbee"
    

# Example-python-Iothub-Postgresql

This is WIES-PaaS iothub example-code include the sso、rabbitmq、Postgresql service。

[cf-introduce Training Video](https://advantech.wistia.com/medias/ll0ov3ce9e)

[IotHub Training Video](https://advantech.wistia.com/medias/up3q2vxvn3)

## Quick Start

#### cf-cli

[https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html?source=post_page---------------------------)

#### python3

[https://www.python.org/downloads/](https://www.python.org/downloads/?source=post_page---------------------------)

#### Postgrsql

You can download pgAdmin so you can see the result in WISE-PaaS Postgresql servince instance

[https://www.postgresql.org/](https://www.postgresql.org/)

![](https://cdn-images-1.medium.com/max/2000/1*iJwh3dROjmveF8x1rC6zag.png)

python3 package(those library you can try application in local):

    #mqtt
    pip3 install paho-mqtt
    #python-backend
    pip3 install Flask

    #python postgresql library
    pip3 install sqlalchemy
    pip3 install psycopg2

## Download this file

    git clone this repository

## Login to WISE-PaaS

![Imgur](https://i.imgur.com/JNJmxFy.png)

    #cf login -skip-ssl-validation -a {api.domain_name}  -u "account" -p "password"

    cf login –skip-ssl-validation -a api.wise-paas.io -u xxxxx@advtech.com.tw -p xxxxxx

    #check the cf status
    cf target

## Application introduce

#### index.py

This is a simple backend application use flask，you can run it use `python3 index.py` and listen on [localhost:3000](localhost:3000)，and the port can get the `3000` or port on WISE-PaaS。

```py
from flask import Flask, render_template
import json
import paho.mqtt.client as mqtt
import os
import sqlalchemy

app = Flask(__name__)

# port from cloud environment variable or localhost:3000
port = int(os.getenv("PORT", 3000))


@app.route('/', methods=['GET'])
def root():

    if(port == 3000):
        return 'hello world! i am in the local'
    elif(port == int(os.getenv("PORT"))):

        return render_template('index.html')


if __name__ == '__main__':
    # Run the app, listening on all IPs with our chosen port number
    app.run(host='0.0.0.0', port=port)
```

`vcap_services` can get the application config on WISE-PaaS，it can help get the credential of your (iothub)mqtt and postgresql service instance，

we create a schema name `livingroom` and table name `temperature`，we bind this to the group name `groupfamily`

`client=mqtt.connect` can help us connect to mqtt and when we connect we will subscribe the `/hello` topic in `on_connect`，`on_message` ca n receivec the message what we send。

**Notice: You need to check the service name on WISE-PaaS Service List**

![Imgur](https://i.imgur.com/6777rmg.png)

```py

IOTHUB_SERVICE_NAME = 'p-rabbitmq'
DB_SERVICE_NAME = 'postgresql-innoworks'

# Get the environment variables
vcap_services = os.getenv('VCAP_SERVICES')
vcap_services_js = json.loads(vcap_services)

# --- MQTT(rabbitmq) ---
credentials = vcap_services_js[IOTHUB_SERVICE_NAME][0]['credentials']
mqtt_credential = credentials['protocols']['mqtt']

broker = mqtt_credential['host']
username = mqtt_credential['username'].strip()
password = mqtt_credential['password'].strip()
mqtt_port = mqtt_credential['port']

# --- Postgresql ---
credentials = vcap_services_js[DB_SERVICE_NAME][0]['credentials']
database_database = credentials['database']
database_username = credentials['username'].strip()
database_password = credentials['password'].strip()
database_port = credentials['port']
database_host = credentials['host']

POSTGRES = {
    'user': database_username,
    'password': database_password,
    'db': database_database,
    'host': database_host,
    'port': database_port,
}

schema = 'livingroom'
table = 'temperature'
group = 'groupfamily'
# connect to server
engine = sqlalchemy.create_engine('postgresql://%(user)s:\
%(password)s@%(host)s:%(port)s/%(db)s' % POSTGRES, echo=True)


engine.execute("CREATE SCHEMA IF NOT EXISTS "+schema+" ;")  # create schema

engine.execute("ALTER SCHEMA "+schema+" OWNER TO "+group+" ;")

engine.execute("CREATE TABLE IF NOT EXISTS "+schema+"."+table+" \
        ( id serial, \
          timestamp timestamp (2) default current_timestamp, \
          temperature integer, \
          PRIMARY KEY (id));")

engine.execute("ALTER TABLE "+schema+"."+table+" OWNER to "+group+";")
engine.execute("GRANT ALL ON ALL TABLES IN SCHEMA "+schema+" TO "+group+";")
engine.execute("GRANT ALL ON ALL SEQUENCES IN SCHEMA "+schema+" TO "+group+";")


# mqtt connect
def on_connect(client, userdata, flags, rc):
    print("Connected with result code "+str(rc))
    client.subscribe("/hello")
    print('subscribe on /hello')


def on_message(client, userdata, msg):

    engine.execute("INSERT INTO "+str(schema)+"."+str(table) +
                   " (temperature) VALUES ("+str(msg.payload.decode())+") ",
                   echo=True)
    print('insert sueecssful')

    print(msg.topic+','+msg.payload.decode())


client = mqtt.Client()

client.username_pw_set(username, password)
client.on_connect = on_connect
client.on_message = on_message

client.connect(broker, mqtt_port, 60)
client.loop_start()

```

we can use some api to debug our application，`/temp`(get) can show all data and `/insert`(post) can insert data into postgresql

```py
# postgresql api
@app.route('/temp', methods=['GET'])
def temp():

    numOfTempsReturned = 30

    res = engine.execute("SELECT * FROM ( SELECT * FROM "+schema+"."+table+" ORDER BY timestamp DESC LIMIT "+str(numOfTempsReturned)+") \
      AS lastRows ORDER BY timestamp ASC;")
    output = []
    for r in res:
        output.append(r)

    return str(output)


@app.route('/insert', methods=['POST'])
def insert_data():

    data = 11
    engine.execute("INSERT INTO "+str(schema)+"."+str(table) +
                   " (temperature) VALUES ("+str(data)+") ", echo=True)

    return 'insert sueecssful'



```

open **`manifest.yml`** and editor the **application name** to yours，because the appication can't duplicate，and you can bind the service rabbitmq in manifest.yml or we will bind it use commit line 。

![Imgur](https://i.imgur.com/OQegiAy.png)

Push application & Bind PostgreSQL、Rabbitmq service instance，The `-c {\"group\":\"groupfamily\"}'` bind the group we define in `index.py`

    #cf push application_name
    cf push python-demo-postgresql --no-start

    #cf bs {application_name} {service_instance_name} -c '{\"group\":\"group_name\"}'
    cf bs python-demo-postgresql postgresql -c '{\"group\":\"groupfamily\"}'

    #bind the rabbitmq service
    cf bs python-demo-postgresql rabbitmq

    #cf start {application_name}
    cf start python-demo-postgresql

**service_instance_name**
![Imgur](https://i.imgur.com/VVMcYO8.png)

get the application environment

#get the application environment
cf env {application name} > env.json

Edit the **publisher.py** `broker、port、username、password` you can find in **env.json**

```py
import paho.mqtt.client as mqtt
import random
#externalHosts
broker="xx.81.xx.10"
#mqtt_port
mqtt_port=1883
#mqtt_username
username="xxxxxxxx-b76f-43e9-8b35-xxxxx83bf941:7b166606-142c-4d00-8f8c-ab7fee64d6db"
password="xxxxxxxxbWGXpuOK5MyxMhgDk"
def on_publish(client,userdata,result):             #create function for callback
    print("data published")

client= mqtt.Client()                           #create client object

client.username_pw_set(username,password)

client.on_publish = on_publish                          #assign function to callback
client.connect(broker,mqtt_port)                                 #establish connection
client.publish("/hello",random.randint(10,30))
```

- bokrer:"VCAP_SERVICES => p-rabbitmq => externalHosts"
- port :"VCAP_SERVICES => p-rabbitmq => mqtt => port"
- username :"VCAP_SERVICES => p-rabbitmq => mqtt => username"
- password: "VCAP_SERVICES => p-rabbitmq => mqtt => password"

open two terminal

#cf logs {application name}
cf logs python-demo-postgresql

.

    python publisher.py

![https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/publish.PNG](https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/publish.PNG)

**result**

![https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/result_com.PNG](https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/result_com.PNG)

![https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/result_admin.PNG](https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/result_admin.PNG)

# Step By Step Tutorial

[https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/README.md](https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/README.md)

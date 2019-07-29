# Example-python-Iothub-Postgresql


This is WIES-PaaS iothub example-code include the sso、rabbitmq、Postgresql service。

**https://wise-paas.advantech.com/en-us**


## Quick Start

cf-cli

[https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html?source=post_page---------------------------)

python3

[https://www.python.org/downloads/](https://www.python.org/downloads/?source=post_page---------------------------)

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

    git clone this respository
    
## Login to WISE-PaaS
    #cf login -skip-ssl-validation -a {api.domain_name}  -u "account" -p "password"
    
    cf login –skip-ssl-validation -a api.wise-paas.io -u xxxxx@advtech.com.tw -p xxxxxx
    
    #check the cf status
    cf target


open **`manifest.yml`** and editor the **application name** to yours，because the appication can't duplicate。

![Imgur](https://i.imgur.com/OQegiAy.png)

open **`templates/index.html`**
    
    #change this **`python-demo-jimmy`** to your **application name**
    var ssoUrl = myUrl.replace('python-demo-jimmy', 'portal-sso');

(In `index.js` the service name need same to WISE-PaaS Service name)
![https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/servicename.PNG](https://github.com/WISE-PaaS/example-python-iothub-postgresql/blob/master/source/servicename.PNG)
![Imgur](https://i.imgur.com/6777rmg.png)

Push application & Bind PostgreSQL service instance

    #cf push application_name
    cf push python-demo-postgresql --no-start
    
    #cf bs {application_name} {service_instance_name} -c '{\"group\":\"group_name\"}' 
    cf bs python-demo-postgresql postgresql -c '{\"group\":\"groupfamily\"}'
    
    #cf start {application_name}
    cf start python-demo-postgresql

  
**service_instance_name**
![Imgur](https://i.imgur.com/VVMcYO8.png)



get the application environment
    
    #get the application environment
    cf env {application name} > env.json 
    
Edit the **publisher.py** `broker、port、username、password` you can find in **env.json**

* bokrer:"VCAP_SERVICES => p-rabbitmq => externalHosts"
* port :"VCAP_SERVICES => p-rabbitmq => mqtt => port"
* username :"VCAP_SERVICES => p-rabbitmq => mqtt => username"
* password: "VCAP_SERVICES => p-rabbitmq => mqtt => password"

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

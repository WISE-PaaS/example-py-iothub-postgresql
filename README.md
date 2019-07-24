# Example-python-Iothub


This is WIES-PaaS iothub example-code include the sso and rabbitmq service。

**https://wise-paas.advantech.com/en-us**

## Quick Start

    git clone this respository
    
    #cf login -skip-ssl-validation -a {api.domain_name}  -u "account" -p "password"
    
    cf login –skip-ssl-validation -a api.wise-paas.io -u xxxxx@advtech.com.tw -p xxxxxx
    
    #check the cf status
    cf target


open the **`manifest.yml`** and editor the application name to yours，because the appication can't duplicate。

    #cf push {application name}
    cf push python-demo-postgresql
    
    #get the application environment
    cf env python-demo-postgresql > env.json 
    
    
Edit the **publisher.py** `broker、port、username、password` you can find in env.json

* bokrer:"VCAP_SERVICES => p-rabbitmq => externalHosts"
* port :"VCAP_SERVICES => p-rabbitmq => mqtt => port"
* username :"VCAP_SERVICES => p-rabbitmq => mqtt => username"
* password: "VCAP_SERVICES => p-rabbitmq => mqtt => password"

Open two terminal
    
    #cf logs {application name}
    cf logs python-demo-postgresql

.

    python publisher.py

![https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/publish.PNG](https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/publish.PNG)


Bind PostgreSQL service instance

    #cf stop application_name
    cf stop python-demo-postgresql
    #cf bs {application_name} {service_instance_name} -c '{\"group\":\"group_name\"}' 
    cf bs python-demo-postgresql postgresql -c '{\"group\":\"groupfamily\"}'
    #cf start {application_name}
    cf start python-demo-postgresql




# Step By Step Tutorial

[https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/REAMME.md](https://github.com/WISE-PaaS/example-python-iothub-sso/blob/master/source/REAMME.md)

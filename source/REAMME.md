
# WISE-PaaS DCCS with python

This tutorial we want to tell you how to build a application in WISE-PaaS include the SSO and DCCS to bind a services instance

## STEP 1:Prepare Environment

cf-cli

[https://docs.cloudfoundry.org/cf-cli/install-go-cli.html](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

python3

[https://www.python.org/downloads/](https://www.python.org/downloads/)

![](https://cdn-images-1.medium.com/max/2000/1*iJwh3dROjmveF8x1rC6zag.png)

## STEP 2:Build application

Open your editor，(I use [visual studio code](https://code.visualstudio.com/) so i use code .)

    mkdir dccs_with_python
    cd dccs_with_python 
    code .

Add new file

    ~/dccs_with_python>touch index.py
    ~/dccs_with_python>touch requirements.txt
    ~/dccs_with_python>touch manifest.yml

we can add three file index.py requirements.txt manifest.yml three file。

index.py

<iframe src="https://medium.com/media/e9fc5d1e6f579b40b1fe381cba58445a" frameborder=0></iframe>

requirements.txt

<iframe src="https://medium.com/media/bb7bf69b1062ec7722dabd37dfa4a7d7" frameborder=0></iframe>

We can check out the back-end application does it work。

    
    ~/dccs_with_python>python index.py

and it run on [http://0.0.0.0:3000/](http://0.0.0.0:3000/)。

![](https://cdn-images-1.medium.com/max/2000/1*9aQ4O2yCzYOPb0XiT_jg5w.png)

when we finish this we want push it to the WISE-PaaS，so we need to have a manifest.ymlThis file can save the config when we push to the WISE-PaaS。

mainfest.yml

<iframe src="https://medium.com/media/865ad4fa3204d9fe3a4da0bc069777bf" frameborder=0></iframe>

* name: application name

* memory: how much memory you want to give to application

* disk_quoat : how much disk_quoat you want to give to application

* buildpack: when we push application to WISE-PaaS help us compile

* command: start the application(like above we start in our terminal)

You need to change {your name} to yourself to avoid the same application name。

Now，we need to login to the WISE-PaaS，so we type those in terminal

    cf login –skip-ssl-validation -a api.wise-paas.io -u xxx[@advantech.com.tw](mailto:Jimmy.Xu@advantech.com.tw) -p password

![](https://cdn-images-1.medium.com/max/2000/1*Ob3UnoOr8q_aYQ1Xbjx70w.png)

* “-a” is your domain name，because i use the WISE-PaaS，my domain name is **“wise-paas.io”**，and you need to add **“api”** in front of it ，

* “-u” and “-p” is your account and password。

![](https://cdn-images-1.medium.com/max/2000/1*t49ZEQ5thu8evJAkEMfCMw.png)

and you can chek it state use

    cf target

cf push python-demo-jimmy (cf push {application name})

![](https://cdn-images-1.medium.com/max/2550/1*CNV8vUcW2j6uqI2JeoeE0Q.png)

![](https://cdn-images-1.medium.com/max/2000/1*5ju0EFf9oGRhT4jPxoKdXw.png)

![](https://cdn-images-1.medium.com/max/2000/1*qgpI_VsxBqTbcMit9TeqqQ.png)

## Step 3:Single Sign On(SSO)

The Single Sign On is the API to help us only depend on WISE-PaaS /EnSaaS account control who can login to WISE-Paas。

So,we need to add a new html file and use jQuery to fetch the login API。

    ~/dccs_with_python>mkdir templates
    ~/dccs_with_python>cd templates
    ~/dccs_with_python/templates>touch index.html

<iframe src="https://medium.com/media/dcc370a97553673c42a8ebcdad976407" frameborder=0></iframe>
> You need to change your ssoUrl to your application name in code

and we need to change our index.py code to send this html file

<iframe src="https://medium.com/media/4b86e56cb0507cc52c44c160188a6b30" frameborder=0></iframe>

and we push it again

    ~/dccs_with_python>cf push python-demo-jimmy

![](https://cdn-images-1.medium.com/max/2000/1*r0gw31PLrvfrUISLGixpow.png)
> You can click the sign in to check does it work，if it doesn’t maybe the ssoUrl doesn’t replace to the **https://portal-sso.wise-paas.io**。

## STEP 4:Bind a Service Instance

Now we want to bind the services instance to the application，it mean we want to let our application owner a rabbit-mq services，so we can use it credential

we can see our Service Instance List have some service，we want to bind the rabbitmq，so we need to edit our manifest.yml。

![](https://cdn-images-1.medium.com/max/2048/1*9WVqSKOSgHVlr1zObWDzyQ.png)

add the service: -rabbitmq

<iframe src="https://medium.com/media/83cff66220321a3836d7d944f85db6b8" frameborder=0></iframe>

### **You can create a service instance by yourself**

Create a p-rabbitmq-innoworks Instances and we name rabbitmq-demo

(add button => save=>name it ”rabbitmq-demo”)

![](https://cdn-images-1.medium.com/max/2046/1*91CMsf-qIXjk4I3jKDTy4g.png)
> You can bind service instance above in mainfest.yml，but you need to create service instance first。

**We create a Credential， choose your service instance **(Credentials=>Create=>save，Application Bind => choose your app)

![](https://cdn-images-1.medium.com/max/2042/1*9JYiHi649rC1xJuHq0Lblg.png)

Now back to our Application List and we can see the service we bind to the application。

![](https://cdn-images-1.medium.com/max/2054/1*cB0vKobiiDUiHLhk-jGgtw.png)

Now we can use our MQTT service to send data or reveice data，we already bind the p-rabbitmq service to our application，and we need to use it environment，so we need to add some code to the index.js。

<iframe src="https://medium.com/media/658ada3813694450e214bcc10347e509" frameborder=0></iframe>
> If you create your service instance yourself，you need to change the service_name in code。

If you change finish，you need to push again，and type cf.exe logs python-demo-jimmy --recent。，you can see we have the correct result code and subscribe success。

![](https://cdn-images-1.medium.com/max/2000/1*s96CpC3VDWVxvwWzBDIQpQ.png)

## STEP 5:Publish the Message to the mqtt

Now we want to get application credential in our environment，you can type 
cf env python-demo-jimmy > env.json to your terminal to get the environment config。

we need to get broker(externalHosts) username password port

![](https://cdn-images-1.medium.com/max/2070/1*pXjtTgKxPrmN6YyICcERgA.png)
> we need the externalHosts
> port，username，password(in the prabbitmq=>mqtt)

<iframe src="https://medium.com/media/58c38b976432710f2be8ac01dc0af8cd" frameborder=0></iframe>

![](https://cdn-images-1.medium.com/max/2000/1*TDTZ2ITHQHRIlmhUEx0O_A.png)

Run the python code can send message，and we can listen the message with cf logs python-demo-jimmy (cf log {application-name})

    python publisher.py

![](https://cdn-images-1.medium.com/max/2466/1*WzwjNwVA7QMZRJn7bGH27Q.png)

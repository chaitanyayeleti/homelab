# Traefik- Home Lab setup using Rasberrypi 4/5


## Prerequisites

- DNS- AzureDNS or Cloudflare # or any traefik Supported dns provider
- Virtual Machine - Standard B1s # as a VPN Gateway
- rasberry Pi 4/5

## Sotware Prerequisites
- Ubuntu or RasberryPi OS # Rasberry Pi 4/5 supports multiple OS i prefer RasberryPi OS
- Docker
- WireGuard # VPN 


## Azure Virtual Machine as VPN Gateway.

  - We should avoid relying solely on the ISP-provided public IP due to potential limitations like Double NAT, which can result in a lack of control over network settings. Instead, opting for an Azure VM as a VPN gateway enables us to enforce Network Security Groups (NSG) and set up specific routes, granting us greater control and enhancing the security of our environment

  - Let's create an Azure VM with the Standard B1s size. To minimize latency, select the closest or preferred location. Opt for the latest version of Ubuntu. Initially, enable ports 80 and 443 for web services, and for security purposes, keep port 22 open but later consider changing it to a custom port. Additionally, for WireGuard VPN, ensure that port 51820/UDP is open, as it operates on UDP.

  ### Accessing the VPN Gateway using SSH
  - To acesses newly created server over SSH from windows you can use CMD or terminal 
     ``` bash
    ssh -p 22 username@publicip 
    ```
  - Follow the path for WireGuard Configuration [WireGuard Configuration on my other Repo](https://github.com/chaitanyayeleti/WireGuard) **Make Sure you configured Static ip on rasberry client**
  - Once it is connected now all the traffic will go though the VPN tunnel . But the  main issue is just imagine 2 devices are connected to the VPN and one of the device you are hosting the website now the incoming traffic where it will go ?
  - For Saving us we have **IP Tables ** we use these to route the traffic this is the reason why u must keep Static ip for the Client device.
  - ececute the below mentioned CMD on VPN GateWay machine
    ``` bash
    sudo iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.16.0.12:80 # you can use same port or port fowrding . eth0 and wg0 are the interface you are telling to forward the packet
    sudo iptables -A FORWARD -i **eth0 ** -o wg0 -p tcp -m tcp --dport 80 -j ACCEPT  # understand properly the highleted once for port forwarding happens
    ```
- Execute both the CMD's for all the required ports for you like 80,443,8080,53 depends on your requirement in my view 80 and 443 is fine for us
- After the execution save the IP Tables by below CMD
  ``` bash
  sudo sh -c "iptables-save > /etc/iptables/rules.v4"
  ```
## End of all networking and VPN related config 

# Install Docker & Docker-compose  
 - on the client machine/ rasberry pi 4/5 install both docker and docker-compose by using below CMD
   ``` bash
   sudo apt-get install docker.io docker-compose -y
   ```
 - Execute below cmd for proper file structure
   ``` bash
   mkdir traefik
   cd traefik
   mkdir data
   cd data
   touch acme.json
   chmod 600 acme.json
   touch traefik.yml
   touch config.yml
   ```

   ``` bash 
    ./traefik
    ├── data
    │   ├── acme.json
    │   ├── config.yml
    │   └── traefik.yml
    └── docker-compose.yml
   ```


## Once the file structure is created Copy the code from my repo files or clone it 

### DNS Verification and records pointing
 - Recommended DNS provider **AzureDNS** or **Cloudflare** [Supported Providers](https://go-acme.github.io/lego/dns/azuredns/)
 - if you are using cloudflare or AzureDNS or any dns provider for A record pointed to VPN Gateway
 - CNAME * and value yourdomain so now whatever subdomain we use it will redirect to your domain

 ### DNS Verification for wild card certificats 
 - We are using Let's encrypt api to get the wild card certificates
 - In traefik.yml file we are using cloudflare as dns provider below is the reference code from the treafik.yml file
 ``` yaml
 certificatesResolvers:
  cloudflare: # you can choose any name here but what we gave same need to provide input in docker-ccompose.yaml file
    acme:
      email: #your email for certification renewal notification it is mandatory
      storage: acme.json
      dnsChallenge:
        provider: cloudflare
        #disablePropagationCheck: true # uncomment this if you have issues pulling certificates through cloudflare, By setting this flag to true disables the need to wait for the propagation of the TXT record to all authoritative name servers.
        resolvers:
          - "1.1.1.1:53"
          - "1.0.0.1:53"
  ```
 - here provider as cloudflare if we are using azure mention azure or azuredns
 - In Docker-compose.yml file at environment we need to provide input attributes for to get certification verification

 ``` yaml
  environment:
  - CF_API_EMAIL= email of cloudflare 
  - CF_DNS_API_TOKEN= API token from cloudflare with Zone.DNS permission Makesure no Space after equalto
  # - CF_API_KEY=YOUR_API_KEY
  # be sure to use the correct one depending on if you are using a token or key
 ```
 - on the same file in labels wee need to pass tag of certificate provider in my case im using cloudflare 
 ``` yaml 
  labels:
    - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
 ```
 - Now create docker network 
 ``` bash 
 docker network create proxy
 ```
 - by defualt the is no auth for the dashboard we will create auth for the traefik dashboard
 ``` bash 
 echo $(htpasswd -nb "<USER>" "<PASSWORD>") | sed -e s/\\$/\\$\\$/g
 # provide username and password it will generate username:hash | copy this
 ```
 - Copy the generated output and replace the label from docker-compose.yaml
 ``` yaml
 labels:
  - "traefik.http.middlewares.traefik-auth.basicauth.users=user:hash"
 ```
 - provide yourdomain details on labels ..... we completed all the required configuration 
 - ececute docker compose file to start traefik
 ```
 docker-compose up -d
 ```
### accesing the dasboard of traefik 
 - Once the docker is up we can acesses the dashboard using the subdomain which we provide on labels in my case it is  
 ``` yaml
     - "traefik.http.routers.traefik-secure.rule=Host(`traefik-dashboard.yourdomain.in`)"
    traefik-dashboard.yourdomain.in
 ```
 - auth using given username and password.
 - you updated the code to recreate/ update container use below cmd 

 ``` bash 
 docker-compose up -d --force-recreate
 ```

 - We made it wild card SSL certificate for 90 days auto-renew hosted on home or even in cloud 







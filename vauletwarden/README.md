# VaultWarden deployment using traefik

### In this deployment we are using .env file to pass input vaules 
 - Bitwarden is a OpenSource password Manger 
 - In current config I'm passing 2 vaules subdomain and admin token
 - File Structure 
 ``` bash 
    ./VauletWarden
    ├── data
    │  
    └── docker-compose.yml
 ```


 - To generate admin token follow below CMD's
 ``` bash
 sudo apt install -y argon2
 echo -n "yourpassword" | argon2 mysalt0123456789 -k 65536 -t 4 -p 2 -e
 ```
 - Copy the output and add into .env file 
 - we are using same docker network proxy and using labels to get identified by the traefik 
 - now execute the cmd to start the container 
 ``` bash
 docker-compose up -d 
 ```
 - in docker-compose.yml file environment we make SIGNUPS_ALLOWED as false so no signups as false that admin token we generated will be use full to create new user for you or else you can make it true and sinup yourself 
 ``` bash 
 environment:
  - WEBSOCKET_ENABLED=true
  - WEB_VAULT_ENABLED=true
  - SIGNUPS_ALLOWED=false
  # Comment admin token to disable admin interface
  - ADMIN_TOKEN=${ADMIN_TOKEN}
 ```
 - There are other environmnt varibles like SMPT,2fa,etc you can add based on your requirement 

## access the website 
 - using your subdomain.yourdomain.in/admin for admin access
 - sub-domain.yourdomain.in for normal access

## Features 
 - Orginazation/team levle Vault 
 - Personal Vault
 - TOTP
 - Password Generator 
 - Password History, password breach check
 - Secure Notes 
 - secure file share ...


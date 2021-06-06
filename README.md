# Load Balancer Solution With Nginx and SSL/TLS

This project deploys an Nginx Load Balancer solution for our [tooling website](https://github.com/realayo/Tooling_Website_Solution) and secures it using SSL/TLS

This project consists of two parts:
- Configure Nginx as a Load Balancer
- Register a new domain name and configure secured connection using SSL/TLS certificates

## Part 1 - Configure Nginx As A Load Balance
1. Create an EC2 VM based on Ubuntu Server 20.04
2. Update `/etc/hosts` file for local DNS with Web Servers’ names (e.g. Web1 and Web2) and their local IP addresses.
![](https://user-images.githubusercontent.com/18899718/120925808-efc99f80-c69f-11eb-919d-cd8fc1d5a1e0.png)
3. Install Nginx
```
sudo apt update
sudo apt install nginx
```
4. Configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
```
sudo vi /etc/nginx/nginx.conf

#insert following configuration into http section

 upstream myproject {
    server Web1 weight=5;
    server Web2 weight=5;
  }

server {
    listen 80;
    server_name www.domain.com;
    location / {
      proxy_pass http://myproject;
    }
  }

#comment out this line
#       include /etc/nginx/sites-enabled/*;
```
5. Save and exit
6. Restart Nginx and check the status
```
sudo system status nginx

● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset:>
     Active: active (running) since Sun 2021-06-06 13:25:35 UTC; 8s ago
       Docs: man:nginx(8)
    Process: 13476 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_proc>
    Process: 13487 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (>
   Main PID: 13488 (nginx)
      Tasks: 2 (limit: 1160)
     Memory: 1.9M
     CGroup: /system.slice/nginx.service
             ├─13488 nginx: master process /usr/sbin/nginx -g daemon on; master>
             └─13489 nginx: worker process

Jun 06 13:25:35 ip-172-31-23-93 systemd[1]: Starting A high performance web ser>
Jun 06 13:25:35 ip-172-31-23-93 systemd[1]: Started A high performance web serv>
lines 1-15/15 (END)
```

## Part 2 - Register a new domain name and configure secured connection using SSL/TLS certificates
1. Register a new domain name with any registrar of your choice in any domain zone(I used freenom)
2. Assign an Elastic IP address to your EC2 `nginx-lb` instance.(You'll find this under `Network & Security`)in your EC2 dashboard.
![](https://user-images.githubusercontent.com/18899718/120927770-a715e480-c6a7-11eb-83d1-ee3ddb2f700f.png)
3. Update `A record` in your registrar(`freenom`) to point to Nginx LB instance using Elastic IP address.
![](https://user-images.githubusercontent.com/18899718/120928680-41c3f280-c6ab-11eb-8ffc-3adcbd1dbff8.png)
4. Configure Nginx to recognize your new domain name by updating `nginx.conf` with `server_name www.<your-domain-name.com> `instead of `server_name www.domain.com`
![](https://user-images.githubusercontent.com/18899718/120928706-66b86580-c6ab-11eb-8519-e035817de2fd.png)
5. To install certbot, make sure `snapd` is running
```
sudo systemctl status snapd
```
![](https://user-images.githubusercontent.com/18899718/120927946-50f57100-c6a8-11eb-9e10-17edc1e7337a.png)
6. Install certbot
```
sudo snap install --classic certbot
```
![](https://user-images.githubusercontent.com/18899718/120928064-c6f9d800-c6a8-11eb-91da-4ba61c1bf981.png)
7. For the certificate to be issued, run
```
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot --nginx
```
Follow the screen prompts are choose the approprate options for you.
8. Test secured access to your Web Solution by trying to reach `https://<your-domain-name.com>`
![](https://user-images.githubusercontent.com/18899718/120928198-3f609900-c6a9-11eb-8cc4-450288e1fd8c.png)
![](https://user-images.githubusercontent.com/18899718/120928512-8e5afe00-c6aa-11eb-8544-e79c1eda50c8.png)

9. By default, LetsEncrypt certificate is valid for 90 days, so it is recommended to renew it at least every 60 days or more frequently. This can be tested by running 
```
sudo certbot renew --dry-run
```
![](https://user-images.githubusercontent.com/18899718/120928360-f3622400-c6a9-11eb-8cf6-040b82cd3451.png)
10. We can automate the renwal process by setting up a `cronjob`
```
crontab -e
```
![](https://user-images.githubusercontent.com/18899718/120928439-4340eb00-c6aa-11eb-86f3-cdf4f5c71de8.png)
Select a prefered editor and add the following line:
```
* */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1
```

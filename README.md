# Apache-As-Load-Balancer
CONFIGURE APACHE AS A LOAD BALANCER

Create a new ec2 instance for the load balanacer
Open TCP Port 80 in Inbound rules for the load balancer security group
Install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers

<img width="1179" alt="Screenshot 2022-12-02 at 00 26 02" src="https://user-images.githubusercontent.com/61475969/205187222-5cfbaae6-b15a-4bea-9667-c47c60e99325.png">

<img width="1117" alt="Screenshot 2022-12-02 at 00 27 38" src="https://user-images.githubusercontent.com/61475969/205187367-2a311e8e-2ef7-45cc-87e4-e5c5f29c0dc9.png">

Installing Packages

Install apache2, libxml and then configure apache for loadbalancing via enabling proxy and proxy_balancer

# Installing apache2
```sudo apt update```
```sudo apt install apache2 -y```
```sudo apt-get install libxml2-dev```

#Enable following modules:
```sudo a2enmod rewrite```
```sudo a2enmod proxy```
```sudo a2enmod proxy_balancer```
```sudo a2enmod proxy_http```
```sudo a2enmod headers```
```sudo a2enmod lbmethod_bytraffic```
#Restart apache2 service
```sudo systemctl restart apache2```
```sudo systemctl status apache2```

<img width="564" alt="Screenshot 2022-12-02 at 00 29 05" src="https://user-images.githubusercontent.com/61475969/205187553-c89ca888-31f6-44a1-bce4-73f70b01be55.png">

Configuring Load Balancer

Edit the default.conf file to add the backend web servers into the loadbalancers proxy for routing.

```sudo vi /etc/apache2/sites-available/000-default.conf```

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>
````
<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/
````

Note: Only 2 servers were added to the proxy list and also other ways to route traffic aside bytraffic includes byrequests, bybusyness, heartbeats which can be specified in ProxySet lbmethod=? 

<img width="1401" alt="Screenshot 2022-12-02 at 05 13 20" src="https://user-images.githubusercontent.com/61475969/205219793-7376fa17-2f2b-4116-b0f4-1c0d91c564db.png">


Restart the apache2 server sudo systemctl restart apache2

On the web browser, test the load balancing connection using the public Ip address of our load balancer server.

<img width="1401" alt="Screenshot 2022-12-02 at 05 08 44" src="https://user-images.githubusercontent.com/61475969/205219341-b2329a28-6e1f-44a2-9049-6fdf0030caaf.png">


To confirm that traffic is routed evenly to both web servers as the load balancer server is receiving traffic (which in our case is by refreshing the webpage) we can check the logs both servers receive sudo tail -f /var/log/httpd/access_log

Server 1

<img width="1401" alt="Screenshot 2022-12-02 at 05 11 09" src="https://user-images.githubusercontent.com/61475969/205219596-36f02b0e-4c7e-42e9-8a97-f8cff190b304.png">


Server 2

<img width="848" alt="Screenshot 2022-12-02 at 00 33 30" src="https://user-images.githubusercontent.com/61475969/205187996-5c3ab4b2-473e-47bf-8f7b-acf0ff02b7f3.png">

Configuring DNS Names (Locally)

In order not to always provide webserver private ip address whenever a new web server needs to be added on the list of loadbalancer proxy, we can specify them on the hosts file and provide a domain name for each which suites us

```sudo vi /etc/hosts```

<img width="547" alt="Screenshot 2022-12-02 at 00 35 02" src="https://user-images.githubusercontent.com/61475969/205188144-b719eddb-6960-450a-9d7d-721bc6796220.png">

<img width="659" alt="Screenshot 2022-12-02 at 00 35 41" src="https://user-images.githubusercontent.com/61475969/205188234-52391402-fd75-4dc6-b7de-193e28bead49.png">

To see this is play we can curl our dns name on the loadbalancer server. Since the DNS names are local DNS configuration we can only access them locally hence the loadbalancer uses them locally to target the backend web servers

Server1_Web1

<img width="667" alt="Screenshot 2022-12-02 at 00 36 50" src="https://user-images.githubusercontent.com/61475969/205188344-bc339f03-8535-435e-848e-445570e88956.png">


Server2_Web2

<img width="667" alt="Screenshot 2022-12-02 at 00 37 20" src="https://user-images.githubusercontent.com/61475969/205188412-709e8639-d90b-4d7c-9bd0-64732018350f.png">

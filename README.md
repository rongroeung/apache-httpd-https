# Apache Httpd with Https Port Configuration in Docker

## I. Generate SSL/TLS Certificate

### 1. Install Certbot
#### >>> If you haven't already installed Certbot, you can do so by running the following commands:
```
apt-get update
apt-get install certbot
```

### 2. Obtain SSL/TLS Certificates
#### >>> Run Certbot to obtain SSL/TLS certificates for your domain. Replace `crossroadscambodia.church` with your actual domain name:
```
certbot certonly --standalone -d crossroadscambodia.church
```
#### >>> Certbot will guide you through the process, and if successful, it will store the certificates in /etc/letsencrypt/live/crossroadscambodia.church/

### 3. Verify Certificate Generation
#### >>> You can verify the generated certificates by checking the directory where Certbot stores them:
```
ls /etc/letsencrypt/live/crossroadscambodia.church/
```
#### >>> You should see the following files:
#### - `cert.pem`: Your domain's certificate.
#### - `chain.pem`: The Let's Encrypt chain certificate.
#### - `fullchain.pem`: Concatenation of cert.pem and chain.pem.
#### - `privkey.pem`: The private key for your certificate.

### 4. Renew Certificates
#### >>> Certbot certificates usually last for 90 days. You can set up a cron job to automatically renew them. To renew manually, run:
```
certbot renew
```

## II. Apply SSL Certificates on HTTPS Port of Apache HTTPD in Docker

### 1. Create `Dockerfile`
#### >>> Create a `Dockerfile` on path `/opt/https-httpd` with the following content:
```
FROM httpd:latest

# Enable SSL module
RUN sed -i '/#LoadModule ssl_module modules\/mod_ssl.so/c\LoadModule ssl_module modules/mod_ssl.so' /usr/local/apache2/conf/httpd.conf

# Copy SSL certificate and key
COPY fullchain.pem /usr/local/apache2/conf/fullchain.pem
COPY privkey.pem /usr/local/apache2/conf/privkey.pem

# Copy default-ssl.conf to enable HTTPS
COPY default-ssl.conf /usr/local/apache2/conf/extra/httpd-ssl.conf

# Update Apache configuration to include SSL configuration
RUN echo "Include /usr/local/apache2/conf/extra/httpd-ssl.conf" >> /usr/local/apache2/conf/httpd.conf
```

### 2. Import SSL Certificates Into Working Directory
#### >>> Import `fullchain.pem` and `privkey.pem` that we've generated using certbot
```
cp -R /etc/letsencrypt/archive/crossroadscambodia.church/fullchain1.pem /opt/https-httpd/fullchain.pem
cp -R /etc/letsencrypt/archive/crossroadscambodia.church/privkey1.pem /opt/https-httpd/privkey.pem
```

### 3. Create `default-ssl.conf`
#### >>> Create a `default-ssl.conf` file on path `/opt/https-httpd` with your SSL configuration. Replace `crossroadscambodia.church` with your actual domain name:
```
<IfModule ssl_module>
    Listen 443 https
    ServerName crossroadscambodia.church
    SSLEngine on
    SSLCertificateFile /usr/local/apache2/conf/fullchain.pem
    SSLCertificateKeyFile /usr/local/apache2/conf/privkey.pem
</IfModule>
```

### 4. Build Docker Image
#### >>> Build the Docker image using the Dockerfile:
```
docker build -t https-httpd .
```

### 5. Run Docker Container
#### >>> Run the Docker container and map the HTTPS port (443) to a port on the host machine:
```
cd /opt/https-httpd
docker run -d -p 443:443 -v /opt/cr-website:/usr/local/apache2/htdocs --restart always --name cr-website-httpd https-httpd
```
#### >>> If docker-compose.yml or .env was modified, we need to stop and start container (Restart won't be affect the changes).

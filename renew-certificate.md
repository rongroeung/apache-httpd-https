# Renew Certbot Certificate

## 1. Delete current certificate
```
certbot delete
```

## 2. Generate a new certificate
```
certbot certonly --manual --preferred-challenges dns -d '*.crossroadscambodia.church' -d 'crossroadscambodia.church'
```

## 3. Deploy a DNS TXT record accordingly as certbot guided. Here's sample guideline from certbot:
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please deploy a DNS TXT record under the name:

_acme-challenge.crossroadscambodia.church.

with the following value:

kDeQ5udLojezoMI0SNIyMKY_XcsiuarxWoNVgVhCJ1Q

Before continuing, verify the TXT record has been deployed. Depending on the DNS
provider, this may take some time, from a few seconds to multiple minutes. You can
check if it has finished deploying with aid of online tools, such as the Google
Admin Toolbox: https://toolbox.googleapps.com/apps/dig/#TXT/_acme-challenge.crossroadscambodia.church.
Look for one or more bolded line(s) below the line ';ANSWER'. It should show the
value(s) you've just added.

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Press Enter to Continue
```

#### >>> Certbot will guide you through the process, and if successful, it will store the certificates in /etc/letsencrypt/live/crossroadscambodia.church/

## 4. Verify Certificate Generation
#### >>> You can verify the generated certificates by checking the directory where Certbot stores them:
```
ls /etc/letsencrypt/live/crossroadscambodia.church/
```
#### >>> You should see the following files:
#### - `cert.pem`: Your domain's certificate.
#### - `chain.pem`: The Let's Encrypt chain certificate.
#### - `fullchain.pem`: Concatenation of cert.pem and chain.pem.
#### - `privkey.pem`: The private key for your certificate.

## 5. Delete old certificate files from httpd path
```
cd /opt/https-httpd
rm -rf fullchain.pem privkey.pem
```

## 6. Import new certificate files to httpd path
```
cp -R /etc/letsencrypt/archive/crossroadscambodia.church/fullchain1.pem /opt/https-httpd/fullchain.pem
cp -R /etc/letsencrypt/archive/crossroadscambodia.church/privkey1.pem /opt/https-httpd/privkey.pem
```

## 7. Remove current httpd containers
```
docker rm -f cr-web-frontend-httpd cr-building-report-httpd
```

## 8. Remove current httpd image
```
docker rmi https-httpd:latest
```

## 9. Build new httpd image
```
cd /opt/https-httpd
docker build -t https-httpd .
```

## 10. Run new httpd containers
```
docker run -d -p 443:443 -v /opt/cr-web-frontend:/usr/local/apache2/htdocs --restart always --name cr-web-frontend-httpd https-httpd
docker run -d -p 7005:443 -v /opt/cr-building-report:/usr/local/apache2/htdocs --restart always --name cr-building-report-httpd https-httpd
```

## 11. Restart Nginx for CR Web Backend Service
```
systemctl restart nginx
```

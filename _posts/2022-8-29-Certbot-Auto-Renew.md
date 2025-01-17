---
layout: post
title: Certbot Auto Renew
---

Ok. We need to authenticate Certbot with Cloudflare API credentials. In this context, the authentication mechanism exists on the DNS layer. 

Check certificate
```
echo | openssl s_client -showcerts -servername $SERVER -connect $SERVER:443 2>/dev/null | openssl x509 -inform pem -noout -text
```

Cloudflare needs credentials. 
```
mkdir /root/.secrets && chmod 700 /root/.secrets
touch /root/.secrets/cloudflare.ini && chmod 400 /root/.secrets/cloudflare.ini
echo "dns_cloudflare_email = icarostangent@email.com" >> /root/.secrets/cloudflare.ini
echo "dns_cloudflare_api_key = redacted" >> /root/.secrets/cloudflare.ini
```


Install Certbot and the CloudFlare DNS authenticator plugin
```
apt-get install certbot python3-certbot-dns-cloudflare
```


Update all existing certs 
```
certbot renew --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini --preferred-challenges dns-01
```


Ad new cert or renew single cert
```
certbot certonly --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini -d example.com,*.example.com --preferred-challenges dns-01
```
or
```
certbot certonly --webroot -w /home/web/www/domains/dev-eaglegatecollege/htdocs/eaglegatecollege/ -d dev.eaglegatecollege.edu
```
Cron job
```
sudo crontab -e
0 0 * * * certbot renew --dns-cloudflare --dns-cloudflare-credentials /root/.secrets/cloudflare.ini --preferred-challenges dns-01 && service lsws condrestart
```

# Hosting website on GCS
## Upload to a GCS bucket
1. create bucket
```
gsutil mb gs://www.example.com
```
2. list the files to be uploaded
```
git ls-files > /tmp/example_com_files.txt
```
3. upload files to bucket (__WARNING: exclude files that you don't intend to make public__)
```
# ignore first N files and upload file by file
tail -n +N-1 /tmp/www_example_files.txt| while read line; do gsutil -m cp $line gs://example.com/$line; done
```
## Setup bucket to serve website
1. set bucket to 'website' mode
```
gsutil web set -m index.html -e 404.html gs://www.example.com
```
2. make bucket 'public' 
```
gsutil uniformbucketlevelaccess set on gs://www.example.com

gsutil iam ch allUsers:objectViewer gs://www.example.com
```
3. confirm to Google that you own the domain by following the instructions [here](https://cloud.google.com/storage/docs/domain-name-verification)
4. test that your website is acessible through Google's public storage api
```
curl http://storage.googleapis.com/www.example.com
```

## Setup your custom domain to serve from bucket
### Option 1. Using GCS Storage APIs to serve website over HTTP 
(__WARNING__: visitors to your website are not going to be happy that your site is __NOT secure__)
1. Login to the Domain Registrar of your domain and add a CNAME entry for your domain. For example,
```
www     CNAME       c.storage.googleapis.com
```
2. Wait between 10 mins to 1 hour for your registrar to publish the entries (*Note: Google Domain Registrar will take just 1 minute since the addresses are within the Google family*). 
    * You can verify by lookup your domain [here](https://gf.dev/dns-lookup) 
    * You can also verify from your command-line:
    ```
    nslookup -type=cname www.example.com
    ```
### Option 2. Using GCS Load Balancer to serve website over HTTPS 
(__WARNING__: this is __expen$$$$ive__)

Follow the instructions provided by Google Cloud to setup a load balancer and SSL cert - [link](https://cloud.google.com/storage/docs/hosting-static-website#lb-ssl)

> __Google's Load Balancer for SSL is only available in Premium Network Tier and as a result, you end up paying very high costs for a simple static website__

### Option 3. Using CloudFlare to serve website over HTTPS
1. Setup a free account at cloudflare.com 
    * Login into your Cloudflare account
    * Click Websites on the left menu
    * Click on "Add a site"
    * Provide the domain name
    * Configure DNS entries at your domain registrar
2. Add cloudflare's nameservers to your domain name
    ```
    collins.ns.cloudflare.com
    norm.ns.cloudflare.com
    ```
3. Wait 1-6 hours for your registrar to publish the entries. (*Note: Google Domain Registrar doesn't like other nameservers and tends to take longer to publish DNS entries*)
    * You can verify by lookup your domain [here](https://gf.dev/dns-lookup) (*Reminder: don't forget to refresh the lookup page to avoid getting stale results*) 
    * You can also verify from your command-line:
    ```
    nslookup -type=cname www.example.com
    ```
4. Configure DNS entries in Cloudflare to proxy to GCS
    * Add a CNAME record  
    ```
    CNAME    wwww    c.storage.googleapis.com    
    ```
5. Wait between 10 mins to 6 hours for DNS entries to get published
6. Verify by accessing your website

# Alternate approaches
## Host your site on Digital Ocean for FREE
Here are some reference links I explored:
* [Cloud Website Hosting - Get Started in Minutes | DigitalOcean](https://www.digitalocean.com/solutions/website-hosting/)
* [How To Deploy a Static Website to the Cloud with DigitalOcean App Platform](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-static-website-to-the-cloud-with-digitalocean-app-platform)
* [How to Deploy a Static Site For Free Using DigitalOcean’s App Platform](https://www.cloudsavvyit.com/8944/how-to-deploy-a-static-site-for-free-using-digitaloceans-app-platform/)
## Host your site on Google Cloud Run using Free Tier
* [Deploy a website with Cloud Run | Codelab](https://codelabs.developers.google.com/codelabs/cloud-run-deploy#0)
* [Firebase Hosting] (https://firebase.google.com/docs/hosting/quickstart) and pay attention to the plan you choose [Firebase Pricing](https://firebase.google.com/pricing)
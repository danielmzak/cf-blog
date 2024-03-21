---
author: dmz
pubDatetime: 2024-03-21T03:00:00.737Z
title: DDNS with cloudflare and cloudflare-ddns
slug: ddns-with-cloudflare-and-cloudflare-ddns
featured: false
tags:
  - DDNS
  - cloudflare
  - cloudflare-ddns
  - ddclient
description: DDNS with cloudflare and cloudflare-ddns
---

**1. Installation**

* **Install cloudflare-ddns:** 
  - `cd ~ && git clone git@github.com:timothymiller/cloudflare-ddns.git`
  - see: https://github.com/timothymiller/cloudflare-ddns
  - follow: https://github.com/timothymiller/cloudflare-ddns?tab=readme-ov-file#-deploy-with-linux--cron
  - run `crontab -e' and insert following line:
  ```
  */15 * * * * ~/cloudflare-ddns/start-sync.sh
  ```

**2. Cloudflare API Token**

* **Create a token:**
  * Go to your Cloudflare profile ->  "My Profile" -> "API Tokens".
  * Click  "Create Token" -> Use the "Edit zone DNS" template.
  * Select your zone and grant only the "Zone:DNS:Edit" permission.
  * In Zone Resources select Include -> All zones
  * Check summary:

      ```text
      Edit zone DNS API token summary
      This API token will affect the below accounts and zones, along with their respective permissions

      All zones - DNS:Edit
      ```

  * Create the token and copy the generated token value.
  * Test token

    ```bash
    curl -X GET "https://api.cloudflare.com/client/v4/user/tokens/verify" \
     -H "Authorization: Bearer <your_token>" \
     -H "Content-Type:application/json"
    ```

**3. Cloudflare zone-id**
  - Login to to Cloudflare
  - Choose your domain
  - On overview page find `Zone ID` and copy it

**4. Configuration of cloudflare-ddns**

  ```
  cd ~/cloudflare-ddns
  cp config-example.json config.json
  ```

  - edit `config.json`:

  ```
  {
    "cloudflare": [
      {
        "authentication": {
          "api_token": "api_token_here",
          "api_key": {
            "api_key": "api_key_here",
            "account_email": "your_email_here"
          }
        },
        "zone_id": "your_zone_id_here",
        "subdomains": [
          {
            "name": "",
            "proxied": false
          },
          {
            "name": "remove_or_replace_with_your_subdomain",
            "proxied": false
          }
        ]
      }
    ],
    "a": true,
    "aaaa": true,
    "purgeUnknownRecords": false,
    "ttl": 300
  }
  ```

  my `config.json`:

  ```
  {
    "cloudflare": [
      {
        "authentication": {
          "api_token": "put your token here"
        },
        "zone_id": "put your zone_id here",
        "subdomains": [
          {
            "name": "home",
            "proxied": false
          }
        ]
      }
    ],
    "a": true,
    "aaaa": true,
    "purgeUnknownRecords": false,
    "ttl": 300
  }
  ```

**5. Verification**

* **Check Logs:**  Review logs for errors (`/var/log/syslog` or similar).
* **Cloudflare Dashboard:** Verify that the A record for your subdomain has been created or updated in the Cloudflare DNS settings.


**Note**
  - The current release of **ddclient** (3.9.1) doesn’t support Cloudflare API Tokens so use **cloudflare-ddns** 
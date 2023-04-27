---
layout: article
title: "cloudflare的使用"
data: 2023-04-27
tags:
  - develop

---

## 获取Bearer Token

在<https://dash.cloudflare.com/profile> 创建api token

## 管理DNS

```bash
curl -X GET "https://api.cloudflare.com/client/v4/zones" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H "Content-Type:application/json" | python3 -m json.tool  # 获取该TOKEN可以管理的DNS zone信息

ZONE_ID={Your Zone ID}
curl -X GET "https://api.cloudflare.com/client/v4/zones/${ZONE_ID}/dns_records" \
    -H "Authorization: Bearer ${ACCESS_TOKEN}" \
    -H "Content-Type:application/json" | python3 -m json.tool  # 获取该DNS zone下的records

curl --request PUT \
  --url https://api.cloudflare.com/client/v4/zones/$(ZONE_ID)/dns_records/$(RECORD_ID) \
  --header 'Content-Type: application/json' \
  --header 'Authorization: Bearer ${ACCESS_TOKEN}' \
  --data '{
  "content": "198.51.100.4",
  "name": "domain.com",
  "proxied": false,
  "type": "A",
  "comment": "Domain verification record",
  "tags": [
    "owner:dns-team"
  ],
  "ttl": 3600
}'                # update DNS record
```

可编写python脚本如下

```python
import requests

wan_ip = requests.get('https://api.ipify.org').content.decode('utf8')
print(wan_ip)
update_dns_url = "https://api.cloudflare.com/client/v4/zones/[ZONE_ID]/dns_records/[RECORD_ID]"

payload = {
    "content": wan_ip,
    "name": "comain.com",
    "proxied": False,
    "type": "A",
    "comment": "pool3",
    "tags": [],
    "ttl": 3600
}
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer n_UDIu5eEaC9LSdgCxlMo4pLFO3dlP5efqNXdreO"
}

response = requests.request("PUT", update_dns_url, json=payload, headers=headers)

print(response.text)
```
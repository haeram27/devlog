# json test data 생성

## javascript

json array data 100 개 생성

```js
Array.from({ length: 100 }, (_, i) => ({
    useRule: true,
    ruleName: `hundred_${i}`,
    description: `hundred_${i}`,
    action: 'block',
    autoExpiration: false,
    expirationTime: "2030-12-31 00:00",
    autoRemove: false,
    adminId: "id",
    ruleset: [
        {
            ip: `1.1.1.1`,
            direction: "in"
        }
    ]
}))
```

실행
javascript 프롬프트에서 붙여넣기 실행

* 브라우저(chrome 등) 개발자 도구의 console

## python

### IP json 생성

코드: ipgenjson.py

```python
import json

"""
Generate IPv4 addresses starting from 1.1.1.1

Args:
    count: Number of IP addresses to generate

Returns:
    List of IP addresses (strings)
"""

# number of IP to generate
COUNT = 100

items = []

for i in range(COUNT):
    # Start from 1.1.1.1 (16843009 = 1*256^3 + 1*256^2 + 1*256 + 1)
    ip_num = 16843009 + i

    # extract each octet
    octet1 = (ip_num >> 24) & 0xFF
    octet2 = (ip_num >> 16) & 0xFF
    octet3 = (ip_num >> 8) & 0xFF
    octet4 = ip_num & 0xFF

    ip = f"{octet1}.{octet2}.{octet3}.{octet4}"
    items.append({"ip": ip, "count": i})

# join ip string with comma and save to json file
with open('./ip_array.json', 'w') as f:
    json.dump(items, f, indent=2)

print(f"Generated {len(items)} items")
print(f"First IP: {items[0]['ip']}")
print(f"Last IP: {items[-1]['ip']}")
```

실행

```bash
python3 ./ipgenjson.py
```

result: ip_array.json

```json
[
  {"ip": "1.1.1.1", "count": 0},
  {"ip": "1.1.1.2", "count": 1},
  ...
]
```

# ip 주소 생성

## python - comma separated ip list

코드: ipgenlist.py

```python
"""
Generate IPv4 addresses starting from 1.1.1.1

Args:
    count: Number of IP addresses to generate

Returns:
    List of IP addresses (strings)
"""

# number of IP to generate
COUNT = 100

ips = []

for i in range(COUNT):
    # Start from 1.1.1.1 (16843009 = 1*256^3 + 1*256^2 + 1*256 + 1)
    ip_num = 16843009 + i

    # extract each octet
    octet1 = (ip_num >> 24) & 0xFF
    octet2 = (ip_num >> 16) & 0xFF
    octet3 = (ip_num >> 8) & 0xFF
    octet4 = ip_num & 0xFF

    ips.append(f"{octet1}.{octet2}.{octet3}.{octet4}")

# join ip string with comma and save to txt file
with open('./ip_list.txt', 'w') as f:
    f.write(','.join(ips))

print(f"Generated {len(ips)} IPs")
print(f"First: {ips[0]}, Last: {ips[-1]}")
```

실행

```bash
python3 ./ipgenlist.py
```

result: ip_list.txt

```text
1.1.1.1,1.1.1.2,...
```

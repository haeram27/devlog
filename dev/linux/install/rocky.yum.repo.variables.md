# variables in .repo file for rhel linux family

## 참고

- [Rockylinux-Software Management-DNF](https://docs.rockylinux.org/10/books/admin_guide/13-softwares/#dnf-dandified-yum)
- [Variables in /etc/yum.repos.d/XXX.repo](https://unix.stackexchange.com/questions/19701/yum-how-can-i-view-variables-like-releasever-basearch-yum0)

## RHEL/CentOS 8 and 9

```bash
/usr/libexec/platform-python -c '
import dnf, json
db = dnf.dnf.Base()
db.conf.substitutions.update_from_etc("/")
print(json.dumps(db.conf.substitutions, indent=2))'
```

### Result Example

- rocky9

```bash
{
  "arch": "x86_64",
  "basearch": "x86_64",
  "releasever": "9",
  "contentdir": "pub/rocky",
  "rltype": "",
  "sigcontentdir": "pub/sig",
  "stream": "9-stream"
}
```

## RHEL/CentOS 6 and 7

```bash
python -c 'import yum, json; yb = yum.YumBase(); print json.dumps(yb.conf.yumvar, indent=2)'
```

## RHEL/CentOS 4 and 5

### if you install python-simplejson

```bash
python -c 'import yum, simplejson as json; yb = yum.YumBase(); print json.dumps(yb.conf.yumvar, indent=2)'
```

### otherwise

```bash
python -c 'import yum, pprint; yb = yum.YumBase(); pprint.pprint(yb.conf.yumvar, width=1)'
```

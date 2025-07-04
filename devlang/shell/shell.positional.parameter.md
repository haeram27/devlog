# positional parameter

### echo test

```bash
echo $@
```

|expr|out|
| --- | --- |
| $*   | arg1 arg2 with space arg3|
| $@   | arg1 arg2 with space arg3|
| "$*" | arg1,arg2 with space,arg3|
| "$@" | arg1 arg2 with space arg3|

### for in test

```bash
for p in "$@"; do echo $p done
```
|expr|out|
| --- | --- |
| $*   | arg1 <br>arg2 with space <br>arg3|
| $@   | arg1 <br>arg2 with space <br>arg3|
| "$*" | arg1 arg2 with space arg3|
| "$@" | arg1 <br>arg2 with space <br>arg3|
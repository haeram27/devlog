# read input

## read

```bash
echo "Enter the name of an animal: "
read 
echo $REPLY

echo -n "Enter the name of an animal: "
read 
echo $REPLY

echo -n "Enter the name of an animal: "
read ANIMAL
echo $ANIMAL

# -p option works only in script
read -p "Enter the name of an animal: "
echo $REPLY

read -p "Enter the name of an animal: " ANIMAL
echo $ANIMAL
```

### read values separated using IFS

```bash
# default IFS is SPACE
echo -n $IFS | hexdump -C

read -r var1 var2 var3 <<< "a b c"
echo $var1
echo $var2
echo $var3

IFS=',' read -r var1 var2 var3 <<< "a,b,c"
echo $var1
echo $var2
echo $var3

IFS='|' read -r var1 var2 var3 <<< "a|b|c"
echo $var1
echo $var2
echo $var3
```

### read and case

[case](https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html)

```bash
echo -n "Enter the name of an animal: "
read ANIMAL
echo -n "The $ANIMAL has "
case $ANIMAL in
  horse | dog | cat) echo -n "four";;
  man | kangaroo ) echo -n "two";;
  *) echo -n "an unknown number of";;
esac
echo " legs."
```

## select

```bash
# * = all files in current working directory
# fname = selected item's value string
# $REPLY = selected item's number

select fname in *;
do
        echo you picked $fname \($REPLY\)
        break;
done
```

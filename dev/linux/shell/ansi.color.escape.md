# ansi color escape

## ref

* https://stackoverflow.com/questions/4842424/list-of-ansi-color-escape-sequences
* https://en.wikipedia.org/wiki/ANSI_escape_code#Colors

## ansi escape code : color

ansi escape code란?

ascii esc(\033, \e)키를 포함하는 start sequence(`\033[`)와 end sequence(m) 사이에 정수 코드(SGR-Select Graphic Rendition parameters)를 삽입하여 terminal이 일반 ascii 코드와 구분(escape)하여 인식할 수 있도록 하는 escape 표현식이다.

ansi escape 코드중 color 코드는 terminal과 약속된 문자 fg/bg color를 지정하는 코드 이다. terminal은 escape code를 만나면 terminal의 속성(text fg/bg color, style)값을 지정된 코드의 값으로 변경한다.

terminal로 escape color 코드가 전달 되면 terminal은 전달된 코드에 맞게 속성 값(text fg/bg color, style)을 변경한다. 하지만 출력의 주체는 terminal 이므로 terminal 마다 또 내부의 color 코드가 다를수 있기 때문에 출력에 반영 되는 최종 color는 다를 수 있다.(terminal에서 코드별 컬러 설정이 변경 가능할 수 있음)

## tput / setaf / setab

tput 명령어를 통해서 문자의 fg/bg 컬러를 변경
단, tpub 명령을 사용해서 적용하는 color는 ansi escape를 직접(\003, \e) 사용하는 것 보다 훨씬 동작이 느리다

### sub-command

```bash
setaf `<ansi-color-code>` # 문자의 fg 컬러 변경
setab `<ansi-color-code>` # 문자의 bg 컬러 변경
```

### ex

$ echo $(tput setaf 1)red$(tput sgr0)
$ echo $(tput setab 21)red$(tput sgr0)
$ echo $(tput bold setaf 1 setab 21)red$(tput sgr0)

### 적용가능 명령어

* echo
* print

## ansi escape color code

### format

`\033[`(escape code start sequence)와 `m`(escape code end sequence) 사이에 font effect를 위한 number code(`SGR-Select Graphic Rendition` parameters)를 ; 구분자로 하여 정의한다.

033 은 ascii octet 표현으로 ESC(escape) 문자를 의미

#### `\033[1;4;31m` 의미?

font-effect 1(bold), 4(underline), 31(fg color) 을 적용
fg color가 여러 번 나열될 경우 마지막 color 적용 (bg 컬러 동일)

#### `\033[0m` 의미?

font-effect 0 (reset)을 적용

### 명령어: echo, print (`\033[<code;...>m` or `\e[<code;...>m`)

```bash
echo  '\e[1;4;31m red \e[0m'
print '\e[1;4;31m red \e[0m'

echo '\033[1;4;31m red \033[0m'
print '\033[1;4;31m red \033[0m'
```

### 명령어: sed (`\o033[<code;...>m`)

```bash
echo red | sed -u -e "s/\(red\)/\o033[1;4;31m\1\o033[0m/gi"
```

### color code: 3-bit / 8-bit(256)

#### 3-bit color code

별도의 선행 코드 없이 컬러 적용
개발 환경 구성에 있어서 3-bit 코드면 충분
fg는 bright(90-97) bg는 40-47(normal)로 조합하는 것이 좋다.

| Code | 의미 |
| --- | --- |
| 30 - 37 | normal fg |
| 40 - 47 | normal bg |
| 90 - 97 | bright fg |
| 100 - 107 | bright bg |

#### ex

```bash
echo "\e[1;94;45m#hello#\e[0m"
```

## code effect

| Code |  Effect |
| --- | --- |
| 0   |  Reset/Normal |
| 1   |  Bold, Brighter (increased intensity) |
| 2   |  Faint, Darker (decreased intensity) |
| 3   |  Italics |
| 4   |  Underlined |
| 5   |  Slow Blink |
| 6   |  Rapid Blink |
| 7   |  Inversion (swap fg and bg) |
| 8   |  Invisible/Conceal (set fg same with bg) |
| 9   |  StrikeThrough |

### 3-bit color code

| fg   | bg  | Color |
| ---  | --- | --- |
| 30   | 40  |  BLACK |
| 31   | 41  |  RED |
| 32   | 42  |  GREEN |
| 33   | 43  |  YELLOW |
| 34   | 44  |  BLUE |
| 35   | 45  |  PURPLE |
| 36   | 46  |  CYAN |
| 37   | 47  |  WHITE |

| fg  | bg  | Bright Color |
| --- | --- | --- |
| 90  | 100 | BLACK |
| 91  | 101 | RED |
| 92  | 102 | GREEN |
| 93  | 103 | YELLOW |
| 94  | 104 | BLUE |
| 95  | 105 | PURPLE |
| 96  | 106 | CYAN |
| 97  | 107 | WHITE |

### 8-bit color code

256(0-255)개의 컬러 지정 가능

선행 코드 필요 (fg=38;5; , bg=48;5;)

| expr | meaning |
| --- | --- |
| 0   | reset |
| 38;5;${color} | foreground |
| 48;5;${color} | background |

#### ${color} code

| Code |  Effect |
| --- | --- |
| 0 - 7     | standard colors (as in `ESC [ 30–37 m`) |
| 8 - 15    | high intensity colors (as in `ESC [ 90–97 m`) |
| 16 - 231  | 6 × 6 × 6 cube (216 colors): 16 + 36 × r + 6 × g + b (0 ≤ r, g, b ≤ 5) |
| 232 - 255 | grayscale from dark to light in 24 steps |

ex: fg(0)=black, bg(15)=white

```bash
echo -e "\033[38;5;0;48;5;15m#hello#\033[0m"
```

#### ansi-escape-code(colors) :: semicolon-separated

| Code | Effect | Note |
| --- | --- | --- |
| 0       | Reset / Normal | all attributes off |
| 1       | Bold or increased intensity | |
| 2       | Faint (decreased intensity) | Not widely supported. |
| 3       | Italic | Not widely supported. Sometimes treated as inverse. |
| 4       | Underline | |
| 5       | Slow Blink |less than 150 per minute |
| 6       | Rapid Blink | MS-DOS ANSI.SYS; 150+ per minute; not widely supported |
| 7       | [[reverse video]] |swap foreground and background colors |
| 8       | Conceal | Not widely supported. |
| 9       | Crossed-out | Characters legible, but marked for deletion. Not widely supported.
| 10      | Primary(default) font | |
| 11–19   | Alternate font | Select alternate font n-10 |
| 20      | Fraktur |hardly ever supported |
| 21      | Bold off or Double Underline | Bold off not widely supported; double underline hardly ever supported. |
| 22      | Normal color or intensity | Neither bold nor faint |
| 23      | Not italic, not Fraktur | |
| 24      | Underline off | Not singly or doubly underlined |
| 25      | Blink off | |
| 27      | Inverse off | |
| 28      | Reveal | conceal off |
| 29      | Not crossed out | |
| 30–37   | Set foreground color | See color table below <br> `\033[<30-37>m` |
| 38      | Set foreground color | Next arguments are `5;<n>` or `2;<r>;<g>;<b>`, see below <br> `\033[38;5;<0-255>m` <br> `\033[38;2;<r>;<g>;<b>m` |
| 39      | Default foreground color | implementation defined (according to standard) |
| 40–47   | Set background color | See color table below <br> `\033[<40-47>m` |
| 48      | Set background color | Next arguments are `5;<n>` or `2;<r>;<g>;<b>`, see below <br> `\033[48;5;<0-255>m` <br> `\033[48;2;<r>;<g>;<b>m` |
| 49      | Default background color | implementation defined (according to standard) |
| 51      | Framed | |
| 52      | Encircled | |
| 53      | Overlined | |
| 54      | Not framed or encircled | |
| 55      | Not overlined | |
| 60      | ideogram underline | hardly ever supported |
| 61      | ideogram double underline | hardly ever supported |
| 62      | ideogram overline | hardly ever supported |
| 63      | ideogram double overline | hardly ever supported |
| 64      | ideogram stress marking | hardly ever supported |
| 65      | ideogram attributes off | reset the effects of all of 60-64 |
| 90–97   | Set bright foreground color | aixterm (not in standard) <br> `\033[<90-97>m` |
| 100–107 | Set bright background color | aixterm (not in standard) <br> `\033[<100-107>m` |

## test functions

```bash
#!/bin/zsh

colortest-style() {
  for style in {0..9}; do
    printf "\033[${style};36;47mstyle:${style}\033[0m ";
    echo
  done
}


colortest-3bit() {
  for fg in {30..37}; do
    for bg in {40..47}; do
      printf "\033[${fg};${bg}mf:${fg},b:${bg}\033[0m ";
    done
    echo
  done
  echo

  for fg in {90..97}; do
    for bg in {40..47}; do
      printf "\033[${fg};${bg}mf:${fg},b:${bg}\033[0m ";
    done
    echo
  done
  echo

  for fg in {30..37}; do
    for bg in {100..107}; do
      printf "\033[${fg};${bg}mf:${fg},b:${bg}\033[0m ";
    done
    echo
  done
  echo

  for fg in {90..97}; do
    for bg in {100..107}; do
      printf "\033[${fg};${bg}mf:${fg},b:${bg}\033[0m ";
    done
    echo
  done
  echo
}


colortest-8bit() {
  echo "foreground:"
  for fgcolor in {1..15};do
    printf "\e[38;5;${fgcolor}m%5d\033[0m" ${fgcolor}
  done
  echo

  for fgcolor in {16..231};do
    if [[ (${fgcolor} -ne 16) && ($((${fgcolor}%6)) -eq 4) ]];then echo ""; fi
    printf "\e[38;5;${fgcolor}m%5d\033[0m" ${fgcolor}
  done
  echo

  for fgcolor in {232..255};do
    printf "\e[38;5;${fgcolor}m%5d\033[0m" ${fgcolor}
  done
  echo
  echo

  echo "background:"
  for bgcolor in {1..15};do
    printf "\e[48;5;${bgcolor}m%5d\033[0m" ${bgcolor}
  done
  echo

  for bgcolor in {16..231};do
    if [[ (${bgcolor} -ne 16) && ($((${bgcolor}%6)) -eq 4) ]];then echo ""; fi
    printf "\e[48;5;${bgcolor}m%5d\033[0m" ${bgcolor}
  done
  echo

  for bgcolor in {232..255};do
    printf "\e[48;5;${bgcolor}m%5d\033[0m" ${bgcolor}
  done
  echo
  echo

  echo "combination:"
  print "\e[38;5;196;48;5;21m combination \033[0m"
}


colortest-tput-8bit() {
  echo "foreground:"
  for color in {1..15};do
    printf  "$(tput setaf ${color})%5d$(tput sgr0)" ${color}
  done
  echo

  for color in {16..231};do
    if [[ (${color} -ne 16) && ($((${color}%6)) -eq 4) ]];then echo ""; fi
    printf  "$(tput setaf ${color})%5d$(tput sgr0)" ${color}
  done
  echo

  for color in {232..255};do
    printf  "$(tput setaf ${color})%5d$(tput sgr0)" ${color}
  done
  echo
  echo

  echo "background:"
  for color in {1..15};do
    printf  "$(tput setab ${color})%5d$(tput sgr0)" ${color}
  done
  echo

  for color in {16..231};do
    if [[ (${color} -ne 16) && ($((${color}%6)) -eq 4) ]];then echo ""; fi
    printf  "$(tput setab ${color})%5d$(tput sgr0)" ${color}
  done
  echo

  for color in {232..255};do
    printf  "$(tput setab ${color})%5d$(tput sgr0)" ${color}
  done
  echo
  echo

  echo "combination:"
  print "$(tput setaf 196 setab 21) combination $(tput sgr0)"
}
```

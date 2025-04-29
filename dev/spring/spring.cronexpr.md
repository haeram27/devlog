# Class CronExpression

- [Class CronExpression](#class-cronexpression)
  - [refs](#refs)
  - [Syntax](#syntax)
  - [Allowed Special Characters](#allowed-special-characters)
    - [Asterisk ( \* )](#asterisk---)
    - [Questionmark ( ? ) - dow, dom](#questionmark------dow-dom)
    - [Comma ( , )](#comma---)
    - [Hyphen ( - )](#hyphen----)
    - [Slash ( / )](#slash---)
    - ['L' - `dom, dow`](#l---dom-dow)
    - ['L-nth'  - `dom`](#l-nth----dom)
    - ['DDDL' - `dow`](#dddl---dow)
    - ['nthW' - `dom`](#nthw---dom)
    - ['LW' - `dom`](#lw---dom)
    - [sharp( dow#nth ) - `dom`](#sharp-downth----dom)
  - [Example expressions](#example-expressions)

---

## refs

<https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/scheduling/support/CronExpression.html>
<https://www.manpagez.com/man/5/crontab/>
<http://crontab.org/>
<https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html>
<https://en.wikipedia.org/wiki/Cron#CRON_expression>

## Syntax

```Plain
┌───────────── second (0-59)
│ ┌───────────── minute (0 - 59)
│ │ ┌───────────── hour (0 - 23)
│ │ │ ┌───────────── day of the month (1 - 31)
│ │ │ │ ┌───────────── month (1 - 12) (or JAN-DEC)
│ │ │ │ │ ┌───────────── day of the week (0 - 7)
│ │ │ │ │ │              (0 or 7 is SUN, or MON(1)-SUN(7))
│ │ │ │ │ │
* * * * * *

```

| Field Name | Allowed Values | Allowed Special Characters |
|:---|:---|:---|
| Seconds | 0-59 | , - * / |
| Minutes | 0-59 | , - * / |
| Hours | 0-23 | , - * / |
| Day-of-month | 1-31 | , - * `?` / `L` `W` |
| Month | 1-12 or JAN-DEC | , - * / |
| Day-of-Week | 1-7 or MON(1)-SUN(0 or 7) | , - * `?` / `L` `#` |
| Year (Optional) | empty, 1970-2199 | , - * / |

## Allowed Special Characters

dom = day of month only
dow = day of week only

### Asterisk ( * )

"every" vaild value in the Timeunit

### Questionmark ( ? ) - dow, dom

"not specified value"

'?' can be used instead of '*' in day-of-week and day-of-month
Some cron-expression implementations use '?' as other purpose (e.g. cron daemon start-up time) so that if both '*' and '?' are available in program then it is recommended to use not '?' but '*'

### Comma (\<unit1\> ,\<unit2\>,... )

separator for non-conseutive value list in a timeunit

- "MON,WED,FRI" (day of week)
- "1,2,3,10-20" (day of month)

### Hyphen ( - )

"ranges"

- "2000-2010" (year) = every year between 2000 and 2010, inclusive.

### Slash ( \<start\>/\<interval\> )

"interval or increments or step"

- "0/15" in the seconds field = "the seconds 0, 15, 30, and 45"
- "5/15" in the seconds field = "the seconds 5, 20, 35, and 50"

### 'L' - `dom, dow`

short-hand for "last"

- 'L' in day-of-month = the last day of the month
- 'L' in day-of-week = simply "7" or "SAT".

### 'L-nth'  - `dom`

"nth(day)-to-last day of the month"

- L-3 = from 3rd to the last day in the month

### '\<DDD\>L' - `dow`

"the last day of week d (or DDD) in the month"

- THUL = last Thursday of the month

### '\<nth\>W' - `dom`

"the nearest weekday(Mon-Fri, no SAT/SUN) from nth day"

- "15W" = "the nearest weekday to the 15th of the month"
  - If the 15th is a Saturday, the trigger will fire on Friday the 14th.
  - If the 15th is a Sunday, the trigger will fire on Monday the 16th.
  - If the 15th is a Tuesday, then it will fire on Tuesday the 15th.

- "1W" = "the 1st weekday of the month"
  - If the 1st is a Saturday, the trigger will fire on Monday the 3rd.

### 'LW' - `dom`

"last weekday of the month".

### sharp( \<dow\>#\<nth\> ) - `dom`

"nth day-of-week"

- "5#2" = the 2nd FRI(5) in the month
- "6#3" = the 3rd SAT(6) in the month

## Example expressions

    "0 0 * * * *" = the top of every hour of every day
    "*/10 * * * * *" = every ten seconds
    "0 0 8-10 * * *" = 8, 9 and 10 o'clock of every day
    "0 0 6,19 * * *" = 6:00 AM and 7:00 PM every day
    "0 0/30 8-10 * * *" = 8:00, 8:30, 9:00, 9:30, 10:00 and 10:30 every day
    "0 0 9-17 * * MON-FRI" = on the hour nine-to-five weekdays
    "0 0 0 25 12 ?" = every Christmas Day at midnight
    "0 0 0 L * *" = last day of the month at midnight
    "0 0 0 L-3 * *" = third-to-last day of the month at midnight
    "0 0 0 1W * *" = 1st weekday of the month at midnight
    "0 0 0 LW * *" = last weekday of the month at midnight
    "0 0 0 * * 5L" = last Friday(5) of the month at midnight
    "0 0 0 * * THUL" = last Thursday(4) of the month at midnight
    "0 0 0 ? * 5#2" = the 2nd Friday(5) in the month at midnight
    "0 0 0 ? * MON#1" = the 1st Monday in the month at midnight

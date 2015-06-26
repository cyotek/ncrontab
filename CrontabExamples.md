# Examples of crontab expressions #

Following are examples of crontab expressions and how they would interpreted as a recurring schedule.

```
* * * * *
```

This pattern causes a task to be launched every minute.

```
5 * * * *
```

This pattern causes a task to be launched once every hour and at the fifth minute of the hour (00:05, 01:05, 02:05 etc.).

```
* 12 * * Mon
```

This pattern causes a task to be launched every minute during the 12th hour of Monday.

```
* 12 16 * Mon
```

This pattern causes a task to be launched every minute during the 12th hour of Monday, 16th, but only if the day is the 16th of the month.

```
59 11 * * 1,2,3,4,5
```

This pattern causes a task to be launched at 11:59AM on Monday, Tuesday, Wednesday, Thursday and Friday. Every sub-pattern can contain two or more comma separated values.

```
59 11 * * 1-5
```

This pattern is equivalent to the previous one. Value ranges are admitted and defined using the minus character.

```
*/15 9-17 * * *
```

This pattern causes a task to be launched every 15 minutes between the 9th and 17th hour of the day (9:00, 9:15, 9:30, 9:45 and so on... note that the last execution will be at 17:45). The slash character can be used to identify periodic values, in the form of a/b. A sub-pattern with the slash character is satisfied when the value on the left divided by the one on the right gives an integer result (a % b == 0).

```
* 12 10-16/2 * *
```

This pattern causes a task to be launched every minute during the 12th hour of the day, but only if the day is the 10th, the 12th, the 14th or the16th of the month.

```
* 12 1-15,17,20-25 * *
```

This pattern causes a task to be launched every minute during the 12th hour of the day, but the day of the month must be between the 1st and the 15th, the 20th and the 25, or at least it must be the 17th.


---


The above examples were slightly adapted from [cron4j Quickstart](http://www.sauronsoftware.it/projects/cron4j/quickstart.php)
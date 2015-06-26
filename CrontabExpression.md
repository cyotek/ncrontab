A crontab expression are a very compact way to express a recurring schedule. A single expression is composed of 5 space-delimited fields:

```
MINUTES HOURS DAYS MONTHS DAYS-OF-WEEK
```

Each field is expressed as follows:

  * A single wildcard (`*`), which covers all values for the field. So a `*` in days means all days of a month (which varies with month and year).
  * A single value, e.g. `5`. Naturally, the set of values that are valid for each field varies.
  * A comma-delimited list of values, e.g. `1,2,3,4`. The list can be unordered as in `3,4,2,6,1`.
  * A range where the minimum and maximum are separated by a dash, e.g. `1-10`. You can also specify these in the wrong order and they will be fixed. So `10-5` will be treated as `5-10`.
  * An interval specification using a slash, e.g. `*/4`. This means every 4th value of the field. You can also use it in a range, as in `1-6/2`.
  * You can also mix all of the above, as in: `1-5,10,12,20-30/5`

The table below lists the valid values for each field:

| **Field**      |  **Range** | **Comment**                                                 |
|:---------------|:-----------|:------------------------------------------------------------|
| MINUTES        |    0-59    | -                                                           |
| HOURS          |    0-23    | -                                                           |
| DAYS           |    0-31    | -                                                           |
| MONTHS         |    1-12    | Zero (0) is not valid. Month names also accepted.           |
| DAYS-OF-WEEK   |    0-6     | Where zero (0) means Sunday. Names of days also accepted.   |

Two fields also accept named values in English: MONTHS and DAYS-OF-WEEKS. So you can use names like `January`, `February`, `March `and so on for MONTHS and `Monday`, `Tuesday`, `Wednesday` and so on for DAYS-OF-WEEK. The names are not case-sensitive and you can even use short forms like `Jan`, `Feb`, `Mar` or `Mon`, `Tue`, `Wed`. In fact, at the moment, the parser in NCrontab will use the first match that it finds. Consequently, if you specify just the letter `J` for the month, then it will be interpreted as January since it occurs before the months June and July. If you specify `Ju`, then June will be assumed for the same reason. However, you should stick to either the 3 letter abbreviations or the full name since that is the norm among cron implementations. Finally, you can also mix numerical and named values, as in `Jan,Feb,3,4,May,Jun,6`.
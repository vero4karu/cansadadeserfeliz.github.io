---
layout: post
title: "Mierdero de los días de la semana"
date: 2016-10-07 11:53:05 -0500
excerpt_separator: <!-- more -->
published: true
comments: true
categories: 
- Sin tema
---

Bienvenidos al mierdero de los días de la semana:

<!-- more -->

Lenguaje                       |Sun  | Mon | Tue | Wed | Thu | Fri | Sat
-------------------------------|-----|-----|-----|-----|-----|-----|-----
postgresql: `EXTRACT(DOW...)`  |  0  |  1  |  2  |  3  |  4  |  5  |  6
ruby: `time.wday`              |  0  |  1  |  2  |  3  |  4  |  5  |  6
Python: `isoweekday()`         |  7  |  1  |  2  |  3  |  4  |  5  |  6
Python: `weekday()`            |  6  |  0  |  1  |  2  |  3  |  4  |  5
mysql: `DAYOFWEEK(...)`        |  1  |  2  |  3  |  4  |  5  |  6  |  7
Java: `DayOfWeek(...)`         |  1  |  2  |  3  |  4  |  5  |  6  |  7
Erlang: `day_of_the_week(...)` |  1  |  2  |  3  |  4  |  5  |  6  |  7

Enlaces:

* [mysql: function dayofweek](https://dev.mysql.com/doc/refman/5.5/en/date-and-time-functions.html#function_dayofweek)
* [postgresql: function datetime](https://www.postgresql.org/docs/current/static/functions-datetime.html)
* [java: function DayOfWeek](https://docs.oracle.com/javase/8/docs/api/java/time/DayOfWeek.html)
* [erlang calendar](http://erlang.org/doc/man/calendar.html)
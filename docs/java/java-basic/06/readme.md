# Java8日期时间

旧得日期类java.util.Date新增了很多过期方法，新的日期类java.time.LocalDate/LocalTime/LocalDateTime，新的日期类提供了静态初始化方法、日期时间加减方法、年月日周星期时分秒的操作以及覆盖、格式化、比较、相差等方法。



## Date

Java8中的 java.util.Date类除去过期的方法，有如下可用的API：

```java
Date();				  //构造器，获取当前时间
Date(long date);	  //通过时间戳初始化日期
getTime();			  //获取时间戳
before(Date date);	  //日期比较
after(Date date);	  //日期比较
compareTo(Date date); //日期比较
from(Instant instant);//新的日期类转为旧的日期类
toInstant();		  //旧的日期类转为新的日期类
```

[测试Demo项目地址](https://github.com/solverpeng/java-code/blob/master/java-basic/java8-date-time/src/main/java/com/solverpeng/java8/DateAPI.java)

## 日期

Java8中 java.time包中提供了新的日期类，LocalDate。表示一个具体的日期，有如下API：

```java
// 实例化日期-静态方法
LocalDate LocalDate#now(); 						//获取当前日期
LocalDate LocalDate#of(2019, 12, 30); 			//初始化一个日期
LocalDate LocalDate#parse("2019-10-23");		//通过字符串初始化日期
LocalDate LocalDate#ofYearDay(2019, 222);		//通过指定年和第n天来初始化日期
LocalDate ofEpochDay(long epochDay);			//距离1970年多少天来初始化日期
```

```java
// 日期加减
LocalDate LocalDate#plusDays(long daysToAdd);		//加多少天，可以为负数
LocalDate plusWeeks(long weeksToAdd);				//加多少周，同上
LocalDate plusMonths(long monthsToAdd);				//加多少月，同上
LocalDate plusYears(long yearsToAdd);				//加多少年，同上
LocalDate plus(long amountToAdd, TemporalUnit unit);//加多少指定单位（年/月/日/周）

LocalDate minusDays(long daysToSubtract);					//同上
LocalDate minusWeeks(long weeksToSubtract);					//同上
LocalDate minusMonths(long monthsToSubtract);				//同上
LocalDate minusYears(long yearsToSubtract);					//同上
LocalDate minus(long amountToSubtract, TemporalUnit unit);	//同上
```

```java
// 年月日周星期
int getYear();				//获取年份
int getMonthValue();		//获取月份，1-12
Month getMonth();			//获取月份，返回Month枚举
int getDayOfMonth();		//获取是当月第多少天
int getDayOfYear();			//获取是当年第多少天
DayOfWeek getDayOfWeek();	//获取是一周中的星期几
boolean isLeapYear();		//是否是闰年
int lengthOfMonth();		//这个月有多少天
int lengthOfYear();			//这个年有多少天
```

```java
// 年月日覆盖
LocalDate withYear(int year);					//返回LocalDate年更改为新值后的拷贝
LocalDate withMonth(int month);					//返回LocalDate月更改为新值后的拷贝
LocalDate withDayOfMonth(int dayOfMonth);		//返回LocalDate日更改为新值后的拷贝
LocalDate withDayOfYear(int dayOfYear);			//返回LocalDate月日更改为新值后的拷贝

LocalDate with(TemporalAdjuster var1);			//返回LocalDate指定字段更改为新值后的拷贝
TemporalAdjusters：
//静态方法
TemporalAdjuster firstDayOfMonth();						//获取当月第一天
TemporalAdjuster lastDayOfMonth();						//获取当月最后一天
TemporalAdjuster firstDayOfNextMonth();					//获取下个月第一天
TemporalAdjuster firstDayOfYear();						//获取当年第一天
TemporalAdjuster lastDayOfYear();						//获取当年最后一天
TemporalAdjuster firstDayOfNextYear();					//获取明年第一天
TemporalAdjuster firstInMonth(DayOfWeek dayOfWeek);		//获取当月第一个星期几
TemporalAdjuster lastInMonth(DayOfWeek dayOfWeek);		//获取当月最后一个星期几
TemporalAdjuster dayOfWeekInMonth(int ordinal, DayOfWeek dayOfWeek);//获取当月第几个星期几
TemporalAdjuster next(DayOfWeek dayOfWeek);				//获取下一个星期几
TemporalAdjuster nextOrSame(DayOfWeek dayOfWeek);		//获取下一个或同一个星期几
TemporalAdjuster previous(DayOfWeek dayOfWeek);			//获取上一个星期几
TemporalAdjuster previousOrSame(DayOfWeek dayOfWeek);	//获取前一个或同一个星期几
	
LocalDate with(TemporalField var1, long var2);	//返回LocalDate指定字段更改为新值后的拷贝
```

```java
//格式化
String format(DateTimeFormatter formatter);				//日期格式化
```

```java
//比较
int compareTo(ChronoLocalDate other);					//日期比较
```

```java
//相差
long until(Temporal endExclusive, TemporalUnit unit); 	//和指定日期相差多少个单位（年/月/日）
Period until(ChronoLocalDate endDateExclusive);
```

```java
//1970-01-01
long toEpochDay();										//距离1970-01-01多少天
```

[测试Demo项目地址](https://github.com/solverpeng/java-code/blob/cf89bbfddee35954c55d50c7c9f6f3407d9a2899/java-basic/java8-date-time/src/main/java/com/solverpeng/java8/LocalDateAPI.java)



## 日期时间

Java8中 java.time包中提供了新的日期类，LocalDateTime。表示一个具体的日期，有如下API：

```java
//实例化日期时间
LocalDateTime now();
LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute);
LocalDateTime of(int year, Month month, int dayOfMonth, int hour, int minute, int second);
LocalDateTime parse(CharSequence text);
LocalDateTime parse(CharSequence text, DateTimeFormatter formatter);
```

```java
//日期时间转换，获取年月日时分秒
LocalDate toLocalDate();
LocalTime toLocalTime();

int getYear();
int getMonthValue();
Month getMonth();
int getDayOfMonth();
int getDayOfYear();
DayOfWeek getDayOfWeek();
int getHour();
int getMinute();
int getSecond();
int getNano();
```

```java
//日期时间加减
LocalDateTime plusYears(long years);
LocalDateTime plusMonths(long months);
LocalDateTime minusYears(long years);
LocalDateTime minusMonths(long months);
```

```java
//拷贝覆盖
LocalDateTime withYear(int year);
LocalDateTime withMonth(int month);
```

```java
//比较
int compareTo(ChronoLocalDateTime<?> other);
boolean isAfter(ChronoLocalDateTime<?> other);
boolean isBefore(ChronoLocalDateTime<?> other);
```

```java
//格式化
String format(DateTimeFormatter formatter);
```

```java
//相差
long until(Temporal endExclusive, TemporalUnit unit);
```

## 时间

Java8中 java.time包中提供了新的日期类，LocalTime。表示一个具体的时间，有如下API：

```java
//实例化时间
LocalTime now();												//当前时间
LocalTime of(int hour, int minute);								//指定时分时间
LocalTime of(int hour, int minute, int second);					//指定时分秒时间
LocalTime parse(CharSequence text);								//通过字符串初始化
LocalTime parse(CharSequence text, DateTimeFormatter formatter);//通过字符串和指定格式初始化
```

```java
//时间加减
LocalTime plus(TemporalAmount amountToAdd);
LocalTime plus(long amountToAdd, TemporalUnit unit);
LocalTime plusHours(long hoursToAdd);
LocalTime plusMinutes(long minutesToAdd);
LocalTime plusSeconds(long secondstoAdd);
LocalTime minus(long amountToSubtract, TemporalUnit unit);
LocalTime minusHours(long hoursToSubtract);
LocalTime minusMinutes(long minutesToSubtract);
```

```java
//时间覆盖
LocalTime with(TemporalAdjuster var1);
LocalTime with(TemporalField var1, long var2);
LocalTime withHour(int var1);						//覆盖小时
LocalTime withMinute(int var1);						//覆盖分
LocalTime withSecond(int var1);						//覆盖秒
```

```java
//格式化
String format(DateTimeFormatter formatter);
```

```java
//时间比较
int compareTo(LocalTime other);
```

[测试Demo项目地址](https://github.com/solverpeng/java-code/blob/master/java-basic/java8-date-time/src/main/java/com/solverpeng/java8/LocalTimeAPI.java)



## Instant



## Date与LocalDateTime\LocalDate\LocalTime互转

Java 8中 java.util.Date类新增了两个方法，分别是`from(Instant instant)`和`toInstant()`方法

```java
public static Date from(Instant var0) {
    try {
        return new Date(var0.toEpochMilli());
    } catch (ArithmeticException var2) {
        throw new IllegalArgumentException(var2);
    }
}

public Instant toInstant() {
    return Instant.ofEpochMilli(this.getTime());
}
```

`from(Instant instant)`将新的日期类转换为旧的日期类，`toInstant()`将旧的日期类转换为新的日期类。都是通过`Instant`作为中介进行转换的。

```java
// 01. java.util.Date --> java.time.LocalDateTime
public void DateToLocalDateTime() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
}
	
// 02. java.util.Date --> java.time.LocalDate
public void DateToLocalDate() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
    LocalDate localDate = localDateTime.toLocalDate();
}
	
// 03. java.util.Date --> java.time.LocalTime
public void DateToLocalTime() {
    java.util.Date date = new java.util.Date();
    Instant instant = date.toInstant();
    ZoneId zone = ZoneId.systemDefault();
    LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
    LocalTime localTime = localDateTime.toLocalTime();
}
	
	
// 04. java.time.LocalDateTime --> java.util.Date
public void LocalDateTimeTodate() {
    LocalDateTime localDateTime = LocalDateTime.now();
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDateTime.atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}
	
	
// 05. java.time.LocalDate --> java.util.Date
public void LocalDateToUdate() {
    LocalDate localDate = LocalDate.now();
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDate.atStartOfDay().atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}
	
// 06. java.time.LocalTime --> java.util.Date
public void LocalTimeTodate() {
    LocalTime localTime = LocalTime.now();
    LocalDate localDate = LocalDate.now();
    LocalDateTime localDateTime = LocalDateTime.of(localDate, localTime);
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDateTime.atZone(zone).toInstant();
    java.util.Date date = Date.from(instant);
}
```


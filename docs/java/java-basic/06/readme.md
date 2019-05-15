# Java8日期时间



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

## 时间



## 时间日期





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


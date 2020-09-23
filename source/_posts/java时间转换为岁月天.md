---
title: java时间转换为岁月天
date: 2020-09-23 19:33:06
tags: [java]
---

```java
	/**
	 * 根据时间字符串转换为 岁月天
	 */
	public String getDateToYearAndMonthAndDays(String date) throws Exception {

		LocalDate today = LocalDate.now();
//		LocalDate playerDate = LocalDate.from(DateTimeFormatter.ofPattern("yyyy-MM-dd").parse("2019-09-22"));
		LocalDate playerDate = LocalDate.from(DateTimeFormatter.ofPattern("yyyy-MM-dd").parse(date));
		//是否当前日期在当日之后
		if (playerDate.isAfter(today)) {
			throw new Exception("传入的日期在当前日期之后"+today.toString());
		}
		//获取出生月份的天数
		int playerDatetoDay = playerDate.getDayOfMonth();
		//获取当前月份的天数
		int nowDayBetween = today.getDayOfMonth();
		long days;
		if (playerDatetoDay > nowDayBetween) {
			//获取出生的月份的最后一天
			LocalDate lastDay = playerDate.with(TemporalAdjusters.lastDayOfMonth());
			days = lastDay.getDayOfMonth() - playerDatetoDay + nowDayBetween;
		} else {
			days = nowDayBetween - playerDatetoDay;
		}
		long years = ChronoUnit.YEARS.between(playerDate, today); //准确年份
		long months = ChronoUnit.MONTHS.between(playerDate, today) % 12; //精确月份
		//获取复位的时间
//		System.out.println(playerDate.plusDays(days).plusMonths(months).plusYears(years));
		//输出拆分的时间
		return years + "岁" + months + "月" + days + "天";
	}
```


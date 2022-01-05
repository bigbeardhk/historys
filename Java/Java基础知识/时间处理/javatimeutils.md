
```java
 /**
     * 传入一个LocalTime,判断当前时间窗
     *
     * @param timeNow LocalTime
     * @return -1非面试时间, 0上午, 1下午, 2晚上
     */
    public static byte getTimeWindow(LocalTime timeNow) {
        Byte timeWindowNow = -2; // 时间窗  -1非面试时间 0上午,1下午,2晚上
        LocalTime timeNowStar = LocalTime.of(0, 0, 0);
        LocalTime timeNowEnd = LocalTime.of(23, 59, 59);
        LocalTime timeNowMorning = LocalTime.of(9, 0, 0);
        LocalTime timeNowAfternoons = LocalTime.of(12, 0, 0);
        LocalTime timeNowNight = LocalTime.of(18, 0, 0);
        if (timeNowStar.isBefore(timeNow) && timeNowMorning.isAfter(timeNow) || timeNow.compareTo(timeNowMorning) == 0) {
            timeWindowNow = -1; // 0-9 左开右闭
        } else if (timeNowMorning.isBefore(timeNow) && timeNowAfternoons.isAfter(timeNow) || timeNow.compareTo(timeNowAfternoons) == 0) {
            timeWindowNow = 0; // 9-12
        } else if (timeNowAfternoons.isBefore(timeNow) && timeNowNight.isAfter(timeNow) || timeNow.compareTo(timeNowNight) == 0) {
            timeWindowNow = 1; // 12-18
        } else if (timeNowNight.isBefore(timeNow) && timeNowEnd.isAfter(timeNow)
                || timeNow.compareTo(timeNowEnd) == 0 || timeNow.compareTo(timeNowStar) == 0) {
            timeWindowNow = 2; // 18-24
        } else {
            timeWindowNow = 4; // 不可能存在
        }
        return timeWindowNow;
    }


```

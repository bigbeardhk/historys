/**
 * Copyright (c) bigbeardhk@163.com Corporation 2022. All Rights Reserved.
 */

package com.hk.study.util;

import java.time.LocalTime;

/**
 * @Author : bigbeardhk
 * @Date : 2022/01/05 21:16
 **/
public class DateUtils {


    /**
     * 传入一个时间点,返回当前时间点所处的时间段(时间窗)
     *
     * @param timeNow 时间点
     * @return 0(凌晨或早上) 1(上午) 2(下午) 3(晚上)
     */
    public static byte getTimeWindow(LocalTime timeNow) {
        Byte timeWindowNow = -1;
        LocalTime timeNowStar = LocalTime.of(0, 0, 0);
        LocalTime timeNowEnd = LocalTime.of(23, 59, 59);
        LocalTime timeNowMorning = LocalTime.of(9, 0, 0);
        LocalTime timeNowAfternoons = LocalTime.of(12, 0, 0);
        LocalTime timeNowNight = LocalTime.of(18, 0, 0);
        if (timeNowStar.isBefore(timeNow) && timeNowMorning.isAfter(timeNow) || timeNow.compareTo(timeNowStar) == 0) {
            timeWindowNow = 0; // [0-9) 左闭右开
        } else if (timeNowMorning.isBefore(timeNow) && timeNowAfternoons.isAfter(timeNow) || timeNow.compareTo(timeNowMorning) == 0) {
            timeWindowNow = 1; // [9-12)
        } else if (timeNowAfternoons.isBefore(timeNow) && timeNowNight.isAfter(timeNow) || timeNow.compareTo(timeNowAfternoons) == 0) {
            timeWindowNow = 2; // [12-18)
        } else if (timeNowNight.isBefore(timeNow) && timeNowEnd.isAfter(timeNow)
                || timeNow.compareTo(timeNowNight) == 0 || timeNow.compareTo(timeNowEnd) == 0) {
            timeWindowNow = 3; // [18-24)
        } else {
            timeWindowNow = -1; // 不可能存在
        }
        return timeWindowNow;
    }
}

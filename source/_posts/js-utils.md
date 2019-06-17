---
title: js utils
date: 2019-06-17 15:39:07
tags: [js]
categories: [js]
---

#  js 时间戳转时间间隔

```js
// 时间转为时间戳
function date2timestamp(datetime) {
    var timestamp = new Date(Date.parse(datetime));
    timestamp = timestamp.getTime();
    timestamp = timestamp / 1000;
    return timestamp;
}

// 时间戳转时间
function timestamp2date(timestamp, mode) {

    if (parseInt(timestamp) > 999999999999){
        timestamp /= 1000;
    }

    var tt = new Date(parseInt(timestamp) * 1000).toLocaleString().replace(/:\d{1,2}$/, ' ').replace(/年|月/g, "-").replace(/日/g, " ").replace(/上午/g, "").replace(/下午/g, "");
    var date_arr = tt.split(" ");

    if (mode == 3) {
        var minute = 60;
        var hour = minute * 60;
        var day = hour * 24;
        var halfamonth = day * 15;
        var month = day * 30;
        var current_timestamp = parseInt(Date.parse(new Date()) / 1000);
        var diffValue = current_timestamp - timestamp;
        var monthC = diffValue / month;
        var dayC = diffValue / day;
        var hourC = diffValue / hour;
        var minC = diffValue / minute;


        /**
         * 2小时内--> 刚刚
         * 2小时-24小时内  --> *小时前
         * 超过24小时-1月内 -->  **天前
         * 超过1个月-1年内  -->  *月前
         * 超过1年 -->  1年前
         */

        if (monthC > 12){
            result = 1 + "年前";
        }
        else if (monthC >= 1) {
            result = parseInt(monthC) + "月前";
        }
        else if (dayC >= 1) {
            result = parseInt(dayC) + "天前";
        }
        else if (hourC >= 2) {
            result = parseInt(hourC) + "小时前";
        }
        else{
            result = "刚刚";
        }
        return result;
    }


    if (mode == 2) {
        var current_timestamp = parseInt(Date.parse(new Date()) / 1000);
        if ((current_timestamp - timestamp) > 7 * 24 * 60 * 60) {
            // 一周之前，显示日期
            return date_arr[0];
        } else {
            var d = new Date();
            var date = d.getFullYear() + "/" + (d.getMonth() + 1) + "/" + d.getDate();
            var b_date = date2timestamp(date + " 00:00:00");
            var e_date = date2timestamp(date + " 23:59:59");

            if (parseInt(timestamp) > parseInt(b_date) && parseInt(timestamp) < parseInt(e_date)) {
                // 今天,只显示时间
                return date_arr[1];
            }

            if (parseInt(timestamp) > parseInt(b_date - 24 * 60 * 60) && parseInt(timestamp) < parseInt(e_date - 24 * 60 * 60)) {
                // 昨天，显示昨天
                return "昨天";
            }

            // 显示周几
            var days = new Array("星期日","星期一","星期二","星期三","星期四","星期五","星期六");
            var day  = new Date(date_arr[0]).getDay();
            return days[day];
        }
    }


    if (mode == 1) {
        // 如果是当天，就不显示日期
        var d = new Date();
        var date = d.getFullYear() + "/" + (d.getMonth() + 1) + "/" + d.getDate();
        var b_date = date2timestamp(date + " 00:00:00");
        var e_date = date2timestamp(date + " 23:59:59");

        if (parseInt(timestamp) > parseInt(b_date) && parseInt(timestamp) < parseInt(e_date)) {
            return date_arr[1];
        }
        return tt;
    }


    return tt;
}
```


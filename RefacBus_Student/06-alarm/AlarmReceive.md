# AlarmReceive.kt

## 1) 역할
- `TimePlaceActivity`에서 예약 시 등록한 **정확 알람(AlarmManager)**의 브로드캐스트를 수신
- 알림 채널 생성(Oreo+) 및 도착 10분 전 알림 노티피케이션 표시

---

## 2) 핵심 로직 요약
- 브로드캐스트 수신 → 노티 생성: `onReceive()`에서 `NotificationManager` 획득 후 알림 표시
- 채널 보장: API 26+에서 `NotificationChannel("bus_reminder")` 동적 생성
- TimePlaceActivity 연동: `setExactAndAllowWhileIdle(triggerAt, pendingIntent)`로 예약 → 이 리시버가 트리거됨
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.alarm

import android.app.NotificationChannel
import android.app.NotificationManager
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.os.Build
import androidx.core.app.NotificationCompat
import com.example.refac_userbus.R

class AlarmReceive : BroadcastReceiver() {
    // 브로드캐스트 수신
    override fun onReceive(context: Context?, intent: Intent?) {
        val notificationManager = context?.getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager

        //채널 보장
        val builder = NotificationCompat.Builder(context, "bus_reminder")
            .setSmallIcon(R.drawable.smile)
            .setContentTitle("셔틀버스 알림")
            .setContentText("곧 셔틀버스가 도착합니다! 늦지않게 탑승해주세요.")
            .setPriority(NotificationCompat.PRIORITY_HIGH)

        //채널 생성
        if(Build.VERSION.SDK_INT >= Build.VERSION_CODES.O){
            val channel = NotificationChannel(
                "bus_reminder",
                "버스 알림",
                NotificationManager.IMPORTANCE_HIGH
            )
            notificationManager.createNotificationChannel(channel)
        }
        //노티 표시
        notificationManager.notify(1, builder.build())
    }
}
```
<details>


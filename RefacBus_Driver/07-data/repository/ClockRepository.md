# ClockRepository.kt

## 1) 역할
- 선택 노선/시간에 대한 **정류장별 예약자 수 집계**
- 운행 종료 시 **운전 종료 시각 저장** (drivers/{uid}/drived/{pushKey}/endTime)  :contentReference[oaicite:0]{index=0}

---

## 2) 핵심 로직 요약
- **정류장 목록 결정**: 노선명에 따라 고정 스테이션 리스트 구성  
- **카운팅 로직**: `users/*/reservations`를 순회하며 `route/time/date(오늘)` 일치 + 해당 정류장인 경우 카운트 증가  
- **반환 포맷**: `List<StationInfo(name, count)>`로 정렬된 정류장 순서 그대로 반환  
- **운행 종료 기록**: 현재 시간(`HH:mm:ss`)을 `endTime`으로 저장

<details>
<summary> 코드 보기 </summary>

```kotlin
class ClockRepository {
    private val db = FirebaseDatabase.getInstance()
    private val auth = FirebaseAuth.getInstance()

    // 정류장 목록 결정
    suspend fun fetchReservationCounts(route: String, time: String): List<StationInfo> {
        val stationNames = when (route) {
            "교내순환" -> listOf("정문", "B1", "C7", "C13", "D6", "A2(건너편)")
            "하양역->교내순환" -> listOf("하양역", "정문", "B1", "C7", "C13", "D6", "A2(건너편)")
            "안심역->교내순환" -> listOf("안심역(3번출구)", "정문", "B1", "C7", "C13", "D6", "A2(건너편)")
            "사월역->교내순환" -> listOf("사월역(3번출구)", "정문", "B1", "C7", "C13", "D6", "A2(건너편)")
            "A2->안심역->사월역" -> listOf("A2(건너편)", "안심역", "사월역")
            else -> emptyList()
        }

        val resultMap = stationNames.associateWith { 0 }.toMutableMap()
        val today = SimpleDateFormat("yy-MM-dd", Locale.getDefault()).format(Date())
        val snapshot = db.getReference("users").get().await()

        // 카운팅 로직
        for (user in snapshot.children) {
            val reservations = user.child("reservations").children
            for (reservation in reservations) {
                val resRoute = reservation.child("route").getValue(String::class.java)
                val resTime = reservation.child("time").getValue(String::class.java)
                val resDate = reservation.child("date").getValue(String::class.java)
                val station = reservation.child("station").getValue(String::class.java) ?: continue

                if (resRoute == route && resTime == time && resDate == today && station in resultMap) {
                    resultMap[station] = resultMap[station]!! + 1
                }
            }
        }
        // 반환 포맷
        return stationNames.map { StationInfo(it, resultMap[it] ?: 0) }
    }

    // 운행 종료 기록
    suspend fun saveEndTime(pushKey: String) {
        val uid = auth.currentUser?.uid ?: return
        val endTime = SimpleDateFormat("HH:mm:ss", Locale.getDefault()).format(Date())
        db.getReference("drivers").child(uid).child("drived").child(pushKey).child("endTime")
            .setValue(endTime).await()
    }
}

```
</details>

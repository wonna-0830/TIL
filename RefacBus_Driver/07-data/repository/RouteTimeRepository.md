# RouteTimeRepository.kt

## 1) 역할
- 고정(핀)된 노선과 시간표를 로드하여 화면에 제공
- 운행 시작 시 **운행 레코드 생성** & **pushKey(고유키) 반환**

---

## 2) 핵심 로직 요약
- **핀 노선 로드**: `/routes/*` 순회 → `isPinned=true`만 필터 → `{routeName: [times...]}` 맵으로 반환(시간 정렬)
- **운행 기록 저장**: `uniqueKey = "$route|$time|$date(yyyy-MM-dd)"`로 `drivers/{uid}/drived/{uniqueKey}`에 `DrivedRecord(route,time,"",date,uniqueKey)` 저장 → `Result.success(uniqueKey)`
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class RouteTimeRepository {
    private val db = FirebaseDatabase.getInstance()
    private val auth = FirebaseAuth.getInstance()

    // 핀 노선 로드
    suspend fun fetchPinnedRoutes(): Map<String, List<String>> {
        val snapshot = db.getReference("routes").get().await()
        val routeMap = mutableMapOf<String, List<String>>()

        for (routeSnapshot in snapshot.children) {
            val isPinned = routeSnapshot.child("isPinned").getValue(Boolean::class.java) ?: false
            if (!isPinned) continue

            val routeName = routeSnapshot.child("name").getValue(String::class.java) ?: continue
            val times = routeSnapshot.child("times").children.mapNotNull { it.getValue(String::class.java) }.sorted()
            routeMap[routeName] = times
        }

        return routeMap
    }

    //운행 기록 저장
    suspend fun saveDrivedRecord(route: String, time: String): Result<String> {
        val date = SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date())
        val uid = auth.currentUser?.uid ?: return Result.failure(Exception("로그인 정보 없음"))
        val uniqueKey = "$route|$time|$date"
        val record = DrivedRecord(route, time, "", date, uniqueKey)

        db.getReference("drivers").child(uid).child("drived").child(uniqueKey).setValue(record).await()
        return Result.success(uniqueKey)
    }
}

```
</details>

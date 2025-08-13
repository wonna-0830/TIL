# SelectBusListRepository.kt

## 1) 역할
- 현재 계정의 운행 기록 목록을 조회하여 화면에 제공
- 각 항목에 대해 **pushKey를 세팅**해 후속 화면 탐색을 용이하게 함 

---

## 2) 핵심 로직 요약
- **인증 확인**: UID 없으면 실패 반환
- **목록 조회**: `drivers/{uid}/drived` 전체 스냅샷을 `DrivedRecord` 리스트로 매핑
- **키 주입**: 각 child의 key를 `record.pushKey`에 설정 후 `Result.success(list)`
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class SelectBusListRepository {
    private val db = FirebaseDatabase.getInstance()
    private val auth = FirebaseAuth.getInstance()

    suspend fun fetchDrivedList(): Result<List<DrivedRecord>> {
        return try {
            //인증 확인
            val uid = auth.currentUser?.uid ?: return Result.failure(Exception("로그인 정보 없음"))
            val snapshot = db.getReference("drivers").child(uid).child("drived").get().await()

            // 목록 조회
            val list = snapshot.children.mapNotNull { snap ->
                val record = snap.getValue(DrivedRecord::class.java)
                //키 주입
                record?.apply { pushKey = snap.key ?: "" }
            }

            Result.success(list)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

```
</details>


# FinishRepository.kt

## 1) 역할
- 특정 운행(`pushKey`)의 노선/시간/종료시각 상세 조회
- 결과를 `Result<DrivedRecord>` 형태로 반환해 오류 처리 **일관성 유지**

---

## 2) 핵심 로직 요약
- **유저 확인**: `auth.currentUser?.uid` 없으면 실패 반환
- **데이터 조회**: `drivers/{uid}/drived/{pushKey}`에서 `DrivedRecord` 로드
- **가공/반환**: 성공 시 `Result.success(record)` / 실패 시 예외로 `Result.failure`
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class FinishRepository {
    private val db = FirebaseDatabase.getInstance()
    private val auth = FirebaseAuth.getInstance()

    suspend fun fetchDrivedRouteInfo(pushKey: String): Result<DrivedRecord> {
        return try {
            //유저 확인
            val uid = auth.currentUser?.uid ?: return Result.failure(Exception("로그인 정보 없음"))

            //데이터 조회
            val snapshot = db.getReference("drivers").child(uid).child("drived").child(pushKey).get().await()
            val record = snapshot.getValue(DrivedRecord::class.java)
                ?: return Result.failure(Exception("운행 정보를 불러올 수 없습니다."))

            // 가공/반환
            Result.success(DrivedRecord(record.route, record.time, record.endTime))

        } catch (e: Exception) {
            Result.failure(e)
        }
    }
}

```
</details>


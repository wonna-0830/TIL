# UserRepository.kt

## 1) 역할
- **로그인**: 이메일/비밀번호 인증 + 정지 계정(isBanned) 확인
- **회원가입**: 운전자 기본정보(name/email/joinDate) `/drivers/{uid}` 저장 

---

## 2) 핵심 로직 요약
- **login**
  - `signInWithEmailAndPassword` → UID 획득
  - `/drivers/{uid}/isBanned` 확인 → 정지 시 signOut() 후 실패, 정상 시 `Result.success(true)`

- **register**
  - `createUserWithEmailAndPassword` → UID/가입일(yyyy-MM-dd) 생성
  - `drivers/{uid}`에 `{ name, email, joinDate }` 저장 → `Result.success(Unit)`
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class UserRepository {
    private val auth: FirebaseAuth = FirebaseAuth.getInstance()
    private val db = FirebaseDatabase.getInstance()

    //로그인
    suspend fun login(email: String, password: String): Result<Boolean> {
        return try {
            //UID 획득
            val authResult = auth.signInWithEmailAndPassword(email, password).await()
            val uid = authResult.user?.uid ?: return Result.failure(Exception("UID 없음"))

            val snapshot = db.getReference("drivers").child(uid).get().await()
            val isBanned = snapshot.child("isBanned").getValue(Boolean::class.java) ?: false

            // 정지된 계정인 지 확인
            if (isBanned) {
                auth.signOut()
                Result.failure(Exception("정지된 계정입니다."))
            } else {
                Result.success(true)
            }
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

    //회원가입
    suspend fun register(email: String, password: String, name: String): Result<Unit> {
        return try {
            //UID/가입일 생성
            val authResult = auth.createUserWithEmailAndPassword(email, password).await()
            val uid = authResult.user?.uid ?: return Result.failure(Exception("UID 없음"))
            val joinDate = SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date())

            val driverInfo = mapOf(
                "name" to name,
                "email" to email,
                "joinDate" to joinDate
            )
            //{ name, email, joinDate } 저장
            db.getReference("drivers").child(uid).setValue(driverInfo).await()

            Result.success(Unit)
        } catch (e: Exception) {
            Result.failure(e)
        }
    }

}
```
</details>



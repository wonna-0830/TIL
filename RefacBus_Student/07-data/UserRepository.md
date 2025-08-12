# UserRepository.kt

## 1) 역할
- 이메일/비밀번호에 대한 로그인과 회원가입 구현
- 회원가입 성공 시 `users/{uid}`에 `name/email/joinDate` 저장
- 현재 사용자 조회/로그아웃
- 에러 시 `Result<T>`로 상위 계층에 전달

---

## 2) 핵심 로직 요약

### 로그인
- `if (uid != null) {` -> `signInWithEmailAndPassword`이후 세팅되면 if문 안 로직 실행
- `if (isBanned) {` -> isBanned가 true면(관리자 기능) 로그인 불가
- `signInWithEmailAndPassword → Result<String /*uid*/>`
  
<details>
<summary> 코드 보기 </summary>
```kotlin
fun login(
    email: String,
    password: String,
    callback: (Result<String>) -> Unit
) {
    auth.signInWithEmailAndPassword(email, password)
        .addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val uid = auth.currentUser?.uid
                if (uid != null) {
                    db.child(uid).get().addOnSuccessListener { snapshot ->
                        val isBanned = snapshot.child("isBanned").getValue(Boolean::class.java) ?: false
                        if (isBanned) {
                            auth.signOut()
                            callback(Result.failure(Exception("정지된 계정입니다.")))
                        } else {
                            callback(Result.success(uid))
                        }
                    }.addOnFailureListener {
                        callback(Result.failure(Exception("유저 정보 조회 실패")))
                    }
                } else {
                    callback(Result.failure(Exception("UID 없음")))
                }
            } else {
                callback(Result.failure(task.exception ?: Exception("로그인 실패")))
            }
        }
    }
```
</details>

### 회원가입
- `if (uid != null) {` -> `createUserWithEmailAndPassword` 이후 세팅이 완료되면 if 안 로직 실행
- `createUserWithEmailAndPassword → users/{uid} setValue (joinDate: yyyy-MM-dd)`

<details>
<summary> 코드 보기 </summary>
```kotlin
fun register(
    email: String,
    password: String,
    name: String,
    callback: (Result<Unit>) -> Unit
) {
    val joinDate = SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date())

    auth.createUserWithEmailAndPassword(email, password)
        .addOnCompleteListener { task ->
            if (task.isSuccessful) {
                val uid = auth.currentUser?.uid
                if (uid != null) {
                    val userData = mapOf(
                        "name" to name,
                        "email" to email,
                        "joinDate" to joinDate
                    )
                    db.child(uid).setValue(userData)
                        .addOnSuccessListener {
                            callback(Result.success(Unit))
                        }
                        .addOnFailureListener {
                            callback(Result.failure(it))
                        }
                } else {
                    callback(Result.failure(Exception("UID가 존재하지 않음")))
                }
            } else {
                callback(Result.failure(task.exception ?: Exception("회원가입 실패")))
            }
        }
    }
```
</details>


# LoginViewModel.kt

## 1) 역할
- 이메일/비밀번호 로그인 기능 구현
- 로그인 성공 시 UID 반환
- 로그인 실패 시 UID 반환
- View(Activity)와 Repository사이에서 중간 계층 역할 수행

---

## 2) 핵심 로직 요약
- 로그인 : `repository.login(email, password)` 호출 -> Result 형태로 성공/실패 전달
- 로딩 상태 표현 : `Result.failure(Exception("로딩 중..."))` -> UI에서 로딩바 처리
- 성공처리 : Repository에서 받아온 `Result.success(uid)`를 전달
- 실패처리 : Repository에서 받아온 `Result.failure()`를 전달
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.login

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import com.example.refac_userbus.data.UserRepository

class LoginViewModel : ViewModel() {
    private val repository = UserRepository()

    private val _loginResult = MutableLiveData<Result<String>>()
    val loginResult: LiveData<Result<String>> = _loginResult

    fun login(email: String, password: String) {
        _loginResult.value = Result.failure(Exception("로딩 중..."))  // 로딩 상태
        repository.login(email, password) { result ->
            _loginResult.postValue(result)
        }
    }
}
```
</details>
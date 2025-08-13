# RegisterViewModel.kt

## 1) 역할
- 이메일/비밀번호 회원가입 기능 구현
- 회원가입 성공 시 UID 반환
- 회원가입 실패 시 UID 반환
- View(Activity)와 Repository사이에서 중간 계층 역할 수행

---

## 2) 핵심 로직 요약
- **회원가입** : `repository.register(email, password, name)` 호출 -> Result 형태로 성공/실패 전달
- **로딩 상태 표현** : `Result.failure(Exception("로딩 중..."))` -> UI에서 로딩바 처리
- **성공처리** : Repository에서 받아온 `Result.success(Unit)`를 전달
- **실패처리** : Repository에서 받아온 `Result.failure()`를 전달
- **ViewModel – 상태 기반 로직** : 유효성 검증을 ViewModel에서 수행.
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_driverapp.feature.register


import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.refac_driverapp.data.repository.UserRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

class RegisterViewModel(private val repository: UserRepository) : ViewModel() {

    private val _registerState = MutableStateFlow<RegisterState>(RegisterState.Idle)
    val registerState: StateFlow<RegisterState> get() = _registerState

    fun register(email: String, password: String, name: String) {
        if (email.isBlank() || password.length < 8) {
            _registerState.value = RegisterState.Error("유효한 이메일과 8자 이상의 비밀번호를 입력해주세요.")
            return
        }

        _registerState.value = RegisterState.Loading

        viewModelScope.launch {
            val result = repository.register(email, password, name)
            _registerState.value = if (result.isSuccess) {
                RegisterState.Success
            } else {
                RegisterState.Error(result.exceptionOrNull()?.message ?: "알 수 없는 오류")
            }
        }
    }

    sealed class RegisterState {
        object Idle : RegisterState()
        object Loading : RegisterState()
        object Success : RegisterState()
        data class Error(val message: String) : RegisterState()
    }
}
```
</details>
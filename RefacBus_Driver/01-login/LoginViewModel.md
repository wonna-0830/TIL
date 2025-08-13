# LoginViewModel.kt

## 1) 역할
- 이메일/비밀번호 로그인 기능 구현
- 로그인 성공 시 UID 반환
- 로그인 실패 시 UID 반환
- View(Activity)와 Repository사이에서 중간 계층 역할 수행

---

## 2) 핵심 로직 요약
- **로그인** : `repository.login(email, password)` 호출 -> Result 형태로 성공/실패 전달
- **로딩 상태 표현** : `LoginUiState.Loading` -> UI에서 로딩바 처리
- **성공처리** : Repository에서 받아온 `Result.success(uid)`를 전달
- **실패처리** : Repository에서 받아온 `Result.failure()`를 전달
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_driverapp.viewmodel

import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.refac_driverapp.data.repository.UserRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

class LoginViewModel(private val repository: UserRepository) : ViewModel() {

    private val _loginState = MutableStateFlow<LoginUiState>(LoginUiState.Idle)
    val loginState: StateFlow<LoginUiState> get() = _loginState

    fun login(email: String, password: String) {
        _loginState.value = LoginUiState.Loading

        viewModelScope.launch {
            val result = repository.login(email, password)
            _loginState.value = if (result.isSuccess) {
                LoginUiState.Success
            } else {
                LoginUiState.Error(result.exceptionOrNull()?.message ?: "알 수 없는 오류")
            }
        }
    }

    sealed class LoginUiState {
        object Idle : LoginUiState()
        object Loading : LoginUiState()
        object Success : LoginUiState()
        data class Error(val message: String) : LoginUiState()
    }
}
```
</details>
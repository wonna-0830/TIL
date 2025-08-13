# FinishViewModel.kt

## 1) 역할
- 운행 종료 화면에서 운행 정보 로드
- 오류 메시지 관리
- Repository와 UI를 연결하는 중간 계층

---

## 2) 핵심 로직 요약
- **운행 정보 로드**: `repository.fetchDrivedRouteInfo(pushKey)` 호출 → 성공 시 `_routeInfo`에 저장
- **오류 처리**: 실패 시 `_error`에 메시지 저장
- **상태 관리**: `MutableStateFlow`로 운행 정보와 오류 상태 유지
  
<details>
<summary> 코드 보기 </summary>

```kotlin

```
</details>

class FinishViewModel(private val repository: FinishRepository) : ViewModel() {
    private val _routeInfo = MutableStateFlow<DrivedRecord?>(null)
    val routeInfo: StateFlow<DrivedRecord?> get() = _routeInfo

    //상태 관리
    private val _error = MutableStateFlow<String?>(null)
    val error: StateFlow<String?> get() = _error

    fun loadRouteInfo(pushKey: String) {
        viewModelScope.launch {
            // 운행 정보 로드
            val result = repository.fetchDrivedRouteInfo(pushKey)
            if (result.isSuccess) {
                _routeInfo.value = result.getOrNull()
            } else {
                //오류 처리
                _error.value = result.exceptionOrNull()?.message
            }
        }
    }
}

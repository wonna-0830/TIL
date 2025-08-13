# SelectBusListViewModel.kt

## 1) 역할
- 운행 기록 목록 조회 및 정렬(최신 날짜 우선)
- 오류 메시지 상태 관리
- Repository와 UI 사이의 **중간 계층** 역할

---

## 2) 핵심 로직 요약
- **목록 조회**: `repository.fetchDrivedList()` 호출 후 결과를 성공/실패로 분기
- **정렬 처리**: `yyyy-MM-dd`를 `Date`로 파싱해 내림차순 정렬 후 `_drivedList`에 반영
- **오류 처리**: 실패 시 예외 메시지를 `_error`에 저장하여 UI에서 구독
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class SelectBusListViewModel(private val repository: SelectBusListRepository) : ViewModel() {

    private val _drivedList = MutableStateFlow<List<DrivedRecord>>(emptyList())
    val drivedList: StateFlow<List<DrivedRecord>> get() = _drivedList

    private val _error = MutableStateFlow<String?>(null)
    val error: StateFlow<String?> get() = _error

    fun loadDrivedList() {
        viewModelScope.launch {
            //목록 조회
            val result = repository.fetchDrivedList()
            if (result.isSuccess) {
                //정렬 처리
                val sorted = result.getOrNull()?.sortedByDescending {
                    SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).parse(it.date)
                } ?: emptyList()
                _drivedList.value = sorted
            } else {
                // 오류 처리
                _error.value = result.exceptionOrNull()?.message
            }
        }
    }
}
```
</details>


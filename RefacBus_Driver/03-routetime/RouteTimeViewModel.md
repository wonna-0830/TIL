# RouteTimeViewModel.kt

## 1) 역할
- 고정 노선/시간 데이터 로드 및 상태(StateFlow)로 노출
- 현재 시각 이후의 시간만 필터링해 제공
- 운행 기록 저장 요청 결과를 Result<String>(pushKey)로 상태로 전달

---

## 2) 핵심 로직 요약
- **노선 로드**: `repository.fetchPinnedRoutes()`로 맵 형태(노선명 → 시간 리스트) 받아 `_routes`에 반영 
- **시간 필터**: `getFilteredTimes(route)`에서 Calendar로 현재 분(min) 계산 → h:mm을 분으로 환산해 현재 이후만 남기고 정렬 반환 
- **운행 저장**: `saveRecord(route, time)` → `repository.saveDrivedRecord(route, time)` 결과를 `_saveResult`에 담아 UI에 전달 
  
<details>
<summary> 코드 보기 </summary>
```kotlin
class RouteTimeViewModel(private val repository: RouteTimeRepository) : ViewModel() {

    private val _routes = MutableStateFlow<Map<String, List<String>>>(emptyMap())
    val routes: StateFlow<Map<String, List<String>>> get() = _routes

    private val _saveResult = MutableStateFlow<Result<String>?>(null)
    val saveResult: StateFlow<Result<String>?> get() = _saveResult

    //노선 로드
    fun loadRoutes() {
        viewModelScope.launch {
            _routes.value = repository.fetchPinnedRoutes()
        }
    }

    //시간 필터
    fun getFilteredTimes(route: String): List<String> {
        val now = Calendar.getInstance()
        val nowInMinutes = now.get(Calendar.HOUR_OF_DAY) * 60 + now.get(Calendar.MINUTE)

        return _routes.value[route]
            ?.filter { time ->
                time.contains(":") && run {
                    val (h, m) = time.split(":").mapNotNull { it.toIntOrNull() }
                    h * 60 + m > nowInMinutes
                }
            }
            ?.sorted()
            ?: emptyList()
    }

    //운행 저장
    fun saveRecord(route: String, time: String) {
        viewModelScope.launch {
            _saveResult.value = repository.saveDrivedRecord(route, time)
        }
    }
}

```
</details>

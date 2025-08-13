# ClockViewModel.kt

## 1) 역할
- 선택된 노선/시간에 따른 정류장별 예약자 수 조회
- 운행 종료 시 종료 시각 저장
- Repository와 Activity 사이의 중간 계층 역할

---

## 2) 핵심 로직 요약
- **정류장 정보 로드**: `repository.fetchReservationCounts(route, time)` 호출 후 `_stationList`에 저장
- **운행 종료**: `repository.saveEndTime(pushKey)` 호출 후 성공 시 콜백 실행
- **상태 관리**: `MutableStateFlow<List<StationInfo>>`로 정류장 정보 상태를 유지하고 UI에 제공
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class ClockViewModel(private val repository: ClockRepository) : ViewModel() {

    // 상태 관리
    private val _stationList = MutableStateFlow<List<StationInfo>>(emptyList())
    val stationList: StateFlow<List<StationInfo>> get() = _stationList

    // 정류장 정보 로드
    fun loadStationInfo(route: String, time: String) {
        viewModelScope.launch {
            val list = repository.fetchReservationCounts(route, time)
            _stationList.value = list
        }
    }

    // 운행 종료
    fun endDrive(pushKey: String, onSuccess: () -> Unit) {
        viewModelScope.launch {
            repository.saveEndTime(pushKey)
            onSuccess()
        }
    }
}

```
</details>


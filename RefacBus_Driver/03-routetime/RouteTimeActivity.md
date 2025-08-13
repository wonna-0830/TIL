# RouteTimeActivity.kt

## 1) 역할
- 노선/시간 스피너 UI 세팅 및 사용자 입력 수집
- ViewModel의 `routes/saveResult` 상태를 구독해 스피너 갱신, 저장 성공 시 ClockActivity로 이동
- 뒤로가기 두 번 눌러 앱 종료 처리

---

## 2) 핵심 로직 요약
- 노선 로드: `viewModel.loadRoutes()` 호출 → `routes` 구독하여 스피너에 노선 목록 바인딩 
- 시간 필터링: 노선 선택 시 `viewModel.getFilteredTimes(route)`로 현재 시각 이후 슬롯만 표시 
- 저장 요청: 버튼 클릭 시 유효성 검증 후 `viewModel.saveRecord(route, time)` 호출 
- 저장 결과 처리: `saveResult` 성공 → `ClockActivity`로 이동(노선/시간/날짜/pushKey 전달), 실패 → 토스트 안내 
- 종료 UX: 뒤로 두 번 눌러 종료(2초 타이머로 플래그 초기화) 
  
<details>
<summary> 코드 보기 </summary>
```kotlin
class RouteTimeActivity : AppCompatActivity() {

    //ViewModel DI
    private val viewModel: RouteTimeViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return RouteTimeViewModel(RouteTimeRepository()) as T
            }
        }
    }

    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_route_time)

        val spinnerRoute = findViewById<Spinner>(R.id.SpinnerRoute)
        val spinnerTime = findViewById<Spinner>(R.id.SpinnerTime)
        val btnCheck = findViewById<Button>(R.id.reservation)

        //노선 로드
        viewModel.loadRoutes()

        lifecycleScope.launchWhenStarted {
            viewModel.routes.collectLatest { routeMap ->
                val routeList = listOf("노선을 선택해주세요") + routeMap.keys
                val adapter = ArrayAdapter(this@RouteTimeActivity, android.R.layout.simple_spinner_item, routeList)
                spinnerRoute.adapter = adapter
            }
        }

        //시간 필터링
        spinnerRoute.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
            override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
                val route = spinnerRoute.selectedItem.toString()
                if (route == "노선을 선택해주세요") {
                    spinnerTime.adapter = null
                    return
                }

                val timeList = listOf("시간을 선택하세요") + viewModel.getFilteredTimes(route)
                val timeAdapter = ArrayAdapter(this@RouteTimeActivity, android.R.layout.simple_spinner_item, timeList)
                spinnerTime.adapter = timeAdapter
            }

            override fun onNothingSelected(parent: AdapterView<*>?) {}
        }

        // 저장 요청 (유효성 검사)
        btnCheck.setOnClickListener {
            val route = spinnerRoute.selectedItem?.toString()
            val time = spinnerTime.selectedItem?.toString()

            if (route == null || route == "노선을 선택해주세요") {
                Toast.makeText(this, "노선을 선택해주세요!", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            if (time == null || time == "시간을 선택하세요") {
                Toast.makeText(this, "시간을 선택해주세요!", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            viewModel.saveRecord(route, time)
        }

        // 저장 결과 처리
        lifecycleScope.launchWhenStarted {
            viewModel.saveResult.collectLatest { result ->
                result?.onSuccess { pushKey ->
                    val intent = Intent(this@RouteTimeActivity, ClockActivity::class.java)
                    intent.putExtra("route", spinnerRoute.selectedItem.toString())
                    intent.putExtra("time", spinnerTime.selectedItem.toString())
                    intent.putExtra("date", SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date()))
                    intent.putExtra("pushKey", pushKey)
                    startActivity(intent)
                }?.onFailure {
                    Toast.makeText(this@RouteTimeActivity, "운행 정보 저장 실패 😢", Toast.LENGTH_SHORT).show()
                }
            }
        }

        // 종료 UX
        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (doubleBackToExitPressedOnce) {
                    finishAffinity()
                } else {
                    doubleBackToExitPressedOnce = true
                    Toast.makeText(this@RouteTimeActivity, "한 번 더 누르면 앱이 종료됩니다.", Toast.LENGTH_SHORT).show()
                    Handler(Looper.getMainLooper()).postDelayed({
                        doubleBackToExitPressedOnce = false
                    }, 2000)
                }
            }
        })

    }
}

```
</details>
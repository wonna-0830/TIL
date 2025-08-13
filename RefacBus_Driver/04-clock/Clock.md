# Clock.kt

## 1) 역할
- 운행 시작 화면(UI) 구성 및 상태 유지
- 현재 시각을 1초 단위로 갱신하여 표시
- ViewModel에서 정류장별 예약자 수 리스트를 받아 RecyclerView에 표시
- 운행 종료 버튼 클릭 시 2단계 확인 후 종료 처리
- 운전 중 뒤로가기 방지 (오작동 방지)

---

## 2) 핵심 로직 요약
- **UI 초기화**: 전달받은 노선명/시간/pushKey를 화면에 표시
- **현재 시각 표시**: `Handler`와 `Runnable`을 이용해 1초마다 시각 업데이트
- **정류장 정보 로드**: `viewModel.loadStationInfo(route, time)` 호출 후 `stationList` 상태 구독하여 어댑터 갱신
- **뒤로가기 방지**: `onBackPressedDispatcher` 콜백에서 토스트만 띄움
- **운행 종료 처리**: 버튼 첫 클릭 시 안내, 두 번째 클릭 시 화면 꺼짐 방지 플래그 해제 → ViewModel의 `endDrive(pushKey)` 호출 후 FinishActivity로 이동

  
<details>
<summary> 코드 보기 </summary>

```kotlin
class ClockActivity : AppCompatActivity() {
    private val viewModel: ClockViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return ClockViewModel(ClockRepository()) as T
            }
        }
    }

    private lateinit var timeHandler: Handler
    private lateinit var timeRunnable: Runnable
    private var finishClickCount = 0

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_clock)
        window.addFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)

        //UI 초기화
        val selectedRoute = intent.getStringExtra("route") ?: ""
        val selectedTime = intent.getStringExtra("time") ?: ""
        val pushKey = intent.getStringExtra("pushKey") ?: ""
        val recyclerView = findViewById<RecyclerView>(R.id.recyclerStations)
        val textRouteName = findViewById<TextView>(R.id.textRouteName)
        val textCurrentTime = findViewById<TextView>(R.id.textCurrentTime)
        val btnCheck = findViewById<Button>(R.id.startButton)

        recyclerView.layoutManager = LinearLayoutManager(this)
        textRouteName.text = "$selectedRoute ($selectedTime)"

        // 현재 시간 표시
        timeHandler = Handler(Looper.getMainLooper())
        timeRunnable = object : Runnable {
            override fun run() {
                val now = SimpleDateFormat("HH:mm:ss", Locale.getDefault()).format(Date())
                textCurrentTime.text = now
                timeHandler.postDelayed(this, 1000)
            }
        }
        timeHandler.post(timeRunnable)

        // ViewModel로부터 리스트 받아오기
        lifecycleScope.launchWhenStarted {
            viewModel.stationList.collectLatest { list ->
                recyclerView.adapter = ClockAdapter(list)
            }
        }

        // 정류장 정보 로드
        viewModel.loadStationInfo(selectedRoute, selectedTime)

        //뒤로가기 방지
        onBackPressedDispatcher.addCallback(this) {
            Toast.makeText(this@ClockActivity, "운전 중에는 뒤로가기가 비활성화됩니다.", Toast.LENGTH_SHORT).show()
        }

        //운행 종료 처리
        btnCheck.setOnClickListener {
            if (finishClickCount == 0) {
                Toast.makeText(this, "한 번 더 누르면 운행이 종료됩니다.", Toast.LENGTH_SHORT).show()
                finishClickCount++
            } else {
                window.clearFlags(WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON)
                viewModel.endDrive(pushKey) {
                    val intent = Intent(this, FinishActivity::class.java)
                    intent.putExtra("pushKey", pushKey)
                    startActivity(intent)
                    finish()
                }
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        timeHandler.removeCallbacks(timeRunnable)
    }
}


```
</details>



# FinishActivity.kt

## 1) 역할
- 운행 종료 후 운행 정보(노선, 시간, 종료시간) 표시
- 로그아웃, 노선 재선택, 앱 종료, 버스 예약 목록 보기 기능 제공
- 뒤로가기 2회 클릭 시 앱 종료 처리
- ViewModel의 상태를 구독해 운행 정보와 오류 메시지를 UI에 반영

---

## 2) 핵심 로직 요약
- **운행 정보 로드**: `viewModel.loadRouteInfo(pushKey)` 호출 → `routeInfo` 구독하여 UI 업데이트
- **오류 처리**: `error` 상태 구독하여 Toast로 표시
- **로그아웃**: FirebaseAuth 로그아웃 + SharedPreferences 초기화 후 로그인 화면으로 이동
- **노선 재선택**: RouteTimeActivity로 화면 전환
- **앱 종료**: `finishAffinity()` + `System.exit(0)`
- **버스 예약 목록**: SelectBusListActivity로 이동
- **뒤로가기 UX**: 2초 내 두 번 누르면 앱 종료
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class FinishActivity : AppCompatActivity() {

    private val viewModel: FinishViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return FinishViewModel(FinishRepository()) as T
            }
        }
    }

    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_finish)

        val pushKey = intent.getStringExtra("pushKey") ?: ""
        val drivedRoute = findViewById<TextView>(R.id.drivedRoute)
        val drivedTime = findViewById<TextView>(R.id.drivedTime)
        val finishTime = findViewById<TextView>(R.id.finishTime)

        val btnLogout = findViewById<Button>(R.id.logOutButton)
        val btnSelectRoute = findViewById<Button>(R.id.nextButton)
        val btnExitApp = findViewById<Button>(R.id.finishApp)
        val btnSelectBus = findViewById<Button>(R.id.selectBus)

        // 운행 정보 로드
        viewModel.loadRouteInfo(pushKey)

        lifecycleScope.launchWhenStarted {
            viewModel.routeInfo.collectLatest { info ->
                info?.let {
                    drivedRoute.text = "노선: ${it.route}"
                    drivedTime.text = "시간: ${it.time}"
                    finishTime.text = "종료시간: ${it.endTime}"
                }
            }
        }

        //오류 처리
        lifecycleScope.launchWhenStarted {
            viewModel.error.collectLatest {
                it?.let { msg -> Toast.makeText(this@FinishActivity, msg, Toast.LENGTH_SHORT).show() }
            }
        }

        //로그아웃
        btnLogout.setOnClickListener {
            AlertDialog.Builder(this)
                .setTitle("로그아웃")
                .setMessage("정말 로그아웃하시겠습니까?")
                .setPositiveButton("예") { _, _ ->
                    com.google.firebase.auth.FirebaseAuth.getInstance().signOut()
                    getSharedPreferences("login_prefs", MODE_PRIVATE).edit().clear().apply()

                    val intent = Intent(this, LoginActivity::class.java)
                    intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
                    startActivity(intent)
                }
                .setNegativeButton("아니오", null)
                .show()
        }

        //노선 선택 화면 가기
        btnSelectRoute.setOnClickListener {
            AlertDialog.Builder(this)
                .setTitle("노선 선택")
                .setMessage("처음으로 돌아가시겠습니까?")
                .setPositiveButton("예") { _, _ ->
                    val intent = Intent(this, RouteTimeActivity::class.java)
                    intent.flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
                    startActivity(intent)
                }
                .setNegativeButton("아니오", null)
                .show()
        }

        // 앱 종료
        btnExitApp.setOnClickListener {
            AlertDialog.Builder(this)
                .setTitle("앱 종료")
                .setMessage("정말 종료하시겠습니까?")
                .setPositiveButton("예") { _, _ ->
                    finishAffinity()
                    System.exit(0)
                }
                .setNegativeButton("아니오", null)
                .show()
        }

        // 버스 예약 목록 화면 가기
        btnSelectBus.setOnClickListener {
            val intent = Intent(this, SelectBusListActivity::class.java)
            startActivity(intent)
            finish()
        }

        // 뒤로가기 UX
        onBackPressedDispatcher.addCallback(this) {
            if (doubleBackToExitPressedOnce) {
                finishAffinity()
                return@addCallback
            }

            doubleBackToExitPressedOnce = true
            Toast.makeText(this@FinishActivity, "한 번 더 누르면 앱이 종료됩니다.", Toast.LENGTH_SHORT).show()

            Handler(Looper.getMainLooper()).postDelayed({
                doubleBackToExitPressedOnce = false
            }, 2000)
        }
    }
}
```
</details>



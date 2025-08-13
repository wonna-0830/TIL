# SelectBusListActivity.kt

## 1) 역할
- 운행 기록(날짜/노선/시간)을 리스트로 표시
- 비어있을 때 “운행 기록 없음” 안내 처리
- 항목 클릭 시 해당 운행의 상세(종료 정보) 화면으로 이동
- 최신 운행으로 빠르게 돌아가기(뒤로 가기 버튼) 지원
- 뒤로가기 두 번 입력 시 앱 종료 UX 처리  :contentReference[oaicite:0]{index=0}

---

## 2) 핵심 로직 요약
- **데이터 로드**: `viewModel.loadDrivedList()` 호출 → `drivedList` 구독하며 ProgressBar/빈 상태/리스트 토글  :contentReference[oaicite:1]{index=1}
- **아이템 클릭**: `DriveAdapter`의 `onItemClick`으로 `FinishActivity`에 `pushKey` 전달 후 이동  :contentReference[oaicite:2]{index=2}
- **오류 처리**: `error` 상태 구독 → Toast로 표시  :contentReference[oaicite:3]{index=3}
- **최신 기록으로 복귀**: “뒤로가기” 버튼 클릭 시 가장 최근 날짜 기록 찾아 `FinishActivity`로 이동  :contentReference[oaicite:4]{index=4}
- **종료 UX**: 뒤로가기 2회 입력 시 `finishAffinity()`로 종료  :contentReference[oaicite:5]{index=5}

<details>
<summary> 코드 보기 </summary>

```kotlin
class SelectBusListActivity : AppCompatActivity() {

    private val viewModel: SelectBusListViewModel by viewModels {
        object : ViewModelProvider.Factory {
            override fun <T : ViewModel> create(modelClass: Class<T>): T {
                return SelectBusListViewModel(SelectBusListRepository()) as T
            }
        }
    }

    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_selectbuslist)

        val recyclerView = findViewById<RecyclerView>(R.id.recyclerView)
        val progressBar = findViewById<ProgressBar>(R.id.progressBar)
        val textNoReservation = findViewById<TextView>(R.id.textNoReservation)
        val backBtn = findViewById<Button>(R.id.btn_home)

        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.addItemDecoration(RecyclerViewDecoration(20))

        progressBar.visibility = View.VISIBLE

        //데이터 로드
        viewModel.loadDrivedList()

        lifecycleScope.launchWhenStarted {
            viewModel.drivedList.collectLatest { list ->
                progressBar.visibility = View.GONE

                if (list.isEmpty()) {
                    textNoReservation.visibility = View.VISIBLE
                    recyclerView.visibility = View.GONE
                } else {
                    textNoReservation.visibility = View.GONE
                    recyclerView.visibility = View.VISIBLE
                    recyclerView.adapter = DriveAdapter(
                        ArrayList(list),
                        onListEmpty = {
                            textNoReservation.visibility = View.VISIBLE
                            recyclerView.visibility = View.GONE
                        },
                        //아이템 클릭
                        onItemClick = { record ->
                            val intent = Intent(this@SelectBusListActivity, FinishActivity::class.java)
                            intent.putExtra("pushKey", record.pushKey)
                            startActivity(intent)
                            finish()
                        }
                    )
                }
            }
        }

        //오류 처리
        lifecycleScope.launchWhenStarted {
            viewModel.error.collectLatest {
                it?.let { msg ->
                    progressBar.visibility = View.GONE
                    Toast.makeText(this@SelectBusListActivity, msg, Toast.LENGTH_SHORT).show()
                }
            }
        }

        //뒤로가기 버튼
        backBtn.setOnClickListener {
            val recent = viewModel.drivedList.value.maxByOrNull {
                SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).parse(it.date)
            }

            if (recent != null) {
                val intent = Intent(this, FinishActivity::class.java)
                intent.putExtra("pushKey", recent.pushKey)
                startActivity(intent)
                finish()
            } else {
                Toast.makeText(this, "운행 기록이 없습니다.", Toast.LENGTH_SHORT).show()
            }
        }

        // 종료 UX
        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (doubleBackToExitPressedOnce) {
                    finishAffinity()
                } else {
                    doubleBackToExitPressedOnce = true
                    Toast.makeText(this@SelectBusListActivity, "한 번 더 누르면 앱이 종료됩니다.", Toast.LENGTH_SHORT).show()
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



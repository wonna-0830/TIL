# SelectBusListActivity.kt

## 1) 역할
- Activity ↔ ViewModel: 내 예약 목록 `LiveData` 관찰해 로딩/빈 상태/리스트 표시 제어
- Activity ↔ Adapter: `ReservationAdapter.updateData()`로 데이터 반영, onListEmpty 콜백으로 빈 상태 UI 전환
- Activity ↔ 다른 화면/설정: FAB 팝업(외부 링크/테마 토글/로그아웃), 홈 버튼(RouteChoose), 뒤로 두 번 종료

---

## 2) 핵심 로직 요약
- 목록 관찰 & UI 전환: `reservations.observe` → `progressBar` 숨김, `textNoReservation`/`recyclerView` 토글
- Adapter 연결: `LinearLayoutManager` + `ReservationAdapter { onListEmpty }` 등록
- 데이터 로드 트리거: `viewModel.loadReservations()` 호출(실시간 구독은 VM에서 처리)
- 팝업 메뉴 상호작용: 테마 토글(SharedPreferences), 예약화면 재진입, 로그아웃(+Login으로 이동)
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.selectbuslist

import android.content.Intent
import android.net.Uri
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.view.View
import android.widget.*
import androidx.activity.OnBackPressedCallback
import androidx.activity.viewModels
import androidx.appcompat.app.AppCompatActivity
import androidx.appcompat.app.AppCompatDelegate
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.example.refac_userbus.R
import com.example.refac_userbus.adapter.ReservationAdapter
import com.example.refac_userbus.design.RecyclerViewDecoration
import com.example.refac_userbus.feature.login.LoginActivity
import com.example.refac_userbus.feature.routechoose.RouteChooseActivity
import com.google.android.material.floatingactionbutton.FloatingActionButton

class SelectBusListActivity : AppCompatActivity() {

    private lateinit var recyclerView: RecyclerView
    private lateinit var progressBar: ProgressBar
    private lateinit var textNoReservation: TextView
    private val viewModel: SelectBusListViewModel by viewModels()
    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_selectbuslist)

        val prefs = getSharedPreferences("MyApp", MODE_PRIVATE)
        val isDarkMode = prefs.getBoolean("dark_mode", false)
        AppCompatDelegate.setDefaultNightMode(
            if (isDarkMode) AppCompatDelegate.MODE_NIGHT_NO else AppCompatDelegate.MODE_NIGHT_YES
        )

        recyclerView = findViewById(R.id.recyclerView)
        progressBar = findViewById(R.id.progressBar)
        textNoReservation = findViewById(R.id.textNoReservation)

        //어댑터 준비
        val adapter = ReservationAdapter {
            textNoReservation.visibility = View.VISIBLE
            recyclerView.visibility = View.GONE
        }

        recyclerView.layoutManager = LinearLayoutManager(this)
        recyclerView.adapter = adapter
        recyclerView.addItemDecoration(RecyclerViewDecoration(20))

        //상태 관찰
        viewModel.reservations.observe(this) { list ->
            progressBar.visibility = View.GONE
            if (list.isEmpty()) {
                textNoReservation.visibility = View.VISIBLE
                recyclerView.visibility = View.GONE
            } else {
                textNoReservation.visibility = View.GONE
                recyclerView.visibility = View.VISIBLE
                adapter.updateData(list)
            }
        }

        //데이터 로드 트리거
        viewModel.loadReservations()

        findViewById<FloatingActionButton>(R.id.fab).setOnClickListener {
            showPopupMenu(it)
        }

        findViewById<Button>(R.id.btn_home).setOnClickListener {
            val intent = Intent(this, RouteChooseActivity::class.java)
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_NEW_TASK)
            startActivity(intent)
            finish()
        }

        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (doubleBackToExitPressedOnce) {
                    finishAffinity()
                    return
                }
                doubleBackToExitPressedOnce = true
                Toast.makeText(this@SelectBusListActivity, "한 번 더 누르면 앱이 종료됩니다.", Toast.LENGTH_SHORT).show()
                Handler(Looper.getMainLooper()).postDelayed({ doubleBackToExitPressedOnce = false }, 2000)
            }
        })
    }

    //팝업 메뉴 상호작용
    private fun showPopupMenu(v: View) {
        val popupMenu = PopupMenu(this, v)
        popupMenu.menuInflater.inflate(R.menu.popup_menu, popupMenu.menu)

        val prefs = getSharedPreferences("MyApp", MODE_PRIVATE)
        val isDarkMode = prefs.getBoolean("dark_mode", false)
        val darkModeItem = popupMenu.menu.findItem(R.id.menu_item_4)
        darkModeItem.title = if (isDarkMode) "다크모드로 변경" else "라이트모드로 변경"

        popupMenu.setOnMenuItemClickListener { item ->
            when (item.itemId) {
                R.id.menu_item_1 -> {
                    val url = "https://www.cu.ac.kr/life/welfare/schoolbus"
                    startActivity(Intent(Intent.ACTION_VIEW, Uri.parse(url)))
                    true
                }
                R.id.menu_item_2 -> {
                    startActivity(Intent(this, SelectBusListActivity::class.java))
                    finish()
                    true
                }
                R.id.menu_item_3 -> {
                    prefs.edit().clear().apply()
                    viewModel.logout()
                    startActivity(Intent(this, LoginActivity::class.java).apply {
                        flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
                    })
                    finish()
                    true
                }
                R.id.menu_item_4 -> {
                    prefs.edit().putBoolean("dark_mode", !isDarkMode).apply()
                    AppCompatDelegate.setDefaultNightMode(
                        if (isDarkMode) AppCompatDelegate.MODE_NIGHT_YES else AppCompatDelegate.MODE_NIGHT_NO
                    )
                    recreate()
                    true
                }
                else -> false
            }
        }
        popupMenu.show()
    }
}

```
<details>



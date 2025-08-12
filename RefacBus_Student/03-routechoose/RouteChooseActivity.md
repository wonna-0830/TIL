# RoutChooseActivity.kt

## 1) 역할
- 사용자 이름 표시 및 즐겨찾기(고정)된 노선 목록 동적 버튼 생성
- 각 노선 버튼 클릭 시 선택된 노선 정보를 TimePlaceActivity로 전달
- FAB 팝업 메뉴를 통한 외부 링크, 예약 확인 화면 이동, 다크모드 토글
- 로그아웃 처리 및 앱 종료 제어

---

## 2) 핵심 로직 요약
- 사용자 이름 로드: `viewModel.loadUserName()` 호출 후 LiveData 관찰하여 `TextView`에 표시
- 노선 목록 로드 및 동적 버튼 생성: `viewModel.loadPinnedRoutes()`로 고정 노선 조회 → `LinearLayout`에 버튼 동적 추가
- 노선 선택 이벤트: 버튼 클릭 시 노선명, 이미지명, 정류장, 시간 리스트를 Intent Extra로 담아 `TimePlaceActivity` 실행
- FAB 팝업 메뉴 기능:
  - 학교 스쿨버스 노선 페이지 열기
  - 예약 확인 화면(`SelectBusListActivity`) 이동
  - 다크모드/라이트모드 전환 및 상태 저장
- 로그아웃 처리: `SharedPreferences` 초기화 → `ViewModel.logout()` → `LoginActivity` 이동
- 뒤로가기 두 번 종료: 2초 내 두 번 누르면 앱 종료 (`finishAffinity()`)
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.routechoose

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
import com.example.refac_userbus.R
import com.example.refac_userbus.feature.selectbuslist.SelectBusListActivity
import com.example.refac_userbus.feature.timeplace.TimePlaceActivity
import com.example.refac_userbus.feature.login.LoginActivity
import com.google.android.material.floatingactionbutton.FloatingActionButton

class RouteChooseActivity : AppCompatActivity() {
    private val viewModel: RouteChooseViewModel by viewModels()
    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        val prefs = getSharedPreferences("MyApp", MODE_PRIVATE)
        val isDarkMode = prefs.getBoolean("dark_mode", false)
        AppCompatDelegate.setDefaultNightMode(
            if (isDarkMode) AppCompatDelegate.MODE_NIGHT_NO
            else AppCompatDelegate.MODE_NIGHT_YES
        )

        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_routechoose)

        val nameTextView = findViewById<TextView>(R.id.userName)
        val routeContainer = findViewById<LinearLayout>(R.id.routeContainer)
        val fab = findViewById<FloatingActionButton>(R.id.fab)
        val logoutBtn = findViewById<Button>(R.id.btn_logout)

        // 사용자 이름
        viewModel.userName.observe(this) { name ->
            nameTextView.text = name
        }

        // 노선 목록 버튼으로 동적 생성
        viewModel.routes.observe(this) { routes ->
            routeContainer.removeAllViews()
            for (route in routes) {
                val button = Button(this).apply {
                    text = route.name
                    layoutParams = LinearLayout.LayoutParams(
                        LinearLayout.LayoutParams.MATCH_PARENT, 0, 1f
                    ).apply { setMargins(0, 8, 0, 8) }

                    //노선 선택 이벤트
                    setOnClickListener {
                        val intent = Intent(this@RouteChooseActivity, TimePlaceActivity::class.java).apply {
                            putExtra("EXTRA_ROUTE_NAME", route.name)
                            putExtra("EXTRA_IMAGE_NAME", route.image)
                            putStringArrayListExtra("EXTRA_STATIONS", ArrayList(route.stops))
                            putStringArrayListExtra("EXTRA_TIMES", ArrayList(route.times))
                        }
                        startActivity(intent)
                    }
                }
                routeContainer.addView(button)
            }
        }

        // 사용자 이름 로드
        viewModel.loadUserName()
        // 고정 노선 조회 후 해당 노선 버튼을 동적으로 생성
        viewModel.loadPinnedRoutes()


        // 팝업 메뉴
        fab.setOnClickListener { showPopupMenu(it) }

        // 로그아웃
        logoutBtn.setOnClickListener {
            prefs.edit().clear().apply()
            viewModel.logout()
            startActivity(Intent(this, LoginActivity::class.java).apply {
                flags = Intent.FLAG_ACTIVITY_NEW_TASK or Intent.FLAG_ACTIVITY_CLEAR_TASK
            })
            finish()
        }

        // 뒤로 두 번 누르면 종료
        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (doubleBackToExitPressedOnce) {
                    finishAffinity()
                } else {
                    doubleBackToExitPressedOnce = true
                    Toast.makeText(this@RouteChooseActivity, "한 번 더 누르면 앱이 종료됩니다.", Toast.LENGTH_SHORT).show()
                    Handler(Looper.getMainLooper()).postDelayed({ doubleBackToExitPressedOnce = false }, 2000)
                }
            }
        })
    }

    private fun showPopupMenu(v: View) {
        val popupMenu = PopupMenu(this, v)
        popupMenu.menuInflater.inflate(R.menu.popup_menu, popupMenu.menu)

        popupMenu.menu.findItem(R.id.menu_item_3).isVisible = false // 로그아웃 메뉴 숨기기

        val isDarkMode = getSharedPreferences("MyApp", MODE_PRIVATE).getBoolean("dark_mode", false)
        val darkModeItem = popupMenu.menu.findItem(R.id.menu_item_4)
        darkModeItem.title = if (isDarkMode) "다크모드로 변경" else "라이트모드로 변경"

        popupMenu.setOnMenuItemClickListener { item ->
            when (item.itemId) {
                R.id.menu_item_1 -> {
                    startActivity(Intent(Intent.ACTION_VIEW, Uri.parse("https://www.cu.ac.kr/life/welfare/schoolbus")))
                    true
                }
                R.id.menu_item_2 -> {
                    startActivity(Intent(this, SelectBusListActivity::class.java))
                    finish()
                    true
                }
                R.id.menu_item_4 -> {
                    val prefs = getSharedPreferences("MyApp", MODE_PRIVATE)
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


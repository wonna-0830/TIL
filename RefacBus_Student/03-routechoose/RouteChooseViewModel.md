# RouteChooseViewModel.kt

## 1) 역할
- 사용자 이름(`users/{uid}/name`) 조회 후 UI에 전달(LiveData)
- 고정(핀)된 노선 목록을 DB에서 조회해 화면에 전달(LiveData)
- 로그아웃 처리(`FirebaseAuth.signOut`)

---

## 2) 핵심 로직 요약
- 사용자 이름 로드: `FirebaseAuth.currentUser.uid` 조회 → `users/{uid}/name`를 `get()` → 성공 시 `_userName` 설정, 실패 시 기본 문구
- 고정 노선 로드: `routes`에서 `orderByChild("isPinned").equalTo(true)`로 필터 → `name/imageName/stops/times` 파싱 →` _routes`에 게시
- 로그아웃: `FirebaseAuth.getInstance().signOut()` 호출
  
<details>
<summary> 코드 보기 </summary>

```kotlin

```
<details>

package com.example.refac_userbus.feature.routechoose

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.database.FirebaseDatabase

class RouteChooseViewModel : ViewModel() {

    private val _userName = MutableLiveData<String>()
    val userName: LiveData<String> = _userName

    private val _routes = MutableLiveData<List<RouteData>>()
    val routes: LiveData<List<RouteData>> = _routes

    //사용자 이름 로드
    fun loadUserName() {
        val uid = FirebaseAuth.getInstance().currentUser?.uid ?: return
        FirebaseDatabase.getInstance().getReference("users").child(uid).child("name")
            .get()
            .addOnSuccessListener {
                _userName.value = it.getValue(String::class.java) ?: "알 수 없음"
            }
            .addOnFailureListener {
                _userName.value = "이름 불러오기 실패"
            }
    }

    //고정 노선 로드
    fun loadPinnedRoutes() {
        val ref = FirebaseDatabase.getInstance().getReference("routes")
        ref.orderByChild("isPinned").equalTo(true)
            .get()
            .addOnSuccessListener { snapshot ->
                val result = mutableListOf<RouteData>()
                for (routeSnapshot in snapshot.children) {
                    val name = routeSnapshot.child("name").getValue(String::class.java) ?: continue
                    val image = routeSnapshot.child("imageName").getValue(String::class.java) ?: ""
                    val stops = routeSnapshot.child("stops").children.mapNotNull { it.getValue(String::class.java) }
                    val times = routeSnapshot.child("times").children.mapNotNull { it.getValue(String::class.java) }

                    result.add(RouteData(name, image, stops, times))
                }
                _routes.value = result
            }
            .addOnFailureListener {
                _routes.value = emptyList()
            }
    }
    //로그아웃
    fun logout() {
        FirebaseAuth.getInstance().signOut()
    }
}

data class RouteData(
    val name: String,
    val image: String,
    val stops: List<String>,
    val times: List<String>
)

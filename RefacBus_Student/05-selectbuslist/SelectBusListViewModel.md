# SelectBusListViewModel.kt

## 1) 역할
- ViewModel ↔ Firebase: `/users/{uid}/reservations`를 `ValueEventListener`로 구독 → 도메인 모델 리스트 생성/정렬
- ViewModel ↔ Activity: 가공된 리스트를 `LiveData`로 노출(실시간 반영)
- 로그아웃: `FirebaseAuth.signOut()` 제공(호출은 Activity에서)

---

## 2) 핵심 로직 요약
- 예약 로드: 현재 UID로 경로 생성 → 스냅샷 순회하며 `ReservationData` 파싱 + `pushKey` 보존
- 정렬 & 게시: 날짜 기준 정렬 후 `_reservations.value` 업데이트(실시간 UI 반영)
- 에러 처리: `onCancelled` 시 빈 리스트 게시(안정적인 UI 상태 유지)
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.selectbuslist

import androidx.lifecycle.LiveData
import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import com.example.refac_userbus.data.model.ReservationData
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.database.*

class SelectBusListViewModel : ViewModel() {

    private val _reservations = MutableLiveData<List<ReservationData>>()
    val reservations: LiveData<List<ReservationData>> = _reservations

    //예약 로드
    fun loadReservations() {
        val uid = FirebaseAuth.getInstance().currentUser?.uid ?: return
        val ref = FirebaseDatabase.getInstance().reference
            .child("users")
            .child(uid)
            .child("reservations")

        ref.addValueEventListener(object : ValueEventListener {
            override fun onDataChange(snapshot: DataSnapshot) {
                val newList = mutableListOf<ReservationData>()
                for (reservationSnapshot in snapshot.children) {
                    val reservation = reservationSnapshot.getValue(ReservationData::class.java)
                    reservation?.let {
                        it.pushKey = reservationSnapshot.key ?: ""
                        newList.add(it)
                    }
                }
                //정렬 & 게시
                newList.sortByDescending { it.date }
                _reservations.value = newList
            }

            //에러 처리
            override fun onCancelled(error: DatabaseError) {
                _reservations.value = emptyList()
            }
        })
    }

    fun logout() {
        FirebaseAuth.getInstance().signOut()
    }
}

```
<details>


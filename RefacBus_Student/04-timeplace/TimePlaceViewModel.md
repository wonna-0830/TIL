# TimePlaceViewModel.kt

## 1) 역할
- 노선 이름으로 라우트 메타(이미지/시간/정류장) 조회 후 스피너와 이미지 뷰 갱신
- 시간/정류장 선택값 검증, 하루 1건(등교/하교 각각) 중복 예약 제한 적용
- 예약 생성(`/users/{uid}/reservations`에 push), 성공 시 선택한 시간 반환
- 알람용 시각을 `millis`로 계산해 Activity가 알람 등록에 사용하도록 제공 

---

## 2) 핵심 로직 요약
- 현재 이후 시간만 스피너에 노출: `HH:mm` 파싱 → 현재 시각 이상만 필터링
- 선택값 검증: “시간/장소를 선택하세요” 가드 문자열로 미선택 방지
- 하루 1건 제한: 당일 예약 조회 → 등교 노선 리스트/하교 노선으로 분리해 중복 차단
- 예약 생성: `ReservationData(route, time, station, date)`를 push, 성공 시 선택 시간 콜백
- 알람 시각 계산: `"HH:mm"`을 오늘 날짜 기준 `Calendar`에 적용해 `timeInMillis` 반환
  
<details>
<summary> 코드 보기 </summary>

```kotlin
package com.example.refac_userbus.feature.timeplace

import android.content.Context
import android.widget.ArrayAdapter
import android.widget.ImageView
import android.widget.Spinner
import androidx.lifecycle.ViewModel
import com.example.refac_userbus.R
import com.example.refac_userbus.data.model.ReservationData
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.database.*
import java.text.SimpleDateFormat
import java.util.*

class TimePlaceViewModel : ViewModel() {

    fun logout() {
        FirebaseAuth.getInstance().signOut()
    }

    fun loadRouteInfo(
        routeName: String,
        context: Context,
        imageView: ImageView,
        timeSpinner: Spinner,
        stationSpinner: Spinner
    ) {
        val routeRef = FirebaseDatabase.getInstance().getReference("routes")

        //노선 로딩 및 스피너 세팅
        routeRef.orderByChild("name").equalTo(routeName)
            .addListenerForSingleValueEvent(object : ValueEventListener {
                override fun onDataChange(snapshot: DataSnapshot) {
                    for (routeSnapshot in snapshot.children) {
                        val imageName = routeSnapshot.child("imageName").getValue(String::class.java) ?: ""
                        val stops = routeSnapshot.child("stops").children.mapNotNull { it.getValue(String::class.java) }
                        val times = routeSnapshot.child("times").children.mapNotNull { it.getValue(String::class.java) }

                        imageView.setImageResource(getImageResIdFromRouteName(routeName))

                        //현재 이후 시간만 필터링
                        val timeList = mutableListOf("시간을 선택하세요")
                        val now = Calendar.getInstance()
                        val currentHour = now.get(Calendar.HOUR_OF_DAY)
                        val currentMinute = now.get(Calendar.MINUTE)

                        for (time in times) {
                            val hour = time.substring(0, 2).toIntOrNull()
                            val minute = time.substring(3).toIntOrNull()
                            if (hour != null && minute != null &&
                                (hour > currentHour || (hour == currentHour && minute >= currentMinute))) {
                                timeList.add(time)
                            }
                        }


                        val timeAdapter = ArrayAdapter(context, R.layout.spinner_item_black, timeList)
                        timeAdapter.setDropDownViewResource(R.layout.spinner_dropdown)
                        timeSpinner.adapter = timeAdapter

                        val stationList = listOf("장소를 선택하세요") + stops
                        val stationAdapter = ArrayAdapter(context, R.layout.spinner_item_black, stationList)
                        stationAdapter.setDropDownViewResource(R.layout.spinner_dropdown)
                        stationSpinner.adapter = stationAdapter
                    }
                }

                override fun onCancelled(error: DatabaseError) {}
            })
    }
    //예약 생성 (중복 검사 포함)
    fun makeReservation(
        context: Context,
        routeName: String,
        selectedTime: String,
        selectedStation: String,
        onSuccess: (String) -> Unit,
        onFailure: (String) -> Unit
    ) {
        // 선택값 검증
        if (selectedTime == "시간을 선택하세요") {
            onFailure("시간을 선택해주세요.")
            return
        }

        if (selectedStation == "장소를 선택하세요") {
            onFailure("정류장을 선택해주세요.")
            return
        }

        val uid = FirebaseAuth.getInstance().currentUser?.uid ?: return
        val ref = FirebaseDatabase.getInstance().reference
            .child("users")
            .child(uid)
            .child("reservations")

        //예약 생성
        val currentDate = SimpleDateFormat("YY-MM-dd", Locale.getDefault()).format(Date())

        ref.get().addOnSuccessListener { snapshot ->
            var alreadySchool = false
            var alreadyReturn = false

            for (child in snapshot.children) {
                val res = child.getValue(ReservationData::class.java)
                if (res != null && res.date == currentDate) {
                    if (res.route in listOf("교내순환", "사월역->교내순환", "안심역->교내순환", "하양역->교내순환")) {
                        alreadySchool = true
                    }
                    if (res.route == "A2->안심역->사월역") {
                        alreadyReturn = true
                    }
                }
            }

            val isSchool = routeName in listOf("교내순환", "사월역->교내순환", "안심역->교내순환", "하양역->교내순환")
            val isReturn = routeName == "A2->안심역->사월역"

            //하루 1건 제한
            if (isSchool && alreadySchool) {
                onFailure("등교버스는 하루에 하나만 예약할 수 있어요.")
                return@addOnSuccessListener
            }
            if (isReturn && alreadyReturn) {
                onFailure("하교버스는 하루에 하나만 예약할 수 있어요.")
                return@addOnSuccessListener
            }

            val data = ReservationData(
                route = routeName,
                time = selectedTime,
                station = selectedStation,
                date = currentDate
            )

            ref.push().setValue(data).addOnSuccessListener {
                onSuccess(selectedTime)
            }.addOnFailureListener {
                onFailure("예약 실패: ${it.message}")
            }
        }
    }

    //알람용 millis 계산
    fun getReservationTimeInMillis(time: String): Long {
        val now = Calendar.getInstance()
        val parts = time.split(":")
        if (parts.size == 2) {
            now.set(Calendar.HOUR_OF_DAY, parts[0].toInt())
            now.set(Calendar.MINUTE, parts[1].toInt())
            now.set(Calendar.SECOND, 0)
        }
        return now.timeInMillis
    }

    private fun getImageResIdFromRouteName(routeName: String): Int {
        return when (routeName) {
            "교내순환" -> R.drawable.map_gyonea
            "하양역->교내순환" -> R.drawable.hayang_station
            "안심역->교내순환" -> R.drawable.map_gyonea
            "사월역->교내순환" -> R.drawable.map_gyonea
            "A2->안심역->사월역" -> R.drawable.ansim_sawel
            else -> R.drawable.dcu_profile
        }
    }
}

```
<details>


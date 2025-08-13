# ClockAdapter.kt

## 1) 역할
- 정류장 목록을 RecyclerView로 표시
- 각 정류장의 예약자 수에 따라 **배지(배경색)**를 다르게 표시
- 기사님 가독성을 위해 **정류장명/인원수 폰트 크기·굵기 강화**  :contentReference[oaicite:0]{index=0}

---

## 2) 핵심 로직 요약
- **ViewHolder 구성**: `textStationName`, `textPassengerCount` 바인딩
- **가독성 강화**: 정류장명 20sp/볼드, 인원수 18sp/볼드로 설정
- **색상 규칙**: 예약 0명=회색, 1~3명=초록, 4명 이상=빨강으로 배경색 지정
- **데이터 바인딩**: `stationList[position]`을 화면에 매핑하고 `getItemCount()`로 총 개수 반환  :contentReference[oaicite:1]{index=1}

  
<details>
<summary> 코드 보기 </summary>

```kotlin
class ClockAdapter(private val stationList: List<StationInfo>) :
    RecyclerView.Adapter<ClockAdapter.StationViewHolder>() {

    class StationViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val stationName: TextView = view.findViewById(R.id.textStationName)
        val passengerCount: TextView = view.findViewById(R.id.textPassengerCount)
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): StationViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_station, parent, false)
        return StationViewHolder(view)
    }

    //ViewHolder 구성
    override fun onBindViewHolder(holder: StationViewHolder, position: Int) {
        //데이터 바인딩
        val station = stationList[position]

        holder.stationName.text = station.name
        //가독성 강화
        holder.stationName.setTextSize(TypedValue.COMPLEX_UNIT_SP, 20f) 
        holder.stationName.setTypeface(null, Typeface.BOLD) 

        holder.passengerCount.text = "${station.reservationCount}명"
        holder.passengerCount.setTextSize(TypedValue.COMPLEX_UNIT_SP, 18f)
        holder.passengerCount.setTypeface(null, Typeface.BOLD)

        // 색상 규칙
        val color = when {
            station.reservationCount == 0 -> Color.GRAY
            station.reservationCount <= 3 -> Color.parseColor("#4CAF50") // 초록
            else -> Color.parseColor("#F44336") // 빨강
        }
        holder.passengerCount.setBackgroundColor(color)
    }

    override fun getItemCount(): Int = stationList.size
}

```
</details>


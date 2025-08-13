# DriveAdapter.kt

## 1) 역할
- 운행 기록(노선/시간/종료시각/날짜) 리스트를 **RecyclerView**로 표시
- **삭제 버튼**으로 운행 기록을 **Firebase Realtime Database**에서 삭제
- 리스트가 비면 `onListEmpty()` 콜백으로 빈 화면 처리
- 항목 클릭 시 상위에서 전달한 `onItemClick(record)` 실행

---

## 2) 핵심 로직 요약
- **ViewHolder 구성**: `textRoute`, `textTime`, `textPlace(=endTime)`, `textDate`, `btnCancel` 바인딩
- **아이템 바인딩**: 레코드 값 화면에 표시, 클릭 시 `onItemClick(reservation)` 호출
- **삭제 흐름**:
  - `AlertDialog`로 확인 문구 표시
  - 확인 시 `drivers/{uid}/drived/{pushKey}` 경로에서 `removeValue()`
  - 어댑터 리스트에서 제거 → `notifyItemRemoved/notifyItemRangeChanged`
  - 비었으면 `onListEmpty()` 호출
  - `Toast`로 결과 안내 
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class DriveAdapter(private val reservationList: ArrayList<DrivedRecord>,
                   private val onListEmpty: () -> Unit,
                   private val onItemClick: (DrivedRecord) -> Unit)
    : RecyclerView.Adapter<DriveAdapter.ReservationViewHolder>() {

    //item_list.xml의 보여줄 데이터를 하나씩 인플레이트
    override fun onCreateViewHolder(
        parent: ViewGroup,
        viewType: Int
    ): ReservationViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_list, parent, false)
        return ReservationViewHolder(view)
    }

    //데이터 바인딩 + 삭제 버튼 클릭 이벤트
    override fun onBindViewHolder(holder: ReservationViewHolder, position: Int) {
        val reservation = reservationList[position]

        //ViewHolder 구성
        holder.textRoute.text = reservation.route
        holder.textTime.text = reservation.time
        holder.textStation.text = reservation.endTime
        holder.textDate.text = reservation.date

        //삭제 흐름
        holder.btnDelete.setOnClickListener {
            //확인 문구 표시
            AlertDialog.Builder(holder.itemView.context)
                .setTitle("운행 기록 삭제")
                .setMessage("\uD83D\uDEA8 운행 노선을 잘못 선택한 경우에만 삭제할 수 있습니다.\n삭제된 노선에 대한 불이익은 책임지지 않습니다.\n정말 삭제하시겠습니까?")
                //확인 시
                .setPositiveButton("확인") { dialog, _ ->
                    val currentUser =
                        com.google.firebase.auth.FirebaseAuth.getInstance().currentUser
                    currentUser?.let { user ->
                        val ref = FirebaseDatabase.getInstance().reference
                            .child("drivers")
                            .child(user.uid)
                            .child("drived")
                            .child(reservation.pushKey)

                        ref.removeValue()
                    }
                    reservationList.removeAt(position)
                    notifyItemRemoved(position)
                    //어댑터리스트에서 제거 후 호출
                    notifyItemRangeChanged(position, reservationList.size)

                    //비었으면
                    if (reservationList.isEmpty()) {
                        onListEmpty()
                    }
                    //결과 안내
                    Toast.makeText(holder.itemView.context, "운행 기록이 삭제되었습니다.", Toast.LENGTH_SHORT).show()
                }
                .setNegativeButton("취소", null) // 취소 누르면 아무 일도 안 함
                .show()
        }
        //아이템 바인딩
        holder.itemView.setOnClickListener {
            onItemClick(reservation)
        }

    }

    override fun getItemCount(): Int = reservationList.size

    //item_list.xml에 보여줄 데이터를 묶기
    class ReservationViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        val textRoute: TextView = itemView.findViewById(R.id.textRoute)
        val textTime: TextView = itemView.findViewById(R.id.textTime)
        val textStation: TextView = itemView.findViewById(R.id.textPlace)
        val textDate: TextView = itemView.findViewById(R.id.textDate)
        val btnDelete: Button = itemView.findViewById(R.id.btnCancel)
    }


}
```
</details>


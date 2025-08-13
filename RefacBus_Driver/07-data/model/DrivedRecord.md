```kotlin
package com.example.refac_driverapp.data.model

data class DrivedRecord(
    //데이터 저장을 위한 데이터클래스 (기사님이 운전한 정보 하나의 객체로 관리)
    val route: String = "",
    val time: String = "",
    val endTime: String = "",
    val date: String = "",
    var pushKey: String = ""  //-> FireBase에 저장된 고유키 (삭제할 때 필요)
)
```
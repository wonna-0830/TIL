# RefacBus_Student – 학생용 스쿨버스 예약 앱 TIL

## 개요
- **설명**: Kotlin + Firebase 기반의 학생용 스쿨버스 예약 앱의 화면별 핵심 로직 정리
- **목적**: Activity, ViewModel, Adapter, Data, UI 컴포넌트별 동작과 역할을 한눈에 파악하고, 리팩토링·유지보수 시 빠르게 참고

---

## 디렉토리 구조

| 번호 | 폴더명                | 내용 |
|------|----------------------|------|
| 01-login       | 로그인 화면 & ViewModel |
| 02-register    | 회원가입 화면 & ViewModel |
| 03-routechoose | 노선 선택 화면 & ViewModel |
| 04-timeplace   | 시간/정류장 선택 화면 & ViewModel |
| 05-selectbuslist | 예약 목록 화면 & ViewModel |
| 06-alarm       | 알람 브로드캐스트 리시버 |
| 07-data        | 데이터 모델, Repository |
| 08-design      | UI 디자인 컴포넌트 |
| 09-adapter     | RecyclerView Adapter |

---

## 문서 링크

### 01-login
- [LoginActivity.md](./01-login/LoginActivity.md)
- [LoginViewModel.md](./01-login/LoginViewModel.md)

### 02-register
- [RegisterActivity.md](./02-register/RegisterActivity.md)
- [RegisterViewModel.md](./02-register/RegisterViewModel.md)

### 03-routechoose
- [RouteChooseActivity.md](./03-routechoose/RouteChooseActivity.md)
- [RouteChooseViewModel.md](./03-routechoose/RouteChooseViewModel.md)

### 04-timeplace
- [TimePlaceActivity.md](./04-timeplace/TimePlaceActivity.md)
- [TimePlaceViewModel.md](./04-timeplace/TimePlaceViewModel.md)

### 05-selectbuslist
- [SelectBusListActivity.md](./05-selectbuslist/SelectBusListActivity.md)
- [SelectBusListViewModel.md](./05-selectbuslist/SelectBusListViewModel.md)

### 06-alarm
- [AlarmReceive.md](./06-alarm/AlarmReceive.md)

### 07-data
- [ReservationData.md](./07-data/ReservationData.md)
- [RouteInfo.md](./07-data/RouteInfo.md)
- [UserRepository.md](./07-data/UserRepository.md)

### 08-design
- [RecyclerViewDecoration.md](./08-design/RecyclerViewDecoration.md)

### 09-adapter
- [ReservationAdapter.md](./09-adapter/ReservationAdapter.md)

---

## 작성 규칙
- 각 `.md` 문서는 **1) 역할 → 2) 핵심 로직 요약 → 3) 코드 보기** 순서
- Activity와 ViewModel은 화면 기능 단위로 묶어 작성
- 코드 블록은 `<details>` 태그로 접어서 가독성 유지
- Firebase 경로, Intent extra key, 주요 상수는 주석으로 명시

---

## 참고자료
- 각각의 스크린샷 확인은 [README](https://github.com/wonna-0830/RefacSchoolBusReservationForUser-kotlin)에 있습니다.
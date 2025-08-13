# RefacBus_Driver – 기사용 스쿨버스 예약 앱 TIL

## 개요
- **설명**: Kotlin + Firebase 기반의 기사용 스쿨버스 예약 앱의 화면별 핵심 로직 정리
- **목적**: Activity, ViewModel, Adapter, Data, UI 컴포넌트별 동작과 역할을 한눈에 파악하고, 리팩토링·유지보수 시 빠르게 참고

---

## 디렉토리 구조

| 번호 | 폴더명                | 내용 |
|------|----------------------|------|
| 01-login       | 로그인 화면 & ViewModel |
| 02-register    | 회원가입 화면 & ViewModel |
| 03-routetime   | 노선/시간 선택 화면 & ViewModel |
| 04-clock       | 운행 중 보여질 시간/학생 수 화면 & ViewModel |
| 05-finish      | 운행 종료 후 화면 & ViewModel |
| 06-selectbuslist| 운행 기록 화면 & ViewModel |
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
- [RouteTimeActivity.md](./03-routetime/RouteTimeActivity.md)
- [RouteTimeViewModel.md](./03-routetime/RouteTimeViewModel.md)

### 04-timeplace
- [clock.md](./04-clock/clock.md)
- [ClockViewModel.md](./04-clock/ClockViewModel.md)

### 05-selectbuslist
- [FinishActivity.md](./05-finish/FinishActivity.md)
- [FinishViewModel.md](./05-finish/FinishViewModel.md)

### 06-alarm
- [SelectBusListActivity.md](./06-selectbuslist/SelectBusListActivity.md)
- [SelectBusListViewModel.md](./06-selectbuslist/SelectBusListViewModel.md)

### 07-data
- [DrivedRecord.md](./07-data/model/DrivedRecord.md)
- [StationInfo.md](./07-data/model/StationInfo.md)

- [ClockRepository.md](./07-data/repository/ClockRepository.md)
- [FinishRepository.md](./07-data/repository/FinishRepository.md) 
- [RouteTimeRepository.md](./07-data/repository/RouteTimeRepository.md)
- [SelectBusList.md](./07-data/repository/SelectBusListRepository.md)
- [UserRepository.md](./07-data/repository/UserRepository.md)
  
  
### 08-design
- [RecyclerViewDecoration.md](./08-design/RecyclerViewDecoration.md)

### 09-adapter
- [ClockAdapter.md](./09-adapter/ClockAdapter.md)
- [DriveAdapter.md](./09-adapter/DriveAdapter.md)

---

## 작성 규칙
- 각 `.md` 문서는 **1) 역할 → 2) 핵심 로직 요약 → 3) 코드 보기** 순서
- Activity와 ViewModel은 화면 기능 단위로 묶어 작성
- 코드 블록은 `<details>` 태그로 접어서 가독성 유지
- Firebase 경로, Intent extra key, 주요 상수는 주석으로 명시

---

## 참고자료
- 각각의 스크린샷 확인은 [README](https://github.com/wonna-0830/RefacSchoolBusReservationForDriver-kotlin)에 있습니다.
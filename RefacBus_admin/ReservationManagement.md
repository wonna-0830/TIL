## ReservationManagement.jsx 톺아보기
### import문 및 중복 로직들
```import React, { useState, useEffect } from 'react';
import { Typography, Box, Button, } from '@mui/material';
import { getDatabase, ref, get, child } from 'firebase/database';
import TabbedContainer from '../components/common/TabbedContainer';
import TabPanel from '../components/common/TabPanel';
import DateSelector from '../components/Reservation/DateSelector';
import ReservationListTable from '../components/Reservation/ReservationListTable';
import StatFilterBar from '../components/Reservation/StatFilterBar';
import DeleteStatFilterBar from '../components/Reservation/DeleteStatFilterBar';
import StatBarChart from '../components/Reservation/StatBarChart';
import SearchBar from '../components/common/SearchBar';
import { useAdmin } from '../context/AdminContext';
import { hasPermission } from '../utils/permissionUtil';


const ReservationManagement = () => {
  const admin = useAdmin();
    if (!admin) return null;
  const [tabIndex, setTabIndex] = useState(0);
  const handleTabChange = (event, newValue) => setTabIndex(newValue);
```

### 날짜별 예약자 목록

#### UI
```<TabPanel value={tabIndex} index={0}>
  {hasPermission(admin, '예약자 목록') ? ( 
    <Box>
      <Box sx={{ display: 'flex', alignItems: 'center', gap: 2,}}>
        <DateSelector
          year={year}
          month={month}
          day={day}
          onYearChange={handleYearChange}
          onMonthChange={handleMonthChange}
          onDayChange={handleDayChange}
          allowEmpty={false} 
        />

       <SearchBar
          value={searchKeyword}
          onChange={(e) => setSearchKeyword(e.target.value)}
          placeholder="이름으로 검색"
        />
        
        <Button variant="contained" onClick={handleSearch}>조회</Button>
      </Box>
      <ReservationListTable data={filteredData} />
    </Box> 
      ) : (
    <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
  )}  
</TabPanel>
```
#### 로직
```const [year, setYear] = useState('');
const [month, setMonth] = useState('');
const [day, setDay] = useState('');
const [searchKeyword, setSearchKeyword] = useState('');
const [reservationList, setReservationList] = useState([]);
const [filteredData, setFilteredData] = useState([]);

const handleYearChange = (e) => setYear(e.target.value);
const handleMonthChange = (e) => setMonth(e.target.value);
const handleDayChange = (e) => setDay(e.target.value);

const handleSearch = async () => {
  if (!year || !month || !day) {
    alert('년, 월, 일을 모두 선택해주세요!');
    return;
  }

  const targetDate = `${year.slice(2)}-${month}-${day}`;
  const db = getDatabase();
  const usersRef = ref(db, 'users');

  try {
    const snapshot = await get(usersRef);
    if (snapshot.exists()) {
      const usersData = snapshot.val();
      let resultList = [];
      Object.entries(usersData).forEach(([uid, userInfo]) => {
        const { name, email, reservations } = userInfo;
        if (reservations) {
          Object.values(reservations).forEach((res) => {
            if (res.date === targetDate) {
              resultList.push({
                name,
                email,
                route: res.route || '',
                time: res.time || '',
                canceled: false
              });
            }
          });
        }
      });
      if (searchKeyword.trim() !== '') {
        resultList = resultList.filter(item =>
          item.name.includes(searchKeyword.trim())
        );
      }
      setReservationList(resultList);
      setFilteredData(resultList);
    } else {
      setReservationList([]);
      setFilteredData([]);
      alert('예약 데이터가 없습니다.');
    }
  } catch (error) {
    alert('예약 데이터를 불러오는 중 오류가 발생했습니다.');
  }
};
```
- `<DateSelector>` -> 날짜 선택 드롭다운
  - `handleYearChange` `handleMonthChange` `handleDayChange` -> 드롭다운 내 년, 월, 일을 선택한 일자로 변환
- 조회 버튼 클릭 시 `handleSearch` 실행
  - `if (!year || !month || !day) {` -> year, month, day 중 값이 하나라도 없으면 알림창이 뜨면서 return;
  - `const targetDate = ${year.slice(2)}-${month}-${day}` -> 데이터베이스 내 날짜와 동일하게 맞추기 위해 slice 사용 및 날짜 세팅
  - `const db = getDatabase();` `const usersRef = ref(db, 'users');` -> user 데이터베이스 가져와 usersRef에 저장
  - `try {`
    - `const snapshot = await get(usersRef);` -> snapshot에 usersRef를 가져올 때까지 기다림
    - `if (snapshot.exists()) {` -> snapshot에 데이터가 존재하면
      - `const usersData = snapshot.val();` -> snapshot을 js에서 사용가능하도록 변환
      - `let resultList = [];`-> resultList 선언
      - `Object.entries(usersData).forEach(([uid, userInfo]) => {` -> usersData를 배열로 바꾼 후 순회
      - `const { name, email, reservations } = userInfo;` -> userInfo에서 name, email, reservations를 추출
      - `if (reservations) {` -> reservations 내 데이터가 존재 할 때 
        - `Object.values(reservations).forEach((res) => {` -> reservations의 내역만 배열로 꺼내 순회
        - `if (res.date === targetDate) {` -> 예약 날짜가 검색할 날짜(targetDate)와 일치할 때
          - `resultList.push({` -> resultList에 이름, 이메일, 노선, 시간 정보를 꺼내 저장
      - `if (searchKeyword.trim() !== '') {` -> searchKeyword(이름) 정보가 들어있으면
        - `resultList = resultList.filter(item => item.name.includes(searchKeyword.trim()));`
        - -> searchKeyword가 포함된 이름의 item들을 필터링해 저장
      - `setReservationList(resultList);` -> 원본 예약 리스트에 백업
      - `setFilteredData(resultList);` -> 필터링된 리스트로 목록에 노출
  


### 예약 통계

#### UI
```<TabPanel value={tabIndex} index={1}>
  {hasPermission(admin, '예약 통계') ? (
    <Box>
      <StatFilterBar
        statType={statType}
        onTypeChange={handleStatTypeChange}
        chartYear={chartYear}
        chartMonth={chartMonth}
        chartDay={chartDay}
        onYearChange={handleChartYearChange}
        onMonthChange={handleChartMonthChange}
        onDayChange={handleChartDayChange}
        selectedRoute={selectedRoute}
        onRouteChange={setSelectedRoute}
        routeList={routeList}
        onSearchClick={handleChartSearch}
      />
      <StatBarChart data={filteredRouteStats} />
    </Box>
    ) : (
    <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
  )}
</TabPanel>
```
#### 로직
```const [statType, setStatType] = useState('routeTotal');
const [filterValue, setFilterValue] = useState('');
const [chartYear, setChartYear] = useState('');
const [chartMonth, setChartMonth] = useState('');
const [chartDay, setChartDay] = useState('');
const [selectedRoute, setSelectedRoute] = useState('');
const [routeList, setRouteList] = useState([]);
const [filteredRouteStats, setFilteredRouteStats] = useState([]);

const handleStatTypeChange = (e) => {
  const selectedType = e.target.value;
  setStatType(selectedType);
  setFilterValue('');
};

const handleChartYearChange = (e) => setChartYear(e.target.value);
const handleChartMonthChange = (e) => setChartMonth(e.target.value);
const handleChartDayChange = (e) => setChartDay(e.target.value);

const handleChartSearch = async () => {
  const db = getDatabase();
  const usersRef = ref(db, 'users');
  const snapshot = await get(usersRef);
  if (!snapshot.exists()) return;

  const usersData = snapshot.val();
  const counts = {};
  let dynamicFilter = '';

  if (statType === 'route') {
    const shortYear = chartYear ? chartYear.slice(2) : '';
    dynamicFilter = [shortYear, chartMonth, chartDay].filter(Boolean).join('-');
  } else if (statType === 'station' || statType === 'time') {
    dynamicFilter = selectedRoute;
  }

  Object.values(usersData).forEach(user => {
    const reservations = user.reservations || {};
    Object.values(reservations).forEach(res => {
      if (statType === 'route' && res.date === dynamicFilter) {
        const route = res.route || '기타';
        counts[route] = (counts[route] || 0) + 1;
      }
       if (statType === 'station' && res.route === dynamicFilter) {
        const station = res.station || '미지정';
        counts[station] = (counts[station] || 0) + 1;
       }
      if (statType === 'time' && res.route === dynamicFilter) {
        const time = res.time || '00:00';
        counts[time] = (counts[time] || 0) + 1;
      }
      if (statType === 'routeTotal') {
        const route = res.route || '기타';
        counts[route] = (counts[route] || 0) + 1;
      }
   });
  });

  const data = Object.entries(counts).map(([name, count]) => ({ name, count }));
  setFilteredRouteStats(data);
};
```
- `handleStatTypeChange` -> 여러가지 통계 중 하나를 선택하면 실행
  - `const selectedType = e.target.value;` -> 선택한 통계명으로 값 저장
  - `setStatType(selectedType);` -> statType을 선택한 통계명으로 변경
  - `setFilterValue('');` -> FilterValue를 초기화
- `handleChartYearChange` `handleChartMonthChange` `handleChartDayChange` -> 날짜별 예약 통계에서 날짜 드롭다운 선택 시 해당 년, 월, 일로 값 저장
- 조회 버튼 클릭 시 `handleChartSearch` 실행
  - realtimeDb의 users를 가져와 저장하고 Js에서 사용 가능하도록 변환, 배열로 저장
  - `if (statType === 'route') {` -> `<MenuItem value="route">날짜별 노선별 예약 수</MenuItem>` 를 선택하면 데이터베이스 내 날짜와 동일한 규칙으로 날짜 변환 후 날짜에 대에서 DynamicFilter에 저장
  - `else if (statType === 'station' || statType === 'time') {` -> `<MenuItem value="station">노선별 정류장 예약 수</MenuItem>` `<MenuItem value="time">노선별 시간대 예약 수</MenuItem>` 선택하면 선택한 노선(selectedRoute)에 대해 DymanicFilter에 저장
  - `Object.values(usersData).forEach(user => {` -> userData에 대해 배열로 만들고 순회 시작
    - `const reservations = user.reservations || {};` -> user.reservations값이 있으면 그대로 사용, 아니면 빈 객체를 대신 사용
    - `Object.values(reservations).forEach(res => {` -> reservations 내 데이터를 배열로 만들고 순회 시작
      - `if (statType === 'route' && res.date === dynamicFilter) {` -> statType이 route고 res.date가 dynamicFilter와 동일하면 
        - `const route = res.route || '기타'; ` -> res.route 값이 있으면 그대로 사용, 아니면 '기타'로 사용
        - `counts[route] = (counts[route] || 0) + 1;` -> 예약 건수를 하나씩 누적, 아니면 0으로
        - 나머지 통계도 동일하게


```useEffect(() => {
  if (statType === 'routeTotal') {
    handleChartSearch();
  }
}, [statType]);
```
- 처음 통계화면을 보여줄 때는 전체 예약 건수에 대한 통계를 보여주기 위함
- `const [statType, setStatType] = useState('routeTotal');` -> 이미 statType은 기본적으로 routeTotal이므로 handelChartSearch를 통해 전체 예약 통계를 보여줌

---

### 예약 취소 통계

#### UI
```<TabPanel value={tabIndex} index={2}>
  <DeleteStatFilterBar
    deleteType={deleteType}
    onDeleteTypeChange={handleDeleteChange}
    deleteYear={deleteYear}
    deleteMonth={deleteMonth}
    deleteDay={deleteDay}
    onYearChange={handleDeleteYearChange}
    onMonthChange={handleDeleteMonthChange}
    onDayChange={handleDeleteDayChange}
    selectedRoute={selectedTime}
    onRouteChange={setSelectedTime}
    routeList={routeList}
    onSearchClick={handleDeleteSearch}
  />
  <StatBarChart data={filteredRouteDeletes} color="#f06292" />
</TabPanel>
```
#### 로직
```const [allDeletedReservations, setAllDeletedReservations] = useState([]);
const [deleteType, setDeleteType] = useState("routeTotal");
const [filteredRouteDeletes, setFilteredRouteDeletes] = useState([]);
const [deleteYear, setDeleteYear] = useState('');
const [deleteMonth, setDeleteMonth] = useState('');
const [deleteDay, setDeleteDay] = useState('');
const [selectedTime, setSelectedTime] = useState('');

const handleDeleteChange = (e) => setDeleteType(e.target.value);
const handleDeleteYearChange = (e) => setDeleteYear(e.target.value);
const handleDeleteMonthChange = (e) => setDeleteMonth(e.target.value);
const handleDeleteDayChange = (e) => setDeleteDay(e.target.value);

useEffect(() => {
  const db = getDatabase();
  const usersRef = ref(db, "users");

  get(usersRef).then((snapshot) => {
    const all = [];
    snapshot.forEach((userSnap) => {
      const reservations = userSnap.child("reservations");
      reservations.forEach((resSnap) => {
        const data = resSnap.val();
        if (data.deleted === true) {
          all.push({
            route: data.route,
            time: data.time,
            date: data.date,
            reason: data.reason,
          });
        }
      });
    });
    setAllDeletedReservations(all);
  });
}, []);
```
- 날짜 드롭다운에 대해 선택한 날짜로 변경
- useEffect -> reservations 내 데이터를 순회하며 data.deleted가 true인 배열만 setAllDeletedReservations에 저장

```const handleDeleteSearch = () => {
  const formatDate = (dateStr) => {
  const [yy, mm, dd] = dateStr.split("-");
  return `20${yy}-${mm}-${dd}`;
  };

  const filtered = allDeletedReservations.filter((r) => {
    if (!r.date) return false;
    const fullDate = formatDate(r.date);
    const [y, m, d] = fullDate.split("-");
    return (
      (!deleteYear || y === deleteYear) &&
      (!deleteMonth || m === deleteMonth) &&
      (!deleteDay || d === deleteDay)
    );
  });

  let result = [];

  if (deleteType === "route") {
    const grouped = {};
    filtered.forEach((r) => {
      grouped[r.route] = (grouped[r.route] || 0) + 1;
    });
    result = Object.entries(grouped).map(([name, count]) => ({ name, count }));
  }

  if (deleteType === "time") {
    const grouped = {};
    allDeletedReservations
      .filter((r) => selectedTime === "" || r.route === selectedTime)
      .forEach((r) => {
        const key = r.time;
        grouped[key] = (grouped[key] || 0) + 1;
      });
    result = Object.entries(grouped).map(([name, count]) => ({ name, count }));
  }

  if (deleteType === "routeTotal") {
    const grouped = {};
    allDeletedReservations.forEach((r) => {
      grouped[r.route] = (grouped[r.route] || 0) + 1;
    });
    result = Object.entries(grouped).map(([name, count]) => ({ name, count }));
  }

  if (deleteType === "reason") {
    const grouped = {};
    allDeletedReservations.forEach((r) => {
      const reasons = r.reason?.split(",") ?? [];
      reasons.forEach((re) => {
        const trimmed = re.trim();
        grouped[trimmed] = (grouped[trimmed] || 0) + 1;
      });
    });
    result = Object.entries(grouped).map(([name, count]) => ({ name, count }));
  }

  setFilteredRouteDeletes(result);
};
```
- 조회 버튼 클릭 시 `handleDeleteSearch` 실행
- `const formatDate = (dateStr) => {` 
  - `const [yy, mm, dd] = dateStr.split("-");` -> 날짜를 `-` 기준으로 나눠서 yy, mm, dd로 분리
  - `return 20${yy}-${mm}-${dd};` -> 분리된 yy, mm, dd를 20yy-mm-dd 형식으로 리턴
- `const filtered = allDeletedReservations.filter((r) => {`
  - `if (!r.date) return false;` -> allDeletedReservations 내 date 값이 없으면 false 값을 리턴
  - `const fullDate = formatDate(r.date);` -> 20yy-mm-dd로 변환한 r.date를 fullDate에 저장
  - `const [y, m, d] = fullDate.split("-");` -> fullDate를 `-` 기준으로 나눠서 분리
  - `(!deleteYear || y === deleteYear) && (!deleteMonth || m === deleteMonth) && (!deleteDay || d === deleteDay)` -> 연도 선택X 또는 선택한 연도와 일치, 월 선택X 또는 선택한 월과 일치, 일 선택X 또는 선택한 일과 일치하는 경우만 true => 선택된 항목이 없으면 무조건 통과, 선택됐다면 값이 일치하는 경우만 통과
  - `if (deleteType === "time") {` -> 선택한 드롭다운 항목이 value가 time이면 
    - `const grouped = {};` -> grouped 선언
    - `allDeletedReservations.filter((r) => selectedTime === "" || r.route === selectedTime).forEach((r) => {` -> allDeletedReservations의 selectedTime이 빈칸이거나, r.route와 selectedTime이 동일하면
      - `const key = r.time;` -> allDeletedReservations의 time 값은 key에 저장
      - `grouped[key] = (grouped[key] || 0) + 1;` -> key 건수를 하나씩 누적, 아니면 0으로
    - `result = Object.entries(grouped).map(([name, count]) => ({ name, count }));` -> grouped에 대해서 name, count로 맵핑
  
  return (
    <Box sx={{ width: '100%' }}>
      
      <TabbedContainer
        tabIndex={tabIndex}
        handleTabChange={handleTabChange}
        labels={["날짜별 예약자 목록", "예약 통계", "예약 취소 분석"]}
      />

      {/* 회색 박스 본문 */}
      <Box
        sx={{
          backgroundColor: '#f5f5f5',
          borderRadius: 2,
          padding: 3,
          marginTop: 2,
          width: '100%',
          maxWidth: 'none',
        }}
      >
        <TabPanel value={tabIndex} index={0}>
          {hasPermission(admin, '예약자 목록') ? ( 
          <Box>
            <Box sx={{ display: 'flex', alignItems: 'center', gap: 2,}}>
              <DateSelector
                year={year}
                month={month}
                day={day}
                onYearChange={handleYearChange}
                onMonthChange={handleMonthChange}
                onDayChange={handleDayChange}
                allowEmpty={false} 
              />

             <SearchBar
                value={searchKeyword}
                onChange={(e) => setSearchKeyword(e.target.value)}
                placeholder="이름으로 검색"
              />
              
              <Button variant="contained" onClick={handleSearch}>조회</Button>
            </Box>
            <ReservationListTable data={filteredData} />
          </Box> 
           ) : (
          <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
        )}  
        </TabPanel>
        <TabPanel value={tabIndex} index={1}>
          {hasPermission(admin, '예약 통계') ? (
          <Box>
            <StatFilterBar
              statType={statType}
              onTypeChange={handleStatTypeChange}
              chartYear={chartYear}
              chartMonth={chartMonth}
              chartDay={chartDay}
              onYearChange={handleChartYearChange}
              onMonthChange={handleChartMonthChange}
              onDayChange={handleChartDayChange}
              selectedRoute={selectedRoute}
              onRouteChange={setSelectedRoute}
              routeList={routeList}
              onSearchClick={handleChartSearch}
            />
            <StatBarChart data={filteredRouteStats} />
          </Box>
          ) : (
          <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
        )}
        </TabPanel>
        <TabPanel value={tabIndex} index={2}>
          <DeleteStatFilterBar
            deleteType={deleteType}
            onDeleteTypeChange={handleDeleteChange}
            deleteYear={deleteYear}
            deleteMonth={deleteMonth}
            deleteDay={deleteDay}
            onYearChange={handleDeleteYearChange}
            onMonthChange={handleDeleteMonthChange}
            onDayChange={handleDeleteDayChange}
            selectedRoute={selectedTime}
            onRouteChange={setSelectedTime}
            routeList={routeList}
            onSearchClick={handleDeleteSearch}
          />
          <StatBarChart data={filteredRouteDeletes} color="#f06292" />
        </TabPanel>
      </Box>
    </Box>
  );
};

export default ReservationManagement;

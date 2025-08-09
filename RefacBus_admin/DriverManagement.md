## DriverManagement.jsx 톺아보기
### import문 및 중복 로직 (userTable 컴포넌트 사용 로직)
```import React, { useState, useEffect } from 'react';
import { Box, Typography} from '@mui/material';
import { ref, get, update, push, set } from "firebase/database";
import { realtimeDb, auth } from "../firebase";
import { sendPasswordResetEmail } from "firebase/auth";
import dayjs from "dayjs"; 
import { UserTable, MemoDialog, MemoHistoryDialog } from '../components/User';
import { TabbedContainer, TabPanel } from '../components/common';
import { DriverListTable, DriverScheduleDialog, DrivingHistoryDialog, ResetPasswordDialog } from '../components/Driver';
import { timeSlots, days, getDayIndex } from '../components/Driver/constants';
import SearchBar from '../components/common/SearchBar';
import { useAdmin } from '../context/AdminContext';
import { hasPermission } from '../utils/permissionUtil';

const DriverManagement = () => {
  const admin = useAdmin();
  if (!admin) return null;
  // 기본 UI 상태 및 탭 변환
  const [tabIndex, setTabIndex] = useState(0);
  const handleTabChange = (event, newValue) => {
    setTabIndex(newValue);
  };

  //사용자 목록 상태 및 검색
  const [searchKeyword, setSearchKeyword] = useState('');
  const [filteredUsers, setFilteredUsers] = useState([]);
  const [allUsers, setAllUsers] = useState([]);
  
  //MoreVertIcon 메뉴 클릭 시
  const [anchorEl, setAnchorEl] = useState(null);
  const [anchorUserId, setAnchorUserId] = useState(null);
  useEffect(() => {
    const fetchUsers = async () => {
      const snapshot = await get(ref(realtimeDb, "drivers"));
      const usersData = snapshot.val();
      const usersArray = Object.entries(usersData).map(([uid, value]) => ({
        uid,
        ...value,
      }));
      setAllUsers(usersArray);
      setFilteredUsers(usersArray);
    };
    fetchUsers();
  }, []);
  
  useEffect(() => {
    const filtered = allUsers.filter((user) =>
      (user.name ?? '').toLowerCase().includes(searchKeyword.toLowerCase())
    );
    setFilteredUsers(filtered);
  }, [searchKeyword, allUsers]);

  //운행 이력 관련
  const [selectedDriver, setSelectedDriver] = useState(null);
  const [drivingHistory, setDrivingHistory] = useState([]);
  const [isHistoryOpen, setIsHistoryOpen] = useState(false);

  //스케줄 등록 관련
  const [currentTargetUser, setCurrentTargetUser] = useState(null);
  const [pinnedRoutes, setPinnedRoutes] = useState([]);
  const [selectedRoute, setSelectedRoute] = useState('');
  const [selectedRouteId, setSelectedRouteId] = useState('');
  const [availableTimes, setAvailableTimes] = useState([]);
  const [selectedTime, setSelectedTime] = useState('');
  const [selectedDays, setSelectedDays] = useState([]);
  const [duration, setDuration] = useState('');
  const [selectedCellTime, setSelectedCellTime] = useState('');
  const [isAddDialogOpen, setIsAddDialogOpen] = useState(false);
  const [isEditing, setIsEditing] = useState(false);
  const [editingScheduleKey, setEditingScheduleKey] = useState(null);

  //스케줄 시각화 관련
  const [expandedDriverUid, setExpandedDriverUid] = useState(null);
  const [coloredSchedule, setColoredSchedule] = useState({});
  const [highlightedCells, setHighlightedCells] = useState(new Set()); 
  const [driverSchedules, setDriverSchedules] = useState({});
  const [driverList, setDriverList] = useState([]);

  //메모 관련 및 다이얼로그
  const [isMemoOpen, setIsMemoOpen] = useState(false);
  const [selectedUserForMemo, setSelectedUserForMemo] = useState(null);
  const [memoText, setMemoText] = useState("");
  const [isWarning, setIsWarning] = useState(false);
  const [isBan, setIsBan] = useState(false);
  const [isMemoHistoryOpen, setIsMemoHistoryOpen] = useState(false);
  const [selectedMemoList, setSelectedMemoList] = useState([]);
  const [memoFilter, setMemoFilter] = useState("all");

  //사용자 정보 수정/초기화 다이얼로그
  const [isEditDialogOpen, setIsEditDialogOpen] = useState(false);
  const [editingUser, setEditingUser] = useState(null);
  const [editedEmail, setEditedEmail] = useState("");
  const [editedName, setEditedName] = useState("");
  const [resetEmail, setResetEmail] = useState('');
  const [isResetOpen, setIsResetOpen] = useState(false);
  const [targetUser, setTargetUser] = useState(null);

useEffect(() => {  
  if (currentTargetUser?.uid) {
    fetchDriverSchedule(currentTargetUser.uid);
  }
}, [currentTargetUser]);

useEffect(() => {
  const fetchDriverList = async () => {
    const snapshot = await get(ref(realtimeDb, "drivers"));
    if (snapshot.exists()) {
      const driversData = snapshot.val();
      const driversArray = Object.entries(driversData).map(([uid, info])=> ({
        uid,
        ...info,
      }));
      setDriverList(driversArray);
    }
  };

  fetchDriverList();
}, []);

useEffect(() => {
  if (driverList.length > 0 && !currentTargetUser) {
    setCurrentTargetUser(driverList[0]);
  }
}, [driverList, currentTargetUser]);

useEffect(() => {
  if (driverList.length > 0) {
    driverList.forEach((driver) => {
      fetchDriverSchedule(driver.uid);
    });
  }
}, [driverList]);

const handleMenuOpen = (event, user) => {
  setAnchorEl(event.currentTarget);
  setAnchorUserId(user.uid);
};

const handleMenuClose = () => {
  setAnchorEl(null);
  setAnchorUserId(null);
};  

const handleOpenMemo = (user) => {
  setSelectedUserForMemo(user);
  setIsMemoOpen(true);
};  

const handleCloseMemo = () => {
  setSelectedUserForMemo(null);
  setIsMemoOpen(false);
  setMemoText("");
  setIsWarning(false);
  setIsBan(false);
};  

const handleSubmitMemo = async () => {
  if (!selectedUserForMemo) return;
  
  const userRef = ref(realtimeDb, `drivers/${selectedUserForMemo.uid}`);
  const memoRef = ref(realtimeDb, `drivers/${selectedUserForMemo.uid}/memo`);
  
  const newMemo = {
    text: memoText.trim(),
    timestamp: dayjs().format('YYYY-MM-DD'),
    writer: '관리자A',
    type: isBan ? 'ban' : isWarning ? 'warning' : 'note'
  };
  
  await push(memoRef, newMemo);
  
  const snapshot = await get(userRef);
  const existingData = snapshot.val();
  
  const updates = {};
  
  if (isWarning) {
    updates.warningCount = (existingData?.warningCount ?? 0) + 1;
  }
  if (isBan) {
    updates.isBanned = true;
  }

  if (Object.keys(updates).length > 0) {
    await update(userRef, updates);
  }

  setAllUsers((prev) =>
    prev.map((u) =>
      u.uid === selectedUserForMemo.uid ? { ...u, ...updates } : u
    )
  );
  setFilteredUsers((prev) =>
    prev.map((u) =>
      u.uid === selectedUserForMemo.uid ? { ...u, ...updates } : u
    )
  );

  handleCloseMemo();
};

const handleOpenMemoHistory = async (user) => {
  const memoRef = ref(realtimeDb, `drivers/${user.uid}/memo`);
  const snapshot = await get(memoRef);

  if (snapshot.exists()) {
    const memoObject = snapshot.val();
    const memoArray = Object.values(memoObject);

    const sorted = memoArray.sort(
      (a, b) => dayjs(b.timestamp).valueOf() - dayjs(a.timestamp).valueO()
    );
    setSelectedMemoList(sorted);
  } else {
    setSelectedMemoList([]);
  }

  setIsMemoHistoryOpen(true);
};

const handleOpenEditDialog = (user) => {
  setEditingUser(user);
  setEditedEmail(user.email ?? "");
  setEditedName(user.name ?? "");
  setIsEditDialogOpen(true);
};

const handleOpenReset = (user) => {
  setTargetUser(user);
  setResetEmail(user.email); 
  setIsResetOpen(true);
};

const handleSendResetEmail = async () => {
  try {
    await sendPasswordResetEmail(auth, resetEmail.trim());
    alert("비밀번호 초기화 메일이 전송되었습니다.");
    setIsResetOpen(false);
  } catch (error) {
    alert("이메일 전송 실패: " + error.message);
  }
};  

const handleUnban = async (uid) => {
  await update(ref(realtimeDb, `drivers/${uid}`), { isBanned: false });
  
  setAllUsers((prev) =>
    prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
  );
  setFilteredUsers((prev) =>
    prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
  );

  handleMenuClose();
};
```

### 기사별 운행 이력 탭 기능

#### UI
```<DriverListTable
  users={filteredUsers}
  expandedDriverUid={expandedDriverUid}
  driverSchedules={driverSchedules}
  coloredSchedule={coloredSchedule}
  onTimeClick={handleTimeOpen}
  onScheduleToggle={handleOpenSchedule}
  onAddClick={handleOpenAddDialog}
  onCellClick={(matchedSchedule) => {
    fetchPinnedRoutes(matchedSchedule.uid).then(() => {
      setSelectedDays(matchedSchedule.days);
      setSelectedTime(matchedSchedule.time);
      setDuration(matchedSchedule.duration);
      setSelectedRoute(matchedSchedule.route);
      setCurrentTargetUser(matchedSchedule.user);
      setIsEditing(true);
      setEditingScheduleKey(matchedSchedule.key);
      handleRouteSelect({ target: { value: matchedSchedule.route } });
      setIsAddDialogOpen(true);
    });
  }}
/>
```

#### 운행 이력 조회
```const handleTimeOpen = async (e, user) => {
  setSelectedDriver(user);
  setIsHistoryOpen(true);

  try {
    const snapshot = await get(ref(realtimeDb, `drivers/${user.uid}/drived`));
    if (snapshot.exists()) {
      const data = snapshot.val();
      const records = Object.values(data)
        .map((item) => ({
          route: item.route,
          date: item.date,
          time: item.time,
          endTime: item.endTime,
        }))
        .sort((a, b) => {
          const aDate = new Date(`${a.date}T${a.time}`);
          const bDate = new Date(`${b.date}T${b.time}`);
          return bDate - aDate;
        });
      setDrivingHistory(records);
    } else {
      setDrivingHistory([]);
    }
  } catch (error) {
    console.error("운행 이력 불러오기 실패:", error);
  }
};
```
- 운행이력의 아이콘 버튼을 누르면 `handleTimeOpen` 실행
  - `setSelectedDriver(user);` -> selectedDriver에 해당 기사의 정보를 저장
  - `setIsHistoryOpen(true);` -> isHistoryOpen이 true로
  - ```<DrivingHistoryDialog
      open={isHistoryOpen}
      onClose={() => setIsHistoryOpen(false)}
      driverName={selectedDriver?.name}
      historyList={drivingHistory}
    />
        ``` 다이얼로그 오픈
  - `const snapshot = await get(ref(realtimeDb, drivers/${user.uid}/drived));` -> realtimeDb의 drivers/${user.id}.drived의 정보를 저장
  - `if (snapshot.exists()) {` -> snapshot에 값이 존재하면
    - `const data = snapshot.val();` -> Js로 사용가능하도록 변환
    - `const records = Object.values(data).map((item) => ({` -> data 내 값을 배열로 만들고 그 중 (route, date, time, endTime)들을 새 객체로 변환
    - `.sort((a, b) => {`
      - `const aDate = new Date(${a.date}T${a.time}); ` -> date와 time을 합쳐서 Date 객체로 만듦
      - `return bDate - aDate;` -> a와 b를 비교해서 최신순으로 정렬
    - `setDrivingHistory(records);` -> setDrivingHistory를 records 값으로 저장, 아니면 빈 배열로
  - 아니면 오류 메세지 노출 

#### 운행 일정 관리 아코디언

```const handleOpenSchedule = (user) => {
  setExpandedDriverUid((prevUid) => (prevUid === user.uid ? null : user.uid));
};
```
- 운행 일정 관리 아이콘을 누르면 실행
- `ExpandedDriverUid`의 현재 Uid가 user.uid와 같으면(열려있으면 닫기) null로, 아니면 user.uid로(닫혀있으면 열기)  
  
#### 스케줄 추가 버튼 클릭 시 
```const handleOpenAddDialog = (user) => {
  setCurrentTargetUser(user);
  setIsAddDialogOpen(true);
  fetchPinnedRoutes(user.uid);
};

const fetchPinnedRoutes = async (uid) => {
  try {
    const snapshot = await get(ref(realtimeDb, `routes`));
    if (snapshot.exists()) {
      const data = snapshot.val();
      const pinned = Object.entries(data)
        .filter(([_, route]) => route.isPinned)
        .map(([key, route]) => ({
          id: key,
          name: route.name,
        }));
      setPinnedRoutes(pinned);
    } else {
      setPinnedRoutes([]);
    }
  } catch (err) {
    console.error("노선 목록 가져오기 실패:", err);
  }
};
```
- `CurrentTargetUser`을 현재 클릭한 유저의 데이터로 저장
- `IsAddDialogOpen`를 true로 -> 다이얼로그 오픈
- `fetchPinnedRoute`에 user.uid 값을 넣어 실행
  - routes의 정보를 가져와 Js에서 사용가능한 배열로 전환해 route 내 isPinned를 필터링해서 저장
  - 그 중 id, name을 가져와 새로운 객체로 생성
  - `setPinnedRoutes` 상태를 pinned로

```const handleCloseAddDialog = () => {
  setIsAddDialogOpen(false);
  setCurrentTargetUser(null);
  setIsEditing(false);
  setEditingScheduleKey(null);
};
```
- `isAddDialogOpen`을 false로 -> 다이얼로그 닫기
- `CurrentTargetUser` 초기화
- `isEditing` 초기화
- `EditingScheduleKey` 초기화

#### 시간표 일정 클릭 시 
```const [selectedRouteDetails, setSelectedRouteDetails] = useState(null);
const handleRouteSelect = async (e) => {
  const routeName = e.target.value;
  setSelectedRoute(routeName);

  const selected = pinnedRoutes.find((route) => route.name === routeName);
  if (!selected) return;

  setSelectedRouteId(selected.id);
  setSelectedRouteDetails(selected); 
  try {
    const snapshot = await get(ref(realtimeDb, `routes/${selected.id}/times`));
    if (snapshot.exists()) {
      const timeList = Object.values(snapshot.val());
      setAvailableTimes(timeList);
    } else {
      setAvailableTimes([]);
    }
  } catch (err) {
    console.error("시간 목록 불러오기 실패:", err);
  }
};
```
- `handleRouteSelect({ target: { value: matchedSchedule.route } });`
- `const routeName = e.target.value;`을 선택한 일정의 이름으로 저장
- `setSelectedRoute(routeName);` -> selectedRoute를 routeName으로 저장
- `const selected = pinnedRoutes.find((route) => route.name === routeName);` -> 노선 선택 드롭다운에서 route.name과 routeName이 같으면 찾아서 저장
- `setSelectedRouteId(selected.id);` -> selectedRouteId를 selected.id로 저장
- `setSelectedRouteDetails(selected); ` -> selectedRouteDetails를 selected로 저장
- 그 이후는 `routes/${selected.id}/times` 경로 데이터를 snapshot에 저장 후 snapshot이 존재하면 Js에 사용가능한 배열로 저장한 후 setAvailableTimes에 저장

#### 확인 버튼 클릭 시
```
<DriverScheduleDialog
  open={isAddDialogOpen}
  onClose={handleCloseAddDialog}
  onSave={handleSaveSchedule}
  pinnedRoutes={pinnedRoutes}
  availableTimes={availableTimes}
  selectedRoute={selectedRoute}
  selectedTime={selectedTime}
  selectedDays={selectedDays}
  duration={duration}
  onChangeRoute={handleRouteSelect}
  onChangeTime={(e) => setSelectedTime(e.target.value)}
  onChangeDays={(e) => setSelectedDays(e.target.value)}
  onChangeDuration={(e) => setDuration(e.target.value)}
/>
```
```const handleSaveSchedule = async () => {
  if (!currentTargetUser || !selectedDays || !selectedTime || !selectedRoute) {
    console.log("조건 안 맞아서 return 됨!");
    return;
  }
  const newSchedule = {
    route: selectedRoute,
    time: selectedTime,
    duration: duration, 
    days: selectedDays,
    createdAt: new Date().toISOString()
  };

  try {
    const scheduleRef = ref(
      realtimeDb,
      `drivers/${currentTargetUser.uid}/schedule`
    );

  if (isEditing && editingScheduleKey) {
    const targetRef = ref(
      realtimeDb,
     `drivers/${currentTargetUser.uid}/schedule/${editingScheduleKey}`
    );
    await set(targetRef, newSchedule);
  } else {
    const newRef = push(scheduleRef);
    await set(newRef, newSchedule);
  }

  setIsAddDialogOpen(false);
  setSelectedRoute('');
  setSelectedTime('');
  setSelectedDays([]);
  setSelectedCellTime('');
  setDuration('');
  setIsEditing(false); 
  setEditingScheduleKey(null);
  fetchDriverSchedule(currentTargetUser.uid);
  setExpandedDriverUid(currentTargetUser.uid);
  } catch (err) {
   console.error("스케줄 저장 실패:", err);
  }
};
```
- `if (!currentTargetUser || !selectedDays || !selectedTime || !selectedRoute) {` -> 하나라도 값이 없으면 return;
- `const newSchedule = {` -> route, time, duration, days, createAt 의 정보를 선택한 정보로 다시 저장
- `const scheduleRef = ref(` -> `drivers/${currentTargetUser.uid}/schedule` 경로의 데이터를 scheduleRef에 저장
- `if (isEditing && editingScheduleKey) {` -> 두 값이 다 존재하면
  - `drivers/${currentTargetUser.uid}/schedule/${editingScheduleKey}` 경로의 데이터를 targetRef에 저장
  - `await set(targetRef, newSchedule);` -> targetRef에 newSchedule을 덮어쓰기
- 완료 후 모든 값 초기화

#### 특정 기사의 스케쥴 가져오기
```const fetchDriverSchedule = async (uid) => {
  const scheduleRef = ref(realtimeDb, `drivers/${uid}/schedule`);
  const snapshot = await get(scheduleRef);
  if (snapshot.exists()) {
    const data = snapshot.val();
    setDriverSchedules((prev) => ({ ...prev, [uid]: data }));

    const newColored = analyzeSchedule(data);
    setColoredSchedule((prev) => ({
      ...prev, [uid]: newColored })); 
  } 
};
```
- `drivers/${uid}/schedule` 의 데이터를 가져와 snapshot에 저장 후 Js에서 사용가능한 배열로 저장
- `setDriverSchedules((prev) => ({ ...prev, [uid]: data }));` -> 기사별 스케쥴 데이터를 상태에 저장
- `analyzeSchedule()` 함수를 통해, 셀 색칠 위치 계산
- `setColoredSchedule((prev) => ({ ...prev, [uid]: newColored }));` 기사 uid 기준으로 어떤 셀을 칠할지 상태에 저장


```const analyzeSchedule = (scheduleObj) => {
  const newColoredCells = [];

  Object.values(scheduleObj).forEach((item) => {
    const { days, time, duration } = item;
    if (!days || !time || !duration) return;

    const [hourStr, minuteStr] = time.split(":");
    const hour = parseInt(hourStr, 10);
    const minute = parseInt(minuteStr, 10);
    let startIndex = -1;

    for (let i = 0; i < timeSlots.length; i++) {
      const slotHour = parseInt(timeSlots[i].split(":")[0], 10);
      const nextSlotHour =
        i < timeSlots.length - 1
          ? parseInt(timeSlots[i + 1].split(":")[0], 10)
          : 24;

      if (hour >= slotHour && hour < nextSlotHour) {
        startIndex = i;
        break;
      }
    }

    if (startIndex === -1) return; 
    days.forEach((day) => {
      const col = getDayIndex(day);
      for (let i = 0; i < duration; i++) {
        const row = startIndex + i;
        const cellKey = `${col}-${row}`;
        newColoredCells.push(cellKey);
      }
    });
  });
  return newColoredCells;
};
```
- `Object.values(scheduleObj).forEach((item) => {` -> 각 스케줄 항목에 대해 순회 시작
- `const [hourStr, minuteStr] = time.split(":"); const hour = parseInt(hourStr, 10);` -> 시간 정보 파싱
- 현재 스케줄이 어느 시간 칸(i번 row)에 해당하는지 계산해서 startIndex 저장 (예: 09:00 → 0번 row, 10:00 → 1번 row
- 식제 셀 좌표를 구해 색칠할 지 여부를 판단



  return (
    <Box sx={{ width: '100%' }}>
      <TabbedContainer
        tabIndex={tabIndex}
        handleTabChange={handleTabChange}
        labels={["기사별 운행 이력", "기사 계정 관리"]}
      />

      <Box
        sx={{
          backgroundColor: '#f5f5f5',
          borderRadius: 2,
          width: '100%',
        }}
      >
        <TabPanel value={tabIndex} index={0}>
          {hasPermission(admin, '기사별 운행 이력') ? (
          <Box sx={{ width: '100%', height: '100%', backgroundColor: '#fff',  }}>
            <SearchBar
              value={searchKeyword}
              onChange={(e) => setSearchKeyword(e.target.value)}
              placeholder="이름으로 검색"
            />
            <DriverListTable
              users={filteredUsers}
              expandedDriverUid={expandedDriverUid}
              driverSchedules={driverSchedules}
              coloredSchedule={coloredSchedule}
              onTimeClick={handleTimeOpen}
              onScheduleToggle={handleOpenSchedule}
              onAddClick={handleOpenAddDialog}
              onCellClick={(matchedSchedule) => {
                fetchPinnedRoutes(matchedSchedule.uid).then(() => {
                  setSelectedDays(matchedSchedule.days);
                  setSelectedTime(matchedSchedule.time);
                  setDuration(matchedSchedule.duration);
                  setSelectedRoute(matchedSchedule.route);
                  setCurrentTargetUser(matchedSchedule.user);
                  setIsEditing(true);
                  setEditingScheduleKey(matchedSchedule.key);
                  handleRouteSelect({ target: { value: matchedSchedule.route } });
                  setIsAddDialogOpen(true);
                });
              }}
            />
                
            <DriverScheduleDialog
              open={isAddDialogOpen}
              onClose={handleCloseAddDialog}
              onSave={handleSaveSchedule}
              pinnedRoutes={pinnedRoutes}
              availableTimes={availableTimes}
              selectedRoute={selectedRoute}
              selectedTime={selectedTime}
              selectedDays={selectedDays}
              duration={duration}
              onChangeRoute={handleRouteSelect}
              onChangeTime={(e) => setSelectedTime(e.target.value)}
              onChangeDays={(e) => setSelectedDays(e.target.value)}
              onChangeDuration={(e) => setDuration(e.target.value)}
            />
            <DrivingHistoryDialog
              open={isHistoryOpen}
              onClose={() => setIsHistoryOpen(false)}
              driverName={selectedDriver?.name}
              historyList={drivingHistory}
            />
          </Box>
          ) : (
          <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
        )}
        </TabPanel>
        <TabPanel value={tabIndex} index={1}>
          {hasPermission(admin, '기사 계정 관리') ? (
          <Box sx={{ width: '100%', height: '100%', backgroundColor: '#fff',  }}>
            <SearchBar
              value={searchKeyword}
              onChange={(e) => setSearchKeyword(e.target.value)}
              placeholder="이름으로 검색"
            />
            <UserTable
              users={filteredUsers}
              anchorEl={anchorEl}
              anchorUserId={anchorUserId}
              onMemoClick={handleOpenMemo}
              onMenuClick={handleMenuOpen}
              onMenuClose={handleMenuClose}
              onUnban={handleUnban}
              onReset={handleOpenReset}
              onEdit={handleOpenEditDialog}
              onMemoHistory={handleOpenMemoHistory}
            />
            <MemoDialog
              open={isMemoOpen}
              memoText={memoText}
              setMemoText={setMemoText}
              isWarning={isWarning}
              setIsWarning={setIsWarning}
              isBan={isBan}
              setIsBan={setIsBan}
              onConfirm={handleSubmitMemo}
              onCancel={handleCloseMemo}
            />
            <MemoHistoryDialog
              open={isMemoHistoryOpen}
              onClose={() => setIsMemoHistoryOpen(false)}
              memoFilter={memoFilter}
              setMemoFilter={setMemoFilter}
              memoList={selectedMemoList}
            />
            <ResetPasswordDialog
              open={isResetOpen}
              email={resetEmail}
              onChangeEmail={(e) => setResetEmail(e.target.value)}
              onSendResetEmail={handleSendResetEmail}
              onClose={() => setIsResetOpen(false)}
            />
          </Box>
          ) : (
          <Typography color="error">이 기능에 대한 권한이 없습니다.</Typography>
        )}
        </TabPanel>
      </Box>
    </Box>
  );
};

export default DriverManagement;
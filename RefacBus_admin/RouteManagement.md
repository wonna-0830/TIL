## RouteManagement.jsx 톺아보기


### import
```
import React, { useState } from 'react';
import { useEffect } from 'react';
import { onValue } from 'firebase/database';import StarIcon from '@mui/icons-material/Star';
import StarBorderIcon from '@mui/icons-material/StarBorder';
import AddIcon from '@mui/icons-material/Add';
import {
  TextField, Box, Table, TableHead, TableCell, TableRow, TableBody,
  Tab, Tabs, Dialog, DialogTitle, DialogContent, DialogActions,
  TableContainer, Button, Typography, Paper
} from '@mui/material';
import { ref, get, update, push, set } from "firebase/database";
import { realtimeDb, auth } from "../firebase";
import IconButton from "@mui/material/IconButton";
import AccessTimeIcon from "@mui/icons-material/AccessTime";
import ExpandMoreIcon from "@mui/icons-material/ExpandMore";
import ExpandLessIcon from '@mui/icons-material/ExpandLess';
import dayjs from "dayjs"; 
```

### 탭 전환 기능 -> ManagerMangement.jsx 68번 참고
```
const TabPanel = ({ children, value, index }) => {
  return (
    <div hidden={value !== index}>
      {value === index && (
        <Box sx={{ p: 3 }}>
          {children}
        </Box>
      )}
    </div>
  );
};

const PlaceTimeManagement = () => {
    const [tabIndex, setTabIndex] = useState(0);
    const [open, setOpen] = useState(false);
    const [route, setRoute] = useState('');
    const [isPinned, setIsPinned] = useState(false);
    const [routeList, setRouteList] = useState([]);

    const handleTabChange = (event, newValue) => {
        setTabIndex(newValue);
        };
    
```
- ```<Tabs
          value={tabIndex}
          onChange={handleTabChange}
          textColor="primary"
          indicatorColor="primary"
          variant="standard" 
          centered           
          sx={{
            minWidth: 'fit-content', 
          }}
        >
          <Tab label="노선  추가 / 삭제" />
          <Tab label="노선 시간대 설정 및 관리" />
          <Tab label="노선 정류장 설정 및 관리" />
        </Tabs>
        ...
        <TabPanel value={tabIndex} index={0}>
    ```    

### 노선 추가 기능의 useEffect

```
    useEffect(() => {
      const routeListRef = ref(realtimeDb, 'routes');
      onValue(routeListRef, snapshot => {
        const data = snapshot.val();
        if (data) {
          const list = Object.entries(data).map(([id, item]) => ({
            id,
            ...item
          }));
          list.sort((a, b) => b.isPinned - a.isPinned);
          setRouteList(list);
        } else {
          setRouteList([]);
        }
      });
    }, []);
```    
- `const routeListRef = ref(realtimeDb, 'routes');` -> realtimeDB 내 routes라는 데이터베이스 참조해여 routeListRef에 저장
- `onValue(reference, callback)` -> 데이터를 실시간으로 감지하는 함수
  - `reference = routeListRef` -> 어디를 감지할건지 경로를 지정한 레퍼런스 객체
  - `callback = snapshot` -> routeListRef에 변화가 생길 때 마다 데이터들을 가져옴
- `const data = snapshot.val();` -> 가져온 snapshot을 JS에서 쓸 수 있게끔 변환 후 data에 저장
- if~ data가 있다면
  - `const list = Object.entries(data).map(([id, item]) => ({` -> data의 id와 item을 키-값 쌍 배열로 변환 후 list에 저장
    - `({id, ...item})` -> value의 쌍을 전체로 펼쳐서 배열 만듦
  - `list.sort((a, b) => b.isPinned - a.isPinned);` -> 배열 간 비교를 통해 isPinned가 true인 배열들을 위로 배치
  - `setRouteList`를 통해 routeList 상태를 list로 
    - `else` -> routeList를 빈 배열 상태로

### 노선 추가 기능 (index = 0)
#### 관련 UI
```
<TabPanel value={tabIndex} index={0}>
          <Button
            variant="contained"
            startIcon={<AddIcon />}
            ***onClick={() => setOpen(true)}***
            sx={{ mb: 2 }}
              >
            새 노선 등록
          </Button>
          
          {/* 팝업 다이얼로그 */}
          ***<Dialog open={open} onClose={() => setOpen(false)}>***
            <DialogTitle>노선 등록</DialogTitle>
              <DialogContent>
                <TextField
                  label="노선"
                  fullWidth
                  margin="normal"
                  value={route}
                  onChange={e => setRoute(e.target.value)}
                  />
                
                <Box sx={{ display: 'flex', alignItems: 'center', mt: 2 }}>
                  <IconButton onClick={() => setIsPinned(prev => !prev)}>
                    {isPinned ? <StarIcon color="primary" /> : <StarBorderIcon />}
                  </IconButton>
                  <Typography>{isPinned ? '' : '현사용 노선에 등록'}</Typography>
                </Box>
                    <Box sx={{ mt: 2 }}>
                        <Button variant="contained" onClick={handleSubmit}>등록</Button>
                        <Button onClick={() => setOpen(false)} sx={{ ml: 1 }}>취소</Button>
                      </Box>
                    </DialogContent>
                    </Dialog>
                    <TableContainer component={Paper}>
                      <Table>
                        <TableHead>
                          <TableRow>
                            <TableCell style={{ width: "50%" }}>노선</TableCell>
                            <TableCell style={{ width: "40%" }}>등록 날짜</TableCell>
                            <TableCell style={{ width: "10%" }}>고정</TableCell>
                          </TableRow>
                        </TableHead>
                        <TableBody>
                          {routeList.map((item) => (
                            <TableRow key={item.id}>
                              <TableCell>
                                <a>
                                  {item.name}
                                </a>
                              </TableCell>
                              <TableCell>{new Date(item.date).toLocaleDateString()}</TableCell>
                              <TableCell>
                                <IconButton onClick={() => togglePinned(item.id, item.isPinned)}>
                                  {item.isPinned ? <StarIcon color="warning" /> : <StarBorderIcon />}
                                </IconButton>
                              </TableCell>
                            </TableRow>
                          ))}
                        </TableBody>
                      </Table>
                    </TableContainer>
        </TabPanel>
```
#### 노선 정보 저장하기
```  
    const handleSubmit = () => {
      const newRoute = {
        name: route,
        isPinned,
        date: new Date().toISOString(),
        writer: '관리자',
      };
  
      const newRef = push(ref(realtimeDb, 'routes'));
      set(newRef, newRoute);
  
      // 초기화
      setOpen(false);
      setRoute('');
      setIsPinned(false);
    };
  
```   
- `새 노선 등록` 버튼 클릭 시 setOpen(true) -> open 상태가 true가 됨
- `노선 등록` 다이얼로그가 open = true 여야 모달이 열림
  - `label="노선"인 TextField` -> value를 {route}로 받고있음
  - `<IconButton onClick={() => setIsPinned(prev => !prev)}>` -> 별표 클릭 칸도 디폴트 값인 isPinned(false)를 클릭하면 true로 바꿈
- `등록 버튼` 클릭 시 handleSubmit 함수 실행
  - `const newRoute = {name, isPinned, date: new Date().toISOString(), writer: '관리자',};` -> newRoute 내의 구성 정의 (데이터클래스)
  - `const newRef = push(ref(realtimeDb, 'routes'));` -> routes 데이터베이스 내 push 하는 함수 선언
  - `set(newRef, newRoute);` -> 모달에서 받아온 newRoute(TextField, IconButton, 현재 날짜 정보 등)를 routes 데이터베이스 안에 push
  - open(false)로 만들어 모달 창 닫기
  - 다음 추가할 노선을 위해 route 내 상태도 공란으로, isPinned 상태도 false로
  
#### 테이블 내 isPinned 상태 변경
```
    const togglePinned = (id, currentState) => {
    const routeRef = ref(realtimeDb, `routes/${id}`);
    update(routeRef, { isPinned: !currentState });
    };
```
- ``` <IconButton onClick={() => togglePinned(item.id, item.isPinned)}>
                                  {item.isPinned ? <StarIcon color="warning" /> : <StarBorderIcon />}
    ```
  - `{routeList.map((item) => (<TableRow key={item.id}>` -> TableBody 정의할 때 
    - useEffect에서 routes 데이터베이스에 데이터가 존재하면 routeList() 안에 데이터 배열이 들어있는상태 -> id 아이템을 가져올 수 있음
  - 클릭 시 togglePinned(클릭한 부분의 해당 id, isPinned 상태)를 실행
  - `const routeRef = ref(realtimeDb, `routes/${id}`);` -> realtimeDb의 routes 내 id들을 참조해 저장
  - `update(routeRef, { isPinned: !currentState });` -> routeRef에 isPinned의 현재 상태(currentState)의 반대(!)로 업데이트

### 검색 필터링 기능 (index = 1, 2) -> userMangement.jsx 79번 줄 참고
```
    const [searchKeyword, setSearchKeyword] = useState('');
    const [filteredRoutes, setFilteredRoutes] = useState([]);
    const [allRoutes, setAllRoutes] = useState([]);
    const [isRouteOpen, setIsRouteOpen] = useState(false);
    const [selectedRoute, setSelectedRoute] = useState(null);
    const [routeText, setRouteText] = useState("");
    const [openRouteId, setOpenRouteId] = useState(null);

      useEffect(() => {
        const fetchRoutes = async () => {
          const snapshot = await get(ref(realtimeDb, "routes"));
          const routesData = snapshot.val();
          const routesArray = Object.entries(routesData).map(([uid, value]) => ({
            uid,
            ...value,
          }));
          const pinnedRoutes = routesArray.filter(route => route.isPinned);
          setAllRoutes(pinnedRoutes);
          setFilteredRoutes(pinnedRoutes);
        };
        fetchRoutes();
      }, []);
    
      useEffect(() => {
        const filtered = allRoutes.filter((route) =>
          (route.name ?? '').toLowerCase().includes(searchKeyword.toLowerCase())
        );
        setFilteredRoutes(filtered);
      }, [searchKeyword, allRoutes]);
```

```
      const handleMenuOpen = (event, route) => {
        setSelectedRoute(route); 
        setIsRouteOpen(true);
      };
    
      const handleCloseMemo = () => {
        setSelectedRoute(null);
        setIsRouteOpen(false);
        setRouteText("");
      };
```

### 노선 시간대, 정류장 관리 (index = 1, 2)

#### 노선 시간대, 정류장 추가
```
<Dialog open={isRouteOpen} onClose={handleCloseMemo}>
              <DialogTitle>시간대 추가</DialogTitle>
              <DialogContent>
                <TextField
                  label="시간 입력"
                  multiline
                  fullWidth
                  value={routeText}
                  onChange={(e) => setRouteText(e.target.value)}
                />
              </DialogContent>
              <DialogActions>
                <Button onClick={handleSubmitMemo} color="primary">확인</Button>
                <Button onClick={handleCloseMemo} color="secondary">취소</Button>
              </DialogActions>
            </Dialog>

<Dialog open={isRouteOpen} onClose={handleCloseMemo}>
              <DialogTitle>정류장 추가</DialogTitle>
              <DialogContent>
                <TextField
                  label="정류장 입력"
                  multiline
                  fullWidth
                  value={routeText}
                  onChange={(e) => setRouteText(e.target.value)}
                />
              </DialogContent>
              <DialogActions>
                <Button onClick={handleSubmitMemo} color="primary">확인</Button>
                <Button onClick={handleCloseMemo} color="secondary">취소</Button>
              </DialogActions>
            </Dialog> 
```

```
      const handleSubmitMemo = async () => {
        if (!selectedRoute || !routeText.trim()) return;

        const targetKey = tabIndex === 1 ? "times" : "stops"; // ✅ 시간/정류장 구분
        const dataRef = ref(realtimeDb, `routes/${selectedRoute.uid}/${targetKey}`);
        await push(dataRef, routeText.trim());

        const updatedSnapshot = await get(ref(realtimeDb, `routes/${selectedRoute.uid}`));
        const updatedData = updatedSnapshot.val();

        setAllRoutes((prev) =>
          prev.map((route) =>
            route.uid === selectedRoute.uid ? { ...route, [targetKey]: updatedData[targetKey] } : route
          )
        );
        setFilteredRoutes((prev) =>
          prev.map((route) =>
            route.uid === selectedRoute.uid ? { ...route, [targetKey]: updatedData[targetKey] } : route
          )
        );

        // 초기화
        setIsRouteOpen(false);
        setRouteText('');
        setSelectedRoute(null);
      };
```
- `if (!selectedRoute || !routeText.trim()) return;` -> 선택한 노선이 없거나, routeText 가 null이면 return (UI의 TextField내 value가 routeText임)
- `const targetKey = tabIndex === 1 ? "times" : "stops";` -> tebIndex가 1이면 시간대, 1이 아니면 정류장 관련이므로 구분해서 로직 실행 (0에서는 쓰이지않으므로 예외가 됨)
- `const dataRef = ref(realtimeDb, routes/${selectedRoute.uid}/${targetKey});` -> realtimeDb 내 routes 데이터베이스의 해당 uid 안 times과 stops(targetkey: 인덱스가 1인지 2인지에 따라) 담아 dataRef에 저장
- `await push(dataRef, routeText.trim());` -> routeText의 공백을 없애고(trim()), dataRef에 push
- `setAllRoutes, setFilterRoutes` -> 검색 필터링 시 useEffect가 실행되면서 생기는 데이터 모임
  - 여기에 route의 uid와 내가 선택한 route의 uid가 같으면 인덱스에 따라 times 또는 stops에 데이터를 갱신
- `setIsRouteOpen(false); setRouteText(''); setSelectedRoute(null);` -> isRouteOpen의 상태를 false로 해 창을 닫고, 다음 정류장과 시간대 추가를 위해 routeText와 selectedRoute의 값을 비움


#### 시간대 아코디언 열고 닫기 + 목록 정렬
```
                <TableCell>
                    <IconButton onClick={() => handleToggleTimes(route.uid)}>
                          {openRouteId === route.uid ? (
                            <ExpandLessIcon />
                          ) : (
                            <ExpandMoreIcon />
                          )}
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  {openRouteId === route.uid && (
                    <TableRow>
                      <TableCell colSpan={3}>
                        {route.times
                          ? (
                            <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 1 }}>
                            {Object.entries(route.times).map(([timeId, timeValue]) => (
                              <Box
                                key={timeId}
                                onClick={() => {
                                  setSelectedTimeInfo({ routeId: route.uid, timeId, value: timeValue });
                                  setEditTimeText(timeValue);
                                  setOpenTimeDialog(true);
                                }}
                              >
                                {timeValue}
                              </Box>
                            ))}
                          </Box>
                        ) : (
                          <Typography color="text.secondary">등록된 시간대가 없습니다</Typography>
                        )}
                      </TableCell>
                    </TableRow>
```

```
      const handleToggleTimes = (uid) => {
      setOpenRouteId(prev => (prev === uid ? null : uid));
      };
```
- `<IconButton onClick={() => handleToggleTimes(route.uid)}>` -> 버튼 클릭 시 route.uid를 담아 handleToggleTimes 실행
- `setOpenRouteId(prev => (prev === uid ? null : uid));` -> 현재 openRouteId의 상태와 uid가 같으면 null(아코디언 목록 닫기) : 아니면 클릭한 uid에 대해서 목록 열기
- `{openRouteId === route.uid && (` -> openRouteId와 route.uid가 같으면 
  - `<TableCell colSpan={3}>` -> 제일 위 목록 4개의 칼럼 때문에 정렬이 깨지므로 병합
  - `{route.times? (` -> route.times가 존재하면?
    - ```<Box>
        {Object.entries(route.times).map(([timeId, timeValue]) => (
        ```
    - route.times에 대한 키-값 배열을 timeId와 timeValue로 매핑 (전체 박스)
    - ```<Box
        key={timeId}
        onClick={() => {
        setSelectedTimeInfo({ routeId: route.uid, timeId, value: timeValue });
        setEditTimeText(timeValue);
        setOpenTimeDialog(true);
        }}
        >
        {timeValue}
      </Box>
      ```
      - `<Box key={timeId} ...>{timeValue}</Box>`로 각각의 시간대 노출
      - `onClick` 각 시간대 박스 클릭 시
        - selectedTimeInfo를 위 상태로 저장
        - editTimeText를 timeValue 값으로 저장 (기존 시간대를 수정창에 텍스트로 채워넣기)
        - openTimeDialog를 true로 -> dialog가 열리고 수정 삭제 가능


#### 다이얼로그 내 수정/삭제
```
                <Dialog open={openTimeDialog} onClose={() => setOpenTimeDialog(false)}>
              <DialogTitle>시간 수정 / 삭제</DialogTitle>
              <DialogContent>
                <TextField
                  label="시간"
                  value={editTimeText}
                  onChange={(e) => setEditTimeText(e.target.value)}
                  fullWidth
                />
              </DialogContent>
              <DialogActions>
                <Button
                  color="error"
                  onClick={async () => {
                    if (!selectedTimeInfo) return;
                    const { routeId, timeId } = selectedTimeInfo;
                    await update(ref(realtimeDb, `routes/${routeId}/times`), {[timeId]: null,});
                   
                    const updatedSnapshot = await get(ref(realtimeDb, `routes/${routeId}`));
                    const updatedData = updatedSnapshot.val();
                   
                    setAllRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setFilteredRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setOpenTimeDialog(false);
                  
                  }}
                >
                  삭제
                </Button>
                <Button
                  color="primary"
                  onClick={async () => {
                    if (!selectedTimeInfo) return;
                    const { routeId, timeId } = selectedTimeInfo;
                    await set(ref(realtimeDb, `routes/${routeId}/times/${timeId}`), editTimeText); // 수정
                    const updatedSnapshot = await get(ref(realtimeDb, `routes/${routeId}`));
                    const updatedData = updatedSnapshot.val();
                   
                    setAllRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setFilteredRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setOpenTimeDialog(false);                 
                  }}
                >
                  수정
                </Button>
              </DialogActions>
            </Dialog>
```
- openTimeDialog가 true-> 다이얼로그 열림
- `<DialogContent><TextField>` -> editTimeText에 저장되었던 Value 값 노출, 값이 바뀌면 editTimeText에 바뀐 내용이 저장됨
- 삭제 버튼 클릭 시
  - ```if (!selectedTimeInfo) return;
     const { routeId, timeId } = selectedTimeInfo;
     await update(ref(realtimeDb, `routes/${routeId}/times`), {[timeId]: null,});
         ```
    - selectedTimeInfo가 없으면 return (작성한 내용이 null이면 실행X)
    - `const~` -> selectedTimeInfo 의 값들 중에 routeid와 timeId만 꺼내 변수로 만듦 (객체 구조 분해 할당)
    - `await update(~)` -> routes 데이터베이스 내 times 안에 timeId를 null로 업데이트 (삭제)
    - ```const updatedSnapshot = await get(ref(realtimeDb, routes/${routeId}));
      const updatedData = updatedSnapshot.val();
        ```
      - updatedSnapshot에 routes의 routeId를 가져와서 updatedData에 updatedSnapshot을 JS로 변환 가능하게 만들어 저장 (삭제 사항을 웹에서 표시할 수 있도록)
    - `setAllRoute(), setFilteredRoute()` 으로 전체 데이터를 업데이트
  - 수정 버튼 클릭 시
    - 삭제 기능과 똑같이 실행하다가
    - `await set(ref(realtimeDb, routes/${routeId}/times/${timeId}), editTimeText); ` -> routes 데이터베이스 내 times의 timeId 값에 대해서 editTimeText로 덮어쓰기 (수정)
    - 나머지도 그대로 삭제 버튼과 동일하게 실행
                    

#### 정류장 아코디언 목록 오픈
```
                <IconButton onClick={(e) => handleStopOpen(e, route)}>
                          <AccessTimeIcon />
                        </IconButton>
                      </TableCell>
                      <TableCell>
                        <IconButton onClick={() => handleToggleTimes(route.uid)}>
                          {openRouteId === route.uid ? (
                            <ExpandLessIcon />
                          ) : (
                            <ExpandMoreIcon />
                          )}
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  {openRouteId === route.uid && (
                    <TableRow>
                      <TableCell colSpan={4}>
                        <Typography sx={{ mb: 1 }}>🚌 정류장 목록</Typography>
                        {route.stops ? (
                          <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 1 }}>
                            {Object.entries(route.stops).map(([stopId, stopName]) => (
                              <Box
                                key={stopId}
                              >
                                {stopName}
                              </Box>
                            ))}
                          </Box>
                        ) : (
                          <Typography color="text.secondary">등록된 정류장이 없습니다</Typography>
                        )}
                      </TableCell>
                    </TableRow>
                  )}
                </React.Fragment>
                ))}
              </TableBody>
```

```
      const [selectedTimeInfo, setSelectedTimeInfo] = useState(null); // { routeId, timeId, value }
      const [openTimeDialog, setOpenTimeDialog] = useState(false);
      const [editTimeText, setEditTimeText] = useState('');

      
      const [isStopDialogOpen, setIsStopDialogOpen] = useState(false);
      const [selectedRouteForStop, setSelectedRouteForStop] = useState(null);
      const [newStopText, setNewStopText] = useState("");

      const handleStopOpen = (event, route) => {
        setSelectedRouteForStop(route);  
        setIsStopDialogOpen(true); 
      };
```
    
### UI
```
  return (
    <Box sx={{ width: '100%' }}>
      
      {/* 탭 메뉴 */}
      <Box
        sx={{
          backgroundColor: '#fff',
          py: 1,
          px: 5,
          boxShadow: 1,
        }}
      >
        <Tabs
          value={tabIndex}
          onChange={handleTabChange}
          textColor="primary"
          indicatorColor="primary"
          variant="standard" // ← 요거!
          centered             // ← 요거!
          sx={{
            minWidth: 'fit-content',  // ← 너무 좁게 붙는 거 방지
          }}
        >
          <Tab label="노선  추가 / 삭제" />
          <Tab label="노선 시간대 설정 및 관리" />
          <Tab label="노선 정류장 설정 및 관리" />
        </Tabs>
      </Box>

      {/* 회색 박스 본문 */}
      <Box
        sx={{
          backgroundColor: '#f5f5f5',
          borderRadius: 2,
          marginTop: 1,
          width: '100%',
          maxWidth: 'none',
        }}
      >
        <TabPanel value={tabIndex} index={0}>
          <Button
            variant="contained"
            startIcon={<AddIcon />}
            onClick={() => setOpen(true)}
            sx={{ mb: 2 }}
              >
            새 노선 등록
          </Button>
          
          {/* 팝업 다이얼로그 */}
          <Dialog open={open} onClose={() => setOpen(false)}>
            <DialogTitle>노선 등록</DialogTitle>
              <DialogContent>
                <TextField
                  label="노선"
                  fullWidth
                  margin="normal"
                  value={route}
                  onChange={e => setRoute(e.target.value)}
                  />
                
                <Box sx={{ display: 'flex', alignItems: 'center', mt: 2 }}>
                  <IconButton onClick={() => setIsPinned(prev => !prev)}>
                    {isPinned ? <StarIcon color="primary" /> : <StarBorderIcon />}
                  </IconButton>
                  <Typography>{isPinned ? '' : '현사용 노선에 등록'}</Typography>
                </Box>
                <Box sx={{ mt: 2 }}>
                    <Button variant="contained" onClick={handleSubmit}>등록</Button>
                    <Button onClick={() => setOpen(false)} sx={{ ml: 1 }}>취소</Button>
                </Box>
            </DialogContent>
        </Dialog>
        <TableContainer component={Paper}>
            <Table>
                <TableHead>
                    <TableRow>
                        <TableCell style={{ width: "50%" }}>노선</TableCell>
                        <TableCell style={{ width: "40%" }}>등록 날짜</TableCell>
                        <TableCell style={{ width: "10%" }}>고정</TableCell>
                    </TableRow>
                </TableHead>
                <TableBody>
                    {routeList.map((item) => (
                        <TableRow key={item.id}>
                            <TableCell>
                                <a>
                                  {item.name}
                                </a>
                            </TableCell>
                            <TableCell>{new Date(item.date).toLocaleDateString()}</TableCell>
                            <TableCell>
                                <IconButton onClick={() => togglePinned(item.id, item.isPinned)}>
                                  {item.isPinned ? <StarIcon color="warning" /> : <StarBorderIcon />}
                                </IconButton>
                            </TableCell>
                        </TableRow>
                        ))}
                </TableBody>
            </Table>
        </TableContainer>
        </TabPanel>
        <TabPanel value={tabIndex} index={1}>
          <Box sx={{ width: '100%', backgroundColor: '#fff', padding: 2 }}>
            <TextField
              label="노선으로 검색"
              variant="outlined"
              size="small"
              value={searchKeyword}
              onChange={(e) => setSearchKeyword(e.target.value)}
              sx={{ width: '500px', mb: 2 }}
            />

            <Table>
              <TableHead>
                <TableRow>
                  <TableCell style={{ width: "40%" }}>노선명</TableCell>
                  <TableCell style={{ width: "20%" }}>일일 운행 회수</TableCell>
                  <TableCell style={{ width: "20%" }}>시간대 추가</TableCell>
                  <TableCell style={{ width: "20%" }}>시간대 목록</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {filteredRoutes.map((route) => (
                  <React.Fragment key={route.uid}>
                    <TableRow key={route.uid}>
                      <TableCell>{route.name}</TableCell>
                      <TableCell>{route.times ? Object.keys(route.times).length + '회' : '0회'}</TableCell>
                      <TableCell>
                        <IconButton onClick={(e) => handleMenuOpen(e, route)}>
                          <AccessTimeIcon />
                        </IconButton>
                      </TableCell>
                      <TableCell>
                        <IconButton onClick={() => handleToggleTimes(route.uid)}>
                          {openRouteId === route.uid ? (
                            <ExpandLessIcon />
                          ) : (
                            <ExpandMoreIcon />
                          )}
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  {openRouteId === route.uid && (
                    <TableRow>
                      <TableCell colSpan={3}>
                        {route.times
                          ? (
                            <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 1 }}>
                            {Object.entries(route.times).map(([timeId, timeValue]) => (
                              <Box
                                key={timeId}
                                onClick={() => {
                                  setSelectedTimeInfo({ routeId: route.uid, timeId, value: timeValue });
                                  setEditTimeText(timeValue);
                                  setOpenTimeDialog(true);
                                }}
                                sx={{
                                  cursor: 'pointer',
                                  width: '19%',
                                  backgroundColor: '#e3f2fd',
                                  padding: '8px',
                                  borderRadius: '4px',
                                  textAlign: 'center',
                                  boxShadow: 1,
                                }}
                              >
                                {timeValue}
                              </Box>
                            ))}

                          </Box>
                        ) : (
                          <Typography color="text.secondary">등록된 시간대가 없습니다</Typography>
                        )}
                      </TableCell>
                    </TableRow>
                  )}
                </React.Fragment>
                ))}
              </TableBody>
            </Table>

            <Dialog open={isRouteOpen} onClose={handleCloseMemo}>
              <DialogTitle>시간대 추가</DialogTitle>
              <DialogContent>
                <TextField
                  label="시간 입력"
                  multiline
                  fullWidth
                  value={routeText}
                  onChange={(e) => setRouteText(e.target.value)}
                />
              </DialogContent>
              <DialogActions>
                <Button onClick={handleSubmitMemo} color="primary">확인</Button>
                <Button onClick={handleCloseMemo} color="secondary">취소</Button>
              </DialogActions>
            </Dialog>
            
            <Dialog open={openTimeDialog} onClose={() => setOpenTimeDialog(false)}>
              <DialogTitle>시간 수정 / 삭제</DialogTitle>
              <DialogContent>
                <TextField
                  label="시간"
                  value={editTimeText}
                  onChange={(e) => setEditTimeText(e.target.value)}
                  fullWidth
                />
              </DialogContent>
              <DialogActions>
                <Button
                  color="error"
                  onClick={async () => {
                    if (!selectedTimeInfo) return;
                    const { routeId, timeId } = selectedTimeInfo;
                    await update(ref(realtimeDb, `routes/${routeId}/times`), {[timeId]: null,});
                   
                    const updatedSnapshot = await get(ref(realtimeDb, `routes/${routeId}`));
                    const updatedData = updatedSnapshot.val();
                   
                    setAllRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setFilteredRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setOpenTimeDialog(false);
                  
                  }}
                >
                  삭제
                </Button>
                <Button
                  color="primary"
                  onClick={async () => {
                    if (!selectedTimeInfo) return;
                    const { routeId, timeId } = selectedTimeInfo;
                    await set(ref(realtimeDb, `routes/${routeId}/times/${timeId}`), editTimeText); // 수정
                    const updatedSnapshot = await get(ref(realtimeDb, `routes/${routeId}`));
                    const updatedData = updatedSnapshot.val();
                   
                    setAllRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setFilteredRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === routeId ? { ...route, times: updatedData.times } : route
                      )
                    );
                    setOpenTimeDialog(false);                 
                  }}
                >
                  수정
                </Button>
              </DialogActions>
            </Dialog>

          </Box>
        </TabPanel>
        <TabPanel value={tabIndex} index={2}>
          <Box sx={{ width: '100%', backgroundColor: '#fff', padding: 2 }}>
            <TextField
              label="노선으로 검색"
              variant="outlined"
              size="small"
              value={searchKeyword}
              onChange={(e) => setSearchKeyword(e.target.value)}
              sx={{ width: '500px', mb: 2 }}
            />

            <Table>
              <TableHead>
                <TableRow>
                  <TableCell style={{ width: "40%" }}>노선명</TableCell>
                  <TableCell style={{ width: "20%" }}>정류장 수</TableCell>
                  <TableCell style={{ width: "20%" }}>정류장 추가</TableCell>
                  <TableCell style={{ width: "20%" }}>정류장 목록</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {filteredRoutes.map((route) => (
                  <React.Fragment key={route.uid}>
                    <TableRow key={route.uid}>
                      <TableCell>{route.name}</TableCell>
                      <TableCell>
                        {route.stops ? Object.keys(route.stops).length + '개' : '0개'}
                      </TableCell>
                      <TableCell>
                        <IconButton onClick={(e) => handleStopOpen(e, route)}>
                          <AccessTimeIcon />
                        </IconButton>
                      </TableCell>
                      <TableCell>
                        <IconButton onClick={() => handleToggleTimes(route.uid)}>
                          {openRouteId === route.uid ? (
                            <ExpandLessIcon />
                          ) : (
                            <ExpandMoreIcon />
                          )}
                        </IconButton>
                      </TableCell>
                    </TableRow>
                  {openRouteId === route.uid && (
                    <TableRow>
                      <TableCell colSpan={4}>
                        <Typography sx={{ mb: 1 }}>🚌 정류장 목록</Typography>
                        {route.stops ? (
                          <Box sx={{ display: 'flex', flexWrap: 'wrap', gap: 1 }}>
                            {Object.entries(route.stops).map(([stopId, stopName]) => (
                              <Box
                                key={stopId}
                                sx={{
                                  width: '19%',
                                  backgroundColor: '#fce4ec',
                                  padding: '8px',
                                  borderRadius: '4px',
                                  textAlign: 'center',
                                  boxShadow: 1,
                                }}
                              >
                                {stopName}
                              </Box>
                            ))}
                          </Box>
                        ) : (
                          <Typography color="text.secondary">등록된 정류장이 없습니다</Typography>
                        )}
                      </TableCell>
                    </TableRow>
                  )}
                </React.Fragment>
                ))}
              </TableBody>
            </Table>

            <Dialog open={isRouteOpen} onClose={handleCloseMemo}>
              <DialogTitle>정류장 추가</DialogTitle>
              <DialogContent>
                <TextField
                  label="정류장 입력"
                  multiline
                  fullWidth
                  value={routeText}
                  onChange={(e) => setRouteText(e.target.value)}
                />
              </DialogContent>
              <DialogActions>
                <Button onClick={handleSubmitMemo} color="primary">확인</Button>
                <Button onClick={handleCloseMemo} color="secondary">취소</Button>
              </DialogActions>
            </Dialog>
            
            <Dialog open={isStopDialogOpen} onClose={() => setIsStopDialogOpen(false)}>
              <DialogTitle>정류장 추가</DialogTitle>
              <DialogContent>
                <TextField
                  label="정류장명"
                  multiline
                  fullWidth
                  value={newStopText}
                  onChange={(e) => setNewStopText(e.target.value)}
                />
              </DialogContent>
              <DialogActions>
                <Button
                  color="primary"
                  onClick={async () => {
                    if (!selectedRouteForStop || !newStopText.trim()) return;
                    const stopsRef = ref(realtimeDb, `routes/${selectedRouteForStop.uid}/stops`);
        
                    await push(stopsRef, newStopText.trim());

                    // 최신 데이터 가져와서 로컬 상태 갱신
                    const updatedSnapshot = await get(ref(realtimeDb, `routes/${selectedRouteForStop.uid}`));
                    const updatedData = updatedSnapshot.val();

                    setAllRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === selectedRouteForStop.uid ? { ...route, stops: updatedData.stops } : route
                      )
                    );
                    setFilteredRoutes((prev) =>
                      prev.map((route) =>
                        route.uid === selectedRouteForStop.uid ? { ...route, stops: updatedData.stops } : route
                      )
                    );

                    // 입력값 초기화
                    setNewStopText("");
                    setIsStopDialogOpen(false);
                  }}
                  
                >
                  확인
                </Button>
                <Button onClick={() => setIsStopDialogOpen(false)} color="secondary">취소</Button>
              </DialogActions>
            </Dialog>

          </Box>
        </TabPanel>
      </Box>
    </Box>
  );
};

export default PlaceTimeManagement;
```
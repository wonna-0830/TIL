## 지금까지 했던 웹 페이지 제작기 중 ManagerManagement.jsx 파일 내 코드들 공부 (2025.07.08)

### 기본 import문
```import React, { useState } from 'react';
import {
  Dialog, DialogTitle, DialogContent, TextField, IconButton, Button, Typography
} from '@mui/material';
import { useEffect } from 'react';
import { onValue } from 'firebase/database';
import Table from '@mui/material/Table';
import TableBody from '@mui/material/TableBody';
import TableCell from '@mui/material/TableCell';
import TableContainer from '@mui/material/TableContainer';
import TableHead from '@mui/material/TableHead';
import TableRow from '@mui/material/TableRow';
import Paper from '@mui/material/Paper';
import StarIcon from '@mui/icons-material/Star';
import StarBorderIcon from '@mui/icons-material/StarBorder';
import AddIcon from '@mui/icons-material/Add';
import { push, ref, set, update } from 'firebase/database';
import { realtimeDb } from '../firebase'; 
import Box from '@mui/material/Box';
import Tabs from '@mui/material/Tabs';
import Tab from '@mui/material/Tab';
```

- 나에게 필요한 모듈 가져오기
- import {모듈 내 가져올 멤버} from 모듈
  - { } 내 여러개의 멤버를 가져올 수 있음
- @mui/material -> MUI에서 제공하는 UI 컴포넌트

---

### TabPanel 컴포넌트
```const TabPanel = ({ children, value, index }) => {
  return (
    <div hidden={value !== index}>
      {value === index && (
        <Box sx={{ p: 3 }}>{children}</Box>
      )}
    </div>
  );
};
```

- 탭 메뉴 전환 시 해당되는 내용만 보여줌
  - value: 현재 내가 선택한 탭 번호, index: 각 탭의 고유 번호 (아래 UI 부분에 고유 번호 부여됨)
  - **value === index** 로 같은 번호일 때만 내용을 보여줌
  - children: 내용
    - `<TabPanel value={tabIndex} index={0}>관리자 역할 구분</TabPanel>` 내 **관리자 역할 구분**이 childern임

---

### useState 상태관리
```const [tabIndex, setTabIndex] = useState(0); 
const [open, setOpen] = useState(false); 
const [title, setTitle] = useState('');
const [url, setUrl] = useState('');
const [isPinned, setIsPinned] = useState(false);
const [notices, setNotices] = useState([]);
```

- 화면에서 사용자가 어떤 동작을 했는지, 어떤 데이터를 가지고 있는지 기억
- **const [상태변수, 상태변수를 바꾸는 함수] = useState(초기값)**

---

### 탭 전환 함수
```const handleTabChange = (event, newValue) => {
  setTabIndex(newValue);
};
```

- 사용자가 탭을 눌렀을 때 어떤 탭이 선택됐는지 저장하는 역할
  - 탭 0번을 누르면 setTabIndex(0)
  - 탭 1번을 누르면 setTabIndex(1)
  - 탭 2번을 누르면 setTabIndex(2) ...
- 현재 선택된 탭 번호를 저장해야만 TabPanel 컴포넌트에서 조건을 보고 내용을 보여줄 수 있음
- ```<Tabs
          value={tabIndex}
          onChange={handleTabChange}
          ...
        </Tabs>
      </Box>
    <Box
        sx={{
          ...
        }}
      >
        <TabPanel value={tabIndex} index={0}>관리자 역할 구분</TabPanel>
  ```
  - 1. Tabs 컴포넌트는 동작 시 onChange 함수를 실행시킴
  - 2. handleTabChange 실행
  - 3. setTabIndex 상태 업테이트 함수를 통해 tabIndex의 값을 사용자가 누른 인덱스 값으로 바꿈
  - 4. TabPanel 내 value={tabIndex} 를 통해 저장된 인덱스 번호를 조회 해 내용을 보여줄 수 있음

---

### 공지사항 등록 함수
```const handleSubmit = () => {
  const newNotice = {
    title,
    url,
    isPinned,
    date: new Date().toISOString(),
    writer: '관리자',
  };
  const newRef = push(ref(realtimeDb, 'notices'));
  set(newRef, newNotice);
  setOpen(false);
  setTitle('');
  setUrl('');
  setIsPinned(false);
 }; 
```

- newNotice는 여러개의 입력값을 하나의 객체로 묶은 것
  - Kotlin으로 치면 data class랑 비슷한 역할이라고 생각하면 됨
- newRef는 Firebase 내 realtime DB의 경로를 가르키는 레퍼런스
  - push()는 firebase에서 고유한 키를 자동으로 만들어주는 함수
  - ref()는 어디에 저장할 건지 경로를 지정해줌 (notices에 저장)
  - ```<TextField
        label="제목"
        fullWidth
        margin="normal"
        value={title}
        onChange={e => setTitle(e.target.value)}
    />``` 와 같이 사용자가 입력한 값을 useState의 setTitle을 사용해 입력값 변경
  - `<Button variant="contained" onClick={handleSubmit}>등록</Button>` 에서 클릭 시 handleSubmit 함수를 실행시키므로 사용자가 입력한 값을 useState의 함수들을 사용해서 입력값을 데이터베이스의 키에 각각 저장
- set(newRef, newNotice);
  - firebase에 데이터를 저장하는 핵심 명령어
  - newRef는 저장할 위치, newNotice는 저장할 데이터
  - newRef에 newNotice를 저장해줘! 라고 알려줌
- setOpen(false);
  - 데이터를 저장하면서 팝업 창을 닫는 역할

---

### Firebase 데이터 실시간 수신
```useEffect(() => {
  const noticesRef = ref(realtimeDb, 'notices');
  onValue(noticesRef, snapshot => {
    const data = snapshot.val();
    if (data) {
      const list = Object.entries(data).map(([id, item]) => ({
        id,
        ...item
      }));
      list.sort((a, b) => b.isPinned - a.isPinned);
      setNotices(list);
    } else {
      setNotices([]);
    }
  });
}, []);
```

- useEffect => 창이 실행될 때 가장 먼저 한 번 실행되는 로직
- const noticesRef = ref(realtimeDb, 'notices'); => noticesRef는 데이터베이스의 notices를 참조
- onValue(noticesRef, snapshot => { => onValue가 데이터 변경을 알려주면서 현재의 데이터를 담은 객체(snapshot)을 자동으로 넘겨줌
  - onValue()는 데이터 변경을 감시하는 함수
  - const data = snapshot.val(); => 현재 시점의 실제 데이터만 꺼냄
- if절 => 원래 firebase 데이터는 객체 형태
  - 배열로 편하게 보여주기 위해서 Object.entries로 바꿔주고 각 아이템에 id를 포함시킴
  - list.sort((a, b) => b.isPinned - a.isPinned);
    - isPinned가 1(true)이면 먼저 오도록, 0(false)이면 나중에 오도록 정렬

---

### 별 클릭으로 고정 상태 바꾸기
```const togglePinned = (id, currentState) => {
  const noticeRef = ref(realtimeDb, `notices/${id}`);
  update(noticeRef, { isPinned: !currentState });
};
```

- togglePinned 함수는 id와 currentState를 요소로 가짐
- **공지사항 등록하는 팝업창 안에서 별표시 아이콘 클릭**
  - ```<IconButton onClick={() => setIsPinned(prev => !prev)}>
                  {isPinned ? <StarIcon color="primary" /> : <StarBorderIcon />}
                </IconButton>```
  - 원래 ispinned의 초기값은 false, 클릭이 되면 setIsPinned를 통해 !flase인 true가 됨 -> 위 데이터 실시간 수신으로 데이터가 동적으로 변환
  - isPinned에서 true이면 starIcon의 컬러가 노란색으로 바뀜
- **공지사항 목록에서 별표시 클릭**
  - `<IconButton onClick={() => togglePinned(notice.id, notice.isPinned)}>`
  - onClick으로 togglePinned가 실행됨 (사용자가 클릭한 별표의 공지사항에 대해서 notice레퍼런스의 id와 isPinned가 요소로 사용됨)
  - notice.id와 notice.isPinned는 그대로 (id, currentState)에 대입
  - noticeRef에 대해서 realtimeDB 내 notices 데이터베이스의 id를 참조 
  - update 함수를 통해 noticeRef의 참조 내 isPinned의 데이터를 반대로 저장(true는 false로 false는 true로!)
- 별표 클릭 시 isPinned의 값을 반전시켜줘야하기 때문에 필요한 함수

---
### 관리자 계정에 권한 설정하기
- userTable 컴포넌트 내 메뉴에 권한 설정 버튼을 추가로 넣고 부여할 권한 선택

#### permissionUtils.js
```export const hasPermission = (admin, key) => {
  return admin?.name === '박정원' || admin?.permissions?.[key];
};
```
- 특정 관리자에게 어떤 기능(key)에 대한 권한이 있는지 확인할 때 사용
- `admin?.name === '박정원'` -> 이름이 박정원이면 무조건 true
- `admin?.permissions?.[key]` -> 그 외에는 해당 key 권한이 true 인지 확인해야됨

#### AdminContext.jsx
```const AdminContext = createContext(null);

export const AdminProvider = ({ children }) => {
  const [admin, setAdmin] = useState(null);

  useEffect(() => {
    const unsubscribe = onAuthStateChanged(auth, async (user) => {
      if (user) {
        console.log("✅ 로그인된 유저 UID:", user.uid);
        const snapshot = await get(ref(realtimeDb, `admin/${user.uid}`));
        const adminData = snapshot.val();
        if (adminData) {
          setAdmin({ uid: user.uid, ...adminData });
        } else {
          setAdmin(null); // 데이터는 없지만 로그인은 되어 있는 경우
        }
      } else {
        setAdmin(null);
      }
    });

    return () => unsubscribe();
  }, []);

  return (
    <AdminContext.Provider value={admin}>
      {children}
    </AdminContext.Provider>
  );
};

export const useAdmin = () => useContext(AdminContext);
```

- `const AdminContext = createContext(null);` -> AdminContext 생성, 데이터를 저장하거나 꺼냄
- `export const AdminProvider = ({ children }) => {` -> Context에 어떤 값을 줄 지 결정하는 컴포넌트
  - `const unsubscribe = onAuthStateChanged(auth, async (user) => {` -> 사용자의 로그인 상태가 바뀔 때 마다 자동으로 호출됨
  - 로그인 상태면 해당 uid로 realtimeDB의 admin 에서 정보를 가져옴
  - 관리자데이터가 있으면 setAdmin으로 상태 저장
  - 없으면 null로 설정해 권한 없는 사용자 처리
- `<AdminContext.Provider value={admin}>` -> 하위 모든 컴포넌트에서 AdminContext를 통해 admin 데이터를 사용 가능하게 됨 

#### Permissions.js
```export const ALL_PERMISSIONS = {
  "회원 관리": true,
  "노선 추가/삭제": true,
  "노선 시간대 설정 및 관리": true,
  "노선 정류장 설정 및 관리": true,
  "날짜별 예약자 목록": true,
  "예약 통계": true,
  "예약 취소 분석": true,
  "기사별 운행 이력": true,
  "기사 계정 관리": true,
  "관리자 역할 구분": true,
  "일정 등록 및 관리": true,
  "공지사항 등록 및 관리": true
};
```
- 각 사용자에 대해서 권한을 줄 목록들을 나열 

#### useEffect 내 사용자 불러올 때 슈퍼계정 필터링하기
```const [isRoleDialogOpen, setIsRoleDialogOpen] = useState(false);
const [selectedUserForRole, setSelectedUserForRole] = useState(null);

//사용자 불러오기
useEffect(() => {
  const fetchUsers = async () => {
    const snapshot = await get(ref(realtimeDb, "admin"));
    const usersData = snapshot.val();
    const usersArray = Object.entries(usersData).map(([uid, value]) => ({
      uid,
      ...value,
    }));
    
    setAllUsers(usersArray);  // 슈퍼 계정도 포함
    setFilteredUsers(usersArray.filter(user => user.name !== "박정원"));
  };
  fetchUsers();
}, []);
```
- 기존 사용자 불러오는 로직에서 `setFilteredUsers(usersArray.filter(user => user.name !== "박정원"));` 이 변경됨
  - AllUsers는 모든 관리자 계정을 저장, 하지만 FilteredUsers은 목록에 슈퍼계정을 제외한 일반 관리자 목록을 다뤄야하기 때문에 이름이 "박정원"인 관리자를 제외한 나머지 관리자만 필터링

#### handleOpenRoleDialog 로직 (권한 설정 다이얼로그)
```const handleOpenRoleDialog = async (user) => {
  if (user.name === "박정원") {
    const userRef = ref(realtimeDb, `admin/${user.uid}/permissions`);
    await set(userRef, ALL_PERMISSIONS);
    alert("슈퍼 계정은 모든 권한이 자동 부여됩니다.");
    return;
  }
  setSelectedUserForRole(user);
  setIsRoleDialogOpen(true);
};
```
- `if (user.name === "박정원")` -> 유저 이름이 "박정원"이면 realtimeDb의 admin/${user.uid}/permissions의 데이터를 저장
  - "박정원"의 permissions 내 데이터에 모든 권한을 true로
  - "슈퍼 계정은 모든 권한이 자동 부여됩니다."라는 알림창 띄우기
  - `return;`으로 끝
- 유저 이름이 "박정원" 제외한 다른 관리자들이면?
  - selectedUserForRole -> 해당 유저 데이터로 변환
  - isRoleDialogOpen -> true로


```const handleSaveRoles = async (updatedPermissions) => {
  if (!selectedUserForRole) return;

  const userRef = ref(realtimeDb, `admin/${selectedUserForRole.uid}/permissions`);
  await set(userRef, updatedPermissions);

  alert("권한이 저장되었습니다.");
  setIsRoleDialogOpen(false);
};
```
```<RoleDialog
  open={isRoleDialogOpen}
  onClose={() => setIsRoleDialogOpen(false)}
  user={selectedUserForRole}
  onSave={handleSaveRoles}
  initialPermissions={selectedUserForRole?.permissions || {}}
/>
```
- `isRoleDialogOpen`가 true로 되면서 다이얼로그 오픈
- `if (!selectedUserForRole) return;` -> selectedUserForRole 내 데이터가 없으면 return;
- realtimeDb내 selectedUserForRole의 유저의 permissions을 가져와 userRef에 저장
- userRef에 업데이트된 권한 updatedPermissions 저장
- 권한 저장 완료 알림창을 띄운 후 isRoleDialog 상태를 false로

---

### UserTable 컴포넌트를 이용한 admin 사용자 목록 내 기능
- UserManagement.jsx`참고
```
    useEffect(() => {
      const filtered = allUsers.filter((user) => {
      const nameMatch = (user.name ?? '').toLowerCase().includes(searchKeyword.toLowerCase());
      const idWithEmail = user.id?.includes('@') ? user.id : `${user.id}@gmail.com`;
      const idMatch = idWithEmail.toLowerCase().includes(searchKeyword.toLowerCase());
      return nameMatch || idMatch;
      });
      setFilteredUsers(filtered);
    }, [searchKeyword, allUsers]);

    //context 메뉴 관련
    const handleMenuOpen = (event, user) => {
      setAnchorEl(event.currentTarget);
      setAnchorUserId(user.uid);
    };
  
    const handleMenuClose = () => {
      setAnchorEl(null);
      setAnchorUserId(null);
    };
  
    //메모 다이얼로그
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
  
      const userRef = ref(realtimeDb, `admin/${selectedUserForMemo.uid}`);
      const memoRef = ref(realtimeDb, `admin/${selectedUserForMemo.uid}/memo`);
  
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
  
  
    const handleUnban = async (uid) => {
      await update(ref(realtimeDb, `admin/${uid}`), { isBanned: false });
  
      setAllUsers((prev) =>
        prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
      );
      setFilteredUsers((prev) =>
        prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
      );
  
      handleMenuClose();
    };
  
  
    //메모 히스토리 관련
    const handleOpenMemoHistory = async (user) => {
      const memoRef = ref(realtimeDb, `admin/${user.uid}/memo`);
      const snapshot = await get(memoRef);
  
      if (snapshot.exists()) {
        const memoObject = snapshot.val();
        const memoArray = Object.values(memoObject);
  
        const sorted = memoArray.sort(
          (a, b) => dayjs(b.timestamp).valueOf() - dayjs(a.timestamp).valueOf()
        );
        setSelectedMemoList(sorted);
      } else {
        setSelectedMemoList([]);
      }
  
      setIsMemoHistoryOpen(true);
    };
  
    //사용자 정보 수정 관련
    const handleOpenEditDialog = (user) => {
      setEditingUser(user);
      setEditedEmail(user.id ?? "");
      setEditedName(user.name ?? "");
      setIsEditDialogOpen(true);
    };
  
    //비밀번호 초기화 관련
    const handleOpenReset = (user) => {
      setTargetUser(user);
      setResetEmail(user.id.includes('@') ? user.id : `${user.id}@gmail.com`);

      setIsResetOpen(true);
    };
  
    const handleSendResetEmail = async () => {
      try {
        const emailToSend = resetEmail.includes('@') ? resetEmail.trim() : `${resetEmail.trim()}@gmail.com`;
        await sendPasswordResetEmail(auth, emailToSend);

        alert("비밀번호 초기화 메일이 전송되었습니다.");
        setIsResetOpen(false);
      } catch (error) {
        alert("이메일 전송 실패: " + error.message);
      }
    };
```

    


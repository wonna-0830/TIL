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

### return 내 UI 구조
- **공통 sx**
  - Box => Material UI에서 제공하는 기본 레이아웃 박스
  - Tabs => 여러개의 버튼을 가로로 나열해서 탭을 만들어 클릭할 때마다 내용을 바꾸는 UI
  - Tab => Tabs 내의 버튼 하나하나를 나타내는 컴포넌트
  - Dialog => 팝업창 역할 (open이 true이면 열림) (DialogTitle => 팝업 제목, DialogContent => 핵심 내용)
  - TextField => 텍스트 입력창
  - py => padding y (위아래 여백), px => padding x (좌우 여백)
  - borderRadius => 모서리 둥글게
  - marginTop => 위쪽과 간격 (marginBottom ...등) => mt, mb로도 사용가능
  - boxShadow => 그림자
  - maxWidth, maxHeight, min~ => 높이, 너비 제한 
#### 세부 기능 목록 UI
```return (
    <Box sx={{ width: '100%' }}>
      <Box
        sx={{
          backgroundColor: '#fff',
          py: 1,
          px: 5,
          boxShadow: 1,}}>
        <Tabs
          value={tabIndex}
          onChange={handleTabChange}
          textColor="primary"
          indicatorColor="primary"
          variant="standard" 
          centered             
          sx={{minWidth: 'fit-content',  }}>
          <Tab label="관리자 역할 구분" />
          <Tab label="주요 기능 접근 제한" />
          <Tab label="일정 등록 및 관리" />
          <Tab label="공지사항 등록 및 관리" />
        </Tabs>
      </Box>
```

- varient => 탭 스타일 (scrollable, fullWidth 등)
- centered => 탭들을 화면 가운데로 정렬 (default => 왼쪽 정렬)
- minWidth: 'fit-content' => 탭 바의 최소 너비를 내용 크기만큼 맞춤

---

#### 각 탭 내 내용

```<Box
        sx={{
          backgroundColor: '#f5f5f5',
          borderRadius: 2,
          padding: 3,
          marginTop: 2,
          width: '100%',
          maxWidth: 'none',
        }}
      >
        <TabPanel value={tabIndex} index={0}>관리자 역할 구분</TabPanel>
        <TabPanel value={tabIndex} index={1}>주요 기능 접근 제한</TabPanel>
        <TabPanel value={tabIndex} index={2}>일정 등록 및 관리</TabPanel>
        <TabPanel value={tabIndex} index={3}>
          <Button
            variant="contained"
            startIcon={<AddIcon />}
            onClick={() => setOpen(true)}
            sx={{ mb: 2 }}
          >
            새 공지사항 등록
          </Button>
  ```

- TapPanel => 탭 번호가 ~번일 때만 안의 내용 보여줌(위의 TapPanel 컴포넌트에 서술)
- 3번 인덱스 탭 클릭 시 setOpen(true)로 아래 Dialog 오픈

---
#### 공지사항 등록 기능 팝업창

```<Dialog open={open} onClose={() => setOpen(false)}>
            <DialogTitle>공지사항 등록</DialogTitle>
            <DialogContent>
              <TextField
                label="제목"
                fullWidth
                margin="normal"
                value={title}
                onChange={e => setTitle(e.target.value)}
              />
              <TextField
                label="URL"
                fullWidth
                margin="normal"
                value={url}
                onChange={e => setUrl(e.target.value)}
              />
              <Box sx={{ display: 'flex', alignItems: 'center', mt: 2 }}>
                <IconButton onClick={() => setIsPinned(prev => !prev)}>
                  {isPinned ? <StarIcon color="primary" /> : <StarBorderIcon />}
                </IconButton>
                <Typography>{isPinned ? '대시보드에 노출됨' : '공지사항에 등록'}</Typography>
              </Box>
              <Box sx={{ mt: 2 }}>
                <Button variant="contained" onClick={handleSubmit}>등록</Button>
                <Button onClick={() => setOpen(false)} sx={{ ml: 1 }}>취소</Button>
              </Box>
            </DialogContent>
</Dialog>
```

- 

---

```<TableContainer component={Paper}>
            <Table>
              <TableHead>
                <TableRow>
                  <TableCell>제목</TableCell>
                  <TableCell>등록일</TableCell>
                  <TableCell>고정</TableCell>
                </TableRow>
              </TableHead>
              <TableBody>
                {notices.map((notice) => (
                  <TableRow key={notice.id}>
                    <TableCell>
                      <a href={notice.url} target="_blank" rel="noopener noreferrer" style={{ textDecoration: 'none', color: '#1976d2' }}>
                        {notice.title}
                      </a>
                    </TableCell>
                    <TableCell>{new Date(notice.date).toLocaleDateString()}</TableCell>
                    <TableCell>
                      <IconButton onClick={() => togglePinned(notice.id, notice.isPinned)}>
                        {notice.isPinned ? <StarIcon color="warning" /> : <StarBorderIcon />}
                      </IconButton>
                    </TableCell>
                  </TableRow>
                ))}
              </TableBody>
            </Table>
          </TableContainer>
        </TabPanel>
      </Box>
    </Box>
  ```

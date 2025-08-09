## userManagement.jsx 파헤치기
```
import React, { useState, useEffect } from 'react';
import {
  TextField, Box, Table, TableHead, TableCell, TableRow, TableBody,
  Menu, MenuItem, Dialog, DialogTitle, DialogContent, DialogActions,
  Checkbox, FormControlLabel, Button, Typography, Paper
} from '@mui/material';
import { ref, get, update, push, set } from "firebase/database";
import { realtimeDb, auth } from "../firebase";
import { sendPasswordResetEmail } from "firebase/auth";
import IconButton from "@mui/material/IconButton";
import MoreVertIcon from "@mui/icons-material/MoreVert";
import EditNoteIcon from '@mui/icons-material/EditNote';
import dayjs from "dayjs"; 
```
```
const UserManagement = () => {
  const [searchKeyword, setSearchKeyword] = useState('');
  const [filteredUsers, setFilteredUsers] = useState([]);
  const [allUsers, setAllUsers] = useState([]);

  const [anchorEl, setAnchorEl] = useState(null);
  const [anchorUserId, setAnchorUserId] = useState(null);

  const [isMemoOpen, setIsMemoOpen] = useState(false);
  const [selectedUserForMemo, setSelectedUserForMemo] = useState(null);
  const [memoText, setMemoText] = useState("");
  const [isWarning, setIsWarning] = useState(false);
  const [isBan, setIsBan] = useState(false);
  const [isMemoHistoryOpen, setIsMemoHistoryOpen] = useState(false);
  const [selectedMemoList, setSelectedMemoList] = useState([]);
  const [memoFilter, setMemoFilter] = useState("all");
  const [isEditDialogOpen, setIsEditDialogOpen] = useState(false);
  const [editingUser, setEditingUser] = useState(null);
  const [editedEmail, setEditedEmail] = useState("");
  const [editedName, setEditedName] = useState("");
  const [resetEmail, setResetEmail] = useState('');
  const [isResetOpen, setIsResetOpen] = useState(false);
  const [targetUser, setTargetUser] = useState(null);
```

## 처음 실행 시 목록 보여주기
```
  useEffect(() => {
    const fetchUsers = async () => {
      const snapshot = await get(ref(realtimeDb, "users"));
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
```
- `fetchUsers` => 비동기 데이터 가져오기
- realtimeDb의 users 데이터를 가져올 때 까지 기다린 후(await) snapshot에 저장
- users 데이터의 항목을 JS 객체로 변환 후 usersData에 저장
- `Object.entries`로 각각의 uid에 대해 데이터를 쌍으로 분리(딕셔너리처럼 변환) 후 usersArray에 저장
  - `object.keys` -> 키를 배열로, `object.values` -> 값만 배열로
- `setAllUsers(usersArray)` -> DB에서 가져온 원본 데이터를 보관
- `setFilteredUsers(usersArray)` -> allUsers를 기준으로 조건에 따라 보여줄 데이터만 담겨있음
  - ```{filteredUsers.map((user) => (
            <TableRow key={user.uid}>
              <TableCell>{user.uid}</TableCell>
              <TableCell>{user.email}</TableCell>
              <TableCell>{user.name}</TableCell>
              <TableCell>{user.joinDate}</TableCell>
              <TableCell>{user.warningCount ?? 0}</TableCell>
              <TableCell>{user.isBanned ? "정지됨" : "X"}</TableCell>
      ```

  - 구분에 보여줄 데이터들이 있는 것만 골라서 `filteredUsers.map`을 통해 보여줌


## 검색 필터링 기능
```
  useEffect(() => {
    const filtered = allUsers.filter((user) =>
      (user.name ?? '').toLowerCase().includes(searchKeyword.toLowerCase())
    );
    setFilteredUsers(filtered);
  }, [searchKeyword, allUsers]);
```
- ```<Box sx={{ width: '100%', backgroundColor: '#fff', padding: 2 }}>
      <TextField
        label="이름으로 검색"
        variant="outlined"
        size="small"
        value={searchKeyword}
        onChange={(e) => setSearchKeyword(e.target.value)}
        sx={{ width: '500px', mb: 2 }}
      />
  ```
  - 검색 UI에 입력값을 넣을 때마다 `searchKeyword` 값이 바뀜(`setSearchKeyword`로 변환)
- allUsers의 데이터 중 대소문자 상관없이(`toLowerCase()`) searchKeyword의 값이 포함되는(`includes`) 값을 필터링해서(`filter`) filtered에 담아 저장
- `setFilteredUsers`로 filtered 안 값들을 화면에 노출
- `[serachKeyword, allUsers]` => 검색창 입력값이 달라질 때, 유저 목록이 달라질 때 마다 계속 실행됨

## 메뉴 열기
```
  const handleMenuOpen = (event, user) => {
    setAnchorEl(event.currentTarget);
    setAnchorUserId(user.uid);
  };

  const handleMenuClose = () => {
    setAnchorEl(null);
    setAnchorUserId(null);
  };
```
- 아이콘 클릭 시 `handleMenuOpen` 실행
  - `setAnchorEl` -> 누른 버튼의 DOM(요소) 저장
  - `setAnchorUserId` -> 누른 버튼에 대한 유저의 uid 기억
- Menu 요소가 열림
  - `anchorEl`가 클릭된 버튼의 유저 값으로 되어있으므로 그 사용자의 위치에 메뉴를 띄우도록 함
  - 클릭한 버튼의 유저의 uid에 대해서 `filteredUsers.map()`에 렌더링된 uid가 일치하면 메뉴가 오픈됨
- 사용자가 메뉴 바깥을 클릭하거나 취소 버튼 클릭하면 `handleMenuClose` 버튼 실행
  - 메뉴를 닫고 나면 다음에 다른 유저의 버튼을 눌렀을 때 새로 열릴 수 있게 초기화 

## 메모 열기
```
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
```
- ```<TableCell>
                <IconButton onClick={() => handleOpenMemo(user)}>
                  <EditNoteIcon />
                </IconButton>
              </TableCell>
    ```
  - 메뉴 버튼 오른쪽의 메모 버튼
- `onClick`으로 `handleOpenMemo` 실행
  - `setSelectedUserForMemo`로 어떤 사용자의 메모버튼을 눌렀는지 기억
- `setIsMemoOpen`을 true 상태로 변경
  - ```<Dialog open={isMemoOpen} onClose={handleCloseMemo}>
        <DialogTitle>관리자 메모</DialogTitle>
        <DialogContent>
          <TextField
            label="메모 내용"
            multiline
            fullWidth
            value={memoText}
            onChange={(e) => setMemoText(e.target.value)}
          />
          <FormControlLabel
            control={<Checkbox checked={isWarning} onChange={(e) => setIsWarning(e.target.checked)} />}
            label="경고 부여"
          />
          <FormControlLabel
            control={<Checkbox checked={isBan} onChange={(e) => setIsBan(e.target.checked)} />}
            label="계정 정지"
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={handleSubmitMemo} color="primary">확인</Button>
          <Button onClick={handleCloseMemo} color="secondary">취소</Button>
        </DialogActions>
      </Dialog>```

  - 메모 작성 팝업이 실행됨 (`open={isMemoOpen}`)
  - 메모 작성 시 setMemoText로 MemoText에 상태가 저장됨
  - 경고 박스에 체크했다면 `setIsWarning`으로 이벤트의 상태가 IsWarning에 저장됨
  - 계정 정지 박스에 체크했다면 setIsban로 isBan 이벤트 상태가 저장됨
- 취소 버튼을 누르면 handleCloseMemo 버튼이 실행
  - `setSelectedUserForMemo(null)`로 저장된 사용자의 정보가 null
  - `setIsMemoOpen(false), setMemoText(""), setIsWarning(false), setIsBan(false)`로 메모 팝업이 닫히며 작성했던 정보가 초기화 됨
- 확인 버튼을 누르면 아래 후술할 로직인 handleSubmitMemo 실행 

## 메모 작성 후 저장하기
```  
  const handleSubmitMemo = async () => {
    if (!selectedUserForMemo) return;

    const userRef = ref(realtimeDb, `users/${selectedUserForMemo.uid}`);
    const memoRef = ref(realtimeDb, `users/${selectedUserForMemo.uid}/memo`);

    const newMemo = {
      text: memoText.trim(),
      timestamp: dayjs().format('YYYY-MM-DD'),
      writer: '관리자A',
      type: isBan ? 'ban' : isWarning ? 'warning' : 'note'
    };

    // memo는 배열 대신 push로 저장
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

    // 상태 관련 업데이트 (memo 제외)
    if (Object.keys(updates).length > 0) {
      await update(userRef, updates);
    }

    // 로컬 상태 반영
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
```
- `if (!selectedUserForMemo) return;` -> 선택된 유저 정보가 없으면 함수 실행X
- `userRef` -> realtimeDb에서 users의 데이터 내 selectedUserForMemo의 사용자 uid를 가져와서 저장
- `memoRef` -> 동일하게 uid 내 memo를 가져와서 저장 (memo가 없으면 생성)
- `newMemo` -> 어떤 정보를 저장할 지 data class 처럼 정의
- `push(memoRef, newMemo)` -> memoRef 위치에 newMemo 객체를 배열처럼 추가
- `snapshot` -> userRef의 데이터를 가져올 때까지 기다림(await)
- `existingData` -> snapshot의 데이터를 js에서 쓸 수 있도록 변환
- `updates` -> 저장할 변경사항을 담는 객체 생성
- `if (isWarning)` -> isWarning 값이 true이면 기존 warningCount에 +1
  - warningCount에 값이 없으면 디폴트값으로 0에 +1 해서 저장
- `if (isBan)` -> isBan 값이 true면 updates안 isBanned 값을 true로 저장
- `if (Object.keys(updates).length > 0)`
  - 객체의 업데이트 정보가 담긴 객체의 키 길이가 0 이상이면(업데이트할 정보가 하나 이상이라도 있으면)
  - userRef 경로의 기존 값 중에서 updates에 있는 항목만 업데이트
- ```setAllUsers((prev) =>
      prev.map((u) =>
        u.uid === selectedUserForMemo.uid ? { ...u, ...updates } : u
      )
    );
    ```
  - `prev` -> 이전 상태값
  - `prev.map((u)=> )` -> prev내의 각 유저(u)에 대해 하나씩 검사해서 새 배열 만듦
  - `u.uid === selectedUserForMemo.uid` -> selectedUserForMemo의 uid가 이전 prev의 uid와 같다면?
  - `{ ...u, ...updates } : u` -> selectedUserForMemo의 user를 찾아서 updated로 병합
  - `setFilteredUser` 도 동일하게 업데이트
- `handleMemoClose`로 메모 닫음


## 정지 해제하기(메뉴)
```
    const handleUnban = async (uid) => {
    await update(ref(realtimeDb, `users/${uid}`), { isBanned: false });

    // 상태 업데이트
    setAllUsers((prev) =>
      prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
    );
    setFilteredUsers((prev) =>
      prev.map((u) => (u.uid === uid ? { ...u, isBanned: false } : u))
    );

    handleMenuClose();
  };
```
- `<MenuItem onClick={() => handleUnban(user.uid)}>정지 해제</MenuItem>`
  - 메뉴 내 정지해제 버튼 클릭 시 실행되는 함수
- realtimeDb내 users에 대해 해당 유저의 uid 레퍼런스의 isBanned 값을 false로 update
- 메모 저장 로직처럼 `allUsers`와 `filteredUsers` 내 이전 데이터 값의 uid와 정지해제 버튼이 클릭된 유저의 uid가 동일할 때 해당 uid를 가진 유저의 isBanned 상태를 false로 업데이트
- `handleMenuClose`로 메뉴창 닫음 

## 메모 이력 열기(메뉴)
```
  const handleOpenMemoHistory = async (user) => {
    const memoRef = ref(realtimeDb, `users/${user.uid}/memo`);
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
```
- `<MenuItem onClick={() => {handleOpenMemoHistory(user);handleMenuClose();}}>메모 보기</MenuItem>`
  - 메모보기 메뉴 클릭 시 `handleOpenMemoHistory(해당 유저)`의 함수와 `handleMenuClose` 함수 실행(메뉴 팝업 닫음)
- 비동기 데이터를 가져오기 위해 `async(user) => {...}` 사용
- `memoRef` -> realtimeDb의 user 데이터 내 해당 유저의 uid에 맞는 memo를 레퍼런스로 저장
- `snapshot` -> memoRef를 가져올 때까지 기다린 후 저장
- `if (snapshot.exists())` -> snapshot 값이 존재하면(메모가 있으면)
  - memoObject 내에 snapshot를 js 객체로 변환 후 저장
  - memoArray에 memoObject의 값들을 배열로 저장
  - `memoArray.sort((a, b) => ...)` -> 배열 정렬하는 메서드
    - 양수면 a가 b보다 뒤로, 음수면 그 반대
    - `dayjs(~.timestamp).valueOf()`-> 해당 날짜를 숫자로 바꿈
    - `dayjs(b.timestamp) - dayjs(a.timestamp)` -> 값이 양수면 b가 먼저 (최신순 정렬)
  - selectedMemoList를 sorted에 저장된 순대로 정렬
  - 아니면 selectedMemoList를 default으로 설정
    - `{selectedMemoList.length === 0 ? (<Typography>메모 이력이 없습니다.</Typography>)` 의 UI로 인해 memo가 없으면 Typography로 대체
- IsMemoHistoryOpen을 true로 변경 

## 메모 이력 필터링
```
  const filteredMemoList = selectedMemoList.filter(memo => 
  memoFilter === "all" ? true : memo.type === memoFilter
  );
```
- 지금 받아오는 selectedMemoList는 위에서 서술한 `selectedMemoList(sorted)`
- ```<Box sx={{ display: 'flex', gap: 1, mb: 2, px: 3 }}>
              <Button variant={memoFilter === "all" ? "contained" : "outlined"} onClick={() => setMemoFilter("all")}>전체</Button>
              <Button variant={memoFilter === "warning" ? "contained" : "outlined"} onClick={() => setMemoFilter("warning")}>경고</Button>
              <Button variant={memoFilter === "ban" ? "contained" : "outlined"} onClick={() => setMemoFilter("ban")}>정지</Button>
      ```
  - 위에서 데이터클래스처럼 정의한 newMemo에서 체크박스에 따라 type을 다르게 해서 저장했음
- 삼항 연산자
  - `조건 ? 참일때 실행할 값 : 거짓일때 실행할 값`
  - `memoFilter은 이미 값이 all` -> true이므로 모든 메모 노출
  - 경고 버튼을 클릭해서 type이 warning이 되면 거짓일 때 실행 할 부분
    - `memo.type이 memoFilter(warning)`이 되어 경고 메모만 노출

## 회원 정보 수정(메뉴)
```
  const handleOpenEditDialog = (user) => {
    setEditingUser(user);
    setEditedEmail(user.email ?? "");
    setEditedName(user.name ?? "");
    setIsEditDialogOpen(true);
  };
```
- `<MenuItem onClick={() => { handleOpenEditDialog(user); handleMenuClose(); }}>회원 수정</MenuItem>`
  - 회원 수정 버튼 클릭 시 handleOpenEditDialog에 클릭한 버튼에 대한 user 정보를 담아서 실행 + handleMenuClose 실행(팝업창 닫힘)
- `EditingUser`를 해당 user 정보로 변환
- `EditedEmail`로 해당 user의 email값을 기억(null 이면 공란)
- `EditedName`으로 해당 user의 name 값을 기억(null 이면 공란)
- `isEditDialogOpen`을 true로 바꿈
  
- ```<Dialog open={isEditDialogOpen} onClose={() => setIsEditDialogOpen(false)}>
        <DialogTitle>회원 정보 수정</DialogTitle>
        <DialogContent>
          <TextField
            margin="dense"
            label="이메일"
            fullWidth
            value={editedEmail}
            onChange={(e) => setEditedEmail(e.target.value)}
          />
          <TextField
            margin="dense"
            label="이름"
            fullWidth
            value={editedName}
            onChange={(e) => setEditedName(e.target.value)}
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setIsEditDialogOpen(false)}>취소</Button>
          <Button onClick={async () => {
            if (!editingUser) return;
            const userRef = ref(realtimeDb, `users/${editingUser.uid}`);
            await update(userRef, {
              email: editedEmail,
              name: editedName,
            });

            // 🔄 로컬 상태도 갱신
            setAllUsers((prev) =>
              prev.map((u) => u.uid === editingUser.uid ? { ...u, email: editedEmail, name: editedName } : u)
            );
            setFilteredUsers((prev) =>
              prev.map((u) => u.uid === editingUser.uid ? { ...u, email: editedEmail, name: editedName } : u)
            );

            setIsEditDialogOpen(false);
          }} color="primary">확인</Button>
        </DialogActions>
      </Dialog>
    ```
  - 회원 수정 팝업이 뜸
- `value={editedEmail} onChange={(e) => setEditedEmail(e.target.value)` 로 editedEmail 내의 값이 바뀔 때 마다 setEditedEmail로 인해 값이 저장됨 (name도 마찬가지!!)
- 취소 버튼을 누르면 `IsEditDialogOpen이 false`가 되면서 팝업창 닫힘
- 확인 버튼을 누르면 비동기 데이터 가져오기 진행
  - `if (!editingUser) return;` -> editingUser이 없으면 로직 실행 X
  - `userRef = ref(realtimeDb, users/${editingUser.uid})` -> realtimeDb 내 수정하고 있는 유저의 uid를 레퍼런스로 저장
  - `await update(userRef, {...})` -> userRef에 있던 email과 name을 editedEmail, editedName으로 업데이트
  - `prev.map((u)=> )` -> prev내의 각 유저(u)에 대해 하나씩 검사해서 새 배열 만듦
  -` u.uid === editingUser.uid` -> editingUser의 uid가 이전 prev의 uid와 같다면?
  - `{ ...u, email: editedEmail, name: editedName } : u` -> editingUser의 user를 찾아서 email과 name을 병합
  - `setFilteredUser` 도 동일하게 업데이트
  - `IsEditDialogOpen`을 false로 -> 팝업 닫힘

## 비밀번호 초기화(메뉴)
```
  const handleOpenReset = (user) => {
    setTargetUser(user);
    setResetEmail(user.email); // 👉 디폴트 이메일
    setIsResetOpen(true);
  };
```
- `<MenuItem onClick={() => { handleOpenReset(user); handleMenuClose(); }}>비밀번호 초기화</MenuItem>`
  - 비밀번호 초기화 버튼 클릭 시 handleOpenReset함수가 해당 user의 정보를 담아 실행 + handleMenuClose로 팝업창 닫음
- `TargetUser`의 값을 해당 user의 정보로 기억
- `ResetEmail`을 해당 user의 email 값으로 저장
- `IsResetOpen`을 true로 -> 비밀번호 초기화 팝업 뜸
- ```<Dialog open={isResetOpen} onClose={() => setIsResetOpen(false)}>
        <DialogTitle>비밀번호 초기화</DialogTitle>
        <DialogContent>
          <TextField
            fullWidth
            label="이메일"
            value={resetEmail}
            onChange={(e) => setResetEmail(e.target.value)}
          />
          <Typography variant="body2" color="textSecondary" sx={{ mt: 1 }}>
            기본 이메일이 입력되어 있으며, 필요 시 변경 가능합니다.
          </Typography>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleSendResetEmail} color="primary">확인</Button>
          <Button onClick={() => setIsResetOpen(false)} color="secondary">취소</Button>
        </DialogActions>
      </Dialog>
    ```
  - `value={resetEmail} onChange={(e) => setResetEmail(e.target.value)`로 resetEmail의 값 변경 가능
  - 취소 버튼 클릭 시 `IsResetOpen`을 false로 -> 팝업창 닫힘
  - 확인버튼 클릭 시 `handleSendResetEmail` 실행

```
  const handleSendResetEmail = async () => {
    try {
      await sendPasswordResetEmail(auth, resetEmail.trim());
      alert("비밀번호 초기화 메일이 전송되었습니다.");
      setIsResetOpen(false);
    } catch (error) {
      alert("이메일 전송 실패: " + error.message);
    }
  };
```
- 비동기식 데이터 가져오기 진행
- auth에 resetEmail과 동일한 이메일이 있는지 확인 후 메일 전송(`sendPasswordResetEmail`)
- 메일 전송 알림 발송
- `IsResetOpen`이 false -> 팝업창 닫힘
- error시 -> 메일 전송 실패 알림 발송 

```
  return (
    <Box sx={{ width: '100%', backgroundColor: '#fff', padding: 2 }}>
      <TextField
        label="이름으로 검색"
        variant="outlined"
        size="small"
        value={searchKeyword}
        onChange={(e) => setSearchKeyword(e.target.value)}
        sx={{ width: '500px', mb: 2 }}
      />

      <Table>
        <TableHead>
          <TableRow>
            <TableCell>uid</TableCell>
            <TableCell>아이디</TableCell>
            <TableCell>이름</TableCell>
            <TableCell>가입 날짜</TableCell>
            <TableCell>경고 횟수</TableCell>
            <TableCell>정지</TableCell>
            <TableCell>작업</TableCell>
            <TableCell>메모</TableCell>
          </TableRow>
        </TableHead>
        <TableBody>
          {filteredUsers.map((user) => (
            <TableRow key={user.uid}>
              <TableCell>{user.uid}</TableCell>
              <TableCell>{user.email}</TableCell>
              <TableCell>{user.name}</TableCell>
              <TableCell>{user.joinDate}</TableCell>
              <TableCell>{user.warningCount ?? 0}</TableCell>
              <TableCell>{user.isBanned ? "정지됨" : "X"}</TableCell>

              <TableCell>
                <IconButton onClick={(e) => handleMenuOpen(e, user)}>
                  <MoreVertIcon />
                </IconButton>
                <Menu
                  anchorEl={anchorEl}
                  open={anchorUserId === user.uid}
                  onClose={handleMenuClose}
                >
                  <MenuItem onClick={() => handleUnban(user.uid)}>정지 해제</MenuItem>
                  <MenuItem onClick={() => { handleOpenReset(user); handleMenuClose(); }}>비밀번호 초기화</MenuItem>
                  <MenuItem onClick={() => { handleOpenEditDialog(user); handleMenuClose(); }}>회원 수정</MenuItem>
                  <MenuItem onClick={() => {handleOpenMemoHistory(user);handleMenuClose();}}>메모 보기</MenuItem>
                </Menu>
              </TableCell>

              <TableCell>
                <IconButton onClick={() => handleOpenMemo(user)}>
                  <EditNoteIcon />
                </IconButton>
              </TableCell>
            </TableRow>
          ))}
        </TableBody>
      </Table>

      <Dialog open={isMemoOpen} onClose={handleCloseMemo}>
        <DialogTitle>관리자 메모</DialogTitle>
        <DialogContent>
          <TextField
            label="메모 내용"
            multiline
            fullWidth
            value={memoText}
            onChange={(e) => setMemoText(e.target.value)}
          />
          <FormControlLabel
            control={<Checkbox checked={isWarning} onChange={(e) => setIsWarning(e.target.checked)} />}
            label="경고 부여"
          />
          <FormControlLabel
            control={<Checkbox checked={isBan} onChange={(e) => setIsBan(e.target.checked)} />}
            label="계정 정지"
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={handleSubmitMemo} color="primary">확인</Button>
          <Button onClick={handleCloseMemo} color="secondary">취소</Button>
        </DialogActions>
      </Dialog>

      <Dialog open={isMemoHistoryOpen} onClose={() => setIsMemoHistoryOpen(false)} maxWidth="sm" fullWidth>
        <DialogTitle>📋 메모 이력</DialogTitle>
            <Box sx={{ display: 'flex', gap: 1, mb: 2, px: 3 }}>
              <Button variant={memoFilter === "all" ? "contained" : "outlined"} onClick={() => setMemoFilter("all")}>전체</Button>
              <Button variant={memoFilter === "warning" ? "contained" : "outlined"} onClick={() => setMemoFilter("warning")}>경고</Button>
              <Button variant={memoFilter === "ban" ? "contained" : "outlined"} onClick={() => setMemoFilter("ban")}>정지</Button>
            </Box>
        <DialogContent>
          {selectedMemoList.length === 0 ? (
            <Typography>메모 이력이 없습니다.</Typography>
          ) : (
            <Box>
              {filteredMemoList.map((memo, index) => (
                <Box key={index}>
                  <Typography variant="body2" sx={{ mb: 1 }}>
                    {memo.timestamp} / {memo.writer}
                  </Typography>
                  <Paper sx={{ p: 1, mb: 2 }}>
                    <Typography>{memo.text}</Typography>
                  </Paper>
                </Box>
              ))}
            </Box>
          )}
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setIsMemoHistoryOpen(false)}>닫기</Button>
        </DialogActions>
      </Dialog>
      <Dialog open={isEditDialogOpen} onClose={() => setIsEditDialogOpen(false)}>
        <DialogTitle>회원 정보 수정</DialogTitle>
        <DialogContent>
          <TextField
            margin="dense"
            label="이메일"
            fullWidth
            value={editedEmail}
            onChange={(e) => setEditedEmail(e.target.value)}
          />
          <TextField
            margin="dense"
            label="이름"
            fullWidth
            value={editedName}
            onChange={(e) => setEditedName(e.target.value)}
          />
        </DialogContent>
        <DialogActions>
          <Button onClick={() => setIsEditDialogOpen(false)}>취소</Button>
          <Button onClick={async () => {
            if (!editingUser) return;
            const userRef = ref(realtimeDb, `users/${editingUser.uid}`);
            await update(userRef, {
              email: editedEmail,
              name: editedName,
            });

            // 🔄 로컬 상태도 갱신
            setAllUsers((prev) =>
              prev.map((u) => u.uid === editingUser.uid ? { ...u, email: editedEmail, name: editedName } : u)
            );
            setFilteredUsers((prev) =>
              prev.map((u) => u.uid === editingUser.uid ? { ...u, email: editedEmail, name: editedName } : u)
            );

            setIsEditDialogOpen(false);
          }} color="primary">확인</Button>
        </DialogActions>
      </Dialog>
      <Dialog open={isResetOpen} onClose={() => setIsResetOpen(false)}>
        <DialogTitle>비밀번호 초기화</DialogTitle>
        <DialogContent>
          <TextField
            fullWidth
            label="이메일"
            value={resetEmail}
            onChange={(e) => setResetEmail(e.target.value)}
          />
          <Typography variant="body2" color="textSecondary" sx={{ mt: 1 }}>
            기본 이메일이 입력되어 있으며, 필요 시 변경 가능합니다.
          </Typography>
        </DialogContent>
        <DialogActions>
          <Button onClick={handleSendResetEmail} color="primary">확인</Button>
          <Button onClick={() => setIsResetOpen(false)} color="secondary">취소</Button>
        </DialogActions>
      </Dialog>
    </Box>
  );
};
export default UserManagement;
```

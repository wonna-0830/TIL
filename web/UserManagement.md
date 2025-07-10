## userManagement.jsx 파헤치기
```
import React, { useState, useEffect } from 'react';
import { TextField, Box, Table, TableHead, TableCell, TableRow, TableBody } from '@mui/material';
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getDatabase } from "firebase/database";
```
```
const UserManagement = () => {
  const [searchKeyword, setSearchKeyword] = useState('')
  const [filteredUsers, setFilteredUsers] = useState([]);
  const [allUsers, setAllUsers] = useState([]);

  useEffect(() => {
    const fetchUsers = async () => {
      const snapshot = await get(ref(db, "users")); // 예시
      const usersData = snapshot.val();
      const usersArray = Object.entries(usersData).map(([uid, value]) => ({
        uid,
        ...value,
      }));
      setAllUsers(usersArray);
      setFilteredUsers(usersArray); // 초기엔 전체 목록 표시
    };

    fetchUsers();
  }, []);
```
```
  useEffect(() => {
    const filtered = allUsers.filter((user) =>
      user.name.toLowerCase().includes(searchKeyword.toLowerCase())
    );
    setFilteredUsers(filtered);
  }, [searchKeyword, allUsers]);
```
```
  return (
  <Box sx={{ width: '100%', backgroundColor: '#fff', padding: 2, boxSizing: 'border-box' }}>
    <TextField
      label="이름으로 검색"
      variant="outlined"
      size="small"
      value={searchKeyword}
      onChange={(e) => setSearchKeyword(e.target.value)}
      sx={{ width: '500px', mb: 2 }} // 아래 마진 추가
    />

    <Table>
      <TableHead>
        <TableRow>
          <TableCell>아이디</TableCell>
          <TableCell>이름</TableCell>
          <TableCell>가입 날짜</TableCell>
          <TableCell>경고 횟수</TableCell>
          <TableCell>작업</TableCell>
        </TableRow>
      </TableHead>
      <TableBody>
        {filteredUsers.map((user) => (
          <TableRow key={user.uid}>
            <TableCell>{user.uid}</TableCell>
            <TableCell>{user.name}</TableCell>
            <TableCell>{user.createdAt}</TableCell>
            <TableCell>{user.warningCount ?? 0}</TableCell>
            <TableCell>{user.isBanned ? "정지됨" : "정상"}</TableCell>
            <TableCell>
              <IconButton><MoreVertIcon /></IconButton>
            </TableCell>
            <TableCell>
              <IconButton>
                <MoreVertIcon />
              </IconButton>
            </TableCell>
          </TableRow>
        ))}
      </TableBody>
    </Table>
  </Box>

)
}
export default UserManagement;
```

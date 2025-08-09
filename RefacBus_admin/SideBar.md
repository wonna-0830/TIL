### SideBar.jsx 컴포넌트와 Home.jsx 페이지의 연결 (feat.App.jsx)

### 사이드바 코드

#### UI 기본 설명
- <Drawer ...> -> 사이드바처럼 옆에 고정되는 박스 (이 안에 메뉴 버튼이 들어감)
- <ListItemButton> -> 메뉴 각각의 버튼
```
const Sidebar = ({ onMenuSelect }) => {
  return (
    <Drawer
      sx={{
        width: drawerWidth,
        flexShrink: 0,
        '& .MuiDrawer-paper': {
          width: drawerWidth,
          boxSizing: 'border-box',
          backgroundColor: '#002857', // 대구가톨릭대 메인 컬러
          color: 'white',
        },
      }}
      variant="permanent"
      anchor="left"
    >
      <Toolbar
        onClick={() => onMenuSelect('home')}
        sx={{
          cursor: 'pointer',
          '&:hover': { opacity: 0.8 },
        }}
      >
        <Typography variant="h6" noWrap component="div">
          DCU 스쿨버스 관리자
        </Typography>
      </Toolbar>
      <Divider sx={{ backgroundColor: '#ffffff33' }} />
      <List>
        <ListItemButton  onClick={() => onMenuSelect('user')}>
          <ListItemIcon sx={{ color: 'white' }}><UserIcon /></ListItemIcon>
          <ListItemText primary="회원 관리" />
        </ListItemButton>
        <ListItemButton onClick={() => onMenuSelect('placetime')}>
          <ListItemIcon sx={{ color: 'white' }}><PlaceTimeIcon /></ListItemIcon>
          <ListItemText primary="노선/시간 관리" />
        </ListItemButton>
        <ListItemButton onClick={() => onMenuSelect('reservation')}>
          <ListItemIcon sx={{ color: 'white' }}><ReservationsIcon /></ListItemIcon>
          <ListItemText primary="예약 현황/통계" />
        </ListItemButton>
        <ListItemButton onClick={() => onMenuSelect('drivenote')}>
          <ListItemIcon sx={{ color: 'white' }}><DriveNoteIcon /></ListItemIcon>
          <ListItemText primary="운행 기록 확인" />
        </ListItemButton>
        <ListItemButton onClick={() => onMenuSelect('managermanage')}>
          <ListItemIcon sx={{ color: 'white' }}><ManagerIcon /></ListItemIcon>
          <ListItemText primary="관리자 계정 관리" />
        </ListItemButton>
      </List>
      <Divider sx={{ backgroundColor: '#ffffff33' }} />
      <List>
        <ListItemButton>
          <ListItemIcon sx={{ color: 'white' }}><LogoutIcon /></ListItemIcon>
          <ListItemText primary="로그아웃" />
        </ListItemButton>
      </List>
    </Drawer>
  );
};
```
### Home.jsx 일부
```
import Sidebar from '../components/Sidebar';
```
```
<Box sx={{ width: 200 }}>
        <Sidebar onMenuSelect={setSelectedMenu} />
      </Box> -> 사이드바

      <Box
        sx={{...}}
      >
      {renderContent()}
      </Box> -> 대시보드
```
```
const [selectedMenu, setSelectedMenu] = useState('home');

  const renderContent = () => {
    switch (selectedMenu) {
      case 'home':
        return <DashBoard />;
      case 'user':
        return <UserManagement />;
      case 'placetime':
        return <PlaceTimeManagement/> ;
      case 'reservation':
        return <ReservationManagement/> ;
      case 'drivenote':
        return <DriverManagement/> ;
      case 'managermanage':
        return <ManagerManagement/> ;
      // 필요한 경우 driver 등 다른 것도 추가
      default:
        return <div>기능을 선택해주세요</div>;
    }
  };
```
#### 로직 설명
- 지금 Sidebar.jsx에 있는 onMenuSelect 함수는 Home.jsx(부모가 만든)에서 props로 전달받은 함수 `<Sidebar onMenuSelect={setSelectedMenu} />`
  - props는 부모 컴포넌트에서 자식 컴포넌트로 정보를 전달하는 방법
  - `const Sidebar = (props) => {props.onMenuSelect('');}` 이렇게도 쓰지만 비구조화 할당으로 `const Sidebar = ({ onMenuSelect }) => {onMenuSelect('');}` 이렇게 많이 씀
- 각각의 목록이 클릭되면 `onClick={() => onMenuSelect('')` 으로 onMenuSelect('') 함수를 실행해달라고 props로 전달함
  - Sidebar.jsx 내 onMenuSelect 로직이 없는 이유
    - 메뉴 목록 클릭 시 진짜 기능은 Home.jsx에서 보여줄 거기 때문에 Home.jsx에서 로직 생성
---
- Home.jsx에 전달된 onMenuSelect('')는 setSelectedMenu와 연결되어있으므로 selectedMenu의 값을 변경 (초기값으로는 'home'화면을 보여줌) 
- `<Box{renderContent()}</Box>` 대시보드 부분에서는 sx 속성 제외 renderContent()를 보여달라고 하고있음
- renderContent 함수는 switch문으로 인해 selectMenu의 값에 따라 return 값이 달라짐. Sidebar.jsx에서 보낸 값에 따라 다르게 페이지 노출

---
#### 페이지간의 연결 (App.jsx)
```
import Login from "./pages/Login"
import Home from "./pages/Home"
import Register from "./pages/Register"
import UserManagement from './pages/UserManagement';
import PlaceTimeManagement from './pages/PlaceTimeManagement';
import ReservationManagement from './pages/ReservationManagement';
import DriverManagement from './pages/DriverManagement.jsx';
import ManagerManagement from './pages/ManagerManagement.jsx';
import DashBoard from './pages/DashBoard.jsx';
```
```
<BrowserRouter>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/Register" element={<Register />}/>
        <Route path="/Home" element={<Home />} />
        <Route path="/user-management" element={<UserManagement />} />
        <Route path="/placetime-management" element={<PlaceTimeManagement />} />
        <Route path="/reservation-management" element={<ReservationManagement />} />
        <Route path="/driver-management" element={<DriverManagement />} />
        <Route path="/manager-management" element={<ManagerManagement />} />
        <Route path="/dashboard" element={<DashBoard />} />
      </Routes>
    </BrowserRouter>
```
- BrowserRouter는 만든 여러 페이지들을 url에 따라 화면을 전환할 수 있게하는 라우팅 기능
- Routes는 여러개의 Route를 감싸는 라우팅 그룹
- `path = "/주소"` 는 어떤 경로로 접근할 때 `element = {<컴포넌트>}` 어떤 컴포넌트를 보여줄 지 정해줌
  - path는 로직 안에서도 navigate()속성을 이용해서 페이지 간 전환을 이용할 때 씀
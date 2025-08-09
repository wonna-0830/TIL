## 지금까지 했던 웹 페이지 제작기 중 Login.jsx 파일 내 코드들 공부 (2025.07.08)
### 기본 import문 일부와 useState는 서술 X (ManagerManagement.jsx 참고)

### import문
```
import { useNavigate } from "react-router-dom";
import { signInWithEmailAndPassword} from "firebase/auth";
```

- react-router-dom을 이용해 페이지 간 이동가능한 모듈 import
- firebase/auth 기능 중 로그인 기능에 필요한 모듈 import

### 패스워드 노출 로직
```
let passwordTimeoutRef = useRef(null);
  
    const togglePasswordVisibility = () => {
    setShowPassword(true);
  
    // 이전 타이머 제거
    if (passwordTimeoutRef.current) {
      clearTimeout(passwordTimeoutRef.current);
    }
  
    // 1초 뒤에 다시 가리기
    passwordTimeoutRef.current = setTimeout(() => {
      setShowPassword(false);
    }, 1000);
  };
```
- let passwordTimeoutRef = useRef(null); => passwordTimeoutRef라는 useRef를 선언
  - useRef는 값을 저장하지만 컴포넌트가 다시 렌더링 될 때 값이 초기화되지 않음 (useState랑 반대), 지역변수랑 비슷한 개념
- 👁 버튼을 누르면 onClick={togglePasswordVisibility} 실행
- 실행된 togglePasswordVisivility는 setShowPassword를 true 상태로 만듦
- if문 -> passwordTimeoutRef가 현재 null이 아니면 clearTimeout을 사용해 초기화
- passwordTimeoutRef.current에 새로운 타이머 설정 1초(1000)뒤에 다시 setShowPassword(false)로 비밀번호 가리기
- setTimeout과 clearTimeout은 JS에서 기본적으로 제공하는 함수
  - setTimeout() => 일정시간이 지난 후 함수를 한 번 실행
  - clearTimeout() => 예약한 타이머를 취소

---

### useEffect
```
useEffect(() => {
    const savedId = localStorage.getItem("savedId");
    if (savedId) {
      setId(savedId);
      setRememberId(true);
    }
  }, []);
```
- localStorage 내 savedId를 가져옴 (앱으로 치면 sharedReference)
- savedId가 있으면 ID 작성란에 savedId를 자동으로 반환
- 아이디 기억하기 체크박스도 자동으로 체크

---

### 로그인 로직
```
const handleLogin = async () => {
    if(!id || !password) {
        alert("이메일과 비밀번호를 입력해주세요.")
        return
    }
    

    const fullEmail = `${id}@example.com`

    try{
        const userCredential = await signInWithEmailAndPassword(auth, fullEmail, password)
        const uid = userCredential.user.uid;

        if (rememberId) {
             localStorage.setItem("savedId", id);
        } else {
            localStorage.removeItem("savedId");
        }
        
        const snapshot = await get(ref(realtimeDb, `admin/${uid}`))
        if (snapshot.exists()){
            const userData = snapshot.val()
            console.log("환영합니다,", userData.name)
            navigate("/Home");
        } else {
            alert("관리자 정보가 존재하지 않습니다.")
        }
    } catch (error) {
        if (error.code === "auth/user-not-found"){
            alert("존재하지 않는 계정입니다.")
        } else if (error.code === "auth/wrong-password"){
            alert("비밀번호를 다시 확인해주세요.")
        } else {
            alert("로그인에 실패했습니다.")
            console.error(error)
        }
    }
  };
```

- 로그인 버튼의 onClick={handleLogin} 를 통해 로직 실행 
  - 이때 id와 password에 값이 들어있음 (사용자가 입력한 정보)
- if문 => 만약 아이디나 비밀번호란 둘 중에 하나라도 공란이면 알림창 띄우기
- 현재 Id만 사용하고있지만 auth는 이메일만 사용하기 때문에  아이디@example.com로 변경
- signInWithEmailAndPassword 를 통해 auth의 fullEmail과 password로 로그인 결과 객체를 userCredential에 리턴해줌
  - 이 함수가 알아서 로그인 정보는 서버로 전송해서 존재하는지 확인까지 다 해줌
  - await 명령어를 통해서 userCredential에 리턴될 때까지 아래 코드 실행하지 않고 기다림
  - 리턴된 userCredential 객체 안에는 user가 있고, 그 안에 uid, email 등 사용자 정보가 들어 있음
- uid 변수에 userCredential내 user안의 uid를 가져옴
- 두번째 if문 => rememberId가 true면 localStorage의 savedId라는 이름으로 id를 저장
  - 아니면 localStorage의 savedId의 내용을 삭제
- snapshot 변수에 realtimeDb내 admin 데이터베이스의 uid를 가져옴
  - snapshot이 존재하면 userData에 snapshot의 실제 값을 꺼냄
  - 로그인 성공 확인을 위해 log를 띄움(여기서 userData 사용)
  - 홈 화면으로 이동
  - else 아니면 알림창 띄움
- catch를 통해서 알림창으로 뜨는 에러코드를 사용자 친화적으로 텍스트 변경
- 도식화
  - ```
  [사용자 입력] → [빈칸 체크] → [이메일 형식 만들기]  
→ [Firebase 로그인 요청] → [로그인 성공 시 uid 확인]  
→ [RealtimeDB에서 관리자 데이터 조회]  
→ (O) 데이터 있음 → 홈으로 이동  
→ (X) 데이터 없음 → 경고창
```
  

---

### 회원가입 버튼 로직
```
const handleRegister = () => {
    navigate("/Register");
  };
```

- 회원가입 버튼을 누르면 회원가입 페이지로 이동

---

### 엔터키 클릭 시 로그인 로직 수행
```
const handleKeyDown = (e) => {
  if (e.key === 'Enter') {
    handleLogin(); // 엔터 누르면 로그인 함수 실행
  }
};
```

- e => 변수이고, 이벤트가 발생한 요소에 대한 정보를 담고 있는 객체!
- e.key는 사용자가 누른 키 이름 => 엔터키일 때 (setOnKeyDownListener 같은 느낌)
- handleLogin()실행
- e는 변수인데 어떻게 이벤트 객체를 받아올 수 있는지? 챗지피티한테 물어봄
  - 아이디, 비밀번호 내 input에 onKeyDown={handleKeyDown}가 들어있음
  - onKeyDown은 내가 키보드 중 어느 한곳(아무거나) 눌렀을 때 수행됨
  - 내가 키보드를 눌렀을 때 handleKeyDown 로직이 수행됨과 동시에 내가 누른 키의 정보가 e로 자동으로 넘어감
  - 그다음 내가 누른 키의 정보 중 어떤키를 눌렀는지에 대한 정보인 e.key와 Enter가 맞는지 비교 후 다음 로직 수행 (e안에는 key, target, type등등 많은 정보가 들어있음)
  - e에 대한 추가적인 설명
    - ```<input type="checkbox" checked={rememberId} onChange={(e) => setRememberId(e.target.checked)}/>```
    - 아이디 기억하기 체크박스 UI내 코드임
    - 여기서 target은 input 태그 전체 => 여기 안에는 type, checked, onchange의 정보가 들어있음
    - 여기서 useState에 rememberId는 초기값이 false
    - onChange가 실행되었다는 건 체크박스를 클릭했다는 것 (= rememberId(true))
    - 그럼 이 이벤트 변수 e의 input태그 내 checked 속성을 setRememberId를 이용해 true로 반환
    - 실제 화면 상태 -> UI로 반영 (checked), UI변화 -> 상태로 반영 (onChange) 양방향으로 연결

---

### UI
```
return (
    <div className="max-w-2xl mx-auto p-10">
      <div className="bg-white shadow-md rounded p-9">
        <h2 className="text-2xl font-bold mb-3 text-balance text-black">관리자 로그인</h2>
        <p className="text-sm font-thin mb-5 text-black text-balance">관리자만 로그인 가능합니다.</p>

        <input

          type="text"
          placeholder="아이디"
          className="w-full mb-4 p-2 border rounded"
          value={id}
          onChange={(e) => setId(e.target.value)}
          onKeyDown={handleKeyDown}
        />

        <div style={{ position: "relative" }}>
            <input
                type={showPassword ? "text" : "password"}
                value={password}
                className="w-full mb-4 p-2 border rounded"
                onChange={(e) => setPassword(e.target.value)}
                placeholder="비밀번호"
                onKeyDown={handleKeyDown}
            />
            <button
                type="button"
                onClick={togglePasswordVisibility}
                style={{
                position: "absolute",
                right: "10px",
                top: "35%",
                transform: "translateY(-50%)"
                }}
            >
                👁
            </button>
        </div>
    

        <button
          onClick={handleLogin}
          className="w-full bg-blue-500 text-white py-2 mb-4 rounded hover:bg-blue-600"
        >
          로그인
        </button>

        <button
          onClick={handleRegister}
          className="w-full bg-slate-400 text-white py-2 mb-4 rounded hover:bg-slate-500"
        >
          회원가입
        </button>
        <label style={{color: "black"}}>
             <input
                type="checkbox"
                checked={rememberId}
                onChange={(e) => setRememberId(e.target.checked)}
            />
          아이디 기억하기
        </label>
      </div>
    </div>
  );
```

- 

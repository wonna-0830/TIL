# LoginActivity.kt

## 1) 역할
- 로그인 화면 UI 초기화 및 이벤트 처리
- ViewModel로부터 로그인 결과를 관찰하여 UI 상태 업데이트
- 로그인 성공 시 자동 로그인 정보 저장 및 다음 화면으로 이동
- 회원가입 버튼 클릭 시 회원가입 화면으로 전환

---

## 2) 핵심 로직 요약
- **버튼 이벤트**
  - `btnLogin.setOnClickListener {` -> 이메일, 비밀번호 공란 체크 -> 로딩바 표시, 로그인 요청
  - `btnRegister.setOnClickListener {` -> RegisterActivity로 이동 후 현재 액티비티 종료
- **로그인 상태 관찰** -> `loginState.collectLatest` 로 로딩바 숨기고 성공/실패 처리
- **로그인 성공 시** -> UID 확인, 자동 로그인 정보 저장, 토스트 메세지 출력 후 RouteTimeActivity로 이동 후 종료
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class LoginActivity : AppCompatActivity() {

    private val viewModel: LoginViewModel by viewModels {
        object : androidx.lifecycle.ViewModelProvider.Factory {
            override fun <T : androidx.lifecycle.ViewModel> create(modelClass: Class<T>): T {
                return LoginViewModel(UserRepository()) as T
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        val etEmail: EditText = findViewById(R.id.et_email)
        val etPwd: EditText = findViewById(R.id.et_pwd)
        val btnLogin: Button = findViewById(R.id.btn_login)
        val autoLogin: CheckBox = findViewById(R.id.autoLogin)
        val progressBar: ProgressBar = findViewById(R.id.progressBar)

        //버튼 이벤트
        btnLogin.setOnClickListener {
            val email = etEmail.text.toString()
            val pwd = etPwd.text.toString()

            if (email.isBlank() || pwd.isBlank()) {
                Toast.makeText(this, "이메일과 비밀번호를 입력해주세요.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }

            viewModel.login(email, pwd)
        }

        val btnRegister: Button = findViewById(R.id.btn_register)

        btnRegister.setOnClickListener {
            val intent = Intent(this, RegisterActivity::class.java)
            startActivity(intent)
        }

        //로그인 상태 관찰
        lifecycleScope.launchWhenStarted {
            viewModel.loginState.collectLatest { state ->
                when (state) {
                    is LoginUiState.Loading -> progressBar.visibility = ProgressBar.VISIBLE
                    is LoginUiState.Success -> {
                        progressBar.visibility = ProgressBar.GONE
                        // 자동 로그인 저장
                        getSharedPreferences("MyApp", MODE_PRIVATE).edit {
                            putBoolean("autoLogin", autoLogin.isChecked)
                            putString("userEmail", etEmail.text.toString())
                            putString("userPassword", etPwd.text.toString())
                        }
                        Toast.makeText(this@LoginActivity, "로그인 성공!", Toast.LENGTH_SHORT).show()
                        startActivity(Intent(this@LoginActivity, RouteTimeActivity::class.java))
                        finish()
                    }
                    is LoginUiState.Error -> {
                        progressBar.visibility = ProgressBar.GONE
                        Toast.makeText(this@LoginActivity, "실패: ${state.message}", Toast.LENGTH_SHORT).show()
                    }
                    else -> Unit
                }
            }
        }
    }
}
```
</details>
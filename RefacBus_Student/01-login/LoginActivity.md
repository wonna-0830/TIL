# LoginActivity.kt

## 1) 역할
- 로그인 화면 UI 초기화 및 이벤트 처리
- ViewModel로부터 로그인 결과를 관찰하여 UI 상태 업데이트
- 로그인 성공 시 자동 로그인 정보 저장 및 다음 화면으로 이동
- 회원가입 버튼 클릭 시 회원가입 화면으로 전환
- 뒤로가기 버튼 두번 클릭 시 앱 종료

---

## 2) 핵심 로직 요약
- 버튼 이벤트
  - `loginButton.setOnClickListener {` -> 이메일, 비밀번호 공란 체크 -> 로딩바 표시, 로그인 요청
  - `registerButton.setOnClickListener {` -> RegisterActivity로 이동 후 현재 액티비티 종료
- 로그인 상태 관찰 -> `loginResult.observe` 로 로딩바 숨기고 성공/실패 처리
- 로그인 성공 시 -> UID 확인, 자동 로그인 정보 저장, 토스트 메세지 출력 후 RouteChooseActivity로 이동 후 종료
- 뒤로가기 두 번 클릭 시 종료 로직 -> 2초 내로 뒤로가기 버튼 두번 누르면 앱 종료
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class LoginActivity : AppCompatActivity() {
    private lateinit var viewModel: LoginViewModel
    private lateinit var emailInput: EditText
    private lateinit var passwordInput: EditText
    private lateinit var loginButton: Button
    private lateinit var registerButton: Button
    private lateinit var progressBar: ProgressBar
    private lateinit var autoLoginCheckBox: CheckBox

    private var doubleBackToExitPressedOnce = false

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        viewModel = ViewModelProvider(this)[LoginViewModel::class.java]

        // UI 초기화
        emailInput = findViewById(R.id.school_num_input)
        passwordInput = findViewById(R.id.number_input)
        loginButton = findViewById(R.id.btn_click)
        registerButton = findViewById(R.id.btn_JOIN)
        progressBar = findViewById(R.id.progressBar)
        autoLoginCheckBox = findViewById(R.id.autoLogin)

        // 버튼 이벤트
        loginButton.setOnClickListener {
            val email = emailInput.text.toString()
            val password = passwordInput.text.toString()
            if (email.isEmpty() || password.isEmpty()) {
                Toast.makeText(this, "이메일과 비밀번호를 입력해주세요.", Toast.LENGTH_SHORT).show()
                return@setOnClickListener
            }
            progressBar.visibility = View.VISIBLE
            viewModel.login(email, password)
        }

        registerButton.setOnClickListener {
            startActivity(Intent(this, RegisterActivity::class.java))
            finish()
        }

        // 로그인 상태 관찰
        viewModel.loginResult.observe(this) { result ->
            progressBar.visibility = View.GONE
            if (result.isSuccess) {
                val uid = result.getOrNull() ?: return@observe
                saveAutoLogin(emailInput.text.toString(), passwordInput.text.toString())
                Toast.makeText(this, "로그인 성공", Toast.LENGTH_SHORT).show()
                startActivity(Intent(this, RouteChooseActivity::class.java))
                finish()
            } else {
                Toast.makeText(this, result.exceptionOrNull()?.message ?: "로그인 실패", Toast.LENGTH_SHORT).show()
            }
        }


        //뒤로가기 두 번 클릭 시 종료 로직 
        onBackPressedDispatcher.addCallback(this, object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                if (doubleBackToExitPressedOnce) {
                    finishAffinity()
                    return
                }

                doubleBackToExitPressedOnce = true
                Toast.makeText(this@LoginActivity, "한 번 더 누르면 종료됩니다.", Toast.LENGTH_SHORT).show()

                android.os.Handler(mainLooper).postDelayed({
                    doubleBackToExitPressedOnce = false
                }, 2000)
            }
        })
    }

    //자동 로그인 로직
    private fun saveAutoLogin(email: String, password: String) {
        val sharedPref = getSharedPreferences("MyApp", MODE_PRIVATE)
        with(sharedPref.edit()) {
            putBoolean("autoLogin", autoLoginCheckBox.isChecked)
            putString("userEmail", email)
            putString("userPassword", password)
            apply()
        }
    }
}
```
<details>



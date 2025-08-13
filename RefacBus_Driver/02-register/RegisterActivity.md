# RegisterActivity.kt

## 1) 역할
- 회원가입 화면 UI 초기화 및 이벤트 처리
- ViewModel로부터 회원가입 결과를 관찰하여 UI 상태 업데이트
- 회원가입 성공 시 다음 화면으로 이동
- **먼저 만들었던 학생용 앱 로직에서 업그레이드 시킨 로직입니다**

---

## 2) 핵심 로직 요약
- **Activity → ViewModel 주입** : 의존성 주입 방식으로 `UserRepository`를 생성자 인자로 전달
- **가입 버튼 클릭 → ViewModel 호출** : 입력값을 받아 ViewModel에 전달
- **Activity – 상태 구독 & UI 업데이트** : 상태 변화에 따라 ProgressBar, Toast, 화면 전환 등을 처리.
  
<details>
<summary> 코드 보기 </summary>

```kotlin
class RegisterActivity : AppCompatActivity() {

    //Activity → ViewModel 주입
    private val viewModel: RegisterViewModel by viewModels {
        object : androidx.lifecycle.ViewModelProvider.Factory {
            override fun <T : androidx.lifecycle.ViewModel> create(modelClass: Class<T>): T {
                return RegisterViewModel(UserRepository()) as T
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_register)

        val etEmail = findViewById<EditText>(R.id.et_email)
        val etPwd = findViewById<EditText>(R.id.et_pwd)
        val etName = findViewById<EditText>(R.id.et_name)
        val btnRegister = findViewById<Button>(R.id.btn_register)
        val progressBar = findViewById<ProgressBar>(R.id.progressBar)

        //가입 버튼
        btnRegister.setOnClickListener {
            val email = etEmail.text.toString().trim()
            val password = etPwd.text.toString().trim()
            val name = etName.text.toString().trim()

            viewModel.register(email, password, name)
        }

        //ViewModel 상태 관찰
        lifecycleScope.launchWhenStarted {
            viewModel.registerState.collectLatest { state ->
                when (state) {
                    is RegisterState.Loading -> progressBar.visibility = View.VISIBLE
                    is RegisterState.Success -> {
                        progressBar.visibility = View.GONE
                        Toast.makeText(this@RegisterActivity, "회원가입 성공!", Toast.LENGTH_SHORT).show()
                        startActivity(Intent(this@RegisterActivity, LoginActivity::class.java))
                        finish()
                    }
                    is RegisterState.Error -> {
                        progressBar.visibility = View.GONE
                        Toast.makeText(this@RegisterActivity, state.message, Toast.LENGTH_SHORT).show()
                    }
                    else -> Unit
                }
            }
        }
    }
}
```
</details>
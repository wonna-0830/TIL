# 📌 TIL - Programmers Lv.1 풀이 정리

## 문제 정보
- **2025.08.28**
- **x만큼 간격이 있는 n개의 숫자**
- **[링크](https://school.programmers.co.kr/learn/courses/30/lessons/12954)**

## 📝 문제 요약
> 정수 x와 자연수 n을 받아 x부터 시작해 x씩 증가하는 숫자를 n개 지니는 리스트를 리턴하기

## 💡 풀이 아이디어 
- n번 동안 x에 x를 누적하는 for 문 만들기

## 🧩 구현 코드
```java
import java.util.*;
class Solution {
    public long[] solution(int x, int n) {
        long[] answer = new long[n];
        long y = x;
        long current = x;
        for (int i = 0; i < n; i++){
            answer[i] = current;
            current += y;
        }
        return answer;
    }
}
```
```java
class Solution {
    public long[] solution(int x, int n) {
        long[] answer = new long[n];
        int y = x;
        for ( int i = 0; i < n; i++){
            answer[i] = (long) x;
            x += y;
        }
        return answer;
    }
}
```

## ⚠️ 실수/배운 점
- 두번째 코드가 내가 작성했던 코드였고, 제출하면서 테스트케이스 12까지는 성공했는데 나머지 두개가 테스트 실패였다. 
- 왜 나머지는 잘되고 두개만 안되는지 생각해봤는데 혹시 x가 int값이라 long이랑 범위차 때문에 안되는건가? 싶어서 gpt에게 물어보니 그렇다고 했다.
- 그래서 x를 long값으로 받아서 다시 더하니까 성공!!
- 근데 이럴거면 애초에 x를 long값으로 받으면 되지 않나요?
- 코딩테스트니까 봐드릴게요 ㅡㅡ

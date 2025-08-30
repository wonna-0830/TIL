# 📌 TIL - Programmers Lv.1 풀이 정리

## 문제 정보
- **2025.08.30**
- **문자열 내 p와 y의 개수**
- **[링크](https://school.programmers.co.kr/learn/courses/30/lessons/12916)**

## 📝 문제 요약
> 대문자와 소문자가 섞여있는 문자열 s의 'p'의 개수와 'y'의 개수를 비교해 같으면 True, 다르면 False를 리턴하는 함수 만들기
> 예를 들어 s가 "pPoooyY"면 true를 return하고 "Pyy"라면 false를 return

## 💡 풀이 아이디어 
- 문자열 배열을 만들어 인덱스 하나에 문자 하나씩 넣고, for문으로 순회하면서 p나 y가 포함되면 1씩 더하기
- 그다음 if문으로 p와 y의 개수가 같으면 answer = true, 다르면 answer = false로 변환

## 🧩 구현 코드
```java
import java.util.*;
class Solution {
    boolean solution(String s) {
        boolean answer = true;

        int p = 0;
        int y = 0;
        
        String[] arr = s.split("");
        for(int i = 0; i < arr.length; i++){
            if(arr[i].equals("y") || arr[i].equals("Y")){
                y += 1;
            } else if(arr[i].equals("p") || arr[i].equals("P")){
                p += 1;
            } else {
                continue;
            }
        }
        
        if(p == y){
            answer = true;
        } else {
            answer = !answer;
        }

        return answer;
    }
}
```


## ⚠️ 실수/배운 점
- 진짜 다 잘했는데... 처음에 arr[i].equals()를 안쓰고 arr[i] == "y" 를 써서 구했다.
- 그러니까 테스트 1은 통과가 되는데, 다음 테스트에서 실패돼서 이게 왜... 하고 열심히 찾아봤더니 
- 아차차; equals()가 있는데 왜 이렇게 했지 하고 바꾸니까 성공했다!!!

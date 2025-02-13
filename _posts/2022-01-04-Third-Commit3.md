---
layout: post
title: "vi 명령어 정리"
author: "Jaehyeon"
---

<h3>vi 편집기 명령어로 들어가기 전에 전체적인 구조 살펴보기</h3>

: linux, unix에서 사용하는 vi 편집기는 명령모드, 입력모드, 마지막 행 모드로 총 3가지 모드로 구성되어 있습니다.

-> 흔히 사람들이 말하는 vi 명령어는 이 세 가지 모드를 자유자재로 왔다 갔다 하면서 코드나 글을 작성하는 것을 말합니다.

i) 명령 모드(command mode) - 처음 vi 명령어로 vi를 시작하게 되면 세팅되는 초기 환경입니다.

ii) 입력 모드(insert mode) - 명령모드에서 "i" 나 "a" 명령을 통해서 입력 모드로 넘어갈 수 있습니다. 

입력모드로 가게 되면, 자유롭게 코드나 글을 작성을 하시면 됩니다. 명령 모드로 다시 돌아오려면 "ESC"를 누르면 됩니다.  

iii) 마지막 행 모드(Last line mode) - 마지막행 모드는 명령모드에서 ":" (콜론) 을 입력하면 화면 맨 밑단에 :______ 하며 입력을 할 수 있는 공간이 나옵니다. 

여기서 현재까지 내가 작성한 이 내용을 저장하고 vi를 종료(wq) 할지, 그냥 종료(q, q!) 할지 등을 입력할 수 있습니다.

<h3>자주 사용하는 vi 명령어</h3>

<h4>명령모드</h4>

1. 파일의 끝으로 이동할 때 - G

2. 한 줄 잘라내기 - dd

3. 세 줄 잘라내기 - 3dd
  
4. 붙여넣기 - p

5. 한 글자 삭제 - x

6. 단어 삭제 - dw

7. 실행 취소 - u

8. 줄의 맨 앞 - o

9. 줄의 맨 뒤 - $


<h4>마지막행 모드 (esc -> : 눌렀을때)</h4>

1. 저장 : w

2. 종료 : q

3. 저장 후 종료 : wq

4. 라인 번호 확인 : set nu


<h3>전체적인 흐름</h3>

- vi 명령으로 창을 엽니다.

- 방향 키와 i, a를 이용해서 편집모드에서 편집을 진행합니다.

- 편집 중에 라인을 잘라낼 때는 esc -> dd 눌러서 한 줄을 잘라냅니다.

- 라인을 붙여 넣고 싶을 때에는 해당 위치의 윗줄에서 p를 누릅니다.

- 작성이 끝나면 esc -> 콜론 (:) 버튼을 누르고 -> wq (저장 후 종료)를 입력합니다.







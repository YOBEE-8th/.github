# 임시비밀번호 발급

- 발급을 요청한 이메일이 DB에 존재하는지 체크
- JavaMailSeender를 사용
- 숫자, 알파벳, 특수문자를 포함하는 문자열을 랜덤으로 생성
- 회원가입 인증 코드와 다르게 제한 시간이없으므로 DB에 바로 암호화하여 저장한다.

## 느낀점

- 숫자의 종류는 10개, 알파벳은 대소문자 52개, 허용된 특수문자의수는 13개에서 랜덤으로 나오니 보면서 입력하는 것도 어려웠다. 웹서비스처럼 링크를 클릭하면 이동하여 비밀번호를 입력하는 방법을 사용하는게 사용자가 더 편할것 같다.
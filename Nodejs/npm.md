
## 21-08-19

# package.json
- 설치한 패키지의 버전 관리
- `npm init`으로 생성
- 각종 속성들.. (name, version, scripts, license ...)

### scripts 
- npm 명령어를 저장해두는 부분
- `npm run [스크립트 명령어]`로 스크립트 실행
- `npm run test`

### `npm install [--save], [--save-dev]` 옵션

### `node_modules` 폴더
- 설치한 패키지들이 저장된 폴더

# package-lock.json
- `node_modules`에 들어 있는 패키지들의 정확한 버전과 의존 관계가 담겨 있음
- npm으로 패키지 설치,수정,삭제할 때마다 패키지들 간의 내부 의존 관계를 저장

### npm 전역 설치
- `npm install --global [패키지명]`
- package.json에 기록되지 않음

### npx
- 패키지를 전역 설치한 것과 같은 효과를 얻을 수 있음
- package.json에도 기록

# 패키지 버전
- SemVer 방식의 버전 넘버링
- ex) 1.0.7
- Major 버전
- Minor 버전
- Patch 버전

### 패키지 버전 기호
- ^ 
  - minor 버전까지만 설치/업데이트
- ~
  - patch 버전까지만 설치/업데이트 
- \>, <, >=, <=, =
- 그 외 @latest, @next 등

# 그 외 명령어들
- `npm outdated`
- `npm uninstall`
- `npm search`
- `npm info` ... 등등
- `npm adduser`, `npm publish` 등 패키지 배포도 가능

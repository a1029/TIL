
## 21-08-20

# Nodejs의 웹 서버 프레임워크
- http모듈의 요청과 응답 객체에 추가 기능들을 부여
- `app.set`
- `app.listen`
- `app.get`
  - get 요청에 대한 처리
- `app.post`
  - post 요청에 대한 처리
- 그외 ...


# 미들웨어
- 익스프레스의 핵심
- 요청과 응답의 중간에 위치
- 요청과 응답을 조작, 기능 추가, 필터링 등
- 보통 `app.use`로 연결, `app.get`, `app.post`에서도 가능

## 미들웨어 패키지들
- morgan
- cookie-parser
- express-session
- body-parser
- multer
- 등등..

# Router
- app.js 코드가 길어지는 것을 막기 위해 라우터를 따로 분리
- `const router = express.Router()`, `router.get('/', ...)`
- app.js에서는 `app.use`로 라우터 연결
- `next('route')` : 다음 미들웨어를 실행하지 않고 다음 라우터로 이동
- 라우트 매개변수
- `router.route`로 요청 메소드 처리 함수를 합칠 수 있음

## 템플릿 엔진
- 자바스크립트를 사용해서 HTML을 렌더링
- pug, nunjucks
- React.js, Vue.js와의 차이점?

### nunjucks
- 모질라에서 개발
- HTML 문법을 그대로 사용하되 추가로 자바스크립트 문법 사용
- 확장자는 html 또는 njk
- 변수, 반복문, 조건문, include, extends/block 
- 렌더링시 `res.render('index')` // index.html
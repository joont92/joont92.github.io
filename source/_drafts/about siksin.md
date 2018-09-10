식신  
푸드테크 회사  
O2O, B2C  

인프라  
Slack, Jira, 웍스모바일  
GitLab, Jenkins, AWS,  

# 흥
L4에서 web1, web2로 로드 밸런싱
아파치가 꺼지면 자동으로 다른쪽으로 포워딩하므로 L4를 따로 컨트롤 할 필요는 없었다
web1, web2에 apache, tomcat 존재. 로그 정리 크론탭 정도가 들어 있었음
db server에 maria db 설치, 크론탭은 포스트 랭킹, 메인 페이지 표출 예약하는 것 정도?  

어플리케이션은 스프링 부트, jpa로 구성  
아파치 2.4
톰캣 8
세션 db는 redis

프로젝트 구조
application.properties는 config 폴더로 수정(run 옵션)
이는 메이븐 빌드시에 profile 값을 줘서 다른 설정을 바라보게 하기 위함
ex) 개발 : application.dev.properties, 상용 : application.properties
빌드 후 묶여진 jar를 서버에 배포한 뒤, java 명령어로 실행

nohup /usr/local/java/bin/java -Dlog4j.configurationFile=file:///home/seeon/heung/config/log4j2.xml -jar /home/seeon/heung/heung-0.0.1-SNAPSHOT.jar --spring.config.location=file:/home/seeon/heung/config/application.properties >> /home/seeon/heung/heung.out &s

자바 소스는 기본 메이븐 형태

템플릿 엔진은 Thymeleaf 사용. 커스텀 태그가 없는것이 깔끔했다.

img, css, javascript는 static 폴더에 따로 지정
의존성 관리도구로는 bower, npm 사용(같이 쓸 필요는 없었음 ^^; npm > bower)
빌드 툴로는 grunt 사용.  src/**/*.js -> build/**/*.min.js
내부는 requireJS, jQuery 사용. 로딩되는 애들은 require, 모듈은 define?

HomeController(사용자), BosController(관리자) 
사용자는 그냥 조건(카테고리, 정렬)에 따라 컨텐츠 불러와서 뿌리는게 다임  
반응형이라 정렬순서 맞추는 부분이 좀 복잡해진거 정도?  
more_data로 불러올때 html string을 불러와서 append 해줬다는 정도.

이미지 저장 방식
포스트 등록시에는 file change 이벤트로 업로드하는 파일을 잡을 수 있음
/bos/contents/temp 컨트롤러에서 MultipartFile로 받은 뒤, 이미지 확장자나 용량체크
통과되면 유니크한 이름으로 temp 폴더에 이동(기본적으로 Multipart파일은 등록시 어딘가에 저장하기 때문)
최종 등록 시 temp 폴더에 저장된 이미지를 실제 저장할 폴더(e.g. post)로 옮기고,
그 경로를 db에 저장함.
컨텐츠의 경우 최종 등록시에 이미지 경로를 전부 replace 하고 디비에 따로 저장하진 않음.
흥의 경우 이미지 확인 버튼 누르면 크롭가능한 모달창이 뜨게 되고,
크기를 지정한만큼 짜른 뒤에 다시 temp에 덮어쓰는 방식이 있음

예외 처리 방식
예외 발생 시 Exception 정보를 받아서 에러 핸들링
일반 요청일 경우 에러 페이지 리턴, json 요청일 경우 json에 에러 메세지 담아서 리턴

AOP에서는 
@Before로 roll 체크, @Around로 AccessDenyException 정도 체크한다. 나머지 에러는 ExceptionHandler에 걸리게 될 것이고..
예외 클래스는 Throwable 받아서 instanceof로 비교한 뒤 변환한다.

# 식신
서버 설정
기존 L4에 web1, web2
web1 끄고 톰캣 껏다켜고,
web2 끄고 톰캣 껏다켜고..
web1, web2는 각각 서버 따로따로.

에서 aws로 변경됨
www.siksinhot.com -> route 53 -> ****.cloudfront.net -> cloud front
-> load balancer -> web1, web2 apache -> *.aws.siksinhot.com 으로 리다이렉트
-> route 53 -> internal... -> load balancer -> ec2 instance load balancing

세션은 엘라스틱캐시(memcached) 사용

RDS는 오로라 디비(MySQL+PostgreSQL 두개 호환되는 AWS용 디비?)

프로젝트 구조  
리액트 라우터에 URL별 컴포넌트 매핑(여기 매핑된 컴포넌트는 컨테이너라고 부름)  
컨테이너는 하나의 페이지를 의미함  
containers 폴더에 컨테이너가 모두 존재. 이를 또 세부 서비스별로 디렉터리로 나눔  
SPA형태의 어플리케이션이므로, 하나의 페이지 내에서도 다양하게 영역이 나뉨.  
containers 디렉터리와 동일하게 components에도 만듬  
components에는 각 페이지별 영역이 크게 크게 나뉘어져 들어가 있었다.
걔네들은 html + 공통 컴포넌트를 사용하였는데, 공통 컴포넌트는 Common에 다 들어있었다.
유틸리티 같은 경우 Utils

프로젝트 내 문법은 전부 es6 문법을 사용하는데,  
webpack의 경우 babel-loader로 transfile 했고,  
node(ssr)의 경우 실행전에 babel-register, babel-polyfill을 실행하고 띄우도록 했다.   

개발시에는 nodemon을 사용해서 소스 변경마다 node가 재시작되게 했고,
webpack 또한 watch모드를 소스 변경마다 새로 빌드되도록 했다.
프로덕션 모드에서는 code splitting이 적용되도록 했다.  
code splitting은 빌드할 때 통짜 min 파일로 나오는게 아니라 리액트 라우터별로 구분해서 빌드파일이 생성되게 한다.  
1.hash.js, 2.hash.js...  
그리고 해당하는 url을 호출할 때 js가 호출되게 한다.  
필요할때만 호출하므로 성능을 좀 더 향상시킬 수 있었다.  
hash의 경우 빌드때마다 매번 고유값을 생성하기 때문에 chunkhash를 사용하는 것이 더 좋다.  
chunkhash의 경우 변경이 있을 경우에만 고유한 값을 새로 부여한다.  

서버사이드 렌더링은 어떻게 구현했나?  
일단 대부분의 컴포넌트들은 api에서 받아온 결과가 있을 경우 출력하는 부분과 없을 경우 출력하는 부분이 나뉘어져 있었다.  
각 컨테이너별로 api를 호출하는 static 메서드가 존재헀다.  
그래서 노드가 요청을 받으면 router에서 해당 컴포넌트를 찾은 뒤 static 메서드에 있는 api 호출을 선행했다. 그리고 그 결과를 공유 store인 appStore(mobx)에 담았다.  
이후 해당 component의 render 메서드가 실행되면서 htmlString이 생성되고,  
그 결과를 브라우저로 돌려주게 된다. 
api 호출이 선행되었으므로 모든 컴포넌트는 빈 값 없이 다 출력되게 되고, 검색엔진에서는 그것을 다 들고간다. 원래 react라면 빈 컴포넌트 호출 후 lifecycle에서 api호출을 실행하고 re-rendering 하는 식으로 한다.  

appStore에 찬 값은 browser 전역 변수에 저장하고, js가 로딩될 떄 그것을 전부 JSON parsing해서 변수에 담는다.  
즉 ssr에서 조회된 api 결과값은 브라우저에서 다시 조회할 필요가 없다는 뜻이다.  

페이지가 이동할 때는 render -> componentDidMount에서 api조회 -> 조회결과가 mobx에 저장 -> observer pattern에 의해 해당 변수를 사용하고 있는 자식 컴포넌트들은 re-rendering

SPA내에서 변화가 있을 떄는 componentWillReceiveProps나 shouldComponentUpdate를 사용했다.  
ReactRouter가 변화하면 containers에서 그것을 받을 수 있었다.  

Common
카드, ETC, Utils 등
지도
지도 위치를 이동할때마다 잡을 수 있는 이벤트들이 각기 제공되고 있었고
그 이벤트가 실행될 떄 마다 지도의 양 끝점 내에 있는 매장 리스트들을 들고와서 뿌렸다.  
매장을 찍을떄는 처음 나오는 매장을 기준으로 원을 그리고 그 안에 다른 매장이 intersect 될 때 그 매장들을 리스트에 넣는다. 그래서 한 묶음을 형성하고, 묶음에 들어간 애들은 삭제.  
그렇게 해서 모든 묶음이 완성되면 + 표시로 표출
줌 레벨이 높을 경우에는 모두 표출하는데 겹치는 애들이 있으니 조금씩 비틀어서 표출

위치별 매장 조회가 아닌 경우 매장리스트들을 목록으로 받아서 뿌렸다.  
그리고 위치는 매장리스트들의 중간지점을 바라보게 했다.  


....
리액트를 사용해야 하는 이유?  
UI를 모듈화 할 수 있다. 게다가 리액트에서 제공하는 class + JSX의 구조는 매우 직관적이다. UI 모듈화는 개인적으로 혁명이라고 생각함. UI쪽에서는 객체지향적 프로그래밍이 불가능했었기 때문에.. 
항상 똑같은 html 코드를 갖다 붙이고 거기에 서버랭귀지를 써가며 코딩했었다.  
이는 변화에 유연하지 못하다. UI가 바뀌면 해당 페이지를 다 수정해줘야 하지 않는가!  
그리고 또 좋은점은, Virtual DOM 이다.  
요즘은 대부분 SPA여서 DOM 조작이 매우 많은데, 일반적으로 그냥 DOM을 조작하게 되면 우리 눈에는 별다를게 없지만 브라우저는 html 트리를 다시 재해석하게 된다.  
Virtual DOM을 이용하면 DOM에 무언가 변경이 행해졌을떄 Virtual DOM이 최종적으로 DOM에 업데이트하기전에 정말 변경되었는지 체크하고, 변경되었으면 이를 최종적으로 적용시키면서 연산횟수를 줄여준다. 그리고 실제 변경여부 체크를 VirtualDOM이 알아서해주기 때문에 우리는 이것에 신경쓰지 않아도 된다.  

JPA를 사용해야 하는 이유?  
패러다임 불일치 해결  
상속, 관계, 탐색, 동일성
SQL 의존성 해결  

IOC란 제어의 역전을 의미하는 말로써 흐름을 직접 주도하는 메인 메서드 개발 방식과는 달리 프레임워크가 흐름을 주도하며 개발자가 만들 애플리케이션 코드를 사용하는 방식이다.  
DI는 이 과정에서 각 클래스간의 의존관계가 있을 경우 주입해주는 역할을 한다.  
이 DI가 진짜 중요한 이유가 뭐냐면, 이 행위 자체가 결합도가 낮은 프로그래밍, 전략패턴을 사용한 프로그래밍을 하는 과정이고, 이 과정을 너무 편리하게 해줘서 우리가 이 프로그래밍 방식을 계속 준수하여 개발하게끔 해준다는 것.  
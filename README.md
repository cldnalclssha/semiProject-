# semiProject-
![image](https://github.com/user-attachments/assets/7cc7862f-1b03-4321-a7b2-046eb74386a7)
프로젝트 기간: 2024.06.24 - 2024.07.16

참여 인원: 총 6명

개발 목표: 다중 배달로 인한 배송 시간 문제를 해결하고, 사용자와 라이더 간의 효율적인 1:1 매칭 시스템을 구현하는 것입니다.<br/>이 시스템은 고객이 요청한 주문을 최적의 조건의 라이더가 직접 선택하여 배송 시간을 최소화하고,<br/> 서비스 품질을 향상시키는 데 목적이 있습니다.

사용 기술 및 개발 환경

운영체제: Window10<br/>
사용 언어: Java, JavaScript<br/>
프레임워크/라이브러리: Spring<br/>
데이터베이스: Oracle<br/>
도구: Visual Studio Code, git, sourcetree, eclipse<br/>
WAS: Apache Tomcat<br/>
협업 도구: Github<br/>

![스크린샷 2024-07-18 140754](https://github.com/user-attachments/assets/bd071456-cd0e-43a5-9b7a-0aaf845ea9b9)
카카오 맵 구현

이 코드는 카카오맵 API와 카카오모빌리티 API를 사용하여 지도를 표시하고, 마커를 추가하며, 마커 사이의 경로를 폴리라인으로 표시하는 기능을 구현한 것입니다. <br/>사용자가 지도상에서 위도와 경도를 보는 것은 직관적이지 않다고 판단하여, 지오코더를 활용하여 실제 주소로 변환하여 사용자에게 제공하도록 했습니다.
```java
<script type="text/javascript"
	src="https://dapi.kakao.com/v2/maps/sdk.js?appkey=API-KEY&libraries=services"></script>
// 'map' 아이디를 가진 HTML 요소를 선택
var mapContainer = document.getElementById('map'); 

// 지도 옵션 설정
var mapOption = {
    center: new kakao.maps.LatLng(33.450701, 126.570667), // 초기 중심 좌표 설정
    level: 3 // 지도 확대 레벨 설정
};

// 지도 생성
var map = new kakao.maps.Map(mapContainer, mapOption);

// 주소-좌표 변환 객체 생성
var geocoder = new kakao.maps.services.Geocoder();

// 브라우저에서 지오로케이션을 지원하는지 확인
if (navigator.geolocation) {
    // 지오로케이션을 통해 현재 위치를 얻어옴
    navigator.geolocation.getCurrentPosition(function(position) {
        var lat = position.coords.latitude; // 위도
        var lon = position.coords.longitude; // 경도
        var locPosition = new kakao.maps.LatLng(lat, lon); // 현재 위치의 좌표 객체 생성
        map.setCenter(locPosition); // 현재 위치를 지도 중심으로 설정
    }, function(error) {
        console.error(error); // 오류 발생 시 콘솔에 오류 메시지 출력
    });
} else {
    console.error("Geolocation is not supported by this browser."); // 지오로케이션을 지원하지 않는 경우
}

// 마커와 폴리라인 배열 초기화
var markers = [];
var polylines = [];

// 지도 클릭 이벤트 리스너 추가
kakao.maps.event.addListener(map, 'click', function(mouseEvent) {
    if (markers.length >= 2) {
        clearMap(); // 마커가 2개 이상인 경우 맵 초기화
    } else {
        var latlng = mouseEvent.latLng; // 클릭한 위치의 좌표 얻기
        var marker = new kakao.maps.Marker({
            position: latlng // 마커 위치 설정
        });
        marker.setMap(map); // 지도에 마커 추가
        markers.push(marker); // 마커 배열에 추가
        getAddressFromCoords(latlng); // 좌표로 주소 찾기
        if (markers.length === 2) {
            setTimeout(() => {}, 5000); // 5초 지연 (명시적 딜레이가 없는 빈 함수)
            findRouteAndDrawLine(); // 경로 찾고 선 그리기
        }
    }
});

// 두 마커 사이의 경로를 찾고 선을 그리는 함수
function findRouteAndDrawLine() {
    var start = markers[0].getPosition(); // 첫 번째 마커 위치
    var end = markers[1].getPosition(); // 두 번째 마커 위치
    
    // 경로 요청 URL 생성
    var url = `https://apis-navi.kakaomobility.com/v1/directions?origin=${start.getLng()},${start.getLat()}&destination=${end.getLng()},${end.getLat()}&waypoints=&priority=RECOMMEND&road_details=false`;
    
    // 경로 요청 API 호출
    fetch(url, {
        method: 'GET',
        headers: {
            'Authorization': 'KakaoAK API-KEY' // 인증 키
        }
    })
    .then(response => response.json()) // 응답을 JSON으로 변환
    .then(data => {
        if (data.routes && data.routes.length > 0) {
            var route = data.routes[0]; // 첫 번째 경로 선택
            var linePath = []; // 경로를 저장할 배열
            route.sections.forEach(section => {
                section.roads.forEach(road => {
                    road.vertexes.forEach((vertex, index) => {
                        if (index % 2 === 0) {
                            linePath.push(new kakao.maps.LatLng(road.vertexes[index+1], road.vertexes[index])); // 경로 좌표 추가
                        }
                    });
                });
            });

            // 폴리라인 생성 및 지도에 추가
            var polyline = new kakao.maps.Polyline({
                path: linePath,
                strokeWeight: 5,
                strokeColor: '#86C1C6',
                strokeOpacity: 0.7,
                strokeStyle: 'solid'
            });

            polyline.setMap(map); // 지도에 폴리라인 추가
            polylines.push(polyline); // 폴리라인 배열에 추가

            var distance = polyline.getLength(); // 폴리라인의 길이 계산
            var distanceKm = (distance / 1000).toFixed(2); // 길이를 km 단위로 변환하여 소수점 2자리까지 표시
            
            var distanceMoney = 5000 + Math.floor(distanceKm * 4000); // 거리 기반 요금 계산
            document.getElementById("price").value = distanceMoney + "원"; // 계산된 요금 표시
            document.getElementById("dis").value = distanceKm + "km"; // 계산된 거리 표시
            console.log('두 마커 사이의 거리는 ' + distanceKm + ' km 입니다.');
        } else {
            console.log('경로를 찾을 수 없습니다.');
        }
    })
    .catch(error => console.log('경로 요청 중 오류 발생:', error)); // 오류 발생 시 콘솔에 출력
}
function getAddressFromCoords(coords) {
    // 좌표를 주소로 변환하는 함수 호출
    geocoder.coord2Address(coords.getLng(), coords.getLat(), function(result, status) {
        // 주소 변환이 성공한 경우
        if (status === kakao.maps.services.Status.OK) {
            var address = result[0].address.address_name; // 변환된 주소를 가져옴
            if (markers.length === 1) {
                // 마커가 하나일 때는 시작 지점의 주소로 설정
                document.getElementById('start-point').value = address;
            } else if (markers.length === 2) {
                // 마커가 두 개일 때는 끝 지점의 주소로 설정
                document.getElementById('end-point').value = address;
            }
        }
    });
}
```
![스크린샷 2024-07-18 140523](https://github.com/user-attachments/assets/fd7d66a7-26fc-4f41-ab3b-cfddf42ef9db)

signup.jsp(회원가입 페이지)

회원가입의 필요한 정보를 입력받고 회원가입 버튼을 누르면 정보들이 post 방식으로 전달되며,<br/> 아이디와 닉네임은 중복검사 버튼을 만들어서 함수가 실행되게 한다.
```java
<div class="container">
      <div class="signup-form">
         <div class="form-header">
            <h1>회원가입</h1>
         </div>
         <form action="${contextPath }/user/insert" method="POST">
            <div class="radio-input-wrapper">
               <label class="label"> <input value="regular" name="role"
                  id="regular" class="radio-input" type="radio" checked>
                  <div class="radio-design"></div>
                  <div class="label-text">일반회원</div>
               </label> <label class="label"> <input value="rider" name="role"
                  id="rider" class="radio-input" type="radio">
                  <div class="radio-design"></div>
                  <div class="label-text">라이더</div>
               </label>
            </div>
<div class="form-group">
    <!-- 아이디 입력란 -->
    <label for="email">아이디</label>
    <input type="text" id="email" name="email" placeholder="실제 사용하는 이메일을 입력하세요." maxlength="100" required>
    <!-- 중복확인 버튼 -->
    <button type="button" class="hi" onclick="idCheck();">email중복확인</button>
</div>

<div class="form-group">
    <!-- 비밀번호 입력란 -->
    <label for="password">비밀번호</label>
    <input type="password" id="password" name="password" placeholder="비밀번호를 입력하세요." maxlength="100" required>
</div>

<div class="form-group">
    <!-- 닉네임 입력란 -->
    <label for="nickname">닉네임</label>
    <input type="text" id="nickname" name="nickname" placeholder="사용할 닉네임을 입력하세요." maxlength="100" required>
    <!-- 중복확인 버튼 -->
    <button type="button" class="hi" onclick="nnCheck();">닉네임 중복확인</button>
</div>

<div class="form-group">
    <!-- 생년월일 입력란 -->
    <label for="birth">생년월일</label>
    <input type="text" id="birth" name="birth" placeholder="생년월일을 입력하세요. ex) 2000.02.17 -> 000217" maxlength="11" required>
</div>

<div class="form-group">
    <!-- 전화번호 입력란 -->
    <label for="phone">전화번호</label>
    <input type="text" oninput="formatPhoneNumber(this)" id="phone" name="phone" placeholder="휴대폰 번호를 입력하세요. ex) 01030532345" maxlength="13" required>
</div>

<div class="form-group">
    <!-- 주소 입력란 -->
    <label for="address">주소</label>
    <input type="text" id="address" name="address" placeholder="주소를 입력하세요." maxlength="255" required>
</div>

<div class="category-selector">
    <div>
        <!-- 성별 선택 -->
        <label>
            <input type="radio" name="gender" id="radio" value="W" checked>
            <span>Women</span>
        </label>
        <label>
            <input type="radio" name="gender" id="radio" value="M">
            <span>Men</span>
        </label>
    </div>
</div>

<button class="tooltip" id="" onclick="cleanPhoneNumber()">회원가입</button>
 </form>
signup.jsp (script. part)

아이디와 닉네임은 중복을 제한하여 회원관리를 용이하게 한다. <br/>ajax를 통해서 중복된 정보를 입력할시에 alert 창이 뜨게 하는 함수.<br/>
 function idCheck() {
            var $email = $(".form-group input[name=email]");

            $.ajax({
                url: "/semi/user/idCheck",
                method: "GET",
                data: { email: $email.val() }, // 데이터를 URL 인코딩 형식으로 전송
                success: function(result) {
                   console.log("result");
                    if (result == 1) { // 이미 사용 중인 이메일
                        alert("이미 사용 중인 이메일입니다.");
                        $email.val("");
                        $email.focus();
                    } else { // 사용 가능한 이메일
                        alert("사용 가능한 이메일입니다.");
                    }
                }
            });
        }
        
        function nnCheck() {
            var $nickname = $(".form-group input[name=nickname]");

            $.ajax({
                url: "/semi/user/nnCheck",
                method: "GET",
                data: { nickname: $nickname.val() }, // 데이터를 URL 인코딩 형식으로 전송
                success: function(result) {
                   console.log("result");
                    if (result == 1) { // 이미 사용 중인 이메일
                        alert("이미 사용 중인 닉네임입니다.");
                        $nickname.val("");
                        $nickname.focus();
                    } else { // 사용 가능한 이메일
                        alert("사용 가능한 닉네임입니다.");
                    }
                }
            });
            
        }<br/>
휴대폰 번호 입력 포맷팅 함수

        function formatPhoneNumber(input) {
    let cleaned = input.value.replace(/\D/g, ''); // 입력값에서 숫자가 아닌 문자 제거
    let formatted = '';

    if (cleaned.length > 3 && cleaned.length <= 7) {
        formatted = cleaned.replace(/(\d{3})(\d+)/, '$1-$2'); // 3-7자리 번호 포맷팅 (XXX-XXXX)
    } else if (cleaned.length > 7) {
        formatted = cleaned.replace(/(\d{3})(\d{4})(\d+)/, '$1-$2-$3'); // 8자리 이상 번호 포맷팅 (XXX-XXXX-XXXX)
    } else {
        formatted = cleaned; // 그 외의 경우 그대로 유지
    }

    input.value = formatted; // 포맷팅된 값으로 업데이트
}

function cleanPhoneNumber() {
    let phoneInput = document.getElementById('phone'); // 'phone' ID를 가진 요소 선택
    phoneInput.value = phoneInput.value.replace(/-/g, ''); // 입력값에서 하이픈 제거
}
```

![스크린샷 2024-07-18 140558](https://github.com/user-attachments/assets/0434e3e1-2159-42cb-87bd-07e02e208469)

login.jsp(로그인 페이지)

이메일과 비밀번호를 입력하고 로그인버튼을 누르면 회원가입이 되어 있는 경우에는 성공, 틀리면 오류 메세지가 뜬다.<br/>
로그인 버튼 밑에는 아이디 찾기와 비밀번호 찾기 페이지로 넘어갈수 있는 버튼이 있다.
```java
<div class="container">
    <form action="${contextPath}/user/login" method="POST">
        <div class="login-box">
            <div class="title">로그인</div>
            
            <!-- 이메일 입력란 -->
            <div class="input-group">
                <div class="input-label">이메일</div>
                <input type="text" name="email" class="input-field" placeholder="Enter your email">
            </div>
            
            <!-- 비밀번호 입력란 -->
            <div class="input-group">
                <div class="input-label">비밀번호</div>
                <input type="password" name="password" class="input-field" placeholder="Enter your password">
            </div>
            
            <!-- 로그인 오류 메시지 -->
            <c:if test="${not empty loginError}">
                <div class="error-message" style="color: red;">${loginError}</div>
            </c:if>
            
            <!-- 로그인 버튼 -->
            <button type="submit" class="login-button">로그인</button>
            
            <!-- 아이디 찾기 링크 -->
            <a class="find-ID" onclick="findid()"> 아이디 찾기</a>
            
            <!-- 비밀번호 찾기 링크 -->
            <a class="find-ID" onclick="findpw()"> 비밀번호 찾기</a>
        </div>
        
        <!-- 이미지 -->
        <img class="image" src="${contextPath}/resources/images/cut_2.png">
    </form>
</div>
아이디 찾기 비밀번호 찾기 함수 구현

//아이디 찾기
	function findid() {
		location.href = '${contextPath}/user/idfind';
	}
	//비밀번호 찾기
	function findpw() {
		location.href = '${contextPath}/user/pwfind';
	}
UserController.java

@PostMapping("/login")
public String login(@ModelAttribute User u, Model model, HttpSession session) {
    // 사용자가 입력한 정보로 로그인을 시도하고, 결과를 받아옵니다.
    User loginUser = uService.login(u);
    
    String url = "";
    // 로그인 실패 시
    if (!(loginUser != null && encoder.matches(u.getPassword(), loginUser.getPassword()))) {
        // 로그인 실패 메시지를 모델에 추가하여 화면에 표시합니다.
        model.addAttribute("loginError", "로그인에 실패하셨습니다");
        url = "/user/login"; // 로그인 페이지로 다시 이동합니다.
    } else {
        // 로그인 성공 시, 세션에 로그인 사용자 정보를 저장합니다.
        session.setAttribute("loginUser", loginUser);
        
        // 이전 페이지로 이동할 URL이 있는 경우, 해당 URL로 리다이렉트합니다.
        String nextUrl = (String) session.getAttribute("nextUrl");
        url = "redirect:" + (nextUrl != null ? nextUrl : "/"); // 기본적으로는 홈 페이지로 이동합니다.
        session.removeAttribute("nextUrl"); // 다음에 사용할 URL 세션에서 제거합니다.
    }
    return url; // 최종적으로 이동할 URL을 반환합니다.
}
user-mapper.xml

로그인하기위해 DB를 통해 검색하는 쿼리
<select id="login" resultType="User">
    <!-- 
        사용자가 입력한 이메일과 상태가 'Y'인 사용자를 검색하는 쿼리입니다.
        #{email}은 사용자가 입력한 이메일을 바인딩합니다.
    -->
    SELECT *
    FROM USERS
    WHERE EMAIL = #{email} AND STATUS = 'Y'
</select>
```

![스크린샷 2024-07-18 140633](https://github.com/user-attachments/assets/cc954e5d-fea0-4dca-8700-58d2dbac09ae)

idfind.jsp(아이디 찾기)

회원가입 할때 입력했던 휴대폰 번호를 입력하면 DB에 저장되어 있는 아이디를 ajax를 통해 비동기로 밑에 띄워주는 코드이다.
```java
 <div class="container">
        <img src="${contextPath}/resources/images/logo.jpg" alt="아이콘" class="icon">
        <form id="idFindForm">
            <p class="instruction">아이디를 찾고자하는 휴대폰 번호를 입력해주세요.</p>
            <input type="text" name="phone" placeholder="휴대폰 번호" class="email-input">
            <button type="button" id="findIdButton" class="custom-button">확인</button>
        </form>
        <p>아이디: <span id="foundId"></span></p>
        <a href="${contextPath}/user/login" class="login-link" >로그인 페이지로 이동하기</a>
    </div>

    <script>
    $(document).ready(function() {
        $('#findIdButton').click(function() {
            var phone = $('input[name="phone"]').val();
            $.ajax({
                type: 'POST',
                url: '${pageContext.request.contextPath}/user/idfind',
                contentType: 'application/x-www-form-urlencoded; charset=UTF-8',
                data: { phone: encodeURIComponent(phone) },
                success: function(response) {
                    $('#foundId').text(response);
                },
                error: function() {
                    $('#foundId').text('아이디를 찾을 수 없습니다.');
                }
            });
        });
    });
pwfind.jsp(비밀번호 찾기)

찾고자 하는 비밀번호에 맞는 아이디와 생년월일을 입력하면 비밀번호를 변경할수 있는 페이지로 이동하게 된다.

<form id="myForm" action="${pageContext.request.contextPath}/user/changepw" method="post">
    <div class="container">
        <img src="${contextPath}/resources/images/logo.jpg" alt="아이콘" class="icon">
  
        <p class="instruction">비밀번호를 찾고자하는 아이디, 이메일을 입력해주세요.</p>
        
        <!-- 생년월일 입력란 -->
        <div class="input-group">
            <input type="text" name="birth" placeholder="생년월일 6자리를 입력하시오 ex)000217" class="input-field">
        </div>
        
        <!-- 이메일 입력란 -->
        <div class="input-group">
            <input type="text" name="email" placeholder="이메일을 입력하세요.." class="input-field">
        </div>
        
        <!-- 확인 버튼 -->
        <button class="submit-btn">확인</button>
  
        <!-- 아이디 찾기 링크 -->
        <a href="${contextPath}/user/idfind" class="find-id-btn">아이디가 기억나지 않는다면? 아이디 찾기</a>
    </div>
</form>
UserController.jsp(아이디/비밀번호 변경 관련)

입력한 아이디와 생년월일과 일치하는 비밀번호가 존재한다면 비밀번호를 변경할수 있는 페이지로 이동할수 있게 되고, <br/>비밀번호를 중복으로 확인하여 일치하면 비밀번호가 업데이트 된다.
@PostMapping("/idfind")
@ResponseBody
public ResponseEntity<String> idfind(@RequestParam String phone) {
   // 사용자 서비스를 통해 입력된 전화번호로 아이디를 찾습니다.
   String username = uService.idfind(phone);
   // 찾은 아이디가 있으면 해당 아이디를 반환하고, 없으면 메시지를 반환합니다.
   String responseMessage = (username != null) ? username : "아이디를 찾을 수 없습니다.";
   return ResponseEntity.ok().contentType(MediaType.TEXT_PLAIN).body(responseMessage);
}

@PostMapping("/changepw")
public String findPassword(@RequestParam("email") String email, @RequestParam("birth") String birth,
      HttpSession session, Model model) {
   // 사용자 서비스를 통해 입력된 이메일과 생년월일로 비밀번호를 찾습니다.
   String password = uService.pwfind(birth, email);
   
   session.setAttribute("email", email);

   if (password != null) {
      // 비밀번호를 찾았을 경우 모델에 비밀번호를 추가하고 성공 페이지로 이동합니다.
      model.addAttribute("password", password);
      return "/user/changepw"; // 성공 페이지로 이동
   } else {
      // 비밀번호를 찾지 못했을 경우 에러 메시지를 추가하고 실패 페이지로 이동합니다.
      model.addAttribute("error", "No matching account found.");
      return "user/pwfind"; // 실패 페이지로 이동
   }
}

@PostMapping("/updatepw")
public String updatePassword(@RequestParam("pw1") String pw1, @RequestParam("pw2") String pw2,
      @SessionAttribute("email") String email, RedirectAttributes ra) {
   // 입력된 두 개의 비밀번호가 일치하는지 확인합니다.
   if (pw1.equals(pw2)) {
      // 입력된 비밀번호를 암호화합니다.
      String encPwd = encoder.encode(pw1);
      // 비밀번호를 업데이트하고 그 결과를 반환합니다.
      int result = uService.updatepw(encPwd, email);

      if (result >= 1) {
         // 비밀번호 업데이트 성공 시 알림 메시지를 추가하고 메인 페이지로 리다이렉트합니다.
         ra.addFlashAttribute("alertMsg", "비밀번호 변경 성공");
         return "redirect:/"; // 성공 페이지로 리다이렉트
      } else {
         // 비밀번호 업데이트 실패 시 메인 페이지로 리다이렉트합니다.
         return "redirect:/"; // 실패 페이지로 리다이렉트
      }
   } else {
      // 입력된 두 개의 비밀번호가 일치하지 않을 경우 메인 페이지로 리다이렉트합니다.
      return "redirect:/"; // 실패 페이지로 리다이렉트
   }
}
user-mapper.xml(아이디/비밀번호 변경관련)

아이디와 생일이 일치하면 DB에 저장된 일치하는 비밀번호가 업데이트된다.
<select id="idfind" resultType="string">
      SELECT EMAIL
      FROM USERS
      WHERE PHONE
      = #{phone}

   </select>

   <select id="pwfind" parameterType="map" resultType="string">
      SELECT
      PASSWORD
      FROM USERS
      WHERE BIRTH = #{birth}
      and EMAIL = #{email}
   </select>


   <update id="pwupdate">
      UPDATE USERS
      SET PASSWORD = #{encPwd}
      WHERE EMAIL =
      #{email}
   </update>
```

![스크린샷 2024-07-18 141051](https://github.com/user-attachments/assets/6ff7af1f-a97f-43c7-8a7b-823c5f33e328)

report.jsp(신고내용 작성 페이지)

사용자가 신고 내용을 입력하고 제출할 수 있는 페이 지를 제공하여, 신고 기능을 구현합니다.
```java
<main>
    <form action="${contextPath}/report/save" method="POST" id="reportForm" class="report-form">
        <!-- 신고 내용을 제출하는 폼 -->
        
        <div class="form-group">
            <label for="reportContent">신고 내용</label> 
            <!-- 신고 내용 입력 필드 레이블 -->
            <textarea id="reportContent" name="content" rows="5" required>
            </textarea> 
            <!-- 신고 내용 입력 필드 -->
        </div>
        
        <div class="button-group">
            <button type="submit" class="btn-submit">신고 제출</button> <!-- 폼 제출 버튼 -->
        </div>
    </form>
</main>
reportList.jsp(신고목록 페이지)

관리자가 사용자가 제출한 신고 내용을 확인할 수 있는 페이지를 제공합니다.
    <div class="header">
        <h2>신고 목록</h2> <!-- 페이지 섹션 제목 -->
    </div>
    <table>
        <thead>
            <tr>
                <th>이름</th> <!-- 테이블 헤더: 이름 -->
                <th>신고 내용</th> <!-- 테이블 헤더: 신고 내용 -->
            </tr>
        </thead>
        <tbody>
            <!-- 신고 목록을 반복해서 출력하는 부분 -->
            <c:forEach items="${reports}" var="report">
                <tr>
                    <td>${report.nickName}</td> <!-- 신고자 이름 -->
                    <td>${report.content}</td> <!-- 신고 내용 -->
                </tr>
            </c:forEach>
        </tbody>
    </table>
    <div class="footer">
        &copy; 2024 신고 목록 서비스. All rights reserved. <!-- 푸터 섹션 -->
    </div>
</div>
report-mapper.xml

DB에 신고내역을 저장하고 신고목록에 보여주는 REPORT-MAPPER
<mapper namespace="report">

	<insert id="saveReport" parameterType="Order"
		useGeneratedKeys="true">
		<selectKey keyProperty="reportId" resultType="int"
			order="BEFORE">
			SELECT SEQ_RENO.NEXTVAL FROM DUAL
	</selectKey>
	
	INSERT INTO REPORTS
	(
		REPORT_ID,
		NICKNAME,
		TITLE,
		CONTENT,
		REPORT_DATE
	) VALUES
	(
		#{reportId},
		#{nickName},
		#{title},
		#{content},
		DEFAULT
	)
	</insert>
	
	<select id="selectReportList" resultType="Report">
		SELECT *
		FROM REPORTS
	</select>
	
</mapper>
```

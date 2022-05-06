# 네이버 검색 API를 활용한 검색 결과 출력하기

* 기능
  * 키워드 검색
  * 페이지 단위 보기 

* 사용 기술
  * 네이버 오픈 API
  * 서블릿
  * 제이쿼리
  * JSON 파싱  

* 오픈 API 이용 신청
  * https://developers.naver.com/main/ 네이버 개발자 센터 접속

![image](https://user-images.githubusercontent.com/86938974/166629313-b039d296-5556-4eb1-a4ae-cde85bb885d0.png)

- 오픈 API 이용 신청

![image](https://user-images.githubusercontent.com/86938974/166629505-3455135c-ce39-4215-844c-e82a3fb65671.png)

- Client ID와 Clien Secret은 기억해둔다. 발급받은 키는 Application - 내 애플리케이션 에서 열람 가능

* API 문서 살펴보기
  * document - 검색
  * 네이버 블로그 검색 결과를 출력해주는 REST API
![image](https://user-images.githubusercontent.com/86938974/166629669-031153ec-8594-461e-9e66-010f387e9a26.png)

![image](https://user-images.githubusercontent.com/86938974/166629809-5daddead-5393-430e-9508-ccbc48fabb19.png)

* 서블릿 구현
![image](https://user-images.githubusercontent.com/86938974/167052381-84c6c128-3608-4d6a-b151-1cfc4d2978c5.png)
```
package api;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.net.HttpURLConnection;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLEncoder;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

@WebServlet("/NaverSearchAPI.do")
public class SearchAPI extends HttpServlet {
    @Override
    protected void service(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {
        // 1. 인증 정보 설정
        String clientId = "본인의 클라이언트 아이디";
        String clientSecret = "본인의 클라이언트 시크릿";

        // 2. 검색 조건 설정
        int startNum = 0;    // 검색 시작 위치
        String text = null;  // 검색어
        try {
             startNum = Integer.parseInt(req.getParameter("startNum"));
             String searchText = req.getParameter("keyword");
             text = URLEncoder.encode(searchText, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            throw new RuntimeException("검색어 인코딩 실패", e);
        }

        // 3. API URL 조합
        String apiURL = "https://openapi.naver.com/v1/search/blog?query=" + text
                        + "&display=10&start=" + startNum;  // json 결과
        //String apiURL = "https://openapi.naver.com/v1/search/blog.xml?query=" + text;  // xml 결과

        // 4. API 호출
        Map<String, String> requestHeaders = new HashMap<>();
        requestHeaders.put("X-Naver-Client-Id", clientId);
        requestHeaders.put("X-Naver-Client-Secret", clientSecret);
        String responseBody = get(apiURL, requestHeaders);

        // 5. 결과 출력
        System.out.println(responseBody);  // 콘솔에 출력

        resp.setContentType("text/html; charset=utf-8");
        resp.getWriter().write(responseBody);  // 서블릿에서 즉시 출력
    }

    private static String get(String apiUrl, Map<String, String> requestHeaders){
        HttpURLConnection con = connect(apiUrl);
        try {
            con.setRequestMethod("GET");
            for(Map.Entry<String, String> header :requestHeaders.entrySet()) {
                con.setRequestProperty(header.getKey(), header.getValue());
            }


            int responseCode = con.getResponseCode();
            if (responseCode == HttpURLConnection.HTTP_OK) { // 정상 호출
                return readBody(con.getInputStream());
            } else { // 에러 발생
                return readBody(con.getErrorStream());
            }
        } catch (IOException e) {
            throw new RuntimeException("API 요청과 응답 실패", e);
        } finally {
            con.disconnect();
        }
    }


    private static HttpURLConnection connect(String apiUrl){
        try {
            URL url = new URL(apiUrl);
            return (HttpURLConnection)url.openConnection();
        } catch (MalformedURLException e) {
            throw new RuntimeException("API URL이 잘못되었습니다. : " + apiUrl, e);
        } catch (IOException e) {
            throw new RuntimeException("연결이 실패했습니다. : " + apiUrl, e);
        }
    }


    private static String readBody(InputStream body){
        InputStreamReader streamReader = new InputStreamReader(body);


        try (BufferedReader lineReader = new BufferedReader(streamReader)) {
            StringBuilder responseBody = new StringBuilder();


            String line;
            while ((line = lineReader.readLine()) != null) {
                responseBody.append(line);
            }


            return responseBody.toString();
        } catch (IOException e) {
            throw new RuntimeException("API 응답을 읽는데 실패했습니다.", e);
        }
    }
}


```
- 애너테이션으로 요청명과 서블릿 매핑
- clientId와 clientSecret 변수에는 오픈 API이용 신청시 발급받은 값 붙여넣기
- 검색 시작 위치(startNum)와 검색어(keyword)를 매개변수로 받는다. 시작 위치는 정수로, 검색어는 한글 깨짐을 방지하기 위해 UTF-8로 인코딩 해준다.
- String apiURL은 검색 결과 데이터를 JSON으로 받기 위한 API이다. 검색어(text)를 쿼리스트링으로 보내는데, 여기에 display와 start 매개변수도 추가한다. display는 한 번에 가져올 검색 결과의 개수, start는 검색 시작 위치이다.
- 클라이언트 아이디와 시크릿을 요청 헤더로 전달해 API를 호출한다.

 * 검색 결과 확인

- 다음 주소로 접속하면 매개변수가 없어서 500에러가 발생한다.
![image](https://user-images.githubusercontent.com/86938974/167054021-815d3b2f-60ed-4b92-bbae-6fe557aaf4c1.png)

- 매개변수를 추가해서 다시 접속해본다.
![image](https://user-images.githubusercontent.com/86938974/167054076-e8c50797-82f6-4de0-afce-1df4427c02af.png)
- 검색결과가 JSON으로 출력된다.
![image](https://user-images.githubusercontent.com/86938974/167054135-6c5415ec-abba-495a-9284-58e005c7de61.png)

* 검색 결과를 웹 브라우저에 출력
 1. JSP에서 먼저 검색어를 입력한 후 [검색 요청] 버튼 클릭, Ajax 메서드를 통해 비동기 방식으로 서블릿에 전송
 2. 서블릿이 네이버 검색 API를 호출해 네이버에서 검색 결과를 JSON으로 반환하면 서블릿에서 직접 출력
 3. JSP는 이 내용을 콜백 메서드에서 받은 후 파싱하여 웹 브라우저에 출력
```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js">
</script>
<script>
// [검색 요청] 버튼 클릭 시 실행할 메서드를 정의합니다.
</script>
<style>
	ul{border:2px #cccccc solid;}
</style>
</head>
<body>
<div>
	<div>
		<form id = "searchFrm">
			한 페이지에 10개씩 출력됨 <br/>
			<select id = "startNum">
				<option value="1">1페이지</option>
				<option value="11">2페이지</option>
				<option value="21">3페이지</option>
				<option value="31">4페이지</option>
				<option value="41">5페이지</option>
			</select>
			<input type="text" id="keyword" placeholder="검색어를 입력하세요."/>
			<button type="button" id="searchBtn">검색 요청</button>
		</form>
	</div>
	<div class = "row" id="searchResult">
		여기에 검색 결과가 출력 됩니다.
	</div>
</div>
</body>
</html>
```
![image](https://user-images.githubusercontent.com/86938974/167054803-76def5cf-1ff7-42bf-8a90-28d8caf387e5.png)

```
<script>
// [검색 요청] 버튼 클릭 시 실행할 메서드를 정의합니다.
$(function()){
	$('#searchBtn').click(function(){
		$.ajax({
			url : "../NaverSearchAPI.do", // 요청 URL
			type : "get",				  // HTTP 메서드
			data : {					  // 매개변수로 전달할 데이터
				keyword : $('#keyword').val(), 					//검색어
				startNum : $('#startNum option:selected').val() //검색 시작 위치
			},
			dataType : "json", // 응답 데이터 형식
			success : sucFuncJson, //요청 성공시 호출할 메서드 설정
			error : errFunc 		// 요청 실패 시 호출할 메서드 설정
		});
	});
});

// 검색 성공 시 결과를 화면에 뿌려준다.
function sucFuncJson(d){
	var str = "";
	$.each(d.items, function(index, item){ 
		str += "<ul>";
		str += "	<li>" + (index+1) + "</li>";
		str += "	<li>" + item.title+ "</li>";
		str += "	<li>" + itme.description + "</li>";
		str += "	<li>" + item.bloggername + "</li>";
		str += "	<li>" + item.bloggerlink + "</li>";
		str += "	<li>" + item.postdate + "</li>";
		str += "	<li><a href='"+ item.link
				+"' target='_blank'>바로가기</a></li>";
		str += "</ul>";
	});
	$('#searchResult').html(str);
}
//실패 시 경고창
function errFunc(e){
	alert("실패: "+e.status);
}
</script>
```
- 검색 요청 버튼(id="searchBtn")을 눌렀을 때 $.ajax메서드 호출
- 이 메서드는 URL은 서블릿의 요청명을 사용하고, get방식 전송
- 매개변수로는 검색어와 검색 시작 위치를 전달하고 콜백 데이터 형식은 JSON으로 지정한다.
- $.each메서드를 사용하여 콜백 데이터 중 items 부분을 반복 파싱한다.items 요소에는 개별 블로그 정보가 원소로 들어간다.
- 파싱된 데이터는 10개씩 문자열에 누적 저장된 후 id가 "searchResult"인 영역에 HTML형태로 출력된다.

* 동작 확인
 * SearchView.jsp실행
 * 한국소프트웨어인재개발원을 검색어 입력란에 입력
![image](https://user-images.githubusercontent.com/86938974/167055957-8c30663c-277e-43d0-b45c-a11a451c8f11.png)
 
 * 검색 결과 총 10개 출력
![image](https://user-images.githubusercontent.com/86938974/167056072-5dd2a694-90c8-47ec-9b34-af4dc6f2bff6.png)

 * 2페이지 선택 후 검색하면 11번째부터의 검색 결과 볼 수 있음
![image](https://user-images.githubusercontent.com/86938974/167056133-1145a8f0-0a58-4d5e-b715-8473bec0797d.png)

* 핵심
 * 오픈 API 사용 -> API 제공자로부터 권한 얻기
 * 대다수 오픈 API가 데이터를 주고 받는 표준 포맷으로 JSON사용
 * 오픈 API를 통해 얻어오는 JSON데이터를 뷰에 적용하기 위해서는 비동기 통신이 필수 -> Ajax와 같은 메서드 사용 







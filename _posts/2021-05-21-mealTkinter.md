---
title: "학교 급식 알리미 프로그램"
excerpt: "학교 급식 정보를 GUI 프로그램으로 제작"
categories:
  - Python
last_modified_at: 2021-05-21T11:00:00-1:00:00
---

# 학교 급식 알리미 프로그램
이 파이썬 프로그램은 이때까지 했던 콘솔 창이 아닌 일반적인 프로그램과 같이 GUI 방식으로 실행됩니다. 여기서 GUI란? 간단하게 그래픽 유저 인터페이스로 그래픽으로 버튼이나 텍스트 상자 등이 만들어져서 그것들과 상호작용(클릭, 키보드 입력 등)하는 프로그램입니다.

## 1. API 란? & Key 발급
우리가 학교 급식 데이터를 얻는 방법에는 몇가지가 있습니다. 그 중에서도 우리가 사용할 방법은 나이스 학교 API 를 이용한 방법입니다. API에 대해 간단하게 알아보면 여러가지 기능들을 함수로 만들어서 프로그래머에게 재공해주는 부가 기능 같은 하나의 프로그램입니다.  
여기서 사용하는 API는 위의 정의와는 조금 다릅니다.(하지만 위에서 말한 API도 상식으로 알아두세요) 여기서 API는 웹에서 사용되는 것으로 서버(흔히 웹이라고도 부릅니다)에 요청을 하면 요청을 바탕으로 서버는 클라이언트(사용자)에게 응답을 보냅니다. 이러한 서비스를 나이스에서도 제공하고 있습니다. 바로 나이스 교육정보 개방 포털인데요. 여기서 학교 시간표, 학교 학사일정 등을 받을 수 있고 우리가 필요한 급식 정보도 받을 수 있습니다.
https://open.neis.go.kr/portal/mainPage.do
회원가입이 되어있지 않더라도 로그인에 들어가서 네이버, 구글, 페이스북 등으로 로그인 하고 간단한 약관 동의만 거치면 회원가입이 완료됩니다.
이제 활용가이드 -> 인증키 신청 으로 들어가서 제목은 "학교 동아리"로 하고 내용은 "학교 동아리에서 이용하기 위해 발급"으로 작성하고 인증키 발급 신청을 해줍니다. 그러면 알파벳과 숫자가 섞인 인증키가 발급됩니다.

## 2. 파이썬으로 요청해보기
먼저 모듈 하나를 설치해야 합니다. 명령 프롬프트(cmd)를 열고 아래의 명령어를 입력합니다.
> pip install requests  

설치 완료되었다면 본격적으로 파이썬 코드로 API 요청을 해보겠습니다.
먼저 나이스 API 요청을 하기 위한 서버의 주소입니다. 네이버 검색을 하다보면 같은 naver.com 에서도 / 뒤에 글자가 바뀌는 것을 볼 수 있습니다. 이를 GET 요청이라 하는데 이건 모르셔도 됩니다. 어쨌든 원하는 데이터를 얻으려면 / 뒤에 문자열에 자신이 필요한 데이터의 정보(학교 번호, 원하는 데이터의 이름 등)를 나이스에서 제공하는 규칙대로 써서 요청을 하면 됩니다. 죽전고등학교의 급식 정보를 요청할 때 사용하는 주소는 다음과 같습니다.
> https://open.neis.go.kr/hub/mealServiceDietInfo?KEY=API_KEY&Type=xml&ATPT_OFCDC_SC_CODE=J10&SD_SCHUL_CODE=7530525&MLSV_YMD=ymd 

이때 API_KEY에는 인증키가 들어가고 ymd에는 8자리로 된 날짜 ex) 20210521 가 들어갑니다. API_KEY 와 ymd 를 채우고 웹 브라우저 주소창에 입력을 해보면 정리되지 않은 데이터가 나올 겁니다. 이 데이터를 가공하는 방법을 배우기 전에 먼저 파이썬 코드에서 이 요청을 수행해보겠습니다.
``` python
import requests

API_KEY = "API_KEY" # 발급받은 API 키
ymd = input("날짜를 입력하세요: ")

url = "https://open.neis.go.kr/hub/mealServiceDietInfo?KEY="+API_KEY+"&Type=xml&ATPT_OFCDC_SC_CODE=J10&SD_SCHUL_CODE=7530525&MLSV_YMD="+ymd
res = requests.get(url) # 아까 간단하게 말했던 GET 요청을 수행합니다.
print(res.text)
```

## 3. XML이란? & XML 다루기
XML은 <> 로 이루어진 태그를 사용해서 데이터를 나타내는 방법입니다.  
```
<학교>
	<죽전고등학교>
		<1학년>
		</1학년>
		<2학년>
			<4반>
			</4반>
			<5반>
				<26>
					임성완
				</26>
			</5반>
			<6반>
			</6반>
		</2학년>
		<3학년>
		</3학년>
	</죽전고등학교>
</학교>
```
이런식으로 작성하고 </태그> 로 태그를 닫아야합니다.

아까 응답으로 받았던 급식 정보도 이러한 XML 형식입니다. 이제 이 XML 을 파싱, 즉 우리가 필요한 정보만 얻어보겠습니다.

``` python
import requests
from xml.etree import ElementTree as ET

API_KEY = "6bf84548b8d94278a5b974a33c2f4119"
ymd = input("날짜를 입력하세요: ")

url = "https://open.neis.go.kr/hub/mealServiceDietInfo?KEY="+API_KEY+"&Type=xml&ATPT_OFCDC_SC_CODE=J10&SD_SCHUL_CODE=7530525&MLSV_YMD="+ymd
res = requests.get(url)
root = ET.ElementTree(ET.fromstring(res.text)).getroot()
# ET.fromstring(): 문자열에서 ET 객체로 변환
# ET.ElementTree(): ET 객체를 XML 트리 구조로 변환
# getroot(): 가장 위의 태그를 이동

'''
아래 코드 설명
해당 날짜에 급식이 없을때 받는 응답:
<RESULT>
	<CODE>INFO-200</CODE>
	<MESSAGE>해당하는 데이터가 없습니다.</MESSAGE>
</RESULT> 
'''
if root.find("MESSAGE") != None: # MESSAGE 태그가 없지 않으면, 즉 존재할 때
	print("해당 날짜에 데이터가 없습니다.")

else: # MESSAGE 태그가 존재하지 않을 때, 즉 급식 정보가 있을 때
 	meal = root.find("row").find("DDISH_NM").text
 	# DDISH_NM의 데이터를 문자열로 얻음
 	meallist = meal.split("<br/>")
 	# <br/>를 기준으로 문자열을 나눠서 각각 급식을 리스트에 넣음
 	result = ""
 	for i in meallist:
 		result += i + "\n"
 	print(result)
'''
위의 코드 설명
해당 날짜에 급식 있을때 받는 응답:
<mealServiceDietInfo>
 	<row>
   		<DDISH_NM>
   			<![CDATA[j통새우튀김오므라이스1.2.5.6.9.10.12.13.15.16.<br/>j미소된장국5.6.8.9.13.<br/>j오이깍둑무침5.6.13.<br/>j파닭꼬치5.6.13.15.<br/>j깍두기9.13.<br/>j미니사과]]>
   		</DDISH_NM>
 	</row>
</mealServiceDietInfo>

따라서 루트 태그로 이동한 뒤 row 태그를 찾아서 이동하고 DDISH_NM 태그를 찾아서 이동하면 아래의 데이터가 나옵니다.
'''
```
그러면 학교 급식이 정상적으로 출력됩니다. 이제 마지막으로 콘솔 창이 아닌 GUI 창을 만들어보겠습니다.

## 4. Tkinter로 GUI 프로그램 만들기
``` python
import tkinter as tk
window = tk.Tk() # 창 생성
window.title("급식알리미") # 창의 이름을 급식알리미로 설정
window.geometry("640x400+100+100") # 창 크기 설정

entry = tk.Entry(window) # 텍스트 입력상자 생성
label = tk.Label(window, text="날짜를 8자리로 입력해주세요 ex)20210329") # 텍스트 출력(입력 불가능)
button = tk.Button(window, text="제출") # 버튼 생성

# 위에서 아래로 위젯들 배치
entry.pack() 
button.pack()
label.pack()

window.mainloop() # 창 닫기 버튼 누를때까지 실행
```
이렇게 하면 간단한 GUI 프로그램이 완성됩니다. 기능은 없지만 만들어보는 것에 의의를 두겠습니다.
이제 아까 했던 내용들을 다 합쳐서 프로그램을 완성하겠습니다.
## 5. 완성
완성된 코드는 이곳에 올리면 글이 너무 길어지기 때문에 제가 올려둔 사이트의 링크를 걸겠습니다.
> https://github.com/JH-CodeMe/mealTkinter/blob/main/meal.py

수고하셨습니다.

> 작성자: 임성완

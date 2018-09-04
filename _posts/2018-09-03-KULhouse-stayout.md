---
title: KULhouse 주말 자동 외박 신청 프로그램
categories: Toy Project
tags: Project python aiohttp
---

# 제작 동기
방학 1주일 남기고 할만한 거 없나 찾아보다가, 미루고 미뤘던 기숙사 자동 외박 신청 코드를 짰다.  
건대 기숙사 쿨하우스의 경우 당일 외박 신청을 다음날 0시 전까지 제출하면 다음날 오전 5시까지 자유롭게 기숙사 출입이 가능하다.  
보통 주말의 경우 주말 외박이라고 해서 금요일부터 일요일까지 외박 신청을 할 수 있는데 횟수에 제한이 없다.  
나 같은 경우 금공강이라 금토일 늦게까지 카페에 있는 등 밖에 있는 경우가 많은데  
한꺼번에 해 놓으면 굳이 금요일마다 귀찮게 쿨하우스 어플 켜고 주말 외박 신청할 필요가 없게 된다.

# Login & Session
`python 3.6`으로 작성했기에 이에 맞춰서 정리할 생각이다.  

우선 로그인을 통해 쿨하우스 입주생임을 인증하고, 로그인 정보가 어디로 POST전송 되는지 알아야 한다.  
직접 로그인을 해보며 `Chrome 개발자 도구` 중에 network 탭을 참고하여 어디로 정보가 가는지 알아본다.  

![02](https://user-images.githubusercontent.com/30731518/44987917-de6d7900-afc3-11e8-8750-5ca29bd29c47.png)

`login_ok.asp`라는 파일 다음으로 `mypage00.asp`라는 파일을 불러온다.  
일단 `login_ok.asp`라는 파일을 살펴 본다.  

![03](https://user-images.githubusercontent.com/30731518/44988061-5dfb4800-afc4-11e8-91bf-17412c7541f6.png)

Request URL이 주황색으로 밑줄 친 부분이고, Request Method가 POST 방식인 것으로 보아 아무래도 로그인 정보는 저 URL로 POST 방식으로 보내지는 것 같다.  
확실하게 확인하기 위해 `Form Data`를 본다.

![04](https://user-images.githubusercontent.com/30731518/44988079-6c496400-afc4-11e8-8cfd-45edca9f5d53.png)

`Form Data`에서는 `mode`, `url`, `std_no`, `pwd`를 넘긴다.  
각각의 정보를 하나의 data로 묶어서 위의 URL로 data를 POST로 넘긴다.  

실제 github에 올린 코드에서는 `config.ini` 파일 안에 로그인시 필요한 학번과 비밀번호를 적고 `configparser`를 이용하여 .ini파일을 불러온 뒤 파싱하도록 했다.

그리고 header의 경우 `'Content-Type': 'application/x-www-form-urlencoded'`과 `'User-Agent'`를 명시하여 적었다.  
(special thanks to Sunyeop Lee)
처음에 'Content-Type'을 제대로 명시하지 않아 실수했던 부분이고, [이 곳](https://gist.github.com/jays1204/703297eb0da1facdc454)이 많은 도움이 되었다.  

그리고 `url encoding`도 함께 해주어야 한다면, `urllib`의 `qiote`를 이용할 것.  

만약 올바르게 request를 보내면 `mypage00.asp`로 올바르게 리다이렉트 될 것이고, response가 제대로 왔는지 이를 통해 확인할 수 있다.

POST 요청은 여러 요청을 보내야 하므로 비동기 처리를 위해 `aiohttp`를 사용했다.  
밑에서 좀 더 자세하게 정리할 생각이다.  

# Stayout Application
로그인에 성공하였고, 세션도 올바르게 유지된다면 원하는 날짜와 기간에 맞춰 해당 URL로 다시 POST 요청을 보낸다.

![05](https://user-images.githubusercontent.com/30731518/44989357-8be28b80-afc8-11e8-849e-2358a5fa948f.png)

![06](https://user-images.githubusercontent.com/30731518/44989365-943ac680-afc8-11e8-8bae-4e3a0ec5200e.png)

외박신청 페이지로 들어가 알맞게 form을 작성하고 확인 버튼을 누르면 신청되었다는 confirm창이 뜬다.  
이를 기준으로 개발자 도구를 열어 패킷을 분석한다.  

![07](https://user-images.githubusercontent.com/30731518/44989372-9b61d480-afc8-11e8-80ea-2be487ca9c8c.png)

`mypage00_01_proc.asp`라는 파일 다음으로 `mypage00_02.asp`라는 파일을 불러온다.  
일단 `mypage00_01_proc.asp`라는 파일을 살펴 본다.  
위의 로그인 처리와 비슷한 구성임을 어렴풋이 느낄 수 있다.  

![08](https://user-images.githubusercontent.com/30731518/44989524-217e1b00-afc9-11e8-9d93-061152d450b9.png)
Request URL과 Request Method로 보아 외박 신청에 대한 데이터는 저 URL로 POST 요청하는 것을 짐작할 수 있다.  
정확하게 알아보기 위해 마찬가지로 `Form Data`를 살펴본다.  

![09](https://user-images.githubusercontent.com/30731518/44989526-2216b180-afc9-11e8-8179-fbecb021d565.png)
요구하는 정보가 많다.  
각각의 값들이 어떤 정보를 의미하는지 정확하게 알아보기 위해서 외박 신청 페이지의 form을 보면서 비교했다.  

우선 `ov_reason`은 주말 외박을 의미하고 ,`sdate`와 `edate`는 각각 신청 시작 날짜와 신청 마지막 날짜를 의미한다.  
`memo`는 신청 사유를 나타낸다.

그리고 `id_no`의 경우 개개인마다 다르므로 특정 페이지에서 파싱하여 가져와야 할 것이다.
이 부분은 `Beautiful Soup`를 이용하여 처리했다.  

그 밖에 `recruit_year`나 `recruit_code` 등등은 그대로 유지했다.  
github에 올려놓은 코드에서는 `recruit_code`역시 `id_no`와 마찬가기로 html parsing하여 정보를 얻어냈다.


위의 두 과정을 코드로 나타낸 `apply_stayout` 함수이다.  

~~~python

async def apply_stayout(sdate, edate):
    async with aiohttp.ClientSession() as session:

        user = Kuluser()

        async with session.post(login_url, data=user.login_payload, headers=header) as login_response:
            async with session.get(info_url, headers=header) as response :
                user.get_info(await response.text())

        user.stayout_payload['sdate'] = sdate
        user.stayout_payload['edate'] = edate

        async with session.post(stayout_url, data=user.stayout_payload, headers=header) as response:
            result = await response.text()
            print("{sdate}부터 {edate}까지 신청이 완료 되었습니다\n".format(sdate=sdate, edate=edate))

~~~

POST 요청 시 보내는 data 및 header, .ini 파일 처리하는 함수 등등은 `Kuluser`라는 `class` 모듈로 따로 빼서 정리했다.  
그리고 `aiohttp`를 이용하기 위해 `async` 처리한 `apply_stayout` 함수는 `sdate`와 `edate`를 파라미터로 받아서 외박 신청 시 필요한 form data에 넣는다.  

~~~python
loop = asyncio.get_event_loop()

tasks = []

//기숙사 거주 기간
start = datetime.datetime(year=2018, month=8, day=24)
day = start
final = datetime.datetime(year=2019, month=2, day=19)


while (day < final):
    sdate = day.strftime("%Y-%m-%d")
    edate = (day + datetime.timedelta(days=2)).strftime("%Y-%m-%d")

    task = asyncio.ensure_future(apply_stayout(sdate, edate))
    tasks.append(task)

    day = day + datetime.timedelta(days=7)

loop.run_until_complete(asyncio.wait(tasks))
~~~

7일 간격으로 날짜를 조절하면서 while문을 돌리고, 각 task에 대해 비동기 처리를 해준다.  

# etc...
코드는 github 페이지에 올려두었고, 여기에서는 어떻게 했는지에 대해서만 간단히 정리했다.

아쉬운 점이나 개선해야 할 점 몇 가지 적어보자면,
1. 모바일 패킷 따서 하는 것도 괜찮았을 것 같다.
2. 생각했던 것 보다 코드가 느리다. Beautiful Soup 사용해서 그런 것 같은데 시간 날 때 원인 분석 좀 해봐야 겠다.
3. 개선해서 위치 추적하고 다음날 0시 전까지 기숙사 아니면 자동으로 외박 신청해주는 것... 만들면 괜찮을 것 같다.



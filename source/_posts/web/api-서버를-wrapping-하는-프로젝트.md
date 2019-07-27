---
title: 'API 서버를 wrapping 하는 프로젝트'
date: 2018-11-25 21:43:08
tags:
    - api 서버  
    - client 서버  
---

API 서버를 하나 구현하고, URL과 document만 제공해주면  
사용하는 쪽에서 network 통신하는 객체를 또 만들어야 함으로 불편함이 뒤따른다.  

이런 불편함을 막기 위해 API 서버를 wrapping한 프로젝트를 만들고,  
(spec을 OAS 3.0으로 작성했다면 openapi generator plugin 같은 것을 사용하여 간단하게 만들 수 있다)  
이런 일련의 과정들(header나 parameter 세팅하고, api 호출하고, 결과값을 변환해서 돌려주는 행위)을 다 수행하도록 한다.  
그리고 이 SDK를 외부에 배포하는 것이다.  

![Api Wrapping](https://cloud2.zoolz.com/MyComputers/Images/Image.aspx?q=bT00MDcyNDcma2V5PTMwMjA5MzIxNDgmdHlwZT1sJno9MjAxOS8wMS8wOCAxODoyNg==)

이러면 사용하는 쪽에서 훨씬 개발을 편리하게 할 수 있다.  
부가적인 요소에 신경 쓸 필요없이 메서드만 호출하고, 리턴되는 결과에만 집중하면 되기 떄문이다.  
(제공해주는 쪽은 일이 늘어나지만...)  

외부에서 사용할 경우 그냥 라이브러리를 제공해주면 되고,  
내부에서 사용할 때는 nexus 같은 레파지토리에 올려서 다운받아 사용하게 하는 것이 좋다.  

<!-- more -->
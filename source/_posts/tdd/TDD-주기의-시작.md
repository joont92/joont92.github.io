---
title: '[tdd] TDD'
 주기의 시작
date: 2019-03-22 00:26:34
tags:
    - 테스트 주도 개발로 배우는 객체지향 설계와 실천
---

되돌아보면 우리는.. 개발을 다 진행해놓고 마지막에 통합하면서 피드백을 받았던 것 같다.  
이 마지막 순간에 변경되는 부분도 굉장히 많고, 서로가 잘못 이해한 부분도 많이 발생한다.  
최악의 경우는 작업을 뒤집거나, 누군가가 포기하고 가야하는 상황도 발생한다.  

그러므로 앞서 얘기했듯이, **빠른 피드백을 받을 수 있게끔 시스템을 구성하는 작업(중첩된 피드백 고리)은 매우 중요**하다.  
- 위에서도 언급했듯이, 이런 시스템의 구성을 나중으로 미루는 것은 매우 위험하다  
- 저 정도 예시면 양반이지, 배포할 수 없어서 프로젝트가 취소되거나 마지막에 테스트를 추가했는데도 불구하고 오류율이 너무 높아 시스템이 폐기되기도 한다(저자의 경험)  
- **피드백은 아주 근본적인 도구이며, 올바른 방향으로 나아가고 있는지 최대한 미리 파악하고자 한다**  

그 중에서도 외부와의 피드백 고리를 형성하는 것은 중첩된 피드백 고리를 구성하는 첫번쨰 단추이고, 가장 중요하다.  
이 말인 즉 `자동화된 빌드・배포・테스트 주기 전체`를 가장 처음부터 구현해야 함을 의미한다.  
근데 사실 말이 쉽지, 이렇게 구성하기에는 할일도 매우 많고, 어렵다.  
그러면 우리는 무엇부터 시작해야 할까?  

# 우선 동작하는 골격을 대상으로 테스트하라  
- 먼저 `동작하는 골격`을 만들어야 한다  
    - 전 구간을 대상으로 자동 빌드・배포・테스트를 할 수 있는 실제 기능을 가장 얇게 구현한 조각을 말한다  
    - 첫 기능을 구현할 수 있을 정도의 자동화, 주요 컴포넌트, 통신 메커니즘이 포함될 것이다  
- 이 구조를 이용해 `유의미한 첫 기능에 대한 인수 테스트`를 작성한다  
    - 이후로는 시스템의 나머지 부분을 대상으로 테스트 주도 개발을 진행할 수 있게 모든 것이 제자리에 놓일 것이다  

> 동작하는 골격을 만드는 과정에 배포 단계를 포함한 이유는 아래와 같다  
> - 배포 단계는 오류가 발생하기 쉬운 활동이므로 실제 환경에 배포해야 할 때 까지 스크립트를 철저히 검증해야 하기 때문이다  
> - 배포 과정을 자동화 하는 것 만큼 프로세스를 이해하는데 도움이 되는 것은 없다  
> - 배포 단계에서 개발팀이 조직의 다른 부문과 접촉하기도 하며, 실제로 어떻게 운영되는지도 배워야하기 때문이다  

참고로 동작하는 골격을 만들때는 골격의 구조에만 집중하고, 테스트가 최대한 표현력을 갖추게끔 테스트를 정리하는 일에 대해서는 크게 신경쓰지 않는다  
동작하는 골격과 그것을 보조하는 기반 구조는 테스트 주도 개발을 시작하는 방법을 도와주기 위해 존재하는 것이기 때문이다  

# 동작하는 골격의 외형 결정  
어떤 전체 구조에 관한 구상 없이는 빌드・배포・테스트 주기를 자동화 할 수 없으므로,  
계획한 첫 출시를 달성하는데 필요할 주요 시스템 컴포넌트와 그러한 컴포넌트의 상호 작용 방식에 대한 대략적인 그림은 필요하다.  
> 이는 화이트보드에 몇분만에 그릴 수 있다  
> 공개된 장소에 이런 구조를 그려놓으면 코드 작성 시 팀이 업무 방향을 참고하는데 도움이 된다(마파 문디)  

당연하지만, 이 같은 초기 구조를 설계하려면 시스템의 목적을 어느 정도 이해해야 한다  
**클라이언트의 요구사항(기능적 요구사항, 비기능적 요구사항)을 고수준 관점에서 바라보고 의사결정에 참고해야 한다**  
그렇지 않으면 위험을 무릅쓰고 하는 일들이 죄다 의미가 없어진다  

![테스트 주도 주기](https://lh3.googleusercontent.com/Sclm2o15GcyE3_jdothc4VLXpeBP_ewwY96loaiD8h9xd3QaDf9DaRP-tHaahWKKGALMbm1V6suovTMgh1QJSR5uFh13mAmt9MMDHis_U8uaWoKj_krT4GtoHoEkPMeIKk5KGbSzOpbso31HabNC92LGtKRabQmHVTmkBwsg6iedYZDOEGeOA_MHvuH9K3qQGnUv12qE6XXlWrkRv3BmPnZTD52jbAtpDyIXDcqOlQ7Y4XXHzgzb8IvRRPTr4OSIB1xczYJ3-MU8Cq3Sd6byjjl0pYz_XlTsgKySHNtfA4RLkCZ-sKBx66sgXwUkqcnYE__bZpjlfg6onHnXirq-TzJOamVOxMxjgZzFIeHHaamQYP2Bn4_4rivwGis_sRGoCyhXbm0bXUFJxddbL_z7VLtxtD8DFPjljZXVdeo7k64Qc7JFo2CDSmjSapAyZ9VhzVLyIVq0t3JX0tgSf6kFY4IXRew9eHaooRywVVeYlo2-NbNzAOsOJOw_BBQMxSnLD-JuY1rsfQNHvJfDp-hiLbKK4x-WA1x9m-uaLkGDu5y8_EgJICKhDMTACTohQVKJkp65i62ggMbqGC-YPRcr9pKmSZqECI_9oHTclcs_SZW9xjVIAbCMlPOe04iPHxAdR-HY6fsoLMVGflZY16nPyHfV28e1BNM=w882-h229-no)  

참고로 이를 `과도한 사전 설계`와 혼동하지 않아야 한다  
실제 피드백을 토대로 배우고 개선해나가는 과정을 시작할 수 있게(+TDD 주기를 시작할 수 있게)하는데 필요한 최소한의 의사결정만을 내리는 것이 좋다  
(현재 생각하고 있는 바가 틀릴 가능성이 있으므로, 시스템이 성장해가면서 세부 사항을 파악해나간다)  

# 피드백 소스 구축  
아래의 글로 `피드백의 중요성`을 재고하고 있다  
> **어플리케이션 설계에 대해 내린 의사결정이나, 어플리케이션 설계가 근거하는 가정의 옳고 그름에 대해서는 아무것도 보장할 수 없다**  
> 그저 최선을 다할 뿐이며 현재 밟고 있는 절차에 피드백을 적용해 최대한 빨리 의사결정이나 가정을 검증하는데 의지할 뿐이다  

아래는 위의 `테스트 주도 주기`가 주는 피드백에 대해 그려놓은 그림이다  

![요구사항 피드백](https://lh3.googleusercontent.com/GEOVJp8MMcArH1mGIoFPjC8uLNKczgp_SrDmxR4NnecXuHRDYgmbtJannmt58E4tuF3A0zo71_TKsIr3AqQwov7Iy_rB0Un00m_ZCiSiQ4uA0ogb77eIoMxfh4iow95OTCFfe5P5RQLXqs8Lt5eaNM0jsS9_U9nYP43DK019lzYn2RGavF1OseqqoxRYQ9kge5czEX7M7NH_XXdOj2QAXcx_Dot70025msdldoyWHjRDFjItSJFP_evjWZcpc95dqxqbM0cWXBYG0Ifyvy_TwZTIQfV1dsveRhSfNWfhcP2Tj6yMWrmUSz2HZjGZzlQoxP6W-DV-bPFeK2C7hgbVe5qMf37g-_4RGtt-RZIEcmSWXqbT9Bgm1TT0YULSzkxlx3lLTtcZ-SdeEW6K02LlIGb7cikJkfD4y5WdGDzB_AfdryWeW1ntA0jttjOe6lY4RmT2ZNbw292FQ01SfcDegLZ7kRn4F3bON7Jv99f8o34WGJAsaWcEd9VT73ur7lau-4NUVYriJNyCmBvMiLByJt1uv5GYdUKdQ4kl_LHIHp6motBXN8lWDCphg5IVVvf48NtPo6iUBryRtWEiTKj67PU5wLOVSvNY2Q8RTRcJFkSnFXXmJkWDWvtNeiLWYV04bTVe_8WFCJ2EK7DnvE9Vs4FOZ92Z5bA=w872-h351-no)  

보다시피 각 과정은 서로에게 피드백이 된다  
이러한 시스템을 구축함으로써 우리의 기능이 요구사항에 얼마나 부합하는지, 우리 시스템 구현은 어떤지 빠르게 평가할 수 있고, 빠르게 반영할 수 있다.  
`최종적으로 우리는 안전해지고, 완전한 소프트웨어를 얻게 될 것이다!!!`  

> 또 하나의 가장 큰 헤택은 `뭘 배우늗 거기에 맞춰 시스템을 변경할 수 있으리라`는 점이다  
> 뭐든 테스트를 먼저 작성한다는 것은 `철저한 회귀 테스트 모음을 갖게 된다`는 것을 의미하기 때문이다  
> 물론 어떤 테스트도 완벽하지 않지만, 견고한 테스트 모음이 있으면 중대한 변경을 안전하게 할 수 있다는 사실은 확실하다  

--- 

처음에는 이러한 순서대로 점진적인 개발을 하는 과정이 불안정하고, 활동량도 굉장히 많을 것이지만 자동화가 구축되고 나면 반복적인 과정으로 안정화된다  

> 나중에 통합 절차를 수행하는 프로젝트는 침착한 분위기에서 시작하지만, 처음으로 시스템 통합을 시도하게 되는 후반부에 힘들어지곤 한다  
> 통합을 나중에 하는 방식의 경우 어떤 일이 일어날지 예측하기 불가능한데, **팀에서 엄청나게 많은 각 부분을 제한된 시간에 짜맞춰야 하고 실패한 부분을 고치는 일도 감안해야 하기 때문이다**  
> 그래서 경험이 풍부한 이해관계자는 프로젝트 초반부에 생기는 불안정성에 고약하게 반응한다  
> 후반후에 들어가면 상황이 더 악화되리라 예상하기 때문이다  
> **비단 통합 뿐만이 아니다. 우리는 각종 테스트들을 먼저 작성함으로써 혼란을 미리 잡고가게 되는 효과를 가지게 된다.**  

![테스트 먼저 vs 테스트 나중에](https://lh3.googleusercontent.com/WZOowcX2_1KjkKHzNQhCF-x0SiMlIFZjuAT1oTsZSwYee0zBcPrvggU64AN952M86987WyWhq1InTB8_z3bOo8VZ04NOP4dPOV5F2b4r-lcmrDqEbLQ6KESACL19EWfShx9DQAFRsKkBfXH363hIKphswwLpJo-u_AEWFZEOTwfD7Di9YaO33Z-Mo80GJhJddpK323W_VrSqvZq7TzAcQuGLsyxdd-OCkh1tA5ZsKmyWqYlgY0s_yBwZUBNPqwli5X4um6O200d-FtifNT4P5Xsh0r02jWW6RWuumJToqbweuOzDa8fN7UQzFViY3DF1Cl9ZdXLFp-HdtgubP74pTGoYGm7rU8ZIdqKt_xkDNdkbGjI8Cf0xiVtWAco1XeSagWDwjvB0O3lAm_cGBXBp-tl22qFlpfMwy8PF1UsvnnNLZ7NBNCrTxOO4gsen0loemkBHmZn3I5k4wMK2fgxNuAlTK-ninGRa8gHK5sXVjK-YOYHZIrU0Di6BPvBRFUKDmxu62ZbSR9G0Dld4sPD8DlihlZNtCiRnA1Z_7XX7B0mX1gVTk9xw34cptI8XTaRBFT5MQKZfymrnf8TKzAHFP0HZm5nb19hG0E5UOvx0CqBriHuPgulR4E0shj66nB8AqM59zlFQnpZt5eXVzniSMq7S0UO4N7Y=w960-h720-no)

가장 중요한 것은 방향 감각을 확보하는 것과 가정을 테스트할 구체적인 구현을 갖춰야 한다는 것이다  
`동작하는 골격`은 프로젝트 초기에 각종 쟁점을 드러내지만, 프로젝트 초기라면 아직까지 그러한 쟁점을 해결할 시간과 예산, 의지가 있을 때이다  

<!-- more -->
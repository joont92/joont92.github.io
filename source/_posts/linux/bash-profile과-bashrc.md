---
title: bash_profile과 bashrc
date: 2017-12-26 18:07:27
tags:
---

리눅스에서 alias를 수정하거나 PATH를 변경할 떄 접하게 되는 대표적인 4가지의 파일들..

```
/etc/profile, /etc/bashrc, ~\.bash_profile, ~\.bash_rc
```

각각의 파일에 대해 간단하게 알아보겠다.

일단 이에 대해 알기전에 **Login Shell**과 **Non-Login Shell**의 차이에 대해 알아야 한다.

##### Login Shell
Shell을 실행할 때 로그인이 필요한 경우를 말한다.
ssh로 접속하거나, su 명령어로 다른계정을 들어갈 때 등이 해당된다.
> \/etc\/profile, ~\/.bash_profile 파일이 이 Shell이 뜰 때 실행되는 파일이다.

##### Non-Login Shell
Shell을 실행할 떄 로그인이 필요하지 않은 경우를 말한다.
즉 Shell이 실행되는 모든 상황을 의미하게 됩니다.
GUI에서 터미널을 띄울때나, bash 명령어로 다시 bash를 실행하는 경우 등이 해당된다.
> \/etc\/bashrc, ~\/.bashrc 파일이 이 Shell이 뜰 때 실행되는 파일이다.

Non-Login Shell은 Login Shell을 포함한다.
Login Shell이 실행될 때 profile과 bashrc 파일이 모두 실행되게 되고,
Non-Login Shell이 실행될 때 bashrc 파일만 실행되게 된다.

### profile
\/etc\/profile 파일의 경우 전역적인 파일로 모든 사용자가 로그인 시 실행되며,
~\/.bash_profile 파일의 경우 지역적인 파일로 해당하는 사용자가 로그인 시만 실행된다.
또한 \/etc\/profile의 경우 어떠한 shell이든 상관없지만, ~\/.bash_profile의 경우 bash shell일 경우에만 해당된다.

### bashrc
profile과 달리 Login 과정이 없으므로 shell을 실행시키는 사용자로 구분한다.
\/etc\/bashrc의 경우 모든 사용자가 shell을 실행시킬 때 마다 실행되며,
~\/.bashrc의 경우 해당하는 사용자가 shell 실행시킬 때 실행된다.

> profile의 경우 대부분 환경 변수같은 것을 명시하고 bashrc의 경우 alias 같은 것을 명시한다..(?)

<!-- more -->
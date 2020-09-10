# IntelliJ
> 인텔리제이를 사용하면서 겪은 문제 및 해결 방법

## Contents


## build 시 OOM(Out Of Memory) 또는 GC overhead limit exceeded 발생시

### 상황
Spring Framework 프로젝트를 IntelliJ에서 빌드할 때 OOM 또는 GC overhead limit exceeded, heap size 초과 등 메모리 초과 관련 에러가 발생한 상황

### 해결방법
가장 간단한 방법은 IntelliJ 설정 중 빌드 heap 크기를 늘린다.

Preferences -> Build, Execution, Deployment -> Complier

![해당 Preferences](./images/preferences_complier.png)

위 그림에서 Build process heap size(빨간 네모) 옆에 숫자를 더 큰 크기로 늘린다.

주의할 점은 기본 설정이 다시 빌드할 때 초기 값으로 초기화되는 듯 하다.
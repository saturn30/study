파드를 단독으로 만들면 파드에 어떤 문제가 생겼을 때 자동으로 북구 되지 않는다.

이러한 파드를 정해진 수만큼 복제하고 관리하는 것이 레플리카 셋이다.

### 동작 과정

1. 레플리카셋 컨트롤러는 레플리카셋 조건을 감시하면서 현재 상태와 원하는 상태가 다른 것을 체크
2. 레플리카셋 컨트롤러가 원하는 상태가 되도록 파드를 생성하거나 제거
3. 스케줄러는 api 서버를 감시하면서 할당되지 않은 파드가 있는지 확인
4. 스케줄러는 할당되지 않은 새로운 파드를 감지하고 적절한 노드에 배치

레플리카셋은 레플리카셋 컨트롤러가 관리하고 파드의 할당은 여전히 스케줄러가 관리한다.

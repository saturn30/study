Deployment는 레플리카 셋을 이용하여 파드를 업데이트하고 이력을 관리하여 롤백 하거나 특정 버전으로 돌아갈 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo-deploy
spec:
  replicas: 4
  selector:
    matchLabels:
      app: echo
      tier: app
  template:
    metadata:
      labels:
        app: echo
        tier: app
    spec:
      containers:
        - name: echo
          image: ghcr.io/subicura/echo:v1
```

yaml 포맷은 레플리카 셋과 동일함

- selector - 라벨 체크 조건
- replicas - 원하는 파드 개수
- template - 생성할 파드 명세

디플로이먼트는 새로운 이미지로 업데이트하기 위해 레플리카 셋을 이용한다. 버전을 업데이트하면 새로운 레플리카 셋을 생성하고 해당 레플리카 세이 새로운 버전의 파드를 생성한다.

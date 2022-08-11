# Pod

[https://subicura.com/k8s/guide/pod.html](https://subicura.com/k8s/guide/pod.html)

Pod는 쿠버네티스에서 관리하는 가장 작은 배포 단위이다. Pod는 한 개 또는 여러 개의 컨테이너를 포함한다.

### Pod의 생성 과정

1. 스케줄러는 api 서버를 감시하면서 할당되지 않은 파드가 있는지 확인
2. 스케줄러는 할당되지 않은 파드를 감지하고 적절한 노드에 할당
3. 노드에 설치된 kubelet은 자신의 노드에 할당된 파드가 있는지 확인
4. kubelet은 스케줄러에 의해 자신에게 할당된 파드의 정보를 확인하고 컨테이너 생성
5. kubelet은 자신에게 할당된 파드의 상태를 api 서버에 전달

노드가 여러 개가 되더라도 스케줄러만 열심히 일하면 문제없는 구조임.

### YAML로 설정파일 작성

```yaml
apiVersion: v1
kind: Pod
metadata:
	name: echo
	labels:
		app: echo
spec:
	containers:
		- name: app
			image: imagename
```

version, kind, metadata, spec은 리소스를 정의할 때 반드시 필요하다.

### 컨테이너 상태 모니터링

쿠버네티스는 컨테이너가 생성되고 서비스가 준비되었다는 것을 체크하는 옵션을 제공하여 초기화하는 동안 서비스되는 것을 막을 수 있다.

- livenessProbe
  - 컨테이너가 정상적으로 동작하는지 체크하고 정상적으로 동작하지 않는다면 컨테이너를 재시작하여 문제를 해결한다.
  - 여러 가지 방식으로 정상 동작인지 체크할 수 있는데 웹의 경우 http get 요청을 보내 확인하는 방법을 사용함
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: echo-lp
    labels:
      app: echo
  spec:
    containers:
      - name: app
        image: ghcr.io/subicura/echo:v1
        livenessProbe:
          httpGet:
            path: /not/exist
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 2 # Default 1
          periodSeconds: 5 # Defaults 10
          failureThreshold: 1 # Defaults 3
  ```
- readinessProbe
  - 컨테이너가 준비되었는지 체크하고 정상적으로 준비되지 않았다면 Pod로 들어오는 요청을 제외한다.
  - livenessProbe와 차이점은 문제가 있어도 Pod를 재시작하지 않고 요청만 제외한다는 점이다.
- livenessProbe + readinessProbe
  - 보통 둘 다 같이 적용한다.
  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: echo-health
    labels:
      app: echo
  spec:
    containers:
      - name: app
        image: ghcr.io/subicura/echo:v1
        livenessProbe:
          httpGet:
            path: /
            port: 3000
        readinessProbe:
          httpGet:
            path: /
            port: 3000
  ```

### 다중 컨테이너

대부분 1파드에 1컨테이너가 들어가지만 여러 개의 컨테이너가 들어가는 경우도 있다.

하나의 Pod에 속한 컨테이너는 서로 네트워크를 로컬 호스트로 공유하고 동일한 디렉터리를 공유할 수 있다.

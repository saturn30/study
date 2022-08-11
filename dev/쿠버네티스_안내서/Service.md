# Service

파드는 자체 IP를 가지고 다른 파드와 통신할 수 있지만, 쉽게 사라지고 생성되는 특징 때문에 직접 통신하는 방법은 권장되지 않는다. 쿠버네티스는 파드와 직접 통신하는 방법 대신, 별도의 고정된 IP를 가진 서비스를 만들고 그 서비스를 통해 파드에 접근하는 방식을 사용함.

외부 요청 → 서비스 → 다수의 파드 중 하나로 요청이 전달됨

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis
spec:
  selector:
    matchLabels:
      app: counter
      tier: db
  template:
    metadata:
      labels:
        app: counter
        tier: db
    spec:
      containers:
        - name: redis
          image: redis
          ports:
            - containerPort: 6379
              protocol: TCP

---
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  ports:
    - port: 6379
      protocol: TCP
  selector:
    app: counter
    tier: db
```

spec.ports.port : 서비스가 생성할 포트

spec.ports.targetPort : 서비스가 접근할 파드의 포트 (기본값은 위와 같음)

spec.selector : 서비스가 접근할 파드의 라벨 조건

### 서비스 종류

- ClusterIP
  - 기본 타입. 클러스터 내부에서만 사용 가능하다
- NordPort
  - 클러스터 외부에서 접근 가능.
- LoadBalancer
  - NordPort의 단점은 노드가 사라졌을 때 자동으로 다른 노드를 통해 접근이 불가능하다는 점이다.
  - 자동으로 살아 있는 노드에 접근하기 위해 모든 노드를 바라보는 로드밸런서가 필요하다.
  - 브라우저는 NordPort에 직접 요청을 보내는 것이 아니라 로드 밸런서에 요청하고 로드 밸런서가 알아서 살아 있는 노드에 접근하면 노드 포트의 단점을 없앨 수 있다.

실전에선 서비스보다 인 드레스를 주로 사용함.

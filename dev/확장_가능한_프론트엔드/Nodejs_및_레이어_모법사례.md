# NodeJS 및 레이어 모범사례

[https://blog.codeminer42.com/nodejs-and-good-practices-354e7d763626/](https://blog.codeminer42.com/nodejs-and-good-practices-354e7d763626/)

소프트웨어는 항상 변경될 수 있다.

좋은 코드를 판단하는 한 가지 기준은 쉽게 변경할 수 있는지이다.

### 관심사와 책임의 분리

“같은 이유로 변경되는 것들을 모으고, 다른 이유로 변경되는 것은 분리하라”

함수, 클래스 또는 모듈이 무엇이든 단일 책임 원칙과 관심사 분리에 적용할 수 있다.

이러한 원칙을 기분으로 소프트웨어를 설계하는 것은 아키텍처에서 시작된다.

### 아키텍처

소프트웨어 개발에서 책임은 단일성을(? 단일한 목적인 듯) 달성하기 위해 전념하는 작업이다.

일반적으로 아키텍처는 도메인, 애플리케이션, 인프라 및 입력 인터페이스의 4개 계층으로 분할되는 경향이 있다.

### 도메인 레이어

이 계층에서 엔티티 및 비즈니스 로직을 수행하고 도메인과 직접적인 관계가 있는 단위를 정의할 수 있다. 소프트웨어에서 가장 격리되고 중요한 계층이며, 애플리케이션 계층에서 유즈케이스를 정의하는 데 사용할 수 있다.

### 애플리케이션 레이어

애플리케이션 레이어는 앱의 실제 동작을 정의하므로 도메인 계층 단위 간의 상호 작용을 수행한다.

인프라 계층에 대한 어댑터로도 사용할 수 있다.

### 인프라 계층

가장 낮은 계층이며 우리 애플리케이션 외부에 있는(db, 이메일 서비스 등) 모든 것의 경계이다.

### 입력 인터페이스 레이어

이 계층은 컨트롤러, cli, gui 등과 같은 앱의 모든 진입 전을 보유한다.

비즈니스 규칙, 유즈 케이스, 지속성 기술 및 다른 종류의 논리에 대한 지식이 없어야 한다.

URL 매개변수와 같은 사용자 입력만 수신하고 사용 사례에 전달하고 최종적으로 사용자에게 응답을 반환해야 한다.

## NodeJS 예제

### 도메인 계층

```jsx
// 도메인 계층
const { attributes } = require("structure");

const User = attributes({
  id: Number,
  name: {
    type: String,
    required: true,
  },
  age: Number,
})(
  class User {
    isLegal() {
      return this.age >= User.MIN_LEGAL_AGE;
    }
  }
);

User.MIN_LEGAL_AGE = 21;
```

시퀄 라이저, 몽구스 와 같은 백엔드 모델 또는 모듈이 포함되지 않는다. 이러한 모듈은 외부 세계와 통신하기 위해 인프라 계층에서 사용하기 위한 것이다.

### 애플리케이션 계층

```jsx
const EventEmitter = require("events");

class CreateUser extends EventEmitter {
  constructor({ usersRepository }) {
    super();
    this.usersRepository = usersRepository;
  }

  execute(userData) {
    const user = new User(userData);

    this.usersRepository
      .add(user)
      .then((newUser) => {
        this.emit("SUCCESS", newUser);
      })
      .catch((error) => {
        if (error.message === "ValidationError") {
          return this.emit("VALIDATION_ERROR", error);
        }

        this.emit("ERROR", error);
      });
  }
}
```

유즈케이스는 애플리케이션 계층에 속하며 프로미스와 달리 성공과 실패의 결과를 가질 수 있다. 이벤트 이미터 패턴을 추천함.

다음과 같이 유즈케이스를 실행하고 각 결과에 대한 리스너를 추가할 수 있다.

```jsx
const UsersController = {
  create(req, res) {
    const createUser = new CreateUser({ usersRepository });

    createUser
      .on("SUCCESS", (user) => {
        res.status(201).json(user);
      })
      .on("VALIDATION_ERROR", (error) => {
        res.status(400).json({
          type: "ValidationError",
          details: error.details,
        });
      })
      .on("ERROR", (error) => {
        res.sendStatus(500);
      });

    createUser.execute(req.body.user);
  },
};
```

### 인프라 계층

```jsx
class SequelizeUsersRepository {
  add(user) {
    const { valid, errors } = user.validate();

    if (!valid) {
      const error = new Error("ValidationError");
      error.details = errors;

      return Promise.reject(error);
    }

    return UserModel.create(user.attributes).then(
      (dbUser) => dbUser.dataValues
    );
  }
}
```

인프라 구현은 복잡하지 않아야 한다?? 이 로직이 상위 계층으로 누출되지 않도록 주의

validate를 도메인 엔티티에 집어넣는구나..

## 레이어 연결

레이어가 다른 레이어와 직접 대화하는 것은 잘못된 것일 수 있다. 레이어 간의 결합을 유발한다.

객체지향 프로그래밍에서 이 문제를 해결할 일반적인 설루션은 종속성 주입이다. 간결하게 클래스의 종속성을 분리할 수 있으므로 종속성을 제거하는 작업이 간단한 작업이 되기 때문에 테스트도 쉽다.

예제

[https://github.com/talyssonoc/node-api-boilerplate](https://github.com/talyssonoc/node-api-boilerplate)

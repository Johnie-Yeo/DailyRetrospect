# Object.create

- 직접적으로 prototype키워드를 사용하진 않지만, Prototype object를 만드는 것과 동일.

```
const healthObj = {
  showHealth : function() {
    console.log(this.name + "님, 오늘은 " + this.healthTime + "에 운동을 하셨네요");
  }
}

const ho = Object.create(healthObj, {
   name: { value: "crong" },
   healthTime: { value: "12:22" } 
})

ho.showHealth();
```

- Object.create는 prototype기반 객체연결(상속형태)을 좀더 매끄럽게 사용하기 위해 탄생했다고 이해할 수 있음.

- Object.create를 사용하면 객체연결구조가 잘 만들어짐.

- 하지만 이 방법은 많이 쓰이지 않고 있는데 이유는, ES6 Classes의 extend를 사용해서 이제 보다 쉽게 클래스간 상속 구조를 만들 수 있게 됐기 때문.
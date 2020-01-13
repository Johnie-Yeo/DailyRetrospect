# Private 만들기

- JS에서 공식적으로는 private이라는 개념이 없지만(객체지향이 아니므로 === 객체지향처럼 사용할수는 있지만 객체지향언어는 아니므로) closure을 활용하면 private을 흉내 낼 수 있다.

```
const obj = function() {
    const age = 1;
    const setAge = (_age) => { age = _age; };
    const getAge = () => age;

    return {
        setAge,
        getAge
    }
}
```
## Q: 옵저버 패턴은 무엇인가요?

주체가 어떤 객체의 상태 변화를 관찰하다가 상태 변화가 있을 대마다 메서드 등을 통해 옵저버 목록에 있는 옵저버들에게 변화를 알려주는 디자인 패턴입니다.

여기서 **주체**란 객체의 상태 변화를 보고 있는 관찰자이며, **옵저버**들은 이 객체의 상태 변화에 따라 전달되는 메서드 등을 기반으로 추가 변화 사항이 생기는 객체들을 의미합니다. 이러한 옵저버 패턴을 주로 이벤트 기반 시스템에 사용하며 [MVC](MVC.md) 패턴에도 사용됩니다.
## Q: 옵저버 패턴을 어떻게 구현하나요?

여러 가지 방법이 있지만, 대표적으로 JS의 [프록시 객체](Proxy.md)를 활용하여 구현할 수 있습니다. 프록시 객체를 통해 객체의 속성이나 메서드 변화 등을 감지하고 이를 미리 설정해 놓은 옵저버들에게 전달하는 방법으로 구현할 수 있습니다.

```Javascript
function createReactiveObject(target, callback) {
	const proxy = new Proxy(target, {
		set(obj, prop, value) {
			if (value !== obj[prop]) {
				const prev = obj[prop];
				obj[prop] = value;
				callback(`${prop}가 [${prev}] >> [${value}]로 변경되었습니다.`);
			}
			return true;
		}
	})
	return proxy;
}

const a = {
	"형규": "솔로"
};

const b = createReactiveObject(a, console.log);
b.형규 = "솔로";
b.형규 = "커플";
// 형규가 [솔로] >> [커플]로 변경되었습니다.
```

프록시 객체의 `set()` 함수가 속성에 대한 접근을 가로채, 형규라는 속성이 솔로에서 커플로 변경되는 것을 감지하여 이를 등록된 `callback`(옵저버)인 `console.log`가 변경사항을 출력할 수 있습니다.

> Q: 현재 작성하거나 예시로 보여준 코드에서는 객체와 알림 로직이 강하게 결합되어 있는데 어떻게 분리할 수 있을까요?
> 
> Q: 만약 콜백(옵저버)가 여러 개라면 어떻게 해야할까요?

위의 코드는 1:1 대응 방식이기 때문에 응집도는 높지만 확장성이 떨어지는 구조입니다. 이를 해결하기 위해서는 주체 클래스를 도입하여 옵저버들을 리스트로 관리하는 방식으로 변경하면 옵저버가 여러개인 경우도 해결할 수 있으며 객체와 알림 로직의 결합을 낮출 수 있습니다.

```javascript
class Observable {
  constructor(target) {
    this.observers = new Set();
    
    return new Proxy(target, {
      set: (obj, prop, value) => {
        if (value !== obj[prop]) {
          const prev = obj[prop];
          obj[prop] = value;
          this.notify(prop, prev, value);
        }
        return true;
      }
    });
  }

  subscribe(callback) {
    this.observers.add(callback);
  }

  unsubscribe(callback) {
    this.observers.delete(callback);
  }

  notify(prop, prev, value) {
    this.observers.forEach(callback => 
      callback(`${prop}가 [${prev}] >> [${value}]로 변경됨`)
    );
  }
}


const target = { "형규": "솔로" };
const b = new Observable(target);

const logger = (msg) => console.log(`[로그]: ${msg}`);
const alarm = (msg) => console.log(`[알림]: 🔔 ${msg}`);

b.subscribe(logger);
b.subscribe(alarm);

b.형규 = "커플"; 
// 출력: [로그]: 형규가 [솔로] >> [커플]로 변경됨
// 출력: [알림]: 🔔 형규가 [솔로] >> [커플]로 변경됨
```

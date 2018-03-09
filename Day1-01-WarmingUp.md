# Warming Up

이 글은 Hika Maeng님의 [Code Spitz 75](https://www.facebook.com/groups/codespitz/) 강의 내용을 정리한 글입니다.

본격적인 객체지향 디자인 패턴에 들어가기에 앞서, ES2015+로 워밍업 좀 해보자.

## Mission: JSON 읽어서 테이블로 표시하기

강의 내용에 있던 소스에 주석 정도만 추가 

```html
<!doctype html>
<html>
<head>
<meta charset="utf-8">
<title>CodeSpitz71-1</title>
</head>
<body>

<section id="data"></section>

<script>
// Table 구현부를 보기 전에 맨 아래 부분을 먼저 보자.

const Table = (_ => {
  // 여기는 private static 영역
  const Private = Symbol();

  return class {
    // 생성자
    constructor(parent) {
      // 가드 올려라(Shield Pattern)
      if (typeof parent != 'string' || !parent) throw "invalid param";
      this[Private] = {parent};
    }

    // public method 영역

    // promise 방식으로 데이터 로딩
    // load(url) {
    //   fetch(url).then(response => {
    //     return response.json();
    //   }).then(json => {
    //     this._render();
    //   })
    // }

    // promise 보다 간단한 async/await라는게 있다니 써보자.
    async load(url) {
      // url에 대한 가드도 있으면 좋아요!

      const response = await fetch(url);
      // 가드 올려라(Shield Pattern)라
      if (!response.ok) throw "invalid response";
      const json = await response.json();
      const {title, header, items} = json;
      // 가드 올려라(Shield Pattern)
      if (!items.length) throw "no items";

      Object.assign(this[Private], {title, header, items});
      this._render();
    }

    // private 메서드
    // 자바스크립트에서는 엄격하게 private를 구현할 수 없으므로
    // 메서드 이름 앞에 _ 를 붙여서 의도를 표현
    // 이런데도 외부에서 _ 메서드를 굳이 호출하는 놈은 개변태. 같이 일하지 않는 게 좋음 ㅋ
  
    // _render()에 데이터를 파라미터로 넘기기 vs this 안의 인스턴스 변수로부터 읽기
    // render()에 파라미터로 넘기면 render()는 파라미터를 또 검증해야함 -> 검증이 여러 곳에서 중복 발생
    // this 안의 인스턴스 변수는 이미 검증된 데이터이므로 this의 데이터는 믿고 쓸 수 있다.
    // 따라서 이 문제는 취향에 의한 선택 문제가 아니라 파라미터를 쓰지 않고 this를 쓰는 것이 맞다.
    _render() {
      // 부모, 데이터 체크
      const fields = this[Private];
      const parent = document.querySelector(fields.parent);
      // 가드 올려라(Shield Pattern)라
      if (!parent) throw "invalid parent";

      // 아래 로직은 데이터 검증이라기보다 데이터 상태에 따른 로직 분기용. 데이터 검증은 로딩 단계에서 해야. 
      if (!fields.items || !fields.items.length) {  
        parent.innerHTML = "no data";
        return;
      } else {
        parent.innerHTML = "";
      }

      // Table 생성
      const table = document.createElement("table");    

      // Caption 생성
      const caption = document.createElement("caption");
      caption.innerHTML = fields.title;
      table.appendChild(caption);

      // THEAD, TH 생성
      table.appendChild(
        fields.header.reduce((thead, data) => {
          const th = document.createElement("th");
          th.innerHTML = data;
          thead.appendChild(th);
          return thead;
        }, document.createElement("thead"))
      );

      // TR, TD 생성 및 부모에 TABLE 추가
      parent.appendChild(
        fields.items.reduce((table, row) => {
          table.appendChild(
            row.reduce((tr, col) => {
              const td = document.createElement("td");
              td.innerHTML = col;
              tr.appendChild(td);
              return tr;
            }, document.createElement("tr"))
          )
          return table;
        }, table)
      );
    }
  };

})();

// 가장 먼저 작성해야 할 부분은 Table의 구현부가 아니라
// Table을 사용할 호스트 코드다.
// 프로그램이 달성하고자 하는 바를 가장 추상적인 수준에서 정의하는 걸로 프로그래밍을 시작하자.
// 화면의 #data 에 json 데이터를 읽어서 뭔가 그린다. 와 같이, 프로그램의 목적을 가장 추상적인 수준에서 기술
const table = new Table("#data");
table.load("https://gist.githubusercontent.com/hikaMaeng/717dc66225e40a8fe8d1c40366d40957/raw/447d44b800ed98817b0d29681be90aa1ec36e4ac/71_1.json");
</script>
</body>
</html>
```

워~ 이건 워밍업이 아냐~ 진 다 빠졌.. 저게 JavaScript 맞아? ㄷㄷㄷ

## 프로그램 작성 흐름

1. 프로그램이 달성하고자 하는 바를 우리말로 먼저 기술해본다.

    - 먼저 가장 추상적인 수준, 그러니까 구체적인 구현 방식을 배제하는 수준에서 기술해본다.
    - 프로그램의 목적을 달성하기 위해 필요한 최소한의 등장 인물을 생각해본다.

1. 최소한의 등장 인물만으로 동작하는 추상적인 코드를 먼저 작성한다.

    - 앞의 코드에서 맨 아래에 있던 부분
    - 구체적인 구현부는 작성하지 않는다.

1. 각 등장 인물의 구체적인 구현부를 작성한다.

    - 등장 인물이 어떤 기능을 해야 프로그램의 목표를 달성할 수 있는지 생각하며 메서드 정의 부분 작성
    - 각 메서드의 동작 방식을 슈도 코드(Pseudo Code)로 작성한다.
    - 구현 부분은 동료에게 넘긴다 ㅋㅋ 가 아니라 스스로 가장 나중에 작성한다.
    

## 공부해야 할 ES2015+

디자인 패턴 좀 배워보러 왔더니 첫날부터 이게 웬 암호학 강의야.. 이대로는 안 되겠다. 공부 좀 해야겠..

### arrow function

- 다른 언어에서는 보통 람다(Lambda)라고 불린다.
- arrow function은 함수지만 arrow function 내에서의 `this`는 JavaScript의 일반적인 함수 안에서의 `this`와는 다르게, 별도로 `this`를 바인딩하지 않고 arrow function이 사용되고 있는 scope에서의 `this`를 가리킨다.
- `function`, `var` 등과 같이 ES3.1에서만 존재하고 ES2015+에는 없는 키워드를 사용하면 ES3.1 호환 모드로 동작하게 되고 성능 저하가 발생한다.

### Symbol

- 식별자를 만드는 ES2015+의 새로운 방식

### async/await

- `promise`보다 더 간략하게 비동기로 함수를 호출하고, 비동기로 값을 받아올 수 있게 해주는 키워드
- `async`는 함수 앞에 붙일 수 있으나, 다른 함수의 파라미터로 넘겨지는 함수 앞에는 붙일 수 없다.
- `await`는 Java8+의 `CompletableFuture`와 유사

### fetch

- XHR을 더 간단한 방식으로 대체하는 ~ES2015+의 새로운 기능..이 아니라~ 브라우저에서 제공하는 API
 
### Object.assign()

- 객체를 복사해서 할당(assign). 이걸 보자. https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Object/assign

### reduce()

- 여러 개의 값을 하나의 값으로 **집계**하는 것
- 같은 말이라도 **벡터를 스칼라로 만들어 주는 것**이라고 하니 몸서리치게 간지난다.
- 백엔드에서 빅데이터 하둡 맵리듀스 어쩌고 하는데 그건가? 그 맵reduce 맞음

## 기타

### 에러의 3단계
		
- 컴파일 에러: JS에서는 SyntaxError뿐

  - 프로그램이 실행조차 되지 않으므로 가장 저렴한 비용으로 수정할 수 있다.

- 런타임 에러: NullPointer, ArrayIndexOutofBounds, ... 등

  - 실행해야만 그리고 실행 중에 항상도 아니고 특정 조건에서만 발생하고, 다른 쪽으로 전파될 수 있으므로 수정에 많은 비용 필요    
  - 따라서 런타임 에러가 발생할 수 있는 상황을 예상해서 그 조건을 가능한 일찍 명시하고, 발생하면 바로 죽여야(throw) 수정 비용을 낮출 수 있다.

- 컨텍스트 에러

  - 내용적으로 잘못 수행되었는데 프로그램은 죽지 않고 정상 가동, 잘못된 상태로 일이 진행/완료됨    
  - 프로그램의 수정만으로 문제를 해결할 수 없으므로 수정에 가장 많은 비용 필요 -> 소송으로 회사 망할 수도..


**비정상 상황에서는 바로 복구하거나 바로 죽도록 프로그램 해야 한다.**

----
참 쉽죠?

![Imgur](https://i.imgur.com/BKC7WxB.jpg)

1부 끝.

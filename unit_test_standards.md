# Mục lục

- [Mục lục](#mục-lục)
- [Unit test standards](#unit-test-standards)
  - [Các concept cần biết trước khi đọc](#các-concept-cần-biết-trước-khi-đọc)
    - [Message](#message)
      - [Khái niệm](#khái-niệm)
      - [Các luồng message](#các-luồng-message)
      - [Các loại message](#các-loại-message)
      - [Test message như thế nào](#test-message-như-thế-nào)
    - [Spy](#spy)
    - [Spy vs mock](#spy-vs-mock)
  - [Code coverage](#code-coverage)
- [Angular unit test](#angular-unit-test)
  - [Quan điểm test](#quan-điểm-test)
  - [TestBed](#testbed)
  - [Testing services](#testing-services)
    - [Service không có dependency](#service-không-có-dependency)
    - [Service có dependency](#service-có-dependency)
    - [Http service](#http-service)
  - [Testing components](#testing-components)
    - [Class testing](#class-testing)
    - [DOM testing](#dom-testing)
      - [Giải nghĩa 1 số term](#giải-nghĩa-1-số-term)
      - [Test cases](#test-cases)
  - [Testing directives](#testing-directives)
  - [Testing pipes](#testing-pipes)
  - [Một số công nghệ khác](#một-số-công-nghệ-khác)

# Unit test standards

## Các concept cần biết trước khi đọc

### Message

#### Khái niệm

Có thể hiểu message là lời nói của các object với nhau. Khi object A gọi đến object B, A gửi cho B 1 message. Đích đến của các message thông thường là các method. Ví dụ

```typescript
class Human {
   constructor(car: Car) {
      this.leg_count = legs_count
      this.arms_count = arms_count
      this.car = car
   }

   open(thing) {}

   drive() {
      this.open(this.car.door)
      this.car.ignite()
      more logic ...
   }
}

```

```typescript
class Car {
   constructor(door: Door, roof: boolean = true) {
      this.door = door
      this.roof = roof
   }

   ignite() {
      wave this.right_arm
   }
}

```

```typescript
const car = new Car(door);

const person = new Human(car);

person.drive();
```

Khi method `drive()` của person được gọi, 1 message đã được gửi đến class `Human`, yêu cầu chạy `drive()`. Khi `person.drive()` chạy, nó cũng gửi đến class `Car` 1 message, yêu cầu chạy method `ignite`

#### Các luồng message

- Incoming message: Các message mà 1 object nhận/hiểu được. Nôm na mà nói thì có thể các incoming message mà object hiểu dựa vào method và property mà nó có. Ở trong ví dụ trên, `car` có các incoming message là `ignite`, `door`, `roof`

- Outgoing message: Các message mà 1 object gửi cho object khác. Ở ví dụ trên, trong method `drive()` của class `Human` khi chạy sẽ gửi đi 1 outgoing message đến class `Car`

- Sent-to-self message: Là các message là 1 object tự gửi cho chính mình. Ở ví dụ trên, method `drive()` của `Human` tự gửi 1 message cho chính nó, yêu cầu chạy method `open()`. Thông thường message này sẽ được gửi cho các private methods

#### Các loại message

- Command: Là các message có side-effect, hoặc gọi đến các method có side-effect

- Query: Là các message không có side-effect, hoặc gọi đến các method không có side-effect

#### Test message như thế nào

![message testing](https://viblo.asia/uploads/515264bb-44e7-4ced-aaa9-f323ae723cc0.png)

- Chủ yếu test incoming message

- Khi test query thì test giá trị trả về

- Khi test command thì test side-effect. Mock nếu cần.

- Outgoing message thì chỉ cần test xem nó được gửi chưa thôi. Nghĩa là chỉ cần biết method nó gửi đến đã được gọi chưa.

- Sent-to-self message thì gần như không cần test, tại nó thường được hướng đến private methods. Các private method chỉ cần được test khi nó là function thử nghiệm, hoặc là logic quá khó thôi, còn không thì nên bỏ quà vì các private method thông thường được gọi trong public methods, khi test các public method thì ta đã test cho cả các private method rồi.

### Spy

Là 1 theo dõi 1 object dùng khi test. Spy có thể cho biết function được theo dõi đã được gọi hay chưa, gọi mấy lần, gọi với argument nào.
Ngoài ra, ta có thể tạo hẳn 1 spy object luôn. Spy object là 1 object mock, và các function của nó đều được theo dõi.

```typescript
class restConnector {
  constructor(
    private httpClient: HttpClient,
    private localStorageService: LocalStorageService,
    private notifyService: NotifyService
  ) {}
} // object gốc

// test
httpClientSpy = jasmine.createSpyObj("HttpClient", [
  "get",
  "post",
  "put",
  "delete",
]);
// spy httpClient

restConnector = new RestConnector(
  httpClientSpy as any,
  localStorageService,
  notifyService
);
// từ giờ, mỗi khi restConnector gọi đến httpClient, message sẽ được redirect đến httpClientSpy
```

### Spy vs mock

- Khi nào muốn test xem function có được gọi không, với argument nào, được gọi bao nhiêu lần thì dùng spy

- Khi nào muốn dùng giá trị trả về để test thì dùng mock

## Code coverage

[Docs](https://angular.io/guide/testing-code-coverage)

- Chạy `ng test --no-watch --code-coverage` để xem code coverage
- Nếu như muốn `ng test` mà có luôn code coverage report thì thêm cái này vào `angular.json`:

  ```JSON
   "test": {
      "options": {
      "codeCoverage": true
      }
   }
  ```

- Để enforce code coverage thì config trong file `karma.conf.js` như sau

  ```javascript
   coverageIstanbulReporter: {
      reports: [ 'html', 'lcovonly' ],
      fixWebpackSourcePaths: true,
      thresholds: {
         statements: 80,
         lines: 80,
         branches: 80,
         functions: 80
      }
   }
  ```

---

# Angular unit test

## Quan điểm test

- Không test những gì của Angular, VD: Router, Activated Routes

- Mock các dependency không cần thiết (các thư viện, service, component, etc.)

- Test public methods, tránh test private methods

- Chỉ test incoming message, không test các message tự gửi

## TestBed

TestBed giống như ngModule nhưng dùng cho test, tác dụng của nó là mô phỏng NgModule, để cho những việc như DI, initiatilize component, service không cần phải làm bằng tay.

Dù dễ dùng nhưng TestBed là không bắt buộc, việc inject hay initialize hoàn toàn có thể làm bằng tay. Vì TestBed lôi cả DOM (ở `createComponent()`) vào test nên nếu như không cần (cần hay không tùy sếp và project) phải test DOM thì có thể tham khảo [link này](https://medium.com/angular-in-depth/angular-testbed-considered-harmful-3f91f647d1fd).

Nếu dùng TestBed, chỉ cần import đủ dùng để test thôi.

P/s: Bằng chứng cho việc TestBed lôi cả DOM vào:

> `TestBed.createComponent()` creates an instance of the `BannerComponent`, adds a corresponding element to the test-runner DOM, and returns a [`ComponentFixture`](https://angular.io/guide/testing-components-basics#component-fixture).

## Testing services

### Service không có dependency

Test như hình ở trên:

- Test public methods trả gì
- Nếu có side effect, và side effect đó nó ảnh hưởng đến class khác/function public khác thì test
- Tránh test private methods, vì khi test public methods thì mình đã test cả private methods rồi

### Service có dependency

- Mock hoặc spy hết các dependency, rồi test như trên
- Mock khi ta quan tâm đến return value, hoặc khi nó sync
- Spy khi ta quan tâm nó được gọi như thế nào, hoặc khi nó async

### Http service

Spy http dependency và test như thường

## Testing components

### Class testing

Test như test service

### DOM testing

Tham khảo [Angular docs](https://angular.io/guide/testing-components-basics#component-dom-testing)

#### Giải nghĩa 1 số term

- `createComponent()`: trả lại 1 `ComponentFixture`

- [`ComponentFixture`](https://angular.io/api/core/testing/ComponentFixture) là 1 lớp abstract của component, cho phép người viết test có thể thao tác được với instance của component và DOM khi test.

- `NativeElement` vs `DebugElement.nativeElement`: Cả 2 đều được dùng để chọc vào DOM, tuy nhiên Debug element thì an toàn hơn vì nó có thể dùng được ở tất cả các plaform

> Angular relies on the `DebugElement` abstraction to work safely across _all supported platforms_. Instead of creating an HTML element tree, Angular creates a `DebugElement` tree that wraps the _native elements_ for the runtime platform. The `nativeElement` property unwraps the `DebugElement` and returns the platform-specific element object.

#### Test cases

[https://angular.io/guide/testing-components-scenarios](https://angular.io/guide/testing-components-scenarios)

1. Component binding {{ value }}

   - Goal: Xem component render view đúng ý không
   - Cách làm:
     - Thay đổi giá trị cần render ra view của component
     - fixture.detectChanges() để chạy change detection bằng tay, vì TestBed không chạy change detection tự động
     - Expect giá trị đúng với giá trị đã set

2. Component có dependency

   - Goal: test class và DOM
   - Cách làm:
     - Mock các dependency đó. Một số dependency không mock được nhưng mà vô hại (ví dụ như là thư viện) thì dùng thẳng được
     - Test như thường

3. Component có async dependency

   - Goal: test class và DOM
   - Cách làm: spy các async dependency và test như thường

4. Input và output

   - Goal: test xem khi truyền data vào `@Input` và trigger event của `@Output` thì event đó có được raise không
   - Cách làm:
     - `@Input`: thay đổi giá trị của `@Input` trong file ts, `detectChanges()`, rồi expect giá trị
     - `@Output`: subscribe event được emit ra, trigger event bằng `triggerEventHandler()` rồi expect event đó có được raise

5. Component được chứa trong component khác

   - Goal: test class và DOM
   - Cách làm: tạo 1 mock component rồi đưa component muốn test vào làm con của component mock rồi test bình thường

6. Routing component

   - Goal: Test xem Router có được gọi không
   - Cách làm: spy Router

7. Routed component

   - Goal: test xem mình đã truyền đúng arguments/params để redirect đến routed component chưa
   - Cách làm:
     - Spy Router
     - Tạo ActivatedRoute stub (như trong guide)
     - Test như thường

8. Nested component

   - Goal: test class và DOM
   - Cách làm: Mock các component được nest trong component được test rồi test như thường

9. Component dùng routerLink

   - Goal: test xem khi truyền 1 cái link vào thì cái link đấy được đúng như thế không, và khi click vào thì input nó giống với output không
   - Cách làm:
     - Tạo 1 mock routerLink directive, có @Input và 1 cái để bắt sự kiện click vào nó
     - expect là click vào thì output của nó đúng với input truyền vào qua directive mock
     - Việc angular có redirect không thì angular đã test rồi nên không cần quan tâm

10. Override component provider

    - Goal: test class và DOM
    - Use case: khi 1 service không được provide ở module, mà là ở component.
    - Cách làm: dùng overrideComponent của TestBed

## Testing directives

- Mục đích test ở đây là test xem directive mình viết có ảnh hưởng lên DOM đúng ý không
- Tạo 1 component mock, đưa directive vào và chọc vào DOM để test

## Testing pipes

- Khi test pipe, ta test class thôi chứ không test DOM, vì khi test component có pipe đó thì có thể expect luôn out put của pipe trên component đó

## Một số công nghệ khác

- Default của Angular
  1. Testing framework: Jasmine (default của angular)
  2. Test runner: Karma
- Một số công nghệ khác
  1. Testing framework: Jest (được ưa chuộng hiện nay)
  2. Test runner: Cypress (được ưa chuộng hiện nay)

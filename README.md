# Nguyên lý S.O.L.I.D

## 1. Nguyên lý SOLID là gì?

- Nguyên lý SOLID là tập hợp các nguyên tắc thiết kế phần mềm giúp chúng ta có thể xây dựng một cấu trúc phần mềm dễ đọc (readable), dễ hiểu (understandable) và dễ bảo trì (maintainable)...

- Nguyên lý SOLID bao gồm 5 nguyên tắc sau:
  - **S** — Single responsibility principl
  - **O** — Open closed principle
  - **L** — Liskov substitution principle
  - **I** — Interface segregation principle
  - **D** — Dependency Inversion principle

## 2. Nội dung của nguyên lý SOLID

### 2.1 Single Responsibility Principle

- “Do one thing and do it well” 
- Mỗi đối tượng (class/function/method...) chỉ nên chịu trách nhiệm cho một việc duy nhất.
- Nếu đối tượng thực hiện quá nhiều công việc thì sẽ trở nên cồng kềnh, trở nên khó đọc, khó maintain. 

Ví dụ:

```javascript
// bad: một hàm vừa làm việc kiểm tra vùa làm insert vào database
const validateAndCreateUser = (req) => {
    let isValidUser = true;
    if (User.find(req.username || req.email) !== null)
        isValidUser = false;
    if (req.role !== 'Admin' && req.role !== 'User')
        isValidUser = false;

    if (isValidUser) {
        User.create(req.username, req.password, req.email, req.role);
    }
}

```

Ta có thể tách thành các hàm con xử lý một nhiệm vụ duy nhất:

```javascript
// good:
const validateUser = (req) => {
    if (User.find(req.username || req.email) !== null)
        return false;
    if (req.role !== 'Admin' && req.role !== 'User')
        return false;
    return true;
}

const createUser = (req) => {
    User.create(req.username, req.password, req.email, req.role);    
}

const validateAndCreateUser = (req) => {
    const isValidUser = validateUser(req);

    if (isValidUser) {
        createUser(req);
    }
}
```



### 2.2 Open/Closed Principle

- "Open to extension, but closed to modification."
- Một đối tượng thì có thể được mở rộng (bằng cách viết thêm code mới) nhưng không được phép thay đổi bên trong các phần code đã cố định.

**Ví dụ:** Trong hàm `validateUser` ở trên, việc kiểm tra role đang kiểm tra thô 2 role. Khi mở rộng thì phải thay đổi code. Giả sử ta cần mở rộng có thể có nhiều role thì có thể tạo bảng Role trong database và xây dựng hàm kiểm tra role:

```javascript
const checkRole = (role) => {
    return Role.findAll().includes(role);
}
```

Khi đó, hàm `validateUser` có thể refactor và sẽ không cần thay đổi gì sau này

```javascript
const validateUser = (req) => {
    if (User.find(req.username || req.email) !== null)
        return false;
    return checkRole(req.role);
}
```

Giả sử cần thêm role mới vào thì ta có thể mở rộng bằng cách tạo hàm `addRole`

```javascript
const addRole = (role) => {
    Role.create(role);
}
```

### 2.3 Liskov substitution principle

	- "Build software systems from interchangeable parts."
	- Một lớp con nên ghi đè các phương thức của lớp cha mà không làm thay đổi tính đúng đắn của chương trình.

 - Ví dụ: 
   - Class Shape có phương thức tính diện tích là getArea()
     - Class Circle kế thừa class Shape cũng có phương thức tính diện tích getArea() thì phương thức này cũng phải dùng để tính diện tích chứ không được sử dụng để làm nhiệm vụ khác.

### 2.4 Interface Segregation Principle

- "Must prevent classes from relying on modules or functions that they dont need." Các interface nên tách biệt với nhau  --> không nên tạo ra các interface cồng kềnh. 
- Thay vì gộp hết tất cả vào 1 interface lớn --> tách ra nhiều interface nhỏ với các methods liên quan với nhau. --> Client không phải phụ thuộc vào các interface không cần thiết. (Chỉ gọi những method cần thiết).

Ví dụ: ta có 1 interface là `IShape` và các class `Circle`, `Rectangle`, `Square` implement `IShape`. Nếu ta cần một hàm draw cho các class đó thì trong IShape phải định nghĩa các hàm draw cho từng class:

```pseudocode
interface IShape {
	function drawCircle();
	function drawRectangle();
	function drawSquare();
}

class Circle implement IShape {};
class Rectangle implement IShape {};
class Square implement IShape {};
```

Như vậy mỗi khi class implement IShape phải định nghĩa đủ các hàm trên nhưng thực tế không dùng. Để tránh việc này, ta có 2 cách để giải quyết:

- Tạo các interface nhỏ hơn: `ICircle`, `IRectangle`, `ISquare`, trong mỗi interface định nghĩa hàm tương ứng.

```pseudocode
interface ICircle {
	function drawCircle();
}

class Circle implement ICircle {};

interface IRectangle {
	function drawRectangle();
}

class Rectangle implement IRectangle {};

interface ISquare {
	function drawSquare();
}

class Square implement ISquare {};
```



- Trong interface `IShape` chỉ khai báo 1 hàm `draw()` duy nhất và các class sẽ implement cụ thể hàm draw theo từng class.

```pseudocode
interface IShape {
	function draw();
}

class Circle implement IShape {
	function draw() {
		// ...
	}
};

class Rectangle implement IShape {
	function draw() {
		// ...
	}
};

class Square implement IShape {
	function draw() {
		// ...
	}
};
```



### 2.5 Dependency Inversion Principle

- "Abstractions must not depend on details. Details must depend on abstractions."

- Các module cấp cao không nên phụ thuộc vào các module  cấp thấp và cả hai đều phải phụ thuộc vào các module trừu tượng và các module trừu tượng không nên phụ thuộc vào các chi tiết. 
- Thông tin chi tiết nên phụ thuộc vào sự trừu tượng.

​	Ví dụ:

Đoạn code bên dưới không tốt vì http request là một hàm tổng quát nhưng bị phụ thuộc vào xử lý của hàm setState

```javascript
http.get('http://example.com/api/examples', (res) => {
	this.setState({
        key1: res.value1,
        key2: res.value2,
        key3: res.value3
    });
});
```

Để tránh như vậy, ta định nghĩa riêng 1 hàm httpRequest cho phép truyền vào url và hàm xử lý kết quả bất kì ứng với url đó. Như vậy có thể sử dụng hàm hàm httpRequest mà không bị ảnh hưởng bởi các hàm xử lý chi tiết

```javascript
const httpRequest = (url, setState) => {
	http.get(url, (res) => setState.setValues(res));
};

const setState = {
	setValues: (res) => {
		this.setState({
			key1: res.value1,
            key2: res.value2,
            key3: res.value3
		});
	}
}
```


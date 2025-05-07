Bạn có muốn mình giải thích sâu hơn về bất kỳ phần nào (như generics, closures, hay methods) không? Hoặc cần ví dụ cụ thể hơn?

- **Đặc trưng**: Pass-by-value, không default param, không overloading.
- **Nâng cao**: Closure, anonymous, defer, panic/recover, method, generics.
- **Cơ bản**: Định nghĩa, tham số, trả về nhiều giá trị, variadic.

### Tóm tắt

---

`package main

import "fmt"

type User struct { Name string }

func processUsers(users ...User) func() int { count := 0 return func() int { count++ fmt.Printf("Processed %d users\n", count) return count } }

func (u *User) updateName(newName string) { [u.Name](http://u.Name) = newName }

func main() { u1 := User{"Alice"} u2 := User{"Bob"}

```
counter := processUsers(u1, u2)
counter() *// In: Processed 1 users*
counter() *// In: Processed 2 users*

u1.updateName("Charlie")
fmt.Println(u1.Name) *// In: Charlie*
```

}`

CollapseWrapCopy

go

Kết hợp các khái niệm trên trong một ví dụ:

### 11. Ví dụ thực tế

---

`func min[T int | float64](a, b T) T { if a < b { return a } return b }

func main() { fmt.Println(min(3, 5)) _// In: 3_ fmt.Println(min(3.1, 2.9)) _// In: 2.9_ }`

CollapseWrapCopy

go

- Có thể giới hạn kiểu:
- [T any]: Type parameter, T có thể là bất kỳ kiểu nào (any là alias của interface{}).

`func printSlice[T any](s []T) { for _, v := range s { fmt.Println(v) } }

func main() { printSlice([]int{1, 2, 3}) _// In: 1 2 3_ printSlice([]string{"a", "b"}) _// In: a b_ }`

CollapseWrapCopy

go

Từ Go 1.18, bạn có thể viết hàm generic để làm việc với nhiều kiểu dữ liệu:

### 10. Hàm với Generics (Go 1.18+)

---

- **Không có generic functions (trước Go 1.18)**: Từ Go 1.18, bạn có thể dùng generics, mình sẽ giải thích ở phần sau.
- **Không có overloading**: Go không cho phép nhiều hàm cùng tên với tham số khác nhau.
- **Không có default parameters**: Go không hỗ trợ giá trị mặc định cho tham số như Python hay JavaScript.
- **Tên hàm**: Bắt đầu bằng chữ cái in hoa (exported, công khai) hoặc chữ thường (unexported, nội bộ package).

### 9. Một số lưu ý quan trọng

---

`func (r *Rectangle) scale(factor int) { r.width *= factor r.height *= factor }

func main() { rect := Rectangle{3, 4} rect.scale(2) fmt.Println(rect.area()) _// In: 24_ }`

CollapseWrapCopy

go

- Dùng pointer receiver để thay đổi giá trị:
- (r Rectangle): Receiver, biến r đại diện cho instance của Rectangle.

`type Rectangle struct { width, height int }

func (r Rectangle) area() int { return r.width * r.height }

func main() { rect := Rectangle{3, 4} fmt.Println(rect.area()) _// In: 12_ }`

CollapseWrapCopy

go

Trong Go, bạn có thể gắn hàm vào một kiểu dữ liệu (struct) để tạo **method**:

### 8. Hàm với Method Receiver (Methods)

---

`func mayPanic() { panic("Something went wrong!") }

func safeCall() { defer func() { if r := recover(); r != nil { fmt.Println("Recovered:", r) } }() mayPanic() }

func main() { safeCall() _// In: Recovered: Something went wrong!_ fmt.Println("Continue") }`

CollapseWrapCopy

go

### Ví dụ:

- **recover**: Bắt và xử lý panic trong hàm gọi bằng defer.
- **panic**: Dừng chương trình và báo lỗi.

Go không dùng try-catch mà dùng panic và recover để xử lý lỗi nghiêm trọng:

### 7. Panic và Recover trong hàm

---

`func main() { defer fmt.Println("1") defer fmt.Println("2") fmt.Println("Done") *// In: Done// 2// 1* }`

CollapseWrapCopy

go

- Nhiều defer được thực thi theo thứ tự **LIFO** (Last In, First Out):

`func readFile() { file, _ := os.Open("test.txt") defer file.Close() *// Đọc file...* }`

CollapseWrapCopy

go

- defer thường dùng để đóng tài nguyên (file, database connection):

`func main() { defer fmt.Println("World") fmt.Println("Hello") *// In: Hello// World* }`

CollapseWrapCopy

go

Từ khóa defer trì hoãn việc thực thi một hàm cho đến khi hàm chứa nó kết thúc:

### 6. defer trong hàm

---

`func modifyPtr(x *int) { *x = 100 }

func main() { a := 10 modifyPtr(&a) fmt.Println(a) _// In: 100_ }`

CollapseWrapCopy

go

- Để thay đổi giá trị, cần dùng **pointer**:

`func modify(x int) { x = 100 }

func main() { a := 10 modify(a) fmt.Println(a) _// In: 10 (không thay đổi)_ }`

CollapseWrapCopy

go

Go mặc định truyền tham số theo **giá trị** (pass-by-value), không phải tham chiếu:

### 5. Tham số truyền theo giá trị

---

`func main() { func(name string) { fmt.Println("Hi,", name) }("Alice") *// In: Hi, Alice// Gán cho biến* add := func(a, b int) int { return a + b } fmt.Println(add(2, 3)) *// In: 5* }`

CollapseWrapCopy

go

Hàm không cần tên có thể được định nghĩa và gọi ngay lập tức:

### 4. Hàm vô danh (Anonymous Functions)

---

- count được lưu trong closure, mỗi lần gọi c() sẽ tăng giá trị.

`func counter() func() int { count := 0 return func() int { count++ return count } }

func main() { c := counter() fmt.Println(c()) _// In: 1_ fmt.Println(c()) _// In: 2_ fmt.Println(c()) _// In: 3_ }`

CollapseWrapCopy

go

Hàm có thể trả về một hàm khác, và hàm trả về này có thể "nhớ" các biến trong scope của hàm cha:

### 3.3. Hàm trả về hàm (Closures)

`func apply(fn func(int) int, x int) int { return fn(x) }

func double(n int) int { return n * 2 }

func main() { result := apply(double, 5) fmt.Println(result) _// In: 10_ }`

CollapseWrapCopy

go

### 3.2. Truyền hàm làm tham số

`func greet(name string) string { return "Hello, " + name }

func main() { say := greet fmt.Println(say("John")) _// In: Hello, John_ }`

CollapseWrapCopy

go

### 3.1. Gán hàm cho biến

- Trả về từ hàm khác.
- Truyền làm tham số.
- Gán cho biến.

Trong Go, hàm là **first-class citizen**, nghĩa là chúng có thể được:

### 3. Hàm như một giá trị (First-class Functions)

---

- nums được xem như một slice []int trong thân hàm.
- nums ...int: Tham số variadic, nhận bất kỳ số lượng int nào.

`func sum(nums ...int) int { total := 0 for _, num := range nums { total += num } return total }

func main() { fmt.Println(sum(1, 2, 3)) _// In: 6_ fmt.Println(sum(1, 2, 3, 4)) _// In: 10_ }`

CollapseWrapCopy

go

Go hỗ trợ hàm với số lượng tham số không cố định (giống ...args trong JavaScript):

### 2.4. Tham số không giới hạn (Variadic Functions)

- return: Tự động trả về x và y mà không cần viết return x, y.
- (x, y int): Hai giá trị trả về được đặt tên.

`func split(sum int) (x, y int) { x = sum * 4 / 9 y = sum - x return _// "Naked return" - không cần chỉ định giá trị_ }

func main() { a, b := split(17) fmt.Println(a, b) _// In: 7 10_ }`

CollapseWrapCopy

go

Bạn có thể đặt tên cho các giá trị trả về, và chúng sẽ được khởi tạo tự động với giá trị mặc định của kiểu (zero value):

### 2.3. Named return values (Giá trị trả về có tên)

- x, y := swap(...): Gán hai giá trị trả về vào hai biến.
- (string, string): Khai báo hai giá trị trả về kiểu string.

`func swap(a, b string) (string, string) { return b, a }

func main() { x, y := swap("hello", "world") fmt.Println(x, y) _// In: world hello_ }`

CollapseWrapCopy

go

Go hỗ trợ trả về nhiều giá trị, một tính năng rất mạnh mẽ và khác biệt so với nhiều ngôn ngữ khác:

### 2.2. Trả về nhiều giá trị

`func sayHello() { fmt.Println("Hello, World!") }

func main() { sayHello() _// In: Hello, World!_ }`

CollapseWrapCopy

go

Hàm có thể không nhận tham số hoặc không trả về giá trị:

### 2.1. Không có tham số hoặc giá trị trả về

### 2. Các đặc điểm cơ bản của hàm

---

- return a + b: Trả về tổng của a và b.
- int: Kiểu trả về.
- (a int, b int): Hai tham số kiểu int.
- add: Tên hàm.

`func add(a int, b int) int { return a + b }

func main() { result := add(3, 5) fmt.Println(result) _// In: 8_ }`

CollapseWrapCopy

go

### Ví dụ cơ bản:

`func tênHàm(thamSố1 kiểu1, thamSố2 kiểu2, ...) kiểuTrảVề { *// Thân hàm* return giáTrị *// Nếu có giá trị trả về* }`

CollapseWrapCopy

go

### Cú pháp:

Hàm trong Go được định nghĩa bằng từ khóa func, theo sau là tên hàm, danh sách tham số (nếu có), kiểu trả về (nếu có), và thân hàm.

### 1. Cú pháp cơ bản của hàm
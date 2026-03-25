# HEXAGONAL ARCHITECTURE

## I. GIỚI THIỆU & TỔNG QUAN

### 1. Hexagonal Architecture là gì?

#### Định nghĩa Cơ Bản

Hexagonal Architecture là kiểu thiết kế phần mềm nhằm tách biệt hoàn toàn lõi nghiệp vụ (business logic) khỏi các yếu tố bên ngoài.

Những yếu tố bên ngoài bao gồm:

- Giao diện người dùng (CLI, Web, API)
- Cơ chế lưu trữ (File, Database)
- Framework, thư viện kỹ thuật
- Hạ tầng (Infrastructure)

Hexagonal Architecture giống như 1 nhà hàng trong khách sạn có:

- Core: Đầu bếp chỉ có nấu ăn và chỉ tập trung vào nó -> Business Logic
- Ports: Cách khách hàng order món đến cho mình: trực tiếp, gọi qua hotline, hoặc có thể đưa nguyên liệu và nhừo đầu bếp chế biến -> interface
- Adapter: là tiếp tân, lắng nghe mong muốn khách hàng, rồi "phiên dịch lại" cho core hiểu để core làm việc -> UI / API / Database / Framework

#### Trọng Tâm Cốt Lõi

- Business logic không biết và không phụ thuộc vào cách hệ thống được sử dụng hoặc triển khai.
- Thay vì để application xoay quanh database hay UI, Hexagonal Architecture đặt Domain Core vào trung tâm, và mọi tương tác với bên ngoài đều phải đi qua Ports và Adapters.

#### Mục Tiêu Cốt Lõi

Hexagonal Architecture hướng tới các mục tiêu sau:

- Domain không phụ thuộc UI, DB, framework                                   
- Có thể test business logic mà không cần file system, database, CLI         
- Có thể thay đổi: JSON file -> Database; CLI -> Web API mà không sửa domain 
- Code dễ đọc, dễ bảo trì, dễ mở rộng                                        

### 2. Tại Sao Cần Hexagonal Architecture?

#### Vấn Đề Của Kiến Trúc Truyền Thống (Layered Architecture)

Trong kiến trúc phân lớp truyền thống (UI -> Service -> Repository -> Database):

Business logic thường:

- Gọi trực tiếp repository
- Phụ thuộc framework

Domain logic bị rò rỉ sang:

- Controller
- Service
- Infrastructure

Hệ quả:

- Code khó test
- Mỗi thay đổi nhỏ (DB, framework) kéo theo nhiều thay đổi lớn
- Business logic không còn là trung tâm

#### Tight Coupling Và Hậu Quả

Tight Coupling xảy ra khi: Business logic biết chi tiết cách dữ liệu được lưu

Service phụ thuộc trực tiếp vào:

- File system
- Database driver
- External API

Hậu quả:

- Không thể test domain độc lập
- Mock khó hoặc không thể mock
- Refactor tốn kém
- Code dễ vỡ dây chuyền

#### Hexagonal Architecture Giải Quyết Vấn Đề Này Như Thế Nào?

Hexagonal Architecture đảo ngược cách suy nghĩ:

- Thay Vì: "Domain gọi database"
- Thì: "Domain định nghĩa cách nó muốn lưu dữ liệu, còn bên ngoài phải phù hợp với domain"

Cụ thể:

1. Domain định nghĩa Ports (interfaces)
2. Adapters implement các Ports
3. Dependency luôn hướng vào trong


## II. CÁC KHÁI NIỆM 

### 1. Domain Core 

#### Domain Là Gì?

Domain Core là phần trung tâm của hệ thống, nơi chứa:

- Nghiệp vụ chính
- Quy tắc kinh doanh (business rules)
- Logic quyết định điều gì hợp lệ / không hợp lệ

#### Vị Trí Của Domain Trong Hexagonal Architecture

Trong Hexagonal Architecture, Domain được coi là thứ có giá trị nhất và cần được bảo vệ nhất trong hệ thống. Mọi thứ khác (CLI, database, framework) chỉ tồn tại để phục vụ Domain.


#### Entities

Entity là đối tượng có:

- Danh tính (identity)
- Trạng thái có thể thay đổi theo thời gian

Ví Dụ Trong Ticket:

Ticket
├── id (Danh tính)
├── status (Trạng thái)
└── priority (Thuộc tính)

Điểm Quan Trọng: Hai ticket khác nhau không chỉ vì dữ liệu khác, mà vì id khác. Entity tự bảo vệ trạng thái hợp lệ của chính nó. Entity chứa logic, không phải chỉ là data holder.

#### Value Objects

Value Object là:

- Không có identity
- So sánh bằng giá trị
- Thường là immutable = bất biến

Nguyên Tắc: Nếu hai value object có cùng giá trị thì được coi là giống nhau.


#### Domain Services

Domain Service dùng khi:

- Logic nghiệp vụ không thuộc về một entity cụ thể nào. Vẫn là logic thuần domain

Ví Dụ:

- Kiểm tra ticket có thể chuyển từ status A → B hay không
- Validate một hành động liên quan nhiều entity

Lưu Ý:
Domain Service không được phụ thuộc infrastructure. Không gọi file system, database, console.

#### Nguyên Tắc Quan Trọng Nhất Của Domain

Domain KHÔNG ĐƯỢC PHỤ THUỘC VÀO:

- File system (fs)
- Database
- CLI / UI
- Framework (NestJS, Express, etc.)
- Thư viện kỹ thuật không mang ý nghĩa domain

Rule Kiểm Tra Nhanh:
Nếu xóa toàn bộ adapters, domain vẫn compile và test được thì đúng. Nếu không thì có vấn đề.

### 2. Ports (Cổng Giao Tiếp)

#### Port Là Gì?

Port là một interface mô tả:

- Domain cần gì từ bên ngoài
- Hoặc cho phép bên ngoài gọi vào hệ thống như thế nào

Port không phải là implementation, mà là contract.


Ports rất quan trọng vì nếu không có port:

- Domain phải gọi trực tiếp database
- Business logic biết chi tiết kỹ thuật
- Mất khả năng thay thế công nghệ

Ports giúp:

- Đảo chiều phụ thuộc
- Domain định nghĩa luật chơi
- Bên ngoài phải theo luật của domain

#### Phân Loại Ports

##### Primary Ports (Driving Ports)

Primary Port mô tả:

- Cách bên ngoài sử dụng hệ thống
- Thường là: Use Cases, Application Services
Quy Tắc:
CLI không gọi domain trực tiếp, mà gọi Primary Port.

##### Secondary Ports (Driven Ports)

Secondary Port mô tả:

- Những gì domain cần từ bên ngoài

Domain Nói: tao cần lưu ticket theo cách này, còn lưu ở đâu là việc của mày.



#### Nguyên Tắc Cực Kỳ Quan Trọng

Ports thuộc core (domain/application), không thuộc adapters.

- Interface được định nghĩa bởi domain
- Adapters chỉ implement, không định nghĩa luật

Nếu interface nằm trong infrastructure thì sai.

### 3. Adapters (Bộ Điều Hợp)

#### Adapter Là Gì?

Adapter là thành phần:

- Hiện thực hóa (implement) Ports
- Chuyển đổi giữa: Thế giới bên ngoài ↔ Domain

Adapter không chứa business rules.

#### Trách Nhiệm Của Adapter

Adapter có nhiệm vụ:

- Parse input
- Map dữ liệu
- Gọi ports
- Chuyển output sang định dạng phù hợp

Adapter không được:

- Validate business rules
- Quyết định logic nghiệp vụ

#### Phân Loại Adapters

##### Primary Adapters

Primary(Driving) Adapter là: Điểm vào của hệ thống

Ví Dụ:

- CLI
- Web Controller (chỉ so sánh, không implement)
- CLI đóng vai trò Primary Adapter
- CLI gọi Primary Ports

##### Secondary Adapters

Secondary(Driven) Adapter là: Kết nối domain với bên ngoài

Ví Dụ:

- File Storage Adapter (JSON)
- Database Adapter (nếu mở rộng)
- Implement TicketRepository
- Biết fs, JSON
- Domain không biết adapter tồn tại

#### Dependency Injection

Adapters được:

- Inject vào application tại composition root
- Không new trực tiếp trong domain

Lợi Ích:

- Dễ test
- Dễ thay thế adapter

#### Rule Sống Còn Của Adapter

- Adapter phục vụ domain, không điều khiển domain.
- Nếu adapter chứa logic nghiệp vụ thì sai.

## III. NGUYÊN TẮC & QUY TẮC THIẾT KẾ

### 1. Dependency Rule (Quy Tắc Phụ Thuộc)

#### Dependency Rule Là Gì?

Dependency Rule là quy tắc cốt lõi của Hexagonal Architecture: Mọi dependency trong hệ thống phải hướng vào trong, về phía Domain Core.

Domain không phụ thuộc vào:

- Adapters
- Infrastructure
- Framework

Adapters phụ thuộc vào Domain (thông qua Ports)

Đây không phải là quy ước folder, mà là quy tắc kiến trúc bắt buộc.

#### Vì Sao Dependency Rule Quan Trọng?

Nếu Domain phụ thuộc vào:

- File system
- Database
- CLI

Thì:

- Domain không thể test độc lập
- Mọi thay đổi kỹ thuật kéo theo thay đổi nghiệp vụ
- Business logic mất vai trò trung tâm

Dependency Rule đảm bảo:

- Domain luôn sạch
- Thay đổi công nghệ không ảnh hưởng đến nghiệp vụ

Trong Hexagonal Architecture:

- Ports chính là abstraction
- Domain định nghĩa abstraction
- Adapters implement abstraction

Đây là điểm khác biệt quan trọng so với layered architecture truyền thống.

#### Inversion of Control (IoC)

Hexagonal Architecture sử dụng Inversion of Control:

- Domain không tự tạo adapter
- Adapter được inject từ bên ngoài


### 2. Separation of Concerns (Phân Tách Trách Nhiệm)

#### Separation of Concerns Là Gì?

Separation of Concerns (SoC) là nguyên tắc:

Mỗi phần của hệ thống chỉ chịu trách nhiệm cho một mối quan tâm cụ thể.

Trong Hexagonal Architecture:

```
Domain → Nghiệp vụ
Application → Điều phối use cases
Adapters → Kết nối kỹ thuật
```


#### Configuration vs Implementation

Hexagonal Architecture tách:

Configuration  : Wiring, dependency injection 
Implementation : Logic thực thi               
Trong Project:

```
main.ts (Configuration)
├── Tạo adapter
├── Inject vào use case
│
Domain / application (Implementation)
├── Không biết ai tạo adapter
└── Chỉ know về logic
```

Lợi Ích:

- Dễ test
- Dễ thay adapter bằng mock/fake

### 3. Các Quy Tắc quan trọng Khi Áp Dụng Hexagonal Architecture

#### Rule 1: Domain Phải Độc Lập Tuyệt Đối

- Có thể compile riêng. Có thể test riêng. Không import infrastructure.

#### Rule 2: Ports Định Nghĩa Bởi Domain / Application

- Không định nghĩa interface trong adapter. Adapter chỉ implement.

#### Rule 3: Adapter Không Chứa Business Logic

- Adapter làm nhiệm vụ dịch. Không làm nhiệm vụ quyết định.

#### Rule 4: Composition Root Nằm Ngoài Domain

- Mọi wiring xảy ra ở một chỗ. Không new adapter trong domain.

#### Rule 5: Đơn Giản Hơn Là Tốt Hơn

- Không thêm abstraction nếu không cần. Hexagonal Architecture không yêu cầu phức tạp.

## IV. SO SÁNH & ALTERNATIVES

### 1. So Sánh Với Layered Architecture

#### Layered Architecture Là Gì?

Layered Architecture thường chia hệ thống thành:

```
Presentation (UI / CLI)
    ↓
Service
    ↓
Repository
    ↓
Database
```

#### Điểm Mạnh Của Layered Architecture

- Dễ hiểu
- Phù hợp project nhỏ
- Setup nhanh

#### Hạn Chế So Với Hexagonal Architecture

Business Logic Thường:

- Phụ thuộc ORM
- Phụ thuộc Database
- Khó test domain độc lập

Khi Đổi Công Nghệ:

- Thay đổi rất nhiều 

#### So Sánh Trực Tiếp

| Tiêu Chí                 | Layered | Hexagonal  |
| ------------------------ | ------- | ---------- |
| Domain Độc Lập           | Không   | Có         |
| Test Domain Không Cần DB | Không   | Có         |
| Dễ Thay Storage          | Không   | Có         |
| Độ Phức Tạp              | Thấp    | Trung Bình |

Với Ticket Manager CLI: Layered chạy được, nhưng khó mở rộng và test.

### 2. So Sánh Với Clean Architecture

#### Clean Architecture Là Gì?

Clean Architecture (Robert C. Martin) tập trung vào:

```
Domain / Entities
    ↓
Use Cases
    ↓
Interface Adapters
    ↓
Frameworks
```

Về Bản Chất: Clean Architecture và Hexagonal Architecture chia sẻ cùng một kiểu ý tưởng .

#### So Sánh Nhanh

| Tiêu Chí             | Clean     | Hexagonal        |
| -------------------- | --------- | ---------------- |
| Domain trung tâm     | Có        | Có               |
| Dependency Vào Trong | Có        | Có               |
| Testability Cao      | Có        | Có               |
| Trình Bày            | Vòng Tròn | Ports & Adapters |
| Ports Rõ Ràng        | Có phần   | Có               |

Hexagonal dễ giải thích hơn trong bối cảnh CLI.


### 3. Khi Nào Nên Sử Dụng Hexagonal Architecture?

#### Phù Hợp Khi:

- Có business logic rõ ràng
- Cần test nghiêm túc
- Có khả năng thay đổi: Storage, UI
- Project sống lâu

#### Không Nên Dùng Khi:

- Script nhỏ, one-off
- Không có domain logic
- Deadline cực gấp, team chưa quen

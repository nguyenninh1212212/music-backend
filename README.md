# Music Backend System – Microservice Architecture

## 1. Tổng quan

Hệ thống **Music Backend System** được xây dựng theo kiến trúc **Microservices**, phục vụ website nghe nhạc có tích hợp thanh toán và các chức năng realtime. Kiến trúc này giúp hệ thống **dễ mở rộng, dễ bảo trì và tách biệt rõ ràng nghiệp vụ** giữa các service.

Hệ thống hỗ trợ:

- Nghe nhạc và quản lý nội dung âm nhạc
- Thanh toán người dùng (PayPal)
- Tìm kiếm nhanh và chính xác
- Xử lý bất đồng bộ và realtime

---

## 2. Kiến trúc tổng thể

Hệ thống được thiết kế theo mô hình **Monolithic Core kết hợp các Service vệ tinh (Hybrid Architecture)**. Trong đó, **Music Service** đóng vai trò là khối monolithic trung tâm, xử lý phần lớn nghiệp vụ chính và điều phối các service chuyên biệt thông qua gRPC.

```
Client (Web / Mobile)
        │
        │ HTTP / WebSocket
        ▼
Music Service (Monolithic Core / Entry Point)
        │
        ├── REST API + JWT Authentication
        ├── Business Logic (Music, User, Order, Payment)
        │
        ├── gRPC / Protobuf
        │        ├──► Audio Service
        │        └──► Download (DL) Service
        │
        ├── RabbitMQ (Async Events)
        ├── Redis (Cache)
        ├── PostgreSQL (Database)
        └── Elasticsearch (Search)
```

Client (Web / Mobile)
│
│ HTTP
▼
Music Service đóng vai trò là **điểm truy cập duy nhất (Entry Point)** cho client, thay thế cho API Gateway truyền thống.

- Client gọi trực tiếp vào Music Service thông qua REST API
- Music Service chịu trách nhiệm:

  - Xác thực và phân quyền (JWT)
  - Điều phối request đến các service vệ tinh

---

## 3. Các thành phần chính

### 3.1 Entry Point – Music Service

- Không sử dụng API Gateway riêng
- Music Service trực tiếp nhận request từ client
- Đóng vai trò tương tự API Gateway ở quy mô nhỏ:

  - Xác thực JWT
  - Routing nội bộ bằng gRPC

---

### 3.2 Authentication & Authorization (Integrated)

- Chức năng xác thực được tích hợp trực tiếp trong Music Service
- Sử dụng **JWT** để xác thực và phân quyền
- Giảm độ phức tạp so với việc tách Auth Service riêng

---

### 3.3 Music Service (Monolithic Core)

- Là **khối trung tâm (core service)** của toàn hệ thống
- Cung cấp REST API cho client
- Quản lý nghiệp vụ chính:

  - Người dùng
  - Bài hát, album, nghệ sĩ
  - Đơn hàng và quyền truy cập nội dung

- Cache dữ liệu hot bằng **Redis**
- Gửi và nhận message bất đồng bộ qua **RabbitMQ**
- Giao tiếp với các service vệ tinh thông qua **gRPC/Protobuf**
- Link: https://github.com/nguyenninh1212212/script-subject

---

### 3.4 Audio Service

- Service chuyên biệt xử lý:

  - Lưu trữ và stream file audio
  - Metadata liên quan đến file âm thanh

- Được gọi từ Music Service thông qua **gRPC**
- Giúp tách tải nặng (audio processing) khỏi khối monolithic core
- Link: https://github.com/nguyenninh1212212/audio-service

---

### 3.5 DL Service

- Xử lý nghiệp vụ tạo embedding nhạc để phục vụ cho recommend nhạc
- Giao tiếp nội bộ với audio Service bằng **gRPC/Protobuf**
- link: https://github.com/nguyenninh1212212/dl-service

---

## 4. Giao tiếp giữa các service

### 🔹 REST + JWT

- Client giao tiếp với hệ thống thông qua REST API
- JWT được sử dụng để xác thực và phân quyền

### 🔹 gRPC / Protobuf

- Giao tiếp nội bộ giữa các microservice
- Tối ưu hiệu năng và giảm overhead so với REST

### 🔹 RabbitMQ (Message Queue)

- Truyền tải message bất đồng bộ
- Xử lý các tác vụ nền như:

  - Lưu lịch sử nghe nhạc
  - Gửi thông báo
  - Đồng bộ dữ liệu

---

## 5. Lưu trữ & tối ưu hiệu năng

### 🔸 PostgreSQL

- Database chính
- Lưu trữ dữ liệu người dùng, bài hát, giao dịch

### 🔸 Redis

- Cache dữ liệu truy cập nhiều
- Giảm tải cho database

### 🔸 Elasticsearch

- Tìm kiếm bài hát, album, nghệ sĩ
- Hỗ trợ full-text search nhanh và chính xác

### 🔸 MongoDb

- Chur yếu lưu embedding nhạc phục vụ cho mục đích gợi ý bài nhạc

## 8. Công nghệ sử dụng

- **Backend**: Spring Boot
- **Database**: PostgreSQL
- **Cache**: Redis
- **Message Queue**: RabbitMQ
- **Search Engine**: Elasticsearch
- **Communication**: REST, gRPC, Protobuf
- **Authentication**: JWT
- **Payment**: PayPal



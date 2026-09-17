# Cinema Log & Release Radar

> ระบบบันทึกความทรงจำการดูหนัง และเตือนตารางหนังเข้าฉาย พัฒนาด้วย Spring Boot ตามหลัก Layered Architecture, SOLID Principles และ Design Patterns

**วิชา:** CP353002 Principles of Software Design and Development

---

## 1. ชื่อโปรเจค

**Cinema Log & Release Radar**
(ระบบบันทึกความทรงจำการดูหนังและเตือนตารางหนังเข้า)

ระบบให้ผู้ใช้บันทึกว่าดูหนังเรื่องไหนไปแล้ว ให้คะแนนและเขียนรีวิว พร้อมทั้งตั้งเตือนหนังที่กำลังจะเข้าฉายที่ตนเองสนใจ เมื่อถึงวันฉายจริงระบบจะแจ้งเตือนอัตโนมัติผ่านช่องทางที่ผู้ใช้เลือกไว้ ข้อมูลหนังทั้งหมดดึงมาจาก TMDB (The Movie Database) API แล้ว sync เก็บไว้ในฐานข้อมูลของระบบเอง

---

## 2. รายชื่อสมาชิกและหน้าที่รับผิดชอบ

| ลำดับ | รหัสนักศึกษา | ชื่อ-นามสกุล | Branch | หน้าที่รับผิดชอบ |
|---|---|---|---|---|
| 1 | 673380409-0 | นางสาวนันทพร ลุนทอง | `นันทพร_673380409-0_01` | Watch Log, Review & Release Radar |
| 2 | 673380591-5 | นางสาวปรายฝน ฮกเซ็ง | `ปรายฝน_673380591-5_01` | User & Infrastructure Lead |
| 3 | 673380592-3 | นางสาวปวริศา สีดาชมภู | `ปวริศา_673380592-3_01` | Movie & External Integration |

> ชื่อ branch ด้านบนเป็นตัวอย่างตามรูปแบบที่โจทย์กำหนด (`ชื่อ_รหัสนักศึกษา_section`) ให้แต่ละคนแทนที่ด้วย section จริงของตนเอง

---

## 3. การเตรียมโปรเจกต์

### 3.1 สร้างโปรเจกต์

สร้างโปรเจกต์ Spring Boot จาก [start.spring.io](https://start.spring.io) โดยเลือก Dependency ดังนี้

| Dependency | ใช้ทำอะไร |
|---|---|
| Spring Web | สร้าง REST API |
| Spring Data JPA | ติดต่อฐานข้อมูลผ่าน Hibernate |
| PostgreSQL Driver | ตัวเชื่อมต่อกับฐานข้อมูล PostgreSQL |
| Validation | ใช้ `@Valid` / Bean Validation กับ Request DTO |
| Spring Boot DevTools | Hot reload ระหว่างพัฒนา |
| Springdoc OpenAPI (Swagger) | สร้างเอกสาร API อัตโนมัติ (เพิ่มผ่าน `pom.xml` เอง เพราะไม่มีใน start.spring.io) |
| Spring Boot Starter Test | สำหรับ JUnit 5 + Mockito |
| Lombok | ลด boilerplate code ของ Entity/DTO |
| Flyway Migration | จัดการ database migration script |

Dependency เพิ่มเติมที่ต้องใส่เองใน `pom.xml`:

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

### 3.2 เตรียมฐานข้อมูล

เปิด pgAdmin หรือ psql แล้วสร้าง Database สำหรับโปรเจคนี้

```sql
CREATE DATABASE cinema_log_db;
```

### 3.3 ตั้งค่า application.properties

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/cinema_log_db
spring.datasource.username=postgres
spring.datasource.password=YOUR_PASSWORD_HERE

spring.jpa.show-sql=true
spring.jpa.hibernate.ddl-auto=validate
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect

spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration

# TMDB External API
tmdb.api.base-url=https://api.themoviedb.org/3
tmdb.api.read-access-token=YOUR_TMDB_TOKEN_HERE
```

> **สำคัญ:** เปลี่ยน `YOUR_PASSWORD_HERE` และ `YOUR_TMDB_TOKEN_HERE` เป็นค่าจริงของตนเอง **ห้าม commit ค่าจริงขึ้น GitHub** ให้ใช้ environment variable หรือไฟล์ `application-local.properties` ที่ใส่ไว้ใน `.gitignore` แทน
> ใช้ `ddl-auto=validate` แทน `update` เพราะโปรเจคนี้ควบคุม schema ผ่าน Flyway migration script ตามข้อกำหนด

---

## 4. โครงสร้าง Project

```
cinema-log-release-radar/
├── code/
│   └── backend/
│       ├── src/main/java/com/example/cinemalog/
│       │   ├── CinemaLogApplication.java
│       │   ├── config/
│       │   │   ├── SwaggerConfig.java
│       │   │   ├── SecurityConfig.java
│       │   │   ├── WebClientConfig.java
│       │   │   └── AsyncConfig.java
│       │   ├── controller/
│       │   │   ├── api/
│       │   │   │   ├── UserController.java
│       │   │   │   ├── MovieController.java
│       │   │   │   ├── WatchLogController.java
│       │   │   │   ├── ReviewController.java
│       │   │   │   └── ReleaseAlertController.java
│       │   │   └── web/
│       │   │       └── HomeViewController.java
│       │   ├── service/
│       │   │   ├── UserService.java
│       │   │   ├── MovieService.java
│       │   │   ├── WatchLogService.java
│       │   │   ├── ReviewService.java
│       │   │   ├── ReleaseAlertService.java
│       │   │   ├── NotificationService.java
│       │   │   └── impl/
│       │   │       ├── UserServiceImpl.java
│       │   │       ├── MovieServiceImpl.java
│       │   │       ├── WatchLogServiceImpl.java
│       │   │       ├── ReviewServiceImpl.java
│       │   │       ├── ReleaseAlertServiceImpl.java
│       │   │       └── NotificationServiceImpl.java
│       │   ├── repository/
│       │   │   ├── UserRepository.java
│       │   │   ├── UserProfileRepository.java
│       │   │   ├── MovieRepository.java
│       │   │   ├── GenreRepository.java
│       │   │   ├── WatchLogRepository.java
│       │   │   ├── ReviewRepository.java
│       │   │   ├── ReleaseAlertRepository.java
│       │   │   └── NotificationRepository.java
│       │   ├── domain/
│       │   │   ├── entity/
│       │   │   │   ├── User.java
│       │   │   │   ├── UserProfile.java
│       │   │   │   ├── Movie.java
│       │   │   │   ├── Genre.java
│       │   │   │   ├── WatchLog.java
│       │   │   │   ├── Review.java
│       │   │   │   ├── ReleaseAlert.java
│       │   │   │   └── Notification.java
│       │   │   └── enums/
│       │   │       ├── AlertStatus.java
│       │   │       └── NotificationChannel.java
│       │   ├── dto/
│       │   │   ├── request/
│       │   │   │   ├── UserRequestDTO.java
│       │   │   │   ├── WatchLogRequestDTO.java
│       │   │   │   ├── ReviewRequestDTO.java
│       │   │   │   └── ReleaseAlertRequestDTO.java
│       │   │   └── response/
│       │   │       ├── UserResponseDTO.java
│       │   │       ├── MovieResponseDTO.java
│       │   │       ├── WatchLogResponseDTO.java
│       │   │       ├── ReviewResponseDTO.java
│       │   │       ├── ReleaseAlertResponseDTO.java
│       │   │       ├── PagedResponseDTO.java
│       │   │       └── ErrorResponseDTO.java
│       │   ├── mapper/
│       │   │   ├── UserMapper.java
│       │   │   ├── MovieMapper.java
│       │   │   ├── WatchLogMapper.java
│       │   │   ├── ReviewMapper.java
│       │   │   └── ReleaseAlertMapper.java
│       │   ├── pattern/
│       │   │   ├── strategy/
│       │   │   │   ├── NotificationStrategy.java
│       │   │   │   ├── EmailNotificationStrategy.java
│       │   │   │   ├── PushNotificationStrategy.java
│       │   │   │   ├── InAppNotificationStrategy.java
│       │   │   │   └── NotificationStrategyFactory.java
│       │   │   ├── state/
│       │   │   │   ├── AlertState.java
│       │   │   │   ├── PendingState.java
│       │   │   │   ├── NotifiedState.java
│       │   │   │   └── ExpiredState.java
│       │   │   ├── observer/
│       │   │   │   ├── MovieReleasedEvent.java
│       │   │   │   └── ReleaseAlertListener.java
│       │   │   └── adapter/
│       │   │       ├── TmdbClient.java
│       │   │       ├── TmdbMovieResponse.java
│       │   │       └── TmdbMovieAdapter.java
│       │   ├── exception/
│       │   │   ├── GlobalExceptionHandler.java
│       │   │   ├── MovieNotFoundException.java
│       │   │   ├── DuplicateAlertException.java
│       │   │   └── ValidationException.java
│       │   └── common/
│       │       ├── util/DateUtils.java
│       │       └── constant/ApiConstants.java
│       ├── src/main/resources/
│       │   ├── application.properties
│       │   ├── application-local.properties
│       │   ├── db/migration/
│       │   │   ├── V1__init_schema.sql
│       │   │   ├── V2__seed_genres.sql
│       │   │   └── V3__add_indexes.sql
│       │   └── templates/
│       ├── pom.xml
│       ├── Dockerfile
│       └── docker-compose.yml
│   └── frontend/                      # React (หรือ Thymeleaf ถ้าไม่แยก)
│       ├── src/
│       │   ├── components/
│       │   ├── pages/
│       │   └── services/api.js
│       └── package.json
├── test/
│   └── src/test/java/com/example/cinemalog/
│       ├── service/
│       ├── controller/
│       └── repository/
├── doc/
│   ├── solid-analysis.md
│   ├── design-patterns.md
│   ├── data-dictionary.md
│   ├── diagrams/
│   └── slide/
├── img/
├── .github/workflows/ci-cd.yml
├── .gitignore
└── README.md
```

---

## 5. Model (Entity)

### 5.1 User — รับผิดชอบโดย: ปรายฝน (User & Infrastructure Lead)

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสผู้ใช้ |
| `username` | String | - | ชื่อผู้ใช้ |
| `email` | String | - | อีเมล |
| `password` | String | - | รหัสผ่าน (เก็บแบบ hashed) |

**ความสัมพันธ์:** `User` (1) — (1) `UserProfile`, `User` (1) — (N) `WatchLog`, `User` (1) — (N) `ReleaseAlert`

### 5.2 UserProfile — รับผิดชอบโดย: ปรายฝน

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสโปรไฟล์ |
| `bio` | String | - | คำอธิบายตัวเอง |
| `avatarUrl` | String | - | รูปโปรไฟล์ |
| `preferredChannel` | Enum (`NotificationChannel`) | - | ช่องทางแจ้งเตือนที่ต้องการ |
| `user_id` | Long | FK | อ้างอิง `User` (ฝั่ง owner) |

**ความสัมพันธ์:** `UserProfile` (1) — (1) `User`

### 5.3 Movie — รับผิดชอบโดย: ปวริศา (Movie & External Integration)

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสหนังในระบบ |
| `tmdbId` | Long | - | รหัสอ้างอิงจาก TMDB |
| `title` | String | - | ชื่อหนัง |
| `overview` | String | - | เรื่องย่อ |
| `posterUrl` | String | - | URL โปสเตอร์ |
| `releaseDate` | LocalDate | - | วันเข้าฉาย |

**ความสัมพันธ์:** `Movie` (N) — (N) `Genre`, `Movie` (1) — (N) `WatchLog`, `Movie` (1) — (N) `Review`, `Movie` (1) — (N) `ReleaseAlert`

### 5.4 Genre — รับผิดชอบโดย: ปวริศา

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสแนวหนัง |
| `name` | String | - | ชื่อแนวหนัง (Action, Drama, ...) |

**ความสัมพันธ์:** `Genre` (N) — (N) `Movie` ผ่านตาราง join `movie_genres`

### 5.5 WatchLog — รับผิดชอบโดย: นันทพร (Watch Log, Review & Release Radar)

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสรายการที่บันทึก |
| `watchedDate` | LocalDate | - | วันที่ดู |
| `rating` | Integer | - | คะแนนที่ให้ (1-10) |
| `note` | String | - | บันทึกส่วนตัว |
| `user_id` | Long | FK | อ้างอิง `User` |
| `movie_id` | Long | FK | อ้างอิง `Movie` |

**ความสัมพันธ์:** `WatchLog` (N) — (1) `User`, `WatchLog` (N) — (1) `Movie`

### 5.6 Review — รับผิดชอบโดย: นันทพร

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสรีวิว |
| `content` | String | - | เนื้อหารีวิว |
| `score` | Integer | - | คะแนน |
| `createdAt` | LocalDateTime | - | วันที่เขียนรีวิว |
| `movie_id` | Long | FK | อ้างอิง `Movie` |
| `user_id` | Long | FK | อ้างอิง `User` |

**ความสัมพันธ์:** `Review` (N) — (1) `Movie`, `Review` (N) — (1) `User`

### 5.7 ReleaseAlert — รับผิดชอบโดย: นันทพร

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสการตั้งเตือน |
| `status` | Enum (`AlertStatus`) | - | สถานะ: `PENDING` / `NOTIFIED` / `EXPIRED` |
| `createdAt` | LocalDateTime | - | วันที่ตั้งเตือน |
| `user_id` | Long | FK | อ้างอิง `User` |
| `movie_id` | Long | FK | อ้างอิง `Movie` |

**ความสัมพันธ์:** `ReleaseAlert` (N) — (1) `User`, `ReleaseAlert` (N) — (1) `Movie`, `ReleaseAlert` (1) — (N) `Notification`

### 5.8 Notification — รับผิดชอบโดย: นันทพร

| Column | Type | Key | Description |
|---|---|---|---|
| `id` | Long | PK | รหัสการแจ้งเตือน |
| `channel` | Enum (`NotificationChannel`) | - | ช่องทางที่ส่ง |
| `sentAt` | LocalDateTime | - | เวลาที่ส่ง |
| `success` | Boolean | - | ส่งสำเร็จหรือไม่ |
| `release_alert_id` | Long | FK | อ้างอิง `ReleaseAlert` |

**ความสัมพันธ์:** `Notification` (N) — (1) `ReleaseAlert`

---

## 6. Repository

| Repository | Method หลัก | ผู้รับผิดชอบ |
|---|---|---|
| `UserRepository` | `findByEmail`, `findByUsername` | ปรายฝน |
| `UserProfileRepository` | `findByUserId` | ปรายฝน |
| `MovieRepository` | `findByTmdbId`, `findByGenresId`, `findAll(Pageable)` | ปวริศา |
| `GenreRepository` | `findByName` | ปวริศา |
| `WatchLogRepository` | `findByUserId`, `findByMovieId` | นันทพร |
| `ReviewRepository` | `findByMovieId`, `findAverageScoreByMovieId` | นันทพร |
| `ReleaseAlertRepository` | `findByUserId`, `findByStatusAndMovieReleaseDate` | นันทพร |
| `NotificationRepository` | `findByReleaseAlertId` | นันทพร |

> ทุก Repository extends `JpaRepository<Entity, Long>` ตามหลัก **Dependency Inversion Principle** — Service เรียกผ่าน interface นี้ ไม่รู้จัก implementation ของ Hibernate โดยตรง

---

## 7. Service

### 7.1 UserService — ผู้รับผิดชอบ: ปรายฝน

| Method | ทำงานอะไร |
|---|---|
| `register(UserRequestDTO)` | สร้างผู้ใช้ใหม่ พร้อมสร้าง `UserProfile` เริ่มต้น |
| `getUserById(Long)` | ค้นหาผู้ใช้จาก id |
| `updateProfile(Long, UserProfileRequestDTO)` | แก้ไขข้อมูลโปรไฟล์ |

### 7.2 MovieService — ผู้รับผิดชอบ: ปวริศา

| Method | ทำงานอะไร |
|---|---|
| `searchFromTmdb(String keyword)` | เรียก TMDB ผ่าน `TmdbClient` แล้วแปลงผ่าน `TmdbMovieAdapter` |
| `syncMovie(Long tmdbId)` | ดึงรายละเอียดหนังจาก TMDB แล้วบันทึก/อัปเดตลงฐานข้อมูลตัวเอง |
| `getMovies(Pageable)` | คืนรายการหนังพร้อม pagination/sorting |
| `getMovieById(Long)` | ดูรายละเอียดหนัง 1 เรื่อง |

### 7.3 WatchLogService — ผู้รับผิดชอบ: นันทพร

| Method | ทำงานอะไร |
|---|---|
| `addWatchLog(WatchLogRequestDTO)` | บันทึกว่า user ดูหนังเรื่องนี้แล้ว |
| `getWatchLogsByUser(Long userId)` | ดูประวัติการดูหนังทั้งหมดของ user |
| `deleteWatchLog(Long id)` | ลบ log ที่บันทึกผิด |

### 7.4 ReviewService — ผู้รับผิดชอบ: นันทพร

| Method | ทำงานอะไร |
|---|---|
| `addReview(ReviewRequestDTO)` | เพิ่มรีวิวให้หนัง |
| `getReviewsByMovie(Long movieId)` | ดูรีวิวทั้งหมดของหนังเรื่องนั้น |
| `getAverageScore(Long movieId)` | คำนวณคะแนนเฉลี่ยของหนัง |

### 7.5 ReleaseAlertService — ผู้รับผิดชอบ: นันทพร

| Method | ทำงานอะไร |
|---|---|
| `createAlert(ReleaseAlertRequestDTO)` | ตั้งเตือนหนังที่ยังไม่ฉาย (สถานะเริ่มต้น `PENDING`) |
| `checkAndPublishReleaseEvents()` | Scheduled job เช็คว่าหนังเรื่องไหนถึงวันฉายแล้ว publish `MovieReleasedEvent` (**Observer**) |
| `updateAlertState(Long alertId)` | เปลี่ยนสถานะ alert ผ่าน `AlertState` (**State**) |
| `cancelAlert(Long id)` | ยกเลิกการติดตาม |

### 7.6 NotificationService — ผู้รับผิดชอบ: นันทพร

| Method | ทำงานอะไร |
|---|---|
| `notifyUser(ReleaseAlert alert)` | เลือก `NotificationStrategy` ตามช่องทางที่ user ตั้งไว้ผ่าน `NotificationStrategyFactory` แล้วส่งจริง (**Strategy**) |
| `logNotification(...)` | บันทึกผลการส่งลงตาราง `Notification` |

---

## 8. GoF Patterns (เลือกกลุ่ม Behavioral — 3 แบบ)

| Pattern | ปัญหาที่แก้ | ไฟล์/คลาสที่ใช้ | ผู้รับผิดชอบ |
|---|---|---|---|
| **Observer** | เมื่อหนังถึงวันฉาย ต้องแจ้งผู้ที่ตั้งเตือนไว้ทุกคน โดยไม่ให้ business logic หลักผูกติดกับวิธีการแจ้งเตือน | `pattern/observer/MovieReleasedEvent.java`, `ReleaseAlertListener.java` | นันทพร |
| **Strategy** | ผู้ใช้เลือกช่องทางแจ้งเตือนต่างกัน (Email/Push/In-app) เพิ่มช่องทางใหม่ได้โดยไม่แก้โค้ดเดิม | `pattern/strategy/NotificationStrategy.java` และ implementation ต่างๆ | นันทพร |
| **State** | `ReleaseAlert` มีสถานะเปลี่ยนตามเวลา (`PENDING → NOTIFIED → EXPIRED`) จัดการ transition โดยไม่ใช้ if-else ซ้อน | `pattern/state/AlertState.java` และ implementation ต่างๆ | นันทพร |

**Pattern เสริม (Structural):**

| Pattern | ปัญหาที่แก้ | ไฟล์/คลาสที่ใช้ | ผู้รับผิดชอบ |
|---|---|---|---|
| **Adapter** | แปลงโครงสร้าง JSON จาก TMDB ให้เข้ากับ domain model ของระบบ โดยไม่ให้ business logic รู้จักโครงสร้างของ TMDB โดยตรง | `pattern/adapter/TmdbClient.java`, `TmdbMovieResponse.java`, `TmdbMovieAdapter.java` | ปวริศา |

> รายละเอียดพร้อมเหตุผลและ Class Diagram อยู่ที่ `doc/design-patterns.md`

---

## 9. การทดสอบโปรเจค (Testing)

### 9.1 ระดับ Unit Test (JUnit 5 + Mockito)

| Service ที่ทดสอบ | Test Case ตัวอย่าง | ผลลัพธ์ที่คาดหวัง |
|---|---|---|
| `WatchLogServiceImpl` | เพิ่ม watch log ด้วย movieId ที่มีอยู่จริง | บันทึกสำเร็จ คืน `WatchLogResponseDTO` |
| `WatchLogServiceImpl` | เพิ่ม watch log ด้วย movieId ที่ไม่มีในระบบ | โยน `MovieNotFoundException` |
| `ReleaseAlertServiceImpl` | ตั้งเตือนหนังเรื่องเดิมซ้ำ | โยน `DuplicateAlertException` |
| `ReleaseAlertServiceImpl` | เรียก `checkAndPublishReleaseEvents()` เมื่อถึงวันฉาย | มีการ publish `MovieReleasedEvent` (mock `ApplicationEventPublisher`) |
| `NotificationServiceImpl` | ผู้ใช้เลือกช่องทาง `EMAIL` | เรียกใช้ `EmailNotificationStrategy` ไม่ใช่ตัวอื่น (mock Factory) |
| `MovieServiceImpl` | เรียก `syncMovie()` เมื่อ TMDB ตอบกลับสำเร็จ | ข้อมูลถูกแปลงผ่าน Adapter และบันทึกถูกต้อง |

### 9.2 ระดับ Integration Test (Spring Boot Test)

- ทดสอบ `WatchLogController` ยิง request จริงผ่าน `MockMvc` แล้วตรวจสอบ status code และ response format
- ทดสอบ Repository ด้วย `@DataJpaTest` เช็คว่าความสัมพันธ์ 1:1 / 1:N / M:N ทำงานถูกต้องจริงในฐานข้อมูลทดสอบ (H2 หรือ Testcontainers)

### 9.3 ทดสอบผ่าน Postman / Swagger UI

| Endpoint | Method | ผลลัพธ์ที่คาดหวัง |
|---|---|---|
| `POST /api/v1/users` | POST | `201 Created` พร้อมข้อมูล user ที่สร้าง |
| `GET /api/v1/movies?page=0&size=10` | GET | `200 OK` พร้อม pagination metadata |
| `POST /api/v1/movies/{id}/watch-logs` | POST | `201 Created` |
| `POST /api/v1/movies/{id}/watch-logs` (movieId ไม่มีจริง) | POST | `404 Not Found` พร้อม error format มาตรฐาน |
| `POST /api/v1/users/{id}/release-alerts` (ตั้งซ้ำ) | POST | `409 Conflict` |

### 9.4 Test Report

รัน `mvn test` แล้ว export ผลลัพธ์เก็บไว้ที่ `test/report/` เพื่อแนบเป็นหลักฐานตาม checklist ของโจทย์

---

## 10. Git Workflow & ขั้นตอนการใช้งาน

### 10.1 กฎการตั้งชื่อ Branch

```
ชื่อ_รหัสนักศึกษา_section
```

ตัวอย่างของกลุ่มนี้:
```
นันทพร_673380409-0_01
ปรายฝน_673380591-5_01
ปวริศา_673380592-3_01
```

### 10.2 โครงสร้าง Branch

| Branch | หน้าที่ |
|---|---|
| `main` | Production — merge ได้เฉพาะเวอร์ชันที่ส่งมอบ |
| `develop` | Integration — รวมงานจากทุกคน |
| `ชื่อ_รหัส_section` | Branch ส่วนตัวของแต่ละคน |

### 10.3 ขั้นตอนเริ่มงาน (ทำครั้งแรกเท่านั้น)

```bash
# 1. ตั้งค่า git ให้ตรงกับบัญชี GitHub ของตัวเอง
git config user.name "ชื่อ-GitHub-ของตัวเอง"
git config user.email "อีเมลที่ผูกกับบัญชี GitHub"

# 2. Clone repository
git clone https://github.com/<กลุ่ม>/cinema-log-release-radar.git
cd cinema-log-release-radar

# 3. สลับไปที่ develop แล้วสร้าง branch ของตัวเอง
git checkout develop
git pull origin develop
git checkout -b นันทพร_673380409-0_01
```

### 10.4 ขั้นตอนทำงานประจำวัน

```bash
# ก่อนเริ่มงานทุกครั้ง ให้ดึงงานล่าสุดของ develop มาก่อน
git checkout develop
git pull origin develop
git checkout นันทพร_673380409-0_01
git merge develop            # เอาโค้ดล่าสุดของทีมมารวมกับ branch ตัวเอง

# เขียนโค้ด แล้ว commit เป็นชิ้นเล็กๆ มีความหมาย
git add .
git commit -m "feat: add watch log creation endpoint"

# push ขึ้น branch ของตัวเอง (ห้าม push ตรงเข้า main/develop)
git push origin นันทพร_673380409-0_01
```

### 10.5 การรวมงานเข้า develop

1. เปิด Pull Request จาก branch ของตัวเองไปยัง `develop`
2. ให้เพื่อนในทีมอย่างน้อย 1 คน Review ก่อน Merge
3. หลัง Merge แล้ว **ลบ branch เก่าใน remote ได้** (แต่เก็บ history ไว้ใน commit log)

### 10.6 Commit Message Convention

```
<type>: <สิ่งที่ทำ>
```

ตัวอย่าง:
```
feat: add release alert creation API
fix: correct notification channel selection logic
refactor: extract alert state interface
test: add unit test for WatchLogService
docs: update solid-analysis.md
```

### 10.7 กฎเหล็กที่ต้องระวัง

- Commit/Push ด้วยบัญชี GitHub ของตนเองเท่านั้น ห้ามฝากคนอื่นทำแทน
- ทุกคนต้องมี commit ที่มีความหมาย **ไม่น้อยกว่า 15 ครั้ง** กระจายตลอดช่วงเวลาทำโปรเจค (ห้าม commit รวดเดียวก่อนส่ง)
- การรวมงานทุกครั้งต้องผ่าน Pull Request พร้อม Reviewer อย่างน้อย 1 คน
- ห้าม Outsource หรือให้คนนอกกลุ่มเขียนโค้ดให้เด็ดขาด

---

## Tech Stack

- **Backend:** Spring Boot 3.x (Java 17+), Spring Data JPA (Hibernate)
- **Database:** PostgreSQL + Flyway Migration
- **API Docs:** Swagger / OpenAPI (`/swagger-ui.html`)
- **Frontend:** React (หรือ Thymeleaf)
- **Testing:** JUnit 5 + Mockito + Spring Boot Test
- **External API:** TMDB (The Movie Database)
- **Deployment:** Docker + Docker Compose, Render/Railway
- **CI/CD:** GitHub Actions (Build → Test → Deploy)

## Deployment URL

_(กรอกลิงก์หลัง deploy สำเร็จ)_

## API Documentation

Swagger UI: `http://<deploy-url>/swagger-ui.html`

# BambangShop Receiver App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a Rocket web framework skeleton that you can work with.

As this is an Observer Design Pattern tutorial repository, you need to implement a feature: `Notification`.
This feature will receive notifications of creation, promotion, and deletion of a product, when this receiver instance is subscribed to a certain product type.
The notification will be sent using HTTP POST request, so you need to make the receiver endpoint in this project.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Receiver" folder.

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    ROCKET_PORT=8001
    APP_INSTANCE_ROOT_URL=http://localhost:${ROCKET_PORT}
    APP_PUBLISHER_ROOT_URL=http://localhost:8000
    APP_INSTANCE_NAME=Safira Sudrajat
    ```
    Here are the details of each environment variable:
    | variable                | type   | description                                                     |
    |-------------------------|--------|-----------------------------------------------------------------|
    | ROCKET_PORT             | string | Port number that will be listened by this receiver instance.    |
    | APP_INSTANCE_ROOT_URL   | string | URL address where this receiver instance can be accessed.       |
    | APP_PUUBLISHER_ROOT_URL | string | URL address where the publisher instance can be accessed.       |
    | APP_INSTANCE_NAME       | string | Name of this receiver instance, will be shown on notifications. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)
3.  To simulate multiple instances of BambangShop Receiver (as the tutorial mandates you to do so),
    you can open new terminal, then edit `ROCKET_PORT` in `.env` file, then execute another `cargo run`.

    For example, if you want to run 3 (three) instances of BambangShop Receiver at port `8001`, `8002`, and `8003`, you can do these steps:
    -   Edit `ROCKET_PORT` in `.env` to `8001`, then execute `cargo run`.
    -   Open new terminal, edit `ROCKET_PORT` in `.env` to `8002`, then execute `cargo run`.
    -   Open another new terminal, edit `ROCKET_PORT` in `.env` to `8003`, then execute `cargo run`.

## Mandatory Checklists (Subscriber)
-   [ ] Clone https://gitlab.com/ichlaffterlalu/bambangshop-receiver to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [ ] Commit: `Create Notification model struct.`
    -   [ ] Commit: `Create SubscriberRequest model struct.`
    -   [ ] Commit: `Create Notification database and Notification repository struct skeleton.`
    -   [ ] Commit: `Implement add function in Notification repository.`
    -   [ ] Commit: `Implement list_all_as_string function in Notification repository.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-1" questions in this README.
-   **STAGE 3: Implement services and controllers**
    -   [ ] Commit: `Create Notification service struct skeleton.`
    -   [ ] Commit: `Implement subscribe function in Notification service.`
    -   [ ] Commit: `Implement subscribe function in Notification controller.`
    -   [ ] Commit: `Implement unsubscribe function in Notification service.`
    -   [ ] Commit: `Implement unsubscribe function in Notification controller.`
    -   [ ] Commit: `Implement receive_notification function in Notification service.`
    -   [ ] Commit: `Implement receive function in Notification controller.`
    -   [ ] Commit: `Implement list_messages function in Notification service.`
    -   [ ] Commit: `Implement list function in Notification controller.`
    -   [ ] Write answers of your learning module's "Reflection Subscriber-2" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Subscriber) Reflections

## Reflection Subscriber-1

### 1. Pada tutorial ini, kita menggunakan `RwLock<>` untuk melakukan sinkronisasi terhadap `Vec<Notification>`. Jelaskan mengapa hal ini diperlukan untuk kasus ini, dan jelaskan juga mengapa kita tidak menggunakan `Mutex<>`?

Menurut saya, `RwLock<Vec<Notification>>` diperlukan karena data `NOTIFICATIONS` digunakan sebagai shared state yang bisa diakses oleh lebih dari satu request atau thread. Pada Receiver app, notifikasi dapat terus bertambah ketika main app mengirim request baru ke endpoint receiver. Di saat yang sama, receiver app juga bisa membaca seluruh notifikasi untuk ditampilkan ke pengguna. Karena ada kemungkinan proses baca dan tulis terjadi dari beberapa thread, kita membutuhkan mekanisme sinkronisasi agar akses ke `Vec<Notification>` tetap aman.

`RwLock` cocok untuk kasus ini karena pola akses datanya cenderung terdiri dari:
- **write** saat notifikasi baru ditambahkan
- **read** saat seluruh notifikasi ingin ditampilkan

Dengan `RwLock`, banyak proses pembacaan bisa berjalan bersamaan selama tidak ada proses penulisan. Ini lebih efisien dibandingkan `Mutex`, karena `Mutex` hanya mengizinkan satu akses dalam satu waktu, baik untuk read maupun write. Jadi, kalau memakai `Mutex`, bahkan beberapa proses baca yang sebenarnya aman dilakukan bersamaan tetap harus antre satu per satu.

Karena itu, menurut saya `RwLock` lebih tepat untuk kasus ini: kita tetap aman dari race condition, tetapi performa pembacaan juga lebih baik dibandingkan jika memakai `Mutex`.

### 2. Pada tutorial ini, kita menggunakan library eksternal `lazy_static` untuk mendefinisikan `Vec` dan `DashMap` sebagai variabel “static”. Dibandingkan dengan Java, di mana kita bisa memutasi isi static variable melalui static function, mengapa Rust tidak mengizinkan kita melakukan hal tersebut?

Menurut saya, Rust tidak mengizinkan mutasi langsung pada static variable biasa karena Rust sangat ketat dalam menjaga **memory safety** dan **thread safety**. Jika static variable dapat diubah secara bebas dari berbagai tempat seperti di Java, maka akan lebih mudah terjadi data race, terutama dalam program yang berjalan secara konkuren.

Rust ingin memastikan bahwa setiap mutasi terhadap shared state dilakukan secara eksplisit dan aman. Karena itu, data global yang dapat berubah tidak cukup hanya dideklarasikan sebagai `static`, tetapi juga perlu dibungkus dengan mekanisme sinkronisasi yang aman, seperti `RwLock`, `Mutex`, atau struktur concurrent lain seperti `DashMap`.

`lazy_static` membantu karena:
1. ia memungkinkan inisialisasi data global yang tidak bisa ditentukan sepenuhnya saat compile-time,
2. ia memberi cara yang aman dan terkontrol untuk membuat shared global state,
3. ia bekerja dengan struktur sinkronisasi yang sesuai dengan prinsip ownership dan borrowing di Rust.

Berbeda dengan Java yang lebih longgar terhadap mutasi objek static, Rust sengaja membatasi hal ini supaya programmer tidak secara tidak sengaja membuat shared mutable state yang berbahaya. Jadi, menurut saya, alasan utama Rust tidak mengizinkan mutasi static biasa seperti Java adalah untuk menjaga keamanan program, terutama dalam konteks concurrency.

#### Reflection Subscriber-2

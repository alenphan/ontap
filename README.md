# TÀI LIỆU ÔN TẬP TOÀN DIỆN KIẾN THỨC DỰ ÁN WEB & SPRING BOOT

> Tài liệu tổng hợp kiến trúc hệ thống, phân tích chi tiết các thẻ HTML, đánh giá ưu/nhược điểm các giải pháp kỹ thuật, và cẩm nang chuyên sâu bóc tách toàn bộ cấu trúc của **Fetch API** và **Axios**.

---

## MỤC LỤC
1. [Kiến trúc hệ thống Spring Boot trong dự án](#1-kiến-trúc-hệ-thống-spring-boot-trong-dự-án)
2. [Ưu điểm & Nhược điểm của các giải pháp trong dự án](#2-ưu-điểm--nhược-điểm-của-các-giải-pháp-trong-dự-án)
3. [Phân tích toàn bộ thẻ HTML có trong dự án & Các thẻ tương đương](#3-phân-tích-toàn-bộ-thẻ-html-có-trong-dự-án--các-thẻ-tương-đương)
4. [Bảng so sánh tổng quan: jQuery vs Fetch API vs Axios](#4-bảng-so-sánh-tổng-quan-jquery-vs-fetch-api-vs-axios)
5. [Bóc tách toàn bộ cấu trúc chuyên sâu của FETCH API](#5-bóc-tách-toàn-bộ-cấu-trúc-chuyên-sâu-của-fetch-api)
6. [Bóc tách toàn bộ cấu trúc chuyên sâu của AXIOS](#6-bóc-tách-toàn-bộ-cấu-trúc-chuyên-sâu-của-axios)
7. [Bảng đối chiếu 1-1 các thuộc tính giữa Fetch API và Axios](#7-bảng-đối-chiếu-1-1-các-thuộc-tính-giữa-fetch-api-và-axios)
8. [Cú pháp thực chiến (Cheat Sheet) cho Frontend & Backend](#8-cú-pháp-thực-chiến-cheat-sheet-cho-frontend--backend)

---

## 1. KIẾN TRÚC HỆ THỐNG SPRING BOOT TRONG DỰ ÁN

Dự án áp dụng mô hình kiến trúc phân tầng kết hợp **Client-Server** và **RESTful API**:

```
[Browser Client]
   │
   ├─► (HTTP GET /) ───────────────────────► [PageController] ──► Trả về index.html
   │
   └─► (HTTP REST API: GET/POST/PUT/DELETE) ─► [StaffController]
                                             [ProjectController]
                                                    │
                                                    ▼
                                          [OrganizationService]
                                                    │
                                                    ▼
                                          [In-Memory RAM: List<Project>]
```

### Chi tiết các tầng:
1. **Presentation Layer (Tầng hiển thị):**
   * File `templates/index.html` + `static/js/api.js`.
   * Sử dụng JavaScript và thư viện Axios để gọi bất đồng bộ (Asynchronous AJAX) tới máy chủ mà không cần tải lại toàn bộ trang (No page reload).
2. **Controller Layer (Tầng điều hướng & tiếp nhận request):**
   * **`PageController` (`@Controller`):** Định tuyến trang web tĩnh ban đầu (`GET /` trả về view `index`).
   * **`StaffController` & `ProjectController` (`@RestController`):** Cung cấp các Endpoint chuẩn RESTful JSON (`/api/staffs`, `/api/projects`).
3. **Service Layer (Tầng nghiệp vụ):**
   * **`OrganizationService` (`@Service`):** Đóng vai trò là *Single Source of Truth* (nguồn dữ liệu duy nhất). Chứa toàn bộ logic thêm, sửa, xóa, tìm kiếm nhân viên theo dự án, tự tăng ID bằng `AtomicLong`.
4. **Data / Domain Layer (Tầng dữ liệu):**
   * Class `Staff` và `Project`: Chứa các thuộc tính (POJO - Plain Old Java Object).
   * Lưu trữ trực tiếp trên bộ nhớ RAM thông qua cấu trúc dữ liệu `ArrayList<Project>`.

---

## 2. ƯU ĐIỂM & NHƯỢC ĐIỂM CỦA CÁC GIẢI PHÁP TRONG DỰ ÁN

| Thành phần / Kỹ thuật | Ưu điểm | Nhược điểm |
| :--- | :--- | :--- |
| **Lưu trữ dữ liệu trong RAM (`In-Memory ArrayList`)** | • Khởi động cực nhanh, không cần cài đặt hay cấu hình cơ sở dữ liệu (MySQL, PostgreSQL).<br>• Dễ dàng viết test và debug nhanh logic.<br>• Không lo lỗi kết nối mạng database. | • **Mất sạch dữ liệu** khi restart hoặc tắt ứng dụng.<br>• Khó mở rộng khi lượng dữ liệu lớn (tràn RAM).<br>• Không hỗ trợ các thao tác truy vấn phức tạp (Transaction, Indexing, ACID). |
| **Tách biệt REST API (`@RestController`) & Giao diện** | • Frontend và Backend độc lập: Frontend có thể viết bằng HTML thuần, React, Vue hoặc Mobile App mà không cần đổi Backend.<br>• Giảm tải băng thông: Server chỉ gửi dữ liệu JSON nhỏ gọn thay vì render toàn bộ cây HTML. | • Phải xử lý logic hiển thị và binding dữ liệu ở cả 2 phía.<br>• Cần nắm vững kỹ thuật bất đồng bộ (`Promise`, `async/await`) ở client. |
| **Sử dụng Axios qua CDN** | • Không cần cài đặt `Node.js`, `npm` hay cấu hình Webpack/Vite phức tạp.<br>• Thư viện nhẹ, tích hợp vào file HTML bằng 1 dòng thẻ `<script>`. | • Phụ thuộc vào kết nối mạng internet để tải CDN (nếu mất mạng CDN sẽ không chạy).<br>• Không tận dụng được tính năng kiểm tra lỗi TypeScript lúc build. |

---

## 3. PHÂN TÍCH TOÀN BỘ THẺ HTML CÓ TRONG DỰ ÁN & CÁC THẺ TƯƠNG ĐƯƠNG

Danh sách toàn bộ các thẻ HTML được sử dụng trong file [index.html](file:///Users/p.thuyen/Documents/spring/spring/src/main/resources/templates/index.html):

### A. Nhóm thẻ cấu trúc trang & Meta
| Thẻ HTML | Mục đích trong dự án | Các thẻ / Cách tương đương |
| :--- | :--- | :--- |
| `<!DOCTYPE html>` | Khai báo chuẩn HTML5 giúp trình duyệt hiển thị đúng chế độ chuẩn. | Bắt buộc cho mọi trang HTML5. |
| `<html lang="vi">` | Thẻ gốc của tài liệu, thuộc tính `lang="vi"` giúp SEO và trình đọc màn hình biết ngôn ngữ là tiếng Việt. | `lang="en"` (nếu là tiếng Anh). |
| `<head>` | Chứa thông tin cấu hình (metadata), tiêu đề, CSS của trang. | Không thể thay thế trong cấu trúc HTML. |
| `<meta charset="UTF-8">` | Định dạng bộ mã ký tự hiển thị đúng tiếng Việt có dấu, không bị lỗi font. | Trước đây dùng `UTF-16`, `ISO-8859-1`. Hiện tại UTF-8 là chuẩn tối ưu. |
| `<meta name="viewport" ...>` | Giúp trang web hiển thị co giãn chuẩn trên cả điện thoại (Responsive). | Thuộc tính quan trọng cho Mobile Friendly. |
| `<title>` | Đặt tiêu đề hiển thị trên tab của trình duyệt web ("Quản lý nhân viên"). | Không có thẻ thay thế trực tiếp. |
| `<style>` | Nhúng trực tiếp mã CSS vào file HTML để tạo giao diện nhanh. | Thẻ `<link rel="stylesheet" href="style.css">` (tách CSS ra file riêng). |
| `<body>` | Chứa toàn bộ nội dung hiển thị cho người dùng nhìn thấy. | Không thể thay thế. |

### B. Nhóm thẻ bố cục ngữ nghĩa (Semantic HTML)
| Thẻ HTML | Mục đích trong dự án | Các thẻ / Cách tương đương |
| :--- | :--- | :--- |
| `<main>` | Đánh dấu vùng nội dung chính yếu của trang. | `<div>`, `<section>`, `<article>`. Dùng `<main>` chuẩn Semantic & SEO hơn. |
| `<h1>`, `<h2>` | Tiêu đề lớn cấp 1 và cấp 2 của trang ("Thêm nhân viên", "Danh sách nhân viên"). | `<h3>` đến `<h6>` cho các tiêu đề nhỏ hơn. |
| `<div>` | Khối chứa đa năng (Container) dùng để bọc nhóm các phần tử để căn chỉnh CSS layout. | `<section>`, `<article>`, `<header>`, `<footer>` (nếu có ý nghĩa ngữ nghĩa cụ thể). |
| `<span>` | Phần tử nội dòng (inline) dùng để chứa đoạn chữ ngắn ("Đang tải dữ liệu..."). | `<p>` (đoạn văn dạng khối), `<label>`, `<small>`. |

### C. Nhóm thẻ Form & Nhập liệu (Form Controls)
| Thẻ HTML | Mục đích trong dự án | Các thẻ / Cách tương đương |
| :--- | :--- | :--- |
| `<form>` | Gom các ô nhập liệu thành một biểu mẫu để bắt sự kiện `submit`. | Bọc bằng thẻ `<div>` thông thường rồi tự bắt sự kiện bấm nút. Tuy nhiên dùng `<form>` chuẩn hơn vì tự hỗ trợ validation (`required`, Enter để submit). |
| `<label for="...">` | Nhãn mô tả cho ô nhập liệu. Khi click vào nhãn thì ô input tương ứng tự focus. | Đặt `placeholder` trong input (nhược điểm: biến mất khi người dùng gõ chữ). |
| `<input type="text">` | Nhập tên nhân viên. | `<textarea>` (nếu cần nhập nội dung dài nhiều dòng). |
| `<input type="email">` | Nhập email, trình duyệt tự kiểm tra định dạng email hợp lệ (có chữ `@`). | `<input type="text">` + dùng regex trong JS để tự validate. |
| `<select>`, `<option>` | Tạo danh sách thả xuống (Dropdown) để người dùng chọn Dự án. | `<input list="...">` đi kèm thẻ `<datalist>`, hoặc Custom Dropdown bằng CSS/JS. |
| `<button type="submit">` | Nút kích hoạt gửi dữ liệu form ("Lưu"). | `<input type="submit" value="Lưu">`. Dùng `<button>` linh hoạt hơn vì có thể chèn icon bên trong. |
| `<button type="button">` | Nút thông thường kích hoạt hàm JS `onResetForm()` mà không làm reload form ("Hủy"). | Thẻ `<a>` hoặc `<span onclick="...">` có style dạng nút bấm. |

### D. Nhóm thẻ Bảng dữ liệu (Table)
| Thẻ HTML | Mục đích trong dự án | Các thẻ / Cách tương đương |
| :--- | :--- | :--- |
| `<table>` | Hiển thị danh sách nhân viên dạng bảng lưới 2 chiều (hàng & cột). | Dùng CSS Grid (`display: grid`) hoặc Flexbox với các thẻ `<div>`. |
| `<thead>`, `<tbody>` | Phân tách phần tiêu đề bảng và phần thân chứa dữ liệu. | Giúp trình duyệt và máy đọc màn hình phân biệt header và data rows. |
| `<tr>` | Định nghĩa một hàng (row) trong bảng. | - |
| `<th>` | Ô tiêu đề cột (in đậm và căn giữa mặc định). | Thẻ `<td>` có thêm CSS in đậm. |
| `<td>` | Ô chứa dữ liệu thông thường. | - |

---

## 4. BẢNG SO SÁNH TỔNG QUAN: JQUERY VS FETCH API VS AXIOS

| Tiêu chí | jQuery (`$.ajax`) | Fetch API | Axios |
| :--- | :--- | :--- | :--- |
| **Bản chất** | Một thư viện DOM & Utility đa năng khổng lồ (từ 2006). | API tiêu chuẩn được tích hợp sẵn trong trình duyệt (Native ES6). | Thư viện chuyên biệt chỉ dùng cho HTTP Request (Promise-based). |
| **Cài đặt** | Có (tải file hoặc nhúng CDN ~88KB). | **Không cần** (Trình duyệt có sẵn). | Có (nhúng CDN hoặc `npm i axios` ~13KB). |
| **Xử lý JSON trả về** | Tự động parse nếu có header application/json. | **Thủ công**: Phải gọi qua bước `.json()`. | **Tự động chuyển đổi** thành Object trong `res.data`. |
| **Bắt lỗi HTTP (404, 500)** | Nhảy vào `.fail()` / `error`. | **Không reject**: 404/500 vẫn vào `then`, phải check `res.ok`. | **Tự động reject**: Mã 4xx, 5xx nhảy ngay vào `catch`. |
| **Interceptors** | Có qua `ajaxSetup` (cồng kềnh). | Không có sẵn (phải tự viết wrapper). | **Rất mạnh**: `interceptors.request` & `response`. |
| **Hủy Request / Timeout** | Thuộc tính `timeout: 5000`. | Phải dùng `AbortController` phức tạp. | Cực đơn giản: `{ timeout: 5000 }` hoặc `signal`. |

---

## 5. BÓC TÁCH TOÀN BỘ CẤU TRÚC CHUYÊN SÂU CỦA FETCH API

### 5.1. Cú pháp cơ bản
```javascript
fetch(url, [options])
```
* **`url`** *(bắt buộc)*: Đường dẫn tới tài nguyên (string hoặc Request object).
* **`options`** *(tùy chọn)*: Một object chứa toàn bộ cấu hình request.

### 5.2. Cấu trúc đầy đủ của `options` (Fetch Configuration)
```javascript
const options = {
    method: 'GET', // 'GET', 'POST', 'PUT', 'DELETE', 'PATCH', 'HEAD'
    
    headers: {
        'Content-Type': 'application/json', // Định dạng dữ liệu gửi lên
        'Authorization': 'Bearer YOUR_TOKEN' // Token xác thực (JWT)
    },
    
    // Dữ liệu gửi kèm body (KHÔNG dùng cho GET và HEAD)
    // Phải tự chuyển đổi sang JSON chuỗi bằng JSON.stringify()
    body: JSON.stringify({ name: "Lan", projectId: 101 }),
    
    mode: 'cors', // 'cors' (mặc định), 'no-cors', 'same-origin'
    
    credentials: 'same-origin', // 'omit' (không gửi cookie), 'same-origin', 'include' (gửi cookie qua cross-origin)
    
    cache: 'default', // 'default', 'no-cache', 'reload', 'force-cache', 'only-if-cached'
    
    redirect: 'follow', // 'follow' (tự động theo redirect), 'error', 'manual'
    
    referrerPolicy: 'no-referrer-when-downgrade',
    
    signal: abortController.signal // Dùng để timeout hoặc hủy request
};
```

### 5.3. Cấu trúc của đối tượng `Response` trong Fetch
Khi `fetch()` thành công, nó trả về một `Response` object với các thuộc tính:
* **`response.ok`**: Kiểu `boolean`, bằng `true` nếu HTTP Status code từ `200` đến `299`.
* **`response.status`**: Mã trạng thái HTTP (ví dụ: `200`, `201`, `404`, `500`).
* **`response.statusText`**: Thông điệp trạng thái (ví dụ: `"OK"`, `"Not Found"`).
* **`response.headers`**: Danh sách header do server trả về.

#### Các hàm đọc dữ liệu từ `Response` (Chỉ được gọi 1 lần duy nhất):
* **`response.json()`**: Phân tích body thành JavaScript Object (JSON).
* **`response.text()`**: Đọc body dưới dạng chuỗi văn bản thuần (HTML, Text).
* **`response.blob()`**: Đọc dữ liệu nhị phân (Ảnh, File PDF, Video).
* **`response.formData()`**: Đọc dữ liệu multipart form.

### 5.4. Code mẫu CRUD hoàn chỉnh chuẩn Production bằng Fetch API
```javascript
// Thiết lập Timeout 5 giây bằng AbortController
const fetchWithTimeout = async (url, options = {}, timeout = 5000) => {
    const controller = new AbortController();
    const timer = setTimeout(() => controller.abort(), timeout);
    try {
        const response = await fetch(url, { ...options, signal: controller.signal });
        // QUAN TRỌNG NHẤT VỚI FETCH: Phải tự kiểm tra response.ok
        if (!response.ok) {
            const errorBody = await response.text();
            throw new Error(`HTTP Error ${response.status}: ${errorBody}`);
        }
        return await response.json();
    } catch (err) {
        if (err.name === 'AbortError') throw new Error("Request bị quá thời gian (Timeout)!");
        throw err;
    } finally {
        clearTimeout(timer);
    }
};

// 1. GET - Lấy danh sách
async function getAllStaffs() {
    return await fetchWithTimeout('/api/staffs');
}

// 2. POST - Thêm mới
async function createStaff(staffData) {
    return await fetchWithTimeout('/api/staffs', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(staffData)
    });
}

// 3. PUT - Cập nhật
async function updateStaff(id, staffData) {
    return await fetchWithTimeout(`/api/staffs/${id}`, {
        method: 'PUT',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(staffData)
    });
}

// 4. DELETE - Xóa
async function deleteStaff(id) {
    return await fetchWithTimeout(`/api/staffs/${id}`, {
        method: 'DELETE'
    });
}
```

---

## 6. BÓC TÁCH TOÀN BỘ CẤU TRÚC CHUYÊN SÂU CỦA AXIOS

### 6.1. Các dạng cú pháp gọi Axios
```javascript
// Dạng 1: Truyền config tổng quát
axios(config);

// Dạng 2: Gọi phương thức tiện ích (Shorthand)
axios.get(url, [config])
axios.post(url, [data], [config])
axios.put(url, [data], [config])
axios.delete(url, [config])
axios.patch(url, [data], [config])

// Dạng 3: Tạo Instance dùng chung (Khuyên dùng nhất)
const API = axios.create({ baseURL: '/api' });
```

### 6.2. Cấu trúc đầy đủ của `Config` trong Axios
```javascript
const axiosConfig = {
    url: '/staffs',
    method: 'post', // get, post, put, delete, v.v.
    baseURL: 'http://localhost:8080/api', // Tiền tố URL
    
    // Header gửi lên
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + token
    },
    
    // URL Query Params: tự nối thành ?projectId=101&status=active
    params: {
        projectId: 101,
        status: 'active'
    },
    
    // Body data: Object truyền thẳng, Axios TỰ ĐỘNG JSON.stringify
    data: {
        name: 'Nguyen Van A',
        email: 'a@gmail.com',
        projectId: 101
    },
    
    // Thời gian tối đa chờ phản hồi (miliseconds). Quá thời gian sẽ bị hủy
    timeout: 5000,
    
    // Tự động gửi kèm Cookie khi gọi Cross-Origin
    withCredentials: true,
    
    // Kiểu dữ liệu mong đợi nhận về: 'json' (mặc định), 'text', 'blob', 'arraybuffer'
    responseType: 'json',
    
    // Xác định mã status nào được coi là thành công (mặc định 2xx)
    validateStatus: function (status) {
        return status >= 200 && status < 300;
    }
};
```

### 6.3. Cấu trúc của đối tượng `Response` trong Axios
Khi gọi thành công, Axios trả về object `response` gồm 6 trường:
```javascript
{
    data: {},          // Dữ liệu thật từ server trả về (ĐÃ ĐƯỢC TỰ ĐỘNG PARSE JSON)
    status: 200,       // Mã HTTP status code
    statusText: 'OK',  // Tên trạng thái HTTP
    headers: {},       // Danh sách Header từ server phản hồi
    config: {},        // Toàn bộ config cấu hình ban đầu của request này
    request: {}        // XMLHttpRequest object của trình duyệt
}
```

### 6.4. Cấu trúc bóc tách lỗi (`Catching Error`) trong Axios
Khi server trả về lỗi `4xx`, `5xx` hoặc mất mạng, Axios nhảy thẳng vào khối `catch(error)`:
```javascript
try {
    const res = await API.get('/staffs/999');
} catch (error) {
    if (error.response) {
        // Server ĐÃ PHẢN HỒI nhưng trả về mã lỗi ngoài dải 2xx (ví dụ 404, 500)
        console.error("Mã lỗi:", error.response.status);
        console.error("Dữ liệu lỗi từ backend:", error.response.data);
    } else if (error.request) {
        // Đã gửi request nhưng KHÔNG nhận được phản hồi (mất mạng hoặc server bị tắt)
        console.error("Không kết nối được server / Mất mạng:", error.request);
    } else {
        // Lỗi xảy ra khi thiết lập request trước khi gửi
        console.error("Lỗi cấu hình:", error.message);
    }
}
```

### 6.5. Tính năng độc quyền cực mạnh: Interceptors (Bộ đánh chặn)
Dùng để tự động gắn Token và bắt lỗi tập trung (Toàn bộ dự án chuyên nghiệp đều dùng):

```javascript
// 1. Tạo instance
const API = axios.create({
    baseURL: '/api',
    timeout: 8000
});

// 2. REQUEST INTERCEPTOR: Chạy TRƯỚC KHI request được gửi đi
API.interceptors.request.use(
    (config) => {
        const token = localStorage.getItem("access_token");
        if (token) {
            config.headers.Authorization = `Bearer ${token}`; // Tự gắn Token
        }
        console.log(`[HTTP Request] Đang gửi tới: ${config.url}`);
        return config;
    },
    (error) => Promise.reject(error)
);

// 3. RESPONSE INTERCEPTOR: Chạy NGAY KHI nhận được phản hồi
API.interceptors.response.use(
    (response) => {
        // Trả về trực tiếp data giúp code bên ngoài không cần gọi .data
        return response.data;
    },
    (error) => {
        if (error.response && error.response.status === 401) {
            alert("Phiên đăng nhập hết hạn! Vui lòng đăng nhập lại.");
            window.location.href = "/login";
        }
        return Promise.reject(error);
    }
);
```

---

## 7. BẢNG ĐỐI CHIẾU 1-1 CÁC THUỘC TÍNH GIỮA FETCH API VÀ AXIOS

| Nhiệm vụ / Tính năng | Fetch API (Native) | Axios (Library) |
| :--- | :--- | :--- |
| **Đường dẫn & method** | `fetch('/api/staffs', { method: 'POST' })` | `axios.post('/api/staffs')` |
| **Gửi Body dữ liệu** | `body: JSON.stringify(data)` | `data: data` *(Tự stringify)* |
| **Query Params (`?a=1`)** | Phải tự ghép chuỗi vào URL hoặc dùng `URLSearchParams` | `params: { a: 1 }` *(Tự ghép)* |
| **Truy cập dữ liệu nhận về** | `const json = await response.json();` | `const data = response.data;` |
| **Header** | `headers: { 'Content-Type': 'application/json' }` | `headers: { ... }` *(Mặc định đã là json)* |
| **Cấu hình Timeout** | Phức tạp (Cần `AbortController` + `setTimeout`) | Rất ngắn gọn: `timeout: 5000` |
| **Gửi kèm Cookie** | `credentials: 'include'` | `withCredentials: true` |
| **Kiểm tra thành công** | Phải tự viết `if (!response.ok) throw ...` | Tự động nhảy `catch` nếu mã lỗi `>= 300` |
| **Đọc dữ liệu Binary (File/Ảnh)** | `await response.blob()` | `responseType: 'blob'` |
| **Hủy Request giữa chừng** | `const c = new AbortController(); signal: c.signal; c.abort();` | `signal: controller.signal` (hoặc `CancelToken`) |

---

## 8. CÚ PHÁP THỰC CHIẾN (CHEAT SHEET) CHO FRONTEND & BACKEND

### A. Backend Spring Boot Annotations
* **`@SpringBootApplication`**: Chú thích khởi chạy ứng dụng (kết hợp `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan`).
* **`@RestController`**: Đánh dấu class phục vụ REST API, tự động gắn `@ResponseBody` cho mọi hàm (trả về JSON).
* **`@Controller`**: Trả về file giao diện template (HTML Thymeleaf).
* **`@RequestMapping("/api")`**: Tiền tố đường dẫn chung cho toàn bộ các hàm trong class.
* **`@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`**: Ánh xạ các method HTTP tương ứng cho CRUD.
* **`@PathVariable`**: Lấy tham số động trên URL (ví dụ: `@GetMapping("/{id}")`).
* **`@RequestBody`**: Chuyển đổi dữ liệu JSON từ body request thành đối tượng Java.
* **`ResponseEntity<?>`**: Đóng gói dữ liệu trả về kèm HTTP Status Code (`HttpStatus.OK`, `HttpStatus.CREATED`, `HttpStatus.NOT_FOUND`).

### B. Frontend JavaScript (ES6+)
* **`axios.create({ baseURL: '/api' })`**: Tạo instance dùng chung.
* **`async / await`**: Cú pháp xử lý bất đồng bộ tuần tự, sạch sẽ hơn `.then().catch()`.
* **`new FormData(formElement)`**: Gom nhanh toàn bộ dữ liệu trong form.
* **`array.map()` & `array.filter()`**: Duyệt và lọc dữ liệu danh sách để render HTML table.

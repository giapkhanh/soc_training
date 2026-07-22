## I. Các tầng mạng

Có 2 mô hình mạng chính: OSI và mô hình mạng TCP/IP

* Mô hình OSI là mô hình được ISO chuẩn hóa gồm 7 tầng. Trên thực tế các hệ thống mạng gần như khó mà tách biệt các tầng riêng lẻ như OSI, nhưng vì nó được phân định quá rõ ràng nên được đưa vào thi khá nhiều

  <img width="177" height="329" alt="image" src="https://github.com/user-attachments/assets/fefb233f-a73e-4786-8652-f7c19dd3d0c2" />

* Mô hình TCP/IP là mô hình thực tế gắn với Internet hơn. Các tầng có chức năng gần nhau được gom lại để tối ưu hóa việc xử lý của hệ điều hành và phần cứng.

  <img width="190" height="302" alt="image" src="https://github.com/user-attachments/assets/847ea266-41e0-4a97-93cf-dd54965a5cc9" />

### 1. Tầng ứng dụng (Application Layer)

**Kiến trúc mạng**: Có 2 mô hình chính là Client-Server và P2P (Peer-to-Peer)

* __Client-Server Architecture:__

  <img width="576" height="347" alt="image" src="https://github.com/user-attachments/assets/b4144297-5a1a-4dcc-994a-2e5c565e2b04" />

 
    * Server là máy chủ luôn bật, có IP cố định.
    * Client không giao tiếp trực tiếp với nhau mà phải thông qua Server.
    * Ví dụ: Web (HTTP), Email.
 
> [!NOTE] 
> **Kiến thức mở rộng nên biết**: Mô hình CDN
> * Mô tả bằng hình vẽ ASCII:
>  ```
>                                    ┌────────────────────┐
>                                    │  User / Browser    │
>                                    │  https://abc.com   │
>                                    └─────────┬──────────┘
>                                              │
>                           (1) User nhập domain abc.com
>                                              │
>                                              ▼
>                               ┌────────────────────────┐
>                               │      Local DNS         │
>                               │ (DNS của ISP/Wifi)     │
>                               └─────────┬──────────────┘
>                                         │
>                  Nếu chưa biết IP -> đi hỏi hệ thống DNS toàn cầu
>                                         │
>                                         ▼
>                       ┌────────────────────────────────┐
>                       │        Authoritative DNS       │
>                       │        của CDN Provider        │
>                       │  (Cloudflare / Akamai / etc)   │
>                       └──────────────┬─────────────────┘
>                                      │
>                  CDN bắt đầu chọn Edge Server phù hợp
>                                      │
>         ┌────────────────────────────┼────────────────────────────┐
>         │                            │                            │
>         ▼                            ▼                            ▼
> 
>  ┌──────────────┐          ┌──────────────┐            ┌──────────────┐
>  │ Edge VN-HN   │          │ Edge SG      │            │ Edge JP      │
>  │ ping: 12ms   │          │ ping: 45ms   │            │ ping: 80ms   │
>  │ tải thấp     │          │ tải cao      │            │ tải thấp     │
>  └──────┬───────┘          └──────┬───────┘            └──────┬───────┘
>         │                          │                           │
>         └──────────────┬───────────┴──────────────┬────────────┘
>                        │
>       CDN dùng nhiều yếu tố để chọn node tốt nhất:
>                        │
>                        │  • khoảng cách địa lý
>                        │  • latency/ping
>                        │  • tải server hiện tại
>                        │  • routing mạng
>                        │  • tình trạng node
>                        │
>                        ▼
>              Chọn: Edge VN-HN (12ms)
>                        │
>                        ▼
>           DNS trả IP của Edge VN-HN cho browser
>                        │
>                        ▼
> ───────────────────────────────────────────────────────────────
> 
>                  Browser bắt đầu kết nối thật
>                        │
>                        ▼
>               ┌────────────────────┐
>               │  CDN Edge VN-HN    │
>               │  Cache Server      │
>               └─────────┬──────────┘
>                         │
>               Kiểm tra cache nội bộ
>                         │
>          ┌──────────────┴──────────────┐
>          │                             │
>          ▼                             ▼
> 
>    [CACHE HIT]                  [CACHE MISS]
>    Có sẵn dữ liệu               Chưa có dữ liệu
>          │                             │
>          │                             ▼
>          │                  ┌────────────────────┐
>          │                  │   Origin Server    │
>          │                  │ App + Database     │
>          │                  └─────────┬──────────┘
>          │                            │
>          │<──────── dữ liệu ──────────┘
>          │
>          ▼
> CDN lưu cache vào Edge Server
>          │
>          ▼
> Trả dữ liệu về Browser 🚀
> 
> ```
>
> * Mô tả bằng hình minh họa trực quan:
>
> <img width="952" height="685" alt="image" src="https://github.com/user-attachments/assets/eaea3ca2-f0ba-485e-922d-798207177d8d" />
>
> ---
>
> <details>
> <summary>Hình minh họa nhưng có thêm phần proxy</summary> 
> <img width="897" height="691" alt="image" src="https://github.com/user-attachments/assets/a344caa7-c87c-4d92-9d81-5088f291fdfe" />
> </details>
>
> ---


* __Peer-to-Peer (P2P) Architecture:__

  <img width="1676" height="788" alt="image" src="https://github.com/user-attachments/assets/143a2ed6-5ba1-4a94-bac5-e92c96941238" />


    * Các máy tham gia (peers) có vai trò bình đẳng, vừa là client vừa là server.
    * Tính mở rộng cao (Self-scalability) nhưng quản lý phức tạp.

**Socket**:
* Là "Giao diện" (Interface) nằm ngay giữa Tầng ứng dụng (Application Layer) và Tầng giao vận (Transport Layer).
* Để tầng giao vận chuyển dữ liệu đến đúng Socket, mỗi Socket phải được định danh bằng một bộ gồm: [Địa chỉ IP + Số hiệu cổng (Port) + Giao thức].

* Phân loại Socket theo dịch vụ tầng giao vận (Transport Layer)

  Có 2 loại socket cốt lõi:
  
  * Stream Socket (SOCK_STREAM) – Gắn liền với TCP:
    * Cung cấp luồng truyền dữ liệu đáng tin cậy, hai chiều, có kết nối.
    * Đảm bảo dữ liệu không mất, đúng thứ tự.
    * Giao thức tầng ứng dụng sử dụng: HTTP, SMTP, FTP.

> [!NOTE]
> 
> Sơ đồ mô hình kết nối TCP:
>
> <img width="621" height="304" alt="image" src="https://github.com/user-attachments/assets/337261f2-41f7-4091-a6c3-71ef34d160df" />
> 
> <img width="3400" height="2926" alt="image" src="https://github.com/user-attachments/assets/424c42b8-de4d-4715-8ee2-90bf7f801b44" />


  * Datagram Socket (SOCK_DGRAM) – Gắn liền với UDP:
    * Cung cấp luồng truyền gói tin độc lập (datagram), không kết nối, không tin cậy.
    * Ưu tiên tốc độ, chấp nhận mất gói.
    * Giao thức tầng ứng dụng sử dụng: DNS, DHCP, VoIP.

> [!NOTE]
> 
> Sơ đồ mô hình UDP:
>
> <img width="561" height="519" alt="image" src="https://github.com/user-attachments/assets/9a86d056-9f45-4f3c-9d1d-5e96dd67855e" />

**Các giao thức cốt lõi**:

**HTTP**:
  * **Lý thuyết:**
    * HTTP chạy trên nền TCP socket, ở cổng 80. 
    * Là giao thức Stateless, nghĩa là nó không lưu giữ kí ức về các trạng thái request trước của bạn.
    * Do là một Stateless protocol, nên để tránh việc bạn gửi request giây trước rồi giây sau server sẽ quên luôn bạn là ai, chúng ta thường cần cookie và session đi kèm với HTTP (Máy server sẽ lưu ID, giải mã cookie và session được lưu ở client để đối chiếu danh tính của bạn).
  * **HTTP Message Format (Cấu trúc gói tin HTTP):**
    * __Request gửi đi__ (từ client):
      ```http
      GET /index.html HTTP/1.1         <-- Request Line (Method + URL + HTTP Version)
      Host: abc.com                    <-- Headers (Các thông tin cấu hình bổ sung)
      User-Agent: Mozilla/5.0
                                       <-- Dòng trống (Bắt buộc để ngăn cách)
      [Dữ liệu gửi lên nếu dùng POST]  <-- Body (Chứa dữ liệu form, file, JSON...)
      ```

      4 Methods kinh điển:

      * `GET`: Xin dữ liệu từ Server (Không có Body).
      * `POST`: Gửi dữ liệu mới lên Server để xử lý/tạo mới (Có dữ liệu ở Body).
      * `PUT`: Ghi đè/Cập nhật dữ liệu cũ.
      * `DELETE`: Xóa dữ liệu trên Server.

    * __Response trả về__ (từ server):
    ```http
    HTTP/1.1 200 OK                  <-- Status Line (Version + Status Code)
    Content-Type: text/html          <-- Headers (Server trả về thông tin của file)
    Content-Length: 1024
                                     <-- Dòng trống
    <html><body>Hi!</body></html>    <-- Body (Mã nguồn HTML, ảnh, dữ liệu JSON...)
    ```

    Các trạng thái (Status code) trả về:

    * `2xx` (Ví dụ 200 OK): Thành công.
    * `3xx` (Ví dụ 301 Moved Permanently, 304 Not Modified): Chuyển hướng / Dùng lại Cache đi.
    * `4xx` (Ví dụ 404 Not Found, 403 Forbidden): Lỗi tại Client (gõ sai URL, hoặc không có quyền vào).
    * `5xx` (Ví dụ 500 Internal Server Error, 502 Bad Gateway): Lỗi tại Server (Sập nguồn, crash code).
   
  * **Cơ chế kết nối**: Có 2 loại là Non-Persistent và Persistent
    * Non-Persistent HTTP (HTTP/1.0 - Cổ điển):
        * Để dễ trình bày ta sẽ quy ước một RTT (Round Trip Time) là thời gian từ lúc 1 client-request đi đến server tới lúc client nhận được phản hồi.
        * Khi dùng TCP kết nối ta biết nó sẽ tốn 1 RTT nhờ hiểu được cơ chế 3-way handshake
        * Sau đó lại tốn thêm 1 RTT nữa để gửi yêu cầu nhận nội dung, và đợi server gửi lại nội dung
        * Vậy tổng tất cả là 2 RTT cho mỗi lần gửi nhận. Điều đáng nói ở đây với Non-Persistent HTTP là **TCP connection sẽ bị đóng kết nối sau khi server gửi xong nội dung**.
     <img width="504" height="446" alt="image" src="https://github.com/user-attachments/assets/676bf166-6743-4b74-a68a-81c10e7e199f" />

  > [!NOTE]
  > Nhắc đến 3-way handshake có thể sẽ gây ra thắc mắc ở đây, rằng nếu dùng TCP connection như vậy thì phải tốn 1.5 RTT mới đúng. Nhưng thực ra gói tin `ACK` cuối cùng được client gửi cùng lượt với request yêu cầu nhận thông tin rồi. Vậy nên đoạn khởi tạo TCP connection đầu tiên ta vẫn tạm tính là 1 RTT.

  * Persistent HTTP (Từ HTTP/1.1 trở đi - Hiện đại):
    * Cũng vẫn là tốn 2 RTT cho việc gửi request, nhận nội dung với lần đầu tiên
    * Điểm khác biệt là, **TCP connection** không hề bị đóng sau mỗi lần server gửi nội dung. Nhờ vậy trang web tải nhanh hơn rõ rệt.

* **HTTPS (HTTP Secure / HTTP over TLS/SSL)**
  * **Lý thuyết**: 
    * Mặc định chạy trên cổng 443.
    * Bản chất HTTPS vẫn là HTTP nhưng được bọc thêm 1 lớp mã hóa TLS/SSL trước khi gửi qua mạng.

    (Mặc dù cổng mặc định của HTTP là 80, của HTTPS là 443. Nhưng trên thực tế admin có thể cấu hình chúng chạy trên bất kì cổng nào khác như 8000, 8080,...)

  * **Quy trình TLS Handshake**:
  <img width="668" height="643" alt="image" src="https://github.com/user-attachments/assets/d662ed98-f0fa-4bff-8465-015193304d17" />

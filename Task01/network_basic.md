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


* **DHCP (Dynamic Host Configuration Protocol)**

  * **Lý thuyết:**
    * **DHCP** chạy trên nền **UDP socket**, sử dụng **cổng 67 (DHCP Server)** và **cổng 68 (DHCP Client)**.
    * Là giao thức tự động cấp phát cấu hình mạng cho các thiết bị (Client) khi tham gia vào mạng.
    * Các thông số DHCP cấp bao gồm: **IP Address**, **Subnet Mask**, **Default Gateway** (Router), và **DNS Server IP**.
    * Giải quyết vấn đề đụng độ IP (IP Conflict) và đỡ công quản trị viên phải đặt IP thủ công cho hàng trăm máy tính.
  * **DHCP Message Format**:
      * Phản hồi từ client và server được dùng chung 1 bộ khung dưới đây. Tùy vào các ô thông tin mà sẽ phân định nó là từ server gửi hay client gửi.

        <img width="269" height="247" alt="image" src="https://github.com/user-attachments/assets/32dcdd26-5966-4200-9606-298967a59469" />

  * **Quy trình cấp IP (DORA Process):**
    * **Discover (`DHCPDISCOVER`):** Client vừa vào mạng chưa có IP, gửi tin **Broadcast** (`255.255.255.255`) ra toàn mạng: *"Có DHCP Server nào ở đây không? Cấp IP cho tôi với!"*
    * **Offer (`DHCPOFFER`):** Server nhận được, chọn một IP còn trống và gửi lại chào mời: *"Tôi có IP `192.168.1.50` này, bạn lấy không?"*
    * **Request (`DHCPREQUEST`):** Client phản hồi **Broadcast**: *"Tôi đồng ý lấy IP `192.168.1.50` của Server này nhé!"* (Gửi Broadcast để các DHCP Server khác nếu có sẽ biết mà thu hồi IP chào mời của họ).
    * **ACK (`DHCPACK`):** Server chốt: *"Đã chốt! Bạn được dùng IP `192.168.1.50` kèm theo Gateway và DNS Server này."*
  
    <img width="781" height="399" alt="image" src="https://github.com/user-attachments/assets/9fc967fa-4a1f-45f9-b3e5-90e6a7e07336" />

> [!NOTE]
> **Cơ chế Thuê IP (Lease Time):**
> * IP cấp qua DHCP **không phải là vĩnh viễn**, mà có hạn dùng gọi là **Lease Time** (ví dụ: 24 giờ).
> * **Cơ chế xin gia hạn (Renew):**
>   * Khi dùng được **50% Lease Time** (T1): Client gửi gói `DHCPREQUEST` trực tiếp (Unicast) tới Server để xin gia hạn.
>   * Nếu Server không trả lời, đến **87.5% Lease Time** (T2): Client sẽ phát Broadcast lại để xin gia hạn từ bất kỳ Server nào khác.

<br>

> [!IMPORTANT]
> * **DHCP Relay Agent:** Nếu DHCP Server nằm ở một Router/Subnet khác, Router ở mạng địa phương phải bật tính năng *Relay Agent* để chuyển tiếp gói tin Broadcast của Client qua mạng khác (chuyển Broadcast thành Unicast).
> * **Rủi ro Rogue DHCP:** Kẻ gian có thể cắm một DHCP Server giả vào mạng để cấp IP/Gateway giả nhằm đánh cắp dữ liệu (Man-in-the-middle). Giải pháp mạng là bật **DHCP Snooping** trên Switch.

* **DNS (Domain Name System)**
    * **Lý thuyết**:
      * **DNS** chạy trên cổng **53**. Mặc định dùng **UDP** (để truy vấn nhanh, nhẹ), nhưng sẽ dùng **TCP** khi dữ liệu trả về lớn hơn 512 bytes hoặc khi đồng bộ dữ liệu giữa các DNS Server (Zone Transfer).
      * Là "danh bạ của Internet", có chức năng chuyển đổi tên miền dễ nhớ đối với con người (`abc.com`) thành địa chỉ IP mà máy tính hiểu (`142.250.198.46`).

  * **DNS Hierarchy (Cây phân cấp DNS)**
    * **Root Domain (`.`):** Cấp cao nhất (do 13 cụm Root Server toàn cầu quản lý).
    * **Top-Level Domain - TLD (`.com`, `.net`, `.vn`):** Do các tổ chức quản lý tên miền điều hành (ví dụ: VNNIC quản lý `.vn`).
    * **Second-Level Domain (`abc.com`):** Tên miền do người dùng/doanh nghiệp mua.
    * **Subdomain (`mail.abc.com`, `dev.abc.com`):** Tên miền phụ do chủ sở hữu tự phân chia.

      <img width="1200" height="600" alt="image" src="https://github.com/user-attachments/assets/7328d113-a9f7-4558-a92c-de721289bc3b" />

  * **DNS Resolution Flow (Quy trình phân giải tên miền)**:
    * **Bước 1 (Client $\rightarrow$ Recursive Resolver):**
        * Người dùng gõ `example.com` trên trình duyệt. 
        * Máy tính gửi yêu cầu truy vấn đệ quy (**Recursive Query**): *"Cho tôi xin IP của `example.com`"*.
    * **Bước 2 và 3 (Resolver $\leftrightarrow$ Root Nameserver)**:
        * Resolver đi hỏi Root Server: *"Ông có biết IP của `example.com` không?"*
        * Root Server trả lời: *"Tôi không biết IP cụ thể, nhưng đây là địa chỉ IP của **TLD Server quản lý đuôi `.com`**, ông sang đó mà hỏi!"*
    * Bước 4 & 5 (Resolver $\leftrightarrow$ TLD Nameserver):
      * Resolver tiếp tục sang hỏi TLD Server (`.com`): *"Ông có biết IP của `example.com` không?"*
      * TLD Server trả lời: *"Tôi cũng không giữ IP trực tiếp, nhưng đây là địa chỉ của **Authoritative Server quản lý miền `example.com`**, ông sang đó mà hỏi!"*
    * **Bước 6 & 7 (Resolver $\leftrightarrow$ Authoritative Nameserver):**
      * Resolver gõ cửa Authoritative Server: *"Ông là nơi giữ bản ghi của `example.com`, cho tôi xin IP chính xác!"*
      * Authoritative Server tìm trong cơ sở dữ liệu (Bản ghi A) và trả về địa chỉ IP chính xác: **`93.184.216.34`**
    * **Bước 8 (Resolver $\rightarrow$ Client):**
      * Resolver nhận được IP, **lưu IP này vào bộ nhớ tạm (Cache)** để dùng cho lần sau, đồng thời gửi phản hồi về cho Client: *"IP của `example.com` là `93.184.216.34` nhé!"*.
      * Lúc này Client mới bắt đầu mở kết nối TCP để gửi request HTTP/HTTPS tới IP `93.184.216.34`..

    <img width="702" height="437" alt="image" src="https://github.com/user-attachments/assets/5c16fd73-ddf4-4ed7-94e8-6852d261d59b" />

    Còn 1 cái hình nữa, đại ý nội dung cũng như trên nhưng chi tiết hơn

    <img width="876" height="754" alt="image" src="https://github.com/user-attachments/assets/f2caa880-6e78-4c8b-8d11-7b68bdff844c" />

> [!NOTE]
> * **DNS Caching (Bộ nhớ tạm):** Để giảm RTT và tránh quá tải hệ thống, kết quả DNS sẽ được lưu cache ở nhiều nơi: **Browser Cache $\rightarrow$ OS Cache $\rightarrow$ Router Cache $\rightarrow$ ISP Local DNS Cache**. Mỗi bản ghi có chỉ số **TTL (Time To Live)** quy định thời gian lưu cache.
> * **Bảo mật hiện đại (DoH & DoT):** Mặc định DNS truyền tin dạng Plaintext (không mã hóa). Hiện nay các trình duyệt sử dụng **DoH (DNS over HTTPS)** hoặc **DoT (DNS over TLS)** để mã hóa truy vấn DNS, tránh bị nhà mạng hoặc kẻ gian theo dõi xem bạn đang truy cập trang web nào.

> [!IMPORTANT]
> **6 Record DNS kinh điển cần nhớ:**
> 
> * **A Record:** Ánh xạ Tên miền $\rightarrow$ Địa chỉ **IPv4**.
> * **AAAA Record:** Ánh xạ Tên miền $\rightarrow$ Địa chỉ **IPv6**.
> * **CNAME (Canonical Name):** Tạo tên giả / trỏ tên miền này sang tên miền khác (Ví dụ: `www.abc.com` $\rightarrow$ `abc.com`).
> * **MX Record (Mail Exchange):** Trỏ tên miền đến Mail Server nhận email.
> * **NS Record (Name Server):** Chỉ định máy chủ DNS nào đang nắm quyền quản lý bản ghi của tên miền này.
> * **TXT Record:** Chứa thông tin văn bản (thường dùng để xác thực tên miền, cài đặt cấu hình chống spam email như SPF, DKIM).

Đúng vậy! Địa chỉ IP và các dải lớp IP chính là **nền móng hạ tầng (Tầng Mạng - Network Layer)** để các dịch vụ như DHCP, DNS hay HTTP có thể vận hành được.

Dưới đây là bản tổng hợp kiến thức về **Địa chỉ IPv4, Các lớp IP, Public/Private IP và CIDR** được thiết kế chuẩn theo phong cách ngắn gọn, dễ hiểu và dễ tra cứu của bạn.

---

## II. ĐỊA CHỈ IP (IPv4) & CÁC DẢI IP

#### Lý thuyết cơ bản:
* Địa chỉ IPv4 gồm **32 bits** (4 bytes), được chia thành **4 octet** (mỗi octet 8 bits, cách nhau bởi dấu chấm).
  * *Ví dụ:* `192.168.1.1` $\rightarrow$ Nhị phân: `11000000.16810000.00000001.00000001`
* Mỗi địa chỉ IP luôn gồm 2 phần:
  1. **Network ID (Địa chỉ mạng):** Giống như tên đường/xóm.
  2. **Host ID (Địa chỉ máy):** Giống như số nhà cụ thể trong xóm đó.

---

##### 1. Cấu trúc 5 lớp IP cổ điển (Classful Addressing)

Thời kỳ đầu, Internet chia IP thành 5 lớp dựa vào giá trị của **Octet đầu tiên**:

| Lớp (Class) | Dải Octet đầu | Subnet Mask mặc định | Tỉ lệ Network / Host | Mục đích sử dụng |
| :--- | :--- | :--- | :--- | :--- |
| **Class A** | `1` $\rightarrow$ `126` | `255.0.0.0` (`/8`) | 8 bits Net / 24 bits Host | Mạng siêu lớn (Tập đoàn toàn cầu, Chính phủ) |
| **Class B** | `128` $\rightarrow$ `191` | `255.255.0.0` (`/16`) | 16 bits Net / 16 bits Host | Mạng vừa (Trường đại học, Công ty lớn) |
| **Class C** | `192` $\rightarrow$ `223` | `255.255.255.0` (`/24`)| 24 bits Net / 8 bits Host | Mạng nhỏ (LAN gia đình, văn phòng nhỏ) |
| **Class D** | `224` $\rightarrow$ `239` | Không có | Dùng cho **Multicast** | Phát tin cùng lúc cho 1 nhóm máy (Streaming, Routing) |
| **Class E** | `240` $\rightarrow$ `255` | Không có | Dùng cho **Nghiên cứu** | Dự phòng / Thử nghiệm của IETF |

* **Quy tắc tính số Host trong 1 mạng:**  
  $$\text{Số lượng máy tối đa} = 2^{(\text{Số bit Host})} - 2$$
  *(Trừ 2 vì: 1 địa chỉ làm **Network ID** và 1 địa chỉ làm **Broadcast ID**).*
  * *Ví dụ Class C (/24):* Có 8 bit Host $\rightarrow$ $2^8 - 2 = 254$ máy tính.

---

##### 2. Phân biệt Public IP vs Private IP (Cực kỳ quan trọng)

Do IPv4 bị cạn kiệt (chỉ có $2^{32} \approx 4.3$ tỷ IP), người ta chia IP thành 2 loại:

#### A. Private IP (IP nội bộ / IP riêng):
* Chỉ dùng trong mạng nội bộ (LAN gia đình, công ty, quán net).
* **Không thể định tuyến trực tiếp trên Internet.**
* Được miễn phí và có thể trùng nhau ở 2 mạng LAN khác nhau.
* **3 Dải Private IP chuẩn (RFC 1918):**
  * **Lớp A:** `10.0.0.0` $\rightarrow$ `10.255.255.255`
  * **Lớp B:** `172.16.0.0` $\rightarrow$ `172.31.255.255`
  * **Lớp C:** `192.168.0.0` $\rightarrow$ `192.168.255.255` *(Rất quen thuộc trên Modem Wi-Fi gia đình)*

###### B. Public IP (IP công cộng):
* Do các nhà mạng (ISP như Viettel, VNPT, FPT) cấp.
* **Duy nhất trên toàn thế giới**, dùng để đi ra Internet.
* Cần phải trả phí để sở hữu.

> **Cơ chế kết nối:** Kỹ thuật **NAT (Network Address Translation)** trên Router sẽ biến hàng trăm máy có *Private IP* trong nhà bạn thành **1 Public IP duy nhất** khi truy cập ra ngoài Internet.

---

##### 3. Chia mạng hiện đại: CIDR (Classless Inter-Domain Routing)

Chia lớp A, B, C cố định gây lãng phí IP (ví dụ Class A cấp cho 1 công ty sẽ thừa tới 16 triệu IP). Ngày nay người ta dùng **CIDR** (ký hiệu dấu gạch chéo `/` sau IP).

* **Ký hiệu:** `192.168.1.0/24`
  * `/24` nghĩa là **24 bits đầu tiên** là phần Mạng (Network), còn lại $32 - 24 = 8$ bits là phần Máy (Host).
* **Bảng tra cứu CIDR phổ biến:**
  * `/32`: Chỉ **1 IP duy nhất** (Thường chỉ 1 máy/Server).
  * `/30`: Có **2 IP dùng được** (Thường dùng cho đường nối giữa 2 Router).
  * `/24`: Có **254 IP dùng được** (Mạng LAN tiêu chuẩn - Subnet Mask `255.255.255.0`).
  * `/16`: Có **65,534 IP dùng được** (Subnet Mask `255.255.0.0`).

---

##### 4. Các địa chỉ IP ĐẶC BIỆT cần thuộc lòng

1. **`127.0.0.1` (Loopback / Localhost):** 
   * Trỏ về chính máy tính của bạn. Dùng cho Developer chạy test server (`http://localhost:3000`). Dải `127.0.0.0/8` đều bị bảo lưu cho Loopback.
2. **`0.0.0.0` (Unspecified Address):** 
   * Đại diện cho "Mọi IP" hoặc dùng khi máy tính chưa có IP (đang gửi gói tin DHCP Discover).
3. **`255.255.255.255` (Limited Broadcast):**
   * Gửi gói tin cho **tất cả** các máy trong cùng mạng LAN địa phương.
4. **`169.254.x.x` (APIPA - Automatic Private IP Addressing):**
   * **Dấu hiệu nhận biết sự cố!** Khi máy tính cài DHCP nhưng **không kết nối được tới DHCP Server**, Windows sẽ tự gắn 1 IP dạng `169.254.x.x` này. (Nếu thấy IP này tức là mạng đang bị lỗi DHCP!).

---

##### ℹ️ Mối liên hệ thực tế tổng hợp (Xâu chuỗi toàn bộ kiến thức):

Nối lại bức tranh toàn bộ kiến thức bạn đã tích lũy từ đầu đến giờ:

1. **Bật máy / Cắm dây:** Máy tính gửi tin Broadcast `255.255.255.255` từ IP `0.0.0.0` để tìm **DHCP Server**.
2. **DHCP:** Cấp cho máy bạn 1 địa chỉ **Private IP** (ví dụ `192.168.1.15/24`), kèm theo IP của **DNS Server** (ví dụ `8.8.8.8`). (Nếu DHCP lỗi $\rightarrow$ Máy nhận IP `169.254.x.x`).
3. **Gõ Web:** Bạn gõ `abc.com` $\rightarrow$ Máy gửi yêu cầu qua **DNS** $\rightarrow$ DNS trả về **Public IP** của Server web đó (ví dụ `142.250.198.46`).
4. **Tải Web:** Máy mở kết nối **HTTP/HTTPS** qua **TCP socket (Port 80/443)** đến IP `142.250.198.46` để lấy dữ liệu trang web về.

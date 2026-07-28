### Cách để biết có bao nhiêu trường trong `index`:

<!-- Để powershell cho lên màu đẹp thôi-->

```powershell
| eventcount summarize=false
| stats count by index
```

---

### Các dấu so sánh được sử dụng: `=`, `!=`, `<`, `<=`, `>`, `>=`
* **Ví dụ:**
  
  ```powershell
  index=windowslogs AccountName!=SYSTEM
  ```
---

### Các phép toán logic được sử dụng: `NOT`, `AND`, `OR`, `IN`.
* Trong đó `AND` là toán tử có độ ưu tiên thấp nhất, không viết gì giữa 2 điều kiện mà chỉ có SPACE thì được hiểu là `AND`.
* Độ ưu tiên cao nhất => thấp nhất:
    * **Lệnh `search` cơ bản**: Dấu `()`, `NOT`, `OR`, `AND`. (Không có hỗ trợ `XOR`)
    * Lệnh `eval` và `where`: `()`, `NOT`, `AND`, `OR`, `XOR`.
* **Ví dụ:**
  ```powershell
  index=windowslogs AccountName!=SYSTEM AND AccountName=James
  index=windowslogs AccountName!=SYSTEM AccountName=James
  ... | where ((A AND B) OR C) XOR D
  ```

---

### Wildcards and CIDR Search: Dùng dấu `*` để tìm từ phù hợp
* **Ví dụ:**
  ```powershell
  index=* SourceIp = "172.*"
  ```

---

### Dấu `" "` và dấu `()`:
  * Dùng `" "` để biểu thị một string
  * Dùng `()` để gom các biểu thức logic, đặt đúng mức độ ưu tiên.

---

### Filter khi search
* `fields`: Chỉ giữ lại các trường dữ liệu cần thiết.
    ```powershell
    index=windowslogs | fields host User SourceIp
    ```
* `dedup`: Lọc các sự kiện trùng lặp, tùy chỉnh số lượng giữ lại
    ```powershell
    index=windowslogs
    | fields EventID User Image Hostname SourceIp
    | dedup SourceIp
    ```

    ```powershell
    index=windowslogs
    | fields EventID User Image Hostname SourceIp
    | dedup 3 SourceIp
    ```
    
* `rename`:
    ```powershell
    index = windowslogs
    | fields EventID User Image Hostname SourceIp
    | rename User as Employee
    ```

    Command `rename` cũng hữu ích để xử lý JSON và XML.
    * Ví dụ một log JSON kiểu này `{"request": {"path": "/admin", "ip": "10.0.0.2"}}`. Splunk sẽ tạo 2 fields: `request.path` và `request.ip`
    * Nếu không muốn gõ lại cái prefix `request.` nhiều lần, có thể làm thế này:
        ```powershell
        index=jsondata
        | rename request.* as * // request.path -> path; request.ip -> ip
        ```

### Regex

Giai đoạn này tạm thời bỏ qua cái đống Regex đáng ghét, có thể tạm thời xem qua ở đây

https://help.splunk.com/en/splunk-enterprise/spl-search-reference/10.4/search-commands/regex

* **Ví dụ**: Dùng regex để giới hạn các field `Value` có đuôi `.exe`
  ```powershell
  index = windowslogs | regex Image = "\.exe$"
  ```

### Cấu trúc hóa kết quả tìm kiếm 

#### Lệnh `table`

* Lệnh `table` này giúp giới hạn những fields cần đọc thành bảng gọn gàng.

```powershell
index=windowslogs | table _time EventID Hostname SourceName
```

#### Các lệnh khác

Ngoài `table` còn một số lệnh khác cũng giúp cấu trúc hóa (structuring) kết quả:

* Lệnh `head`:

  Giới hạn trả về số kết quả mới nhất, tùy chỉnh số lượng.

  ```powershell
  index=windowslogs | head 2
  ```

* Lệnh `tail`:

  Giới hạn số kết quả cũ nhất, tùy chỉnh số lượng.

  ```powershell
  index=windowslogs | tail 20
  ```

* Lệnh `sort`:

  Sắp xếp kết quả trả về theo thứ tự dựa vào field được chỉ định

  ```powershell
  index=windowslogs | sort User
  ```

* Lệnh `reverse`:

  Đảo ngược thứ tự hiển thị được sắp xếp ban đầu.

  ```powershell
  index=windowslogs | reverse
  ```

#### Dùng `_time` kết hợp với `table` để sắp xếp kết quả theo timeline

Cách này thuận tiện để theo dõi theo thời gian, lại vừa có thể thêm bớt các fields tùy ý.

```powershell
index = windowslogs Hostname = Salena.Adam
| table _time Hostname EventID Category
| reverse
```

#### Subsearches

Kết hợp `join` với subsearch `[ ]` ta có thể cho ra bảng kết quả giống kiểu như join thêm vài cột vào 1 bảng trong SQL Server.

```powershell
index=windowslogs EventID=1
| join LogonId
    [ search index=windowslogs EventID=4624
    | rename TargetLogonId as LogonId
    | fields LogonId LogonType IpAddress]
| table _time Image User LogonType IpAddress
```

### Tra cứu EventID

* Nơi tra cứu EventID (Mã sự kiện Windows):

  https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx

* Tra cứu SID (Mã định danh tài khoản):

  https://learn.microsoft.com/en-us/windows/win32/secauthz/well-known-sids

* Tra cứu tiến trình process, PID, PPID (tóm lại là để nhận diện hành vi của cái `.exe` đó làm gì)

  https://lolbas-project.github.io/

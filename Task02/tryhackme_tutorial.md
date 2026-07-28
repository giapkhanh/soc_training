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

* **Ví dụ**: Dùng regex để giới hạn các fields có đuôi `.exe`
  ```powershell
  index = windowslogs | regex Image = "\.exe$"
  ```

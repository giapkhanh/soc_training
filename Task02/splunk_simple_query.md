# Tài liệu Hướng dẫn & Tra cứu Splunk Processing Language (SPL)

Tài liệu này tổng hợp đầy đủ các khái niệm cơ bản, tính năng cốt lõi và kiến trúc hệ thống của Splunk.

---

## I. Khái niệm Cốt lõi (Core Concepts)

### 1. Events (Sự kiện)

 **Mô tả:** Là tập hợp các giá trị đi kèm với một mốc thời gian (timestamp). Đây là đơn vị dữ liệu nhỏ nhất trong Splunk, có thể có một hoặc nhiều dòng (như văn bản, tệp cấu hình, stack trace, log web...).

* **Ví dụ về một Event:**
```text
173.26.34.223 - - [01/Mar/2021:12:05:27 -0700] "GET /trade/app?action=logout HTTP/1.1" 200 2953

```


* **Lưu ý (Transactions):** Bạn có thể nhóm các event liên quan đến nhau theo thời gian thành một **Transaction** (ví dụ: toàn bộ hành vi mua sắm của 1 khách hàng trong một phiên truy cập).

---

### 2. Metrics (Chỉ số)

 **Mô tả:** Điểm dữ liệu metric bao gồm timestamp và một hoặc nhiều phép đo (measurements), cùng các chiều dữ liệu (dimensions) bổ sung thông tin.

* **Ví dụ về điểm dữ liệu Metric:**
* **Timestamp:** `08-05-2020 16:26:42.025-0700`
* **Measurement:** `metric_name:os.cpu.user=42.12`, `metric_name:max.size.kb=345`
* **Dimensions:** `hq=us-west-1`, `group=queue`, `name=azd`


* **Lưu ý:** Metric và Event có thể tìm kiếm và liên kết (correlate) cùng nhau nhưng được lưu trữ ở các loại Index riêng biệt.

---

### 3. Host, Source, và Source Type

* **Host:** Tên thiết bị vật lý hoặc máy ảo nơi event sinh ra (dùng để tìm toàn bộ dữ liệu từ thiết bị đó).
* **Source:** Tên tệp, thư mục, luồng dữ liệu (data stream) hoặc đầu vào sinh ra event.
* **Source Type:** Phân loại định dạng của dữ liệu (ví dụ: log HTTP web server, Windows event log...).
* **Mối quan hệ:** Nhiều `source` khác nhau có thể chia sẻ chung một `sourcetype`.
* *Ví dụ:* Tệp `source=/var/log/messages` và cổng `source=UDP:514` đều có thể thuộc `sourcetype=linux_syslog`.



---

### 4. Fields (Trường dữ liệu)

 **Mô tả:** Các cặp tên-giá trị (name/value) có thể tìm kiếm, giúp phân biệt sự kiện này với sự kiện khác.

* **Cách trích xuất:** Splunk tự động trích xuất các trường khi ghi dữ liệu (index-time) hoặc khi tìm kiếm (search-time) dựa trên cấu hình.
* **Công cụ hỗ trợ:** Sử dụng **Field Extractor** để tự động tạo và kiểm tra biểu thức chính quy (RegEx) hoặc các ký tự phân cách (dấu cách, dấu phẩy...).

---

### 5. Tags (Thẻ)

**Mô tả:** Đối tượng tri thức (knowledge object) giúp gán tên gợi nhớ cho các cặp field/value.

* **Công dụng:** Nhóm các giá trị liên quan hoặc theo dõi các giá trị trừu tượng (như IP address, ID number) bằng tên dễ hiểu hơn.
* **Phạm vi gán:** Áp dụng cho bất kỳ tổ hợp trường/giá trị nào, bao gồm event types, hosts, sources, sourcetypes.

---

### 6. Index-Time vs Search-Time

| Giai đoạn | Quy trình xử lý |
| --- | --- |
| **Index-Time** | Đọc dữ liệu từ source $\rightarrow$ Phân loại sourcetype $\rightarrow$ Trích xuất timestamp $\rightarrow$ Tách event $\rightarrow$ Ghi dữ liệu vào đĩa (Index). |
| **Search-Time** | Khi có yêu cầu truy vấn $\rightarrow$ Đọc event đã lưu từ Index $\rightarrow$ Trích xuất các Field từ văn bản thô (raw text). |

---

### 7. Indexes (Chỉ mục)

**Mô tả:** Nơi lưu trữ các sự kiện trên đĩa cứng. Theo mặc định, dữ liệu được lưu tại index `main`. Bạn có thể tạo nhiều index riêng lẻ cho các nguồn đầu vào khác nhau để tối ưu hiệu năng.

---

## II. Tính năng Cốt lõi & Bổ sung (Core & Additional Features)

### 1. Tính năng Cốt lõi (Core Features)

* **Searches (Truy vấn):** Sử dụng ngôn ngữ SPL để lấy event, tính toán thống kê, phát hiện mẫu dữ liệu và dự báo xu hướng.
* **Reports (Báo cáo):** Là các tìm kiếm đã được lưu lại (saved searches). Có thể chạy ad-hoc, lập lịch định kỳ, hoặc dùng để hiển thị trên Dashboard.
* **Dashboards (Bảng điều khiển):** Gồm các panel chứa biểu đồ, hộp tìm kiếm, kết nối trực tiếp với các saved search hoặc tìm kiếm thời gian thực.
* **Alerts (Cảnh báo):** Tự động kích hoạt hành động (gửi Email, gửi Webhook...) khi kết quả tìm kiếm thỏa mãn điều kiện chỉ định.

---

### 2. Tính năng Bổ sung (Additional Features)

* **Datasets & Table Views:** Tạo và quản lý các tập dữ liệu như Lookups, Data Models. Table Views cho phép thao tác trực quan mà không cần viết quá nhiều SPL.
* **Data Model:** Tập hợp các dataset được tổ chức phân cấp. Có thể tăng tốc (acceleration) để cải thiện vượt bậc hiệu năng Dashboard.
* **Apps:** Gói cấu hình, dashboard và đối tượng tri thức phục vụ riêng cho các mục đích cụ thể (như Unix/Windows admin, Security...).
* **Distributed Search:** Mô hình tìm kiếm phân tán, mở rộng quy mô bằng cách tách biệt lớp quản lý tìm kiếm và lớp lưu trữ/index.

---

## III. Thành phần Hệ thống (System Components)

```
[Forwarder] ---> (Gửi dữ liệu thô) ---> [Indexer] <--- (Gửi truy vấn) --- [Search Head]

```

1. **Forwarders:** Thể hiện (instance) Splunk thu thập và chuyển tiếp dữ liệu về cho Indexer.
2. **Indexer:** Chịu trách nhiệm phân tích dữ liệu thô thành event, ghi vào Index và thực hiện tìm kiếm khi nhận truy vấn.
3. **Search Head:** Tiếp nhận câu truy vấn từ người dùng, điều phối truy vấn đến các Indexer (Search Peers), sau đó tổng hợp kết quả trả về.

---

## IV. Ngôn ngữ Tìm kiếm - Search Processing Language (SPL)

### 1. Cấu trúc câu lệnh căn bản

Một câu truy vấn Splunk gồm nhiều lệnh nối với nhau qua ký tự đường ống `|` (Pipe). Output của lệnh phía trước sẽ làm Input cho lệnh phía sau.

```splunk
search | command1 arguments1 | command2 arguments2 | ...

```

* **Mặc định:** Đầu chuỗi pipeline luôn ngầm định lệnh `search`. Giữa các từ khóa tìm kiếm ngầm hiểu là toán tử `AND`.
* **Ví dụ:**
```splunk
sourcetype=access_combined error | top 5 uri

```


*(Tìm tất cả event thuộc `sourcetype=access_combined` chứa từ "error", sau đó thống kê Top 5 giá trị `uri` xuất hiện nhiều nhất.)*

---

### 2. Bộ điều chỉnh thời gian (Time Modifiers)

Bạn có thể giới hạn khoảng thời gian tìm kiếm ngay trong truy vấn bằng `earliest` và `latest`:

* **Cú pháp:** `[+|-]<number><unit>@<snap_time_unit>`
* **Đơn vị thời gian (`unit`):** `s` (second), `m` (minute), `h` (hour), `d` (day), `w` (week), `m` (month).
* **Ký tự làm tròn (`@` - Snap to):** Làm tròn thời gian về mốc bắt đầu của đơn vị đó (ví dụ: `@h` làm tròn về đầu giờ, `@w0` là Chủ Nhật).
* **Ví dụ:**
```splunk
"error" earliest=-1d@d latest=@h

```


*(Tìm từ "error" từ 00:00:00 ngày hôm qua đến đầu giờ hiện tại của ngày hôm nay.)*

---

### 3. Tìm kiếm con (Subsearches)

Tìm kiếm con được đặt trong cặp dấu ngoặc vuông `[...]`. Kết quả của truy vấn con sẽ làm tham số cho truy vấn chính bên ngoài.

* **Ví dụ:**
```splunk
sourcetype=syslog [ search login error | return 1 user ]

```


*(Tìm trong log `syslog` tất cả sự kiện liên quan đến `user` vừa bị lỗi đăng nhập gần nhất.)*

---

### 4. Nguyên tắc Tối ưu truy vấn (Optimizing Searches)

1. **Thu hẹp dữ liệu đầu vào:** Chỉ định rõ `index` (ví dụ: `index=web` thay vì tìm trên toàn bộ index).
2. **Giới hạn thời gian:** Thu hẹp `earliest` / `latest` vừa đủ nhu cầu (dùng `-1h` thay vì `-1w`).
3. **Từ khóa cụ thể:** Tìm từ khóa rõ ràng (như `fatal_error`) thay vì dùng wildcard quá rộng (như `*error*`).
4. **Lọc sớm:** Đưa các lệnh lọc dữ liệu (như `where`, `search`, `fields`) lên càng sớm càng tốt trong chuỗi pipeline.

---

## SPL2 (Search Processing Language v2)

### 1. Khái niệm & Tổng quan

> **Mô tả:** SPL2 là phiên bản ngôn ngữ tìm kiếm thế hệ mới của Splunk, được thiết kế nhằm đơn giản hóa cú pháp, tối ưu hiệu năng, giảm thiểu các lệnh thừa/hiếm dùng và mang lại tính nhất quán cao hơn khi làm việc với các hệ thống dữ liệu lớn.

---

### 2. Điểm khác biệt chính giữa SPL1 và SPL2

* **Tính nhất quán về cú pháp:** SPL2 chuẩn hóa việc gọi hàm, sử dụng tham số và quản lý kiểu dữ liệu rõ ràng hơn.
* **Loại bỏ lệnh thừa:** Tối ưu hóa bộ lệnh, loại bỏ các lệnh ít được sử dụng hoặc các lệnh có tính năng trùng lặp từ SPL1.
* **Xử lý dữ liệu linh hoạt:** Hỗ trợ tốt hơn cho việc truy vấn cả dữ liệu cấu trúc (structured) và phi cấu trúc (unstructured).
* **Tích hợp sâu:** Tối ưu sẵn cho các môi trường như Splunk Cloud, các công cụ Machine Learning và phân tích luồng dữ liệu thời gian thực (streaming analytics).

---

### 3. Cấu trúc cú pháp cơ bản trong SPL2

#### a. Cú pháp truy vấn cơ bản

```splunk
$dataset = search <dieu_kien_tim_kiem> 
| <lenh_1> <tham_so> 
| <lenh_2> <tham_so>

```

#### b. Ví dụ minh họa

```splunk
$web_errors = search index="main" sourcetype="access_combined" status >= 400
| stats count() by status, uri
| sort -count

```

---

### 4. Bảng so sánh nhanh SPL1 vs SPL2

| Tiêu chí | SPL1 | SPL2 |
| --- | --- | --- |
| **Cú pháp lệnh** | Đôi khi thiếu nhất quán giữa các lệnh | Chuẩn hóa, chặt chẽ và nhất quán |
| **Xử lý kiểu dữ liệu** | Tự động ép kiểu linh hoạt nhưng dễ gây nhầm lẫn | Quản lý kiểu dữ liệu rõ ràng |
| **Tải công việc (Workload)** | Tối ưu cho tập dữ liệu tĩnh / Index truyền thống | Tối ưu tốt cho cả dữ liệu tĩnh và Data Streaming |
| **Nguồn tài liệu tham khảo** | *Nguồn tài liệu SPL1 truyền thống* | Tham khảo chi tiết tại [SPL2 Search Reference](https://www.splunk.com/en_us/blog/learn/splunk-cheat-sheet-query-spl-regex-commands.html) |

---

### 5. Lưu ý khi chuyển đổi sang SPL2

* **Kiểm tra tương thích:** Không phải tất cả các lệnh cũ trên SPL1 đều hoạt động nguyên bản trên SPL2.
* **Cấu trúc lệnh:** Một số biểu thức `eval` và hàm thống kê `stats` có sự điều chỉnh nhỏ về cách truyền tham số.

---

## Nhóm 1: Cơ bản - Lọc, Chọn lọc & Sắp xếp dữ liệu

### 1. `search`

> **Mô tả vắn tắt:** Lọc các sự kiện (events) hoặc kết quả tìm kiếm khớp với biểu thức điều kiện (từ khóa, cặp field=value, toán tử Boolean).

* **Cú pháp:** `| search <biểu_thức_tìm_kiếm>`
* **Giải thích tham số:**
* `<biểu_thức_tìm_kiếm>`: Chuỗi từ khóa, biểu thức so sánh (ví dụ: `status=200`), hoặc toán tử logic (`AND`, `OR`, `NOT`).


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | search status=404 OR status=500

```


* **Lưu ý / Tips:**
* Lệnh `search` nằm ngầm định ở đầu mỗi chuỗi truy vấn (pipeline).
* Đặt lệnh `search` càng sớm càng tốt để loại bỏ dữ liệu không cần thiết ngay từ đầu, giúp tăng tốc hiệu năng.



---

### 2. `fields`

> **Mô tả vắn tắt:** Giữ lại các trường (fields) cần thiết hoặc loại bỏ các trường không dùng đến trong tập kết quả.

* **Cú pháp:** `| fields [+|-] <tên_trường_1> <tên_trường_2> ...`
* **Giải thích tham số:**
* `+` (mặc định): Chỉ giữ lại danh sách các trường được liệt kê.
* `-`: Loại bỏ danh sách các trường được liệt kê khỏi kết quả.


* **Ví dụ thực tế:**
```splunk
index=web | fields + host, status, clientip

```


* **Lưu ý / Tips:**
* Loại bỏ các trường dữ liệu dung lượng lớn không cần thiết bằng `fields -` giúp giảm tải bộ nhớ và tăng tốc độ xử lý cho các lệnh phía sau.



---

### 3. `sort`

> **Mô tả vắn tắt:** Sắp xếp kết quả tìm kiếm theo thứ tự tăng dần hoặc giảm dần dựa trên một hoặc nhiều trường.

* **Cú pháp:** `| sort [count] [+|-]<tên_trường_1> [[+|-]<tên_trường_2> ...]`
* **Giải thích tham số:**
* `+` (mặc định): Sắp xếp tăng dần (Ascending).
* `-`: Sắp xếp giảm dần (Descending).
* `count`: (Tùy chọn) Giới hạn số lượng kết quả trả về sau khi sắp xếp (ví dụ: `sort 10 -count`).


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | sort clientip, -status

```


* **Lưu ý / Tips:**
* Có thể kết hợp sắp xếp nhiều trường cùng lúc bằng cách phân tách bằng dấu phẩy.



---

### 4. `head` / `tail`

> **Mô tả vắn tắt:**
> * `head`: Lấy ra $N$ kết quả đầu tiên trong tập dữ liệu.
> * `tail`: Lấy ra $N$ kết quả cuối cùng trong tập dữ liệu.
> 
> 

* **Cú pháp:**
* `| head <N>`
* `| tail <N>`


* **Giải thích tham số:**
* `<N>`: Số lượng dòng kết quả cần giữ lại (số nguyên dương).


* **Ví dụ thực tế:**
```splunk
index=main | head 20

```


* **Lưu ý / Tips:**
* Lệnh `head` rất hữu ích khi bạn muốn kiểm tra nhanh mẫu dữ liệu (sample) mà không cần chờ chạy hết toàn bộ kết quả.



---

### 5. `dedup`

> **Mô tả vắn tắt:** Loại bỏ các sự kiện trùng lặp dựa trên một hoặc nhiều trường chỉ định, giữ lại kết quả xuất hiện đầu tiên.

* **Cú pháp:** `| dedup [số_lượng_giữ_lại] <tên_trường_1> [tên_trường_2 ...]`
* **Giải thích tham số:**
* `số_lượng_giữ_lại`: (Tùy chọn) Số bản ghi trùng lặp tối đa được giữ lại cho mỗi tổ hợp giá trị (mặc định là 1).
* `<tên_trường>`: Trường dùng để kiểm tra tính trùng lặp.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | dedup clientip

```


* **Lưu ý / Tips:**
* Dùng `dedup` khi bạn chỉ muốn lấy danh sách giá trị duy nhất (unique) của một trường kèm theo toàn bộ thông tin gốc của event đó.

---

## Nhóm 2: Trung cấp - Biến đổi dữ liệu & Bảng biểu

### 1. `table`

> **Mô tả vắn tắt:** Hiển thị dữ liệu dưới dạng bảng tĩnh với các trường (fields) được chỉ định làm cột.

* **Cú pháp:** `| table <tên_trường_1> <tên_trường_2> ...`
* **Giải thích tham số:**
* `<tên_trường>`: Danh sách các trường bạn muốn giữ lại và hiển thị theo thứ tự cột từ trái sang phải.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | table _time, clientip, status, uri

```


* **Lưu ý / Tips:**
* Khác với `fields`, lệnh `table` sẽ loại bỏ toàn bộ các trường không được liệt kê (kể cả văn bản thô `_raw`) và chỉ trả về dạng bảng kết quả cuối cùng.



---

### 2. `rename`

> **Mô tả vắn tắt:** Đổi tên một hoặc nhiều trường trong tập kết quả giúp tên cột dễ đọc và trực quan hơn trên báo cáo.

* **Cú pháp:** `| rename <tên_cũ> AS <tên_mới> [<tên_cũ_2> AS <tên_mới_2> ...]`
* **Giải thích tham số:**
* `<tên_cũ>`: Tên trường hiện tại trong dữ liệu.
* `AS`: Từ khóa bắt buộc dùng để nối tên cũ và tên mới.
* `<tên_mới>`: Tên mới bạn muốn gán (nếu tên mới có khoảng trắng, cần đặt trong dấu cú pháp ngoặc kép `" "` ví dụ: `"Mã trạng thái"`).


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | rename clientip AS "Địa chỉ IP", status AS "Mã lỗi"

```


* **Lưu ý / Tips:**
* Có thể sử dụng ký tự đại diện `*` để đổi tên nhiều trường có cấu trúc tương tự (ví dụ: `rename user_* AS account_*`).



---

### 3. `eval`

> **Mô tả vắn tắt:** Tính toán biểu thức logic hoặc toán học và gán giá trị kết quả vào một trường mới hoặc ghi đè lên trường hiện có.

* **Cú pháp:** `| eval <tên_trường> = <biểu_thức_hoặc_hàm>`
* **Giải thích tham số:**
* `<tên_trường>`: Tên của trường sẽ chứa kết quả tính toán.
* `<biểu_thức_hoặc_hàm>`: Phép toán (`+`, `-`, `*`, `/`), phép nối chuỗi (`.`), hoặc các hàm `eval` như `if()`, `case()`, `lower()`, `round()`...


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | eval response_time_sec = response_time / 1000

```


* **Lưu ý / Tips:**
* `eval` là một trong những lệnh linh hoạt nhất trong SPL, dùng để làm sạch, chuẩn hóa và tạo dữ liệu tính toán mới trước khi đi vào các lệnh thống kê.



---

### 4. `where`

> **Mô tả vắn tắt:** Lọc các bản ghi dựa trên biểu thức điều kiện `eval`. Lệnh này đặc biệt dùng để so sánh giá trị giữa 2 trường khác nhau hoặc dùng phép so sánh logic phức tạp.

* **Cú pháp:** `| where <biểu_thức_logic_eval>`
* **Giải thích tham số:**
* `<biểu_thức_logic_eval>`: Điều kiện kiểm tra trả về `TRUE` hoặc `FALSE` (ví dụ: `fieldA > fieldB`, `like(field, "pattern%")`).


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | eval speed_limit = 5000 | where response_time > speed_limit

```


* **Lưu ý / Tips:**
* **Phân biệt `search` và `where`:** Lệnh `search` dùng để tìm từ khóa/chuỗi hằng số, còn lệnh `where` có thể so sánh trực tiếp giá trị của hai trường dữ liệu với nhau (ví dụ: `where bytes_in > bytes_out`).

---

## Nhóm 3: Nâng cao - Thống kê & Phân tích

### 1. `top` / `rare`

> **Mô tả vắn tắt:**
> * `top`: Thống kê các giá trị xuất hiện **nhiều nhất** của một trường, kèm theo số lượng (`count`) và tỷ lệ phần trăm (`percent`).
> * `rare`: Thống kê các giá trị xuất hiện **ít nhất** của một trường.
> 
> 

* **Cú pháp:**
* `| top [limit=N] <tên_trường_1> [by <tên_trường_2>]`
* `| rare [limit=N] <tên_trường_1> [by <tên_trường_2>]`


* **Giải thích tham số:**
* `limit=N`: Giới hạn lấy $N$ kết quả hàng đầu (mặc định là 10).
* `by`: Gom nhóm thống kê theo một trường khác.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | top limit=5 uri by status

```


* **Lưu ý / Tips:**
* Mặc định lệnh sẽ tự động tạo ra 2 cột dữ liệu bổ sung là `count` và `percent`.



---

### 2. `stats`

> **Mô tả vắn tắt:** Tính toán các chỉ số thống kê (tổng, trung bình, đếm, giá trị duy nhất...) trên tập dữ liệu và gom nhóm theo các trường chỉ định.

* **Cú pháp:** `| stats <hàm_thống_kê>(<trường>) [AS <tên_mới>] [by <tên_trường_gom_nhóm>]`
* **Giải thích tham số:**
* `<hàm_thống_kê>`: Các hàm phổ biến như `count()`, `dc()` (distinct count), `avg()`, `sum()`, `min()`, `max()`, `values()`, `earliest()`, `latest()`...
* `by`: Trường dữ liệu dùng để phân loại/nhóm kết quả.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats count, dc(clientip) as unique_ips, avg(bytes) as avg_bytes by status

```


* **Lưu ý / Tips:**
* `stats` là lệnh biến đổi dữ liệu cốt lõi nhất trong Splunk. Sau khi dùng `stats`, các trường không xuất hiện trong hàm thống kê hoặc trong mệnh đề `by` sẽ bị loại bỏ khỏi kết quả.



---

### 3. `chart` / `timechart`

> **Mô tả vắn tắt:**
> * `chart`: Trả về kết quả dưới dạng bảng hai chiều (2-axis) để dựng biểu đồ cột, đường, tròn...
> * `timechart`: Tương tự `chart` nhưng trục $X$ luôn mặc định là thời gian (`_time`).
> 
> 

* **Cú pháp:**
* `| chart <hàm_thống_kê>(<trường_1>) over <trường_trục_X> [by <trường_phân_loại>]`
* `| timechart [span=<khoảng_thời_gian>] <hàm_thống_kê>(<trường>) [by <trường_phân_loại>]`


* **Giải thích tham số:**
* `span`: (Chỉ dùng cho `timechart`) Chia nhỏ mốc thời gian thành các khoảng đều nhau (ví dụ: `span=1h`, `span=1d`, `span=5m`).
* `over`: Chỉ định trường làm trục hoành cho lệnh `chart`.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | timechart span=1h avg(response_time) by host

```


* **Lưu ý / Tips:**
* `timechart` tự động tối ưu hóa các khoảng thời gian trống và rất thích hợp để làm nguồn dữ liệu cho các Dashboard biểu đồ chuỗi thời gian (time-series visualizations).



---

### 4. `mstats`

> **Mô tả vắn tắt:** Tương tự như `stats`, nhưng được thiết kế và tối ưu riêng cho dữ liệu dạng chỉ số (Metrics Index) thay vì sự kiện văn bản (Events Index).

* **Cú pháp:** `| mstats <hàm_thống_kê>(_value) WHERE metric_name="<tên_metric>" [by <dimensions>] [span=<khoảng_thời_gian>]`
* **Giải thích tham số:**
* `_value`: Trọng tâm phép đo của chỉ số Metric.
* `WHERE metric_name`: Điều kiện lọc theo tên Metric (hỗ trợ ký tự đại diện `*`).


* **Ví dụ thực tế:**
```splunk
| mstats avg(_value), count(_value) WHERE metric_name="*.cpu.percent" by metric_name span=30s

```


* **Lưu ý / Tips:**
* `mstats` có tốc độ truy vấn vượt trội hơn nhiều so với `stats` khi làm việc với các hệ thống giám sát tải CPU, RAM, Network (dữ liệu Metrics).

---

## Nhóm 4: Chuyên sâu - Trích xuất, Làm giàu & Nhóm chuỗi dữ liệu

### 1. `rex`

> **Mô tả vắn tắt:** Sử dụng biểu thức chính quy (Regular Expression - RegEx) để trích xuất các trường dữ liệu mới từ văn bản thô (`_raw`) hoặc từ một trường hiện có ngay tại thời điểm truy vấn (search-time).

* **Cú pháp:** `| rex [field=<tên_trường>] "<regex_pattern>"`
* **Giải thích tham số:**
* `field`: (Tùy chọn) Trường dữ liệu cần áp dụng RegEx (mặc định là `_raw`).
* `<regex_pattern>`: Biểu thức RegEx chứa các nhóm được đặt tên theo dạng `(?<tên_field_mới>pattern)` để gán giá trị trích xuất vào trường mới.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | rex field=_raw "Useragent: (?<user_agent>.*) - IP"

```


* **Lưu ý / Tips:**
* `rex` rất hữu ích khi dữ liệu của bạn chưa được Splunk tự động trích xuất field sẵn ở bước Indexing.



---

### 2. `lookup`

> **Mô tả vắn tắt:** Mở rộng và làm giàu dữ liệu bằng cách đối chiếu một trường trong sự kiện với bảng dữ liệu bên ngoài (tệp CSV, KV Store, hoặc cơ sở dữ liệu) để tự động thêm các trường thông tin tương ứng.

* **Cú pháp:** `| lookup <tên_file_lookup> <trường_đối_chiếu_sự_kiện> AS <trường_đối_chiếu_lookup> OUTPUT <trường_cần_lấy_1> [trường_cần_lấy_2 ...]`
* **Giải thích tham số:**
* `<tên_file_lookup>`: Tên định danh của bảng Lookup đã được định nghĩa trong Splunk.
* `AS`: Dùng để liên kết tên trường trong log với tên cột tương ứng trong file Lookup nếu chúng khác tên.
* `OUTPUT`: Chỉ định rõ danh sách các cột/trường từ bảng Lookup mà bạn muốn thêm vào kết quả tìm kiếm.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | lookup usertable.csv clientip AS ip OUTPUT user_name, user_department

```


* **Lưu ý / Tips:**
* Việc chỉ định rõ tham số `OUTPUT` giúp tối ưu hiệu năng vì Splunk chỉ lấy đúng những thông tin bạn thực sự cần thay vì tải toàn bộ cột từ file lookup.



---

### 3. `transaction`

> **Mô tả vắn tắt:** Gom nhóm các sự kiện riêng lẻ có liên quan thành một chuỗi giao dịch (Transaction) dựa trên các trường chung (như Session ID, IP, User) và các giới hạn về mặt thời gian.

* **Cú pháp:** `| transaction <tên_trường_1> [tên_trường_2 ...] [maxspan=<thời_gian>] [maxpause=<thời_gian>] [startswith=<dieu_kien>] [endswith=<dieu_kien>]`
* **Giải thích tham số:**
* `<tên_trường>`: Các trường dùng để xác định các event thuộc cùng một giao dịch (ví dụ: `clientip`, `sessionid`).
* `maxspan`: Thời lượng tối đa cho toàn bộ một giao dịch (ví dụ: `maxspan=30m` - tổng chuỗi không quá 30 phút).
* `maxpause`: Khoảng thời gian dừng/ngắt tối đa giữa 2 sự kiện liên tiếp trong cùng giao dịch (ví dụ: `maxpause=5s`).
* `startswith` / `endswith`: Điều kiện đánh dấu sự kiện bắt đầu hoặc kết thúc của giao dịch.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | transaction clientip host maxspan=30s maxpause=5s startswith="login" endswith="logout"

```


* **Lưu ý / Tips:**
* Lệnh `transaction` sẽ tự động tạo ra hai trường mới là `duration` (tổng thời gian diễn ra giao dịch) và `eventcount` (số lượng sự kiện trong giao dịch đó).
* *Lưu ý hiệu năng:* `transaction` tiêu tốn nhiều tài nguyên bộ nhớ hơn so với `stats`, do đó nếu chỉ cần đếm hoặc nhóm cơ bản, bạn nên ưu tiên sử dụng `stats` trước.

---

## Nhóm 4: Chuyên sâu - Trích xuất, Làm giàu & Nhóm chuỗi dữ liệu

### 1. `rex`

> **Mô tả vắn tắt:** Sử dụng biểu thức chính quy (Regular Expression - RegEx) để trích xuất các trường dữ liệu mới từ văn bản thô (`_raw`) hoặc từ một trường hiện có ngay tại thời điểm truy vấn (search-time).

* **Cú pháp:** `| rex [field=<tên_trường>] "<regex_pattern>"`
* **Giải thích tham số:**
* `field`: (Tùy chọn) Trường dữ liệu cần áp dụng RegEx (mặc định là `_raw`).
* `<regex_pattern>`: Biểu thức RegEx chứa các nhóm được đặt tên theo dạng `(?<tên_field_mới>pattern)` để gán giá trị trích xuất vào trường mới.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | rex field=_raw "Useragent: (?<user_agent>.*) - IP"

```


* **Lưu ý / Tips:**
* `rex` rất hữu ích khi dữ liệu của bạn chưa được Splunk tự động trích xuất field sẵn ở bước Indexing.



---

### 2. `lookup`

> **Mô tả vắn tắt:** Mở rộng và làm giàu dữ liệu bằng cách đối chiếu một trường trong sự kiện với bảng dữ liệu bên ngoài (tệp CSV, KV Store, hoặc cơ sở dữ liệu) để tự động thêm các trường thông tin tương ứng.

* **Cú pháp:** `| lookup <tên_file_lookup> <trường_đối_chiếu_sự_kiện> AS <trường_đối_chiếu_lookup> OUTPUT <trường_cần_lấy_1> [trường_cần_lấy_2 ...]`
* **Giải thích tham số:**
* `<tên_file_lookup>`: Tên định danh của bảng Lookup đã được định nghĩa trong Splunk.
* `AS`: Dùng để liên kết tên trường trong log với tên cột tương ứng trong file Lookup nếu chúng khác tên.
* `OUTPUT`: Chỉ định rõ danh sách các cột/trường từ bảng Lookup mà bạn muốn thêm vào kết quả tìm kiếm.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | lookup usertable.csv clientip AS ip OUTPUT user_name, user_department

```


* **Lưu ý / Tips:**
* Việc chỉ định rõ tham số `OUTPUT` giúp tối ưu hiệu năng vì Splunk chỉ lấy đúng những thông tin bạn thực sự cần thay vì tải toàn bộ cột từ file lookup.



---

### 3. `transaction`

> **Mô tả vắn tắt:** Gom nhóm các sự kiện riêng lẻ có liên quan thành một chuỗi giao dịch (Transaction) dựa trên các trường chung (như Session ID, IP, User) và các giới hạn về mặt thời gian.

* **Cú pháp:** `| transaction <tên_trường_1> [tên_trường_2 ...] [maxspan=<thời_gian>] [maxpause=<thời_gian>] [startswith=<dieu_kien>] [endswith=<dieu_kien>]`
* **Giải thích tham số:**
* `<tên_trường>`: Các trường dùng để xác định các event thuộc cùng một giao dịch (ví dụ: `clientip`, `sessionid`).
* `maxspan`: Thời lượng tối đa cho toàn bộ một giao dịch (ví dụ: `maxspan=30m` - tổng chuỗi không quá 30 phút).
* `maxpause`: Khoảng thời gian dừng/ngắt tối đa giữa 2 sự kiện liên tiếp trong cùng giao dịch (ví dụ: `maxpause=5s`).
* `startswith` / `endswith`: Điều kiện đánh dấu sự kiện bắt đầu hoặc kết thúc của giao dịch.


* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | transaction clientip host maxspan=30s maxpause=5s startswith="login" endswith="logout"

```


* **Lưu ý / Tips:**
* Lệnh `transaction` sẽ tự động tạo ra hai trường mới là `duration` (tổng thời gian diễn ra giao dịch) và `eventcount` (số lượng sự kiện trong giao dịch đó).
* *Lưu ý hiệu năng:* `transaction` tiêu tốn nhiều tài nguyên bộ nhớ hơn so với `stats`, do đó nếu chỉ cần đếm hoặc nhóm cơ bản, bạn nên ưu tiên sử dụng `stats` trước.



---

# Danh sách các hàm Thống kê phổ biến (Common Stats Functions)

> Các hàm này được sử dụng làm tham số trong các lệnh thống kê như `stats`, `chart`, `timechart`, và `mstats` để gom nhóm, tính toán chỉ số, hoặc hợp nhất dữ liệu từ tập sự kiện.

---

## Nhóm 1: Thống kê Cơ bản & Đếm (Counting & Aggregations)

### 1. `count(X)`

> **Mô tả vắn tắt:** Đếm số lượng sự kiện xuất hiện. Nếu truyền vào trường $X$, chỉ đếm những sự kiện mà trường $X$ có giá trị (không `NULL`).

* **Cú pháp:** `count([truong])` hoặc `count(eval(dieu_kien))`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats count, count(clientip) as ip_count, count(eval(status>=400)) as error_count by host

```



---

### 2. `dc(X)` *(Distinct Count)*

> **Mô tả vắn tắt:** Đếm số lượng giá trị **duy nhất (không trùng lặp)** của trường $X$.

* **Cú pháp:** `dc(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats dc(clientip) as unique_users by uri

```



---

### 3. `sum(X)` / `sumsq(X)`

> **Mô tả vắn tắt:**
> * `sum(X)`: Tính tổng giá trị của tất cả các bản ghi trong trường $X$.
> * `sumsq(X)`: Tính tổng bình phương các giá trị trong trường $X$.
> 
> 

* **Cú pháp:** `sum(truong_so)` / `sumsq(truong_so)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats sum(bytes) as total_bytes_sent

```



---

### 4. `avg(X)`

> **Mô tả vắn tắt:** Tính giá trị trung bình cộng (Average) của trường $X$. *(Có thể dùng ký tự đại diện `*` cho tên trường, ví dụ: `avg(*delay)`)*.

* **Cú pháp:** `avg(truong_so)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats avg(response_time) as avg_response_time by status

```



---

## Nhóm 2: So sánh & Bách phân vị (Min/Max & Percentiles)

### 1. `min(X)` / `max(X)`

> **Mô tả vắn tắt:** Tìm giá trị nhỏ nhất (`min`) hoặc lớn nhất (`max`) của trường $X$. *(Nếu dữ liệu là dạng chuỗi chữ, kết quả sẽ dựa trên thứ tự bảng chữ cái Alphabet)*.

* **Cú pháp:** `min(truong)` / `max(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats min(response_time), max(response_time) by host

```



---

### 2. `median(X)` / `mode(X)`

> **Mô tả vắn tắt:**
> * `median(X)`: Tìm giá trị trung vị (giá trị nằm ở chính giữa tập dữ liệu đã sắp xếp) của trường $X$.
> * `mode(X)`: Tìm giá trị xuất hiện với tần suất nhiều nhất của trường $X$.
> 
> 

* **Cú pháp:** `median(truong)` / `mode(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats median(bytes), mode(status) by host

```



---

### 3. `perc<X>(Y)`

> **Mô tả vắn tắt:** Trả về giá trị đạt mốc bách phân vị thứ $X$ (Percentile) của trường $Y$ (ví dụ: `perc95(response_time)` lấy mốc bách phân vị 95%).

* **Cú pháp:** `perc<X>(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats perc95(response_time) as p95_latency, perc99(response_time) as p99_latency

```



---

## Nhóm 3: Độ biến thiên & Phân tán (Variability & Dispersion)

### 1. `range(X)`

> **Mô tả vắn tắt:** Tính khoảng biến thiên (hiệu số giữa giá trị lớn nhất và giá trị nhỏ nhất: $Max - Min$) của trường $X$.

* **Cú pháp:** `range(truong_so)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats range(response_time) as latency_range by host

```



---

### 2. `stdev(X)` / `stdevp(X)`

> **Mô tả vắn tắt:**
> * `stdev(X)`: Tính độ lệch chuẩn mẫu (Sample Standard Deviation) của trường $X$.
> * `stdevp(X)`: Tính độ lệch chuẩn tổng thể (Population Standard Deviation) của trường $X$.
> 
> 

* **Cú pháp:** `stdev(truong_so)` / `stdevp(truong_so)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats stdev(bytes) as bytes_std_dev by status

```



---

### 3. `var(X)`

> **Mô tả vắn tắt:** Tính phương sai mẫu (Sample Variance) của trường $X$.

* **Cú pháp:** `var(truong_so)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats var(response_time) as latency_variance

```



---

## Nhóm 4: Thứ tự Thời gian & Liệt kê Giá trị (Values & Chronology)

### 1. `earliest(X)` / `latest(X)`

> **Mô tả vắn tắt:** Trả về giá trị của trường $X$ xuất hiện sớm nhất (`earliest`) hoặc mới nhất (`latest`) theo thứ tự thời gian.

* **Cú pháp:** `earliest(truong)` / `latest(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats earliest(_time) as first_seen, latest(status) as current_status by clientip

```



---

### 2. `values(X)`

> **Mô tả vắn tắt:** Gom tất cả các giá trị duy nhất của trường $X$ thành một danh sách đa giá trị (Multivalued entry), sắp xếp theo thứ tự bảng chữ cái.

* **Cú pháp:** `values(truong)`
* **Ví dụ thực tế:**
```splunk
sourcetype=access_combined | stats values(status) as all_statuses, values(uri) as visited_uris by clientip

```

---

# Các ví dụ truy vấn thực tế (Search Examples)

## 1. Lọc & Sắp xếp dữ liệu (Filter & Order Results)

* **Lấy ra 20 kết quả đầu tiên:**
```splunk
... | head 20

```


* **Lấy 20 kết quả cuối cùng theo thứ tự ngược lại:**
```splunk
... | tail 20

```


* **Đảo ngược thứ tự tập kết quả:**
```splunk
... | reverse

```


* **Sắp xếp theo IP tăng dần, sau đó theo URL giảm dần:**
```splunk
... | sort ip, -url

```



---

## 2. Gom nhóm & Chuỗi giao dịch (Group Results)

* **Gom nhóm sự kiện có cùng IP, bắt đầu bằng `signon` và kết thúc bằng `purchase`:**
```splunk
... | transaction clientip startswith="signon" endswith="purchase"

```


* **Gom các sự kiện có cùng `host` và `cookie` trong khoảng 30 giây, không nghỉ quá 5 giây:**
```splunk
... | transaction host cookie maxspan=30s maxpause=5s

```


* **Gom cụm kết quả tương đồng (Cluster) và lấy 20 cụm lớn nhất:**
```splunk
... | cluster t=0.9 showcount=true | sort limit=20 -cluster_count

```



---

## 3. Báo cáo Thống kê & Biểu đồ (Reporting & Visualization)

* **Tạo bảng đếm sự kiện và hiển thị biểu đồ nhỏ (sparkline):**
```splunk
... | stats sparkline count by host

```


* **Thống kê thời gian đếm sự kiện từ nguồn web phân theo host:**
```splunk
... | timechart count by host

```


* **Tính giá trị trung bình CPU mỗi phút cho từng host:**
```splunk
... | timechart span=1m avg(CPU) by host

```


* **Thống kê giá trị trung bình của tất cả các trường có hậu tố `lay` (như delay, xdelay):**
```splunk
... | stats avg(*lay) by date_hour

```


* **Lấy Top 20 URL xuất hiện nhiều nhất:**
```splunk
... | top limit=20 url

```



---

## 4. Thống kê Bổ sung & Báo cáo Nâng cao (Advanced Reporting)

* **Tính thời lượng trung bình toàn cục và thêm trường `avgdur` vào từng event gốc:**
```splunk
... | eventstats avg(duration) as avgdur

```


* **Tính tổng dồn (cumulative sum) của dung lượng `bytes` theo thời gian:**
```splunk
... | streamstats sum(bytes) as bytes_total | timechart max(bytes_total)

```


* **Dự báo xu hướng dựa trên biểu đồ thời gian (Predictive Analytics):**
```splunk
... | timechart count | predict count

```


* **Tính đường trung bình động (Moving Average) cho 5 sự kiện gần nhất:**
```splunk
... | timechart count | trendline sma5(count) as smoothed_count

```


* **Liệt kê danh sách tên tất cả các metric trong chỉ mục `_metrics`:**
```splunk
| mcatalog values(metric_name) WHERE index=_metrics

```

# Bài 3: Xử lý xung đột phức tạp trong quá trình Rebase

## 1. Mục tiêu

Thực hành xử lý xung đột phức tạp khi sử dụng `git rebase`, cụ thể:

* Phân biệt cách xử lý conflict giữa `merge` và `rebase`.
* Giải quyết conflict thủ công ở từng commit trong quá trình rebase.
* Đưa các commit của nhánh `feature-api` lên trên các commit mới nhất của `main`.
* Đảm bảo lịch sử Git sau cùng thẳng và không phát sinh merge commit.

---

## 2. Bối cảnh

Ban đầu, nhánh `main` có file `config.json`:

```json
{
  "port": 8080,
  "debug": false
}
```

Sau đó tạo nhánh `feature-api` từ `main`.

### Các commit trên `feature-api`

Nhánh `feature-api` có 2 commit:

1. `feat: change port`

   * Thay đổi `port` từ `8080` thành `9000`.

2. `feat: enable debug`

   * Thay đổi `debug` từ `false` thành `true`.

### Các commit mới trên `main`

Trong khi đó, nhánh `main` có thêm 2 commit:

1. `update port on main`

   * Thay đổi `port` từ `8080` thành `8081`.

2. `add env config`

   * Thêm cấu hình môi trường:

```json
"env": "production"
```

Do `main` và `feature-api` cùng sửa các vùng liên quan trong `config.json`, khi thực hiện rebase sẽ phát sinh conflict.

---

## 3. Các bước thực hiện

### Bước 1: Khởi tạo cấu hình ban đầu

Đứng trên nhánh `main` và tạo file `config.json`:

```json
{
  "port": 8080,
  "debug": false
}
```

Commit:

```bash
git add config.json
git commit -m "init config"
```

---

### Bước 2: Tạo nhánh feature-api

```bash
git checkout -b feature-api
```

Thay đổi `port`:

```json
{
  "port": 9000,
  "debug": false
}
```

Commit:

```bash
git add config.json
git commit -m "feat: change port"
```

Tiếp tục thay đổi `debug`:

```json
{
  "port": 9000,
  "debug": true
}
```

Commit:

```bash
git add config.json
git commit -m "feat: enable debug"
```

---

### Bước 3: Tạo các thay đổi mới trên main

Quay lại `main`:

```bash
git checkout main
```

Thay đổi `port` thành `8081`:

```json
{
  "port": 8081,
  "debug": false
}
```

Commit:

```bash
git add config.json
git commit -m "update port on main"
```

Tiếp tục thêm cấu hình `env`:

```json
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
```

Commit:

```bash
git add config.json
git commit -m "add env config"
```

---

## 4. Thực hiện Rebase

Quay lại nhánh `feature-api`:

```bash
git checkout feature-api
```

Kiểm tra trạng thái:

```bash
git status
```

Thực hiện rebase:

```bash
git rebase main
```

Git bắt đầu áp dụng lần lượt các commit của `feature-api` lên trên `main`.

---

## 5. Conflict lần 1 - Commit `feat: change port`

Khi Git áp dụng commit:

```text
feat: change port
```

xung đột xảy ra trong file:

```text
config.json
```

Nguyên nhân:

* `main` đã thay đổi `port` từ `8080` thành `8081`.
* `feature-api` muốn thay đổi `port` từ `8080` thành `9000`.

Git không thể tự quyết định giá trị nào cần giữ.

Kiểm tra trạng thái:

```bash
git status
```

Mở `config.json`, Git có thể đánh dấu conflict tương tự:

```text
<<<<<<< HEAD
{
  "port": 8081,
  "debug": false,
  "env": "production"
}
=======
{
  "port": 9000,
  "debug": false
}
>>>>>>> feat: change port
```

### Cách giải quyết

Vì mục tiêu của nhánh `feature-api` là thay đổi port thành `9000`, đồng thời không làm mất cấu hình `env` của `main`, tiến hành chỉnh sửa thủ công thành:

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

Sau khi lưu file:

```bash
git add config.json
```

Tiếp tục rebase:

```bash
git rebase --continue
```

Nếu Git mở trình soạn thảo commit message, giữ nguyên message:

```text
feat: change port
```

và lưu lại để tiếp tục.

---

## 6. Conflict lần 2 - Commit `feat: enable debug`

Sau khi giải quyết conflict đầu tiên, Git tiếp tục áp dụng commit thứ hai:

```text
feat: enable debug
```

Nếu Git phát sinh conflict trong `config.json`, kiểm tra:

```bash
git status
```

Conflict có thể xuất hiện do commit này được tạo trên trạng thái cũ của `feature-api`, trong khi file hiện tại đã bao gồm các thay đổi từ `main` và kết quả giải quyết conflict trước đó.

Tiến hành mở `config.json` và giải quyết thủ công.

Mục tiêu cuối cùng là giữ toàn bộ thay đổi cần thiết:

* `port` = `9000` từ `feature-api`.
* `debug` = `true` từ commit `feat: enable debug`.
* `env` = `"production"` từ `main`.

Kết quả mong muốn:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Sau khi chỉnh sửa:

```bash
git add config.json
```

Tiếp tục:

```bash
git rebase --continue
```

Nếu Git yêu cầu xác nhận commit message, giữ nguyên:

```text
feat: enable debug
```

---

## 7. Kiểm tra sau khi Rebase hoàn tất

Kiểm tra trạng thái:

```bash
git status
```

Kết quả mong muốn:

```text
On branch feature-api
nothing to commit, working tree clean
```

Kiểm tra lịch sử:

```bash
git log --graph --oneline
```

Lịch sử mong muốn có dạng thẳng:

```text
* <commit mới> feat: enable debug
* <commit mới> feat: change port
* <commit> add env config
* <commit> update port on main
* <commit> init config
```

Các commit của `feature-api` được đặt nối tiếp phía trên các commit mới nhất của `main`.

Không xuất hiện merge commit.

---

## 8. Kết quả cuối cùng của config.json

Sau khi hoàn thành rebase, nội dung `config.json` là:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Trong đó:

| Cấu hình            | Nguồn thay đổi |
| ------------------- | -------------- |
| `port: 9000`        | `feature-api`  |
| `debug: true`       | `feature-api`  |
| `env: "production"` | `main`         |

---

## 9. Các lệnh Git đã sử dụng

```bash
git checkout main

git checkout -b feature-api

git add config.json
git commit -m "init config"

git commit -m "feat: change port"
git commit -m "feat: enable debug"

git checkout main

git commit -m "update port on main"
git commit -m "add env config"

git checkout feature-api

git rebase main

git status

git add config.json
git rebase --continue

git status
git log --graph --oneline
```

Trong trường hợp muốn hủy toàn bộ quá trình rebase và quay lại trạng thái trước khi rebase:

```bash
git rebase --abort
```

---

## 10. Tổng kết quá trình xử lý Conflict

### Conflict lần 1

**Commit đang được áp dụng:**

```text
feat: change port
```

**Vấn đề:**

`main` thay đổi `port` thành `8081`, trong khi `feature-api` thay đổi thành `9000`.

**Cách giải quyết:**

Giữ `port: 9000` theo yêu cầu của `feature-api` và đồng thời giữ `env: "production"` từ `main`.

Kết quả:

```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

Sau đó:

```bash
git add config.json
git rebase --continue
```

### Conflict lần 2

**Commit đang được áp dụng:**

```text
feat: enable debug
```

**Vấn đề:**

Commit tiếp theo của `feature-api` cần được áp dụng trên phiên bản `config.json` đã được giải quyết conflict ở bước trước.

**Cách giải quyết:**

Giữ các thay đổi hợp lệ của cả hai nhánh:

```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Sau đó:

```bash
git add config.json
git rebase --continue
```

---

## 11. Ảnh chụp kết quả

### Git log sau khi hoàn thành Rebase

Chạy:

```bash
git log --graph --oneline
```

Chèn ảnh chụp màn hình kết quả vào đây:

> **Ảnh chụp:** `git-log-after-rebase.png`

![Git log sau khi hoàn thành Rebase](git-log-after-rebase.png)

---

## 12. Kết luận

Qua bài thực hành, em đã thực hiện thành công quá trình rebase nhánh `feature-api` lên nhánh `main` và xử lý các conflict phát sinh theo từng commit.

Các conflict được giải quyết thủ công bằng cách:

1. Kiểm tra trạng thái bằng `git status`.
2. Mở file bị conflict.
3. Phân tích thay đổi của `main` và `feature-api`.
4. Chỉnh sửa thủ công để giữ các thay đổi cần thiết.
5. Chạy `git add`.
6. Sử dụng `git rebase --continue` để tiếp tục quá trình rebase.
7. Kiểm tra lịch sử bằng `git log --graph --oneline`.

Kết quả cuối cùng là lịch sử Git được sắp xếp thẳng, không có merge commit phụ, và các commit của `feature-api` nằm nối tiếp sau các commit mới nhất của `main`.

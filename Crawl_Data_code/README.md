# README - Hướng dẫn Crawl Data Tiki

**Mục đích:** Bộ công cụ phục vụ việc crawl **Thông tin sản phẩm** và **Bình luận đánh giá** từ Tiki theo cơ chế bán tự động, tối giản sự can thiệp của con người.

# I. Giới thiệu các thành phần

Bộ code bao gồm 3 file thực thi chính:

1. `product_id.py`: Lấy danh sách ID sản phẩm từ danh mục.
2. `product_data.py`: Crawl thông tin chi tiết của từng sản phẩm.
3. `comments_data.py`: Crawl dữ liệu bình luận và đánh giá từ khách hàng.

# II. Yêu cầu hệ thống

# 1. Thư viện hỗ trợ

Dự án sử dụng các thư viện: `requests`, `pandas`, `tqdm`, `selenium`.

# 2. Cài đặt nhanh

Chạy lệnh sau trong terminal để cài đặt các thư viện cần thiết:

**pip install requests pandas tqdm selenium**

# III. Cấu trúc dữ liệu

# 1. Danh sách ID sản phẩm (`product_{id}.csv`)

| Cột                | Mô tả                                   |
| :----------------- | :-------------------------------------- |
| `product_id`       | ID của sản phẩm                         |

# 2. Thông tin sản phẩm (`data_{id}.csv`)

| Cột             | Mô tả                                     |
| :-------------- | :---------------------------------------- |
| `product_id`    | ID của sản phẩm                           |
| `product_name`  | Tên của sản phẩm                          |

# 3. Dữ liệu bình luận (`comments_data_{id}.csv`)

| Cột             | Mô tả                                     |
| :-------------- | :---------------------------------------- |
| `comment_id`    | ID của bình luận                          |
| `product_id`    | ID của sản phẩm                           |
| `customer_id`   | ID của khách hàng                         |
| `customer_name` | Tên của khách hàng                        |
| `rating`        | Số sao đánh giá                           |
| `title`         | Tiêu đề bình luận                         |
| `content`       | Nội dung đánh giá                         |
| `created_at`    | Thời điểm viết đánh giá                   |
| `purchased_at`  | Thời điểm mua sản phẩm                    |
| `seller_id `    | ID của shop bán hàng                      |
| `seller_name`   | Tên của shop bán hàng                     |


# IV. Hướng dẫn sử dụng

# Bước 1: Chuẩn bị ID danh mục

Xác định `id` của danh mục cần crawl từ đường dẫn trên trình duyệt.  
_Ví dụ:_ Với URL `https://tiki.vn/nha-sach-tiki/c8322`, lấy `id = 8322`.  
**Lưu ý:** Cập nhật ID này vào biến cấu hình ở đầu mỗi file Python.

# Bước 2: Thu thập danh sách ID sản phẩm

Chạy script để lấy toàn bộ danh sách ID sản phẩm thuộc danh mục đã chọn.

- **Input:** ID danh mục.
- **Output:** File `product_{id}.csv`.

```bash
python product_id.py
```

# Bước 3: Crawl thông tin chi tiết sản phẩm

Sử dụng danh sách ID từ Bước 1 để lấy thông tin chi tiết từng sản phẩm.

- **Input:** File `product_{id}.csv`
- **Output:** File `data_{id}.csv`

```bash
python product_data.py
```

# Bước 4: Crawl bình luận và đánh giá

Thu thập toàn bộ lịch sử đánh giá của khách hàng dựa trên danh sách sản phẩm.

- **Input:** File `product_{id}.csv`
- **Output:** File `comments_data_{id}.csv`

```bash
python comments_data.py
```

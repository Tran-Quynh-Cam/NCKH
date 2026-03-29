# NCKH - Phát hiện spam review tiếng Việt (Tiki)

## 1. Tổng quan dự án

Dự án gồm 3 khối chính:

1. Crawl dữ liệu từ Tiki (sản phẩm + review)
2. Xử lý văn bản và tạo đặc trưng hành vi
3. Pseudo-labeling để mở rộng tập huấn luyện

Pipeline tổng quan:

1. Lấy danh sách `product_id` theo danh mục
2. Crawl thông tin sản phẩm
3. Crawl review theo từng sản phẩm
4. Làm sạch dữ liệu và tạo feature
5. Huấn luyện mô hình cơ sở (FastText + behavioral features)
6. Chạy pseudo-labeling theo nhiều vòng
7. Lưu mô hình cuối, scaler và lịch sử vòng lặp

## 2. Nguồn dữ liệu

Google Drive data:

https://drive.google.com/drive/folders/1G1V8hyTfphz7Pa_h9ZLqNeVwN5Py3tV6?usp=drive_link

Các file dữ liệu chính được dùng trong notebook:

- `Product.csv`
- `Comments.csv`
- `merged_df.csv`
- `labeled_db.csv`

Artifact đầu ra đã có trong repo:

- `Pseudo_labeling_code/pseudo_history.csv`
- `Pseudo_labeling_code/final_pseudo_model.pkl`
- `Pseudo_labeling_code/scaler.pkl`

## 3. Cấu trúc workspace

```text
NCKH/
|-- README.md
|-- Crawl_Data_code/
|   |-- README.md
|   |-- product_id.py
|   |-- product_data.py
|   `-- comments_data.py
|-- Processing_code/
|   |-- Processing.ipynb
|   |-- 10__dataset.csv
|   `-- vietnamese-stopwords.txt
`-- Pseudo_labeling_code/
    |-- pseudo_labeling_fasttext.ipynb
    |-- pseudo_history.csv
    |-- final_pseudo_model.pkl
    `-- scaler.pkl
```

## 4. Mô tả các module

### 4.1 Crawl_Data_code

- `product_id.py`
  - Lấy cookie + guest token bằng Selenium
  - Gọi API `api/v2/products` theo category
  - Lưu `product_{id}.csv`

- `product_data.py`
  - Đọc `product_{id}.csv`
  - Gọi API `api/v2/products/{product_id}`
  - Trích xuất `product_id`, `product_name`
  - Lưu `data_{id}.csv`

- `comments_data.py`
  - Đọc `product_{id}.csv`
  - Gọi API `api/v2/reviews`
  - Trích xuất các trường review
  - Loại bỏ `content` rỗng
  - Lưu `comments_data_{id}.csv`

Lưu ý:

- Cả 3 script đang dùng biến `id = 8322` (hard-code).
- Cần Chrome/Chromedriver tương thích để chạy Selenium headless.

### 4.2 Processing_code

Notebook chính: `Processing.ipynb`

Nội dung xử lý chính:

1. Nạp `Product.csv` và `Comments.csv`
2. Làm sạch/chuẩn hóa văn bản
3. Tạo behavioral features:
   - `delta_minutes`
   - `small_gap`
   - `burst_count`
   - `F3_flag`
   - `mean_similarity`
   - `max_similarity`
   - `F4_Flag`
   - `time_gap_minutes`
   - `F5_flag`
4. Tổng hợp thành `merged_df`
5. Tạo mẫu 10% cho tập ban đầu có nhãn

File hiện có:

- `Processing_code/vietnamese-stopwords.txt` (205 dòng)
- `Processing_code/10__dataset.csv` (32068 dòng)

### 4.3 Pseudo_labeling_code

Notebook chính: `pseudo_labeling_fasttext.ipynb`

Nội dung chính:

1. Nạp `labeled_db.csv` và `merged_df.csv`
2. Tách tập `unlabeled`
3. Tạo embedding FastText (`cc.vi.300.bin`) + tokenization Underthesea
4. Kết hợp 310 features:
   - 300 chiều embedding văn bản
   - 10 behavioral features
5. So sánh mô hình cơ sở: RandomForest vs LightGBM
6. Chạy pseudo-labeling nhiều vòng với threshold động
7. Lưu model/scaler/lịch sử

Artifact đã có:

- `Pseudo_labeling_code/pseudo_history.csv` (10 vòng)
- `Pseudo_labeling_code/final_pseudo_model.pkl`
- `Pseudo_labeling_code/scaler.pkl`

## 5. Hướng dẫn chạy nhanh

### 5.1 Cài đặt dependencies

Cho crawl scripts:

```bash
pip install requests pandas tqdm selenium
```

Cho notebook pseudo-labeling:

```bash
pip install underthesea fasttext lightgbm scikit-learn joblib scipy matplotlib
```

### 5.2 Chạy crawl

```bash
cd Crawl_Data_code
python product_id.py
python product_data.py
python comments_data.py
```

### 5.3 Chạy processing và pseudo-labeling

1. Chạy `Processing_code/Processing.ipynb` từ trên xuống.
2. Chạy `Pseudo_labeling_code/pseudo_labeling_fasttext.ipynb` từ trên xuống.

## 6. Schema đầu ra chính

### 6.1 `comments_data_{id}.csv`

- `comment_id`
- `product_id`
- `customer_id`
- `customer_name`
- `rating`
- `title`
- `content`
- `created_at`
- `purchased_at`
- `seller_id`
- `seller_name`

### 6.2 Processed dataset (`10__dataset.csv`)

Ví dụ các cột:

- `product_name`
- `content_text`
- `rating`
- `delta_minutes`
- `small_gap`
- `burst_count`
- `F3_flag`
- `mean_similarity`
- `max_similarity`
- `F4_Flag`
- `time_gap_minutes`
- `F5_flag`

## 7. Ghi chú tái lập

- Notebook đang dùng flow Google Colab + Google Drive, cần sửa `root_path` nếu chạy local.
- File pretrained FastText `cc.vi.300.bin` không có sẵn trong repo vì dung lượng lớn.
- Phần crawl đã có delay/retry, nên giữ nguyên để hạn chế bị chặn.
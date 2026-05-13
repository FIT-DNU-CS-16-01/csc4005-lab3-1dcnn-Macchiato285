# CSC4005 Lab 3 Report – UrbanSound8K with 1D-CNN

## 1. Thông tin sinh viên

- Họ tên:Ngô Nguyễn Quang Linh
- Mã sinh viên:1671040018
- Lớp:KHMT 16-01
- Link GitHub repo:https://github.com/FIT-DNU-CS-16-01/csc4005-lab3-1dcnn-Macchiato285
- Link W&B run/project:https://wandb.ai/ngoquanglinh2853-none/csc4005-lab3-urbansound-1dcnn?nw=nwuserngoquanglinh2853

---

## 2. Mục tiêu thí nghiệm

Mô tả ngắn gọn mục tiêu của lab:

- phân loại âm thanh môi trường trên UrbanSound8K,
- sử dụng MFCC/log-mel làm chuỗi đặc trưng theo thời gian,
- xây dựng và huấn luyện 1D-CNN,
- theo dõi thí nghiệm bằng W&B,
- phân tích lỗi bằng confusion matrix.

---

## 3. Dữ liệu và tiền xử lý

### 3.1. Dataset

- Dataset: UrbanSound8K
- Số lớp: 10
- Các lớp:
    air_conditioner
    car_horn
    children_playing
    dog_bark
    drilling
    engine_idling
    gun_shot
    jackhammer
    siren
    street_music
- Fold dùng để train: fold1 → fold8
- Fold dùng để validation: fold9
- Fold dùng để test: fold10

### 3.2. Tiền xử lý audio

Điền cấu hình đã dùng:

| Thành phần | Giá trị |
|---|---|
| Sample rate | 16000 |
| Duration | 4.0 |
| Feature type | MFCC  |
| n_mfcc / n_mels | 40 |
| n_fft | 1024 |
| hop_length | 512 |
| Augmentation | Có augmentation nhẹ |

Giải thích ngắn: vì sao cần đưa audio về cùng sample rate và cùng độ dài?

- Trong bài toán audio classification, các file âm thanh thường có sample rate và độ dài khác nhau. Vì vậy cần chuẩn hóa toàn bộ audio về cùng sample rate để dữ liệu đồng nhất và mô hình học ổn định hơn. Ngoài ra, việc pad/crop audio về cùng độ dài giúp tất cả input có cùng shape trước khi đưa vào Conv1D.

## 4. Mô hình 1D-CNN

Mô tả kiến trúc mô hình:

```text
Input feature sequence
→ Conv1D block 1
→ Conv1D block 2
→ Conv1D block 3
→ Global Average Pooling
→ Dense classifier
→ Softmax
```

Bảng cấu hình:

| Thành phần | Giá trị |
|---|---|
| model_name | mfcc_1dcnn |
| hidden_channels | 64 |
| dropout | 0.3 |
| optimizer | AdamW |
| learning rate | 0.001 |
| weight decay | 0.0001 |
| batch size | 32 |
| epochs | 3 |
| patience | 3 |

---

## 5. Kết quả thực nghiệm

### 5.1. Kết quả chính

| Metric | Giá trị |
|---|---:|
| Best validation accuracy | 0.305 |
| Test accuracy | Chưa chạy baseline đầy đủ |
| Average epoch time | Khoảng vài giây / epoch |
| Total parameters | ~77,000 |
| Trainable parameters | ~77,000 |

### 5.2. Learning curves

Chèn hình `curves.png`.

Nhận xét:

- Train loss và validation loss đều giảm theo epoch.
- Train accuracy và validation accuracy tăng dần trong quá trình huấn luyện.
- Chưa xuất hiện dấu hiệu overfitting rõ rệt vì train_acc và val_acc vẫn tăng cùng nhau.
- Early stopping chưa xảy ra do mới chạy cấu hình debug với số epoch nhỏ.

### 5.3. Confusion matrix

Chèn hình `confusion_matrix.png`.

Nhận xét:

- Một số lớp như drilling và jackhammer có thể dễ bị nhầm do đặc trưng âm thanh tương đối giống nhau.
- Các lớp chứa nhiều nhiễu nền hoặc âm thanh liên tục cũng dễ gây nhầm lẫn.
- Nguyên nhân có thể đến từ đặc trưng âm thanh tương tự, độ dài clip ngắn và dữ liệu môi trường có nhiều noise.

---

## 6. W&B tracking

Dán link W&B:

```text
https://wandb.ai/ngoquanglinh2853-none/csc4005-lab3-urbansound-1dcnn?nw=nwuserngoquanglinh2853
```

Ảnh chụp hoặc mô tả dashboard cần có:

- learning curves,
- final metrics,
- configuration,
- confusion matrix image.

---

## 7. Phân tích và thảo luận

Trả lời ngắn các câu hỏi:

1.Vì sao dùng 1D-CNN thay vì MLP cho chuỗi đặc trưng audio?
1D-CNN có khả năng học các pattern cục bộ theo thời gian của chuỗi audio, trong khi MLP không tận dụng được cấu trúc thời gian của dữ liệu.
2.Kernel 1D trong bài này đang trượt theo chiều nào?
Kernel Conv1D trượt theo chiều thời gian của chuỗi MFCC.
3.MFCC giúp mô hình học dễ hơn raw waveform ở điểm nào?
MFCC đã tóm tắt thông tin phổ âm thanh thành dạng đặc trưng gọn hơn nên mô hình học nhanh và ổn định hơn so với waveform thô.
4.Mô hình hiện tại còn hạn chế gì?
Accuracy chưa cao.
Một số lớp âm thanh dễ bị nhầm.
Dataset có nhiều nhiễu nền.
Mô hình còn khá nhỏ nên khả năng biểu diễn còn hạn chế.
5Có thể cải thiện kết quả bằng cách nào?
Train nhiều epoch hơn.
Tăng dữ liệu train.
Dùng log-mel thay cho MFCC.
Áp dụng augmentation mạnh hơn.
Sử dụng mô hình sâu hơn hoặc raw waveform CNN.
---

## 8. Bài mở rộng nếu có

Nếu làm raw waveform hoặc log-mel, điền bảng sau:

| Pipeline | Feature/Input | Test accuracy | Nhận xét |
|---|---|---:|---|
| Baseline | MFCC + 1D-CNN |  |  |
| Extension 1 | log-mel + 1D-CNN |  |  |
| Extension 2 | raw waveform + 1D-CNN |  |  |

---

## 9. Kết luận

- Bài lab giúp hiểu quy trình phân loại âm thanh bằng Deep Learning.
- MFCC là đặc trưng phù hợp cho bài toán audio classification cơ bản.
- 1D-CNN có khả năng học các pattern thay đổi theo thời gian của âm thanh.
- W&B hỗ trợ theo dõi learning curves và so sánh các thí nghiệm hiệu quả.
- Việc phân tích confusion matrix giúp hiểu rõ mô hình đang nhầm ở những lớp nào và nguyên nhân gây lỗi.
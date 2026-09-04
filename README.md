# Phân Tích Dữ Liệu Khách Hàng Viễn Thông (Telco Customer Churn)

Dự án phân tích và dự đoán hành vi rời bỏ của khách hàng (Customer Churn) trong lĩnh vực viễn thông, từ đó đưa ra các giải pháp giữ chân khách hàng hiệu quả. 

## 1. Thành Viên Nhóm
*   **Trương Thế Minh Khánh** (23520727): Làm sạch dữ liệu, Kỹ thuật đặc trưng (Feature Engineering), Mã hoá và chuẩn hoá dữ liệu, Chuẩn bị dữ liệu đầu vào.
*   **Trần Anh Duy** (23520389): Phân tích khám phá dữ liệu (EDA), phân tích các yếu tố ảnh hưởng, trực quan hoá dữ liệu và rút ra insight.
*   **Nguyễn Văn Tiên** (23521580): Huấn luyện mô hình, đánh giá, so sánh và tối ưu hoá mô hình.

## 2. Giới Thiệu Bộ Dữ Liệu
*   **Tên dataset**: `WA_Fn-UseC_-Telco-Customer-Churn.csv` (nguồn: Kaggle).
*   **Quy mô**: 7.043 dòng và 21 thuộc tính.
*   **Đặc điểm**: Dữ liệu bao gồm các mảng thông tin chính như Thông tin nhân khẩu học (gender, SeniorCitizen, Partner,...), Thông tin tài khoản (tenure, Contract, PaymentMethod, MonthlyCharges,...) và Các dịch vụ đăng ký (InternetService, TechSupport, StreamingTV,...).

## 3. Quy Trình Thực Hiện

### 3.1. Tiền Xử Lý Dữ Liệu (Data Preprocessing)
*   **Làm sạch dữ liệu**: Xử lý kiểu dữ liệu không hợp lệ (biến `TotalCharges` từ Object sang Numeric) và điền khuyết các khách hàng có `tenure = 0` bằng giá trị `0`.
*   **Kỹ thuật đặc trưng (Feature Engineering)**: Khai phá thêm 5 biến mới để tăng cường độ chính xác cho mô hình:
    *   `TotalServices`: Đếm tổng số dịch vụ khách hàng đang sử dụng.
    *   `AvgPricePerService`: Giá cước trung bình trên mỗi dịch vụ.
    *   `FamilyCluster`: Phân loại quy mô gia đình (từ biến Partner và Dependents).
    *   `SupportPackage`: Đếm số lượng dịch vụ kỹ thuật hỗ trợ.
    *   `HighRiskProfile`: Gắn cờ nhóm rủi ro cao (Hợp đồng theo tháng + Thanh toán Séc điện tử).
*   **Chuẩn bị huấn luyện**: Mã hoá One-Hot Encoding cho các biến phân loại, Scaling (StandardScaler) cho các biến số, và chia tập Train/Test theo tỷ lệ 80/20 có Stratify.

### 3.2. Phân Tích Khám Phá (EDA) & Insights
*   **Tình trạng mất cân bằng dữ liệu**: Tỷ lệ khách hàng rời bỏ (Churn) chiếm 26.5%, trong khi khách hàng ở lại chiếm 73.5%.
*   **Nhóm khách hàng cao tuổi (Senior Citizens)**: Có tỷ lệ rời bỏ rất cao (41.7%), đặc biệt là khi không sử dụng dịch vụ hỗ trợ kỹ thuật (rời bỏ 50.6%) và sống độc thân (rời bỏ 40.2%).
*   **Dịch vụ Fiber Optic (Cáp quang)**: Mang lại doanh thu cao nhất (trung bình $92/tháng) nhưng lại có tỷ lệ rời bỏ cao nhất (~42%). Khi khách hàng được tích hợp vào hệ sinh thái sử dụng nhiều dịch vụ cùng lúc (7-8 dịch vụ), tỷ lệ rời bỏ giảm sâu xuống chỉ còn 8.3% - 17.4%.
*   **Loại hợp đồng & Thanh toán**: Hợp đồng trả từng tháng (Month-to-month) và thanh toán bằng Electronic check là hai yếu tố cảnh báo rủi ro rời bỏ cao nhất.

### 3.3. Xây Dựng & Tối Ưu Mô Hình
*   **Baseline Models**: Thử nghiệm 5 mô hình phổ biến bao gồm Logistic Regression, Random Forest, SVM, KNN, và Decision Tree. Logistic Regression ban đầu cho kết quả ROC-AUC tốt nhất (0.8422) nhưng Recall lại khá thấp (0.5455).
*   **Feature Selection**: Tổng hợp từ 5 phương pháp (Correlation, LR Weights, RF MDI, Permutation Importance) để chọn ra 9 biến mang tính cốt lõi nhất nhằm giảm nhiễu (tenure, TotalCharges, MonthlyCharges, InternetService, Contract, AvgPricePerService, PaperlessBilling, TotalServices, PaymentMethod).
*   **Fine-Tuning**: Tối ưu hoá siêu tham số bằng GridSearchCV. Sau tinh chỉnh, mô hình Random Forest cho thấy sự vượt trội khi Recall tăng vọt từ 0.4759 lên 0.7380 và ROC-AUC đạt 0.8408.
*   **Threshold Moving (Tối ưu ngưỡng)**: Do đặc thù rủi ro chi phí (bỏ sót khách hàng rời bỏ gây thiệt hại cao hơn chăm sóc nhầm), nhóm đã điều chỉnh ngưỡng ra quyết định (threshold) của Random Forest xuống mức `0.4`. Tại ngưỡng này, mô hình phát hiện được tới **83%** khách hàng có nguy cơ rời bỏ (Recall = 0.8289) với mức Accuracy ổn định (0.7346).

## 4. Kết Luận
Bằng việc kết hợp Feature Engineering, giảm chiều dữ liệu và tìm kiếm Threshold tối ưu, dự án đã xây dựng thành công một mô hình Random Forest nhạy bén trong việc đánh hơi rủi ro khách hàng rời đi. Doanh nghiệp có thể dựa vào mô hình này để thiết kế các chiến dịch giữ chân khách hàng mục tiêu, cụ thể như tập trung cung cấp Tech Support cho người cao tuổi và xây dựng gói hệ sinh thái dịch vụ đi kèm cho người dùng cáp quang.

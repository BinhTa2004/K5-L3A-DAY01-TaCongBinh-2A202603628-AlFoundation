# Báo cáo cá nhân — Day 01: Lọc tin nhắn rác

- **SEED:** 6231; thử thách số 4.
- **Dữ liệu:** 5.169 tin sau bỏ trùng, gồm 653 spam và 4.516 ham. Train/validation/test: 3.619/775/775 tin; SUB gồm 20 spam và 20 ham.
- **LLM thực tế trong notebook:** OpenRouter, model nhỏ `openai/gpt-6-luna`, model lớn `openai/gpt-6.1-terra`.

## 1. Bằng chứng

| Tệp | Trạng thái |
| --- | --- |
| [day01_lab.ipynb](day01_lab.ipynb) | Có notebook và output; báo cáo cuối ghi 15/15 kiểm tra đạt, đủ 10 dự đoán |
| [lab1_report.json](lab1_report.json) | Có SEED, kiểm tra, dự đoán và kết quả các model |
| [hand_calc.jpg](hand_calc.jpg) | Ảnh viết tay gồm năm phép tính |

Năm giá trị tính tay đã qua kiểm tra: baseline accuracy 0,87; sigmoid(0) 0,50; F1 ví dụ 0,73; cosine ví dụ 0,60; chi phí theo giả định đề 10,50 USD/ngày.

## 2. Kết quả chính

| Phương án | F1 test | F1 SUB 40 | Train (s) | Tin/s cả lô | p95 (ms/tin) | USD/1.000 tin |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| BoW + LogReg | 0,897 | 0,919 | 1,702 | 34.411,762 | 0,075 | 0 |
| BoW + MLP | 0,943 | 0,947 | 3,484 | 51.516,095 | 0,207 | 0 |
| Encoder + kNN | 0,907 | 0,974 | 36,266 | 52,699 | 128,830 | 0 |
| Encoder + LogReg | 0,956 | 1,000 | 36,580 | 64,607 | 29,178 | 0 |
| Encoder + MLP head | 0,956 | 0,974 | 37,250 | 65,202 | 22,906 | 0 |
| LLM nhỏ zero-shot | — | 0,950 | 0 | 0,988 | 1.480,001 | 0,030 |
| LLM lớn zero-shot | — | 0,974 | 0 | 0,724 | 1.642,484 | 0,303 |
| LLM nhỏ few-shot | — | 0,889 | 0 | 0,881 | 2.086,627 | 0,111 |

LLM chỉ chạy trên SUB, chưa có F1 toàn bộ test. So sánh chất lượng giữa LLM và các phương án khác bằng F1 SUB. Chi phí là ước tính theo quy tắc token và bảng giá notebook; 0 ở model local nghĩa là không tính phí API, chưa gồm hạ tầng.

MLP tăng F1 test khoảng 4,6 điểm phần trăm so với BoW + LogReg. Validation loss thấp nhất tại chỉ số epoch 4, sau đó tăng trong khi train loss giảm, cho thấy overfitting. Trên embedding, thêm MLP head chưa tăng F1 test rõ rệt ở mức làm tròn hiển thị.

## 3. Dự đoán và thực tế

1. **MLP so với LogReg:** Tôi dự đoán MLP không tốt hơn hoặc có F1 gần LogReg. Thực tế F1 tăng từ 0,897 lên 0,943. Mạng có thể học thêm quan hệ phi tuyến giữa các đặc trưng BoW, dù vẫn có dấu hiệu overfit.
2. **Sáu tin khó:** Tôi dự đoán BoW đúng 3–4/6, embedding đúng 4–5/6. Thực tế cả hai đúng 5/6 nhưng sai ở các tin khác nhau. Có embedding tốt hơn không có nghĩa bộ phân loại luôn tốt hơn trên mọi tin.
3. **Few-shot:** Tôi dự đoán F1 giữ nguyên hoặc tăng nhẹ và chi phí tăng 2–3 lần. Thực tế F1 giảm từ 0,950 xuống 0,889, chi phí 40 tin tăng từ khoảng 0,00121 lên 0,00445 USD, khoảng 3,7 lần, và có hai nhãn không hợp lệ. Ví dụ có thể chưa đại diện, prompt chưa nhấn mạnh trả đúng một từ; chưa đủ bằng chứng xác định nguyên nhân duy nhất.

## 4. Phân tích 50 case khó và năm case bonus

Bảng dưới là kết quả lần chạy 50 tin gốc, trước khi thêm bonus.

| Phương án | F1 SUB thường | F1 bộ khó | Accuracy bộ khó |
| --- | ---: | ---: | ---: |
| BoW + LogReg | 0,919 | 0,278 | 48% |
| BoW + MLP | 0,947 | 0,359 | 50% |
| Encoder + kNN | 0,974 | 0,759 | 72% |
| Encoder + LogReg | 1,000 | 0,692 | 68% |
| Encoder + MLP head | 0,974 | 0,640 | 64% |
| LLM nhỏ zero-shot | 0,950 | 0,966 | 96% |
| LLM nhỏ + prompt phòng thủ | Chưa đo riêng | 1,000 | 100% |

BoW + LogReg tụt nhiều nhất, khoảng 64,1 điểm phần trăm F1. Test thường cùng phân phối với train và còn các mẫu gần trùng nên có thể lạc quan. Hai ví dụ cụ thể:

- **Case 11, spam giọng tự nhiên:** mở đầu “Hey it's Jess from the gym” và mời đăng ký ưu đãi. BoW + LogReg sai case này vì sai toàn bộ 10 tin trong nhóm. Giọng thân mật và cách quảng cáo khác dữ liệu quen thuộc có thể khiến mô hình bỏ sót ý định spam.
- **Case 29, né từ khóa:** mở đầu “C0ngr4tulat10ns! Y0u h4ve w0n”. BoW + MLP sai case này vì sai toàn bộ sáu tin trong nhóm. Thay chữ bằng số tạo token khác từ quen thuộc; MLP vẫn dùng BoW nên không tự khắc phục được hạn chế đó.

Encoder + kNN khó nhất với nhóm nói về spam (25% đúng); hai head LogReg/MLP khó nhất với spam giọng tự nhiên (40%/30%). LLM thường và phòng thủ đều đúng 100% nhóm prompt injection. Prompt phòng thủ nâng hai nhóm ham trông giống spam và spam giọng tự nhiên từ 90% lên 100%, không làm giảm nhóm nào trong lần chạy này.

Có thể chuẩn hóa ký tự ngụy trang trước BoW nhưng phải kiểm tra nguy cơ biến dạng ham. Bộ khó dùng phát hiện điểm yếu và kiểm tra hồi quy, không đại diện đầy đủ cho tin nhắn thực tế.

**Kết quả sau bonus:** File `notebook9beff1bee7.ipynb` đã lưu lần đánh giá 55 tin (50 tin gốc và năm tin bổ sung):

| Phương án | F1 bộ 55 tin | Accuracy bộ 55 tin |
| --- | ---: | ---: |
| BoW + LogReg | 0,244 | 43,6% |
| BoW + MLP | 0,326 | 47,3% |
| Encoder + kNN | 0,758 | 70,9% |
| Encoder + LogReg | 0,712 | 69,1% |
| Encoder + MLP head | 0,643 | 63,6% |
| LLM nhỏ zero-shot | 1,000 | 100% |
| LLM nhỏ + prompt phòng thủ | 1,000 | 100% |

BoW + LogReg vẫn tụt nhiều nhất, từ F1 SUB 0,919 xuống 0,244, khoảng 67,5 điểm phần trăm. Hai phương án LLM đều đúng 55/55 nên năm case thêm chưa đánh lừa được model tốt nhất; chưa thấy lợi ích bổ sung của prompt phòng thủ trong lần chạy này. Khác biệt với lần chạy 50 tin có thể gồm cả biến thiên giữa các lần gọi API, không thể quy toàn bộ cho năm tin mới.

Năm case có tham khảo AI, không phải hoàn toàn tự nghĩ. Các bẫy lần lượt là ham chứa từ khóa nhận thưởng, lời mời kiếm tiền yêu cầu đóng phí, mạo danh quản lý, ký tự ngụy trang và lệnh đổi nhãn chèn trong tin.

## 5. Thử thách cá nhân

Đề số 4: chạy kNN trên cùng embedding với k = 1, 5, 25, so sánh F1.

| k | F1 test |
| ---: | ---: |
| 1 | 0,9263 |
| 5 | 0,9072 |
| 25 | 0,8731 |

k=1 tốt nhất trong ba giá trị đã thử. k nhỏ có nguy cơ nhạy với nhiễu, nhưng không có nghĩa luôn cho kết quả kém. k lớn có thể đưa thêm hàng xóm ít liên quan vào bỏ phiếu và làm lệch về lớp ham chiếm đa số. Đây là so sánh theo yêu cầu bài; khi phát triển thực tế cần chọn k trên validation thay vì test. Code vòng lặp có tham khảo AI, như khai báo bên dưới.

## 6. Phương án cho sản phẩm thật

Bốn phương án đạt p95 < 50 ms là BoW + LogReg, BoW + MLP, Encoder + LogReg và Encoder + MLP head. Encoder + LogReg là ứng viên về chất lượng với F1 test 0,956 và p95 29,178 ms. Tuy nhiên, thông lượng khoảng 65 tin/s chưa đáp ứng trung bình khoảng 116 tin/s của 10 triệu tin/ngày; cần tối ưu hoặc chạy song song và đo tải thực tế. F1 của Encoder + LogReg là 0,692 trên bộ 50 tin và 0,712 trên bộ 55 tin, cho thấy chưa đủ bằng chứng để triển khai ngay.

Theo chi phí làm tròn trong bảng, xử lý toàn bộ 10 triệu tin/ngày trong 30 ngày bằng LLM tốn khoảng 9.000 USD/tháng với model nhỏ zero-shot, 90.900 USD với model lớn và 33.300 USD với model nhỏ few-shot. Chưa gồm hạ tầng; các p95 LLM đều vượt SLA hiện tại.

Có thể dùng model local cho tin chắc chắn và chuyển tin không chắc sang LLM có prompt phòng thủ. Cần đánh giá cả lỗi model local tự tin đoán sai, đồng thời đo lại p95 và chi phí toàn hệ thống. Router không tự bảo đảm đạt SLA vì nhánh gọi LLM vẫn chậm.

## 7. AI log và điều đã học

Bảng dưới tổng hợp các nhóm trao đổi với AI trong quá trình làm bài, gồm cả code và nội dung báo cáo có tham khảo; các phần này không phải hoàn toàn tự làm.

| Phần | Hỏi AI gì | AI cung cấp gì | Kết quả kiểm chứng |
| --- | --- | --- | --- |
| Quy trình và dự đoán | Cần làm gì, cách chạy và ước lượng `predict()` | Hướng dẫn đọc dữ liệu, giả thuyết tham khảo | Notebook có 10 dự đoán và kết quả để đối chiếu |
| TODO 1–3 | Nhớ lại Syntax | Code syntax | Các ô kiểm tra lưu trong notebook đạt |
| Tải encoder | Lỗi Hugging Face và token hết hạn | Phân tích traceback, tắt token ngầm và tăng timeout | Notebook có kết quả encode và đánh giá embedding |
| Markdown | Giải thích các thí nghiệm từ output | Bản nháp câu trả lời theo số liệu | Giải thích năm dòng train và đối chiếu kết quả thí nghiệm |
| Bonus | Gợi ý năm tin khó | Năm case và nhãn tham khảo | Đánh giá 55 tin, hai LLM đúng toàn bộ |


Điều rút ra: accuracy cao có thể che giấu recall spam thấp. Embedding, hidden layer hoặc few-shot không tự bảo đảm chất lượng tăng; cần so sánh trên cùng dữ liệu và xem lỗi cụ thể. Chất lượng trên test thường cũng chưa đủ để bảo đảm độ bền, thông lượng và độ trễ khi triển khai.

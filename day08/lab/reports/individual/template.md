# Báo Cáo Cá Nhân — Lab Day 08: RAG Pipeline

**Họ và tên:** ____________________  
**Vai trò trong nhóm:** Tech Lead  
**Ngày nộp:** 2026-04-13  
**Độ dài yêu cầu:** 500–800 từ

---

## 1. Tôi đã làm gì trong lab này? (100-150 từ)

Trong vai trò Tech Lead, tôi tập trung vào việc giữ nhịp 4 sprint và đảm bảo pipeline chạy end-to-end thay vì tối ưu cục bộ từng phần. Ở Sprint 1–2, tôi chốt cấu trúc xử lý chính: `index.py` cho preprocess/chunk/embed/store và `rag_answer.py` cho flow retrieve -> generate -> trả về sources. Ở Sprint 3, tôi phối hợp với Retrieval Owner để thử variant `hybrid + rerank`, đồng thời giữ nguyên baseline dense làm mốc so sánh. Tôi cũng chủ động chuẩn hóa interface giữa các file để Eval Owner có thể gọi `rag_answer()` bằng config khác nhau mà không phải chỉnh code nhiều chỗ. Ở Sprint 4, tôi review scorecard, đối chiếu các case fail (đặc biệt q10), rồi điều phối việc chỉnh prompt/retrieval theo nguyên tắc “thay đổi nhỏ, đo lại ngay” để tránh overfit.

---

## 2. Điều tôi hiểu rõ hơn sau lab này (100-150 từ)

Sau lab này, tôi hiểu rõ hơn hai điểm. Thứ nhất là “grounded answer” không chỉ là thêm câu “answer from context”, mà là thiết kế rule ra quyết định rõ: khi nào trả lời trực tiếp, khi nào suy luận theo policy chung, khi nào mới được abstain. Nếu rule này mơ hồ, model dễ từ chối quá mức hoặc trả lời lan man dù retrieval đúng. Thứ hai là evaluation loop quan trọng hơn cảm giác chủ quan khi đọc vài câu trả lời. Trước đây tôi nghĩ hybrid + rerank gần như luôn tốt hơn dense, nhưng scorecard cho thấy variant có thể kém baseline nếu rerank làm lệch ngữ cảnh. Vì vậy, bài học lớn là phải giữ baseline mạnh, đo bằng cùng bộ test, và đọc per-question thay vì chỉ nhìn average tổng.

---

## 3. Điều tôi ngạc nhiên hoặc gặp khó khăn (100-150 từ)

Điều làm tôi bất ngờ nhất là Context Recall gần như luôn cao, nhưng chất lượng answer vẫn không ổn ở một số câu hard. Ban đầu tôi giả thuyết lỗi chính nằm ở retrieval (không lấy đúng tài liệu), nhưng khi đối chiếu scorecard thì nhiều câu đã retrieve đúng source rồi mà answer vẫn thiếu trọng tâm hoặc abstain sai. Ca mất thời gian nhất là q10: baseline còn trả lời được theo quy trình chuẩn, trong khi variant `hybrid + rerank` lại trả “không đủ dữ liệu”. Điều này cho thấy rerank không chỉ “lọc noise” mà còn có thể làm mất chunk quan trọng cho generation. Khó khăn thực tế của vai trò Tech Lead là cân bằng giữa việc “đẩy nhanh tuning” và việc giữ thí nghiệm sạch để biết biến nào gây ra regression.

---

## 4. Phân tích một câu hỏi trong scorecard (150-200 từ)

**Câu hỏi:** q10 — “Nếu cần hoàn tiền khẩn cấp cho khách hàng VIP, quy trình có khác không?”

**Phân tích:**

Đây là câu phản ánh rõ nhất ranh giới giữa “thiếu dữ liệu thật” và “không có case riêng nhưng vẫn có policy chung”. Với baseline dense, hệ thống trả lời theo hướng: tài liệu không nêu quy trình VIP riêng, nhưng có quy trình hoàn tiền chuẩn (ticket -> CS review -> Finance xử lý 3-5 ngày). Điểm baseline cho câu này ở mức trung bình (Faithfulness/Relevance chưa cao tuyệt đối) nhưng vẫn hữu ích cho người dùng. Ngược lại, variant `hybrid + rerank` trong lần chạy chính bị rơi về mức rất thấp vì trả lời “Không đủ dữ liệu...”, tức là bỏ qua policy chung đang có trong context.

Tôi đánh giá lỗi chính nằm ở tầng retrieval-selection + generation policy, không phải indexing. Index vẫn chứa đúng nội dung refund policy; vấn đề là sau khi rerank, tín hiệu policy chuẩn không còn được model dùng hiệu quả, và prompt cũ cho phép abstain quá dễ. Vì vậy variant không cải thiện mà còn regression. Hướng xử lý phù hợp là làm rerank an toàn hơn (trộn điểm retrieval gốc, hạn chế lật kèo cực đoan) và siết rule generation để ưu tiên “áp dụng quy trình chung” trước khi abstain.

---

## 5. Nếu có thêm thời gian, tôi sẽ làm gì? (50-100 từ)

Tôi sẽ thử hai cải tiến cụ thể. Một là thay cross-encoder rerank hiện tại bằng model multilingual hoặc bật cơ chế fallback về thứ tự retrieval gốc khi rerank confidence thấp, vì kết quả eval cho thấy variant đang tụt ở câu tiếng Việt khó. Hai là thêm một lớp answer post-check đơn giản: nếu câu trả lời là abstain nhưng context có policy chung liên quan thì buộc regenerate theo policy đó. Mục tiêu là giảm false abstain như q10 mà không làm tăng hallucination.

---

*Lưu file này với tên: `reports/individual/[ten_ban].md`*
*Ví dụ: `reports/individual/nguyen_van_a.md`*

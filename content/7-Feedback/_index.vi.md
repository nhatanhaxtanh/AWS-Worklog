---
title: "Chia sẻ, đóng góp ý kiến"
date:
weight: 7
chapter: false
pre: " <b> 7. </b> "
---

> Tôi viết phần này để ghi lại trải nghiệm của riêng mình với chương trình First Cloud Journey (FCJ) và gửi gắm vài góp ý để các bạn thực tập sau có thể tận dụng cơ hội tốt hơn nữa.

---

### Đánh Giá Chung

**1. Môi trường làm việc**  
AWS mang đến đúng kiểu môi trường tôi nghĩ chỉ có sau nhiều năm làm việc: văn phòng tiện nghi, phòng tập trung yên tĩnh và hệ thống công cụ hoạt động trơn tru. Điều bất ngờ nhất là **sự gần gũi** của mọi người xung quanh. Các anh chị kỹ sư senior luôn sẵn sàng dừng lại để giải thích bối cảnh, còn quản lý thì chủ động bảo tôi nhắn tin khi gặp bế tắc. Mô hình hybrid giúp tôi linh hoạt: những ngày cần tập trung thì làm remote, còn khi cần brainstorm thì lên văn phòng.

Vì FCJ xem sự tò mò là điểm mạnh, tôi tự tin đặt nhiều câu hỏi hơn bình thường. Với sandbox account và credits sẵn có, tôi dựng hạ tầng thực, phá hỏng nó rồi sửa lại dưới sự hướng dẫn—điều mà sách vở không thể mang lại.

**2. Sự hỗ trợ của Mentor / Team Admin**  
Mentor của tôi đóng vai trò huấn luyện thay vì chỉ đưa đáp án. Khi mang đến một blocker, tôi luôn được gợi mở đọc whitepaper, thử công cụ hoặc phân tích log để tự rút ra lời giải. Những điểm nổi bật:

- **1:1 hàng tuần** để demo tiến độ, retros các sai sót và lên kế hoạch cho rủi ro sắp tới
- **Code review kỹ lưỡng** nhấn mạnh khả năng đọc, test và vận hành của mỗi commit
- **Phiên đào sâu kiến trúc** để tôi hiểu các đánh đổi chứ không chỉ nhìn sơ đồ cuối cùng
- **Định hướng nghề nghiệp** xoay quanh certifications, danh mục dự án và lộ trình sau khi tốt nghiệp

Song song, team admin lo trọn hậu cần—from quyền IAM đến hỗ trợ thiết bị—trước khi tôi kịp nhận ra vấn đề. Nhờ vậy tôi có thể dồn toàn bộ thời gian cho việc ship tính năng.

**3. Sự phù hợp giữa công việc và chuyên ngành học**  
Dự án Bandup IELTS vừa bám sát nền tảng khoa học máy tính vừa kéo tôi sang địa hạt cloud thực chiến:

| Kiến Thức Học Thuật Áp Dụng | Kỹ Năng Mới Phát Triển |
|----------------------------|-----------------------|
| Lập trình Python | AWS Lambda & dàn nhạc serverless |
| Cấu trúc dữ liệu & thuật toán | RAG pipelines & vector search |
| Kiến thức cơ sở dữ liệu | DynamoDB, ElastiCache, mô hình dữ liệu |
| Kiến thức mạng máy tính | Thiết kế VPC, phân subnet, kiểm soát bảo mật |
| Kỹ thuật phần mềm | Tự động hóa CI/CD, Infrastructure as Code |

Bài tập trên lớp hiếm khi đòi hỏi SLA hay tích hợp AI thật sự. Việc tự tay xây dựng luồng audio với Gemini và embedding với Titan giúp tôi thấm rõ những yếu tố như độ trễ, thông lượng và chi phí.

**4. Cơ hội học hỏi & phát triển kỹ năng**  
FCJ kết hợp giữa khung học tập có định hướng và sự tự chủ thực chiến. Trong 12 tuần tôi đã:

- **Làm việc với hơn 15 dịch vụ AWS** thuộc compute, integration, data và AI (Lambda, API Gateway, SQS, DynamoDB, S3, ECS, Fargate, Bedrock, ...)
- **Sở hữu trọn vòng đời tính năng**: từ định nghĩa scope, viết code, test đến triển khai 4 Lambda functions
- **Rèn kỹ năng tối ưu chi phí**, chứng minh được mức tiết kiệm 72% nhờ quy trình audio native của Gemini
- **Ghi chép đầy đủ** thông qua bộ workshop này để người đi sau có lộ trình rõ ràng
- **Thử nghiệm tích hợp AI/ML** mang lại giá trị ngay cho người học

Những buổi checkpoint với mentor giữ cho tôi không đi chệch hướng, đồng thời vẫn đủ tự do để thử nghiệm và học từ sai sót.

**5. Văn hóa & tinh thần đồng đội**  
Mỗi Leadership Principle của Amazon mà tôi từng đọc đều xuất hiện trong đời sống hằng ngày. Ví dụ:

- Đồng đội đầu tư thời gian kèm tôi hoàn thiện demo dù đã tối muộn
- Khi gây ra sự cố, cả nhóm làm post-mortem không đổ lỗi mà rút kinh nghiệm
- Đề xuất về luồng audio native với Gemini không chỉ được lắng nghe mà còn trở thành chuẩn mặc định
- Tinh thần “Day 1” không chỉ là khẩu hiệu; mọi người liên tục hỏi làm sao để Bandup phục vụ học viên tốt hơn

Cảm giác được góp sức thực sự chứ không phải chỉ quan sát khiến tôi gắn bó ngay từ tuần đầu.

**6. Chính sách / phúc lợi cho thực tập sinh**  
FCJ thực sự đầu tư cho thực tập sinh:

- ✅ Phụ cấp xứng đáng với giá trị công việc
- ✅ Giờ giấc linh hoạt giúp cân bằng lịch học đại học
- ✅ Truy cập không giới hạn Skill Builder, whitepaper và tech talk nội bộ
- ✅ Kết nối với kỹ sư từ nhiều org khác nhau để trao đổi nghề nghiệp
- ✅ Sản phẩm thực tế đủ sức đưa thẳng vào portfolio

---

### Câu Hỏi Phản Ánh

**Điều khiến tôi hài lòng nhất trong kỳ thực tập?**

Được ship tính năng mà người học thực sự sử dụng. Khoảnh khắc Speaking Evaluator nuốt file audio, xử lý qua Gemini rồi trả về điểm band thông qua Lambda pipeline do tôi xây dựng khiến tôi nhận ra: code của mình đang giúp ai đó học tốt hơn.

**Điều gì có thể cải thiện cho thế hệ thực tập tiếp theo?**

- **Cấp quyền AWS sớm hơn** để mọi người lao vào xây dựng ngay ngày đầu
- **Bộ onboarding có cấu trúc** gồm kiến thức AWS nền tảng, cách cấu hình CLI và các lưu ý bảo mật
- **Ghép cặp giữa các thực tập sinh** trên cùng sáng kiến để học hỏi lẫn nhau
- **Cơ hội luân chuyển/shadow** các team AWS khác để hiểu bức tranh rộng hơn

**Tôi có giới thiệu chương trình này cho bạn bè không?**

Chắc chắn. Nếu thích cloud engineering, FCJ mang lại:
- Quyền truy cập trực tiếp vào dịch vụ AWS và quy trình production
- Mentor tận tâm đồng hành
- Quyền sở hữu đối với các deliverable quan trọng
- Thành quả rõ ràng để trình bày với nhà tuyển dụng

Bạn chỉ cần tinh thần tò mò và sự nỗ lực nghiêm túc.

---

### Đề Xuất & Kỳ Vọng Tương Lai

**Một vài ý tưởng để FCJ tiếp tục nâng tầm:**

1. **Playbook tập trung cho thực tập sinh** với hướng dẫn setup môi trường, tip xử lý sự cố và mẫu kiến trúc
2. **Buổi show-and-tell cách tuần** để thực tập sinh demo tiến độ, nhận góp ý và luyện kỹ năng trình bày
3. **Khuyến khích thi chứng chỉ** như voucher hoặc nhóm học Cloud Practitioner/Solutions Architect
4. **Nhóm alumni FCJ** kết nối người đi trước với thế hệ hiện tại để chia sẻ kinh nghiệm

**Tôi có muốn tiếp tục đồng hành?**

Tất nhiên. Tôi đặt mục tiêu:
- Quay lại sau khi tốt nghiệp với vai trò cloud engineer hoặc solutions architect
- Làm mentor cho khóa FCJ kế tiếp để truyền lửa và kinh nghiệm
- Cập nhật bộ workshop này giúp đàn em onboard nhanh hơn

**Lời kết**

FCJ đã tái định hình cách tôi nghĩ về việc xây dựng trên nền tảng đám mây. Tôi bước vào với kiến thức sách vở và sự tò mò; tôi rời đi với quy trình làm việc đã được kiểm chứng, kỹ năng giao tiếp tốt hơn và sự tự tin khi thiết kế hệ thống an toàn, mở rộng được. Cảm ơn tất cả mọi người tại AWS đã đầu tư cho hành trình của tôi. 🙏

---

*"Sự trưởng thành đến khi tò mò gặp gỡ tinh thần trách nhiệm. FCJ chính là điểm giao đó đối với tôi."*

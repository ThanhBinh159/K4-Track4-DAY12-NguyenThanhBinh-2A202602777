# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Nguyễn Thanh Bình |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | 2A202602777 |
| GitHub | [ThanhBinh159/K4-Track4-DAY12-NguyenThanhBinh-2A202602777](https://github.com/ThanhBinh159/K4-Track4-DAY12-NguyenThanhBinh-2A202602777) |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | 6520 / 3635–3639 |
| Số seed huấn luyện bạn dùng (`--seeds`) | 5 |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,259 | 20,4 | Chặn |
| IDM | 1,532 | 0,1 | Baseline |
| EgoMLP | 0,762 | 15,0–37,6 | Chặn |
| PercMLP | 0,622 | 0,5–2,3 | Chặn |
| PercMLP+ (sau khi sửa) | 0,827 | 0,2–0,3 | Qua cổng |

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

`Acc_LV` gần như không khớp đạo hàm của `Speed_LV` (tương quan 0,045), trong khi cột gia tốc xe theo sau đạt 0,929; tôi dùng đạo hàm của tốc độ xe dẫn đã làm mượt để tìm ca phanh.
Dữ liệu có 9.993 quỹ đạo gốc, trong đó 383 quỹ đạo trùng hệt nhau đã bị loại; còn 9.610 quỹ đạo.
Nếu bản trùng rơi vào cả train và test, kết quả kiểm tra sẽ lạc quan giả tạo.
Tôi chia theo quỹ đạo, vì các khung hình liền nhau của cùng một chuyến lái gần như giống nhau và sẽ gây rò rỉ nếu chia theo khung.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Refactor float32 khác từng bit ở cả 600/600 đoạn nhưng 0 đoạn vượt dung sai `atol=rtol=0,001` (sai khác lớn nhất 0,000097 m/s²); chạy lại cùng mã thì 0 đoạn khác.
Quy tắc AEB mới làm lệch 11/600 đoạn. Ở đoạn 59, các khung 98–101 có TTC 1,43–1,49 s, dưới ngưỡng 1,5 s; lệnh cũ khoảng −5,44 đến −5,91 m/s² được ép xuống −6,0 m/s².
Vì thay đổi thứ tự phép tính và độ chính xác số thực gây sai khác rất nhỏ, so theo dung sai hợp lý hơn so từng bit.
Phát nhanh nhất có 0 khung trễ/rớt; phát theo thời gian thật có 581 trễ và 576 rớt trên 59.497 khung, làm thay đổi 373/600 đoạn vì thời hạn xử lý có thật.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới có tốc độ cruise 14 m/s, khoảng cách 2,76–59,31 m và 60 × 45 = 2.700 ô.
Biên vật lý cho thấy 3,63% ô bất khả thi khi trễ 0 s và 10,22% khi trễ 0,6 s; mô phỏng phanh tối đa lệch 0 ô ngoài biên ở cả hai mức trễ.
Vùng va chạm của IDM tăng đúng từ 3,63% lên 10,22% (thêm 6,59 điểm phần trăm), trùng vùng bất khả thi; không nên quy các ô đó thành lỗi của bộ điều khiển.
Có 614 ca xe dẫn phanh được khai thác, nhưng mức phanh mạnh ở phân vị 99,9% của người lái trong log chỉ khoảng 3,6 m/s²; 12,67% lưới đòi hỏi phanh mạnh hơn mức này, nên log không phủ đủ vùng khó.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

| Bộ điều khiển | L2 (m) | Hạng L2 | Va chạm TB (%) | Hạng va chạm | Dải qua seed (%) |
|---|---:|---:|---:|---:|---:|
| constant-velocity | 1,259 | 3 | 20,4 | 3 | 20,4 |
| IDM | 1,532 | 4 | 0,1 | 1 | 0,1 |
| EgoMLP | 0,762 | 2 | 22,2 | 4 | 15,0–37,6 |
| PercMLP | 0,622 | 1 | 1,0 | 2 | 0,5–2,3 |

Các cặp đảo hạng: constant-velocity/IDM, constant-velocity/EgoMLP, IDM/EgoMLP, IDM/PercMLP.
IDM không bắt chước chính xác người lái nên L2 cao, nhưng tự điều chỉnh khoảng cách khi chạy vòng kín. EgoMLP thiếu khoảng cách và tốc độ xe dẫn, nên lỗi tích lũy và nó không phản ứng kịp lúc xe dẫn phanh.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

Trên 2.883 quỹ đạo kiểm tra, EgoMLP dao động từ 15,0% đến 37,6% va chạm qua 5 seed, chênh 22,5 điểm phần trăm.
Seed tốt nhất 3636 có 433/2.883 va chạm, khoảng Wilson 95% là 13,8–16,4%; seed xấu nhất 3639 có 1.083/2.883, khoảng 35,8–39,3%.
Hai khoảng không chồng nhau; dao động này không thể giải thích đơn thuần bằng sai số lấy mẫu trên cùng tập kiểm tra.
Nếu chỉ báo seed 3636 hoặc 3639, tôi có thể đưa ra hai bức tranh rất khác về cùng kiến trúc; vì vậy cần báo dải qua seed và dùng seed xấu nhất ở cổng.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Tỷ lệ ô có **bất kỳ va chạm** nào trên lưới độ trễ 0,6 s:

| Bộ điều khiển | Xe sau phát lại (%) | Xe sau phản ứng (%) |
|---|---:|---:|
| constant-velocity | 80,6 | 80,6 |
| IDM | 80,9 | 10,2 |
| EgoMLP | 82,8 | 82,8 |
| PercMLP | 78,4 | 36,1 |

Xe sau IDM được hiệu chỉnh với `T=1,4 s`: Wasserstein headway 0,35 so với 0,87 cho xe không phản ứng; headway trung vị log là 1,85 s. Xe sau chỉ phát lại gây 70,6% va chạm phía sau giả cho IDM, khi phản ứng còn 0%; thứ hạng IDM/constant-velocity đổi từ 80,9/80,6% sang 10,2/80,6%.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP rớt R2, R3, R4, R5 và R6: seed xấu nhất va chạm 2,3% trên log (cận trên 95% là 2,9%), 9,0% trên lát xe dẫn phanh; dải qua seed là 0,5–2,3%.
Trên lưới cắt vào trễ 0,6 s, nó đâm ở 25,9% ô còn tránh được (IDM 0%) và có 36,1% ô va chạm trong thế giới xe sau phản ứng (IDM 10,2%). Log thiếu các ca cắt vào khó, còn sai số bắt chước tích lũy khi mô hình tự lái.
Tôi thêm 197.020 cửa sổ từ 20.000 ca cắt vào với nhãn IDM, rồi làm 2 vòng DAgger để học cả những trạng thái mô hình tự đi vào. PercMLP+ còn 0,2–0,3% va chạm trên log, 0,3% ô va chạm có thể tránh và qua cả 7 quy tắc.
L2 vòng hở tăng từ 0,622 lên 0,827 m dù an toàn vòng kín cải thiện; điểm bắt chước log không đủ để quyết định phát hành.
Qua cổng chỉ có nghĩa là không kém IDM theo các biên đã đặt: IDM vẫn đạt 0,1% va chạm log và 0% ô va chạm có thể tránh. Ở tốc độ ngoài tập huấn luyện 20 m/s, PercMLP+ còn 0,3–1,2% ô như vậy, IDM 0%.
Vì PercMLP+ học nhãn từ IDM và chưa chứng minh tốt hơn thầy, tôi chưa thay IDM trong sản phẩm; trước tiên cần kiểm thử đa tác nhân, cảm biến/phần cứng thật và chạy giám sát có cổng dừng rõ ràng.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Mô phỏng chỉ theo một chiều dọc, nên không kiểm tra được quỹ đạo ngang, thời điểm xe thực sự cắt làn hay các tác nhân phức tạp; lưới 2.700 ô không đại diện cho mọi cảnh giao thông.
Xe dẫn trong log không phản ứng với xe thử; kết luận cũng rất nhạy với cách mô hình hóa xe sau: tỷ lệ va chạm của IDM đổi từ 80,9% khi xe sau phát lại xuống 10,2% khi xe sau phản ứng.
Bài lab không đo cảm biến, che khuất, lỗi nhận dạng hay trễ xử lý trên phần cứng thật; thử nhiễu khoảng cách 1 m, tốc độ 0,5 m/s và trễ 0,6 s mới chỉ chạm một phần vấn đề.
Log chỉ gồm 9.610 quỹ đạo sạch và 12,67% lưới đòi hỏi phanh mạnh hơn mức phân vị 99,9% trong log; ở tốc độ 20 m/s ngoài tập huấn luyện, PercMLP+ vẫn đâm ở 0,3–1,2% ô còn tránh được.
Trước khi phát hành cần kiểm thử thêm tình huống đa làn và tác nhân tương tác, đo trễ/cảm biến trên phần cứng thật, rồi đánh giá qua nhiều chuyến chạy giám sát với cùng cổng và dải bất định.

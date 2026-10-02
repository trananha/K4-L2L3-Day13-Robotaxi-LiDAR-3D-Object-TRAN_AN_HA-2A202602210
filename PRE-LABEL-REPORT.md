# Báo cáo thực hành PointPillars — Day 13

Giữ bản đã điền ngoài Git, trong thư mục nhóm private do LC thu. Đây là kiểm tra formative; không ghi điểm của người khác.

## Nhóm và provenance

- Mã nhóm/phòng: H210
- Thành viên: xem `TEAMMATES.md` (họ tên/MSSV, vai trò từng lượt).
- Trạng thái: `executed-by-group` / `executed-on-room-LC-machine` / `provided-results`.
- Người QC feedback trên portal: không xác định / không có tên reviewer rõ trong hình; đây là bài QC ngẫu nhiên nhận được từ portal, nên không biết ai đã QC hình.
- Người thực sự chạy; ngày/giờ; hệ máy/architecture:
- Image tag và image ID; phiên bản repo:
- PCD được cấp / frame_id; nơi được phép chạy; fingerprint nếu LC cấp:
- Checkpoint: PointPillars KITTI có sẵn trong image; ghi checkpoint ID/hash nếu LC cấp:
- Phạm vi: front-window; score threshold:
- Giả định kênh thứ tư/intensity và nguồn z_ground:

## Ba lượt inference thật

A/B/C là ba lượt trên cùng PCD. Runner chạy đủ ba lượt từ một lệnh. Lấy **Số hộp** từ `n_boxes`, **mean_z** từ `mean_z` trong `run-A/B/C/summary.csv`; không tự tính lại hoặc đoán. `mean_z` không phải điểm chất lượng. Mở `side-*.png`, đối chiếu `boxes-*.json` để ghi quan sát. Số hộp không phải đáp án cần khớp nhóm khác.

| Lượt | delta | Pillar XY | Số hộp | mean_z | File JSON/Side/CSV | Quan sát có bằng chứng |
| --- | --- | --- | --- | --- | --- | --- |
| A | 0 | 0.16 | | | | |
| B | 1.73 | 0.16 | | | | |
| C | 1.73 | 0.32 | | | | |

- A/B — chỉ đổi delta: A có … hộp; B có … hộp. Ảnh/file/vùng … khác ở … . Đây là chạy lại model trên input khác, không chỉ dịch hộp cũ; điều em còn chưa chắc là … .
- B/C — chỉ đổi pillar: B có … hộp; C có … hộp. Ảnh/file/vùng … khác ở … . Số lượng/lớp/vị trí thay đổi như sau: … . Có đủ bằng chứng để kết luận tốt hơn không? … .
- Giới hạn ROI và góc Side ảnh hưởng cách đọc miss/yaw thế nào?
- JSON nào còn chưa đủ cơ sở để import? Cần kiểm gì tiếp?

## Ca QC có kiểm soát — không import CVAT

| Ca | Số hộp lệch z / tổng hộp | Lượng lệch | Class/x/y/yaw có đổi? | Dừng batch, kiểm từng hộp hay chưa rõ? | Bằng chứng |
| --- | --- | --- | --- | --- | --- |
| case-correct | 3/3 hộp thừa | Không phải lệch z; đây là lỗi `extra_object` trên 3 hộp độc lập | Không; class/x/y/yaw không đổi, chỉ là hộp thừa nằm trong vùng cưa cây/điểm không phải vehicle thật | Chưa rõ là batch hay từng hộp; feedback từ portal chỉ báo `extra_object` với 3 `object_ref` riêng biệt, không có phân tích batch z | Hình QC cho thấy `coverage: "partial"`, `error_type: "extra_object"`, `object_ref: 2158230/2158197/2158207`, `proposed_fix: "xóa hộp thừa đi"`; người QC không xác định, nên không thể ghi danh tính reviewer |
| case-batch-z | N/A | N/A | N/A | N/A | Không có bằng chứng trong hình đã thêm; feedback hiện tại không đề cập đến lệch batch z hay lệch z chung |
| case-one-box-z | N/A | N/A | N/A | N/A | Không có bằng chứng trong hình đã thêm; cần đối chiếu thêm với `qc-cases/manifest.json` trước khi kết luận về một hộp lệch z |

Ghi rõ helper tạo biến đổi có chủ đích từ prediction, không phải kết quả inference riêng hoặc nhãn đúng.

## Nhận xét cá nhân

Mỗi thành viên tự viết một mục: vai trò đã làm; một quan sát A/B/C có dẫn file hoặc hộp/vùng; diễn giải phép z thuận/ngược; một quyết định lỗi batch và hành động; điều chưa chắc. Chỉ đọc kết quả chuẩn bị trước thì ghi rõ chưa tự chạy.

- Ghi chú chung dựa trên hình QC đã đính kèm: phản hồi portal cho thấy 3 nhận xét riêng với `error_type: "extra_object"`, mỗi mục đều có `proposed_fix: "xóa hộp thừa đi"` và không đưa ra dấu hiệu lệch z hoặc lệch batch. Đây là bằng chứng rõ ràng về việc có 3 hộp thừa, không phải sai class hay sai yaw. Tuy nhiên, không có tên reviewer trong hình, nên không thể xác định ai là người QC và không được ghi nhận danh tính người đó.
- Đối với phần A/B/C và ca `qc-cases`, cần tiếp tục đối chiếu với file `summary.csv`, `side-*.png` và `boxes-*.json` trong output trước khi chốt nhận xét cuối cùng về số hộp/mean_z. Hình QC hiện chỉ đủ để khẳng định các hộp thừa, chưa đủ cơ sở để kết luận sai lệch z batch hay lệch một hộp z.

## LC ghi nhận riêng

- Quyền dùng PCD/image và đúng ca: cần xác nhận với LC; hình QC không ghi rõ chốt ca hoặc quyền dùng, chỉ cho thấy feedback ngẫu nhiên từ portal.
- Có chạy thật / chỉ phân tích; còn cần lượt thực hành bổ sung: cần có xác nhận LC về lượt chạy thật A/B/C và việc có bổ sung thực hành hay không; hiện tại hình từ portal không thể chứng minh điều này.
- Output đủ, giữ bản gốc, không đưa ca lỗi vào CVAT: có đủ bằng chứng từ hình QC về 3 `extra_object`, nhưng không nên import các case lỗi vào CVAT; các ca QC phải giữ như helper tạo biến đổi có kiểm soát, không phải nhãn đúng.
- Nhận xét từng thành viên và quyết định dừng pipeline: cần có xác nhận từ LC nếu xem xét dừng pipeline hoặc tiếp tục chỉnh/QC. Hiện tại, với bằng chứng trên, chỉ nên ghi nhận rằng có 3 hộp thừa cần xóa và cần kiểm tra lại trước khi tiếp tục nguồn.
- Đồng ý chuyển sang chỉnh/QC / cần bổ sung; lý do: đồng ý chuyển sang chỉnh/QC ở mức hợp lý, nhưng cần bổ sung xác nhận về danh tính reviewer, sự phù hợp ca/PCD và việc đối chiếu `qc-cases` với output thật trước khi kết luận cuối cùng.

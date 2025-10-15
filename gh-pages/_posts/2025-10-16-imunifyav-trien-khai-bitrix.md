---
layout: post
title: "Triển khai ImunifyAV tự động cho Bitrix"
date: 2025-10-16 00:00:00 +0700
tags: [imunifyav, security, bitrix]
---

> Bài viết này tổng hợp lại toàn bộ script trong thư mục `imunifyav/` và hướng dẫn bạn tái sử dụng chúng để triển khai ImunifyAV nhanh chóng cho hạ tầng Bitrix.

## Tổng quan giải pháp
- Bộ script giúp tự động hoá quá trình cài đặt ImunifyAV, cấu hình tích hợp với giao diện Bitrix và thiết lập cảnh báo khi phát hiện mã độc.
- Hai script cài đặt chính hỗ trợ song song cho môi trường CentOS/CloudLinux (`iav_setup.sh`) và Debian (`iav_setup_debian.sh`), trong khi các script còn lại đóng vai trò hỗ trợ (cung cấp dữ liệu người dùng/domain, hook thông báo, proxy repo…).
- Tất cả đều giả định hệ thống chạy với user `bitrix`, thư mục web chính `/home/bitrix/www` (hoặc `/var/www/bitrix/default` trên Debian) và yêu cầu sẵn quyền `root`.

## Chuẩn bị trước khi chạy
- SMTP nội bộ hoạt động để script có thể gửi email qua `mailx`.
- Cài sẵn `curl`, `wget` và đảm bảo truy cập được tới internet. Nếu bị chặn HTTPS tới CloudLinux, cấu hình thêm reverse proxy theo mẫu `nginx_proxy.conf`.
- Xác định email nhận cảnh báo, ví dụ `security@example.com`.
- Với Debian, cần các package: `s-nail`, `jq`.

## Các script thành phần
- **get-users.sh**: trả về danh sách user tích hợp cho ImunifyAV thông qua `jq`. Mặc định tạo user `bitrix` với UID 600.
- **get-domains.sh**: cung cấp thông tin domain chính và domain phụ của Bitrix, giúp giao diện Imunify hiển thị đúng cấu trúc thư mục.
- **iav_hook.sh**: hook đa năng đọc payload JSON, gửi mail hoặc lưu file nhật ký. Bạn chỉ việc chỉnh `MAIL_TO` và thư mục lưu.
- **iav_notify.sh**: phương án gửi thông báo siêu gọn (một dòng email). Có thể dùng độc lập nếu không cần logic phức tạp.
- **iav_setup.sh / iav_setup_debian.sh**: trái tim của quá trình cài đặt. Chúng tải toàn bộ script hỗ trợ, tạo symlink giao diện, cấu hình ignore list, thiết lập lịch quét, cập nhật tham số hiệu năng, và đăng ký hook thông báo.
- **nginx_proxy.conf**: template cấu hình Nginx reverse proxy tới các endpoint của CloudLinux phòng trường hợp mạng nội bộ chặn HTTPS trực tiếp.

## Cài đặt trên CentOS/CloudLinux
Chạy script với tham số email nhận cảnh báo:

```bash
curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/imunifyav/iav_setup.sh \
  | bash -s -- security@example.com
```

Script sẽ thực hiện:
- Cài `mailx`, `jq`, `oniguruma`.
- Tạo `/etc/sysconfig/imunify360/integration.conf` và map các script `get-users.sh`, `get-domains.sh`.
- Đồng bộ thư mục giao diện `/opt/iav/.imunifyav` vào `/home/bitrix/www/`.
- Tải về hook/utility script, set quyền thực thi và chèn email vào `iav_hook.sh`.
- Cài ImunifyAV thông qua `imav-deploy.sh` (mặc định lấy từ mirror `imun.0fr.ru`).
- Đăng ký hook cho sự kiện `USER_SCAN_MALWARE_FOUND` và `CUSTOM_SCAN_MALWARE_FOUND`.
- Thêm danh sách thư mục bỏ qua (cache, backup, upload của Bitrix).
- Tuỳ chỉnh cấu hình scan (Hyperscan, giới hạn CPU/IO/RAM) và tạo cron quét hằng ngày lúc 01:10.
- Điều chỉnh `login.defs` để UID/GID 600 trở thành user thường, phù hợp với user `bitrix`.

## Cài đặt trên Debian
Thao tác tương tự nhưng dùng script dành riêng:

```bash
curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/imunifyav/iav_setup_debian.sh \
  | bash -s -- security@example.com
```

Điểm khác biệt:
- Dùng `apt` và `s-nail`.
- Symlink giao diện về `/var/www/bitrix/default`.
- Mirror repo sử dụng domain `repoimun.0fr.ru` và `filesimun.0fr.ru`, `apiimun.0fr.ru`.
- Chỉnh sửa đường dẫn bỏ qua, cron và ignore list tương ứng môi trường Debian.
- Trigger khởi động lại hàng loạt service Imunify để áp dụng cấu hình mới.

## Tuỳ chỉnh cảnh báo và lưu trữ
- Để nhận mail giàu thông tin, bật `MAIL_ENABLE=yes` trong `iav_hook.sh` và cấu hình `MAIL_TO`.
- Nếu cần lưu log, giữ `FILE_ENABLE=yes`; script sẽ tạo thư mục `/tmp/imunify-script-files` và log theo tên sự kiện.
- Có thể thêm custom handler bên dưới mỗi `# append your stuff here` để gọi webhook, Slack, Telegram…

## Quản lý danh sách bỏ qua
- Các script thêm sẵn ignore list cho cache/backup của Bitrix để tránh false positive.
- Nếu bạn lưu mã nguồn ở thư mục khác, bổ sung vào `/etc/sysconfig/imunify360/malware-filters-admin-conf/ignored.txt`.
- Khi cần quét đầy đủ, có thể tạm thời xoá các dòng ignore và chạy:

```bash
imunify-antivirus update sigs
imunify-antivirus malware on-demand start --path='/home/bitrix/' --file-mask="*.php,*.js,*.html,.htaccess"
```

## Lịch quét và tài nguyên
- Cron mặc định chạy lúc 01:10 mỗi ngày: cập nhật chữ ký trước khi quét.
- Tham số `--intensity-cpu` và `--intensity-io` được đặt ở mức 3, phù hợp server vừa phải. Điều chỉnh tại `/etc/cron.d/iav_scan_schedule`.
- Các lệnh `imunify-antivirus config update` trong script đã giới hạn CPU/IO/RAM khi quét để không ảnh hưởng website.

## Kiểm tra sau cài đặt
1. Đăng nhập giao diện ImunifyAV và xác minh danh sách domain hiển thị đúng.
2. Thực hiện quét thử rồi kiểm tra email/đường dẫn log nhận được từ `iav_hook.sh`.
3. Đảm bảo cron đã tạo: `crontab -l -u root | grep iav_scan_schedule`.
4. Theo dõi `/opt/iav/` xem các script có cập nhật và quyền sở hữu `root:_imunify`.

## Gợi ý bảo trì
- Khi CloudLinux thay đổi endpoint, cập nhật các mirror trong `nginx_proxy.conf` và phần `sed` của script.
- Giữ `jq`, `imunify-antivirus` ở phiên bản mới. Có thể tự động hoá bằng cron `yum update imunify-antivirus` hoặc `apt upgrade`.
- Định kỳ rà soát ignore list để tránh bỏ sót file độc hại mới.

Với bộ script này, bạn có thể chuẩn hoá việc triển khai ImunifyAV cho nhiều máy chủ Bitrix, giảm thao tác thủ công và đảm bảo mỗi đợt quét đều gửi cảnh báo kịp thời.

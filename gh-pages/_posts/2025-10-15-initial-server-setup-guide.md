---
layout: post
title: "Thiết lập máy chủ ban đầu hiệu quả với các Script tự động"
date: 2025-10-15 11:00:00 +0000
categories: initial-setup
---

Chào các bạn DevOps và Full-stack Developer!

Việc thiết lập một máy chủ mới thường tốn rất nhiều thời gian và công sức, đặc biệt là khi bạn cần đảm bảo hiệu suất, bảo mật và các cấu hình tối ưu. Trong bài viết này, chúng ta sẽ khám phá bộ sưu tập các script tự động trong thư mục `initial_server_setup/` giúp bạn đơn giản hóa quá trình này.

## `initial_setup.sh`: Trái tim của quá trình thiết lập

Script `initial_setup.sh` là điểm khởi đầu chính, tự động hóa nhiều tác vụ quan trọng. Khi chạy script này, bạn sẽ được hướng dẫn qua các bước sau:

1.  **Cài đặt các công cụ hữu ích**: Tự động cài đặt các gói phần mềm cần thiết như `wget`, `byobu`, `chrony`, `certbot`, `htop`, `ncdu`, `iotop`, `mc`, `tmux`, `jpegoptim`, `optipng`, `libwebp-tools`, `ImageMagick`, `php-pecl-imagick`, `mysqltuner`, `smem`, `lsof` và nhiều công cụ khác giúp quản lý và tối ưu hóa hệ thống.
2.  **Cấu hình môi trường làm việc**: Thiết lập `liquidprompt` cho một terminal prompt thông minh, cấu hình `nano` với syntax highlighting, và tối ưu hóa hệ thống với `tuned-adm profile virtual-guest`.
3.  **Bảo mật SSH**: Tự động tạo cặp khóa SSH (`ed25519`) cho người dùng `root` và `bitrix`, tăng cường bảo mật cho việc truy cập máy chủ.
4.  **Let's Encrypt và SSL**: Cấu hình `certbot` để tự động gia hạn chứng chỉ SSL từ Let's Encrypt, đảm bảo các dịch vụ web của bạn luôn được bảo mật với HTTPS.
5.  **Tối ưu hóa dịch vụ hệ thống**: Thiết lập `systemd` để tự động khởi động lại các dịch vụ quan trọng như Nginx, HTTPD, Memcached khi gặp lỗi, đồng thời tăng giới hạn tài nguyên (`LimitNPROC`, `LimitNOFILE`).
6.  **Cấu hình Nginx và PHP**: Áp dụng các cấu hình Nginx cơ bản (gzip, server tokens) và các tinh chỉnh PHP (`max_input_vars`, `pcre.recursion_limit`) để cải thiện hiệu suất và bảo mật.
7.  **Giới hạn hệ thống (ulimit)**: Tăng giới hạn số lượng tiến trình và tệp mở cho người dùng, giúp máy chủ xử lý tải cao tốt hơn.
8.  **Đồng bộ thời gian**: Chuyển từ `ntpd` sang `chronyd` để đồng bộ thời gian chính xác hơn.
9.  **Cấu hình MySQL**: Gọi script `mysql_setup.sh` để áp dụng các cấu hình MySQL tối ưu cho hiệu suất và độ ổn định.
10. **Cấu hình Logrotate**: Thiết lập `logrotate` cho các tệp log của Bitrix, giúp quản lý dung lượng đĩa hiệu quả.
11. **Cập nhật bảo mật tự động**: Gọi script `yum_cron_secure.sh` để cài đặt và cấu hình `yum-cron`, đảm bảo máy chủ luôn được cập nhật các bản vá bảo mật quan trọng.
12. **Cài đặt tùy chọn**: Script sẽ hỏi bạn có muốn cài đặt và cấu hình Postfix (cho email), Zabbix (giám sát), Antivirus (bảo mật) và GitLab Runner (CI/CD) hay không, cho phép bạn tùy chỉnh quá trình thiết lập.

## Các Script hỗ trợ quan trọng

Ngoài `initial_setup.sh`, các script sau đóng vai trò quan trọng trong việc hoàn thiện cấu hình máy chủ:

*   **`cron_deduplicator_template.sh`**: Một template hữu ích để đảm bảo chỉ có một instance của cron job chạy tại một thời điểm, tránh các vấn đề về tài nguyên và dữ liệu.
*   **`mydumper_cnf_setup.sh`**: Tự động thêm cấu hình cho `mydumper` và `myloader` vào tệp `.my.cnf`, giúp việc sao lưu và phục hồi MySQL dễ dàng hơn.
*   **`mysql_setup.sh`**: Chứa các tinh chỉnh chi tiết cho MySQL, bao gồm `max_connections`, `long_query_time`, `innodb_buffer_pool_size`, `table_open_cache`, và nhiều thông số khác để tối ưu hóa cơ sở dữ liệu.
*   **`opcache.php`**: Một công cụ PHP đơn giản nhưng mạnh mẽ để theo dõi trạng thái và cấu hình của OPcache, giúp bạn đảm bảo PHP đang hoạt động hiệu quả.
*   **`postfix.sh`**: Cấu hình Postfix để gửi email từ máy chủ, bao gồm cả tùy chọn thiết lập relay với xác thực, rất quan trọng cho các ứng dụng cần gửi thông báo qua email.
*   **`sysctl.sh`**: Tối ưu hóa các tham số kernel (`sysctl`) như `fs.nr_open`, `net.core`, `net.ipv4` để cải thiện hiệu suất mạng và khả năng xử lý tệp của hệ thống.
*   **`yum_cron_secure.sh`**: Đảm bảo máy chủ của bạn luôn được bảo vệ bằng cách tự động cài đặt các bản cập nhật bảo mật thông qua `yum-cron`.

## Khi nào nên sử dụng?

Bộ script này lý tưởng cho:

*   **Thiết lập máy chủ mới**: Giúp bạn nhanh chóng có một máy chủ được cấu hình tốt từ đầu.
*   **Tối ưu hóa máy chủ hiện có**: Áp dụng các tinh chỉnh hiệu suất và bảo mật một cách có hệ thống.
*   **Đảm bảo tính nhất quán**: Sử dụng các script này trên nhiều máy chủ để đảm bảo cấu hình đồng nhất.

## Lời khuyên cho DevOps/Full-stack Developer

*   **Đọc và hiểu script**: Mặc dù các script này tự động hóa, nhưng việc đọc và hiểu chúng là rất quan trọng để bạn biết chính xác những gì đang diễn ra trên máy chủ của mình.
*   **Tùy chỉnh**: Các script này cung cấp một nền tảng tốt. Đừng ngần ngại tùy chỉnh chúng để phù hợp với nhu cầu cụ thể của dự án và môi trường của bạn.
*   **Kiểm tra**: Luôn kiểm tra các thay đổi trong môi trường staging trước khi triển khai lên production.

Hy vọng bài viết này giúp bạn có cái nhìn tổng quan và biết cách tận dụng bộ script `initial_server_setup/` để quản lý máy chủ hiệu quả hơn!

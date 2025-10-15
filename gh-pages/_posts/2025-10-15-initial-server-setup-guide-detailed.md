--- 
layout: post
title: "Thiết lập Máy chủ Linux Ban đầu: Hướng dẫn Chi tiết cho DevOps và Full-stack Developer"
date: 2025-10-15 11:30:00 +0000
categories: initial-setup
---

Chào các bạn DevOps và Full-stack Developer!

Việc thiết lập một máy chủ Linux mới từ đầu có thể là một nhiệm vụ tốn thời gian và dễ mắc lỗi. Để đảm bảo máy chủ của bạn hoạt động hiệu quả, bảo mật và tuân thủ các tiêu chuẩn, việc tự động hóa là chìa khóa. Bộ script `initial_server_setup` này được thiết kế để giúp bạn thực hiện điều đó, biến quá trình cấu hình ban đầu thành một quy trình nhanh chóng và đáng tin cậy.

Bài viết này sẽ đi sâu vào từng phần của script `initial_setup.sh` và các script hỗ trợ, giải thích chi tiết từng bước, cung cấp các đoạn mã quan trọng, lý do cần chúng và cách bạn có thể tùy chỉnh.

## Cách Thực Hiện

Để bắt đầu, bạn chỉ cần tải xuống và chạy script `initial_setup.sh` trên máy chủ Linux (ưu tiên CentOS/RHEL) của mình.

```bash
bash <(curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/initial_setup.sh)
```

**Lưu ý quan trọng:** Luôn đọc và hiểu nội dung của bất kỳ script nào trước khi chạy nó trên môi trường production. Script này sẽ thực hiện các thay đổi sâu rộng trên hệ thống của bạn.

## Phân Tích Chi Tiết `initial_setup.sh`

Script `initial_setup.sh` là một tập hợp các lệnh được thực thi tuần tự để cấu hình máy chủ. Dưới đây là phân tích từng phần:

### 1. Cài đặt các Chương trình Hữu ích

Đầu tiên, script sẽ cài đặt một loạt các gói phần mềm cần thiết cho việc quản lý, giám sát và tối ưu hóa máy chủ.

```bash
yum -y install --enablerepo=epel wget byobu chrony net-tools certbot ncdu iotop htop bind-utils traceroute mc bash-completion bash-completion-extras yum-utils nano tmux deltarpm jpegoptim optipng libwebp-tools ImageMagick php-pecl-imagick mysqltuner smem lsof
```

**Giải thích:**
*   `wget`, `curl`: Công cụ tải file từ internet.
*   `byobu`, `tmux`: Công cụ quản lý phiên terminal, cho phép bạn duy trì các phiên làm việc ngay cả khi mất kết nối SSH.
*   `chrony`: Đồng bộ hóa thời gian hệ thống chính xác hơn `ntpd`.
*   `net-tools`, `bind-utils`, `traceroute`: Các công cụ mạng cơ bản để chẩn đoán và kiểm tra kết nối.
*   `certbot`: Công cụ tự động hóa việc cấp phát và gia hạn chứng chỉ SSL/TLS từ Let's Encrypt.
*   `ncdu`, `iotop`, `htop`, `smem`, `lsof`: Các công cụ giám sát hệ thống mạnh mẽ để theo dõi dung lượng đĩa, I/O, tiến trình, bộ nhớ và các file đang mở.
*   `mc`: Midnight Commander, một trình quản lý file dựa trên terminal.
*   `bash-completion`: Tự động hoàn thành lệnh trong bash.
*   `nano`: Trình soạn thảo văn bản đơn giản.
*   `deltarpm`: Tăng tốc cập nhật `yum`.
*   `jpegoptim`, `optipng`, `libwebp-tools`, `ImageMagick`: Các công cụ tối ưu hóa hình ảnh, rất quan trọng cho các ứng dụng web.
*   `php-pecl-imagick`: Extension PHP cho ImageMagick.
*   `mysqltuner`: Script phân tích cấu hình MySQL và đưa ra khuyến nghị tối ưu hóa.

**Tại sao cần:** Các công cụ này là xương sống cho mọi hoạt động quản trị và phát triển trên máy chủ, từ gỡ lỗi đến tối ưu hóa hiệu suất.

### 2. Cấu hình Môi trường Bash và Liquidprompt

Script cấu hình file `.bashrc` cho người dùng `root` và `bitrix` (nếu có), thêm các alias hữu ích và tích hợp `liquidprompt` để có một terminal prompt thông minh.

```bash
cat <<\EOT >> ~/.bashrc
alias mc='mc -x'
alias door='wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/bitrix/test.php'
alias lst='ls -alt --time-style=long-iso'
alias ncdu='ncdu --color off'
[[ $- = *i* ]] && source /opt/liquidprompt/liquidprompt
export VISUAL=nano
EOT

# ... (tương tự cho người dùng bitrix) ...

wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/liquidprompt -N -O /opt/liquidprompt/liquidprompt
wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/liquidpromptrc -N -O ~/.config/liquidpromptrc
source /opt/liquidprompt/liquidprompt
```

**Giải thích:**
*   **Aliases**: Các lệnh tắt như `mc -x` (Midnight Commander với hỗ trợ chuột), `lst` (liệt kê file chi tiết theo thời gian), `ncdu --color off` (ncdu không màu).
*   **`liquidprompt`**: Một prompt Bash/Zsh thông minh, hiển thị thông tin hữu ích như trạng thái Git, tải hệ thống, thời gian chạy lệnh, v.v. Nó giúp bạn có cái nhìn tổng quan về môi trường làm việc ngay trên dòng lệnh.
*   **`export VISUAL=nano`**: Đặt `nano` làm trình soạn thảo mặc định.
*   **`~/.nanorc`**: Cấu hình syntax highlighting cho `nano`.

**Tại sao cần:** Cải thiện đáng kể trải nghiệm làm việc trên terminal, giúp bạn làm việc nhanh hơn và hiệu quả hơn.

### 3. Bảo mật SSH Key

Script tự động tạo cặp khóa SSH (ED25519) cho người dùng `root` và `bitrix`.

```bash
ssh-keygen -t ed25519 -q -f "$HOME/.ssh/id_ed25519" -N ""
# ... (tương tự cho người dùng bitrix) ...
chown root. /home/bitrix/.ssh/authorized_keys
```

**Giải thích:**
*   `ssh-keygen -t ed25519`: Tạo cặp khóa SSH sử dụng thuật toán ED25519, được coi là an toàn và hiệu quả hơn RSA.
*   `-q`: Chế độ yên lặng.
*   `-f "$HOME/.ssh/id_ed25519"`: Chỉ định đường dẫn và tên file cho khóa riêng tư.
*   `-N ""`: Không đặt passphrase cho khóa (cần cẩn trọng khi sử dụng trong môi trường production).
*   `chown root. /home/bitrix/.ssh/authorized_keys`: Đảm bảo quyền sở hữu đúng cho file `authorized_keys` của người dùng `bitrix` để tăng cường bảo mật.

**Tại sao cần:** SSH key là phương pháp xác thực an toàn hơn nhiều so với mật khẩu, đặc biệt quan trọng cho việc truy cập máy chủ từ xa.

### 4. Cấu hình Certbot (Let's Encrypt)

Phần này thiết lập Certbot để tự động gia hạn chứng chỉ SSL/TLS, đảm bảo website của bạn luôn có HTTPS hợp lệ.

```bash
mkdir -p /etc/letsencrypt/renewal-hooks/deploy/
echo -e '#!/bin/sh\nservice nginx reload' > /etc/letsencrypt/renewal-hooks/deploy/nginx.sh
chmod +x /etc/letsencrypt/renewal-hooks/deploy/nginx.sh

cat <<EOT > /etc/sysconfig/certbot
# ...
DEPLOY_HOOK="--deploy-hook 'systemctl reload nginx'"
CERTBOT_ARGS="--allow-subset-of-names"
EOT

systemctl enable certbot-renew.timer && systemctl start certbot-renew.timer
```

**Giải thích:**
*   **`nginx.sh` deploy hook**: Script này được Certbot chạy sau khi gia hạn chứng chỉ thành công. Nó sẽ reload Nginx để áp dụng chứng chỉ mới.
*   **`/etc/sysconfig/certbot`**: File cấu hình cho Certbot.
    *   `DEPLOY_HOOK`: Chỉ định lệnh sẽ chạy sau khi gia hạn. Ở đây là `systemctl reload nginx`.
    *   `CERTBOT_ARGS="--allow-subset-of-names"`: Cho phép Certbot gia hạn chứng chỉ ngay cả khi một số tên miền trong chứng chỉ không thể xác thực được (hữu ích trong một số trường hợp đặc biệt).
*   **`certbot-renew.timer`**: Kích hoạt và khởi động timer của systemd để tự động chạy Certbot định kỳ (thường là hai lần một ngày) để kiểm tra và gia hạn chứng chỉ.

**Tại sao cần:** HTTPS là tiêu chuẩn bắt buộc cho các website hiện đại, đảm bảo bảo mật dữ liệu truyền tải và cải thiện SEO. Tự động hóa gia hạn giúp tránh tình trạng chứng chỉ hết hạn gây gián đoạn dịch vụ.

### 5. Tối ưu hóa Systemd Services

Script tăng giới hạn tài nguyên và cấu hình tự động khởi động lại cho các dịch vụ quan trọng như Nginx, HTTPD, Memcached.

```bash
mkdir -p /etc/systemd/system/nginx.service.d && echo -e '[Service]\nRestart=on-failure\nRestartSec=5\nLimitNPROC=65535\nLimitNOFILE=1000000' >> /etc/systemd/system/nginx.service.d/override.conf
# ... (tương tự cho httpd và memcached) ...
systemctl daemon-reload
```

**Giải thích:**
*   **`override.conf`**: Các file này được sử dụng để ghi đè (override) các cấu hình mặc định của dịch vụ systemd mà không cần chỉnh sửa file `.service` gốc.
*   `Restart=on-failure`: Dịch vụ sẽ tự động khởi động lại nếu nó thoát với mã lỗi.
*   `RestartSec=5`: Chờ 5 giây trước khi thử khởi động lại.
*   `LimitNPROC=65535`: Tăng giới hạn số lượng tiến trình mà dịch vụ có thể tạo.
*   `LimitNOFILE=1000000`: Tăng giới hạn số lượng file mà dịch vụ có thể mở.
*   `systemctl daemon-reload`: Tải lại cấu hình systemd để áp dụng các thay đổi.

**Tại sao cần:** Đảm bảo các dịch vụ quan trọng luôn hoạt động ổn định, tự động phục hồi sau sự cố và có đủ tài nguyên để xử lý tải cao.

### 6. Cấu hình Nginx

Các tinh chỉnh cơ bản cho Nginx và tải xuống các file cấu hình Nginx bổ sung.

```bash
echo 'gzip_vary on;' >> /etc/nginx/bx/settings/z_bx_custom.conf
systemctl reload nginx

# Tải xuống các file cấu hình Nginx bổ sung
wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/nginx/ssl.common.conf -N -P /etc/nginx/bx/conf/
wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/nginx/block_access.conf -N -P /etc/nginx/
# ... và nhiều file khác ...

nginx -t && systemctl reload nginx
```

**Giải thích:**
*   `gzip_vary on;`: Bật `Vary: Accept-Encoding` header cho gzip, giúp các proxy cache phân biệt giữa các phiên bản nén và không nén của tài nguyên.
*   Các file `.conf` được tải xuống từ thư mục `nginx/` bao gồm:
    *   `ssl.common.conf`: Cấu hình SSL/TLS chung.
    *   `block_access.conf`: Các quy tắc chặn truy cập.
    *   `qrator.conf`, `acme_well_known.conf`, `locations.conf`, `seo.conf`: Các cấu hình cụ thể cho Bitrix, Let's Encrypt challenge, định tuyến URL và SEO.
*   `nginx -t`: Kiểm tra cú pháp cấu hình Nginx.
*   `systemctl reload nginx`: Tải lại cấu hình Nginx mà không làm gián đoạn dịch vụ.

**Tại sao cần:** Tối ưu hóa Nginx là rất quan trọng để đạt được hiệu suất web cao, bảo mật và khả năng xử lý lưu lượng truy cập lớn.

### 7. Cấu hình PHP

Script áp dụng các tinh chỉnh cho PHP thông qua file `z_bx_custom.ini`.

```bash
cat <<EOT >> /etc/php.d/z_bx_custom.ini
mail.add_x_header = Off
pcre.recursion_limit = 100000
cgi.fix_pathinfo = 0
max_input_vars = 100000
EOT
mv -f /etc/php.d/20-curl.ini.disabled /etc/php.d/20-curl.ini
```

**Giải thích:**
*   `mail.add_x_header = Off`: Tắt việc thêm header `X-PHP-Originating-Script` vào email, có thể tăng cường bảo mật nhỏ.
*   `pcre.recursion_limit = 100000`: Tăng giới hạn đệ quy cho PCRE (Perl Compatible Regular Expressions), hữu ích cho các ứng dụng PHP sử dụng nhiều regex phức tạp.
*   `cgi.fix_pathinfo = 0`: Một cài đặt bảo mật quan trọng. Khi `cgi.fix_pathinfo` được bật, PHP sẽ cố gắng "sửa" đường dẫn file nếu nó không tìm thấy file chính xác, điều này có thể dẫn đến lỗ hổng bảo mật (ví dụ: thực thi mã PHP trong các file không phải PHP).
*   `max_input_vars = 100000`: Tăng số lượng biến đầu vào tối đa mà PHP có thể xử lý, hữu ích cho các form lớn hoặc các ứng dụng phức tạp.
*   `mv -f /etc/php.d/20-curl.ini.disabled /etc/php.d/20-curl.ini`: Kích hoạt extension `curl` cho PHP.

**Tại sao cần:** Tối ưu hóa PHP giúp cải thiện hiệu suất ứng dụng web và khắc phục các vấn đề liên quan đến giới hạn tài nguyên hoặc bảo mật.

### 8. Giới hạn Hệ thống (System Limits)

Script cấu hình giới hạn tài nguyên cho người dùng và tiến trình thông qua `limits.d`.

```bash
echo -e 'root soft nproc unlimited\n* soft nproc 65535\n* hard nproc 65535\n* soft nofile 1000000\n* hard nofile 1000000' > /etc/security/limits.d/20-nproc.conf  && sysctl --system
```

**Giải thích:**
*   **`/etc/security/limits.d/20-nproc.conf`**: File cấu hình giới hạn tài nguyên cho người dùng.
    *   `root soft nproc unlimited`: Người dùng `root` có thể tạo số lượng tiến trình không giới hạn (soft limit).
    *   `* soft nproc 65535`: Tất cả người dùng khác có thể tạo tối đa 65535 tiến trình (soft limit).
    *   `* hard nproc 65535`: Giới hạn cứng cho số lượng tiến trình.
    *   `* soft nofile 1000000`: Giới hạn mềm cho số lượng file mở.
    *   `* hard nofile 1000000`: Giới hạn cứng cho số lượng file mở.
*   `sysctl --system`: Áp dụng các thay đổi `sysctl` ngay lập tức.

**Tại sao cần:** Các ứng dụng web và cơ sở dữ liệu thường cần mở rất nhiều file và tạo nhiều tiến trình. Tăng các giới hạn này giúp ngăn chặn lỗi "Too many open files" hoặc "Cannot fork" dưới tải cao.

### 9. Tối ưu hóa Kernel với `sysctl.sh`

Script `sysctl.sh` được gọi để áp dụng các tinh chỉnh kernel quan trọng, cải thiện hiệu suất mạng và hệ thống.

```bash
curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/sysctl.sh | bash
```

**Giải thích chi tiết về `sysctl.sh`:**
Script này điều chỉnh các thông số kernel thông qua `sysctl` để tối ưu hóa hiệu suất mạng (TCP/IP stack), quản lý bộ nhớ và các giới hạn khác.

**Các thông số quan trọng:**
*   `fs.nr_open`, `fs.file-max`: Tăng giới hạn số lượng file mà hệ thống có thể mở.
*   `kernel.msgmni`, `kernel.sem`, `kernel.shmmax`, `kernel.shmall`: Các thông số liên quan đến System V IPC (Inter-Process Communication), quan trọng cho một số ứng dụng.
*   `net.core.wmem_max`, `net.core.rmem_max`: Tăng kích thước buffer gửi/nhận tối đa cho socket.
*   `net.ipv4.tcp_rmem`, `net.ipv4.tcp_wmem`: Cấu hình kích thước buffer TCP.
*   `net.core.netdev_max_backlog`, `net.core.somaxconn`: Tăng hàng đợi cho các kết nối mạng đến.
*   `net.ipv4.tcp_fin_timeout`, `net.ipv4.tcp_keepalive_*`: Tinh chỉnh các thông số TCP keepalive và timeout để quản lý kết nối hiệu quả hơn.
*   `net.ipv4.tcp_max_syn_backlog`: Tăng hàng đợi cho các yêu cầu kết nối TCP SYN.
*   `vm.swappiness`: Giảm giá trị này (ví dụ: 10) để kernel ít sử dụng swap hơn, ưu tiên giữ dữ liệu trong RAM.
*   `net.ipv4.ip_local_port_range`: Mở rộng dải cổng cục bộ có sẵn.
*   `net.netfilter.nf_conntrack_max`: Tăng số lượng kết nối được theo dõi bởi Netfilter (conntrack), quan trọng cho các máy chủ có nhiều kết nối đồng thời.

**Tại sao cần:** Tối ưu hóa kernel là một bước quan trọng để máy chủ có thể xử lý lượng lớn kết nối mạng, quản lý tài nguyên hiệu quả và duy trì hiệu suất cao dưới tải nặng.

### 10. Cấu hình Mydumper/Myloader với `mydumper_cnf_setup.sh`

Script này tự động thêm cấu hình cho `mydumper` và `myloader` vào file `.my.cnf` của người dùng `root`.

```bash
curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/mydumper_cnf_setup.sh | bash
```

**Giải thích chi tiết về `mydumper_cnf_setup.sh`:**
Script này đọc thông tin `user`, `password`, `socket` từ phần `[client]` trong `/root/.my.cnf` và sau đó thêm các phần `[mydumper]` và `[myloader]` vào file đó với cùng thông tin xác thực.

**Đoạn mã quan trọng:**
```bash
# Trong mydumper_cnf_setup.sh
sectionContent=$(sed -n '/^\['client\]/,/^\['/p' $pathToIniFile | sed -e '/^\['/d' | sed -e '/^$/d');
username=$(getValueFromINI "$sectionContent" "user");
password=$(getValueFromINI "$sectionContent" "password");
socket=$(getValueFromINI "$sectionContent" "socket");

echo -e "\n[mydumper]\nuser=${username}\npassword=${password}\nsocket=${socket}\n\n[myloader]\nuser=${username}\npassword=${password}\nsocket=${socket}\n" >> $pathToIniFile
```

**Tại sao cần:** `mydumper` và `myloader` là các công cụ sao lưu/phục hồi MySQL song song, nhanh hơn `mysqldump` truyền thống. Việc cấu hình sẵn thông tin xác thực giúp việc sử dụng chúng trở nên thuận tiện và an toàn hơn.

### 11. Tối ưu hóa MySQL với `mysql_setup.sh`

Script này áp dụng một loạt các tinh chỉnh cho MySQL để cải thiện hiệu suất và độ ổn định.

```bash
curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/mysql_setup.sh | bash
```

**Giải thích chi tiết về `mysql_setup.sh`:**
Script này tạo file `/etc/mysql/conf.d/z_bx_custom.cnf` và điền vào đó các cấu hình MySQL được khuyến nghị.

**Các thông số quan trọng:**
*   `max_connections`: Số lượng kết nối tối đa mà MySQL server chấp nhận.
*   `long_query_time`, `log-queries-not-using-indexes`: Cấu hình log slow query, giúp bạn xác định các truy vấn cần tối ưu hóa.
*   `innodb_buffer_pool_size`: Kích thước bộ nhớ đệm cho InnoDB, đây là thông số quan trọng nhất ảnh hưởng đến hiệu suất MySQL. Giá trị này nên được đặt dựa trên tổng RAM của máy chủ.
*   `innodb_buffer_pool_instances`: Chia buffer pool thành nhiều instance để giảm contention.
*   `table_open_cache`: Số lượng bảng tối đa mà MySQL có thể giữ trong bộ nhớ đệm.
*   `innodb_flush_log_at_trx_commit`: Kiểm soát độ bền của giao dịch. Giá trị `0` hoặc `2` có thể cải thiện hiệu suất nhưng giảm độ bền trong trường hợp mất điện.
*   `skip-name-resolve`: Tắt phân giải DNS cho các kết nối client, tăng tốc độ kết nối.
*   `skip-networking`: Chỉ cho phép kết nối cục bộ (qua socket), tăng cường bảo mật nếu MySQL không cần truy cập từ xa.
*   `default_password_lifetime=0`: Vô hiệu hóa chính sách hết hạn mật khẩu mặc định.

**Tại sao cần:** MySQL là trái tim của nhiều ứng dụng web. Việc tối ưu hóa cấu hình MySQL là cực kỳ quan trọng để đảm bảo ứng dụng của bạn hoạt động nhanh chóng và ổn định, đặc biệt dưới tải cao.

### 12. Đồng bộ thời gian với Chrony

Script chuyển từ `ntpd` sang `chronyd` để đồng bộ thời gian.

```bash
systemctl stop ntpd
systemctl disable ntpd
systemctl enable chronyd
systemctl restart chronyd
```

**Giải thích:** `chronyd` thường được ưu tiên hơn `ntpd` trên các hệ thống Linux hiện đại vì nó có khả năng đồng bộ thời gian nhanh hơn, chính xác hơn và hoạt động tốt hơn trong các môi trường ảo hóa hoặc khi máy chủ thường xuyên bị tạm dừng.

**Tại sao cần:** Đồng bộ thời gian chính xác là rất quan trọng cho các hoạt động log, chứng chỉ SSL, xác thực và nhiều tác vụ hệ thống khác.

### 13. Vô hiệu hóa các Dịch vụ không cần thiết

Script dừng và vô hiệu hóa một số dịch vụ có thể không cần thiết trong môi trường cụ thể này.

```bash
systemctl stop stunnel.service
systemctl disable stunnel
systemctl stop httpd-scale.service
systemctl disable httpd-scale.service
find /etc/cron.d/bx_httpd-scale -type f -print0 | xargs -0 sed -i 's/* * * * */#* * * * */g'
```

**Giải thích:**
*   `stunnel`: Một proxy SSL/TLS. Có thể không cần thiết nếu bạn đã sử dụng Nginx để xử lý SSL.
*   `httpd-scale`: Một dịch vụ liên quan đến việc mở rộng Apache HTTPD. Có thể không cần thiết nếu bạn đang sử dụng Nginx làm reverse proxy hoặc không sử dụng Apache.
*   `sed -i 's/* * * * */#* * * * */g'`: Comment out các dòng trong cron job liên quan đến `bx_httpd-scale` để vô hiệu hóa chúng.

**Tại sao cần:** Vô hiệu hóa các dịch vụ không cần thiết giúp giảm thiểu bề mặt tấn công, giải phóng tài nguyên hệ thống và đơn giản hóa việc quản lý.

### 14. Cấu hình Logrotate

Script thiết lập `logrotate` cho các file log của Bitrix.

```bash
wget https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/bitrixlog -N -P /etc/logrotate.d/
```

**Giải thích chi tiết về `bitrixlog`:**
File này định nghĩa cách `logrotate` sẽ xử lý các file log của Bitrix.

**Đoạn mã quan trọng (từ `bitrixlog`):**
```
/home/bitrix/www/*.log /home/bitrix/ext_www/*/*.log {
        su bitrix bitrix
        daily
        dateext
        dateformat -%Y-%m-%d
        rotate 30
        copytruncate
        notifempty
        missingok
        compress
        sharedscripts
        }
```

**Giải thích:**
*   `su bitrix bitrix`: Chạy `logrotate` với quyền của người dùng `bitrix`.
*   `daily`: Xoay log hàng ngày.
*   `dateext`: Thêm ngày vào tên file log đã xoay.
*   `rotate 30`: Giữ lại 30 file log đã xoay.
*   `copytruncate`: Sao chép file log gốc và sau đó cắt ngắn nó, giúp các ứng dụng vẫn có thể ghi vào file gốc mà không cần khởi động lại.
*   `compress`: Nén các file log đã xoay.

**Tại sao cần:** Log file có thể phát triển rất lớn, chiếm dụng dung lượng đĩa. `logrotate` giúp quản lý kích thước log, tiết kiệm không gian và dễ dàng hơn trong việc phân tích log.

### 15. Vô hiệu hóa HTTPD Access Logs

Script vô hiệu hóa ghi log truy cập cho HTTPD (Apache).

```bash
find /etc/httpd/ -type f -print0 | xargs -0 sed -i 's/CustomLog/#CustomLog/g'
find /etc/httpd/ -type f -print0 | xargs -0 sed -i 's/LogFormat "%h/LogFormat "%a/g'
systemctl reload httpd
```

**Giải thích:**
*   `sed -i 's/CustomLog/#CustomLog/g'`: Comment out các dòng `CustomLog` trong cấu hình Apache, ngăn Apache ghi log truy cập.
*   `sed -i 's/LogFormat "%h/LogFormat "%a/g'`: Thay đổi định dạng log để sử dụng địa chỉ IP thực (`%a`) thay vì hostname (`%h`), hữu ích khi có reverse proxy như Nginx.

**Tại sao cần:** Nếu Nginx đang hoạt động như một reverse proxy và ghi log truy cập, việc vô hiệu hóa log của Apache có thể tránh trùng lặp và tiết kiệm tài nguyên.

### 16. Cập nhật Bảo mật Tự động với `yum_cron_secure.sh`

Script này cài đặt và cấu hình `yum-cron` để tự động cài đặt các bản cập nhật bảo mật.

```bash
bash <(curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/yum_cron_secure.sh)
```

**Giải thích chi tiết về `yum_cron_secure.sh`:**
Script này cài đặt gói `yum-cron` và sau đó ghi đè file cấu hình `/etc/yum/yum-cron.conf` với các cài đặt được tối ưu hóa.

**Đoạn mã quan trọng (từ `yum_cron_secure.sh`):**
```bash
# Trong /etc/yum/yum-cron.conf
update_cmd = security
download_updates = yes
apply_updates = yes
random_sleep = 360
emit_via = stdio
```

**Giải thích:**
*   `update_cmd = security`: Chỉ áp dụng các bản cập nhật bảo mật.
*   `download_updates = yes`: Tự động tải xuống các bản cập nhật.
*   `apply_updates = yes`: Tự động cài đặt các bản cập nhật.
*   `random_sleep = 360`: Chờ một khoảng thời gian ngẫu nhiên (tối đa 360 phút) trước khi chạy, giúp phân tán tải trên các máy chủ cập nhật.
*   `emit_via = stdio`: Gửi thông báo qua stdout, cho phép cron gửi email thông báo nếu được cấu hình.

**Tại sao cần:** Bảo mật là ưu tiên hàng đầu. Tự động cập nhật các bản vá bảo mật giúp bảo vệ máy chủ của bạn khỏi các lỗ hổng đã biết mà không cần can thiệp thủ công thường xuyên.

### 17. Các Cài đặt Tùy chọn

Cuối cùng, script sẽ hỏi bạn có muốn cài đặt và cấu hình các dịch vụ bổ sung hay không:

*   **Postfix (cho email)**:
    ```bash
    echo "Do you wish to install postfix?"
    select yn in "Yes" "No"; do
        case $yn in
            Yes ) bash <(curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/initial_server_setup/postfix.sh) ; break;; 
            No ) exit;; 
        esac
    done
    ```
    **Giải thích:** Nếu bạn chọn "Yes", script `postfix.sh` sẽ được chạy. Script này cấu hình Postfix làm Mail Transfer Agent (MTA) để gửi email từ máy chủ. Nó cũng cung cấp tùy chọn thiết lập relay SMTP với xác thực, rất quan trọng cho các ứng dụng cần gửi email thông báo hoặc giao dịch.
    **Cấu hình:** Script `postfix.sh` sẽ hỏi bạn các thông tin như relay SMTP server, cổng, login, password và domain.

*   **Zabbix (giám sát)**:
    ```bash
    echo "Do you wish to install zabbix?"
    select yn in "Yes" "No"; do
        case $yn in
            Yes )
                    read -p "Please enter a ip of zabbix server : " server_ip
                    read -p "Please enter a zabbix server domain : " domain
                    read -p "Please enter a hostname for this zabbix node : " hostname

                    curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/bitrix/zabbix.sh | bash -s -- ${server_ip} ${domain} ${hostname}  ; break;;

            No ) exit;; 
        esac
    done
    ```
    **Giải thích:** Nếu bạn chọn "Yes", script `bitrix/zabbix.sh` sẽ được chạy để cài đặt và cấu hình Zabbix Agent, cho phép máy chủ này được giám sát bởi một Zabbix Server.
    **Cấu hình:** Bạn cần cung cấp IP của Zabbix Server, domain của Zabbix Server và hostname cho node này.

*   **Antivirus**:
    ```bash
    echo "Do you wish to install antivirus?"
    select yn in "Yes" "No"; do
        case $yn in
            Yes )
                    read -p "Please enter your email : " your_mail

                    curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/av/av_setup.sh | bash -s -- ${your_mail}  ; break;;

            No ) exit;; 
        esac
    done
    ```
    **Giải thích:** Nếu bạn chọn "Yes", script `av/av_setup.sh` sẽ được chạy để cài đặt và cấu hình giải pháp antivirus.
    **Cấu hình:** Bạn cần cung cấp địa chỉ email của mình để nhận thông báo.

*   **Backup Script**:
    ```bash
    echo "Do you wish to install backup_script?"
    select yn in "Yes" "No"; do
        case $yn in
            Yes )
                    read -p "Please enter your sitename : " sitename

                    curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/bitrix/auto_setup.sh | bash -s -- /home/bitrix/www ; break;;

            No ) exit;; 
        esac
    done
    ```
    **Giải thích:** Nếu bạn chọn "Yes", script `bitrix/auto_setup.sh` sẽ được chạy để thiết lập các script sao lưu tự động.
    **Cấu hình:** Bạn cần cung cấp tên trang web của mình.

*   **GitLab Runner**:
    ```bash
    echo "Do you wish to install gitlab-runner?"
    select yn in "Yes" "No"; do
        case $yn in
            Yes ) curl -sL https://raw.githubusercontent.com/YogSottot/useful_scripts/master/git/runner.sh | bash ; break;; 
            No ) exit;; 
        esac
    done
    ```
    **Giải thích:** Nếu bạn chọn "Yes", script `git/runner.sh` sẽ được chạy để cài đặt GitLab Runner, cho phép máy chủ này thực hiện các tác vụ CI/CD từ GitLab.

**Tại sao cần:** Các tùy chọn này cho phép bạn mở rộng chức năng của máy chủ theo nhu cầu cụ thể của dự án, từ gửi email, giám sát, bảo mật đến tự động hóa CI/CD.

## Kết luận

Bộ script `initial_server_setup` cung cấp một giải pháp toàn diện để tự động hóa quá trình cấu hình máy chủ Linux ban đầu. Bằng cách tận dụng các script này, bạn có thể:

*   **Tiết kiệm thời gian**: Giảm đáng kể thời gian thiết lập thủ công.
*   **Đảm bảo tính nhất quán**: Áp dụng cùng một cấu hình trên nhiều máy chủ.
*   **Tăng cường bảo mật**: Tự động hóa các biện pháp bảo mật cơ bản.
*   **Cải thiện hiệu suất**: Áp dụng các tinh chỉnh tối ưu cho hệ thống, Nginx, PHP và MySQL.

Hãy nhớ rằng, việc hiểu rõ từng bước và tùy chỉnh script theo môi trường của bạn là chìa khóa để tận dụng tối đa bộ công cụ mạnh mẽ này. Chúc bạn thành công!

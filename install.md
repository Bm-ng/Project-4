# Project-4
Dựng mô hình Graph monitor (Grafana + InfluxDB + Telegraf)
https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-centos-7

https://github.com/influxdata/telegraf/tree/master/plugins/inputs

# Dựng mô hình Graph monitor (Grafana + InfluxDB + Telegraf)
## 1 - Thêm kho lưu trữ 
- Thiết lập tệp cấu hình kho lưu trữ giúp việc cài đặt liền mạch.
- Tạo tập tin mới:
```sh
#vim /etc/yum.repos.d/influxdata.repo 
```
- Cấu hình:
```sh
[influxdb]
name = InfluxData Repository - RHEL $releasever
baseurl = https://repos.influxdata.com/rhel/$releasever/$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
```
- Lưu tệp và thoát khỏi trình chỉnh sửa. Bây giờ chúng ta có thể cài đặt và cấu hình InfluxDB

## 2 - Cài đặt InfluxDB và cấu hình xác thực

- InfluxDB là một cơ sở dữ liệu nguồn mở được tối ưu hóa để lưu trữ nhanh, có tính sẵn sàng cao và truy xuất dữ liệu chuỗi thời gian. InfluxDB rất tốt cho việc theo dõi hoạt động, số liệu ứng dụng và phân tích thời gian thực.
- Chạy lệnh sau để cài đặt InfluxDB:
```sh
#yum install influxdb
```
- Trong quá trình cài đặt, bạn sẽ được yêu cầu nhập khóa GPG. Xác nhận rằng bạn muốn nhập khóa này để cài đặt có thể tiếp tục.

- Khi quá trình cài đặt hoàn tất, hãy khởi động dịch vụ InfluxDB:
```sh
#systemctl start influxdb
#systemctl enable influxdb
```
- Kích hoạt xác thực người dùng để hạn chế quyền truy cập vào cơ sở dữ liệu. Cho phép tạo ít nhất một người dùng quản trị.
```sh
#influx
``` 
- Tạo tài khoản người dùng và password:
```sh
>CREATE USER "bmng" WITH PASSWORD 'Abc@123' WITH ALL PRIVILEGES
```
- Xác minh rằng người dùng đã được tạo:
```sh
>show users 
```
- Bạn sẽ thấy đầu ra sau, xác minh rằng người dùng của bạn đã được tạo:
```sh
Output
    user  admin
    ----  -----
    bmng true
```
- Thoát ra khỏi influx: 
```sh
>exit 
```

- Mở tập tin /etc/influxdb/influxdb.conftrong trình soạn thảo. Đây là tệp cấu hình cho InfluxDB.
```sh
#vim /etc/influxdb/influxdb.conf
```
 - Xác định vị trí `[http]`, bỏ ghi chú `auth-enabledtùy` chọn và đặt giá trị của nó thành `true`:
 ```sh
 ...
    [http]
      # Determines whether HTTP endpoint is enabled.
      # enabled = true

      # The bind address used by the HTTP service.
      # bind-address = ":8086"

      # Determines whether HTTP authentication is enabled.
      auth-enabled = true
...
```
- Lưu lại tệp cấu hình và khởi động lại influxdb
```sh
#systemctl restart influxdb 
```

## 3 - Cài đặt Telegraf

- Telegraf là một đại lý nguồn mở thu thập các số liệu và dữ liệu trên hệ thống mà nó đang chạy hoặc từ các dịch vụ khác. Telegraf sau đó ghi dữ liệu vào InfluxDB hoặc các đầu ra khác.
- Chạy lệnh sau để cài đặt Telegraf:
```sh
#yum install telegraf
```
- Telegraf sử dụng plugin để nhập dữ liệu đầu vào và đầu ra. Plugin đầu ra mặc định là dành cho InfluxDB. Vì đã kích hoạt xác thực người dùng cho IndexDB, nên phải sửa đổi tệp cấu hình của Telegraf để chỉ định tên người dùng và mật khẩu đã định cấu hình. Mở tệp cấu hình Telegraf trong trình chỉnh sửa:
```sh
#vim /etc/telegraf/telegraf.conf
``` 
- Xác định vị trí `[outputs.influxdb]`phần và cung cấp tên người dùng và mật khẩu:
```sh
[[outputs.influxdb]]
      ## The full HTTP or UDP endpoint URL for your InfluxDB instance.
      ## Multiple urls can be specified as part of the same cluster,
      ## this means that only ONE of the urls will be written to each interval.
      # urls = ["udp://localhost:8089"] # UDP endpoint example
      urls = ["http://localhost:8086"] # required
      ## The target database for metrics (telegraf will create it if not exists).
      database = "telegraf" # required

      ...

      ## Write timeout (for the InfluxDB client), formatted as a string.
      ## If not provided, will default to 5s. 0s means no timeout (not recommended).
      timeout = "5s"
      username = "bmng"
      password = "Abc@123"
      ## Set the user agent for HTTP POSTs (can be useful for log differentiation)
      # user_agent = "telegraf"
      ## Set UDP payload size, defaults to InfluxDB UDP Client default (512 bytes)
      # udp_payload = 512
```
- Lưu tệp, thoát trình chỉnh sửa và bắt đầu Telegraf:
```sh 
#systemctl start telegraf
```
- Telegraf hiện đang thu thập dữ liệu và viết nó lên InfluxDB. Hãy mở bảng điều khiển InfluxDB và xem Telegraf đang lưu trữ các phép đo nào trong cơ sở dữ liệu. Kết nối với tên người dùng và mật khẩu bạn đã cấu hình trước đó:
```sh
#influx -username 'bmng' -password 'Abc@123'
------
>show databases

```
- Bạn sẽ thấy `telegraf` cơ sở dữ liệu được liệt kê trong đầu ra:
```sh
Output
    name: databases
    name
    ----
    _internal
    telegraf
```
*Lưu ý : Nếu bạn không thấy telegrafcơ sở dữ liệu, hãy kiểm tra cài đặt Telegraf bạn đã định cấu hình để đảm bảo bạn đã chỉ định tên người dùng và mật khẩu phù hợp.*

- Thực hiện lệnh sau để chuyển sang cơ sở dữ liệu Telegraf:
```sh
>use telegraf
>show measurements
```
- Đầu ra:
```sh
Output
    name: measurements
    name
    ----
    cpu
    disk
    diskio
    kernel
    mem
    processes
    swap
    system
```
- Như bạn có thể thấy, Telegraf đã thu thập và lưu trữ rất nhiều thông tin trong cơ sở dữ liệu này.

- Có hơn 60 plugin đầu vào cho Telegraf. Nó có thể thu thập các số liệu từ nhiều dịch vụ và cơ sở dữ liệu phổ biến, bao gồm:

Apache
Cassandra
Docker
Elaticsearch
Graylog
IPtables
MySQL
PostgreSQL
Redis
SNMP
...
- Bạn có thể xem hướng dẫn sử dụng cho từng plugin đầu vào bằng cách chạy trong cửa sổ terminal.`telegraf -usage plugin-name`

- Thoát khỏi bảng điều khiển InfluxDB: 
```sh
>exit 
```

## 4 - Cài đặt Grafana
- Bây giờ, cài đặt Grafana từ kho lưu trữ YUM chính thức của nó, bắt đầu bằng cách thêm cấu hình sau vào `/etc/yum.repose.d/grafana.repo` tệp lưu trữ.
```sh 
#vim /etc/yum.repose.d/grafana.repo
```sh
[grafana]
name=grafana
baseurl=https://packages.grafana.com/oss/rpm
repo_gpgcheck=1
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
```
- Sau khi thêm kho lưu trữ vào cấu hình YUM, cài đặt gói Grafana:
```sh
#yum install grafana
```
- Khi bạn đã cài đặt Grafana , tải lại cấu hình trình quản lý systemd, khởi động máy chủ grafana, kiểm tra xem dịch vụ có hoạt động hay không bằng cách xem trạng thái của nó và cho phép nó tự động khởi động khi khởi động.

```sh
# systemctl daemon-reload 
# systemctl start grafana-server 
# systemctl status grafana-server 
# systemctl enable grafana-server
```
- Tiếp theo, mở cổng 3000 mà máy chủ Grafana nghe.

```sh
# firewall-cmd --add-port=3000/tcp --permanent
# firewall-cmd --reload
```























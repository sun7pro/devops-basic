# 1. Ý nghĩa
> Deployer là cli tool dùng để tự động hóa phần lớn công việc trong quá trình deploy ứng dụng lên server.

> Sử dụng được cho rất nhiều framework PHP.

> Có thể hình dung sơ bộ như sau :
- Tất cả những thông số input đầu vào được khai báo trong deploy.php file(file này được cli tạo ra và quản lí sau khi init deployer).
- Cli deployer sẽ đọc file này và dựng lên luồng xử lí dữ liệu.
Luồng giống như chúng ta làm thủ công, từ việc login server remote cho đến cài đặt module cho project, pull code từ git hub, ...
- Nó sử dụng giao thức SSH để giao tiếp với server, luồng xử lí được chia theo đơn vị là `task` - khởi phát các action. 

# 2. Cài đặt
> Download deployer.phar về với tên là deployer.phar luôn:
```bash
curl -LO https://deployer.org/deployer.phar
```
> Di chuyển nó vào thư mục /usr/local/bin/dep:
```bash
mv deployer.phar /usr/local/bin/dep
```
> Đánh dấu nó với hệ thống là thư mục chứa lệnh để thực thi:
```bash
chmod +x /usr/local/bin/dep
```
> Vào root project folder rồi tạo file deploy.php bằng command:
```bash
dep init
```
> Fetch các component cần thiết về project:
```bash
composer global require deployer/deployer
```

# 3. Những điều cần biết
> Lưu ý:

> Ở đây chỉ để cập tới việc deploy code, ngoài ra để ứng dụng có thể chạy được thì hiển nhiên là chúng ta phải setting database,
> tạo key SSH thông luồng giữa local-server-github, user trên server, config host, ...

> Vì bản chất deployer sử dụng giao thức SSH để lưu chuyển data qua lại giữa các bên nên nó sẽ không đảm nhiệm được những nhiệm vụ ngoài phạm vi.

## 3.1. Các khai báo cốt lõi trong deploy.php
### Định nghĩa, sử dụng biến global
```bash
set('param', 'value');
```
Riêng việc định nghĩa biến env sẽ update env file trên server.
```bash
get('param');
```
### Định nghĩa task
```bash
task('taskName', function () {
   // vạn sự tùy tâm
   // chạy command, nhập input, rollback version release, ....
});
```
Cũng có thể group các task, hoặc các command vào 1 alias như thế này:
```bash
task('taskName', [
    'task1|command1',
    'task2|command2',
    ...
]);
```
Riêng với taskName 'deploy' được ngầm hiểu là task bắt đầu tiến trình deploy.
### Chạy command cụ thể trên server
```bash
run('cd {{release_path}} && command1 && command2');
```
### Thay đổi thứ tự thực hiện các task
```bash
after('task1', 'task2');
```
```bash
before('task1', 'task2');
```
Nhìn vào đây, ta có thể linh hoạt được kha khá công việc thay vì sửa file.

Ví dụ hôm nay muốn deploy branch khác, trước khi thực thi deploy task hãy bắn ra yêu cầu nhập: 
```bash
ask('What branch to deploy?', 'default branch');
```
### Khai báo host để deploy lên
```bash
host('domain[a:f]', 'domain1', 'domain2', ...)
    ->user('user')
    ->port('port');
```

Hoặc có thể tự config host và include `inventory('hosts.yml');` như https://deployer.org/docs/hosts.html

## 3.2. Luồng deploy
Bình thường thì `deploy` task thực hiện với nhiều task nhỏ hơn được định nghĩa sẵn bên trong tùy vào version framework php:
```bash
task('deploy', [
    'deploy:prepare',
    'deploy:lock',
    'deploy:release',
    ...
    'deploy:unlock',
    'cleanup',
    'success'
]);
```

Tuy nhiên cũng có thể override cho phù hợp. Ý nghĩa cụ thể cho từng task ở https://deployer.org/docs/flow.html

Trường hợp deploy đồng thời lên nhiều server để tối ưu hiệu năng và tốc độ,
giải pháp đưa ra là build version release duy nhất 1 lần ngay trên local và copy tới các remote server https://deployer.org/docs/advanced/deploy-strategies.html.

## 3.3. Cấu trúc thư mục trong deploy_path (Laravel)
>-- .dep
>
>-- current
>
>-- releases
>
>>--> 1
>
>>--> ...
>
>-- shared
>
>>--> .env
>
>>--> storage

- .dep: folder chứa meta data.
- current: bản release gần nhất releases->1 được deploy lên.
- releases: tổng hợp các version đã được deploy theo ngăn xếp.
- shared: các folder, file được public share giữa các release. Mặc định là .env và storage.

# 4. Sử dụng
> Để có thể deploy được code 1 ứng dụng đơn giản lên 1 server thì mọi việc khá dễ dàng.
> Chỉ cần đảm bảo 1 vài thuộc tính input trong deploy.php như sau:
- repository: repo github để clone code.
- branch: xác định branch lấy code(mặc định lấy tên branch theo local).
- host: ip/host, user, deploy_path.

Chạy task deploy bằng command:
```bash
dep deploy
```
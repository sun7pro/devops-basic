Trong phần này, chúng ta cùng thảo luận cách deploy một ứng dụng lên một remote server. Phần này sẽ là tiếp nối bạn cần làm sau khi đã cấu hình php-fpm, apache2 hoặc nginx trên remote server.

> Trong phạm vi bài viết, tôi sử dụng ứng dụng Laravel và remote server là EC2 làm demo

Công việc trong phần deployment này sẽ bao gồm 3 phần chính

-   Kết nối tới remote server qua giao thức ssh
-   Pull source code mới nhất về remote server này
-   Tiến hành build ứng dụng bao gồm cài đặt các package, module cần thiết. Tiếp theo là caching và tối ưu cho ứng dụng.

# Trước khi bắt đầu

Để thực hiện được việc deploy qua các công cụ hay xây dựng các ứng dụng phục vụ việc deploy, trước tiên các bạn cần biết các bước thủ công để deploy 1 ứng dụng ra sao.

> Note: Các bước cấu hình php-fpm và nginx cho ứng dụng có thể tham khảo tại. Trong phạm vi bài viết, chúng tôi coi như bạn đã setup máy chủ thành công, công việc deploy chỉ là cập nhật mã nguồn và build mã nguồn mới nhất của ứng dụng.

## Bước 1: Kết nối với remote server

Trước tiên bạn cần có 1 hosting hoặc máy chủ cá nhân ảo, tiếng Anh là Virtual Private Server (VPS). VPS hosting là 1 trong các dịch vụ hosting phổ biến nhất để ban có thể sử dụng làm nền tảng thực hiện việc triển khai ứng dụng.

Hiện có rất nhiều các nhà cung cấp dịch vụ VPS như các cloud, có thể kể đến như Amazon với dịch vụ EC2, Google Cloud với Cloud Computing Services đều là các dịch vụ tốt phục vụ công việc của bạn.

Khi đã có một máy chủ cloud cho riêng mình, bạn cần phải thực hiện kết nối remote vào máy chủ này. Giao thức chủ yếu được sử dụng là ssh.

Kết nối với một remote server với `user` là `username`:

```bash
ssh -a username@remote_server
```

## Bước 2: Pull source code mới nhất

Khi đã ssh được trên server, tiếp theo chúng ta sẽ tiến hành triển khai mã nguồn trên chính server này.

Đầu tiên là cập nhật code trên server là mới nhất qua lệnh git.

```bash
 # Di chuyển vào thư mục source code
cd /var/www/html/project_batch

# Checkout code sang nhánh bạn muốn deploy
git checkout branch_name

# Pull code mới nhất về
git pull remote_name branch_name
```

## Bước 3: Thực hiện các lệnh build ứng dụng

Với một ứng dụng laravel, deploy một ứng dụng cơ bản cần sử dụng các lệnh build sau:

```bash
# Build ứng dụng, cài đặt php modules, packages
composer install

# Build JS, CSS Assets

## Install javascript, css module
npm install
## Run all Mix tasks and minify output...
npm run production

# Migrate the project
php artisan migrate

# Caching, optimize
php artisan optimize
```

Hmm, cũng không phức tạp lắm nhỉ, chỉ như vậy là chúng ta đã thực hiện các bước thành công để build, deploy đầy đủ code mới nhất cho ứng dụng.

Trên đây là các bước thủ công, bạn phải tự chạy đủ các lệnh trên mỗi lần deploy. Trong phần tiếp theo chúng ta sẽ xem xét làm sao để auto deploy các câu lệnh này, tránh việc chạy thủ công bằng cơm.

Let's start!
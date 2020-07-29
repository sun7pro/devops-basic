[Trong phần tổng quan](overview.md), tất cả các lệnh từ việc remote vào server cho đến cập nhật mã nguồn và build ứng dụng đều cần thực hiện thủ công. Trong phần tiếp theo, chúng ta hãy sử dụng shell script để thực hiện tất cả các công việc đó chỉ bằng 1 lệnh nhé :D.

# Shell script

Bạn có để ý rằng tất cả các lệnh ở trên đều là các shell scrpit nên việc đầu tiên ta nghĩ tới là có thể sử dụng các tệp bash để chạy các lệnh này.

## 80% công việc

Ý tưởng chung là bạn sẽ tìm cách viết được một shell script (`.sh`), tìm được câu lệnh cho phép bạn `ssh` đến remote server và thực thi các câu lệnh trên đó. Thật may mắn, các shell script cho phép chúng ta thực hiện công việc này.

### Cú pháp bash đơn giản để chạy nhiều commands trên remote server

Để kết nối ssh và chạy nhiều lệnh command 1, command 2 trên remote server, bạn có thể sử dụng cú pháp:

```bash
ssh bar@foo "command1 && command2"
```

Ví dụ bạn muốn xem thông tin `host` và `username`

```bash
ssh user@host "date && hostname"
```

### Chạy file script trên remote server thì sao?

Chúng ta tất nhiên là không muốn viết nhiều lệnh như `date && hostname` như trên mà thay vào đó, giải pháp sẽ là đưa chúng vào 1 file script chứa tất cả các lệnh muốn chạy trên deploy và sử dụng bash để chạy file này.

```bash
#!/bin/bash
# Name: test.sh
# Purpose: Run multiple commands on a remote box
# ----------------------------------------------------
uptime
date
whoami
```

Khi đó, để xem các thông tin trên remote server, chúng ta sử dụng lệnh sau:

```bash
ssh vivek@server1.cyberciti.biz 'bash -s' < /path/to/test.sh
```

Có vẻ không khó khăn lắm nhỉ. Chỉ với hơn chục dòng lệnh mà 80% công việc đã được giải quyết, công việc còn lại của chúng ta chỉ là chỉnh sửa file script thành các lệnh bạn cần sử dụng trên remote server để tiến hành deploy ứng dụng.

## Xây dựng shell script

Chúng ta sẽ xây dựng file `deploy.sh`, file chứa các lệnh để install các dependency, build ứng dụng.
```bash
#!/bin/bash
# Name: deploy.sh
# Purpose: Run multiple commands on a remote server to deploy our application
# ----------------------------------------------------
cd /var/www/html/our_project

git checkout develop

git pull sun develop

# Build ứng dụng
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

Cấp quyền cho thư file này để có thể thực thi:
```bash
chmod +x deploy.sh
```

Tiến hành deploy nào:
```bash
ssh -a user@host 'bash -s' < /path/to/deploy.sh
```

## Tạo alias

Bạn có thể tạo alias cho từng project để gõ lệnh 1 cách nhanh chóng.
```bash
sudo vi ~/.zshrc
```

Thêm nội dung vào file này

```bash
alias deploy-demo="ssh -a user@host 'bash -s' < /path/to/deploy.sh"
```

Như vậy chúng ta có thể deploy ứng dụng với lệnh đơn giản trên chính máy của bạn

```bash
deploy-demo
```

Bạn có thể xem thêm cách tạo `alias` với `bash` hoặc `zsh` tại [đây](https://viblo.asia/p/back-to-basic-linux-command-line-part-2-m68Z0MXNlkG#_tao-alias-7)
## Tùy chọn cho các tham số

Trong ví dụ trên các tham số như đường dẫn `thư mục deploy`, `tên nhánh` và `username` và `hostname` đang được cố định trong file. Bạn hoàn toàn có thể thay đổi tùy biến bằng cách để cho người dùng nhập tùy biến.

Ví dụ đơn giản sau sẽ nhận tham số tùy biến từ người dùng

```bash
echo Hello, who am I talking to?
read varname
echo It\'s nice to meet you $varname
```

Áp dụng cho project, tôi sẽ để người dùng nhập các thông tin bao gồm hostname, username, thư mục source và git branch sẽ thực hiện deploy

```bash
#!/bin/bash
# Name: index.sh
# Purpose: Deploy our application to remote server
# ----------------------------------------------------

GREEN='\033[0;32m'
NC='\033[0m' # No Color

# Ask the user for their name
echo What is your hostname \(or IP address\)?
read hostname

echo What is username on remote server?
read username

echo What is the project path?
read path

echo What is branch you want deploy?
read branch

printf "${GREEN}Wait a second, deployment is processing ......${NC}\n"

ssh -a ${username}@${hostname} "bash -s $path $branch" < /path/to/deploy.sh
```

Và file `deploy.sh` sẽ nhận các tham số như sau:

```bash
#!/bin/bash
# Name: deploy.sh
# Purpose: Run multiple commands on a remote server to deploy our application
# ----------------------------------------------------
set -e # Stopr script if one of the commands failed

GREEN='\033[0;32m'
NC='\033[0m' # No Color

print_info () {
    msg=$@
    printf "${GREEN}${msg}${NC}\n"
}

# $1: Source code patch
# $2: Source code branch

print_info "1. Checkout code"

cd $1

git checkout $2

git pull origin $2

print_info "2. Build the source code"
print_info "Composer install"
# Build ứng dụng
composer install

# Build JS, CSS Assets

print_info "3. Webpack building"
## Install js, css module
npm install

## Run all Mix tasks and minify output...
npm run production

# Migrate the project
print_info "4. Migrate database"
php artisan migrate
print_info "Migrate database successfully"

# Caching, optimize
print_info "5. Caching, optimize"
php artisan optimize

print_info "The deployment was successful!"
```

Như vậy chúng ta vừa hoàn thành 1 chương trình shell script đơn giản thực hiện deploy code.

# Tài liệu tham khảo

- [How to change the output color of echo in Linux](https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux)

- [How To Run Multiple SSH Command On Remote Machine And Exit Safely](https://www.cyberciti.biz/faq/linux-unix-osx-bsd-ssh-run-command-on-remote-machine-server/)

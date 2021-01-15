Sun* CI là một CI/CD tool cây nhà lá vườn, được phát triển bởi chính Sun Asterisk. Tất cả các dự án tại Sun* có thể áp dụng tool này mà không lo đến các vấn đề bảo mật, phụ thuộc vào bên thứ ba hoặc các chi phí liên quan đến CI/CD.

File cấu hình của Sun* CI được lấy cảm hứng từ GitLab CI, bởi vậy các tham số trong file cấu hình cũng có phần tương tự. Chúng ta cùng tìm hiểu chi tiết nhé.

Trước hết, cũng như các tool CI/CD khác, file cấu hình `.sun-ci.yml` định nghĩa cấu trúc và thứ tự các pipelines cho project và xác định
- Tất cả các jobs trong workspace
- Thứ tự stages (các giai đoạn)
- Các gì để thực khi, gọi là `Sun* CI Runner`

Dưới đây là một ví dụ trong file cấu hình cho một project Laravel sử dụng Sun* CI:
```yaml
workspace: true

stages:
  - build
  - test

jobs:
- name: build
  stage: build
  image: sunasteriskrnd/php-workspace:7.4
  services:
  - image: postgres:12-alpine
    name: postgres_test
    environment:
      POSTGRES_DB: xxx
      POSTGRES_USER: xxx
      POSTGRES_PASSWORD: xxx
  environment:
    APP_ENV: testing
  cache:
  - key: comopser_vendor_$CI_BRANCH
    paths:
      - vendor
  before_script:
  - cp .env.example .env.testing
  - composer install
  - php artisan key:generate
  - php artisan migrate
  script:
  - composer sniff
  - composer test
  after_script:
  - echo "Finish job"
  only:
    branches:
    - master
    events:
    - pull_request
  except:
    branches:
    - develop
    events:
    - push
  artifacts:
    paths:
    - app/
    expires_in: 3 days

- name: test:node
  stage: test
  image: node:12-alpine
  script:
  - yarn
  - yarn lint
  ```


# Chi tiết các tham số
## stage
`stage` được định nghĩa cho mỗi job và dựa vào từ khóa `stages` định nghĩa toàn cục. Nó cho phép nhóm các jobs thành các stages khác nhau và các jobs trong cùng 1 stage được thực thi song song (tùy thuộc vào điều kiện nhất định).

Hãy nhìn lại ví dụ trên cho phần `stages`
```yaml
stages:
  - build
  - test
```
Như vậy project đang sử dụng 2 stages và thứ tự chạy sẽ là `build` đầu tiên, nếu build thực hiện thành công sẽ chạy stage `test`. Trong một stage `test` bạn có thể định nghĩa nhiều các jobs con, ví dụ như bạn cần `test` cho `php` và `test` cho cả `javascript`, 2 jobs này rõ ràng là độc lập nên hoàn toàn có thể chạy song song. Để làm việc này chúng ta sử dụng 2 tham số `name` và `stage` sẽ được đề cập ở phần sau.
```yaml
stages:
  - build
  - test
jobs
  - name: test:php
    stage: test
    ...
  - name: test:js
    stage: test
```

## images
Được sử dụng để chỉ định một Docker image làm môi trường cho jobs
```yaml
image: ruby:2.6
```
## services
Được sử dụng để chỉ định một Docker image, được liên kết tới 1 base image trong `image`
- `services.*.name`: định nghĩa tên của service
```yaml
jobs:
  - name: build
    stage: build
    image: sunasteriskrnd/php-workspace:7.4
    services:
    - image: postgres:12-alpine
      name: postgres_test
      environment:
        POSTGRES_DB: xxx
        POSTGRES_USER: xxx
        POSTGRES_PASSWORD: xxx
```
## script, before_script and after_script
- `script`: từ khóa bắt buộc, định nghĩa các câu lệnh chạy trong 1 job. Nó là các shell script được thực thi bởi Runner.
- `before_script` được sử dụng để định nghĩa tập các lệnh nên được chạy trước mỗi job.
- `after_script` được sử dụng để định nghĩa tập các lệnh các lệnh sẽ được chạy mỗi job bao gồm cả những job lỗi.
```yaml
- name: build
  stage: build
  image: sunasteriskrnd/php-workspace:7.4
  before_script:
    - cp .env.example .env.testing
  script:
    - composer install
  after_script:
    - php artisan key:generate
```
## cache
`cache` được sử dụng để xác định tập các files, thư mục bên được cache lại giữa các jobs. Bạn chắc chắn sẽ cần cache lại các gói phụ thuộc (các file trong thư mục `vendor` hay `node_modules`), để không phải cài lại chúng quá nhiều lần.
```yaml
- name: build
  stage: build
  image: sunasteriskrnd/php-workspace:7.4
  before_script:
    - cp .env.example .env.testing
  script:
    - composer install
  cache:
    - key: comopser_vendor_$CI_BRANCH
      paths:
        - vendor
```
- `cache.*.paths`: tệp hoặc thư mục nào sẽ được cache.
- `cache.*.key`: định nghĩa quan hệ của cache, cho phép bạn có thể có 1 single cache cho tất cả các jobs, cache cho mỗi job hay cache cho mỗi branch. Điều đó phụ thuộc vào các [biến định nghĩa trước](https://ci.sun-asterisk.com/docs/guide/variables/predefined.html) của Sun\* CI.

## environment
`environment` định nghĩa các biến môi trường được sử dụng trong job
```yaml
jobs:
  - name: build
    stage: build
    image: sunasteriskrnd/php-workspace:7.4
    environment:
      APP_ENV: testing
...
```

## only / except
Đây là hai tham số định nghĩa khi nào các jobs sẽ chạy
- `only` định nghĩa tên nhánh và tags job sẽ chạy
- `except` định nghĩa tên nhánh và tags job sẽ không chạy
```yaml
jobs:
  - name: build
    stage: build
    image: sunasteriskrnd/php-workspace:7.4
    only:
      branch:
        - develop
        - master
      events:
        - push
    except:
      branch:
        - branch_1
        - branch_2
      events:
        - pull_request
```
## artifacts
`artifacts` được sử dụng để xác định tập các files, thư mục được đính kèm trong job
```yaml
jobs:
- name: Build php
  stage: build
  image: sunasteriskrnd/php-workspace:7.4
  script:
  - composer sniff
  - composer test
  artifacts:
    paths:
    - app/
    expires_in: 3 days
```
Hết rồi. 

Các tham số cấu hình trong Sun \*CI khá đơn giản, bản thân tên của chúng cũng đã rõ cho mục đích sử dụng. Hi vọng với bài viết ngắn gọn này, bạn hoàn toàn sẵn sàng áp dụng Sun \*CI tool cho project của mình.

# Tài liệu tham khảo
- [Sun \*CI Document](https://ci.sun-asterisk.com/docs/guide/yaml/#configuration-example)
- [Config tham khảo cho project Laravel](https://github.com/minhnv2306/aws-training/pull/9)

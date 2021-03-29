# Tài liệu hướng dẫn Devops cơ bản từ Group 7Pro - Sun Asterisk
## 1. Giới thiệu
1.1. Mục đích

Tài liệu được xây dựng trước tiên là tham chiếu cho các thành viên trong group, áp dụng các công cụ, kĩ thuật trong DevOps cơ bản vào các dự án. Trong phần cơ bản, chúng tôi muốn giới thiệu:
- Tổng quan về DevOps, đây là một văn hóa, một chức danh hay công việc?
- Tài liệu và các cấu hình chung cho các CI/CD tools: Sun CI, CircleCI, Github action, áp dụng cho Laravel project.
- Tài liệu một số dịch vụ Cloud thông dụng: AWS
- Tài liệu về SSH và một số phần liên quan đến ứng dụng web: LAMP, LEMP
- Tài liệu cho một số dịch vụ theo dẽo, giám sát project.

1.2. Các kỹ thuật và công cụ tìm hiểu

CI/CI tools:
+ [x] [CircleCI](./docs/cicd/circle-ci/README.md)
+ [ ] Github Action
+ [ ] Sun*CI
+ [ ] Jenkins
+ [ ] Gitlab CI

Cloud Services:
+ [ ] Amazon Web Service (AWS)
+ [ ] Google Cloud Platform (GCP)

Operation system (OS)
+ [ ] Ubuntu 18.04
+ [ ] CentOS 7

Deploy tools
+ [x] [Deployment overview](./docs/cicd/overview.md)
+ [x] [Shell script](./docs/cicd/shell-script.md)
+ [ ] PHP appilcation
+ [ ] NodeJs application
+ [ ] <strong>[Tool]</strong>: Rocketeer
+ [ ] <strong>[Tool]</strong>: Deployer

Configuration management
+ [ ] Ansible

## 2. Nội dung chi tiết

- [ ] [DevOps là gì? Văn hóa? Hay một chức danh công việc?...](./docs/intro/README.md)
- [ ] Tạo môi  trường thực hành sử dụng sshd với docker
  + [x] Ubuntu server, [hướng dẫn thực hành](./docs/ssh/ubuntu-test-server.md)
  + [ ] CentOS server
  + [ ] AmazonLinux server
- [x] [SSH connect guide](./docs/ssh/README.md)
- [ ] [Linux, process management](./docs/linux/README.md)
- [ ] Hướng dẫn cài đặt môi trường cơ bản cho test server: [lamp stack](./docs/lamp/README.md), [lemp stack](./docs/lemp/README.md)
- [ ] Deploy tools
  + [x] [Deployment overview](./docs/cicd/overview.md)
  + [x] [Shell script](./docs/cicd/shell-script.md)
  + [ ] PHP appilcation
  + [ ] NodeJs application
  + [ ] <strong>[Tool]</strong>: Rocketeer
  + [ ] <strong>[Tool]</strong>: Deployer
- [ ] [Áp dụng CI, CW notification](./docs/cicd/README.md)
  + [x] [CircleCI](./docs/cicd/circle-ci/README.md)
  + [ ] Github Action
  + [ ] Sun*CI
  + [ ] Jenkins
  + [ ] Gitlab CI
- [ ] Configuration management
  + [ ] [Ansible](./docs/ansible/README.md)
- [ ] [Tìm hiểu và áp dụng Terraform for AWS, GCP, Azure](./docs/terraform/README.md)
- [ ] [Tìm hiểu về monitoring](./docs/monitoring/README.md)
  + [ ] Prometheus
  + [ ] Grafana
  + [ ] Zabbix
- [ ] [AWS Labs](./docs/aws-labs/README.md)

Happy learning!

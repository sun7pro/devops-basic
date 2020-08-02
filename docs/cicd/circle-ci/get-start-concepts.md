Hướng dẫn dưới đây sẽ giới thiệu 1 vài khái niệm cơ bản để giúp bạn hiểu CircleCI quản lý pipelines CICD như thế nào.

- [Projects](#projects)
- [Configuration](#configuration)
- [User Types](#user-type)
- [Pipelines](#pipelines)
- [Orbs](#orbs)
- [Jobs](#jobs)
- [Executors and Images](#executors-and-images)
- [Steps](#steps)
- [Workflows](#workflows)
- [Caches, Workspaces and Artifacts](#caches-workspaces-and-artifacts)

# Projects
Một project CircleCI chia sẻ tên các repositories trên VCS của bạn (Github hoặc Bitbucket). Lựa chọn Add Project từ CircleCI application để vào Projects dashboard, nơi mà bạn có thể set up và follow được những project bạn có quyền truy cập.

Trên Projects Dashboard, bạn có thể:
- *Set Up* bất kỳ project nào bạn là chủ (owner) của VCS
- *Follow* bất kỳ project nào trong cách tổ chức của bạn để có thể truy cập vào các pipeline và theo dõi thông báo email đối với các trạng thái của project.

![](https://circleci.com/docs/assets/img/docs/CircleCI-2.0-setup-project-circle101_cloud.png)

# Configuration
CircleCI coi *cấu hình như là code (configuration as code)*.  Tất cả các quá trình CI, CD của bạn sẽ được sắp xếp thông qua 1 file có tên là `config.yml`. File `config.yml` nằm trong thư mục `.circleci` ở root của project. CircleCI sử dụng cú pháp YAML file cho config của mình. `circle.yml` là file đầy sức mạnh để định nghĩa các toàn bộ `pipelines` trong project.
```
├── .circleci
│   ├── config.yml
├── README
└── all-other-project-files-and-folders
```
Các tùy chọn trong cấu hình, bạn có thể tham khảo tại [đây](https://circleci.com/docs/2.0/configuration-reference/)

Cấu hình CircleCI của bạn có thể được điều chỉnh để phù hợp với nhiều nhu cầu khác nhau của dự án của bạn. Các khái niệm sau đây, được sắp xếp theo thứ tự độ chi tiết và phụ thuộc, mô tả các thành phần của hầu hết các dự án CircleCI chung:
- **Pipeline**: Nó đại diện cho toàn bộ cho phần cấu hình của bạn
- **Workflows**: Nhiệm vụ của nó là điều phối nhiều *jobs*
- **Jobs**: Nhiệm vụ của nó là chạy 1 loạt *steps*, cái mà thực thi các commands
- **Steps**: Chạy các commands (như là cài đặt các dependencies hoặc chạy tests) và shell scripts để thực hiện các công việc được yêu cầu cho project.

Đây là 1 ví dụ sử dụng Java application đối với rất nhiều thành phần trong cấu hình:

![](https://circleci.com/docs/assets/img/docs/config-elements.png)

# User Types
Chúng ta cùng phân loại người dùng liên quan đến các projects CircleCI, hầu hết trong số đó đều có quyền kế thừa từ tài khoản VSC. Từ kế thừa ở đây có thể hiểu là các type tương tự từ bên VSC sang.

- **Organization Administrator** là mức level được kế thừa từ VCS:
    - GitHub: **Owner** và theo dõi ít nhất một dự án building trên CircleCI
    - Bitbucket: **Admin** và theo dõi ít nhất một dự án building trên CircleCI
- **Project Administrator** là người thêm một repository Github hoặc Bitbucket vào CircleCI như 1 Project.
- **User** là 1 người dùng cá nhân trong 1 tổ chức, kế thừa từ VCS
- Một CircleCI user là bất cứ ai có thể đăng nhập vào CircleCI platform với username và password. Những người dùng phải được thêm vào GitHub hoặc Bitbucket để xem và theo dõi các CircleCI projects liên quan. Những người dùng có thể không xem được project data, data được lưu trữ trong các biến môi trường.

## Pipelines

Một pipeline CircleCI là tập đầy đủ quá trình bạn chạy khi bạn kích hoạt công việc trên các projects. Các pipelines bao gồm các workflow cái mà được điều phối các jobs. Tất cả được định nghĩa trong [file cấu hình](https://circleci.com/docs/2.0/concepts/#config)

Pipeline có 1 vài cách biểu thị để tương tác với file cấu hình của bạn:
- Sử dụng các endpoint API mới để [kích hoạt 1 pipeline](https://circleci.com/docs/api/v2/#trigger-a-new-pipeline)
- Sử dụng các đối số để kích hoạt các [workflows có điều kiện](https://circleci.com/docs/2.0/pipeline-variables/#conditional-workflows)
- Đối với `version 2.1`, nó cung cấp
	+ Tái sử dụng các elements cấu hình, bao gồm excecutors, commands và jobs
	+ Cấu hình các package tái sử dụng, được biết đến như orbs
	+ Cải thiện các thông báo lỗi validation cấu hình
	+ Tùy chọn khả năng auto-cancel, với **Advanced Setting**, để hủy bỏ quy trình workflows khi các build mới được kích hoạt trên các nhánh không xác định

Note: Thực sự quan trọng để xem xét cẩn thận sự ảnh hưởng chức năng auto-cancel, ví dụ, nếu bạn có các cấu hình tự động deploy trên các nhánh không xác định.

Để biết chi tiết hơn về pipelines và làm sao bạn có thể sử dụng các thuộc tính của chúng trong workflows và jobs, hãy xem thêm các hướng dẫn sau:

- [Transitioning to Pipelines](https://circleci.com/docs/2.0/build-processing/#transitioning-to-pipelines)
- [Viewing Pipelines in the UI](https://circleci.com/docs/2.0/pipelines/#overview)
- [Pipeline Variables](https://circleci.com/docs/2.0/pipeline-variables/)

# Orbs

Orbs là các đoạn trích (snippets) của code có thể tái sử dụng, cái mà giúp tự động thực hiện các quá trình lặp lại, tăng tốc độ setup project và làm cho tích hợp với tools bên thứ 3 dễ dàng hơn. Xem các [Using Orbs](https://circleci.com/docs/2.0/using-orbs/) để biết chi tiết sử dụng orbs như thế nào trong config và giới thiệu để thiết kế orb. Trang [này](https://circleci.com/orbs/registry/) cung cấp tìm kiếm các orbs để giúp config của bạn đơn giản hơn.

Biểu đồ dưới đây minh họa 1 ví dụ Java trở nên đơn giản hơn rất nhiều khi sử dụng orbs. Ví dụ minh hoạt sử dụng lại [Maven orb](https://github.com/CircleCI-Public/circleci-demo-java-spring/tree/2.1-orbs-config):

![](https://circleci.com/docs/assets/img/docs/config-elements-orbs.png)

# Jobs

Jobs là các blocks building trong config của bạn. Jobs là tập các steps cái mà chạy command/scripts yêu cầu. Mỗi job phải xác nhận, khai báo với một excecutor cái mà là một trong `docker`, `machine`, `windows` hoặc `macos`. `machine` bao gồm các image mặc định nếu không xác định, `docker` thì bạn phải chỉ ra primary image, với `macos` bạn phải xác định Xcode version và với `windows` bạn phải sử dụng Windows orb.

![](https://circleci.com/docs/assets/img/docs/job.png)

# Executors and Images

Mỗi job riêng biệt được định nghĩa trong config sẽ chạy 1 executor duy nhất. Một executor có thể là một docker container hoặc một virtual machine chạy Linux, Windows hoặc MacOS. Chú ý rằng, macOS hiện không có sẵn trên các self-hosted cài đặt của CircleCI Server.

![](https://circleci.com/docs/assets/img/docs/executor_types.png)

Bạn có thể định nghĩa một image cho mỗi executor. Một image là một system đã được đóng gói, cái mà có nhiều lệnh để tạo và chạy container hoặc máy ảo. CircleCI cung cấp một tập các images để sử dụng với Docker executor. Để biết thêm thông tin, vui lòng xem hướng dẫn [Pre-Built CircleCI Docker Images](https://circleci.com/docs/2.0/circleci-images/)

```yaml
version: 2.1

jobs:
 build1: # job name
   docker: # Specifies the primary container image,
     - image: buildpack-deps:trusty
     - image: postgres:9.4.1 # Specifies the database image
      # for the secondary or service container run in a common
      # network where ports exposed on the primary container are
      # available on localhost.
       environment: # Specifies the POSTGRES_USER authentication
        # environment variable, see circleci.com/docs/2.0/env-vars/
        # for instructions about using environment variables.
         POSTGRES_USER: root
#...
 build2:
   machine: # Specifies a machine image that uses
   # an Ubuntu version 14.04 image with Docker 17.06.1-ce
   # and docker-compose 1.14.0, follow CircleCI Discuss Announcements
   # for new image releases.
     image: ubuntu-1604:201903-01
#...       
 build3:
   macos: # Specifies a macOS virtual machine with Xcode version 11.3
     xcode: "11.3.0"
# ...   
```

Primary Container được định nghĩa bởi image đầu tiên được liệt kê trong file `.circleci/config.yml`. Đây là nơi các lệnh được thực thi. Docker executor sẽ tạo ra 1 container với Docker image. Machine executor sẽ tạo ra 1 image máy ảo Ubuntu hoàn chỉnh. Xem thêm [Choosing an Executor Type](https://circleci.com/docs/2.0/executor-types/) để so sánh và cân nhắc sử dụng Container hay máy ảo. Các images tiếp theo có thể được thêm để tạo ra các Secondary/Service Containers.

Khi sử dụng docker executor và chạy các lệnh docker, key `setup_remote_docker` có thể được sử dụng để tạo ra (spin up) các docker container khác chạy các lệnh này, để tăng cường bảo mật. Chi tiết có thể xem thêm tại [Running Docker Commands](https://circleci.com/docs/2.0/building-docker-images/#accessing-the-remote-docker-environment)

# Steps

Steps là các hành động cần để hoàn thành job. Steps thường là tập các lệnh có khả năng thực thi. Ví dụ, step `checkout` sẽ là *built-in* step có sẵn trong toàn bộ CircleCI projects, check out source code cho một job qua SSH. Sau đó `run` step sẽ cho phép bạn chạy các lệnh tùy biến như thực thi `make test` sử dụng một shell mặc định không cần đăng nhập theo mặc định. Các lệnh cũng có thể định nghĩa [bên ngoài khai báo job](https://circleci.com/docs/2.0/configuration-reference/#commands-requires-version-21) để có thể sử dụng lại trong config của bạn. 
```yaml
#...
jobs:
  build:
    docker:
      - image: <image-name-tag>
    steps:
      - checkout # Special step to checkout your source code
      - run: # Run step to execute commands, see
      # circleci.com/docs/2.0/configuration-reference/#run
          name: Running tests
          command: make test # executable command run in
          # non-login shell with /bin/bash -eo pipefail option
          # by default.
#...
```


# Workflows

Workflows định nghĩa tập các jobs và thứ tự chạy của chúng. Nó có thể chạy các jobs theo thứ tự, đồng thời, theo schedule hoặc với một xử lý thủ công sử khi một job hoàn tất.

![](https://circleci.com/docs/assets/img/docs/workflow_detail_newui.png)

Ví dụ dưới đây chỉ ra workflow được gọi `build_and_test` chạy build1 và sau đó các jobs `build2` và `build3` chạy đồng thời
```yaml
version: 2.1

jobs:
  build1:
    docker:
      - image: circleci/ruby:2.4-node
      - image: circleci/postgres:9.4.12-alpine
    steps:
      - checkout
      - save_cache: # Caches dependencies with a cache key
          key: v1-repo-{{ .Environment.CIRCLE_SHA1 }}
          paths:
            - ~/circleci-demo-workflows
      
  build2:
    docker:
      - image: circleci/ruby:2.4-node
      - image: circleci/postgres:9.4.12-alpine
    steps:
      - restore_cache: # Restores the cached dependency.
          key: v1-repo-{{ .Environment.CIRCLE_SHA1 }}
      - run:
          name: Running tests
          command: make test
  build3:
    docker:
      - image: circleci/ruby:2.4-node
      - image: circleci/postgres:9.4.12-alpine
    steps:
      - restore_cache: # Restores the cached dependency.
          key: v1-repo-{{ .Environment.CIRCLE_SHA1 }}
      - run:
          name: Precompile assets
          command: bundle exec rake assets:precompile
#...                          
workflows:
  build_and_test: # name of your workflow
    jobs:
      - build1
      - build2:
          requires:
           - build1 # wait for build1 job to complete successfully before starting
           # see circleci.com/docs/2.0/workflows/ for more examples.
      - build3:
          requires:
           - build1 # wait for build1 job to complete successfully before starting
           # run build2 and build3 concurrently to save time.
```

## Caches, Workspaces and Artifacts

![](https://circleci.com/docs/assets/img/docs/workspaces.png)

Một cache lưu trữ một file hoặc thư mục như các dependencies hoặc mã nguồn trong kho lưu trữ dự án. Mỗi job có thể chứa các step riêng biệt cho cache dependencies từ các jobs trước đó để tăng tốc độ build.

```yaml
version: 2.1

jobs:
  build1:
    docker: # Each job requires specifying an executor
    # (either docker, macos, or machine), see
    # circleci.com/docs/2.0/executor-types/ for a comparison
    # and more examples.
      - image: circleci/ruby:2.4-node
      - image: circleci/postgres:9.4.12-alpine
    steps:
      - checkout
      - save_cache: # Caches dependencies with a cache key
      # template for an environment variable,
      # see circleci.com/docs/2.0/caching/
          key: v1-repo-{{ .Environment.CIRCLE_SHA1 }}
          paths:
            - ~/circleci-demo-workflows

  build2:
    docker:
      - image: circleci/ruby:2.4-node
      - image: circleci/postgres:9.4.12-alpine
    steps:
      - restore_cache: # Restores the cached dependency.
          key: v1-repo-{{ .Environment.CIRCLE_SHA1 }}      
```

Workspace là một cơ chế lưu trữ workflow-aware. Một workspace chứa dữ liệu duy nhất cho job, cái mà có thể cần thiết cho các jobs ở bên dưới (downstream). Mỗi workflow có một workspace tạm thời liên kết với nó. Workspace có thể được sử dụng để truyền các dữ liệu duy nhất build được của 1 job tới các jobs khác cùng workflow.

Các Artifacts lưu trữ dữ liệu sau khi một workflow được hoàn thành và có thể được sử dụng để lưu trữ lâu dài các đầu ra của quá trình build của bạn.

```yaml
version: 2.1

jobs:
  build1:
#...   
    steps:    
      - persist_to_workspace: # Persist the specified paths (workspace/echo-output)
      # into the workspace for use in downstream job. Must be an absolute path,
      # or relative path from working_directory. This is a directory on the container which is
      # taken to be the root directory of the workspace.
          root: workspace
            # Must be relative path from root
          paths:
            - echo-output

  build2:
#...
    steps:
      - attach_workspace:
        # Must be absolute path or relative path from working_directory
          at: /tmp/workspace
  build3:
#...
    steps:
      - store_artifacts: # See circleci.com/docs/2.0/artifacts/ for more details.
          path: /tmp/artifact-1
          destination: artifact-file
#...
```
Theo dõi bảng dưới đây để xem những điểm khác biệt giữa Artifacts, Workspaces, and Caches


| Type | Lifetime | Use | Example |
| -------- | -------- | -------- | -------- |
| Artifacts     | Nhiều tháng     | Giữ các Artifacts lâu dài     | Có sẵn trong Artifacts tab của **Job page** dưới  `tmp/circle-artifacts.<hash>/containe` hoặc đường dẫn tương tự   |
| Workspaces     | Thời gian của workflow     | Đính kèm vào workspace trong container phía dưới với step `attach_workspace`    | `attach_workspace`  sao chép và tạo lại toàn bộ nội dung workspace khi nó chạy   |
| Caches     | Nhiều tháng     | Lưu trữ các dữ liệu ngắn (non-vital) có thể giúp các job chạy nhanh hơn, ví dụ npm hoặc Gem packages     | Bước `save_cache` với một `path` chứa danh sách thư mục cache lại với `key` duy nhất (`key` có thể là branch, build number hoặc sửa đổi version). Sử dụng cache với  bước `restore_cache` và `key` tương ứng    |

Đọc thêm [Persisting Data in Workflows: When to Use Caching, Artifacts, and Workspaces](https://circleci.com/blog/persisting-data-in-workflows-when-to-use-caching-artifacts-and-workspaces/) để biết thêm về các khái niệm và sử dụng workspaces, caching, và artiffacts.

# Tài liệu tham khảo

[Getting Started Concepts](https://circleci.com/docs/2.0/concepts/)

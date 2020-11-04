Caching là một trong những cách hiệu quả nhất để làm các jobs nhanh hơn trên CircleCI bằng cách sử dụng lại data từ cách "tải về rất đắt" từ các jobs trước đó.

![](https://circleci.com/docs/assets/img/docs/caching-dependencies-overview.png)

Caching đặc biệt hữu ích với các công cụ **quản lý sự phụ thuộc các package** như Yarn, Bundler hoặc Pip. Khi các dependencies  được lưu trữ trong cache, lệnh như `yarn install` sẽ chỉ cần tải về các dependencies mới, nếu không có, không cần download lại gì cho mỗi lần build.

> Warning: Cache files giữa các executors khác nhau, ví dụ như Docker and Machine, Linux, Windows or MacOS, hoặc CircleCI Image and Non-CircleCI Image có thể dẫn đến lỗi quyền truy cập file và lỗi đường dẫn. Những lỗi này thường do không tìm thấy users, users với UIDs khác và đường dẫn không có.  Hãy cẩn thận và kiểm tra kĩ sử dụng caching files trong các trường hợp này.

# Example Caching Configuration
Khá đơn giản để config cache. Ví dụ dưới đây cập nhật 1 cache nếu nó thay đổi bằng cách sử dụng checksum của file `pom.xml`:
```yaml
    steps:
      - restore_cache:
         keys:
           - m2-{{ checksum "pom.xml" }}
           - m2- # used if checksum fails
```

# Introduction
Tự động caching dependency không có sẵn trên CircleCI 2.0, do đó rất quan trọng để lên kế hoạch và thực thi chiến lược caching một cách tốt nhất. Bạn phải thực hiện config thủ công trong CircleCI 2.0 để có thể có chiến lược và kiểm soát tốt hơn.

Phần tài liệu này sẽ mô tả cách có thể sử dụng caching thủ công, giá và các lợi ích của các chiến lược được chọn, một vài tips để tránh cách vấn đề với caching. **Chú ý rằng:** Docker images được sử dụng cho CircleCI 2.0 job được tự động cache trên hạ tầng server nếu có thể.

Để biết thêm thông tin về tính năng cao hơn để tái sử dụng các layers của Docker image, xem thêm tại [Enabling Docker Layer Caching
](https://circleci.com/docs/2.0/docker-layer-caching/)

# Overview

Một cache lưu trữ một hệ thống cấp bậc của file dưới một key. Sử dụng cache để lưu trữ dữ liệu làm job của bạn nhanh hơn nhưng trong trường hợp không tìm được cache hoặc khôi phục cache bằng 0, job vẫn chạy thành công. Ví dụ bạn có thể cache `NPM` đường dẫn package (như `node_modules`) trong lần đầu bạn chạy job và nó sẽ download toàn bộ dependencies của bạn, sau đó cache chúng và - miễn là cache của bạn hợp lệ - cache này sẽ được sử dụng để tăng tốc độ cho job trong lần chạy tiếp theo.

Bộ nhớ đệm là sự cân bằng giữa độ tin cậy (không sử dụng bộ đệm lỗi thời hoặc không phù hợp) và đạt hiệu suất tối đa (sử dụng bộ đệm đầy đủ cho mỗi bản dựng).

Nói chung, an toàn hơn để bảo vệ độ tin cậy hơn so với rủi ro một bản build bị lỗi hoặc tạo ra bản build rất nhanh chóng bằng cách sử dụng các phụ thuộc cũ. Vì vậy, lý tưởng là cân bằng hiệu suất đạt được trong khi vẫn duy trì độ tin cậy cao.

> Hãy sử dụng cache để vừa đạt độ tin cậy (không cache mọi thứ, đảm bảo các denpendencies luôn là mới nhất) vừa cân bằng hiệu suất cao
# Caching Libraries
Các dependencies quan trọng nhất đối với bộ đệm trong công việc là các thư viện mà dự án của bạn phụ thuộc vào. Ví dụ cache các thư viện được cài với `pip` trong Pythin hoặc `npm` hoặc Node.js. 

Các công cụ không được yêu cầu rõ ràng cho dự án của bạn được lưu trữ tốt nhất trên Docker images. Các Docker images được xây dựng trước bởi CircleCI có các công cụ được cài đặt sẵn chung cho việc xây dựng các dự án sử dụng ngôn ngữ mà hình ảnh được tập trung vào. Ví dụ: image `Circleci / ruby: 2.4.1` có các công cụ hữu ích như git, openssh-client và gzip được cài đặt sẵn.

![](https://circleci.com/docs/assets/img/docs/cache_deps.png)

# Writing to the Cache in Workflows
Các jobs trong một workflow có thể chia sẻ cache. Lưu ý điều này cho phép tạo ra các điều kiện trong cache trên các jobs khác nhau trong các workflows.

**Cache bất biến khi ghi**: một chi cache được ghi bởi một key cụ thể như `node-cache-master`, nó không thể được ghi lại. Xem xét một workflow của 3 jobs, Job3 phụ thuộc vào Job1 và Job2: {Job1, Job2} -> Job3. Tất cả chúng được đọc và ghi với cache key giống nhau.

Khi chạy workflow, Job3 có thể sử dụng cache được ghi bởi Job1 hoặc Job2. Do cache là bất biến, điều đó sẽ đọc ở bất kì job nào được lưu trữ cache đầu tiên. Điều này thường không thể chấp nhận được vì kết quả không mang tính quyết định - một phần của kết quả phụ thuộc. Bạn có thể làm cho workflow này xác định bằng cách thay đổi dependencies của job: làm cho Job1 và Job2 ghi vào cache theo cách khác nhau và Job3 sẽ chọn từ chỉ một hoặc chắc chắn rằng chỉ có một thứ tự sắp xếp: Job1 -> Job2 ->Job3.

Có nhiều trường hợp phức tạp hơn, khi mà jobs có thể lưu sử dụng một ket động như `node-cache-{{ checksum "package-lock.json" }}` và khôi phục cache sử dụng một phần của key trùng với `node-cache-`.  Khả năng cho một điều kiện vẫn tồn tại nhưng chi tiết có thể thay đổi. Ví dụ, downstream job sử dụng cache từ upstearm job chạy cuối cùng.

Một điều kiện khác có thể khi chia sẻ cache giữa các jobs. Xem xét trường hợp 1 workflow không có liên kết phụ thuộc: Job1 and Job2. Job2 sử dụng cache được lưu từ Job1. Job2 có thể đôi khi khôi phục một cache thành công và đôi khi không tìm thấy, mặc dù Job1 đã lưu nó. Job2 có thể cũng lấy cache từ workflow trước đó. Nếu điều đó xảy ra, điều đó có nghĩa Job2 đã cố load cache trước khi Job1 lưu nó. Điều này có thể được giải quyết bằng cách tạo ra sự phụ thuộc trong workflow: Job1->Job2. Điều này bắt buộc Job2 phải chờ cho đến khi Job1 kết thúc.

# Restoring Cache
CircleCI khôi phục cache theo thứ tự danh sách key được liệt kê trong  bước `restore_cache`. Mỗi key cache được đặt tên theo project và được truy xuất dựa vào trùng tiền tố (prefix-matched). Cache được khôi phục từ trùng key đầu tiên. Nếu có nhiều sự trùng khớp cache được tạo gần nhất sẽ được sử dụng.

Ví dụ dưới đây, 2 keys được cung cấp:
```yaml
    steps:
      - restore_cache:
          keys:
            # Find a cache corresponding to this specific package-lock.json checksum
            # when this file is changed, this key will fail
            - v1-npm-deps-{{ checksum "package-lock.json" }}
            # Find the most recently generated cache used from any branch
            # Trong trường hợp v1-npm-deps-abc và v1-npm-deps-xyz đều có, nó sẽ khôi phục từ 1 trong 2, chọn cái tạo gần nhất
            - v1-npm-deps-
```
Vì key thứ 2 ít rõ ràng hơn key đầu, nhiều khả năng sẽ có sự khác biệt giữa trạng thái hiện tại và cache được tạo gần đây nhất. Khi một tool phụ thuộc chạy, nó sẽ phát hiện ra các dependencies lỗi và cập nhật chúng. Điều này được gọi là **khôi phục cache từng phần**

Hmm, hãy nói chi tiết hơn nào:

Mỗi dòng trong  `keys`: danh sách tất cả quản lý *one cache* (mỗi dòng không tương ứng với cache riêng của nó). Danh sách keys `(v1-npm-deps-{{ checksum "package-lock.json" }} and v1-npm-deps-)` trong ví dụ này đại diện cho cache đơn. Khi đến lúc khôi phục bộ đệm, CircleCI trước tiên xác thực bộ đệm dựa trên khóa đầu tiên (và cụ thể nhất), sau đó bước qua các keys khác để tìm bất kỳ thay đổi nào về khóa bộ đệm.

Ở đây, khóa đầu tiên là hàm nối checksum của file `package-lock.json` và chuỗi `v1-npm-deps-`, nếu file này thay đổi trong commit của bạn, CircleCI sẽ nhìn thấy 1 cache-key mới

Khóa tiếp theo không chứa thành phần động, đơn giản chỉ là chuỗi tĩnh: `v1-npm-deps-`. Nếu bạn muốn làm mất hiệu lực cache thủ công, bạn có thể chuyển từ `v1` thành `v2` trong file `config.yml` của bạn. Trong trường hợp này, bạn có một cache mới với key `v2-npm-deps` được lưu trữ

# Managing Caches
## Cache Expiration

Cache được tạo ra từ `save_cache` step sẽ được lưu trữ trong vòng 15 ngày
## Clearing Cache
Nếu bạn cần nhận được dọn dẹp cache khi phiên bản công cụ quản lý ngôn ngữ hoặc dependency của bạn thay đổi, hãy sử dụng chiến lược đặt tên tương tự như ví dụ trước và sau đó thay đổi tên khóa bộ đệm trong file `config.yml` của bạn và cam kết thay đổi để xóa bộ đệm.

> Tip: Cache là bất biến do đó hữu ích là bắt đầu tất cả các cache keys của bạn với một tiền tố version, ví dụ như `v1-....`. Điều này có thể cho phép bạn sinh lại tất cả các che bằng cách tăng version trong tiền tố này.

Ví dụ bạn có thể muốn xóa cache trong các trường hợp sau bằng cách tăng tên cache key:
- Thay đổi phiên bản quản lý sự phụ thuộc, ví dụ từ 4 lên 5.
- Thay đổi phiên bản của ngôn ngữ, bạn có thể thay đôỉ từ ruby 2.3 lên 2.4.
- Dependencies được xóa trong project của bạn.

> Tip: Hãy cẩn thận khi sử dụng các ký tự đặc biệt trong key cache (ví dụ như :, ?, &, =, /, #) vì chúng có thể gây ra các lỗi khi build. Nói chung, hãy sử dụng keys với các kí tự `[a-z][A-Z]` trong tiền tố cache key.

## Cache Size
Chúng tôi khuyên rằng giữ kích thước cache dưới 500MB. Đây là giới hạn để kiểm tra cache nếu bạn không muốn thời gian check quá dài. Bạn có thể xem kích thước bộ đệm từ page CircleCI Jobs trong bước `restore_cache`. Kích thước cache lớn hơn được cho phép nhưng có thể gây ra sự cố do cơ hội giải nén và lỗi cao hơn trong quá trình tải xuống. Để giảm kích thước bộ đệm, hãy xem xét chia thành nhiều bộ đệm riêng biệt.

# Basic Example of Dependency Caching

Để lưu trữ cache của 1 file hoặc 1 đường dẫn, thêm `save_cache` step vào file `.circleci/config.yml`:
```yaml
    steps:
      - save_cache:
          key: my-cache
          paths:
            - my-file.txt
            - my-project/my-dependencies-directory
```
Đường dẫn cho các thư mục là tương đối từ `working_directory` trong job của bạn. Bạn có thể sử dụng đường dẫn tuyệt đối nếu bạn thích.

# Using Keys and Templates
Một cache-key được người dùng định nghĩa có thể bao gồm các **giá trị động** - chúng được gọi là các **templates**. Bất cứ gì bạn nhìn thấy trong dấu ngoặc nhọn là một template:
```
myapp-{{ checksum "package-lock.json" }}
```
Và giá trị nó sinh ra với key duy  nhất như sau:
```
myapp-+KlBebDceJh_zOWQIAJDLEkdkKoeldAldkaKiallQ<etc>
```
Nếu nội dung `package-lock.json` thay đổi, hàm `checksum` cũng trả về chuỗi khác, duy nhất, cho thấy cần vô hiệu hóa cache.

Trong khi chọn các templates phù hợp cho key cache, hãy nhớ rằng lưu cache không phải là một toán tử miễn phí, sẽ mất một thời gian để tải bộ đệm lên bộ lưu trữ CircleCI. Để tránh tạo bộ đệm mới mỗi bản dựng, hãy có một khóa tạo bộ đệm mới chỉ khi có thứ gì đó thực sự thay đổi.

Đây là 1 vài ví dụ trong chiến thuật sử dụng cache cho các mục đích khác nhau:
- `myapp-{{ checksum "package-lock.json" }}` - Cache được sinh ra khi có sự thay đổi trong file `package-lock.json`, các nhánh khác nhau của project sẽ sinh ra các cache key như nhau
- `myapp-{{ .Branch }}-{{ checksum "package-lock.json" }}` - Cache được sinh ra khi có sự thay đổi trong file `package-lock.json`, các nhánh khác nhau của project sẽ sinh ra các cache key riêng biệt
- `myapp-{{ epoch }}` - Mỗi bản build sẽ sinh ra các cache keys riêng biệt

Trong khi các step được thực thi, các template ở trên sẽ được thay thế bằng cách giá trị thời gian chạy và sử dụng string kết quả làm key. Bảng sau đây sẽ mô tả các cache key templates có sẵn:


| Template   | Description |
| -------- | -------- |
| `{{ checksum "filename" }}`     | Một chuỗi băm base64 mã hóa SHA256 từ nội dung file nhận vào     |
| `{{ .Branch }}`     | VCS branch đang build    |
| `{{ .BuildNum }}`     | Số CircleCI job đang build    |
| `{{ .Revision }}`     | Revision VCS đang build     |
| `{{ .Environment.variableName }}  `     | Biến môi trường `variableName` (hỗ trợ bất kí biến môi trường nào trích xuất bởi CircleCI hoặc được thêm vào Context cụ thể, không phải bất kì biến môi trường tùy chọn nào)     |
| `{{ epoch }}`     | Số giây đã trôi qua kể từ 00:00:00 theo giờ quốc tế còn được gọi là POSIX hoặc Unix epoch. Key cache này là 1 lựa chọn tốt nếu bạn muốn chắc chắn rằng cache luôn mới cho mỗi lần chạy     |
| `{{ arch }} `     | Thông tin hệ điều hành hoặc CPU. Nó sẽ hữu ích khi cache biên dịch phụ thuộc kiến trúc server     |

## Ghi chú thêm về sử dụng Keys và Templates

- Khi xác định một định danh duy nhất cho cache, cẩn thận  về việc lạm dụng sử dụng template key có tính chuyên dụng cao như `{{ epoch }}`. Bạn có thể sử dụng template ít cụ thể hơn như `{{ .Branch }}` hoặc `{{ checksum "filename" }}` bạn sẽ tăng tỉ lệ cache được sử dụng
- Các biến cache cũng có thể chấp nhận các đối số nếu muốn, ví dụ `v1-deps-<< parameters.varname >>.`
- Bạn không phải sử dụng các mẫu động cho khóa bộ nhớ cache của mình. Bạn có thể sử dụng một string tĩnh và có thể bắt đầu (thay đổi) tên của nó để buộc vô hiệu hóa bộ đệm.

# Chiến lược sử dụng cache
Trong các trường hợp, build tools của bạn bao gồm xử lý cho dependencies, khôi phục cache một phần có thể được ưu tiên hơn so với khôi phục cache từ con số 0 vì lý do hiệu suất. Nếu bạn khôi phục zero cache, bạn phải cài lại tất cả các dependencies, có thể dẫn đến giảm hiệu năng. Một cách khác là lấy phần lớn các dependencies từ cache cũ thay vì bắt đầu từ con số 0.

Tuy nhiên, đối với 1 vào ngôn ngữ khác, cache một phần mang những rủi ro khi không phù hợp với các dependencies đã khai báo hoặc có vấn đề cho đến khi bạn build mà không có cache. Nếu dependencies thay đổi không thường xuyên, hãy xem xét khôi phục cache từ zero cache key đầu tiên.

Sau đó, theo dõi chi phí về thời gian. Nếu hiệu năng từ việc khôi phục zero cache chứng tỏ rằng có ý nghĩa theo thời gian, sau đó xem xét việc thêm một khóa khôi phục một phần.

Liệt kê nhiều khóa để khôi phục cache làm tăng tỉ lệ có được cache một phần. Tuy nhiên. mở rộng phạm vi `restore_cache` có thể dẫn đến các rủi ro khó hiểu. Ví dụ, nếu bạn có một dependencies với Node v6 trong một branh update, nhưng các branch khác của bạn vẫn chỉ đang ở Node v5, một bước `restore_cache` sẽ tìm kiếm từ các branch khác và khôi phục các dependencies không tương thích này.

## Using a Lock File

Checksum từ các lockfiles quản lý sự phụ thuộc của các ngôn ngữ (ví dụ `Gemfile.lock` hoặc `yarn.lock`) có thể là một khóa hữu ích.

Một sự thay thể là thực hiện `ls -laR your-deps-dir > deps_checksum` và tham chiếu nó với `{{ checksum "deps_checksum" }}`. Ví dụ, trong Python, để có được cache cụ thể hơn checksum file `requirements.txt` bạn có thể cài đặt các dependencies với virtualenv tại root trong project `venv` và sau đó thực thi `ls -laR venv > python_deps_checksum`.

## Sử dụng nhiều cache cho các ngôn ngữ khác nhau

Bạn có thể giảm chi phí cache bằng cách chia các job của bạn qua nhiều caches. Bằng cách chỉ định nhiều bước `restore_cache` với các key khác nhau, mỗi cache được giảm kích thước do đó làm giảm việc không tìm thấy cache. Hãy xem xét phân tách cache bởi 1 vài ngôn ngữ `(npm`, `pip`, or `bundler`) nếu bạn biết mỗi dependency quản lý lưu trữ nó như thế nào, làm sao để nâng cấp nó và làm sao để kiểm tra được các dependencies.

## Cache lại các bước "tốn kém"

Một số ngôn ngữ và framworks nhất định có nhiều bước đắt hơn và có thể nên được cache lại. `Scala` và `Elixir` là hai ví dụ khi mà caching các bước biên dịch là cực kì hiệu quả. Rails cũng vậy, khi bạn nhận thấy hiệu suất sẽ cải thiện khi cache phần assets từ frontend (ví dụ `npm run dev`).

Đừng cache mọi thứ nhưng cân nhắc cache các steps tốn kiếm, như biên dịch project :D

# Source Caching

Trong CircleCI 1.0, bạn có thể cache lại repository git của bạn, tiết kiệm thời gian cho bước `checkout` - đặc biệt cho các dự án lớn. Đây là một ví dụ cho việc cache mã nguồn
```yaml
    steps:
      - restore_cache:
          keys:
            - source-v1-{{ .Branch }}-{{ .Revision }}
            - source-v1-{{ .Branch }}-
            - source-v1-

      - checkout

      - save_cache:
          key: source-v1-{{ .Branch }}-{{ .Revision }}
          paths:
            - ".git"
```
Trong ví dụ trên `restore_cache` tìm kiếm cache từ revision git hiện tại, sau đó tìm kiếm từ nhánh hiện tại và cuối cùng là một vài cache khác bất kể nhánh và revison.

Nếu source code của bạn thay đổi thường xuyên, chúng tôi khuyên bạn sử dụng nhiều các keys cụ thể. Điều này tạo ra một bộ đệm mã nguồn chi tiết hơn sẽ cập nhật thường xuyên hơn khi thay đổi sửa đổi chi nhánh và git hiện tại.

Thậm chí với trường hợp hẹp nhất của `restore_cache` (`source-v1-{{ .Branch }}-{{ .Revision }}`), cache mã nguồn có thể rất có lợi khi, ví dụ chạy các bản build đi build lại với các git revision giống nhau (ví dụ với [ API-triggered builds](https://circleci.com/docs/api/#trigger-a-new-build-by-project-preview)) hoặc khi sử dụng Workflows, khi mà bạn có thể cần bước `checkout` với repository giống nhau cho mỗi Workflows.

Điều đó nói rằng, nó đáng để so sánh thời gian build có và không có sourch caching, `git clone` thường nhanh hơn `restore_cache`.

NOTE: Lệng built-in `checkout` sẽ vô hiệu hóa các bộ sưu tập rác tự động của git. Bạn sẽ có thể chọn chạy thủ công `git gc` trong bước `run` trước khi chạy `save_cache` để giảm kích thước cache được lưu.

# Tổng kết lại
### Cache để làm gì?
- Cache giúp bạn chạy job nhanh hơn. Tuy nhiên hãy đảm bảo cân bằng giữa sự nhanh này với độ tin cậy, và bạn không nên cache mọi thứ.
- Cache dependencies không được tự động thực hiện trong CircleCI 2, do đó bạn phải thực hiện thủ công.
### Ghi cache
- Cache bất biến khi ghi
- Có thể kèm theo template (các biến động) để tạo ra key cụ thể hơn. Chi tiết các biến động này nên là gì bạn có thể tham khảo ở trên
```
myapp-{{ checksum "package-lock.json" }}
```
### Khôi phục cache
Cache sẽ khôi phục dựa vào 2 điều kiện
- Cache được khôi phục từ trùng key đầu tiên. Ví dụ `myapp-+KlBebDceJh_zOWQIAJDLEkdkKoeldAldkaKiallQ`
- Trong trường hợp không có key trùng khớp, nó sẽ sử dụng key **một phần,** ví dụ như `myapp-` có các key trùng khớp là: `my-app-1`, `my-app-2`, `my-app-3`. Nếu có nhiều sự trùng khớp cache được tạo gần nhất sẽ được sử dụng.
### Chiến lược sử dụng cache hiệu quả
- Sử dụng các lock file: Checksum từ các lockfiles quản lý sự phụ thuộc của các ngôn ngữ (ví dụ `Gemfile.lock` hoặc `yarn.lock`) có thể là một khóa hữu ích.
- Sử dụng nhiều cache cho các ngôn ngữ khác nhau: hiểu ngôn ngữ và cache lại các dependencies một cách hợp lý
- Cache lại các bước "tốn kém"
- Source caching: Caching mã nguồn hiệu quả tránh phải clone nhiều lần.

# Tài liệu tham khảo
https://circleci.com/docs/2.0/caching/

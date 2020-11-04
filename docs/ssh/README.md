# SSH
> SSH là giao thức sử dụng cách thức mã hóa để đạt mục đích bảo mật dữ liệu giữa 2 đầu kết nổi. 
> Tất cả thông tin về user auth, command, dữ liệu đầu vào, thậm chí là file content đều được mã hóa rồi mới transfer qua không gian mạng.
## Giao thức SSH
Có 2 việc cần làm để có thể bắt đầu với 1 phiên làm việc qua ssh.
> 1 . Thỏa thuận chuẩn mã hóa, các bước sau sẽ tuần tự được thực hiện:
- Client và server đồng thuận sử dụng 1 số nguyên lớn gọi là `seed value`.
- Client và server đồng thuận dùng cùng 1 algorithm dựa trên `seed value`.
- Cả 2 bên tạo ra 1 key bí mật độc lập nhau, cùng giá trị ở 2 nơi và không trao đổi qua mạng.
- Key bí mật bên trên kết hợp `seed value`, algorithm tạo ra 1 key mới phân phối cho máy còn lại.
- 2 bên sau đó dùng key bí mật của chính nó,
`seed value` và key vừa tạo của máy còn lại để generate ra một key chung cuối cùng dùng để mã hóa,
giải mã (Quá trình này vẫn làm độc lập, không transfer. Thuật toán làm điều này là `Diffie-Hellman Key Exchange Algorithm`)
> 2 . Xác thực user.
- Phần dưới đây sẽ giải thích, trình bày về này !
## SSH keys
> Để xác thực người dùng chúng ta cần có 2 key, đó là public key và private key, 2 key này phải tương ứng theo cặp mới có thể sử dụng.
- Public key dùng cho việc mã hóa và nó được công khai, bên nào cũng có thể nắm giữ.
- Private key dùng để giải mã dữ liệu mà được mã hóa bởi public key phía trên,
key này chỉ đặt tại bên sử dụng dữ liệu như client, không bao giờ phân phối tới nơi nào khác.
### `ssh-keygen`
> Mặc định lệnh sẽ tạo ra cặp key theo thuật toán RSA.

> Format: `ssh-keygen [-p] [-f filePathForSaving] [-t algorithmName] [-b keySizeAccordingToBit]` .

- Các algorithmName có thể sử dụng là rsa, dsa, ecdsa, ed25519.
Có thể xem [Chi tiết tại đây](https://www.ssh.com/ssh/keygen/) để sử dụng cho phù hợp
.
- Nếu không chỉ định tên file để lưu key thì mặc định sẽ là `/home/{user}/.ssh/id_{algorithmName}` đối với private key,
và `/home/{user}/.ssh/id_{algorithmName}.pub` cho public key.
- Quá trình tạo key sẽ cho phép chúng ta có thêm một option bảo mật tại client là nhập `passphrase`,
có thể coi nó dùng để tạo 1 khóa (`identity keys`) để xác thực chính public key.
- [-p] sẽ dùng lại key đã tồn tại và chỉ thay đổi passphrase nếu nhập.
> Copy 1 public key tới server thay vì copy bằng tay.
> khi chúng ta muốn cho ai đó quyền ssh vào server chẳng hạn,
> đảm bảo không có chuyện "rõ ràng anh add chú vào rồi còn gì, key của chú ở đâu ra thế ?":
```bash
ssh-copy-id -i path user@host
```
### `ssh-agent`
> Nó được hiểu là 1 trình phụ trợ, giúp tự động xác minh khớp `identity keys` với `passphrase`.

> Nghĩ lại thì cũng mất công, nhập passphrase xong để tăng bảo mật xong lại đi dùng tool để không phải nhập tay nữa,
> thế quên không tắt máy xong có ông nào vào bật ssh nghịch thì làm thế nào =)) [Có 1 giải pháp đáng đọc thử ở đây cho vấn đề này](https://www.cyberciti.biz/faq/unix-linux-appleosx-bsd-ssh-add-agent-command-set-lifetime/)

> Trên Linux, `ssh-agent` đã có sẵn và tất nhiên là đã phải chạy `ssh-keygen` để tạo ra cặp key trước.

> Để chắc chắn nó đã được enable thì:
```bash
eval `ssh-agent`
```
> Kiểm tra xem nó có đang hoạt động dựa vào biến môi trường:
```bash
echo $SSH_AGENT_SOCK
```
### `ssh-add`
> Command này sử dụng các private key trong ~/.ssh/id_{algorithmName} để generate `identity keys` thêm vào `agent`.
```bash
ssh-add [fileNamePrivateKey]
```
Nó sẽ yêu cầu nhập `passphrase` mà được khai báo lúc chạy `ssh-keygen`.
Bình thường thì mỗi khi ssh sẽ phải nhập `passphrase`, còn sau lần nhập này thì agent sẽ lưu lại nó và không hỏi lại nữa.
> Để xem list `identity keys`:
```bash
ssh-add -l
```
## SSH client
### `ssh` command
> Để mở phiên kết nối ssh remote tới máy đích, chỉ cần chạy command:
```bash
ssh userAccount_On_Server@Ip_Or_HostName
```
Ví dụ : `ssh Jonathan.Galindo@127.1.2.123` truy cập cây thư mục phân quyền theo user là Jonathan.Galindo trên server 127.1.2.123.
### SSH config
### `known_hosts`
> Client lưu `host key` của các server mà nó đã từng remote tới, gọi chung là `known_hosts`.

> Nó được lấy ra từ "/etc/ssh/" trên server và lưu tại client ở "~/.ssh/known_hosts" ứng với user client.
## SSH server
### SSHD config
### `authorized_keys`
> File trên server được dùng để khai báo public key của các client.
```bash
vim ~/.ssh/authorized_keys
```

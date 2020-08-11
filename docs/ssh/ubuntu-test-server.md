SSH là giao thức thông dụng nhất để kết nối đến server (VPS hay Cloud server).

Trong hướng dẫn này chúng ta sẽ sử dụng Docker để tạo một môi trường server ảo, từ đó có thể thực hành devops cơ bản mà không cần phải thuê cloud server.

Server ảo này tương tự như một instance EC2, sử dụng Ubuntu 18.04. Khi tạo 1 EC2 instance thường AWS sẽ cung cấp cho chúng ta 1 file pem để có thể ssh connect đến server. Tương tự, ở đây chúng ta cũng có 1 file pem có sẵn dành cho mục đích test.

Docker image: [sun7pro/ssh-test-server:ubuntu18.04](https://github.com/sun7pro/docker-library/tree/master/ssh-test-server).

Các bước thực hành:

1. Tải file pem để connect vào server ảo tại [đây](https://github.com/sun7pro/docker-library/blob/master/ssh-test-server/ubuntu_id_rsa)

    Trước khi sử dụng, cần phải chmod lại file pem với quyền 400 hoặc 600, tức là chỉ cho phép user owner có quyền đọc/ghi, các user khác không có bất cứ quyền gì. Nếu không ssh sẽ thông báo lỗi:
    ```sh
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    @         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
    @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
    Permissions 0644 for 'ubuntu_id_rsa' are too open.
    ```

    Change mod:
    ```sh
    chmod 600 ubuntu_id_rsa
    ```
2. Khởi chạy docker container
    ```sh
    docker run -d --name=ssh-ubuntu18 -p 2222:22 sun7pro/ssh-test-server:ubuntu18.04
    ```

    Muốn khởi động lại container:

    ```sh
    docker restart ssh-ubuntu18
    ```

    Hoặc xóa để tạo lại container:

    ```sh
    docker stop ssh-ubuntu18
    docker rm ssh-ubuntu18
    ```

    SSH server chạy ở cổng 22 bên trong container và được publish ra ngoài host machine ở cổng 2222. Như vậy chúng ta có ssh server với config như sau:

    ```sh
    Host ssh-ubuntu18
        HostName 0.0.0.0
        Port 2222
        User ubuntu
        IdentitiesOnly yes
        IdentityFile /home/ubuntu/Projects/devops-basic/ubuntu_id_rsa
    ```

    ```sh
    ssh ssh-ubuntu18
    ```

    Hoặc kết nối trực tiếp không qua ssh-config:

    ```sh
    ssh -o IdentitiesOnly=yes -i ./ubuntu_id_rsa ubuntu@0.0.0.0 -p 2222
    ```
3. Server is up
    Sau khi bạn đã kết nối được vào Ubuntu server, bạn có thể tiếp tục thực hành cài phần mềm, cài LAMP stack, tool theo dõi hệ thống, thực hành Ansible,...
4. Your turn :D

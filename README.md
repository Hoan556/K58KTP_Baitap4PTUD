# SỬ DỤNG KẾT QUẢ ĐÃ LÀM Ở BÀI TẬP 3, BỔ SUNG VÀO DOCKER COMPOSE ĐỂ CÓ THÊM SERVICE 8N8:

## SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ TẠO 1 file docker-compose.yml chứa:

Mariadb: sử dụng image: mariadb:latest để làm hệ quản trị csdl cho wordpress, thêm các biến môi trường: TZ: "Asia/Ho_Chi_Minh", MARIADB_ROOT_PASSWORD, MARIADB_DATABASE, MARIADB_USER, MARIADB_PASSWORD (giá trị tuỳ ý)

Phpmyadmin: sử dụng image: phpmyadmin:latest để đăng nhập vào mariadb rồi tạo csdl trống (chỉ để xem, ko cần tạo bảng từ đây, wordpress sẽ làm hết), khai báo biến môi trường: PMA_HOST: <tên service mariadb>, PMA_ARBITRARY: 1

WordPress: sử dụng image: wordpress:latest, truyền các tham số môi trường cho wordpress là các thông tin truy cập csdl mariadb, tạo bởi Phpmyadmin, khai báo biến môi trường: WORDPRESS_DB_HOST: <tên service mariadb>, WORDPRESS_DB_NAME, WORDPRESS_DB_USER, WORDPRESS_DB_PASSWORD (giá trị theo mariadb đã khai báo)

Cloudflared: sử dụng image: cloudflare/cloudflared:latest , full command và token lấy từ dashboard của cloudflare, dùng AI chuyển sang dạng docker compose

N8n : sử dụng image: n8nio/n8n:latest, nhớ truyền biến môi trường WEBHOOK_URL theo sub-domain đã add router cho cloudflared tunnel (ví dụ: WEBHOOK_URL=https://k58-n8n.tdh.io.vn/ )

### Yêu cầu: sau khi có 5 service này trong file docker-compose.yml :

pull các images về và chạy chúng (up -d)

Kiểm tra các service đã running ok (ko bị restart liên tục)

Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)

Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)

Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào!

Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp

Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...

Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở

Truy cập sub-domain3 để cấu hình n8n:

tạo tài khoản admin : nhớ điền đúng email

Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY

Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

Create workflow (home page => overview => Create workflow)

Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token

Access Token thì lấy ở Telegram qua việc chát với @BotFather

Cần chát với bot @BotFather để đẻ ra bot mới của riêng mình. bot này sẽ là nơi nhận lệnh (promt) để AI sinh html => n8n sẽ dùng html này để đăng bài lên wp

Sau khi tạo bot mới cần copy lấy Token, và chát lần đầu với bot mới này, nội dung bất kỳ (bước này quan trọng!)

Add (nối tiếp vào sau node Telegram Trigger) node: AI Google Gemini => Message a model => Set up Credential => cần Nhập API KEY

Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys
cần tạo project mới, sẽ lấy được API KEY

Nhập API Key lên giao diện n8n

kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)
Turn on Output Content as JSON : để kết quả trả về dạng json

Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?

Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.

// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
  title: cleanData.post_title,
  content: cleanData.post_content
};
Add (nối tiếp vào sau node Code in JavaScript) node: WordPress => Create a Post

Set up Credential: vào wp tại url: https://sub-domain1/wp-admin => vào mục Tài Khoản => chọn user đã tạo lúc setup wordpress => Mật khẩu ứng dụng => Nhập n8n và bấm "Thêm mật khẩu ứng dụng" => copy chuỗi 24 kí tự : Đây là mật khẩu ứng dụng => paste vào mục Password của n8n Credential

Wordpress URL: điền giá trị https://sub-domain1/ (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)

Ignore SSL Issues (Insecure): TURN ON

Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content

Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)

PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

Kết quả cuối cùng cần đặt được:

từ điện thoại, chát với telegram bot

nội dung chát được tự động gửi tới node Telegram trigger => Gửi tới Google Gemini Message a model (bản chất là gửi Prompt) : Nhận về json kết quả của Prompt => Gửi sang node Code in JavaScript để tách tiêu đề và nội dung => gửi đến node WordPress để Create a Post(đăng bài) với tiêu đề và nội dung từ node trước gửi sang.

f5 wordpress để thấy bài viết mới đã lên sóng.

Chụp ảnh quá trình thao tác/cấu hình/các kết quả trung gian đạt được

Nhận xét thành quả đạt được!!!

demo kết quả cuối cùng:

chát với bot:
### TẠO THƯ MỤC
<img width="915" height="567" alt="image" src="https://github.com/user-attachments/assets/e1b620e3-2055-4986-8c09-5fdc01247c4b" />
-Tạo file:
nano docker-compose.yml
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7a210f97-eb93-40a1-8511-df7837d47e32" />

<img width="924" height="694" alt="image" src="https://github.com/user-attachments/assets/a924aca8-4e96-4e44-b7f6-ff0a4f724cef" />

CHẠY DOCKER

docker compose pull

<img width="756" height="839" alt="image" src="https://github.com/user-attachments/assets/35ac4e4b-d5c4-4a2c-812a-6a0455718873" />
``` text
  chạy container
     docker compose up -d
  kiểm tra
     docker ps
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fdec9dae-2a7f-4248-8bd7-47ee7aadebad" />

Cấu hình cloudflare tunnel add router để public wordpress lên sub-domain1 (dùng để truy cập wordpress)

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a7fc4285-ab63-4c34-9ad5-aee900c66176" />

Cấu hình cloudflare tunnel add router để public Phpmyadmin lên sub-domain2 (dùng để truy cập phpmyadmin)

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7c953907-f223-43c2-acee-c306d74de3b8" />

Cấu hình cloudflare tunnel add router để public n8n này lên sub-domain3 (dùng để truy cập và cấu hình n8n)

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/4b14ff22-4868-419d-adb3-b383d3433853" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c3fc5eb8-4eb7-494b-9a1b-f153f0c97a50" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu chưa có bảng nào

- https://pma.nguyenvanhoan.id.vn
  
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/ee65f91f-e289-4faa-a054-3d691a4b2ce6" />

- https://wordpress.nguyenvanhoan.id.vn
  
Truy cập sub-domain1 để cài đặt wordpress (làm theo hướng dẫn của wordpress)

<img width="965" height="1079" alt="image" src="https://github.com/user-attachments/assets/191c327b-d087-463d-bebe-68b5cd3b88a4" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/9dab6edf-c009-4c32-9ba0-596f1fe1d4ea" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/2b8c3aa1-4e2c-4ecb-b795-4c917cf421a6" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/74a61f7e-ebed-492f-a152-d6726de9f3c7" />

Truy cập sub-domain2 để quan sát xem cơ sở dữ liệu có những bảng dữ liệu nào sau khi cài wp

- https://pma.nguyenvanhoan.id.vn
- 
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8a2ea8f6-8c19-4e42-979a-98f3c7d08f35" />

Tạo 1 bài viết trong wordpress giới thiệu về bản thân sinh viên: thông tin cá nhân, sở thích, ... bài viết có thể chứa hình ảnh, âm thanh, video, ...

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/de5dce8a-86f3-40f4-a16a-356ed8f79053" />
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/706a6965-d5e8-4e3d-8206-1f2c4e1ed36d" />

Tạo 1 bài viết trong wordpress giới thiệu về nhữn kiến thức mà em đã học được ở môn Phát triển ứng dụng với mã nguồn mở

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/12b8df69-90d5-443e-9b33-be48539c2980" />

Truy cập sub-domain3 để cấu hình n8n http://wordpress:80

- https://n8n.nguyenvanhoan.id.vn
  
<img width="950" height="1078" alt="image" src="https://github.com/user-attachments/assets/5356e513-ec53-4542-b37d-cfb0340516da" />

Send me a Licence key, bước này điền đủ thông tin, làm chậm sẽ thấy mục gửi License key về mail (n8n sẽ gửi email KEY cho dùng), check email để lấy KEY

<img width="1181" height="2560" alt="image" src="https://github.com/user-attachments/assets/10dcf260-fc56-4b47-b527-90b7d461602f" />

Activate License key: vào trang chủ => SETTING (góc dưới trái) => Usage and plan => Enter activation key: paste key từ email vào đây => Activate => sẽ nhận đc thông báo (góc dưới phải) Your Registered Community Edition has been successfully activated.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/87b9bd7a-b91e-49a6-8b7c-2b839084c5bb" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d3888746-4203-43a3-a60c-dae3573c5185" />

Create workflow (home page => overview => Create workflow)

Add trigger node: tìm node: Telegram => OnMessage ; cấu hình Credential: Set up Credential => cần Nhập Access Token

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8884b549-4c3b-481c-ab07-6d5ab3d19b95" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/589995f9-c6fb-4cb1-a733-b80472ef30bc" />

Lấy API KEY tại trang: https://aistudio.google.com => https://aistudio.google.com/api-keys

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/fbc82480-6c3c-4243-9b5c-28d847abae8a" />

Nhập API Key lên giao diện n8n

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/82e56922-4ff1-4d22-8563-78c598c0124b" />

kéo thả nội dung đã chát với bot của telegram (phía bên trái) vào nội dung phần PROMPT kết quả được {{ $json.message.text }}, cần gõ thêm vào sau {{ $json.message.text }} để promt dài hơn : vd ({{ $json.message.text }}. Kết quả sinh ra ở định dạng HTML+CSS để tôi dùng HTML+CSS này tạo bài viết cho wordpress.)

Turn on Output Content as JSON : để kết quả trả về dạng json

Có thể thử nghiệm các thành phần khác trong Options (add Options: System message, ...) => đưa ra cái nào đáng dùng?

Bấm nút Execute step và chờ Gemini xử lý. Khi thành công, ở cột Output bên phải sẽ thấy AI trả về một chuỗi text thô lộn xộn nhưng chứa cấu trúc post_title và post_content.

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/d5838ed9-8f5e-48ec-ab88-84da70aeeecb" />

Add (nối tiếp vào sau node Message a model) node: Code in JavaScript

Code js ở dạng này, có thể phải thay đổi tuỳ theo json AI trả về.
``` text
// 1. lấy dữ liệu gốc
const rawText = $input.first().json.content.parts[0].text;

// 2. Chuyển đổi chuỗi (đã được bọc JSON) thành Object trong JavaScript
const cleanData = JSON.parse(rawText);

// 3. Trả về kết quả định dạng lại gọn gàng cho n8n sử dụng
return {
 title: cleanData.post_title,
 content: cleanData.post_content
};  
```
<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/a44a870d-6834-4495-a232-188b63998c6c" />

Wordpress URL: điền giá trị (giá trị này cũng khai báo trong biến môi trường WEBHOOK_URL của n8n)

Ignore SSL Issues (Insecure): TURN ON

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/1405fa91-939a-4f81-bcab-c77eb686e1fe" />

Cấu hình node Create a Post: bấm nút Execute previous nodes để thấy trường giá trị của node trước trả về, kéo nội dung phần title (bên trái) vào trường title, tương tự kéo nội dung content vào content

Add field (Thêm thuộc tính): Status == Publish (bài đăng sẽ ở trạng thái xuất bản ngay lập tức, mặc định nó ở giá trị Draft bản nháp)

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/870074bf-feea-4f18-9c7e-1b1169231d30" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/54bb7265-630f-4f15-ba47-5072a365321d" />

PUBLISH flow (góc trên phải) Nút này thực hiện việc xuất bản flow <=> flow sẽ tự động thực thi khi thoả mãn điều kiện trigger

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/e2d902e6-784c-4c41-9c92-2e3c0770dcde" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/7373d568-3cef-4fe5-abcc-8b985e201af7" />
Kết quả cuối cùng cần đặt được:

từ điện thoại, chát với telegram bot

<img width="1181" height="2560" alt="image" src="https://github.com/user-attachments/assets/a5f4d132-0b53-41c7-9b50-fdad8071a849" />


<img width="1290" height="2796" alt="image" src="https://github.com/user-attachments/assets/5c1764ae-8a1c-4f13-8e95-6da3fffa0a57" />

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8bba61a9-8419-412d-9940-880b1f96c351" />


<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/c6628f38-3da1-4f59-bfa6-1226dc57be4d" />


## Nhận xét thành quả đạt được!!!
Sau khi hoàn thành bài tập, em đã triển khai thành công hệ thống WordPress trên Ubuntu bằng Docker Compose với các service gồm MariaDB, PhpMyAdmin, WordPress, Cloudflared và n8n. Các container hoạt động ổn định và có thể truy cập từ Internet thông qua subdomain bằng Cloudflare Tunnel.

Trong quá trình thực hiện, em đã cài đặt và cấu hình WordPress thành công, kiểm tra cơ sở dữ liệu bằng PhpMyAdmin và tạo được các bài viết theo yêu cầu của đề bài. Đồng thời, em cũng cấu hình n8n kết hợp với Telegram Bot và Google Gemini AI để xây dựng hệ thống tự động tạo và đăng bài lên WordPress.

Tuy nhiên, trong quá trình làm bài em cũng gặp một số lỗi như lỗi cấu hình Cloudflare Tunnel, lỗi route DNS bị trùng, lỗi kết nối giữa n8n với Telegram/Gemini và một số lỗi trong quá trình cấu hình workflow. Ngoài ra, có lúc workflow không hoạt động đúng do chưa publish hoặc lỗi credential bị mất kết nối. Sau khi tìm hiểu tài liệu và kiểm tra lại cấu hình từng bước, em đã khắc phục được các lỗi trên và hệ thống hoạt động ổn định.

Kết quả cuối cùng đạt được là chỉ cần gửi nội dung qua Telegram, hệ thống sẽ tự động gửi yêu cầu đến Gemini AI để tạo nội dung bài viết, xử lý dữ liệu bằng JavaScript và đăng trực tiếp lên website WordPress. Qua bài tập này, em hiểu rõ hơn về Docker, Docker Compose, Cloudflare Tunnel, n8n automation cũng như cách tích hợp AI vào một bài toán thực tế.






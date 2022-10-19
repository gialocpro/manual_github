/// Git sử dụng 


- clone git
git clone đường dẫn file git từ github
//vd: git clone https://github.com/gialoc18/flutterproject.git
rm -rf git-vs-github
// xóa thư mục hiện tại theo tips javascript

- cài đặt biến cho người dùng hiện tại
git config --global uer.name "hoangloc"
git congfig -- global uer.email "hoangloc.it18@gmail.com"

// kiểm tra xem đã config đúng chưa
git config user.name
git config user.email

- commit và push code lên github
b1: git add tên file
// vd: git add index.js
b2: git status
b3: git commit -m "comment thông báo"
b4: git push origin main
//main chính là tên của nhánh code chính

-kéo dữ liệu thay đổi trên github về
git pull
- kiểm tra sự thay đổi (check log)
git log --pretty=oneline
- tạo thêm nhánh trong github xong có thể merge code vào code chính
git checkout -b tên nhánh
b1: git add tên file
// vd: git add index.js
b2: git status
b3: git commit -m "comment thông báo"
b4: git push -u origin addMul
vd: git checkout -b addMul
// so sánh file nhánh đang sử dùng với file gốc
git diff main
- kiểm tra nhánh
git branch

/// git checkout tên nhánh để xem dữ liệu của nhánh hiện tại
vd: git checkout main

- merge nhánh phụ vào nhánh chính
b1: git checkout main
b2: git diff addMul // có thể so sánh hoặc không
b3: git merge addMul
b4: sẽ đẩy  như bình thường 
+ git add tên file
// vd: git add index.js
+ git status
+ git commit -m "comment thông báo"
+ git push origin main

- file cofig quản lý dự án //gitignore file những file không nên đẩy


// git status kiểm tra file
//git cho file tạo 1 file (.gitignore) không đẩy lên git

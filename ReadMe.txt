2024/11/25
Kevin Cheng

* Docker Compose 說明
  1. LocalDB use MariaDB
  2. 分離 env 與 secret 文件，其中 env 管理地區、語言、mariadb root 帳戶空密碼與自訂帳號名稱 mariadb，
    secret (db_pwd) 管理自訂帳號密碼。
  3. 獨立 Docker 網路容器指定固定 IP ， 並對宿主機與網路內部開放 3306 連接埠。若無獨立網路，請參考下方
    網路管理說明。
  4. 運行指令 docker compose up -d 

* db image 版本選用原則
  1. 版本選擇皆以當前(文件撰寫日期)最新的長期規劃版本
  2. Linux 系統為 Ubuntu 依當前 LTS 版本採用 ubuntu 24.04.1 (Noble Numbat)
  3. SQL Server 依當前 MariaDB LTS 版本採用 mariadb 11.4.4
  4. 綜上所述，採用 image 為 mariadb:11.4.4-noble

* 參考資料與 image 來源
  1. ubuntu: https://releases.ubuntu.com/
  2. mariadb: https://mariadb.com/kb/en/mariadb-server-release-dates/
  3. docker hub: docker-image: https://hub.docker.com/layers/library/mariadb/11.4.4-noble/images/sha256-4cadd89bceead0d68254d502acef11bc0b5e40794fb6ae8720932592c22fac48?context=explore

* Docker 網路管理
  1. 檢視所有自訂 Docker 網路
    docker network ls --filter type=custom
  2. 建立專案網路，並命名為 dockerNet
    docker network create --driver bridge --subnet "172.18.0.0/16" --gateway "172.18.0.1" dockerNet
  3. 檢視指定網路下，IP 指派給容器的狀況
    docker network inspect dockerNet
  4. 移除指定 Docker 網路
    docker network rm dockerNet

* SQL 帳號管理與權限說明
  1. 容器建立完成後，root 僅預設本機連線，自訂帳號雖提供外部網路連線，僅有讀取權限。
  2. 進入容器
    docker exec -it <容器名稱> /bin/bash
  3. 進入 mariadb 
    mariadb
  4. 檢視當前 DB 帳號
    SELECT User, Host, Password, Grant_priv, Super_priv FROM mysql.user;
  5. 建立自訂 DB 帳號
    CREATE USER '<帳號>'@'%' IDENTIFIED BY '<密碼>';
  6. 授權自訂帳號 DB 權限
    GRANT ALL PRIVILEGES ON *.* TO '<帳號>'@'%';
  7. 授權生效
    FLUSH PRIVILEGES;

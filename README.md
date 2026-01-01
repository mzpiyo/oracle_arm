### 仅支持密钥登录，需要密码生成的请自行修改`oracle_arm.py`

## 步骤

### 一、新建`data`文件夹，编辑`.env`、`config`文件以及私钥`p.pem`并保存到`data`文件夹内

#### 1. `.env`文件按以下格式保存内容（配置 Gotify 通知）
```bash
USE_TG=True
GOTIFY_URL=[https://push.example.com](https://push.example.com)
GOTIFY_TOKEN=A1b2C3d4E5
```

* `USE_TG=True`: 启用通知推送总开关（True 为开启，False 为关闭）。
* `GOTIFY_URL`: 填写您的 Gotify 服务器地址（例如 `https://push.yourdomain.com`，末尾不要带 `/`）。
* `GOTIFY_TOKEN`: 填写 Gotify 的 Application Token (点击 Apps -> Create Application 获取)。

#### 2. `config`文件按以下格式保存内容（OCI 鉴权配置）

```ini
[DEFAULT]
user=ocid1.user.oc1..aaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxx
fingerprint=xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx
key_file=/opt/oci/p.pem
tenancy=ocid1.tenancy.oc1..aaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxx
region=us-ashburn-1

```

* `user`、`fingerprint`、`tenancy`、`region`的取值请参考[甲骨文官方文档](https://docs.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm#SDK_and_CLI_Configuration_File#ariaid-title3)。
* **注意**：`region` 必须与您在 `main.tf` 中配置的区域一致。

#### 3. `key_file` 私钥文件

* 将您的 API 签名私钥（通常为 `.pem` 格式）放入 `data` 文件夹。
* 在上面的 `config` 文件中，`key_file` 路径请**保持为 `/opt/oci/文件名**`（因为这是 Docker 容器内的映射路径），**不要**改成 VPS 的绝对路径。

### 二、准备 `main.tf` 文件

参考[大鸟博客](https://www.daniao.org/14121.html)的`1、生成main.tf`获取`main.tf`文件，保存到与`data`文件夹相同的目录层。

* 请确保 `main.tf` 中的 SSH 公钥、子网 ID (Subnet ID) 和镜像 ID (Source ID) 是正确的。

### 三、新建`docker-compose.yml`文件，粘贴以下内容并保存到与`data`文件夹相同的目录层

```yaml
services:
  oci:
    image: ghcr.io/mzpiyo/oracle-arm:latest
    container_name: oci
    restart: always
    volumes:
      - ./data:/opt/oci
      - ./main.tf:/oci/main.tf

```

### 四、附上文件位置图

* 您的目录结构应该如下所示：

```text
.
├── docker-compose.yml
├── main.tf
└── data/
    ├── .env
    ├── config
    └── p.pem (您的私钥文件)

```

### 五、启动

在目录下输入以下命令启动容器：

```bash
docker-compose pull && docker-compose up -d

```

启动后可使用 `docker logs -f oci` 查看运行日志。

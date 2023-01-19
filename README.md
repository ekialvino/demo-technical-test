
# Aplikasi Server Node JS menggunakan CloudFormation di AWS

Untuk langkah pertama pastikan sudah login ke AWS Console. pilih AZ yg diinginkan sebagai contoh saya memilih US East (N. Virginia)
## Membuat Keypair SSH EC2

1. pilih service EC2 kemudian dimenu samping kiri pilih sub menu Key Pairs pada menu Network & Security

2. Kemudian pilih Create key pair di posisi kanan atas

3. Isi nama keypair nya contoh : demo-nodejs dan biarkan default settingannya kemudian create key pair untuk di proses

4. secara otomatis key pair akan terdownload yang akan kita gunakan untuk login SSH ke EC2
## Setting Cloud Formation di AWS

## Setting Cloud Formation di AWS

1. Login ke AWS Amazon Console dan masuk ke Cloudformation

2. Kemudian pilih Create Stack untuk membuat cloud Formation

3. sebelumnya isi silahkan copy script di bawah ini kemudian di save dengan format yaml contoh : nodejs.yaml

```bash
AWSTemplateFormatVersion: '2010-09-09'

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      ImageId: ami-083569368b7d8f84d
      KeyName: demo-nodejs
      SecurityGroups:
        - !Ref InstanceSecurityGroup

  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
```

4. setelah disimpan, kembali ke Cloud Formation AWS dan pada Spesific template pilih upload template dan pilih template yang telah dibuat sebelumnya dan next.

5. kemudian berikan nama stack nya contoh:demo-nodejs dan klik next.

6. kemudian create stack dan tunggu hingga proses pembuatan ec2 selesai
## Login ke EC2

1. Cara buka terminal/powershell kemudian masuk ke ke folder key pair yg telah di download tadi. Kemudian Login

2. untuk mac/linux ubah mode file
```bash
chmod 400 demo-nodejs.pem
```

3. kemudian login menggunakan ssh alamat domain yg diberikan amazon atau ip public

```bash
ssh -i "demo-nodejs.pem" centos@ec2-54-167-233-28.compute-1.amazonaws.com
```


## Konfigurasi Firewall

1. Jika centos 7 yg dipilih belum ada firewall,silahkan install firewall terlebih dahulu contoh : firewalld

```bash
sudo yum install -y firewalld
```

2. kemudian jalankan dan enable firewalld

```bash
sudo systemctl start firewalld && sudo systemctl enable firewalld
```

3. Open Port 22,80 dan 443

```bash
   sudo firewall-cmd --zone=public --permanent --add-port=22/tcp
   sudo firewall-cmd --zone=public --permanent --add-port=80/tcp
   sudo firewall-cmd --zone=public --permanent --add-port=443/tcp
   ```

4. reload firewalld untuk memperbaharui konfigurasi firewall

```bash
    sudo firewall-cmd --reload
   ```

5. untuk tahap ini port hanya membuka yg diperlukan saja.
## Install Node JS di EC2
1. setelah selesai pembuatan EC2 kemudian login ssh menggunakan console

2. update terlebih dahulu centos
   
   ```bash
   sudo yum update -y
   ```

3. install epel-release untuk mendapatkan repo terbaru 

   ```bash
   sudo yum install -y epel-release
   ```

4. setelah itu, install nodejs

   ```bash
   sudo yum install -y nodejs
   ```

5. install npm

```bash
sudo yum install -y npm
```

6. kemudian install pm2 untuk membuat aplikasi bisa berjalan di belakang layar / background

   ```bash
   sudo npm i pm2 -g
   ```

## Install Nginx

1. install nginx untuk reverse proxy

   ```bash
   sudo yum install -y nginx
   ```

2. start dan enable nginx

```bash
sudo systemctl start nginx && sudo systemctl enable nginx
```

3. setelah itu pastikan nginx berjalan dengan baik dengan buka browser dan ketikkan ip public EC2 di AWS

4. jika berhasil, selanjutnya masuk ke folder /etc/nginx/conf.d

```bash
cd /etc/nginx/conf.d
```

5. jika ingin membuat subdomain yg mengarah ke IP Public EC2 ini. silahkan tambahkan A record. untuk contoh ini saya hanya menggunakan subdomain dari AWS

6. buat file bernama test.conf

```bash
sudo vi test.conf
```

7. Tambahkan script seperti dibawah dan ganti sub domain yang sesuai contoh: ec2-54-167-233-28.compute-1.amazonaws.com.

```bash
server {
    server_name  ec2-54-167-233-28.compute-1.amazonaws.com;

    #access_log  /var/log/nginx/host.access.log  main;
    # set client body size to 2M #
    location / {
        proxy_pass                              http://localhost:8081;
        proxy_set_header Host                   $host;
        proxy_set_header X-Real-IP              $remote_addr;
        proxy_set_header X-Forwarded-For        $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto      $scheme;
        proxy_set_header X-Forwarded-Host       $host;
        proxy_set_header X-Forwarded-Port       $server_port;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }

}
```

kemudian simpan file test.conf

8. pastikan file berjalan dengan baik dengan ketikkan perintah 

```bash
sudo nginx -t
```

9. kemudian reload nginx untuk melakukan update konfigurasi

```bash
sudo nginx -s reload
```
## GIT Clone Repository

1. masuk ke folder /home/centos

```bash
cd /home/centos
```

2. Kemudian git clone file js sederhana untuk mencoba node js server

```bash
https://github.com/ekialvino/demo-technical-test.git
```

3. masuk ke folde demo-technical-test

```bash
cd demo-technical-test
```

4. kemudian jalankan file dengan menggunakan pm2

```bash
pm2 start index
```

5. setelah itu silahkan coba di browser dengan mengetikkan sub domain AWS atau IP Public EC2.

6. jika setelah dicoba, gagal error nginx. silahkan ketikkan perintah

```bash
sudo setsebool -P httpd_can_network_connect 1
```

7. kemudian coba kembali dan aplikasi node js berhasil diakses yg bertuliskan 

```bash
Hello World from Node JS
```

## Konfigurasi SSL 

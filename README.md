
# Aplikasi Server Node JS menggunakan CloudFormation di AWS




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
         ImageId: ami-0f96495a064477ffb
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


##Konfigurasi firewall

1. start dan enable firewalld

```bash
   sudo systemctl start firewalld && sudo systemctl enable firewalld
   ```

2. Open Port 22,80 dan 443

```bash
   sudo firewall-cmd --zone=public --permanent --add-port=22/tcp
   sudo firewall-cmd --zone=public --permanent --add-port=80/tcp
   sudo firewall-cmd --zone=public --permanent --add-port=443/tcp
   ```

3. reload firewalld untuk memperbaharui konfigurasi firewall

```bash
    sudo firewall-cmd --reload
   ```

4. untuk tahap ini port hanya membuka yg diperlukan saja.

## Install Node JS di EC2
1. setelah selesai pembuatan EC2 kemudian login ssh menggunakan console

2. update terlebih dahulu centos
   
   ```bash
   sudo yum update
   ```

3. install epel-release untuk mendapatkan repo terbaru 

   ```bash
   sudo yum install epel-release
   ```

4. setelah itu, install nodejs

   ```bash
   sudo yum install nodejs
   ```

5. kemudian install pm2 untuk membuat aplikasi bisa berjalan di belakang layar / background

   ```bash
   sudo npm i pm2 -g
   ```

6. selanjutnya install nginx untuk reverse proxy

   ```bash
   sudo yum install nginx
   ```


## Clone project dari github

```bash
git clone https://github.com/ekialvino/demo-technical-test.git
```


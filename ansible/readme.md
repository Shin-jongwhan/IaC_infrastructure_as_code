### 250627
# Ansible
### 각 서버 패키지 자동 관리를 고민하다가 ansible을 쓰기로 결정했다.
#### <br/>

### 내가 관리하고 싶은 것은 이렇다.
- 공통적으로 패키지를 설치해야 할 게 있다. docker, git, kubernetes 등이다.
- 그리고 각 서비스를 운영하는 서버 성격마다 설치해야 하는 패키지들이 있다.
#### <br/>

### 비슷한 성격을 가진 툴로는 Ansible vs Chef vs Puppet가 있다.
#### ![image](https://github.com/user-attachments/assets/737a0be0-1eb5-40dd-bfdf-03547fd6e709)
### <br/><br/><br/>

## 설치
### root 계정으로 접속한다.
```
sudo apt update
sudo apt install ansible -y
```
### <br/>

### 버전 확인
```
ansible --version
```
#### ![image](https://github.com/user-attachments/assets/545ac327-d7d0-4026-82d4-f0b605c03904)
### <br/>

### 각 서버에서 sudo 권한을 가진 일반 사용자 계정에 접속해서 ssh key를 생성한다.
```
ssh-keygen -t rsa -b 4096
```
#### <br/>

### ansible을 설치한 곳에서 각 서버 ssh key를 가져온다.
```
ssh-copy-id jhshin@server

# 비밀번호 없이 자동 접속 확인
ssh jhshin@server
```
### <br/>

### ansible hosts 파일을 작성한다. 폴더명은 아무거나 상관 없음
```
mkdir ~/ansible/
vi hosts
```
#### <br/>

### hosts 파일
```
[db]
db ansible_host=db ansible_user=jhshin

[backend]
backend ansible_host=backend ansible_user=jhshin

[all_servers:children]
db
backend
```
### <br/>

### ansible 접속 확인
#### SUCCESS, ping - pong이 나오면 된 거다.
```
ansible -i hosts all_servers -m ping
```
#### ![image](https://github.com/user-attachments/assets/26d4aea6-3ea4-42cc-9d8f-bb74cb769f59)
### <br/>


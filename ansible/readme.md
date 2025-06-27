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

### sudo 접속을 허용한다.
#### 비밀번호 없는 sudo 권한을 jhshin에게 설정해야 에러가 안 난다.
```
echo "jhshin ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/jhshin
```
#### sudo 접속을 풀어두지 않으면 이렇게 에러가 난다.
#### ![image](https://github.com/user-attachments/assets/97250867-d9f6-4b1f-92ba-e515da68203e)
### <br/>

### ansible hosts 파일을 작성한다. 폴더명은 아무거나 상관 없음
```
mkdir ~/ansible/
vi hosts
```
#### <br/>

### hosts 파일
```
[service]
service ansible_host=service ansible_connection=local

[db]
db ansible_host=db ansible_user=jhshin

[backend]
backend ansible_host=backend ansible_user=jhshin

[all_servers:children]
service
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
### <br/><br/>

## playbook 작성하기
### 나는 모든 서버에 공통적으로 python3.13.5를 설치할 거다.
#### install_python.yml
```
- hosts: all_servers
  become: yes
  tasks:
    - name: 필수 패키지 설치
      apt:
        name:
          - build-essential
          - libncurses-dev
          - libssl-dev
          - libsqlite3-dev
          - tk-dev
          - libgdbm-dev
          - libc6-dev
          - libbz2-dev
          - libffi-dev
          - zlib1g-dev
          - software-properties-common
          - wget
        state: present
        update_cache: yes

    - name: Python 3.13.5 다운로드
      get_url:
        url: https://www.python.org/ftp/python/3.13.5/Python-3.13.5.tgz
        dest: /tmp/Python-3.13.5.tgz

    - name: 압축 해제
      unarchive:
        src: /tmp/Python-3.13.5.tgz
        dest: /tmp/
        remote_src: yes

    - name: configure 실행
      command: ./configure --enable-optimizations
      args:
        chdir: /tmp/Python-3.13.5
        creates: /tmp/Python-3.13.5/Makefile

    - name: make 실행
      command: make -j"{{ ansible_processor_cores | default(2) }}"
      args:
        chdir: /tmp/Python-3.13.5
        creates: /tmp/Python-3.13.5/python

    - name: make altinstall 실행
      command: make altinstall
      args:
        chdir: /tmp/Python-3.13.5
        creates: /usr/local/bin/python3.13

    - name: 심볼릭 링크 생성 (python, pip)
      file:
        src: "{{ item.src }}"
        dest: "{{ item.dest }}"
        state: link
        force: yes
      loop:
        - { src: '/usr/local/bin/python3.13', dest: '/usr/bin/python' }
        - { src: '/usr/local/bin/pip3.13', dest: '/usr/bin/pip' }

    - name: 설치된 Python 버전 확인
      command: python --version
      register: python_version_output
      changed_when: false

    - debug:
        msg: "설치된 파이썬 버전: {{ python_version_output.stdout }}"
```
### <br/>

### playbook을 실행해본다.
```
ansible-playbook -i hosts install_python.yml --limit all_servers
```
#### <br/>

### 설치가 진행되면 각 TASK 마다 로그가 출력된다.
#### ![image](https://github.com/user-attachments/assets/88bff292-8def-49e1-8320-4c2418652d34)
#### <br/>

### 마지막에 각 서버에 잘 설치가 되었는지 확인할 수 있다.
#### ![image](https://github.com/user-attachments/assets/0b297992-6674-427b-bc4b-976ea37639f6)
### <br/>

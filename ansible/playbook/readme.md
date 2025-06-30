### 250630
## playbook 기본 실행
- -i hosts : hosts 라는 파일 안에 정의된 서버 목록(= 인벤토리)을 기준으로 playbook yaml을 실행한다는 것이다.
- --limit \[server_name\] : hosts 파일 안에 정의된 server 또는 server group으로 한정하여 적용하게 한다.
```
ansible-playbook -i hosts add_user_docker_group.yml
```
### <br/><br/>

## parameter 지정
### yaml 파일에는 vars 라는 곳에 지정한다.
### 그리고 vars 사용은 중괄호 2개 안에 파라미터를 적어서 사용한다.
#### ex) add_user_docker_group.yml
```
- hosts: all_servers
  become: yes
  vars:
    target_user: jhshin

  tasks:
    - name: Add user '{{ target_user }}' to docker group
      user:
        name: "{{ target_user }}"
        groups: docker
        append: yes

    - name: Run 'docker ps' with sudo to verify installation
      shell: sudo docker ps
      register: docker_test
      changed_when: false
      ignore_errors: true

    - name: Show Docker test result
      debug:
        msg: |
          Docker access test result:
          {{ docker_test.stdout }}
```
### <br/>

### 명령어 예시
```
ansible-playbook -i hosts add_user_docker_group.yml --extra-vars "target_user=jhshin"
```

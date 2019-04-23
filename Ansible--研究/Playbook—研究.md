# Playbook—研究


# [Playbook官方文件參考](https://docs.ansible.com/ansible/latest/user_guide/playbooks_intro.html#playbook-language-example)
# 優點：
                - 可快速佈署複雜任務
                - 語法簡單易懂好寫
# 範例：利用playbook佈署nginx至多台機器

操作思路：

             1 **）建立專案目錄**
             2 **）建立yml編寫playbook任務**
             3 **）建立hosts將欲佈署的機器IP寫到此文件中**
             4 **）執行playbook**
    專案目錄---.yml
          |----hots
          |----.conf


    mkdir mission_nginx
    sudi vim nginx.yml

#nginx.yml

    ---
    - hosts: webservers
      vars:
        hello: Ansible
     
      tasks:
      - name: Add repo 
        yum_repository:
          name: nginx
          description: nginx repo
          baseurl: http://nginx.org/packages/centos/7/$basearch/
          gpgcheck: no
          enabled: 1
      - name: Install nginx
        yum:
          name: nginx
          state: latest
      - name: Copy nginx configuration file
        copy:
          src: ./site.conf
          dest: /etc/nginx/conf.d/site.conf
      - name: Start nginx
        service:
          name: nginx
          state: started
      - name: Create wwwroot directory
        file:
          dest: /var/www/html
          state: directory
      - name: Create test page index.html
        shell: echo "hello {{hello}}" > /var/www/html/index.html
    

 #hosts

    [webservers]
    192.168.244.133
    192.168.244.134
    [webservers:vars]
    ansible_ssh_user=root
    ansible_ssh_pass=root
    

#site.conf

    server {
        listen 80;
        server_name www.ctnrs.com;
        location / {
            root   /var/www/html;
            index  index.html;
        }
    }

執行playbook

    \[指令\] [選項 -i 指定主機清單 '劇本yml格式']
    ansible-playbook -i hosts nginx.yml
![](https://paper-attachments.dropbox.com/s_3F93927CAB1DE6F3B52A85E9BF2D69A135E52F179DA9F1C9A98010085EB010EA_1554197553949_image.png)

![](https://paper-attachments.dropbox.com/s_3F93927CAB1DE6F3B52A85E9BF2D69A135E52F179DA9F1C9A98010085EB010EA_1554197620463_image.png)

----------
# 一、YAML語法及文件結構
- 以縮進表示層級關係。
- 不可用”tab”縮進。
- 字符後必須縮進”1”個空格(：)。
- “-----”表示開始。
- “#”表示注釋。

文件結構：

    ---
    - hosts: webservers               #主機(組)
      vars:
        hello: Ansible
        remote_user:                  #用戶
    
      tasks:                          #任務
      - name: Add repo                #執行任務的名稱--新增repo
        yum_repository:               #任務描述
          name: nginx
          description: nginx repo
          baseurl: http://nginx.org/packages/centos/7/$basearch/
          gpgcheck: no
          enabled: 1
          
## **在變更時執行操作**（handlers）

**notify：在任務結束時觸發**
**handlers：由特定條件觸發Tasks**

    ---
     - hosts: webservers
       gather_facts: no
    
       tasks:
        - name: Copy nginx configuration file
          copy:
            src: ./site.conf
            dest: /etc/nginx/conf.d/site.conf
          notify:
            - restart nginx
      
       handlers:
        - name: restart nginx
            service: name=nginx state=reloaded
## **任務控制**（tags）：指定被tag的任務執行
    \[指令\] [playbook] [選項] "tags"
    ansible-playbook example.yml --tags "install,configuration,restart"


    ---
    - hosts: webservers
      gather_facts: no
    
      tasks:
        - name: Install redis
          yum: name=redis state=present
          tags: install
       
        - name: Copy redis configuration file
          copy: src=redis.conf dest=/etc/redis/redis.conf
          tags: configuration
    
        - name: Restart redis
          service: name=redis state=restarted
          tags: restart
----------
## 案例：自動部署Tomcat：
    #指定版本為：9.0.17
    ---
    - hosts: tomcat
      gather_facts: no
      vars:
         tomcat_version: 9.0.17
         tomcat_install_dir: /usr/local
    
      tasks:
        - name: Install jdk1.8
          yum: name=java-1.8.0-openjdk state=present
    
        - name: Download tomcat
          get_url: url=http://ftp.mirror.tw/pub/apache/tomcat/tomcat-9/v9.0.17/bin/apache-tomcat-9.0.17.tar.gz dest=/tmp
    
        - name: Unarchive tomcat-9.0.17.tar.gz
          unarchive:
            src: /tmp/apache-tomcat-9.0.17.tar.gz
            dest: "{{ tomcat_install_dir }}"
            copy: no
    
        - name: Start tomcat
          shell: cd {{ tomcat_install_dir }} &&
                 mv apache-tomcat-9.0.17 tomcat9 &&
                 cd tomcat9/bin && nohup ./startup.sh &





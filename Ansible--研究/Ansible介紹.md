# Ansible介紹
技術研究與知識來源--[珂儿吖](https://www.cnblogs.com/keerya/)

               [itread01](https://www.itread01.com/content/1524049083.html)

https://docs.ansible.com/：官方文件

https://github.com/ansible/ansible-examples



# **簡介：**

Ansible是自動化運維工具，由Python開發，集合了眾多運維工具（puppet、chef、func、fabric）的優點，實現了批量**系統配置**、批量**程序部署**、批量**運行命令**等功能。
　　Ansible是由**模組化**工作，本身沒有批量部署的能力。真正具有批量部署的是Ansible所運行的模塊，Ansible只是提供一種框架。Ansible不需要在遠程主機上安裝client/agents，因為它是透過SSH和遠端主機連線。Ansible目前已被紅帽官方收購，是自動化運維工具中大家認可度最高的，並且上手容易，學習簡單。

## 優點：
- 部署簡單
- 配置簡單、功能強大、擴展性強
- 有大量的模組可供使用
- 提供一個功能強大、操作性強的Web管理界面和REST API接口——AWX平台。
## 架構
![架構圖](http://i2.51cto.com/images/blog/201804/18/49b289fe3e1a97987823dba2ff599b77.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

- `Ansible`：Ansible核心程式。
- `HostInventory`：記錄由Ansible管理的主機信息，包括端口、密碼、ip等。
- `Playbooks`：YAML格式文件，多個任務定義在一個文件中，定義主機需要調用哪些模塊來完成的功能。
- `CoreModules`：**核心模塊**，主要操作是通過調用核心模塊來完成管理任務。
- `CustomModules`：自定義模塊，完成核心模塊無法完成的功能，支持多種語言。
- `ConnectionPlugins`：連接插件，Ansible和Host通信使用
## 執行模式

Ansible系統由控制主機對被管節點的操作方式可分為兩類，即`adhoc`和`playbook`：

- ad-hoc模式(點對點模式) 
      使用單個模塊，支持批量執行單條命令。ad-hoc命令是一種可以快速輸入的命令，而且不需要保存起來的命令。**就相當於bash中的一句話shell。**
- playbook模式(劇本模式) 
      是Ansible 主要管理方式，也是Ansible功能強大的關鍵所在。**playbook通過多個task集合完成一類功能**，如Web服務的安裝部署、數據庫服務器的批量備份等。可以簡單地把playbook理解為通過組合多條ad-hoc操作的配置文件。
## 工作流程
![Ansible任務執行流程](http://i2.51cto.com/images/blog/201804/18/b2da1ffd7a3fdc4b69adc58532c5d968.jpg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)


簡單理解就是Ansible在運行時，首先讀取`ansible.cfg`中的配置，根據規則獲取`Inventory`中的管理主機列表，並行的在這些主機中執行配置的任務，最後等待執行返回的結果。

1. 加載配置文件，預設 /etc/ansible/ansible.cfg；
2. 搜尋對應的主機配置文件，找到要執行的主機或者組(group)；
3. 加載自己對應的模塊文件，如command；
4. 通過ansible將模塊或命令生成對應的臨時py文件(python腳本)， 並將該文件傳輸至遠程服務器；
5. 對應執行用戶的家目錄的`.ansible/tmp/XXX/XXX.PY`文件；
6. 給文件+x 執行權限；
7. 執行並返回結果；
8. 刪除臨時py文件，`sleep 0`退出；
# 配置詳解
## 安裝方式

ansible安裝常用兩種方式，`yum安装`和`pip程序安装`。
**使用pip（python的包管理模塊）安裝**
　　首先，我們需要安裝一個`python-pip`包，安裝完成以後，則直接使用`pip`命令來安裝我們的包，具體操作過程如下：

        yum install python-pip
        pip install ansible

**使用yum 安裝**
　　yum安裝是我們很熟悉的安裝方式了。我們需要先安裝`epel-release`，然後再安裝ansible即可。

        yum install epel-release -y
        yum install ansible –y

**使用apt 安裝**

        sudo apt-get install -y ansible

**使用mac安裝**

        brew install ansible

安裝目錄如下(yum安裝)：
　　配置文件目錄：/etc/ansible/ 
　　執行文件目錄：/usr/bin/ 
　　Lib庫依賴目錄：/usr/lib/pythonX.X/site-packages/ansible/ 
　　Help文檔目錄： /usr/share/doc/ansible-XXX/ 
　　Man文檔目錄：/usr/share/man/man1/
**ansible配置文件**：/etc/ansible/ansible.cfg

     inventory = / etc / ansible / hosts＃這個參數表示資源清單inventory文件的位置
     library = / usr / share / ansible＃指向存放Ansible模塊的目錄，支持多個目錄方式，只要用冒號（:) 隔開就可以
     forks = 5＃並發連接數，默認為5
     sudo_user = root＃設置默認執行命令的用戶
     remote_port = 22＃指定連接被管節點的管理端口，默認為22端口，建議修改，能夠更加安全
     host_key_checking = False＃設置是否檢查SSH主機的密鑰，值為真/假。關閉後第一次連接不會提示配置實例
     timeout = 60＃設置SSH連接的超時時間，單位為秒
     log_path = /var/log/ansible.log＃指定一個存儲ansible日誌的文件

**ansuble主機清單 ：**/etc/ansible/hosts
　　裡面保存的是一些ansible 需要連接管理的主機列表：

    1、 直接指明主機地址或主機名：
        ## green.example.com#
        # blue.example.com#
        # 192.168.100.1
        # 192.168.100.10
    2、 定義一個主機組[組名]把地址或主機名加進去
        [mysql_test]
        192.168.253.159 
        192.168.253.160
        192.168.253.153

需要注意的是，這裡的組成員可以使用通配符來匹配，這樣對於一些標準化的管理來說就很輕鬆方便了。
　　我們可以根據實際情況來配置我們的主機列表，具體操作如下：

    [root@server ~]# vim /etc/ansible/hosts
        [web]
        192.168.37.122:22 ansible_ssh_user=  ansible_ssh_pass= 
        192.168.37.133:22 ansible_ssh_user=  ansible_ssh_pass= 
    -----引用組的變量-----    
        [web:vars]
        ansible_ssh_user=
        ansible_ssh_pass=
    #------------------------------------------------------------------------------------#
    ansible_ssh_host
        將要連線的遠端主機名.與你想要設定的主機的別名不同的話,可通過此變數設定.
    ansible_ssh_port
        ssh埠號.如果不是預設的埠號,通過此變數設定.
    ansible_ssh_user
        預設的 ssh 使用者名稱
    ansible_ssh_pass
        ssh 密碼(這種方式並不安全,我們強烈建議使用 --ask-pass 或 SSH 金鑰)
    ansible_sudo_pass
        sudo 密碼(這種方式並不安全,我們強烈建議使用 --ask-sudo-pass)
    ansible_sudo_exe (new in version 1.8)
        sudo 命令路徑(適用於1.8及以上版本)
    ansible_connection
        與主機的連線型別.比如:local, ssh 或者 paramiko. Ansible 1.2 以前預設使用 paramiko.1.2 以後預設使用 'smart','smart' 方式會根據是否支援 ControlPersist, 來判斷'ssh' 方式是否可行.
    ansible_ssh_private_key_file
        ssh 使用的私鑰檔案.適用於有多個金鑰,而你不想使用 SSH 代理的情況.
    ansible_shell_type
        目標系統的shell型別.預設情況下,命令的執行使用 'sh' 語法,可設定為 'csh' 或 'fish'.
    ansible_python_interpreter
        目標主機的 python 路徑.適用於的情況: 系統中有多個 Python, 或者命令路徑不是"/usr/bin/python",比如  \*BSD, 或者 /usr/bin/python
# Ansible 常用命令

**ansible 命令集**

> `/usr/bin/ansible`　　Ansibe AD-Hoc臨時命令執行工具，常用於臨時命令的執行
> `/usr/bin/ansible-doc` 　　Ansible模塊功能查看工具
> `/usr/bin/ansible-galaxy`　　下載/上傳優秀代碼或Roles模塊的官網平台，基於網絡的
> `/usr/bin/ansible-playbook`　　Ansible定制自動化的任務集編排工具
> `/usr/bin/ansible-pull`　　Ansible遠程執行命令的工具，拉取配置而非推送配置（使用較少，海量機器時使用，對運維的架構能力要求較高）
> `/usr/bin/ansible-vault`　　Ansible文件加密工具
> `/usr/bin/ansible-console`　　Ansible基於Linux Consoble界面可與用戶交互的命令執行工具

　　其中，我們比較常用的是`/usr/bin/ansible`和`/usr/bin/ansible-playbook`。
**ansible-doc 命令**
　　ansible-doc 命令常用於獲取模塊信息及其使用幫助，一般用法如下：

        ansible-doc -l              #獲取全部模塊的信息
        ansible-doc -s MOD_NAME     #獲取指定模塊的使用帮助

　　我們也可以查看一下ansible-doc的全部用法：

    [root@server ~]# ansible-doc
    Usage: ansible-doc \[options\] [module...]
    
    Options:
      -h, --help            show this help message and exit　　# 顯示命令參數API文檔
      -l, --list            List available modules　　         #列出可用的模塊
      -M MODULE_PATH, --module-path=MODULE_PATH　　            #指定模塊的路徑
                            specify path(s) to module library (default=None)
      -s, --snippet         Show playbook snippet for specified module(s)　　#顯示劇本制定模塊的用法
      -v, --verbose         verbose mode (-vvv for more, -vvvv to enable　　# 顯示ansible-DOC的版本號查看模塊列表：
                            connection debugging)
      --version             show program's version number and exit
![](https://d2mxuefqeaa7sj.cloudfront.net/s_DE1092155A01106D38A633F9E11CADC330EAF30CF8A039E3A7112F76E9B8D1A8_1553661806572_image.png)


**ansible 命令詳解**
　　命令的具體格式如下：

    ansible <host-pattern> \[-f forks\] [-m module_name] [-a args]

　　也可以通過`ansible -h`來查看幫助，下面我們列出一些比較常用的選項，並解釋其含義：

> `-a MODULE_ARGS`　　　#模塊的參數，如果執行默認COMMAND的模塊，即是命令參數，如： “date”，“pwd”等等
> `-k`，`--ask-pass`#ask for SSH password。登錄密碼，提示輸入SSH密碼而不是假設基於密鑰的驗證
> `--ask-su-pass`#ask for su password。su切換密碼
> `-K`，`--ask-sudo-pass`#ask for sudo password。提示密碼使用sudo，sudo表示提權操作
> `--ask-vault-pass`#ask for vault password。假設我們設定了加密的密碼，則用該選項進行訪問
> `-B SECONDS`#後台運行超時時間
> `-C`#模擬運行環境並進行預運行，可以進行查錯測試
> `-c CONNECTION`#連接類型使用
> `-f FORKS`#並行任務數，默認為5 
> `-i INVENTORY`#指定主機清單的路徑，默認為`/etc/ansible/hosts`
> `--list-hosts`#查看有哪些主機組
> `-m MODULE_NAME`#執行模塊的名字，默認使用command模塊，所以如果是只執行單一命令可以不用-m參數
> `-o`#壓縮輸出，嘗試將所有結果在一行輸出，一般針對收集工具使用
> `-S`#用su命令
> `-R SU_USER`#指定su的用戶，默認為root用戶
> `-s`#用sudo命令
> `-U SUDO_USER`#指定sudo到哪個用戶，默認為root用戶
> `-T TIMEOUT`#指定ssh默認超時時間，默認為10s，也可在配置文件中修改
> `-u REMOTE_USER`#遠程用戶，默認為root用戶
> `-v`#查看詳細信息，同時支持`-vvv`，`-vvvv`可查看更詳細信息

**ansible 配置公私鑰**

    #1.生成私鑰
    [root@server ~]# ssh-keygen 
    #2.向主機分發私鑰
    [root@server ~]# ssh-copy-id root@192.168.37.122
    [root@server ~]# ssh-copy-id root@192.168.37.133

這樣的話，就可以實現無密碼登錄。
　　#注意，如果出現了一下報錯：

        -bash: ssh-copy-id: command not found

　　那麼就證明我們需要安裝一個包：

        yum -y install openssh-clientsansible
# Ansible 常用模塊

**1）主機連通性測試**

1. 　　我們使用`ansible web -m ping`命令來進行主機連通性測試，效果如下：
    [root@server ~]# ansible web -m ping
    192.168.37.122 | SUCCESS => {
        "changed": false, 
        "ping": "pong"
    }
    192.168.37.133 | SUCCESS => {
        "changed": false, 
        "ping": "pong"
    }

**2）command 模塊**
　　這個模塊可以直接在遠程主機上執行命令，並將結果返回本主機。
　　舉例如下：

    [root@server ~]# ansible web -m command -a 'ss -ntl'
    192.168.37.122 | SUCCESS | rc=0 >>
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    LISTEN     0      128          *:111                      *:*                  
    LISTEN     0      5      192.168.122.1:53                       *:*                  
    LISTEN     0      128          *:22                       *:*                  
    LISTEN     0      128    127.0.0.1:631                      *:*                  
    LISTEN     0      128          *:23000                    *:*                  
    LISTEN     0      100    127.0.0.1:25                       *:*                  
    LISTEN     0      128         :::111                     :::*                  
    LISTEN     0      128         :::22                      :::*                  
    LISTEN     0      128        ::1:631                     :::*                  
    LISTEN     0      100        ::1:25                      :::*                  
    
    192.168.37.133 | SUCCESS | rc=0 >>
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    LISTEN     0      128          *:111                      *:*                  
    LISTEN     0      128          *:22                       *:*                  
    LISTEN     0      128    127.0.0.1:631                      *:*                  
    LISTEN     0      128          *:23000                    *:*                  
    LISTEN     0      100    127.0.0.1:25                       *:*                  
    LISTEN     0      128         :::111                     :::*                  
    LISTEN     0      128         :::22                      :::*                  
    LISTEN     0      128        ::1:631                     :::*                  
    LISTEN     0      100        ::1:25                      :::*  

　　命令模塊接受命令名稱，後面是空格分隔的列表參數。給定的命令將在所有選定的節點上執行。它不會通過shell進行處理，比如$HOME和操作如"<"，">"，"|"，";"，"&"工作（需要使用（shell）模塊實現這些功能）。注意，該命令不支持`| 管道命令`。
　　下面來看一看該模塊下常用的幾個命令：

> chdir #在執行命令之前，先切換到該目錄
> executable #切換shell來執行命令，需要使用命令的絕對路徑
> free_form #要執行的Linux指令，一般使用Ansible的-a參數代替。
> creates #一個文件名，當這個文件存在，則該命令不執行,可以
> 用來做判斷
> removes #一個文件名，這個文件不存在，則該命令不執行

　　下面我們來看看這些命令的執行效果：

    [root@server ~]# ansible web -m command -a 'chdir=/data/ ls'    #先切换到/data/ 目录，再执行“ls”命令
    192.168.37.122 | SUCCESS | rc=0 >>
    aaa.jpg
    fastdfs
    mogdata
    tmp
    web
    wKgleloeYoCAMLtZAAAWEekAtkc497.jpg
    
    192.168.37.133 | SUCCESS | rc=0 >>
    aaa.jpg
    fastdfs
    mogdata
    tmp
    web
    wKgleloeYoCAMLtZAAAWEekAtkc497.jpg
    [root@server ~]# ansible web -m command -a 'creates=/data/aaa.jpg ls'       #如果/data/aaa.jpg存在，则不执行“ls”命令
    192.168.37.122 | SUCCESS | rc=0 >>
    skipped, since /data/aaa.jpg exists
    
    192.168.37.133 | SUCCESS | rc=0 >>
    skipped, since /data/aaa.jpg exists
    [root@server ~]# ansible web -m command -a 'removes=/data/aaa.jpg cat /data/a'      #如果/data/aaa.jpg存在，则执行“cat /data/a”命令
    192.168.37.122 | SUCCESS | rc=0 >>
    hello
    
    192.168.37.133 | SUCCESS | rc=0 >>
    hello

**3）shell 模塊**
　　shell模塊可以在遠程主機上調用shell解釋器運行命令，支持shell的各種功能，例如管道等。

    [root@server ~]# ansible web -m shell -a 'cat /etc/passwd |grep "keer"'
    192.168.37.122 | SUCCESS | rc=0 >>
    keer:x:10001:1000:keer:/home/keer:/bin/sh
    
    192.168.37.133 | SUCCESS | rc=0 >>
    keer:x:10001:10001::/home/keer:/bin/sh

　　只要是我們的shell命令，都可以通過這個模塊在遠程主機上運行，這裡就不一一舉例了。
**4）copy 模塊**
　　這個模塊用於將文件複製到遠程主機，同時支持給定內容生成文件和修改權限等。
　　其相關選項如下：

> `src`　　　　#被複製到遠程主機的本地文件。可以是絕對路徑，也可以是相對路徑。如果路徑是一個目錄，則會遞歸複製，用法類似於"rsync" 
> `content`　　　#用於替換"src"，可以直接指定文件的值
> `dest`　　　　# 必選項，將源文件複製到的遠程主機的**絕對路徑**
> `backup`　　　#當文件內容發生改變後，在覆蓋之前把源文件備份，備份文件包含時間信息
> `directory_mode`　　　　#遞歸設定目錄的權限，默認為系統默認權限
> `force`　　　　#當目標主機包含該文件，但內容不同時，設為"yes"，表示強制覆蓋；設為"no"，表示目標主機的目標位置不存在該文件才複製。默認為"yes" 
> `others`　　　　#所有的file模塊中的選項可以在這裡使用

用法舉例如下：
**①複製文件：**

    [root@server ~]# ansible web -m copy -a 'src=~/hello dest=/data/hello' 
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "dest": "/data/hello", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
        "mode": "0644", 
        "owner": "root", 
        "size": 12, 
        "src": "/root/.ansible/tmp/ansible-tmp-1512437093.55-228281064292921/source", 
        "state": "file", 
        "uid": 0
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "dest": "/data/hello", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
        "mode": "0644", 
        "owner": "root", 
        "size": 12, 
        "src": "/root/.ansible/tmp/ansible-tmp-1512437093.74-44694985235189/source", 
        "state": "file", 
        "uid": 0
    }

**② 給定內容生成文件，並製定權限**

    [root@server ~]# ansible web -m copy -a 'content="I am keer\n" dest=/data/name mode=666'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "checksum": "0421570938940ea784f9d8598dab87f07685b968", 
        "dest": "/data/name", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "497fa8386590a5fc89090725b07f175c", 
        "mode": "0666", 
        "owner": "root", 
        "size": 10, 
        "src": "/root/.ansible/tmp/ansible-tmp-1512437327.37-199512601767687/source", 
        "state": "file", 
        "uid": 0
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "checksum": "0421570938940ea784f9d8598dab87f07685b968", 
        "dest": "/data/name", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "497fa8386590a5fc89090725b07f175c", 
        "mode": "0666", 
        "owner": "root", 
        "size": 10, 
            "src": "/root/.ansible/tmp/ansible-tmp-1512437327.55-218104039503110/source", 
        "state": "file", 
        "uid": 0
    }

　　我們現在可以去查看一下我們生成的文件及其權限：

    [root@server ~]# ansible web -m shell -a 'ls -l /data/'
    192.168.37.122 | SUCCESS | rc=0 >>
    total 28
    -rw-rw-rw-   1 root root   12 Dec  6 09:45 name
    
    192.168.37.133 | SUCCESS | rc=0 >>
    total 40
    -rw-rw-rw- 1 root     root       12 Dec  5 09:45 name

　　可以看出我們的name文件已經生成，並且權限為666。
**③關於覆蓋**
　　我們把文件的內容修改一下，然後選擇覆蓋備份：

    [root@server ~]# ansible web -m copy -a 'content="I am keerya\n" backup=yes dest=/data/name mode=666'
    192.168.37.122 | SUCCESS => {
        "backup_file": "/data/name.4394.2017-12-06@09:46:25~", 
        "changed": true, 
        "checksum": "064a68908ab9971ee85dbc08ea038387598e3778", 
        "dest": "/data/name", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "8ca7c11385856155af52e560f608891c", 
        "mode": "0666", 
        "owner": "root", 
        "size": 12, 
        "src": "/root/.ansible/tmp/ansible-tmp-1512438383.78-228128616784888/source", 
        "state": "file", 
        "uid": 0
    }
    192.168.37.133 | SUCCESS => {
        "backup_file": "/data/name.5962.2017-12-05@09:46:24~", 
        "changed": true, 
        "checksum": "064a68908ab9971ee85dbc08ea038387598e3778", 
        "dest": "/data/name", 
        "gid": 0, 
        "group": "root", 
        "md5sum": "8ca7c11385856155af52e560f608891c", 
        "mode": "0666", 
        "owner": "root", 
        "size": 12, 
        "src": "/root/.ansible/tmp/ansible-tmp-1512438384.0-170718946740009/source", 
        "state": "file", 
        "uid": 0
    }

　　現在我們可以去查看一下：

    [root@server ~]# ansible web -m shell -a 'ls -l /data/'
    192.168.37.122 | SUCCESS | rc=0 >>
    total 28
    -rw-rw-rw-   1 root root   12 Dec  6 09:46 name
    -rw-rw-rw-   1 root root   10 Dec  6 09:45 name.4394.2017-12-06@09:46:25~
    
    192.168.37.133 | SUCCESS | rc=0 >>
    total 40
    -rw-rw-rw- 1 root     root       12 Dec  5 09:46 name
    -rw-rw-rw- 1 root     root       10 Dec  5 09:45 name.5962.2017-12-05@09:46:24~

　　可以看出，我們的源文件已經被備份，我們還可以查看一下`name`文件的內容：

    [root@server ~]# ansible web -m shell -a 'cat /data/name'
    192.168.37.122 | SUCCESS | rc=0 >>
    I am keerya
    
    192.168.37.133 | SUCCESS | rc=0 >>
    I am keerya

　　證明，這正是我們新導入的文件的內容。
**5）file 模塊**
　　該模塊主要用於設置文件的屬性，比如創建文件、創建鏈接文件、刪除文件等。
　　下面是一些常見的命令：

> `force`　　#需要在兩種情況下強制創建軟鏈接，一種是源文件不存在，但之後會建立的情況下；另一種是目標軟鏈接已存在，需要先取消之前的軟鏈，然後創建新的軟鏈，有兩個選項：yes|no 
> `group`　　#定義文件/目錄的屬組。後面可以加上`mode`：定義文件/目錄的權限
> `owner`　　#定義文件/目錄的屬主。後面必須跟上`path`：定義文件/目錄的路徑
> `recurse`　　#遞歸設置文件的屬性，只對目錄有效，後面跟上`src`：被鏈接的源文件路徑，只應用於`state=link`的情況
> `dest`　　#被鏈接到的路徑，只應用於`state=link`的情況
> `state`　　#狀態，有以下選項：
  > `directory`：如果目錄不存在，就創建目錄
  > `file`：即使文件不存在，也不會被創建
  > `link`：創建軟鏈接
  > `hard`：創建硬鏈接
  > `touch`：如果文件不存在，則會創建一個新的文件，如果文件或目錄已存在，則更新其最後修改時間
  > `absent`：刪除目錄、文件或者取消鏈接文件

　　用法舉例如下：
**①創建目錄：**

    [root@server ~]# ansible web -m file -a 'path=/data/app state=directory'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "gid": 0, 
        "group": "root", 
        "mode": "0755", 
        "owner": "root", 
        "path": "/data/app", 
        "size": 6, 
        "state": "directory", 
        "uid": 0
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "gid": 0, 
        "group": "root", 
        "mode": "0755", 
        "owner": "root", 
        "path": "/data/app", 
        "size": 4096, 
        "state": "directory", 
        "uid": 0
    }

　　我們可以查看一下：

    [root@server ~]# ansible web -m shell -a 'ls -l /data'
    192.168.37.122 | SUCCESS | rc=0 >>
    total 28
    drwxr-xr-x   2 root root    6 Dec  6 10:21 app
    
    192.168.37.133 | SUCCESS | rc=0 >>
    total 44
    drwxr-xr-x 2 root     root     4096 Dec  5 10:21 app

　　可以看出，我們的目錄已經創建完成。
**②創建鏈接文件**

    [root@server ~]# ansible web -m file -a 'path=/data/bbb.jpg src=aaa.jpg state=link'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "dest": "/data/bbb.jpg", 
        "gid": 0, 
        "group": "root", 
        "mode": "0777", 
        "owner": "root", 
        "size": 7, 
        "src": "aaa.jpg", 
        "state": "link", 
        "uid": 0
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "dest": "/data/bbb.jpg", 
        "gid": 0, 
        "group": "root", 
        "mode": "0777", 
        "owner": "root", 
        "size": 7, 
        "src": "aaa.jpg", 
        "state": "link", 
        "uid": 0
    }

　　我們可以去查看一下：

    [root@server ~]# ansible web -m shell -a 'ls -l /data'
    192.168.37.122 | SUCCESS | rc=0 >>
    total 28
    -rw-r--r--   1 root root 5649 Dec  5 13:49 aaa.jpg
    lrwxrwxrwx   1 root root    7 Dec  6 10:25 bbb.jpg -> aaa.jpg
    
    192.168.37.133 | SUCCESS | rc=0 >>
    total 44
    -rw-r--r-- 1 root     root     5649 Dec  4 14:44 aaa.jpg
    lrwxrwxrwx 1 root     root        7 Dec  5 10:25 bbb.jpg -> aaa.jpg

　　我們的鏈接文件已經創建成功。
**③刪除文件**

    [root@server ~]# ansible web -m file -a 'path=/data/a state=absent'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "path": "/data/a", 
        "state": "absent"
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "path": "/data/a", 
        "state": "absent"
    }

　　我們可以查看一下：

    [root@server ~]# ansible web -m shell -a 'ls /data/a'
    192.168.37.122 | FAILED | rc=2 >>
    ls: cannot access /data/a: No such file or directory
    
    192.168.37.133 | FAILED | rc=2 >>
    ls: cannot access /data/a: No such file or directory

　　發現已經沒有這個文件了。
　　
**6）fetch 模塊**
　　該模塊用於從遠程某主機獲取（複製）文件到本地。
　　有兩個選項：

> `dest`：用來存放文件的目錄
> `src`：在遠程拉取的文件，並且必須是一個**file**，不能是**目錄**

　　具體舉例如下：

    [root@server ~]# ansible web -m fetch -a 'src=/data/hello dest=/data'  
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "dest": "/data/192.168.37.122/data/hello", 
        "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
        "remote_checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "remote_md5sum": null
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "dest": "/data/192.168.37.133/data/hello", 
        "md5sum": "6f5902ac237024bdd0c176cb93063dc4", 
        "remote_checksum": "22596363b3de40b06f981fb85d82312e8c0ed511", 
        "remote_md5sum": null
    }

　　我們可以在本機上查看一下文件是否複製成功。要注意，文件保存的路徑是我們設置的接收目錄下的`被管制主机ip`目錄下：

    [root@server ~]# cd /data/
    [root@server data]# ls
    1  192.168.37.122  192.168.37.133  fastdfs  web
    [root@server data]# cd 192.168.37.122
    [root@server 192.168.37.122]# ls
    data
    [root@server 192.168.37.122]# cd data/
    [root@server data]# ls
    hello
    [root@server data]# pwd
    /data/192.168.37.122/data

**7）cron 模塊**
　　該模塊適用於管理`cron`計劃任務的。
　　其使用的語法跟我們的`crontab`文件中的語法一致，同時，可以指定以下選項：

> `day=`#日應該運行的工作( 1-31, , /2, ) 
> `hour=`#小時( 0-23, , /2, ) 
> `minute=`#分鐘( 0-59, , /2, ) 
> `month=`#月( 1-12, *, / 2, ) 
> `weekday=`#週( 0-6 for Sunday-Saturday,, ) 
> `job=`#指明運行的命令是什麼
> `name=`#定時任務描述
> `reboot`#任務在重啟時運行，不建議使用，建議使用special_time 
> `special_time`#特殊的時間範圍，參數：reboot （重啟時），annually（每年），monthly（每月），weekly（每週），daily（每天），hourly（每小時）
> `state`#指定狀態，present表示添加定時任務，也是默認設置，absent表示刪除定時任務
> `user`#以哪個用戶的身份執行

　　舉例如下：
**①添加計劃任務**

    [root@server ~]# ansible web -m cron -a 'name="ntp update every 5 min" minute=*/5 job="/sbin/ntpdate 172.17.0.1 &> /dev/null"'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "envs": [], 
        "jobs": [
            "ntp update every 5 min"
        ]
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "envs": [], 
        "jobs": [
            "ntp update every 5 min"
        ]
    }

　　我們可以去查看一下：

    [root@server ~]# ansible web -m shell -a 'crontab -l'
    192.168.37.122 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null
    
    192.168.37.133 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null

　　可以看出，我們的計劃任務已經設置成功了。
**②刪除計劃任務**
　　如果我們的計劃任務添加錯誤，想要刪除的話，則執行以下操作：
　　首先我們查看一下現有的計劃任務：

    [root@server ~]# ansible web -m shell -a 'crontab -l'
    192.168.37.122 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null
    #Ansible: df everyday
    * 15 * * * df -lh >> /tmp/disk_total &> /dev/null
    
    192.168.37.133 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null
    #Ansible: df everyday
    * 15 * * * df -lh >> /tmp/disk_total &> /dev/null

　　然後執行刪除操作：

    [root@server ~]# ansible web -m cron -a 'name="df everyday" hour=15 job="df -lh >> /tmp/disk_total &> /dev/null" state=absent'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "envs": [], 
        "jobs": [
            "ntp update every 5 min"
        ]
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "envs": [], 
        "jobs": [
            "ntp update every 5 min"
        ]
    }

　　刪除完成後，我們再查看一下現有的計劃任務確認一下：

    [root@server ~]# ansible web -m shell -a 'crontab -l'
    192.168.37.122 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null
    
    192.168.37.133 | SUCCESS | rc=0 >>
    #Ansible: ntp update every 5 min
    */5 * * * * /sbin/ntpdate 172.17.0.1 &> /dev/null

　　我們的刪除操作已經成功。
**8）yum 模塊**
　　顧名思義，該模塊主要用於軟件的安裝。
　　其選項如下：

> `name=`　　#所安裝的包的名稱
> `state=`　　# `present`--->安裝，`latest`--->安裝最新的, `absent`--->卸載軟件。
> `update_cache`　　#強制更新yum的緩存
> `conf_file`　　#指定遠程yum安裝時所依賴的配置文件（安裝本地已有的包）。
> `disable_pgp_check`　　#是否禁止GPG checking，只用於`present`or `latest`。
> `disablerepo`　　#臨時禁止使用yum庫。只用於安裝或更新時。
> `enablerepo`　　#臨時使用的yum庫。只用於安裝或更新時。

　　下面我們就來安裝一個包試試看：

    [root@server ~]# ansible web -m yum -a 'name=htop state=present'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "msg": "", 
        "rc": 0, 
        "results": [
            "Loaded plugins: fastestmirror, langpacks\nLoading mirror speeds from cached hostfile\nResolving Dependencies\n--> Running transaction check\n---> Package htop.x86_64 0:2.0.2-1.el7 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package         Arch              Version                Repository       Size\n================================================================================\nInstalling:\n htop            x86_64            2.0.2-1.el7            epel             98 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 98 k\nInstalled size: 207 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : htop-2.0.2-1.el7.x86_64                                      1/1 \n  Verifying  : htop-2.0.2-1.el7.x86_64                                      1/1 \n\nInstalled:\n  htop.x86_64 0:2.0.2-1.el7                                                     \n\nComplete!\n"
        ]
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "msg": "Warning: RPMDB altered outside of yum.\n** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:\nipa-client-4.4.0-12.el7.centos.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-12.el7.centos.x86_64\nipa-client-common-4.4.0-12.el7.centos.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-12.el7.centos.noarch\nipa-common-4.4.0-12.el7.centos.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-12.el7.centos.noarch\n", 
        "rc": 0, 
        "results": [
            "Loaded plugins: fastestmirror, langpacks\nLoading mirror speeds from cached hostfile\nResolving Dependencies\n--> Running transaction check\n---> Package htop.x86_64 0:2.0.2-1.el7 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package         Arch              Version                Repository       Size\n================================================================================\nInstalling:\n htop            x86_64            2.0.2-1.el7            epel             98 k\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 98 k\nInstalled size: 207 k\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : htop-2.0.2-1.el7.x86_64                                      1/1 \n  Verifying  : htop-2.0.2-1.el7.x86_64                                      1/1 \n\nInstalled:\n  htop.x86_64 0:2.0.2-1.el7                                                     \n\nComplete!\n"
        ]
    }

　　安裝成功。
**9）service 模塊**
　　該模塊用於服務程序的管理。
　　其主要選項如下：

> `arguments`#命令行提供額外的參數
> `enabled`#設置開機啟動。
> `name=`#服務名稱
> `runlevel`#開機啟動的級別，一般不用指定。
> `sleep`#在重啟服務的過程中，是否等待。如在服務關閉以後等待2秒再啟動。(定義在劇本中。) 
> `state`#有四種狀態，分別為：`started`--->啟動服務，`stopped`--->停止服務，`restarted`--->重啟服務，`reloaded`--->重載配置

　　下面是一些例子：
**①開啟服務並設置自啟動**

    [root@server ~]# ansible web -m service -a 'name=nginx state=started enabled=true' 
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "enabled": true, 
        "name": "nginx", 
        "state": "started", 
        ……
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "enabled": true, 
        "name": "nginx", 
        "state": "started", 
        ……
    }

　　我們可以去查看一下端口是否打開：

    [root@server ~]# ansible web -m shell -a 'ss -ntl'
    192.168.37.122 | SUCCESS | rc=0 >>
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port              
    LISTEN     0      128          *:80                       *:*                                  
    
    192.168.37.133 | SUCCESS | rc=0 >>
    State      Recv-Q Send-Q Local Address:Port               Peer Address:Port                    
    LISTEN     0      128          *:80                       *:*                  

　　可以看出我們的80端口已經打開。
**②關閉服務**
　　我們也可以通過該模塊來關閉我們的服務：

    [root@server ~]# ansible web -m service -a 'name=nginx state=stopped'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "name": "nginx", 
        "state": "stopped", 
        ……
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "name": "nginx", 
        "state": "stopped", 
        ……
    }

　　一樣的，我們來查看一下端口：

    [root@server ~]# ansible web -m shell -a 'ss -ntl | grep 80'
    192.168.37.122 | FAILED | rc=1 >>
    
    192.168.37.133 | FAILED | rc=1 >>

　　可以看出，我們已經沒有80端口了，說明我們的nginx服務已經關閉了。
**10）user 模塊**
　　該模塊主要是用來管理用戶賬號。
　　其主要選項如下：

> `comment`　　#用戶的描述信息
> `createhome`　　#是否創建家目錄
> `force`　　#在使用state=absent時,行為與userdel –force一致. 
> `group`　　#指定基本組
> `groups`　　#指定附加組，如果指定為(groups=)表示刪除所有組
> `home`　　#指定用戶家目錄
> `move_home`　　#如果設置為home=時,試圖將用戶主目錄移動到指定的目錄
> `name`　　#指定用戶名
> `non_unique`　　#該選項允許改變非唯一的用戶ID值
> `password`　　#指定用戶密碼
> `remove`　　#在使用state=absent時,行為是與userdel – remove一致
> `shell`　　#指定默認shell 
> `state`　　#設置帳號狀態，不指定為創建，指定值為absent表示刪除
> `system`　　#當創建一個用戶，設置這個用戶是系統用戶。這個設置不能更改現有用戶
> `uid`　　#指定用戶的uid

　　舉例如下：
**①添加一個用戶並指定其uid**

    [root@server ~]# ansible web -m user -a 'name=keer uid=11111'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "comment": "", 
        "createhome": true, 
        "group": 11111, 
        "home": "/home/keer", 
        "name": "keer", 
        "shell": "/bin/bash", 
        "state": "present", 
        "stderr": "useradd: warning: the home directory already exists.\nNot copying any file from skel directory into it.\nCreating mailbox file: File exists\n", 
        "system": false, 
        "uid": 11111
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "comment": "", 
        "createhome": true, 
        "group": 11111, 
        "home": "/home/keer", 
        "name": "keer", 
        "shell": "/bin/bash", 
        "state": "present", 
        "stderr": "useradd: warning: the home directory already exists.\nNot copying any file from skel directory into it.\nCreating mailbox file: File exists\n", 
        "system": false, 
        "uid": 11111
    }

　　添加完成，我們可以去查看一下：

    [root@server ~]# ansible web -m shell -a 'cat /etc/passwd |grep keer'
    192.168.37.122 | SUCCESS | rc=0 >>
    keer:x:11111:11111::/home/keer:/bin/bash
    
    192.168.37.133 | SUCCESS | rc=0 >>
    keer:x:11111:11111::/home/keer:/bin/bash

**② 刪除用戶**

    [root@server ~]# ansible web -m user -a 'name=keer state=absent'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "force": false, 
        "name": "keer", 
        "remove": false, 
        "state": "absent"
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "force": false, 
        "name": "keer", 
        "remove": false, 
        "state": "absent"
    }

　　一樣的，刪除之後，我們去看一下：

    [root@server ~]# ansible web -m shell -a 'cat /etc/passwd |grep keer'
    192.168.37.122 | FAILED | rc=1 >>
    
    192.168.37.133 | FAILED | rc=1 >>

　　發現已經沒有這個用戶了。
**11）group 模塊**
　　該模塊主要用於添加或刪除組。
　　常用的選項如下：

> `gid=`　　#設置組的GID號
> `name=`　　#指定組的名稱
> `state=`　　#指定組的狀態，默認為創建，設置值為`absent`為刪除
> `system=`　　#設置值為`yes`，表示創建為系統組

　　舉例如下：
**①創建組**

    [root@server ~]# ansible web -m group -a 'name=sanguo gid=12222'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "gid": 12222, 
        "name": "sanguo", 
        "state": "present", 
        "system": false
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "gid": 12222, 
        "name": "sanguo", 
        "state": "present", 
        "system": false
    }

　　創建過後，我們來查看一下：

    [root@server ~]# ansible web -m shell -a 'cat /etc/group | grep 12222' 
    192.168.37.122 | SUCCESS | rc=0 >>
    sanguo:x:12222:
    
    192.168.37.133 | SUCCESS | rc=0 >>
    sanguo:x:12222:

　　可以看出，我們的組已經創建成功了。
**②刪除組**

    [root@server ~]# ansible web -m group -a 'name=sanguo state=absent'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "name": "sanguo", 
        "state": "absent"
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "name": "sanguo", 
        "state": "absent"
    }

　　照例查看一下：

    [root@server ~]# ansible web -m shell -a 'cat /etc/group | grep 12222' 
    192.168.37.122 | FAILED | rc=1 >>
    
    192.168.37.133 | FAILED | rc=1 >>

　　已經沒有這個組的相關信息了。
**12）script 模塊**
　　該模塊用於將本機的腳本在被管理端的機器上運行。
　　該模塊直接指定腳本的路徑即可，我們通過例子來看一看到底如何使用的：
　　首先，我們寫一個腳本，並給其加上執行權限：

    [root@server ~]# vim /tmp/df.sh
        #!/bin/bash
    
        date >> /tmp/disk_total.log
        df -lh >> /tmp/disk_total.log 
    [root@server ~]# chmod +x /tmp/df.sh 

　　然後，我們直接運行命令來實現在被管理端執行該腳本：

    [root@server ~]# ansible web -m script -a '/tmp/df.sh'
    192.168.37.122 | SUCCESS => {
        "changed": true, 
        "rc": 0, 
        "stderr": "Shared connection to 192.168.37.122 closed.\r\n", 
        "stdout": "", 
        "stdout_lines": []
    }
    192.168.37.133 | SUCCESS => {
        "changed": true, 
        "rc": 0, 
        "stderr": "Shared connection to 192.168.37.133 closed.\r\n", 
        "stdout": "", 
        "stdout_lines": []
    }

　　照例查看一下文件內容：

    [root@server ~]# ansible web -m shell -a 'cat /tmp/disk_total.log'
    192.168.37.122 | SUCCESS | rc=0 >>
    Tue Dec  5 15:58:21 CST 2017
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda2        47G  4.4G   43G  10% /
    devtmpfs        978M     0  978M   0% /dev
    tmpfs           993M   84K  993M   1% /dev/shm
    tmpfs           993M  9.1M  984M   1% /run
    tmpfs           993M     0  993M   0% /sys/fs/cgroup
    /dev/sda3        47G   33M   47G   1% /app
    /dev/sda1       950M  153M  798M  17% /boot
    tmpfs           199M   16K  199M   1% /run/user/42
    tmpfs           199M     0  199M   0% /run/user/0
    
    192.168.37.133 | SUCCESS | rc=0 >>
    Tue Dec  5 15:58:21 CST 2017
    Filesystem      Size  Used Avail Use% Mounted on
    /dev/sda2        46G  4.1G   40G  10% /
    devtmpfs        898M     0  898M   0% /dev
    tmpfs           912M   84K  912M   1% /dev/shm
    tmpfs           912M  9.0M  903M   1% /run
    tmpfs           912M     0  912M   0% /sys/fs/cgroup
    /dev/sda3       3.7G   15M  3.4G   1% /app
    /dev/sda1       1.9G  141M  1.6G   9% /boot
    tmpfs           183M   16K  183M   1% /run/user/42
    tmpfs           183M     0  183M   0% /run/user/0

　　可以看出已經執行成功了。
**13）setup 模塊**
　　該模塊主要用於收集信息，是通過調用facts組件來實現的。
　　facts組件是Ansible用於採集被管機器設備信息的一個功能，我們可以使用setup模塊查機器的所有facts信息，可以使用filter來查看指定信息。整個facts信息被包裝在一個JSON格式的數據結構中，ansible_facts是最上層的值。
　　facts就是變量，內建變量。每個主機的各種信息，cpu顆數、內存大小等。會存在facts中的某個變量中。調用後返回很多對應主機的信息，在後面的操作中可以根據不同的信息來做不同的操作。如redhat系列用yum安裝，而debian系列用apt來安裝軟件。
**①查看信息**
　　我們可以直接用命令獲取到變量的值，具體我們來看看例子：

    [root@server ~]# ansible web -m setup -a 'filter="*mem*"'   #查看内存
    192.168.37.122 | SUCCESS => {
        "ansible_facts": {
            "ansible_memfree_mb": 1116, 
            "ansible_memory_mb": {
                "nocache": {
                    "free": 1397, 
                    "used": 587
                }, 
                "real": {
                    "free": 1116, 
                    "total": 1984, 
                    "used": 868
                }, 
                "swap": {
                    "cached": 0, 
                    "free": 3813, 
                    "total": 3813, 
                    "used": 0
                }
            }, 
            "ansible_memtotal_mb": 1984
        }, 
        "changed": false
    }
    192.168.37.133 | SUCCESS => {
        "ansible_facts": {
            "ansible_memfree_mb": 1203, 
            "ansible_memory_mb": {
                "nocache": {
                    "free": 1470, 
                    "used": 353
                }, 
                "real": {
                    "free": 1203, 
                    "total": 1823, 
                    "used": 620
                }, 
                "swap": {
                    "cached": 0, 
                    "free": 3813, 
                    "total": 3813, 
                    "used": 0
                }
            }, 
            "ansible_memtotal_mb": 1823
        }, 
        "changed": false
    }

　　我們可以通過命令查看一下內存的大小以確認一下是否一致：

    [root@server ~]# ansible web -m shell -a 'free -m'
    192.168.37.122 | SUCCESS | rc=0 >>
                  total        used        free      shared  buff/cache   available
    Mem:           1984         404        1122           9         457        1346
    Swap:          3813           0        3813
    
    192.168.37.133 | SUCCESS | rc=0 >>
                  total        used        free      shared  buff/cache   available
    Mem:           1823         292        1207           9         323        1351
    Swap:          3813           0        3813

　　可以看出信息是一致的。
**②保存信息**
　　我們的setup模塊還有一個很好用的功能就是可以保存我們所篩選的信息至我們的主機上，同時，文件名為我們被管制的主機的IP，這樣方便我們知道是哪台機器出的問題。
　　我們可以看一看例子：

    [root@server tmp]# ansible web -m setup -a 'filter="*mem*"' --tree /tmp/facts
    192.168.37.122 | SUCCESS => {
        "ansible_facts": {
            "ansible_memfree_mb": 1115, 
            "ansible_memory_mb": {
                "nocache": {
                    "free": 1396, 
                    "used": 588
                }, 
                "real": {
                    "free": 1115, 
                    "total": 1984, 
                    "used": 869
                }, 
                "swap": {
                    "cached": 0, 
                    "free": 3813, 
                    "total": 3813, 
                    "used": 0
                }
            }, 
            "ansible_memtotal_mb": 1984
        }, 
        "changed": false
    }
    192.168.37.133 | SUCCESS => {
        "ansible_facts": {
            "ansible_memfree_mb": 1199, 
            "ansible_memory_mb": {
                "nocache": {
                    "free": 1467, 
                    "used": 356
                }, 
                "real": {
                    "free": 1199, 
                    "total": 1823, 
                    "used": 624
                }, 
                "swap": {
                    "cached": 0, 
                    "free": 3813, 
                    "total": 3813, 
                    "used": 0
                }
            }, 
            "ansible_memtotal_mb": 1823
        }, 
        "changed": false
    }

　　然後我們可以去查看一下：

    [root@server ~]# cd /tmp/facts/
    [root@server facts]# ls
    192.168.37.122  192.168.37.133
    [root@server facts]# cat 192.168.37.122 
    {"ansible_facts": {"ansible_memfree_mb": 1115, "ansible_memory_mb": {"nocache": {"free": 1396, "used": 588}, "real": {"free": 1115, "total": 1984, "used": 869}, "swap": {"cached": 0, "free": 3813, "total": 3813, "used": 0}}, "ansible_memtotal_mb": 1984}, "changed": false}


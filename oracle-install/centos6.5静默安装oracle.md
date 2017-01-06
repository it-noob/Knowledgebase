centos6.5静默安装oracle

由于最近尝试在Linux下安装Oracle，公司运维提供的教程里都是用图形界面安装的，而我本地的centos是没有安装桌面组件的，选用的basic server，所以一直在想弄一个静默安装，特此整理，过程如下，一次成功；安装环境为虚拟机，实体机本人未亲自测试。

本此案中省略centos安装，以及ssh工具和sftp工具的使用介绍，每个操作的用户均以#(root用户)和$(oracle用户：安装过程中会创建)

1. 在根目录下新建文件夹

   `# mkdir /opt/oracle_software`

2. 讲oracle11gR2的连个安装文件上传至上一步所创建目录中，文件可以去Oracle官网下载，使用unzip命令解压缩，在oracle_software目录下操作

   `# unzip linux.x64_11gR2_database_1of2.zip`

   `# unzip linux.x64_11gR2_database_2of2.zip`

3. 安装所需要的包，包不要版本号，通过yum进行安装，具体的版本号是根据系统版本来决定需要的版本号是什么，通过输入版本号前的包名会自动找到和系统相符的包。执行以下命令一次性安装完成，如果无法上网只能一个个下载再上传安装了，不作讨论。对于安装时可以先检查要的包是否已安装，已安装的包可以不用再安装了，检查命令 `rpm -qa | grep binutils`

   ```shell
    # yum -y install \

   binutils \

   compat-libcap1  \

   compat-libstdc++-33 \

   compat-libstdc++-33*.i686 \

   elfutils-libelf-devel \

   gcc \

   gcc-c++ \

   glibc*.i686 \

   glibc \

   glibc-devel \

   glibc-devel*.i686 \

   ksh \

   libgcc*.i686 \

   libgcc \

   libstdc++ \

   libstdc++*.i686 \

   libstdc++-devel \

   libstdc++-devel*.i686 \

   libaio \

   libaio*.i686 \

   libaio-devel \

   libaio-devel*.i686 \

   make \

   sysstat \

   unixODBC \

   unixODBC*.i686 \

   unixODBC-devel \

   unixODBC-devel*.i686 \

   libXp
   ```

4. 创建用户组，并修改oracle用户登录密码

   ```shell
   # groupadd oinstall
   # groupadd dba
   # useradd -g oinstall -G dba oracle
   # passwd oracle
   --- 根据提示输入oracle用户的新密码
   ```

5. 创建安装目录并授权：此处有些资料显示是使用775的权限，但是我在安装过程中使用775权限不能安装成功，出现错误权限不够，所以我采用的是777的高权限

   ```shell
   # mkdir -p /opt/oracle
   # mkdir -p /opt/oracle/11gR2
   # mkdir -p /opt/oracle/dbrecovery
   # mkdir -p /opt/oracle/oradata
   # chown -R oracle:oinstall /opt/oracle
   # chown -R oracle:oinstall /opt/oracle_software
   # chmod -R 777 /opt/oracle
   # chmod -R 777 /opt/oracle_software
   ```

6. 设置环境变量

   ```shell
   # vi /home/oracle/.bash_profile
   ---将下段添加到上述文件中
   export ORACLE_BASE=/opt/oracle
   export ORACLE_HOME=$ORACLE_BASE/11gR2
   export ORACLE_SID=orcl
   export PATH=$PATH:$ORACLE_HOME/bin
   ```

7. 使环境变量生效

   `# source /home/oracle/.bash_profile`

8. 添加资源要求

   ```shell
   # vi /etc/security/limits.conf
   ---将下段添加到上述文件中
   oracle soft nproc 2047
   oracle hard nporc 16384
   oracle soft nofile 1024
   oracle hard nofile 65536
   oracle soft stack 10240
   ```

9. 配置Linux内核参数

   ```shell
   # vi/etc/sysctl.conf
   ---将以下段添加到上述文件中，添加时请将文件中原有kernel.shmmax 和kernel.shmmni注释，以下值也可以先通过查询本系统参数将其填入

   fs.aio-max-nr = 1048576
   fs.file-max = 6815744
   kernel.shmall = 2097152
   kernel.shmmax = 536870912
   kernel.shmmni = 4096
   kernel.sem = 250 32000 100 128
   net.ipv4.ip_local_port_range = 9000 65500
   net.core.rmem_default = 262144
   net.core.rmem_max = 4194304
   net.core.wmem_default = 262144
   net.core.wmem_max = 1048586
   ```

10. 执行命令使配置生效

    `# /sbin/sysctl -p`

11. 复制响应文件

    `# cp /opt/oracle_software/database/response/db_install.rsp  /opt/oracle`

12. 查看主机名并修改：其中centos为系统主机名，oracle.centos接下来在响应文件需要单独使用

    ```shell
    ---注意：/etc/sysconfig/network中的hostname要与/etc/hosts中的一致
    # vi /etc/sysconfig/network
    NETWORKING=yes
    HOSTNAME=centos
    # vi /etc/hosts
    127.0.0.1   localhost centos localhost4 localhost4.localdomain4 oracle.centos
    ::1         localhost centos localhost6 localhost6.localdomain6 oracle.centos
    ```

13. 编辑相应文件`$ vi /opt/oracle/db_install.rsp`

    以下是完整的配置文件

    ```shell
    ####################################################################
    ## Copyright(c) Oracle Corporation 1998,2008. All rights reserved.##
    ##                                                                ##
    ## Specify values for the variables listed below to customize     ##
    ## your installation.                                             ##
    ##                                                                ##
    ## Each variable is associated with a comment. The comment        ##
    ## can help to populate the variables with the appropriate        ##
    ## values.							  ##
    ##                                                                ##
    ## IMPORTANT NOTE: This file contains plain text passwords and    ##
    ## should be secured to have read permission only by oracle user  ##
    ## or db administrator who owns this installation.                ##
    ##                                                                ##
    ####################################################################

    #------------------------------------------------------------------------------
    # Do not change the following system generated value. 
    #------------------------------------------------------------------------------(保持默认不要修改)
    oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v11_2_0

    #------------------------------------------------------------------------------
    # Specify the installation option.
    # It can be one of the following:
    # 1. INSTALL_DB_SWONLY
    # 2. INSTALL_DB_AND_CONFIG
    # 3. UPGRADE_DB
    #-------------------------------------------------------------------------------
    oracle.install.option=INSTALL_DB_AND_CONFIG

    #-------------------------------------------------------------------------------
    # Specify the hostname of the system as set during the install. It can be used
    # to force the installation to use an alternative hostname rather than using the
    # first hostname found on the system. (e.g., for systems with multiple hostnames 
    # and network interfaces)
    #-------------------------------------------------------------------------------(上一步设置的主机名)
    ORACLE_HOSTNAME=oracle.centos

    #-------------------------------------------------------------------------------
    # Specify the Unix group to be set for the inventory directory.  
    #-------------------------------------------------------------------------------
    UNIX_GROUP_NAME=oinstall

    #-------------------------------------------------------------------------------
    # Specify the location which holds the inventory files.
    #-------------------------------------------------------------------------------
    INVENTORY_LOCATION=/opt/oracle/oraInventory

    #-------------------------------------------------------------------------------
    # Specify the languages in which the components will be installed.             
    #
    # en   : English                  ja   : Japanese                  
    # fr   : French                   ko   : Korean                    
    # ar   : Arabic                   es   : Latin American Spanish    
    # bn   : Bengali                  lv   : Latvian                   
    # pt_BR: Brazilian Portuguese     lt   : Lithuanian                
    # bg   : Bulgarian                ms   : Malay                     
    # fr_CA: Canadian French          es_MX: Mexican Spanish           
    # ca   : Catalan                  no   : Norwegian                 
    # hr   : Croatian                 pl   : Polish                    
    # cs   : Czech                    pt   : Portuguese                
    # da   : Danish                   ro   : Romanian                  
    # nl   : Dutch                    ru   : Russian                   
    # ar_EG: Egyptian                 zh_CN: Simplified Chinese        
    # en_GB: English (Great Britain)  sk   : Slovak                    
    # et   : Estonian                 sl   : Slovenian                 
    # fi   : Finnish                  es_ES: Spanish                   
    # de   : German                   sv   : Swedish                   
    # el   : Greek                    th   : Thai                      
    # iw   : Hebrew                   zh_TW: Traditional Chinese       
    # hu   : Hungarian                tr   : Turkish                   
    # is   : Icelandic                uk   : Ukrainian                 
    # in   : Indonesian               vi   : Vietnamese                
    # it   : Italian                                                   
    #
    # Example : SELECTED_LANGUAGES=en,fr,ja
    #------------------------------------------------------------------------------
    SELECTED_LANGUAGES=en,zh_CN

    #------------------------------------------------------------------------------
    # Specify the complete path of the Oracle Home.
    #------------------------------------------------------------------------------
    ORACLE_HOME=/opt/oracle/11gR2

    #------------------------------------------------------------------------------
    # Specify the complete path of the Oracle Base. 
    #------------------------------------------------------------------------------
    ORACLE_BASE=/opt/oracle

    #------------------------------------------------------------------------------
    # Specify the installation edition of the component.                        
    #                                                             
    # The value should contain only one of these choices.        
    # EE     : Enterprise Edition                                
    # SE     : Standard Edition                                  
    # SEONE  : Standard Edition One
    # PE     : Personal Edition (WINDOWS ONLY)
    #------------------------------------------------------------------------------
    oracle.install.db.InstallEdition=EE

    #------------------------------------------------------------------------------
    # This variable is used to enable or disable custom install.
    #
    # true  : Components mentioned as part of 'customComponents' property
    #         are considered for install.
    # false : Value for 'customComponents' is not considered.
    #------------------------------------------------------------------------------(此处为false则不安装下述内容oracle.install.db.customComponents，安装时间为半小时左右，如为true,则需安装oracle.install.db.customComponents中的内容，安装时间为一个半小时左右)
    oracle.install.db.isCustomInstall=false

    #------------------------------------------------------------------------------
    # This variable is considered only if 'IsCustomInstall' is set to true. 
    #
    # Description: List of Enterprise Edition Options you would like to install.
    #
    #              The following choices are available. You may specify any
    #              combination of these choices.  The components you choose should
    #              be specified in the form "internal-component-name:version"
    #              Below is a list of components you may specify to install.
    #        
    #              oracle.rdbms.partitioning:11.2.0.1.0 - Oracle Partitioning
    #              oracle.rdbms.dm:11.2.0.1.0 - Oracle Data Mining
    #              oracle.rdbms.dv:11.2.0.1.0 - Oracle Database Vault 
    #              oracle.rdbms.lbac:11.2.0.1.0 - Oracle Label Security
    #              oracle.rdbms.rat:11.2.0.1.0 - Oracle Real Application Testing 
    #              oracle.oraolap:11.2.0.1.0 - Oracle OLAP
    #------------------------------------------------------------------------------
    oracle.install.db.customComponents=oracle.server:11.2.0.1.0,oracle.sysman.ccr:10.2.7.0.0,oracle.xdk:11.2.0.1.0,oracle.rdbms.oci:11.2.0.1.0,oracle.network:11.2.0.1.0,oracle.network.listener:11.2.0.1.0,oracle.rdbms:11.2.0.1.0,oracle.options:11.2.0.1.0,oracle.rdbms.partitioning:11.2.0.1.0,oracle.oraolap:11.2.0.1.0,oracle.rdbms.dm:11.2.0.1.0,oracle.rdbms.dv:11.2.0.1.0,orcle.rdbms.lbac:11.2.0.1.0,oracle.rdbms.rat:11.2.0.1.0

    ###############################################################################
    #                                                                             #
    # PRIVILEGED OPERATING SYSTEM GROUPS                                  	      #
    # ------------------------------------------                                  #
    # Provide values for the OS groups to which OSDBA and OSOPER privileges       #
    # needs to be granted. If the install is being performed as a member of the   #		
    # group "dba", then that will be used unless specified otherwise below.	      #
    #                                                                             #
    ###############################################################################

    #------------------------------------------------------------------------------
    # The DBA_GROUP is the OS group which is to be granted OSDBA privileges.
    #------------------------------------------------------------------------------
    oracle.install.db.DBA_GROUP=dba

    #------------------------------------------------------------------------------
    # The OPER_GROUP is the OS group which is to be granted OSOPER privileges.
    #------------------------------------------------------------------------------
    oracle.install.db.OPER_GROUP=oinstall

    #------------------------------------------------------------------------------
    # Specify the cluster node names selected during the installation.
    #------------------------------------------------------------------------------
    oracle.install.db.CLUSTER_NODES=

    #------------------------------------------------------------------------------
    # Specify the type of database to create.
    # It can be one of the following:
    # - GENERAL_PURPOSE/TRANSACTION_PROCESSING          
    # - DATA_WAREHOUSE                                
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.type=GENERAL_PURPOSE

    #------------------------------------------------------------------------------
    # Specify the Starter Database Global Database Name. 
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.globalDBName=orcl

    #------------------------------------------------------------------------------
    # Specify the Starter Database SID.
    #------------------------------------------------------------------------------(步骤6设置的SID)
    oracle.install.db.config.starterdb.SID=orcl

    #------------------------------------------------------------------------------
    # Specify the Starter Database character set.
    #                                              
    # It can be one of the following:
    # AL32UTF8, WE8ISO8859P15, WE8MSWIN1252, EE8ISO8859P2,
    # EE8MSWIN1250, NE8ISO8859P10, NEE8ISO8859P4, BLT8MSWIN1257,
    # BLT8ISO8859P13, CL8ISO8859P5, CL8MSWIN1251, AR8ISO8859P6,
    # AR8MSWIN1256, EL8ISO8859P7, EL8MSWIN1253, IW8ISO8859P8,
    # IW8MSWIN1255, JA16EUC, JA16EUCTILDE, JA16SJIS, JA16SJISTILDE,
    # KO16MSWIN949, ZHS16GBK, TH8TISASCII, ZHT32EUC, ZHT16MSWIN950,
    # ZHT16HKSCS, WE8ISO8859P9, TR8MSWIN1254, VN8MSWIN1258
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.characterSet=AL32UTF8

    #------------------------------------------------------------------------------
    # This variable should be set to true if Automatic Memory Management 
    # in Database is desired.
    # If Automatic Memory Management is not desired, and memory allocation
    # is to be done manually, then set it to false.
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.memoryOption=true

    #------------------------------------------------------------------------------
    # Specify the total memory allocation for the database. Value(in MB) should be
    # at least 256 MB, and should not exceed the total physical memory available 
    # on the system.
    # Example: oracle.install.db.config.starterdb.memoryLimit=512
    #------------------------------------------------------------------------------(根据自己机子配置选择，虚拟机我配置了4G内存所以设定了1024若失败修改此项重新执行)

    oracle.install.db.config.starterdb.memoryLimit=1024

    #------------------------------------------------------------------------------
    # This variable controls whether to load Example Schemas onto the starter
    # database or not.
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.installExampleSchemas=false

    #------------------------------------------------------------------------------
    # This variable includes enabling audit settings, configuring password profiles
    # and revoking some grants to public. These settings are provided by default. 
    # These settings may also be disabled.    
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.enableSecuritySettings=true

    ###############################################################################
    #                                                                             #
    # Passwords can be supplied for the following four schemas in the	      #
    # starter database:      						      #
    #   SYS                                                                       #
    #   SYSTEM                                                                    #
    #   SYSMAN (used by Enterprise Manager)                                       #
    #   DBSNMP (used by Enterprise Manager)                                       #
    #                                                                             #
    # Same password can be used for all accounts (not recommended) 		      #
    # or different passwords for each account can be provided (recommended)       #
    #                                                                             #
    ###############################################################################

    #------------------------------------------------------------------------------
    # This variable holds the password that is to be used for all schemas in the
    # starter database.
    #-------------------------------------------------------------------------------(全局密码，安装过程可能会警告，忽略即可)
    oracle.install.db.config.starterdb.password.ALL=oracledb

    #-------------------------------------------------------------------------------
    # Specify the SYS password for the starter database.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.password.SYS=

    #-------------------------------------------------------------------------------
    # Specify the SYSTEM password for the starter database.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.password.SYSTEM=

    #-------------------------------------------------------------------------------
    # Specify the SYSMAN password for the starter database.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.password.SYSMAN=

    #-------------------------------------------------------------------------------
    # Specify the DBSNMP password for the starter database.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.password.DBSNMP=

    #-------------------------------------------------------------------------------
    # Specify the management option to be selected for the starter database. 
    # It can be one of the following:
    # 1. GRID_CONTROL
    # 2. DB_CONTROL
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.control=DB_CONTROL

    #-------------------------------------------------------------------------------
    # Specify the Management Service to use if Grid Control is selected to manage 
    # the database.      
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.gridcontrol.gridControlServiceURL=

    #-------------------------------------------------------------------------------
    # This variable indicates whether to receive email notification for critical 
    # alerts when using DB control.   
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.dbcontrol.enableEmailNotification=false

    #-------------------------------------------------------------------------------
    # Specify the email address to which the notifications are to be sent.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.dbcontrol.emailAddress=

    #-------------------------------------------------------------------------------
    # Specify the SMTP server used for email notifications.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.dbcontrol.SMTPServer=


    ###############################################################################
    #                                                                             #
    # SPECIFY BACKUP AND RECOVERY OPTIONS                                 	      #
    # ------------------------------------		                              #
    # Out-of-box backup and recovery options for the database can be mentioned    #
    # using the entries below.						      #	
    #                                                                             #
    ###############################################################################

    #------------------------------------------------------------------------------
    # This variable is to be set to false if automated backup is not required. Else 
    # this can be set to true.
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.automatedBackup.enable=false

    #------------------------------------------------------------------------------
    # Regardless of the type of storage that is chosen for backup and recovery, if 
    # automated backups are enabled, a job will be scheduled to run daily at
    # 2:00 AM to backup the database. This job will run as the operating system 
    # user that is specified in this variable.
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.automatedBackup.osuid=

    #-------------------------------------------------------------------------------
    # Regardless of the type of storage that is chosen for backup and recovery, if 
    # automated backups are enabled, a job will be scheduled to run daily at
    # 2:00 AM to backup the database. This job will run as the operating system user
    # specified by the above entry. The following entry stores the password for the
    # above operating system user.
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.automatedBackup.ospwd=

    #-------------------------------------------------------------------------------
    # Specify the type of storage to use for the database.
    # It can be one of the following:
    # - FILE_SYSTEM_STORAGE
    # - ASM_STORAGE
    #------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.storageType=FILE_SYSTEM_STORAGE

    #-------------------------------------------------------------------------------
    # Specify the database file location which is a directory for datafiles, control
    # files, redo logs.         
    #
    # Applicable only when oracle.install.db.config.starterdb.storage=FILE_SYSTEM 
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.fileSystemStorage.dataLocation=/opt/oracle/oradata

    #-------------------------------------------------------------------------------
    # Specify the backup and recovery location.
    #
    # Applicable only when oracle.install.db.config.starterdb.storage=FILE_SYSTEM 
    #-------------------------------------------------------------------------------
    oracle.install.db.config.starterdb.fileSystemStorage.recoveryLocation=/opt/oracle/dbrecovery

    #-------------------------------------------------------------------------------
    # Specify the existing ASM disk groups to be used for storage.
    #
    # Applicable only when oracle.install.db.config.starterdb.storage=ASM
    #-------------------------------------------------------------------------------
    oracle.install.db.config.asm.diskGroup=

    #-------------------------------------------------------------------------------
    # Specify the password for ASMSNMP user of the ASM instance.                  
    #
    # Applicable only when oracle.install.db.config.starterdb.storage=ASM_SYSTEM 
    #-------------------------------------------------------------------------------
    oracle.install.db.config.asm.ASMSNMPPassword=

    #------------------------------------------------------------------------------
    # Specify the My Oracle Support Account Username.
    #
    #  Example   : MYORACLESUPPORT_USERNAME=metalink
    #------------------------------------------------------------------------------
    MYORACLESUPPORT_USERNAME=

    #------------------------------------------------------------------------------
    # Specify the My Oracle Support Account Username password.
    #
    # Example    : MYORACLESUPPORT_PASSWORD=password
    #------------------------------------------------------------------------------
    MYORACLESUPPORT_PASSWORD=

    #------------------------------------------------------------------------------
    # Specify whether to enable the user to set the password for
    # My Oracle Support credentials. The value can be either true or false.
    # If left blank it will be assumed to be false.
    #
    # Example    : SECURITY_UPDATES_VIA_MYORACLESUPPORT=true
    #------------------------------------------------------------------------------
    SECURITY_UPDATES_VIA_MYORACLESUPPORT=false

    #------------------------------------------------------------------------------
    # Specify whether user wants to give any proxy details for connection. 
    # The value can be either true or false. If left blank it will be assumed
    # to be false.
    #
    # Example    : DECLINE_SECURITY_UPDATES=false
    #------------------------------------------------------------------------------(必须为true)
    DECLINE_SECURITY_UPDATES=true

    #------------------------------------------------------------------------------
    # Specify the Proxy server name. Length should be greater than zero.
    #
    # Example    : PROXY_HOST=proxy.domain.com 
    #------------------------------------------------------------------------------
    PROXY_HOST=

    #------------------------------------------------------------------------------
    # Specify the proxy port number. Should be Numeric and atleast 2 chars.
    #
    # Example    : PROXY_PORT=25 
    #------------------------------------------------------------------------------
    PROXY_PORT=

    #------------------------------------------------------------------------------
    # Specify the proxy user name. Leave PROXY_USER and PROXY_PWD 
    # blank if your proxy server requires no authentication.
    #
    # Example    : PROXY_USER=username 
    #------------------------------------------------------------------------------
    PROXY_USER=

    #------------------------------------------------------------------------------
    # Specify the proxy password. Leave PROXY_USER and PROXY_PWD  
    # blank if your proxy server requires no authentication.
    #
    # Example    : PROXY_PWD=password 
    #------------------------------------------------------------------------------
    PROXY_PWD=
    ```

14. 授权`$ chmod 777 /opt/oracle/db_install.rsp`

15. 安装

    ```shell
    $ /opt/oracle_software/database/runInstaller -ignoreSysPrereqs -ignorePrereq -silent -nowelcome -responseFile /opt/oracle/db_install.rsp
    ```

    安装过程中后台进程在运行着，终端上看不到所有的提示信息，当看见提示日志文件存储路径时可以重新打开一个终端登录到oracle用户下进行实时查看

    例入：

    `tail -f /opt/oracle/oraInventory/logs/installActions2016-10-22_02-30-53PM.log`

16. 观察安装情况，过程中如果出现FATAL需要查看问题重新安装，出现WARNNING警告无需理会。

    当看见启动安装的终端中出现"Successfully Setup Software"说明安装成功，按回车键结束安装。

17. 运行以下脚本进行配置

    ```shell
    # opt/oracle/oraInventory/orainstRoot.sh
    # /opt/oracle/11gR2/root.sh
    ```

    如果提示找不到上述文件说明未安装成功

18. 登录sqlplus并配置，检查，设定系统重启自动启动服务和监听的自行配置

    ```sql
    ---使用sqlplus
    $ sqlplus / as sysdba
    ALTER system SET processes=2000 scope=spfile;
    ALTER system SET sessions=2200 scope=spfile;
    ALTER profile DEFAULT LIMIT FAILED_LOGIN_ATTEMPTS unlimited;
    ALTER profile DEFAULT LIMIT PASSWORD_LIFE_TIME unlimited;
    ---停止实例
    shutdown immediate;
    ---重新开启
    startup open;
    ---查询验证
    select * from scott.emp;
    ---启动监听
    lsnrctl start
    ---查询监听状态
    lsnrctl status
    ```

    ​


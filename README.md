sepolicy是SEAndroid的源码
***
SEAndroid ：全称Security Enhancements for Android，以SELinux为核心  

SELinux ：是美国国家安全局搞的一个Modules，是开源的，后来Google将其应用到自家的Android平台上来  

DAC : (Discretionary Access Control )  在SELinux出现之前，Linux系统所用的安全模型。在这个模型里面，进程所拥有的权限与执行它的用户权限相同，

MAC : (Mandatory Access Control)
SELinux的安全模型，进程所拥有的权限，都必须在安全配置文件（.te）里面定义。这个进程只有在配置文件中定义的权限。

Flask : MAC的大概架构  
flask定义了    

* Subject : process  进程,被称之为客体  

* Object: file, socket, IPC object… 对象，是客体访问的对象  

* Security Context : 安全信息，是一个字符串，用来表示subject或object的安全属性  

* Security Identifiers (SID) : 用来map到Security context的一个东西  

* Security Model : 是Type Enforcement (TE), Identity-Based Access Control (IBAC), RBAC, 与 optional MLS的结合，
其中重点内容是TE。

***
下面说的文件都在/sepolicy 下面

TE: 
定义domain对object的权限  

完整的格式如下  
> rule_name source_type target_type : class perm_set  
Type -> processes & object  

举个例子，在untrusted app 里面，我们可以在最后找到两行。  

> allow untrusted_app shell_data_file:file rw_file_perms;
  allow untrusted_app shell_data_file:dir r_dir_perms;  
  这两行允许 untrusted_app 进程，对shell_data_file 下面的文件进行读和写  
  以及读shell_data_file 目录的权限  

  其中untrusted_app的定义是非系统app  
  shell_data_file的定义可以在file_contexts文件夹里面找到  
> " /data/local/tmp(/.*)?	u:object_r:shell_data_file:s0  "  
  
  这行整体来看叫做security labeling,右边是security context，左边是这个context与什么文件路径链接起来。  
  然后详细说一下security context的构成部分。  
  最左边的u表示user,在users里面定义。  
  object_r是roles，在SEAndroid里面，一个user可以属于不同的roles。  
  最后的s0是security level,SEAndroid里面给系统资源都分配了一个security level,只有有对应的security level的进程才能访问这些资源。  

知道了这些内容，我们差不多就可以根据实际情况添加对应的security policy了。
比如untrusted_app是不能在自己的目录下面创造文件夹和文件的。
如果在untrusted.te加上这两行之后：  
>
Allow untrusted_app shell_data_file:file create_file_perms  
Allow untrusted_app shell_data_file:dir create_dir_perms  

untrusted_app就有了在/data/local/tmp里面创建文件和文件夹的权限。  











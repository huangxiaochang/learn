# kityMinder脑图多人协同

## 1. 定义原子性操作
  ### 节点的原子性操作：
    - 1.新增
    - 2.删除
    - 3.属性修改：
      - 1.新增属性
      - 2.修改属性
      - 3.删除属性

  然后把xmind中的一些指令操作转换成原子性操作，如：
  更改移动某个节点到其他节点，则需要转成删除和新增两个原子操作

## 2.冲突处理
  ### 2.1 冲突提示，人工进行解决

## 3.版本管理(时序)
  ### 3.1 本地的版本
    客户端设置一个操作队列来记录本地的操作，当收到服务器版本更新时，进行本地版本的更新：
    - 1.当操作队列为空时，直接进行更新
    - 2.当操作队列不为空时。进行版本的合并和冲突的处理

    **客户端在提交之前，一定要保持到最新版本才能提交**
  ### 3.2 远程的版本
    服务器进行了加锁，同一时间只处理一个提交（对于客户端的提交，要进行版本控制，
    客户端的提交，一定要是基于最新的版本），
    处理完后，远程版本回增加，同时转发到各客户端。

## 4.客户端操作转换： 
  ### 节点新增：要把节点插入的位置转换为远程最新版本的位置，如本地操作池中的操作为在父节点的子节点中的第三个
            位置插入节点a,但是服务器最新的版本操作也是在第三个位置插入了b,那么本地应该转换为在第四个位置前
            插入节点a。如果本地增加的节点在远程的后面，则无需转换

  ### 删除转换：
      - 1. 如果本地和远程删除的是同一个节点，则忽略本地的删除操作，因为此节点已经删除
      - 2.如果不是同一个节点，要进行本地删除节点的转换，如远程已经删除了第二个节点，而本地要删除第三个节点，
      则需要把本地要删除的节点转成删除第二个节点，如果远程删除的节点在本地要删除的节点的后面，则不需转换
      - 3.如果远程删除了某个节点，则需要把本地转换忽略该节点的子节点的所有操作
      - 4.删除，修改同一节点： 
        1.远程删除，本地修改，本地的修改操作需要忽略
        2.远程修改，本地删除，保持本地删除操作，无需转换
      
  

  ### 属性修改(只存在同一节点)： 
    - 1.属性的新增：
      - 1.当发生冲突时，即增加的是同一个属性时：由客户端操作人员进行冲突的解决后再进行操作的转换：
        1.1 如果选择只应用远程的属性，则忽略本地的新增操作
        1.2 如果选择应用本地的新增属性或者合并属性值。则需要把本地的新增操作转换成修改操作。
      - 2.没有冲突时，不需本地操作转换
    
    - 2.属性的修改：
      - 1. 如果更改的是同一个属性（如进行节点标题的修改），进行冲突处理：
        1.如果只保留远程的修改，则需转换忽略本地的修改操作
        2.如果合并或应用本地的修改操作，则保留本地的修改操作，无需转换
      - 2.不同属性，不需本地转换
    
    - 3.属性的删除： 
      - 1.如果删除的是同一个属性，则需转换操作忽略本地删除
      - 2.如果删除的不是同一属性，则不需转换
    
    - 4.删除-修改同一属性： 
      - 1.远程删除，本地修改，则进行冲突处理：
        1.1 如果选择应用本地修改，则需要转换本地修改成新增属性
        1.2 如果选择不应用本地修改，则需要忽略本地的修改操作
      - 2.远程修改， 本地删除，不需进行转换
    

  **在进行转换时，需要注意删除-修改(同一节点/属性)的情况**

## 5.其他
  - 1. 服务器的作用只是进行版本的控制和操作的转发，不进行操作的转换，操作的转换交由客户端实现
  - 2. 对于时序先后的问题，通过服务器加锁来解决，对于离线和版本的问题，通过客户端转换操作来解决



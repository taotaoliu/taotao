---
title: SVN-常用命令
date: 2018-02-01 18:13:48
tags:
---
1. 检出: checkout 缩写 co
如果不带 --password  参数传输密码的话，会提示输入密码，建议不要用明文的 --password 选项。其中 username 与 password前是两个短线，不是一个。默认check到当前目录。

    ```
    svn  checkout  http://路径(目录或文件的全路径)　[本地目录全路径] --username　用户名
    svn  checkout  svn://路径(目录或文件的全路径)　 [本地目录全路径] --username　用户名

    //例子：
    svn co svn://localhost/files          --username taotao
    svn checkout http://localhost/files   --username taotao
    ```

2. 导出(导出一个干净的不带.svn文件夹的目录树)

    ```
    svn  export  [-r 版本号]  http: //路径(目录或文件的全路径) --username　用户名
    svn  export  [-r 版本号]  svn:  //路径(目录或文件的全路径) --username　用户名
    svn  export  本地检出的(即带有.svn文件夹的)目录全路径  要导出的本地目录全路径

    //例子：
    svn export svn://localhost/files --username wzhnsc
    svn export svn://localhost/files --username wzhnsc
    svn export /home/files /home/tofiles
    ```

3. 添加新文件: add

    ```
    svn　add　文件名

    //例子：
    svn add picture.png
    ```

4. 提交: commit 缩写 ci

    ```
    svn　commit　-m　“提交备注信息文本“　[-N]　[--no-unlock]　文件名
    svn　ci　-m　“提交备注信息文本“　    [-N]　[--no-unlock]　文件名

    //例子：
    svn commit -m “提交我的测试用test.swift“ test.swift
    svn ci -m “提交我的测试用test.swift“ test.swift
    ```

5. 更新文件: update

    ```
    svn　update
    svn　update　-r　修正版本　文件名
    svn　update　文件名

    //例子：
    svn update      

    //后面没有目录，默认将当前目录以及子目录下的所有文件都更新到最新版本

    svn update -r 200 test.swift    

    //将版本库中的文件 test.swift还原到修正版本（revision）200

    //svn update test.swift

    //更新与版本库同步。提交的时候提示过期冲突，需要先update修改文件，然后清除
    svn resolved，最后再提交commit。
    ```

6. 删除文件: delete

    ```
    svn　delete　svn://路径(目录或文件的全路径) -m “删除备注信息文本”
    svn　delete　文件名 

    //例子：
    svn delete svn://localhost/testapp/test.swift -m “删除测试文件test.swift”

    //如下：
    svn delete test.swift 
    ```

7. 加锁: lock

    ```
    svn　lock　-m　“加锁备注信息文本“　[--force]　文件名 
    svn　unlock　文件名

    //例子：
    svn lock -m “锁信测试用test.swift文件“ test.swift 
    svn unlock test.swift
    ```

8. 比较差异: diff

    ```
    svn　diff　     文件名 
    svn　diff　-r　 修正版本号m:修正版本号n　文件名

    //例子：
    svn diff test.swift               

    //将修改的文件与基础版本比较

    svn diff -r 200:201 test.swift   

    //对修正版本号200和修正版本号201比较差异
    ```

9. 查看文件或者目录状态: status  缩写 st

    ```
    svn st 目录路径/名

    //目录下的文件和子目录的状态，正常状态不显示 //?：不在svn的控制中M：内容被修改；C：发生冲突A：预定加入到版本库 K：被锁定 

    svn  -v 目录路径/名

    svn status -v 目录路径/名

    //显示文件和子目录状态,第一列保持相同，第二列显示工作版本号，第三和第四列显示最后一次
    //修改的版本号和修改人
    ```

10. 查看日志: log

    ```
    svn　log　文件名

    //例子：
    svn log test.swift    

    //显示这个文件的所有修改记录，及其版本号的变化
    ```

11. 查看文件详细信息: info

    ```
    svn　info　文件名

    //例子：
    svn info test.swift
    ```

12. 查看版本库下的文件和目录列表: list 缩写 ls

    ```
    svn　list　svn://路径(目录或文件的全路径)
    svn　ls　svn://路径(目录或文件的全路径)

    //例子：
    svn list svn://localhost/test
    svn ls svn://localhost/test   

    //显示svn://localhost/test目录下的所有属于版本库的文件和目录 
    ```

13. 恢复本地修改: revert

    ```
    svn　revert　[--recursive]　文件名

    //注意: 本子命令不会存取网络，并且会解除冲突的状况。但是它不会恢复被删除的目录。

    //例子：
    svn revert taotao.swift      

    //丢弃对一个文件的修改

    svn revert --recursive .     

    //恢复一整个目录的文件，. 为当前目录 
    ```

14. 把工作拷贝更新到别的URL: switch

    ```
    svn　switch　http://目录全路径　  

    //本地目录全路径

    //例子：
    svn switch http://localhost/test/456 .  

    //(原为123的分支)当前所在目录分支到
    ```

15. 新建分支: copy
    ```
    svn copy

    //例子：
    svn copy branchA branchB  -m "make B branch"
    //从branchA拷贝出一个新分支branchB
    ```

16. 合并分支: merge
    ```

    //例子：
    svn merge branchA branchB  

    //把对branchA的修改合并到分支branchB
    ```

17. 解决冲突：resolved

    ```
    svn　resolved　[本地目录全路径]

    //例子：
    $ svn update
    C foo.c
    Updated to revision 31.
    //如果你在更新时得到冲突，你的工作拷贝会产生三个新的文件：

    $ ls
    foo.c
    foo.c.mine
    foo.c.r30
    foo.c.r31
    //当你解决了foo.c的冲突，并且准备提交，
    //运行svn resolved让你的工作拷贝知道你已经完成了所有事情。
    //你可以仅仅删除冲突的文件并且提交，
    //但是svn resolved除了删除冲突文件，还修正了一些记录在工作拷贝管理区域的记录数据。
    ```

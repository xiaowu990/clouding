# 1.4.1基本介绍


# 1.4.2三种模式
![[Pasted image 20230719104554.png]]
## Normal Mode
* 打开文件后进入的就是Normal Mode
* 此状态下，敲击键盘的动作被视为命令，而不是输入字符
* i进入Insert Mode，：进入可视化模式
* 其他模式按ESC键代表退回到Normal Mode
### 快捷键
1. yy：拷贝当前行，粘贴到当前行的下一行。
2. nyy：拷贝当前n行（从当前行开始，包括当前行，向下得n行）
3. dd：删除当前行
4. ndd：删除当前行向下的n行(包括当前行)
5. 粘贴：p:在当前光标所在行的的下一行开始粘贴，shift+p：从当前光标所在行开始粘贴
6. 在文件中查找某个单词：命令行输入 /（查找内容），按n查找下一个。如/hello 再按回车
7. 撤销动作：按一个“u”
8. 直接到最首行：gg，直接到最底行：G
## Insert Mode
![[Pasted image 20230719102502.png]]

## Visual Mode
* 在这个模式中，可以利用指令完成读取，存盘，替换，离开，显示行号的等动作。
- wq保存退出
- q无修改直接退出
- q!不保存修改直接退出（如果有修改了，不想保存直接退出只能用q!）
- 设置行号：set nu，取消行号：set nonu
- 跳转到指定行：输入指定的行数，再按shift+g
更多命令请查看官方说明文档

# VIM配置
[vim的使用和配置_vim简单配置_@每天都要敲代码的博客-CSDN博客](https://blog.csdn.net/m0_61933976/article/details/124653524?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169278977516800226579107%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169278977516800226579107&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_click~default-4-124653524-null-null.142^v93^insert_down1&utm_term=vim%E9%85%8D%E7%BD%AE&spm=1018.2226.3001.4187)
![[Pasted image 20230823192741.png]]


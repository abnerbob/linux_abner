用到就记一笔，记录自己常用的
超全 wiki: https://www.cnblogs.com/ataoxz/p/18872084

三种模式：
命令模式
插入模式
命令行模式 (底层模式、末行模式)

进入命令模式 Esc 或 Ctrl+[

1.  文本检索
向后搜索  :/关键字   向前搜索 :?关键字
向后查找继续 N  向前查找 n
高亮:set hisearch 取消高亮 :nohisearch  (临时)  :set nohisearch (彻底)
忽略大小写 :set ignorecase 关闭忽略大小写:set noignorecase
设置行号  :set nu 关闭行号:set nonu

2. 文本编辑
撤回上一次修改 u
删除行 dd
删除当前行开始的n行 ndd

3. 光标移动
移动到文件头:gg
移动到文件尾:G
移动到n行:ng (:n)

Title       : The document of shell project
Author      : Mergo 
Logo: True
Package: [fontset=windowsnew]ctex

[TITLE]
# 大体思路：
    1.最外层是一个循环
    
    
```cpp
//cmd用于存储解析后的单个命令；
//cmdline用于存放从屏幕中读入后的整个命令；
//MaxLineLength cmdline 的最长长度
//MaxArgc 一个命令的最多参数的个数
//CmdPtr 用于指向当下cmdline 的解析位置
//
int MaxCmdC = 10;
int MaxLineLength = 1024;
int MaxArgc = 20;
struct cmd{
  char* arg[MaxArgc];//一个命令之后跟的参数
  int infd;//这里是输入输出的文件描述符
  int outfd;
}cmds[MaxCmdC];
char* cmdline[MaxLLineLength];//用于存放从屏幕中读来的cmdline
char* cmdptr;
int CmdCout;
 
int main(){
  init();//初始化各种量，包括信号，cmdline(用于存放命令）
  while(1){
    print_promt();//打印提示符
    get_cmdline(); //读取屏幕上的字符，放到char* cmdline[MaxLineLegth]中
    //这里缺少一个空回车的情况
    if(!builtin()){
      if(!parse_cmdline()) 
        printf("Command not found!");
      else
        do_exeve();
      //这里缺少执行错误退出的情况
    }
  }
}
//返回值未知
//参数未知
/* 功能：
 * 先fork()，父进程、子进程
 *如果是&，父进程不需要等待子进程结束；如果不是，需要等待
 *如果是<,将确定inputfile name
 *如果是>,>>; 确定 outfile name;
do_exeve(){
  
  
  
  
  
  
  
  
  
}
//返回值：如果成功，返回1；不成功，返回0；
//参数：未知
int builtin(){
  


}
```





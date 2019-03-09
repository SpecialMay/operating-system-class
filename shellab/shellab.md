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
//CmdCount 指cmd的数量
//append 指是否存在>> 追加
//MaxCmdCount 最多的cmd 的个数
int MaxCmdCount= 10;
int MaxLineLength = 1024;
int MaxArgc = 20;
struct cmd{
  char* arg[MaxArgc];//一个命令之后跟的参数
  int infd;//这里是输入输出的文件描述符
  int outfd;
}cmds[MaxCmdCount];
char* cmdline[MaxLLineLength];//用于存放从屏幕中读来的cmdline
char* cmdptr;
int CmdCount;
int append;
int main(){
  init();//初始化各种量，包括信号，cmdline(用于存放命令）
  while(1){
    print_promt();//打印提示符
    get_cmdline(); //读取屏幕上的字符，放到char* cmdline[MaxLineLegth]中
    //这里缺少一个空回车的情况
    if(!builtin()){
      parse_cmdline();
      else
        do_exeve();
      //这里缺少执行错误退出的情况
    }
  }
}
//返回值:未知
//参数: 对cmdline 进行操作，将结果存到cmds[]中
/* 功能：
 * 首先check
 *如果是&，父进程不需要等待子进程结束；如果不是，需要等待
 *如果是<,将确定inputfile name
 *如果是>,>>; 确定 outfile name;  如果是>> 需要将append 变成1
 *如果有管道|，就按照每个管道分开各个命令
 *用一个for循环check 管道|，找出有多少个命令；CmdCount++；
 *将每个命令的第一个元素放入cmds[i]中，随后将后面的元素放入cmds[i].args[]中
 *将每个cmd 的具体的值也要放入cmd[i]中
 */
 char* getname(){
   /* 前提是check(sign) 检查到sign，并将CmdPtr 移动到sign所在的位置
    * 返回值是sign 后面一个文件名
    * 指针CmdPtr从sign的后一个元素开始
    */
    char* name[1024];
    while(*CmdPtr==' '||*CmdPtr=="\t") CmdPtr++;
    while(*CmdPtr!=' '
      &&*CmdPtr!='\t'
      &&*CmdPtr!='\n'
      &&*CmdPtr!='|'
      &&*CmdPtr!='>'
      &&*CmdPtr!='<') {
        *name = *CmdPtr;
         name++;CmdPtr++;
     }
   CmdPtr = cmdline;
   return name;
 }
 int check(char sign){
   //从当前的CmdPtr 开始遇到的第一个sign
   //如果有，则将指针移动到sign 之后的那个位置
   //如果没有，CmdPtr 保持当前位置不变
   while(*CmdPtr!=sign&&*CmdPtr!='\0'&&CmdPtr!='\n'){
     CmdPtr++;
   }
   if(*CmdPtr==sign){
     CmdPtr++;
     return 1;
   }
   CmdPtr = cmdline;
   return 0;
 }
 void getcommand(int i){
   /*目的：将> >> < | '\0' '\n' 之前的那些args放进对应的cmd[i]的arg[j]中*/
   /*规定：开始时，CmdPtr指针放在cmd的第一个；最后，指针放在最后的> >> < | \0 \n 处*/
   /*分割过程：
    * 外循环遇见上述任何一个符号，循环终止；内循环： 遇见'-'时,代表一个新的arg; 遇见空格，代表一个arg结束
    * 将每个arg的第一个元素的指针放进arg[j]中,作为一个字符串
    */
   int j = 0;
   while(*CmdPtr!='>'
        &&*CmdPtr!='<'
        &&*CmdPtr!='\0'
        &&*CmdPtr!='|'
        &&*CmdPtr!='\n'){
          if(*CmdPtr!=' '&&*CmdPtr!='\t'){
            cmd[i].arg[j]=ComPtr;
            j++;
            while(*ComPtr!=' '||*ComPtr!='\t')
               ComPtr++;
            *ComPtr='\0'
          }
          ComPtr++;
        }
   
   
 }

void pase_cmdline(){
  if(check('<')) inputfile = getname('<');
  if(check('>')) {/*这里注意可能会有两个>,即>> 的情况*/
    if(*CmdPtr=='>') CmdPtr++;
    outputfile = getname('>');
  }
  if(check('&')) append = 1;
  getcommand(0);//这时候CmdPtr指针移到第一个"|"或者移到最后的'\0'
  CmdCount=1;
  int i =0;
  for(i=1;i<MaxCmdCount;i++){
    if(*CmdPtr=='\0'||*CmdPtr=="\n") break;
    CmdPtr++;//这时候CmdPtr在第i-1个管道|的后一位；
    while(*CmdPtr==' '||*CmdPtr=='\t')/*这里一个是tab键和一个空格;目的是将指针移到第一个空格后*/
      CmdPtr++;
    if(!check('|')) break; //这时候CmdPtr指针移到第i个"|"或者移到最后的'\0'
    CmdPtr++;
    getcommand(i);
    CmdCount++;
  }
  /*实现管道命令的标准输入输出的转化
   *pipe 的实现：如果不是最后一条命令，那么要新建pipe ，让下一个命令的infd 指向pipe的写端，这个命令的outfd 指向pipe的读端*/
  for(i=0;i<CmdCount;i++){
    
    
    
  }
}
/*在执行某个命令时，遍历整个结构体，’
 *先fork()，父进程、子进程
 *<写的实现：如果有，那么将第一个命令的infd 指向 inputfile
 *>的实现，如果有，将最后一个命令的outfd指向outfile
 *>>在前面的时候，就让append=1；
 *接下来是各种命令的实现：vi ls grep  
 * vi 打开文件，如果没有创建？ 先实现创建文件、打开文件、保存退出、不保存退出、esc 键终止输入
 *grep 用于查找文件中符合条件的字符串
 *ls 
 * 上面三个命令不用细分，只需要执行execvp 就行了
 *backgroud 的情况:
 *首先是知识：前台进程和后台进程的区别：
 *前台作业能够接受SIGINT、SIGQUIT信号；
 *如果是后台进程，接收到SIGINT 执行sigint_handler ;其中SIGINT 是interrupt from keyboard,CTRL+C 可以发出SIGINT
 *并且将这个进程的pid 加入到Pid的组里（链表）
 */
do_exeve(){
  if(fork()==0){
    if
    exe
    
    
  }
  else{
    
    
    
  }
  
  
  
  
  
  
  
  
}
//返回值：如果成功，返回1；不成功，返回0；
//参数：未知
/*功能：history cd exit
 *判断cmdline 的第一个string 是不是内置命令列表里的，如果不是，return 0；
 *如果是，通过列表找到那个对应的函数（这里可能需要一个二维的列表），从而执行对应的函数
 */
int builtin(){
  
}
```





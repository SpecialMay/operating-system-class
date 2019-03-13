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
//backgound 代表是否在后台运行
int backgound;
int MaxCmdCount= 10;
int MaxLineLength = 1024;
int MaxArgc = 20;
int MaxHistory = 1024;
struct CMD{
  char* arg[MaxArgc];//一个命令之后跟的参数
  int infd;//这里是输入输出的文件描述符
  int outfd;
}cmd[MaxCmdCount];
char cmdline[MaxLineLength];//用于存放从屏幕中读来的cmdline
char* CmdPtr = NULL;
int CmdCount = 0 ;
int append;
int MAXNAME = 1024;
char inputfile[MAXNAME+1];
char outputfile[MAXNAME+1];
int lastpid;
struct Node{
  pid_t npd;
  char *backcn[20];//这个是什么东西？？？
  Node* next=NULL;
}NODE;
char* History[MaxHistory];/*history*/
int current_history =0;
/*为什么要setup_signal:  由于在自己的shell里面，CTRL+C  CTRL+/不应该使自己的进程终止 ，所以要频闭掉
*但是在background = 0 的进程中，要将这个恢复，从而可以中断。。。子进程中断的具体过程是什么？？？？？？
*sigint_handler 到底是怎么执行的？？？？？？？？？
*c 语言中字符串要怎么存？？？？？？
*background 的情况到底有哪些不同？将pid 加入 pid链表
*pid链表到底是怎么回事 ？？？？？什么时候能够用得到？？？？
*内置命令的研究写法？？？
*各种异常退出情况的研究？？？？？
*/

int main(){
  setup_signal();/*由于在自己的shell里面，CTRL+C  CTRL+/不应该使自己的进程终止 ，所以要频闭掉*/
  while(1){
    init();//初始化各种量，包括信号，cmdline(用于存放命令）    
    print_promt();//打印提示符
   if(get_cmdline()==-1); //读取屏幕上的字符，放到char* cmdline[MaxLineLegth]中
      continue;
    //这里缺少一个空回车的情况： 上述的if 已经处理
    CmdPtr = cmdline;
    if(!builtin()){
      parse_cmdline();
      do_exeve();
      //这里缺少执行错误退出的情况： 在do_execel中已经处理
    }
  }
}
void setup_signal(){
  signal(SIGINT,sigint_handler);
  signal(SIGKILL,SIG_IGN);
}
void sigint_handler(int sig){
  printf("\nmyshell$");
}
void init(){
  //未完待续：各种初始化
  CmdCount = 0;
  append =0;
  CmdPtr = cmdline:
  background = 0;
  /*这种全部化为0 有用吗？？？*/
  memset(cmdline,0,sizeof(cmdline));
  memset(inputfile,0,sizeof(inputfile));
  memset(outputfile,0,sizeof(outputfile));
  memset(cmd,0,sizeof(cmd));//这个初始化是来干嘛的
  int i;
  for (i=0; i<MaxCmdCount+1; i++)
	{
		cmd[i].infd = 0;
		cmd[i].outfd = 1;
	}
}
void pring_promt(){
  printf("myshell$")
}
int get_cmdline(){
  if(fgets(cmdline,MaxLineLength,stdin)=NULL)
    return -1;
   return 0;
}
//返回值:文件名
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
    append=0;
    if(*CmdPtr=='>') {
      CmdPtr++;
      append=1;
     }
    outputfile = getname('>');
  }
  background = 0;
  if(check('&')) background = 1;
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
  /*实现infd 和outfd 的描述符设定:
   *用一个for循环创建CmdCount-1 个管道，然后将第i 个cmd 的outfd 指向 第i个管道的写端fd[1];将第i+1个cmd 的infd 指向第i个管道的fd[0]
   *每次循环创建一个管道，共创建CmdCount-1个管道；处理outfd
   *第一次，没有写；最后一次，没有读
   */
   i=0;
   int fd[2];
   if(inputfile!='\0') cmd[0].infd = open(inputfile,O_RDONLY);
   if(outputfile!='\0'){
     if(append) 
      cmd[CmdCount-1].outfd = open(outfile,OWRONLY|OCREAT|O_APPEND,0666);
     else
      cmd[CmdCount-1].outfd = open(outfile,OWRONLY|OCREAT|OTRUNC,0666);
   }
   for(i=0;i<CmdCount-1;i++){
      pipe(&fd);
      cmd[i].outfd = fd[1];
      if(i<CmdCount-2)
        cmd[i+1].infd = fd[0]; 
   }
}
/*返回值：可以指明运行是否成功
 *输入值：未知
 *在执行某个命令时，遍历整个结构体，’
 *先fork()，父进程、子进程
 *<写的实现：如果有，那么将第一个命令的cmd[CmdCount-1].infd->inputfile 指向 inputfile
 *>的实现，如果有，将最后一个命令的outfd指向outfile
 *>>如果有，将最后一个cmd[CmdCount-1].outfd->outfile 的末尾
 *pipe 的实现：在第一个进程开始之前，创建第一个pipe，在第一个进程把它设置为标准输出，在下一个进程把它设置标准输出
 *1.首先创建一个管道；
 *2.forK（） 一个子进程（子进程执行前面一个命令）cmd[i-1]；父进程执行cmd[i]
 *3.
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
void do_exeve(){
 int i = 0;
 pid_t pid;
 if（CmdCount==0) return;
 for(i=0;i<CmdCount;i++){
   if(pid=fork()==0){
     if(cmd[i].indf!=0){/*当它不是屏幕输入时*/
       close(0);
       dup(cmd[i].infd);
     }
     if(cmd[i].outfd!=1) {/*当它不是屏幕输出时*/
       close(1);
       dup(cmd[i].outfd);
     }
     if(cmd[i].infd==0&&background==1)
        cmd[i].infd = open("/dev/null",O_RDONLY);
        /*后台程序的输出默认指向/dev/null文件，/dev/null是shell的一个特殊文件，任何输入这个文件的东西将会被丢弃*/ 
     if(background==0){
       signal(SIGINT,SIG_DFL);//恢复SIGINT的功能，该进程可以通过ctrl+C 终止
       signal(SIGQUIT,SIG_DFL);//恢复SIGQUIT的功能，该进程可以通过ctrl+\终止
     }  
     execvp(cmd[i].arg[0],cmd[i].arg);
     perror("exe_fail");     
     exit(EXIT_FAILURE);
   }
   else if(pid<=-1){
     /*错误退出*/
     perror("fork_fail");
     exit(EXIT_TAILURE);
   }
   else{/*父进程*/
      if(background ==1){
    /*添加进入jobs的链表*/
    //咋地？
      }
   }
 }
/*这里有个地方需要waitpid*/
}
//返回值：如果成功，返回1；不成功，返回0；
/*参数：直接使用cmdline 然后取第一个空格之前的元素，作为一个键值对在一个函数指针列表中查找相同的元素，
果找到一个相同的元素则执行那个函数，然后返回1，如果没有找到，那么就返回0*/
/*功能：history cd exit
 *判断cmdline 的第一个string 是不是内置命令列表里的，如果不是，return 0；
 *如果是，通过列表找到那个对应的函数（这里可能需要一个二维的列表），从而执行对应的函数
 */
 /*builtin 要实现history cd exit*/
 /*如果解析命令*/
 #include <string.h>

int builtin(){
  /*截取第一个空格前的元素，放入cmd中;第一个元素是命令，第二个元素是参数*/
  if(current_history<MaxHistory){
  History[current_history] = *cmdline;
  current_history++;
  }
  char* builtin_cmd[2];
  memset(builtin,0,sizeof(builtin_cmd));
  char* cmdptr=cmdline;
  while(*cmdptr==' '){
    cmdptr++;
  }
  builtin_cmd[0]=cmdptr;
  while(*cmdptr!='\n'){
    if(*cmdptr==' '){
      builtin_cmd[1]=cmdptr++;
      break;
    }
    cmdptr++;
  }
  void(*function[])(char*){history,exit,cd};
  int i;
  for(i=0;i<3;i++){
    if(strcmp(function[i],*builtin_cmd[0])==0){
       function[i](builtin_cmd[1]);
    } 
  }
}

void history(char* builtin_cmd[2]{
  int n = atoi(*builtin_cmd[1]);
  int i=0;
  int HisPritNum=0;
  if(n>current_history) {n = current_history;printf("%d","There are ",current_history,"records;")
  for(;i<n;i++){
    printf("%s",History[i]);
  }
}
void exit(char* builtin_cmd[2]){
  /*检查是否有background 的进程，如果有，那么就询问后再退出*/
  printf("%s","exit");
  exit(0);
}
void cd(char* builtin_cmd[2]){
  getcommond(0);
  int fd;
  fd = open(*(cmd[0].arg),O_RDONLY);
  fchdir(fd);
  close(fd);
}
```





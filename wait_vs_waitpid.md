# wait vs. waitpid

之前解了一個bug，很要命的bug，東看西看了老半天也沒有端倪，先放一個sample，大家可以參考一下：
```
switch (fork())
{
	case -1:
		break;
	case 0:
		execl("/util/AAA", "/util/AAA", "-s", "OOXX", NULL);
		exit(1);
	default:
		wait(&status);
		if(WIFEXITED(status)) 
		{
			ret = WEXITSTATUS(status);
		} 
		else if(WIFSIGNALED(status))
		{
			ret = WTERMSIG(status)
		}
		break;
}
```
非常常見的用法，幾乎只要fork和exec系列API都會這樣寫，實在也看不出甚麼問題，但偏偏，今天我們的系統hang死在wait上。
 
為什麼?!
 
這個例子看起來非常正常，沒甚麼問題阿，就child跑完然後parent回收結果，說真的，每個programmer身邊一定也很多這樣的code
 
好！
 
問題來了！
 
若是今天，child比parent還要早起來，會發生甚麼事？
 
答案是，child跑完了，然後parent才起來wait，那這parent是要等洨阿？就會hang死。
 
解法其實有很多，我們通常稱這種問題叫做timing issue，顧名思義，平常沒事，但若那個timing督到就有issue，而這種問題通常sleep就搞定了，如下
```
switch (fork())
{
	case -1:
		break;
	case 0:
		sleep(1);
		execl("/util/AAA", "/util/AAA", "-s", "OOXX", NULL);
		exit(1);
	default:
		wait(&status);
		if(WIFEXITED(status)) 
		{
			ret = WEXITSTATUS(status);
		} 
		else if(WIFSIGNALED(status))
		{
			ret = WTERMSIG(status)
		}
		break;
}
```
只要讓child睡一下，child就會從run queue被排到interruptable，這樣在run queue的parent就能先拿到time slot。但這種解法太智障了，其實還有另外一種做法，就是用waitpid替代wait
```
switch (cpid = fork())
{
	case -1:
		break;
	case 0:
		sleep(1);
		execl("/util/AAA", "/util/AAA", "-s", "OOXX", NULL);
		exit(1);
	default:
		waitpid(cpid, &status, WNOHANG);
		if(WIFEXITED(status)) 
		{
			ret = WEXITSTATUS(status);
		} 
		else if(WIFSIGNALED(status))
		{
			ret = WTERMSIG(status)
		}
		break;
}
```
waitpid一樣會有child first的問題，但用WNOHANG就可以避免hang死的問題，即使child process先執行完，pid消失waitpid就會return。記得caller必須要做好error handler，不然問題就會從原本的hang死變成un-expected。
 
P.S. 在kernel 2.6.33(?)的kernel config就有CHILD FIRST的開關，確切版本如果有熟悉新kernel 的再跟我說吧





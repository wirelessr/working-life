# C語言的單元測試

如果你非常想做單元測試，但你所處的環境實在架不起來單元測試的環境，那麼這篇非常適合你。我工作的環境是純C語言，而且是現成數百萬行的code，並且run在embaded上，這三個實在是對單元測試非常嚴苛的條件阿！

所幸我發現了一個好東西：
[CuTest](http://sourceforge.net/projects/cutest/?source=dlp)

一個很簡單就可以跑起來的C語言單元測試環境

下載解開後，把CuTest.c、CuTest.h、CuTestTest.c和make-tests.sh放到你想放的地方，然後寫一個簡單的sample：StrUtil.c (其實CuTestTest.c就已經是個範例了)
```
#include "CuTest.h"
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

char *StrToUpper(char *str)
{
	return str;
}

void TestStrToUpper(CuTest *tc)
{
	char *input = strdup("hello world");
	char *actual = StrToUpper(input);
	char *expected = "HELLO WORLD";
	CuAssertStrEquals(tc, expected, actual);
}

CuSuite *StrUtilGetSuite()
{
	CuSuite *suite = CuSuiteNew();
	SUITE_ADD_TEST(suite, TestStrToUpper);
	return suite;
}
```
接著就是執行 
> ./make-tests.sh > testScript.c

最後編譯 gcc testScript.c CuTest.c StrUtil.c CuTestTest.c，然後就可以很開心的單元測試一下了
> ./a.out

```
..................................F

There was 1 failure:
1) TestStrToUpper: StrUtil.c:14: expected "HELLO WORLD" but was "hello world"

!!!FAILURES!!!
Runs: 35 Passes: 34 Fails: 1
```


#关于zircon/kernel的include

1、#include<zircon/types.h> --> zircon/system/public/zircon/types.h

2、#include<fbl/xxx.h> --> zircon/kernel/lib/fbl/include/fbl/xxx.h

3、#include<ktl/xxx.h> --> zircon/kernel/lib/ktl/include/ktl/xxx.h

4、#include"object/xxx.h" --> zircon/kernel/object/include/object/xxx.h

5、include<pow2.h> --> zircon/kernel/include/pow2.h

6、#include<lib/counters.h> --> zircon/kernel/lib/counters/include/lib/counters.h

7、#include<object/dispatcher.h> --> zircon/kernel/object/include/object/dispatcher.h 应该写成#include "object/dispatcher.h"

8、/fuchsia/zircon/kernel/object/include/object/handle.h #include<stdint.h> --> libc++系统库，暂时不完全能确定，/fuchsia/zircon/kernel/lib/libc/include/stdint.h，但是参考2 和 3可以得出结论，如果引用的是后者，则应该写成#include<libc/stdint.h>

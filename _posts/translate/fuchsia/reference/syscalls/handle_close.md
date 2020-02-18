# zx_handle_close

## 含义

Close a handle.

## 概要

```c
#include <zircon/syscalls.h>
zx_status_t zx_handle_close(zx_handle_t handle);
```

## 描述

zx_handle_close()关闭一个句柄，当没有其它句柄指向其指向的对象时，这个对象就被回收。
如果被关闭的句柄正被用来等待zx_object_wait_one()或zx_object_wait_many()，那么这个
wait调用就会终止。关闭一个'ZX_HANDLE_INVALID'并不是一个错误，就像free(NULL)一样是
一个合法调用。

## 权限

None.

## 返回值

zx_handle_close()返回ZX_OK表示成功.

## 错误码

ZX_ERR_BAD_HANDLE不是一个合法的句柄。

## 参考

  * zx_handle_close_many()
  * zx_handle_duplicate()
  * zx_handle_replace()

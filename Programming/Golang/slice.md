# Slice

```go
package main

import (
    "fmt"
)

func main() {
    var s = []string{"1", "2", "3"}
    modifySlice(s)
    fmt.Println(s[0])
    fmt.Println(s)
}

func modifySlice(i []string) {
    i[0] = "3"
    i = append(i, "4")
    i[1] = "3"
}
```

Run result
```
3
[3 2 3]
```

解析：cap未变化时，slice是对数组的引用，并且append会修改被引用数组的值。append操作导致cap变化后，会复制被引用的数组，然后切断引用关系。

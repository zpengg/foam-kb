- ``` go
  func evacuate(t *maptype, h *hmap, oldbucket uintptr) {
    // 定位老的 bucket 地址
    b := (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
    // 结果是 2^B，如 B = 5，结果为32
    newbit := h.noldbuckets()
    // key 的哈希函数
    alg := t.key.alg
    // 如果 b 没有被搬迁过
    if !evacuated(b) {
    var (
        // 表示bucket 移动的目标地址
        x, y   *bmap
        // 指向 x,y 中的 key/val
        xi, yi int
        // 指向 x，y 中的 key
        xk, yk unsafe.Pointer
        // 指向 x，y 中的 value
        xv, yv unsafe.Pointer
    )
    // 默认是等 size 扩容，前后 bucket 序号不变
    // 使用 x 来进行搬迁
    x = (*bmap)(add(h.buckets, oldbucket*uintptr(t.bucketsize)))
    xi = 0
    xk = add(unsafe.Pointer(x), dataOffset)
    xv = add(xk, bucketCnt*uintptr(t.keysize))、
    - // 如果不是等 size 扩容，前后 bucket 序号有变
    // 使用 y 来进行搬迁
    if !h.sameSizeGrow() {
        // y 代表的 bucket 序号增加了 2^B
        y = (*bmap)(add(h.buckets, (oldbucket+newbit)*uintptr(t.bucketsize)))
        yi = 0
        yk = add(unsafe.Pointer(y), dataOffset)
        yv = add(yk, bucketCnt*uintptr(t.keysize))
    }
    - // 遍历所有的 bucket，包括 overflow buckets
    // b 是老的 bucket 地址
    for ; b != nil; b = b.overflow(t) {
        k := add(unsafe.Pointer(b), dataOffset)
        v := add(k, bucketCnt*uintptr(t.keysize))
    - // 遍历 bucket 中的所有 cell
        for i := 0; i < bucketCnt; i, k, v = i+1, add(k, uintptr(t.keysize)), add(v, uintptr(t.valuesize)) {
            // 当前 cell 的 top hash 值
            top := b.tophash[i]
            // 如果 cell 为空，即没有 key
            if top == empty {
                // 那就标志它被"搬迁"过
                b.tophash[i] = evacuatedEmpty
                // 继续下个 cell
                continue
            }
            // 正常不会出现这种情况
            // 未被搬迁的 cell 只可能是 empty 或是
            // 正常的 top hash（大于 minTopHash）
            if top < minTopHash {
                throw("bad map state")
            }
    - k2 := k
            // 如果 key 是指针，则解引用
            if t.indirectkey {
                k2 = *((*unsafe.Pointer)(k2))
            }
    - // 默认使用 X，等量扩容
            useX := true
            // 如果不是等量扩容
            if !h.sameSizeGrow() {
                // 计算 hash 值，和 key 第一次写入时一样
                hash := alg.hash(k2, uintptr(h.hash0))
    // 如果有协程正在遍历 map
                if h.flags&iterator != 0 {
                    // 如果出现 相同的 key 值，算出来的 hash 值不同
                    if !t.reflexivekey && !alg.equal(k2, k2) {
                        // 只有在 float 变量的 NaN() 情况下会出现
                        if top&1 != 0 {
                            // 第 B 位置 1
                            hash |= newbit
                        } else {
                            // 第 B 位置 0
                            hash &^= newbit
                        }
                        // 取高 8 位作为 top hash 值
                        top = uint8(hash >> (sys.PtrSize*8 - 8))
                        if top < minTopHash {
                            top += minTopHash
                        }
                    }
                }
    - // 取决于新哈希值的 oldB+1 位是 0 还是 1
                // 详细看后面的文章
                useX = hash&newbit == 0
            }
    - // 如果 key 搬到 X 部分
            if useX {
                // 标志老的 cell 的 top hash 值，表示搬移到 X 部分
                b.tophash[i] = evacuatedX
                // 如果 xi 等于 8，说明要溢出了
                if xi == bucketCnt {
                    // 新建一个 bucket
                    newx := h.newoverflow(t, x)
                    x = newx
                    // xi 从 0 开始计数
                    xi = 0
                    // xk 表示 key 要移动到的位置
                    xk = add(unsafe.Pointer(x), dataOffset)
                    // xv 表示 value 要移动到的位置
                    xv = add(xk, bucketCnt*uintptr(t.keysize))
                }
                // 设置 top hash 值
                x.tophash[xi] = top
                // key 是指针
                if t.indirectkey {
                    // 将原 key（是指针）复制到新位置
                    *(*unsafe.Pointer)(xk) = k2 // copy pointer
                } else {
                    // 将原 key（是值）复制到新位置
                    typedmemmove(t.key, xk, k) // copy value
                }
                // value 是指针，操作同 key
                if t.indirectvalue {
                    *(*unsafe.Pointer)(xv) = *(*unsafe.Pointer)(v)
                } else {
                    typedmemmove(t.elem, xv, v)
                }
    - // 定位到下一个 cell
                xi++
                xk = add(xk, uintptr(t.keysize))
                xv = add(xv, uintptr(t.valuesize))
            } else { // key 搬到 Y 部分，操作同 X 部分
                // ……
                // 省略了这部分，操作和 X 部分相同
            }
        }
    }
    // 如果没有协程在使用老的 buckets，就把老 buckets 清除掉，帮助gc
    if h.flags&oldIterator == 0 {
        b = (*bmap)(add(h.oldbuckets, oldbucket*uintptr(t.bucketsize)))
        // 只清除bucket 的 key,value 部分，保留 top hash 部分，指示搬迁状态
        if t.bucket.kind&kindNoPointers == 0 {
            memclrHasPointers(add(unsafe.Pointer(b), dataOffset), uintptr(t.bucketsize)-dataOffset)
        } else {
            memclrNoHeapPointers(add(unsafe.Pointer(b), dataOffset), uintptr(t.bucketsize)-dataOffset)
        }
    }
    }
    - // 更新搬迁进度
    // 如果此次搬迁的 bucket 等于当前进度
    if oldbucket == h.nevacuate {
    // 进度加 1
    h.nevacuate = oldbucket + 1
    // Experiments suggest that 1024 is overkill by at least an order of magnitude.
    // Put it in there as a safeguard anyway, to ensure O(1) behavior.
    // 尝试往后看 1024 个 bucket
    stop := h.nevacuate + 1024
    if stop > newbit {
        stop = newbit
    }
    // 寻找没有搬迁的 bucket
    for h.nevacuate != stop && bucketEvacuated(t, h, h.nevacuate) {
        h.nevacuate++
    }
  
    // 现在 h.nevacuate 之前的 bucket 都被搬迁完毕
  
    // 所有的 buckets 搬迁完毕
    if h.nevacuate == newbit {
        // 清除老的 buckets
        h.oldbuckets = nil
        // 清除老的 overflow bucket
        // 回忆一下：[0] 表示当前 overflow bucket
        // [1] 表示 old overflow bucket
        if h.extra != nil {
            h.extra.overflow[1] = nil
        }
        // 清除正在扩容的标志位
        h.flags &^= sameSizeGrow
    }
    }
  }
  ```
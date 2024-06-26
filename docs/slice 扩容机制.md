-
- 小容量会2倍扩，1.18 阈值是256，之前是1024
	- 在原来的slice容量`oldcap`小于1024的时候，新 slice 的容量`newcap`的确是`oldcap`的2倍。
	- 老版本
	- ```
	  newcap += newcap / 4
	  ```
	- 当原slice容量(oldcap)小于256的时候，新slice(newcap)容量为原来的2倍#go1.18
	- 原slice容量超过256，新slice容量 newcap = oldcap+(oldcap+3*256)/4
	- ```go # 1
	  // go 1.9.5 src/runtime/slice.go:82
	  func growslice(et *_type, old slice, cap int) slice {
	      // ……
	      newcap := old.cap
	  	doublecap := newcap + newcap
	  	if cap > doublecap {
	  		newcap = cap
	  	} else {
	  		if old.len < 1024 {
	  			newcap = doublecap
	  		} else {
	  			for newcap < cap {
	  				newcap += newcap / 4
	  			}
	  		}
	  	}
	  	// ……
	  	
	  	capmem = roundupsize(uintptr(newcap) * ptrSize)
	  	newcap = int(capmem / ptrSize)
	  }
	  ```
	- ```go
	  // go 1.18 src/runtime/slice.go:178
	  func growslice(et *_type, old slice, cap int) slice {
	      // ……
	      newcap := old.cap
	  	doublecap := newcap + newcap
	  	if cap > doublecap {
	  		newcap = cap
	  	} else {
	  		const threshold = 256
	  		if old.cap < threshold {
	  			newcap = doublecap
	  		} else {
	  			for 0 < newcap && newcap < cap {
	                  // Transition from growing 2x for small slices
	  				// to growing 1.25x for large slices. This formula
	  				// gives a smooth-ish transition between the two.
	  				newcap += (newcap + 3*threshold) / 4
	  			}
	  			if newcap <= 0 {
	  				newcap = cap
	  			}
	  		}
	  	}
	  	// ……
	      
	  	capmem = roundupsize(uintptr(newcap) * ptrSize)// 内存对其
	  	newcap = int(capmem / ptrSize)
	  }
	  ```
	-
- 内存对齐 roundupsize
	- 进行内存对齐之后，新 slice 的容量是要 `大于等于` 按照前半部分生成的`newcap`
	- 不一定发生在 len = cap 的时候，可能会提前
		- sizeclasses.go golang对象大小表 1 -32k
		- ```go
		  // class  bytes/obj  bytes/span  objects  tail waste  max waste
		  //     1          8        8192     1024           0     87.50%
		  //     2         16        8192      512           0     43.75%
		  //     3         32        8192      256           0     46.88%
		  //     4         48        8192      170          32     31.52%
		  //     5         64        8192      128           0     23.44%
		  //     6         80        8192      102          32     19.07%
		  //     7         96        8192       85          32     15.95%
		  //     8        112        8192       73          16     13.56%
		  //     9        128        8192       64           0     11.72%
		  //    10        144        8192       56         128     11.82%
		  
		  //    ...
		  //    65      28672       57344        2           0      4.91%
		  //    66      32768       32768        1           0     12.50%
		  
		  ```
		-
- 一次性添加多个的时候，若超过 double cap, 会根据 roundupsize 重新计算一个
-
- [[copy]]
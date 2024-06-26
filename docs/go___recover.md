- 也只是一个 `runtime.gorecover` 调用。
- ```
  TEXT main.main.func1(SB) /Users/changkun/dev/go-under-the-hood/demo/7-lang/panic/main.go
    (...)
    main.go:5		0x104e09d		488d442428		LEAQ 0x28(SP), AX			
    main.go:5		0x104e0a2		48890424		MOVQ AX, 0(SP)				
    main.go:5		0x104e0a6		e8153bfdff		CALL runtime.gorecover(SB)		
    (...)
  ```
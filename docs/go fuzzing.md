- link::[史上最强代码自测方法，没有之一！ | Go 技术论坛](https://learnku.com/articles/65809')
  tags:: #go1.18
-
- ![](https://cdn.learnku.com/uploads/images/202203/07/73865/kTT65wuElD.webp!large)
-
- ```go
  func FuzzSum(f *testing.F) {
    rand.Seed(time.Now().UnixNano())
  
    f.Add(10)
    f.Fuzz(func(t *testing.T, n int) {
      n %= 20
      var vals []int64
      var expect int64
      for i := 0; i < n; i++ {
        val := rand.Int63() % 1e6
        vals = append(vals, val)
        expect += val
      }
  
      assert.Equal(t, expect, Sum(vals))
    })
  }
  ```
- Fuzz 传入的基本类型
- 但 里面 `func(t *testing.T, n int)`, 可以继续构造参数和调用其他方法
-
-
- ```go
  // 安装 mockgen
  go install github.com/golang/mock/mockgen@latest
  
  mockgen -source=foo.go -package=foo > mock_gen.go
  
  ```
-
- ```go
  func TestGetFromDB(t *testing.T) {
    ctrl := gomock.NewController(t)
    defer ctrl.Finish() // 断言 DB.Get() 方法是否被调用
    - m := NewMockDB(ctrl)
    m.EXPECT().Get(gomock.Eq("Tom")).Return(100, errors.New("not exist"))// stub
    - if v := GetFromDB(m, "Tom"); v != -1 {
    t.Fatal("expected -1, but got", v)
    }
  }
  ```
-
- 常见打桩
	- 返回值
	- 调用次数
	- 调用顺序
-
- mock 作用的是接口，因此将依赖抽象为接口，而不是直接依赖具体的类。
- 不直接依赖的实例，而是使用[[依赖注入]]降低耦合性。
-
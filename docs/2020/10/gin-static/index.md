# 一拳搞定 Gin | 前后端分离下静态资源部署到后端访问

本文介绍通过 Gin 框架访问后端的静态资源

<!--more-->

## 一、前言

最近给公司做运维平台，前端是基于 Ant Design Pro 构建的 React 项目，后端是基于 Gin 搭建的 Go Web 项目。在部署的时候考虑到方便性，选择了把构建好的前端资源放到后端服务器，通过启动后端服务器直接运行前后端程序。

## 二、实战分析

### 2.1 静态资源

代码：

```go
func main() {
	r := gin.Default()

	dir, _ := os.Getwd()
	// static
	r.Static("/assets", dir+"/src/gin/assets")
	r.StaticFS("/more_static", http.Dir(dir+"/src/gin/assets"))
	r.StaticFile("/favicon.ico", dir+"/src/gin/resources/favicon.ico")

	// Listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

访问如下地址：

```
http://127.0.0.1:8080/more_static/favicon.ico
http://127.0.0.1:8080/assets/favicon.ico
http://127.0.0.1:8080/favicon.ico
```

均可以请求到头像图片。

#### 静态文件

> 访问某个路径，直接找到对应的资源返回。

使用：

```go
  r.StaticFile("/favicon.ico", dir+"/src/gin/resources/favicon.ico")
```

源码分析：

```go
// File writes the specified file into the body stream in a efficient way.
func (c *Context) File(filepath string) {
	http.ServeFile(c.Writer, c.Request, filepath)
}
```

- Gin 接收到的 HTTP 请求会到这里，然后交给 http 文件服务。

> /net/http/fs.go:674

```go
func ServeFile(w ResponseWriter, r *Request, name string) {
	if containsDotDot(r.URL.Path) {
		// Too many programs use r.URL.Path to construct the argument to
		// serveFile. Reject the request under the assumption that happened
		// here and ".." may not be wanted.
		// Note that name might not contain "..", for example if code (still
		// incorrectly) used filepath.Join(myDir, r.URL.Path).
		Error(w, "invalid URL path", StatusBadRequest)
		return
	}
	dir, file := filepath.Split(name)
	serveFile(w, r, Dir(dir), file, false)
}
```

- 分隔名称，会分离出资源的路径和名称
- 然后调用寻找文件的逻辑

> /net/http/fs.go:546

```go
// name is '/'-separated, not filepath.Separator.
func serveFile(w ResponseWriter, r *Request, fs FileSystem, name string, redirect bool) {
	const indexPage = "/index.html"

	// redirect .../index.html to .../
	// can't use Redirect() because that would make the path absolute,
	// which would be a problem running under StripPrefix
	if strings.HasSuffix(r.URL.Path, indexPage) {
		localRedirect(w, r, "./")
		return
	}

	f, err := fs.Open(name)
	if err != nil {
		msg, code := toHTTPError(err)
		Error(w, msg, code)
		return
	}
	defer f.Close()

	d, err := f.Stat()
	if err != nil {
		msg, code := toHTTPError(err)
		Error(w, msg, code)
		return
	}

  ...
}
```

- 用的 net 包的 filesystem 打开文件，如果不存在则包 HTTP 的 404 错误
- 如果是文件夹会找 index.html 文件，本文不扩展

```go
func (d Dir) Open(name string) (File, error) {
	if filepath.Separator != '/' && strings.ContainsRune(name, filepath.Separator) {
		return nil, errors.New("http: invalid character in file path")
	}
	dir := string(d)
	if dir == "" {
		dir = "."
	}
	fullName := filepath.Join(dir, filepath.FromSlash(path.Clean("/"+name)))
	f, err := os.Open(fullName)
	if err != nil {
		return nil, mapDirOpenError(err, fullName)
	}
	return f, nil
}
```

- 拼接完整的路径
- 使用标准的 `os.Open()` 打开

#### 静态目录

> 路径加上某个前缀对应访问某个目录下的资源。

使用：

```go
	r.StaticFS("/more_static", http.Dir(dir+"/src/gin/assets"))
```

源码分析：

```go
func (group *RouterGroup) Static(relativePath, root string) IRoutes {
	return group.StaticFS(relativePath, Dir(root, false))
}
```

> /github.com/gin-gonic/gin/fs.go:24

```go
func Dir(root string, listDirectory bool) http.FileSystem {
	fs := http.Dir(root)
	if listDirectory {
		return fs
	}
	return &onlyfilesFS{fs}
}
```

- 这里主要是能够支持是否列出目录下的内容（HTTP 默认文件的实现）
- 静态目录是关闭的，文件服务器也可以配置一样的效果

```go
func (f neuteredReaddirFile) Readdir(count int) ([]os.FileInfo, error) {
	// this disables directory listing
	return nil, nil
}
```

- 实现反馈 nil

#### 文件服务器

> 能够通过 HTTP 请求访问文件，类似于 FTP 服务器。

使用：

```go
	r.StaticFS("/more_static", http.Dir(dir+"/src/gin/assets"))
```

效果如下：

{{< figure src="/2020/10/gin-static/fs.jpg" title="文件服务器" >}}

源码分析：

```go
// StaticFS works just like `Static()` but a custom `http.FileSystem` can be used instead.
// Gin by default user: gin.Dir()
func (group *RouterGroup) StaticFS(relativePath string, fs http.FileSystem) IRoutes {
	if strings.Contains(relativePath, ":") || strings.Contains(relativePath, "*") {
		panic("URL parameters can not be used when serving a static folder")
	}
	handler := group.createStaticHandler(relativePath, fs)
	urlPattern := path.Join(relativePath, "/*filepath")

	// Register GET and HEAD handlers
	group.GET(urlPattern, handler)
	group.HEAD(urlPattern, handler)
	return group.returnObj()
}
```

> /github.com/gin-gonic/gin/routergroup.go:185

```go
func (group *RouterGroup) createStaticHandler(relativePath string, fs http.FileSystem) HandlerFunc {
	absolutePath := group.calculateAbsolutePath(relativePath)
	fileServer := http.StripPrefix(absolutePath, http.FileServer(fs))

	return func(c *Context) {
		if _, nolisting := fs.(*onlyfilesFS); nolisting {
			c.Writer.WriteHeader(http.StatusNotFound)
		}

		file := c.Param("filepath")
		// Check if file exists and/or if we have permission to access it
		f, err := fs.Open(file)
		if err != nil {
			c.Writer.WriteHeader(http.StatusNotFound)
			c.handlers = group.engine.noRoute
			// Reset index
			c.index = -1
			return
		}
		f.Close()

		fileServer.ServeHTTP(c.Writer, c.Request)
	}
}
```

> /net/http/server.go:2040

```go
func StripPrefix(prefix string, h Handler) Handler {
	if prefix == "" {
		return h
	}
	return HandlerFunc(func(w ResponseWriter, r *Request) {
		if p := strings.TrimPrefix(r.URL.Path, prefix); len(p) < len(r.URL.Path) {
			r2 := new(Request)
			*r2 = *r
			r2.URL = new(url.URL)
			*r2.URL = *r.URL
			r2.URL.Path = p
			h.ServeHTTP(w, r2)
		} else {
			NotFound(w, r)
		}
	})
}
```

- 去除下前缀的空格
- copy 了 request 去执行接下去的任务

```go
func FileServer(root FileSystem) Handler {
	return &fileHandler{root}
}

func (f *fileHandler) ServeHTTP(w ResponseWriter, r *Request) {
	upath := r.URL.Path
	if !strings.HasPrefix(upath, "/") {
		upath = "/" + upath
		r.URL.Path = upath
	}
	serveFile(w, r, f.root, path.Clean(upath), true)
}
```

- 前缀 URL 加上 /
- 文件服务逻辑，和静态文件

### 单页路径处理

上面讲的都是 gin 本身基础内容，下面说下项目中的实战。

#### 静态文件位置

项目文件夹如下：

{{< figure src="/2020/10/gin-static/package.jpg" title="目录" >}}

- dist 前端 `npm intall` 打出来的包
- pkg 后端代码

{{< figure src="/2020/10/gin-static/dist.jpg" title="dist目录" >}}

- index.html 单页应用，只有一个 HTML，其它都是 .js 渲染出来的

#### 项目配置

通过下面代码加到 gin 里面。

```go
root := path.Join(dir, "dist")

r.Use(static.NewFileSystem(root, "/", constant.ApiVersion, false).HandlerFunc())
```

参数说明：
- root 文件服务的根目录
- `"/"` 规则，既匹配全部
- 不包含的路径，指后端接口路径，比如 `/api/v1`
- false 上面提到默认的实现有体现，ture 会罗列文件夹下的文件列表，用作 FTP 服务器

#### 原理说明

### 前端页面

官方的示例

```go
func main() {
	r := gin.Default()

	// Serves unicode entities
	r.GET("/json", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"html": "<b>Hello, world!</b>",
		})
	})

	// Serves literal characters
	r.GET("/purejson", func(c *gin.Context) {
		c.PureJSON(200, gin.H{
			"html": "<b>Hello, world!</b>",
		})
	})

	// listen and serve on 0.0.0.0:8080
	r.Run(":8080")
}
```

上面的两个请求返回效果如图：

{{< figure src="/2020/10/gin-static/json.jpg" title="json" >}}

{{< figure src="/2020/10/gin-static/pure.jpg" title="pure" >}}


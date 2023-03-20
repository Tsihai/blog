---
title: Spring Boot+FastDFS 前后端分离文件上传
---
在Spring Boot前后端分离环境下做文件上传, 在生产环境中, 我们可以搭建独立的文件服务器, 结合FastDFS还可以搭建独立的分布式文件服务系统, 这样文件管理服务器不仅方便管理还易于扩展, 可以解决临时目录丢失的问题.
如果网络请求使用Axios, 那么文件上传有两种不同的实现方式：
## 后端配置
上传的准备工作, 其实本质就是在后端提供一个文件上传接口
### 单文件上传
```java
@RestController
public class FileUploadController {
    /**
     * 日期格式 作为目录拼接
     */
    SimpleDateFormat sdf = new SimpleDateFormat("/yyyy/MM/dd/");
    
    @PostMapping("/upload")
    public String upload(MultipartFile file, HttpServletRequest request) {
        String realPath = request.getServletContext().getRealPath("/");
        // 并且在 uploadFile 文件夹中通过日期对上传的文件归类保存
        String format = sdf.format(new Date());
        String path = realPath + format;
        File folder = new File(path);
        // 如果文件夹不存在, 则创建文件夹
        if(!folder.exists()) {
            // 创建n层目录
            folder.mkdirs();
        }
        // 获取文件扩展名后缀, 对上传的文件重命名，避免文件重名
        String oldName = file.getOriginalFilename();
        String newName = UUID.randomUUID().toString() + oldName.substring(oldName.lastIndexOf("."));
        try {
            // 文件保存
            file.transferTo(new File(folder, newName));
            // 生成上传文件的访问路径
            String s = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + format + newName;
            return s;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return "";
    }
}
```
上传的文件按照日期进行归类, 使用UUID给文件重命名避免重名
### 多文件上传
```java
@RestController
public class MultiFileUploadController {
    /**
     * 日期格式 作为目录拼接
     */
    SimpleDateFormat sdf = new SimpleDateFormat("/yyyy/MM/dd/");
    
    @PostMapping("/upload2")
    public void upload2(MultipartFile[] files, HttpServletRequest request) {
        String realPath = request.getServletContext().getRealPath("/");
        // 并且在 uploadFile 文件夹中通过日期对上传的文件归类保存
        String format = sdf.format(new Date());
        String path = realPath + format;
        File folder = new File(path);
        // 如果文件夹不存在, 则创建文件夹
        if(!folder.exists()) {
            // 创建n层目录
            folder.mkdirs();
        }
        try {
            for (MultipartFile file : files) {
                // 获取文件扩展名后缀, 对上传的文件重命名，避免文件重名
                String oldName = file.getOriginalFilename();
                String newName = UUID.randomUUID().toString() + oldName.substring(oldName.lastIndexOf("."));
                // 文件保存
                file.transferTo(new File(folder, newName));
                // 生成上传文件的访问路径
                String s = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + format + newName;
                System.out.println("s = " + s);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
**多文件上传使用的是数组MultipartFile[], 需要遍历上传的文件**

## 通过Ajax实现文件上传
```vue
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <script src="https://code.jquery.com/jquery-3.5.1.js" integrity="sha256-QWo7LDvxbWT2tbbQ97B53yJnYU3WhH/C8ycbRAkjPDc=" crossorigin="anonymous"></script>
</head>
<body>
<div id="result"></div>
    <input type="file" id="file">
    <input type="button" value="上传" onclick="uploadFile()">
<script>
    function uploadFile() {
        var file = $("#file")[0].files[0];
        var formData = new FormData();
        formData.append("file", file);
        formData.append("username", "sihai");
        $.ajax({
            type:'post',
            url:'/upload',
            processData: false,
            contentType: false,
            data: formData,
            success: function (msg) {
                $("#result").html(msg);
            }
        })
    }
</script>
</body>
</html>
```
<blockquote>
<ul>
<li>从file对象中, 获取要上传的文件, 由于此处为单文件上传, 所以获取数组中的第一项</li>
<li>构建一个FormData, 用于存放上传的数据, FormData不可以像Java中的StringBuffer使用链式配置</li>
<li>构建好FormData后便可以上传数据, FormData就是需要上传的数据</li>
<li>1.请求方法为post<br/>2.设置Content-Type为multipart/form-data</li>
</ul>
</blockquote>

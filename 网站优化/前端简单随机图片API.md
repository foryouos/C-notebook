# randomimgs

> 通过`前端js`跳转实现随机图片API

## 思路

* Github page可以上传静态代码
* js可以随机读取url文件，并在js数组中用random函数实现随机返回url部分内容
* 使用js跳转功能，当用户通过Github Page点击到前端页面时，背后的Js将根据随机函数，随机跳转到图片的url
* 图片以第三方图床的形式保存
* [TG图床](https://imgtg.com/)
* Github Page使用cloudflare实现加速同步部署

## 代码实现

```text
2023/03/16/l51Lx
2023/03/14/fLKzb
2023/03/09/Y0iNK
2023/03/09/Y0hOs
2023/03/09/Y0zdB
2023/03/09/Y0kTl
2023/03/09/Y0qLD
2023/03/09/YSCTr
2023/03/09/YS2LU
2023/03/09/YSj7p
```
****
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="author" content="瓶子的跋涉">
    <title>Random photo API</title>
    <script src="index.js" type="text/javascript" ></script>
</head>
<body>
</body>
</html>
```
****

```js
//当打开html自动调用
window.onload = function() {
    //调用随机函数
    loadRandomImage();
}
//函数
function loadRandomImage() {
    var xhttp = new XMLHttpRequest();
    xhttp.onreadystatechange = function() {
        //始终进行
        if (xhttp.status === 200) {
            //将所有图片url放入imageUrls当中
            var imageUrls = this.responseText.split("\n");
            //实现随机访问
            var randomIndex = Math.floor(Math.random() * imageUrls.length);
            //实现额面跳转
            window.location.href="https://i.imgtg.com/" + imageUrls[randomIndex] + ".jpg";
        }
        else{
            console.log("图片文件状态异常，可联系foryouos@qq.com")
        }
    };
    //打开文件
    xhttp.open("GET", "photos_urls.txt", true);
    xhttp.send();
}
```


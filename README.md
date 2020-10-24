



 

#### GitHub图床是个不错的选择，利用jsDelivr CDN加速访问（jsDelivr 是一个免费开源的 CDN 解决方案），PicGo工具一键上传，操作简单高效，GitHub和jsDelivr都是大厂，不用担心跑路问题，不用担心速度和容量问题，而且完全免费，可以说是目前免费图床的最佳解决方案！


##### 新建GitHub仓库
登录/注册[GitHub](https://github.com/)，新建一个仓库，填写好仓库名，仓库描述，根据需求选择是否为仓库初始化一个README.md描述文件.
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos1.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos2.png)



#####  `– 生成一个Token`

在主页依次选择【Settings】-【Developer settings】-【Personal access tokens】-【Generate new token】，填写好描述，勾选【repo】，然后点击【Generate token】生成一个Token，注意这个Token只会显示一次，自己先保存下来，或者等后面配置好PicGo后再关闭此网页:
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos3.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos4.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos5.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos6.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos7.png)






#####  `– 配置PicGo`

前往下载[PicGo](https://github.com/Molunerfinn/picgo/releases)，安装好后开始配置图床
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos11.png)

设定仓库名：按照【用户名 / 图床仓库名】的格式填写

设定分支名：【master】

设定Token：粘贴之前生成的【Token】

指定存储路径：填写想要储存的路径，如【ITRHX-PIC/】，这样就会在仓库下创建一个名为 ITRHX-PIC 的文件夹，图片将会储存在此文件夹中

设定自定义域名：它的的作用是，在图片上传后，PicGo会按照【自定义域名+上传的图片名】的方式生成访问链接，放到粘贴板上，因为我们要使用jsDelivr加速访问，所以可以设置为【https://cdn.jsdelivr.net/gh/用户名/图床仓库名 】.

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos8.png)

##### `– 进行高效创作`
配置好PicGo后，我们就可以进行高效创作了，将图片拖拽到上传区，将会自动上传并复制访问链接，将链接粘贴到博文中就行了，访问速度杠杠的，此外PicGo还有相册功能，可以对已上传的图片进行删除，修改链接等快捷操作，PicGo还可以生成不同格式的链接、支持批量上传、快捷键上传、自定义链接格式、上传前重命名等，更多功能自己去探索吧！

![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos10.png)
![](https://cdn.jsdelivr.net/gh/liaoyio/imgHosting/img-lyphotos9.png)

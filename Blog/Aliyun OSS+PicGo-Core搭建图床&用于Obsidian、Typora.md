# 1. 开通阿里云OSS服务
# 2. PicGo-Core安装及配置
PicGo依赖于Node环境，需要先安装Node环境
安装`homebrew`包管理工具
```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
执行上述命令完成安装！
安装`NVM`node版本管理工具
终端输入`{shell icon}brew install nvm
`homebrew`安装并不会自动配置环境遍历变量需手动添加
配置nvm环境变量
```sh
# nvm Config
export NVM_DIR="$HOME/.nvm"
[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"
```
安装稳定版node
`{sh icon}nvm install stable`
安装PicGo-Core
`{sh icon}npm install picgo -g`
使用`{sh icon}picgo -v`命令查看PicGo-Core是否安装成功
不必安装PicGo（app），和core主要有以下区别：
- PicGo提供了GUI，比PicGo-Core更易于配置
- PicGO-Core是PicGo底层核心组件，支持CLI、API调用
- 使用PicGo-Core上传会消耗较少的计算资源，仅在上传过程中运行；PicGo上传时，需始终保持运行，占用更过计算机资源。
## PicGO-Core配置
打开PicGo-Core配置文件`config.json`，配置阿里云OSS
`{sh icon} vim ~/.picgo/config.json`
```json
{
  "picBed": {
    "uploader": "aliyun", // 代表当前的默认上传图床为 aliyun,
    "aliyun": {
      "accessKeyId": "xxxxxxxxxxxxxxxxxx", // OSS中的accessKeyId
      "accessKeySecret": "xxxxxxxxxxxxxxxxxxxxxx", // OSS中的accessKeySecret
      "bucket": "atlantis-picgo-core", // 存储空间名
      "area": "oss-cn-beijing", // 存储区域代号
      "path": "picgo/", // 自定义存储路径
      "customUrl": "", // 自定义域名，注意要加 http://或者 https://
      "options": "" // 针对图片的一些后缀处理参数 PicGo 2.2.0+ PicGo-Core 1.4.0+
    },
    "current": "smms"
  },
  "picgoPlugins": { // 插件
    "picgo-plugin-rename-file": true
  },
  "picgo-plugin-rename-file": {
    "format": "{y}{m}{d}{h}{i}{s}-{rand}-{origin}"
  }
}
```
阿里云OSS部分参数位置：
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250709182048-79718d-20250709182047417.png)
安装插件
`{sh icon}picgo install rename-file`
插件说明文档
```txt
format，文件(路径)格式，默认为空，自定义文件路径及文件名，例如：
fix-dir/{localFolder:2}/{y}/{m}/{d}/{h}-{i}-{s}-{hash}-{origin}-{rand:6}
 
上传文件名为/images/test/localImage.jpg的文件时，会重命名为
fix-dir/images/test/2020/07/24/21-40-31-36921a9c364ed4789d4bc684bcb81d62-localImage-fa2c97.jpg
 
命名规则：
{y} 年，4位
{m} 月，2位
{d} 日期，2位
{h} 小时，2位
{i} 分钟，2位
{s} 秒，2位
{ms} 毫秒，3位(v1.0.4)
{timestamp} 时间戳(秒)，10位(v1.0.4)
{hash}，文件的md5值，32位
{origin}，文件原名（会去掉后缀）
{rand:<count>}, 随机数，<count>表示个数，默认为6个，示例：{rand：32}、{rand}
{localFolder:<count>}, <count>表示层级 ，默认为1，示例：{localFolder:6}、{localFolder}
```
### PicGo-Core上传测试
复制任意一张图片在终端中输入`picgo u`即可上传复制的图片，成功后并返回阿里云OSS提供的访问地址。
# 3. Obsidian&Typora配置
## Obsidian
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250709183151-c05a8c-20250709183149729.png)
命令：`{sh icon} /Users/qishuai/.nvm/versions/node/v24.3.0/bin/node /Users/qishuai/.nvm/versions/node/v24.3.0/bin/picgo u`
> ⚠️注意： macOS图形界面不会自动加载`~/.zshrc`环境变量，所以nvm配置的node环境在GUI中是不可见的。需要配置完整的Node和PicGo路径
> 


## Typora
![image.png](https://atlantis-picgo-core.oss-cn-beijing.aliyuncs.com/picgo/20250709183217-7c74f6-20250709183216840.png)

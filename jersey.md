linkis的 DataWorkCloudApplication启动类，JettyServletWebServerFactory创建时，已经移除了jersey的ServletContainer，使用spring的DispatcherServlet。如果上游组件有依赖了DataWorkCloudApplication启动类，那么原来jersey模式的http无法直接使用。


**请求返回的结构体调整**
2 http请求返回实体 javax.ws.rs.core.Response jersey内部有做单独的处理,封装为了Response，
spingmvc风格模式需要修改，直接返回Message
  return Message.messageToResponse(Message.ok().data("test", data));
=>
  return Message.ok().data("test", data)

**jackson升级替换**
org.codehaus.jackson 从v2版本时已经从 codehaus 移交到github 并重命名为com.fasterxml.jackson
jersey老版本使用的是老版本的jackson，springmvc使用的是新版本的fasterxml
替换为springmvc风格时，需要升级jackson


**web应用新增启动参数**

```
spring.mvc.servlet.path=/api/rest_j/v1
#spring.spring.mvc.servlet.path=/api/rest_j/v1
#--spring.mvc.servlet.path=/api/rest_j/v1
```

**注解对比**


|  jersey注解| springmvc注解 | 备注 |
| --- | --- | --- |
|  @GET |   @RequestMapping(method = RequestMethod.GET)|  |
|  @POST| @RequestMapping(method = RequestMethod.POST) |  |
|  @DELETE| @RequestMapping(method = RequestMethod.DELETE) |  |
|  @PUT| @RequestMapping(method = RequestMethod.PUT) |  |
| @Path("/dss") | @RequestMapping(path = "/dss) |  |
|  @FormDataParam("system") String system | @RequestParam(value ="system",required = false)|request为false|
 |  @QueryParam("system") String system |@RequestParam(value ="system",required = false)|request为false|
|  @PathParam("id") Long id|@PathVariable("id") Long id |  |
| FormDataMultiPart form  |@RequestParam("file") List<MultipartFile> files  | 默认参数名为file，用法需要修改 |
|@Context  |  直接删除|  |
|  @DefaultValue("1000") @QueryParam("pageSize") int pageSize, |   @RequestParam(value = "pageSize",defaultValue = "1000")|  |
|@Consumes(MediaType.APPLICATION_JSON)| @RequestMapping(consumes = {"application/json"})||
|@Produces(MediaType.APPLICATION_JSON)|@RequestMapping(produces = {"application/json"})| |
|参数 org.codehaus.jackson.JsonNode|@RequestBody com.fasterxml.jackson.databind.JsonNode jsonNode|jersey老版本使用的是老版本的jackson，springmvc使用的是新版本的JsonNode


**jackson主要替换点**

```
jackson-1.X 方法： getBooleanValue()、getFields()、getElements()、getIntValue()
jackson-2.X 方法： booleanValue()、fields()、elements() 和 intValue()
详细可参考
https://stackoverflow.com/questions/55896802/upgrade-of-jackson-from-org-codehaus-jackson-to-com-fasterxml-jackson-version-1
http://www.cowtowncoder.com/blog/archives/2012/04/entry_469.html

```

**Linkis组件jar包和测试环境信息**
Linkis mvn包引入方式
对应的1.0.3-SNAPSHOT包已上传至行内maven仓库 ,mvn引用示例如下
```
<dependency>
  <groupId>org.apache.linkis</groupId>
  <artifactId>linkis-httpclient</artifactId>
  <version>1.0.3-SNAPSHOT</version>
</dependency>
```
Linkis Apache1.0.3已部署的行内测试环境信息
```
部署主机：10.107.97.41
Linkis管理台访问地址 http://10.107.97.41:8890/
gateway http://10.107.97.41:9001/

```



替换的正则

```
1.GET|POST|PUT|DELETE替换
查找值
@(GET|POST|PUT|DELETE)
@Path\("([{}/:.*_a-zA-Z]*)"\)
替换值
@RequestMapping(value = "$2",method = RequestMethod.$1)

2.@Context移除
查找值
@Context HttpServlet
替换值
HttpServlet

3.返回数据结构体Response 修改为Message（Message.messageToResponse不需要执行）
查找值
Message.messageToResponse\((.*)\);$
替换值
$1;

4.http处理方法返回值替换
查找值
public Response ([a-zA-Z]*\(HttpServletRequest request)
替换值
public Message $1

5.参数替换
输入值
@(FormDataParam|QueryParam)\(([a-zA-Z_"]*)\) -> @RequestParam(value = $2,required = false)
FormDataMultiPart -> @RequestParam("file") List<MultipartFile> files
@PathParam -> @PathVariable
@DefaultValue 修改为@RequestParam 的defaultValue属性


@Component
@Path\(([a-zA-Z/"]+)\)

->
@RestController
@RequestMapping(path = $1)


@Path\(([a-zA-Z/"]+)\)
@Component
->
@RestController
@RequestMapping(path = $1)
```

## 包名修改替换脚本
```
#!/bin/bash
export GIT_COMPLETION_CHECKOUT_NO_GUESS=1
echo "test!"
rm -rf tmp.txt
find  ../ -name wedatasphere -type d  >>tmp.txt
echo "find result:"
echo "==============="
cat tmp.txt

rm -rf exe.sh
echo "=======begin to replace ========"
echo "#!/bin/bash" >>exe.sh
cat tmp.txt | while read line
do

  #info="git mv $line ${line/webank/apache}"
  dist="mkdir -pv  ${line/com\/webank\/wedatasphere/org\/apache}"
  echo $dist>>exe.sh
  info="git mv $line/* ${line/com\/webank\/wedatasphere/org\/apache}"
  echo $info >> exe.sh
  echo "rm -rf ${line/com\/webank\/wedatasphere/com}">>exe.sh
  echo "">>exe.sh
  echo "">>exe.sh
#  rm -rf $line
#  $info
#  info="cp $name $line "
#  echo "$info"

#  $info
done
echo "=======end========"

```

1.常用打包命令

mvn clean package -Dmaven.test.skip=true -- 跳过单测打包
mvn clean install -Dmaven.test.skip=true -- 跳过单测打包，并把打好的包上传到本地仓库
mvn clean deploy -Dmaven.test.skip=true -- 跳过单测打包，并把打好的包上传到远程仓库

2.[maven](https://so.csdn.net/so/search?q=maven&spm=1001.2101.3001.7020)命令

mvn -v //查看版本
mvn archetype:create //创建 Maven 项目
mvn compile //编译源代码
mvn test-compile //编译测试代码
mvn test //运行应用程序中的单元测试
mvn site //生成项目相关信息的网站
mvn package //依据项目生成 jar 文件
mvn install //在本地 Repository 中安装 jar
mvn -Dmaven.test.skip=true //忽略测试文档编译
mvn clean //清除目标目录中的生成结果
mvn clean compile //将.java类编译为.class文件
mvn clean package //进行打包
mvn clean test //执行单元测试
mvn clean deploy //部署到版本仓库
mvn clean install //使其他项目使用这个jar,会安装到maven本地仓库中
mvn archetype:generate //创建项目架构
mvn dependency:list //查看已解析依赖
mvn dependency:tree com.xx.xxx //看到依赖树
mvn dependency:analyze //查看依赖的工具
mvn help:system //从中央仓库下载文件至本地仓库
mvn help:active-profiles //查看当前激活的profiles
mvn help:all-profiles //查看所有profiles
mvn help:effective -pom //查看完整的pom信息

3.注意
maven 命令要在IDEA的Terminal窗口执行
执行maven命令需要当前目录有pom依赖，可以用cd命令切换目录

4.打包时注意：

当mvn仓库里缺少jar包，同时又从中央仓库自动下载不下来的时候，就需要自己下载jar包然后放仓库里了，
但是有时候只是简单的把jar和source放仓库的文件夹下，并不管用，这个时候你可以用命令把jar把打进去：
mvn install:install-file -Dfile=D:\xxx.jar -DgroupId=commons-dbcp -DartifactId= commons-dbcp -Dversion= 1.4 -Dpackaging=jar
-Dfile 是存在本地磁盘里jar 的路径，后面的就不用说了吧！install:install-file 看清楚了！！！这个-file跟install是连着的。

mvn dependency:tree命令解决jar包冲突
当项目出现jar包冲突时,用命令mvn dependency:tree 查看依赖情况
mvn dependency:tree 查看依赖树,查看包结构间的依赖
mvn dependency:tree >d:/tmp 把结果输出到文件，
然后再pom.xml文件里排除掉冲突的jar包
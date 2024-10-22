#### Java位运算的基础及使用（意义）

  *     *       *         * 前言
        *       一、位运算基础
        *       二、位运算应用
        *       三、位运算试题

##### 前言

这几天在看HashMap的源码，但里面的位运算太多，看得有点晕。故，先整理位运算相关知识。  
在了解位运算的计算后，又在思考，使用位运算的意义是什么，毕竟平时开发基本没用过位运算。经大量的资料查找，整理了两个自己感觉比较好的位运算利用例子，特在此记录，分享。  
**另外，毕竟位运算的代码可读性差，请大家谨慎使用。**

##### 一、位运算基础

1、位运算是针对整数的二进制进行的位移操作  
2、整数 32位 , 正数符号为0，负数符号为1。十进制转二进制 不足32位的，最高位补符号位，其余补零  
3、在Java中，整数的二进制是以补码的形式存在的  
4、位运算计算完，还是补码的形式，要转成原码，再得出十进制值  
5、正数：原码=反码=补码 负数：反码=原码忽略符号位取反, 补码=反码+1

例如：十进制4 转二进制在计算机中表示为（补码） 00000000 00000000 00000000 00000100

例如：十进制-4 转二进制在计算机中表示为（补码） 11111111 11111111 11111111 11111100

**负数转二进制过程（以-4为例）**


​    

    原码：10000000 00000000 00000000 00000100（转二进制，最高位为符号位）  
    反码：11111111 11111111 11111111 11111011（符号位不变，其余取反）  
    补码：11111111 11111111 11111111 11111100（反码+1）


**-4 << 1 计算过程**


​    

    -4 补码  11111111 11111111 11111111 11111100  
    左移一位 11111111 11111111 11111111 11111000 （这时候还是补码）
    # 如果最高位符号位为0，就不需要继续操作了，因为正数的补码=原码，如果最高位是1，继续往下走
    转成反码 11111111 11111111 11111111 11110111 （补码-1）  
    转成原码 10000000 00000000 00000000 00001000 （忽略符号位取反）  
    转十进制 -8  


  * 左移（ << ） 整体左移，右边空出位补零，左边位舍弃 （-4 << 1 = -8）
  * 右移（ >> ） 整体右移，左边空出位补零或补1（负数补1，整数补0），右边位舍弃 （-4 >> 1 = -2）
  * 无符号右移（ >>> ）同>>,但不管正数还是负数都左边位都补0 （-4 >>> 1 = 2147483646）
  * 与（ & ）每一位进行比较，两位都为1，结果为1，否则为0（-4 & 1 = 0）
  * 或（ | ）每一位进行比较，两位有一位是1，结果就是1（-4 | 1 = -3）
  * 非（ ~ ） 每一位进行比较，按位取反（符号位也要取反）（~ -4 = 3）
  * 异或（ ^ ）每一位进行比较，相同为0，不同为1（^ -4 = -3）

* * *

##### 二、位运算应用

在一个系统中，用户一般有查询(Select)、新增(Insert)、修改(Update)、删除(Delete)四种权限，四种权限有多种组合方式，也就是有16中不同的权限状态（2的4次方）。

**Permission**

一般情况下会想到用四个boolean类型变量来保存：


​    

    public class Permission {
    
    	// 是否允许查询
    	private boolean allowSelect;
    
    	// 是否允许新增
    	private boolean allowInsert;
    
    	// 是否允许删除
    	private boolean allowDelete;
    
    	// 是否允许更新
    	private boolean allowUpdate;
    
    	// 省略Getter和Setter
    }


上面用四个boolean类型变量来保存每种权限状态。

**NewPermission**

下面是另外一种方式，使用位掩码的话，用一个二进制数即可，每一位来表示一种权限，0表示无权限，1表示有权限。


​    

    public class NewPermission {
    	// 是否允许查询，二进制第1位，0表示否，1表示是
    	public static final int ALLOW_SELECT = 1 << 0; // 0001
    
    	// 是否允许新增，二进制第2位，0表示否，1表示是
    	public static final int ALLOW_INSERT = 1 << 1; // 0010
    
    	// 是否允许修改，二进制第3位，0表示否，1表示是
    	public static final int ALLOW_UPDATE = 1 << 2; // 0100
    
    	// 是否允许删除，二进制第4位，0表示否，1表示是
    	public static final int ALLOW_DELETE = 1 << 3; // 1000
    
    	// 存储目前的权限状态
    	private int flag;
    
    	/**
    	 *  重新设置权限
    	 */
    	public void setPermission(int permission) {
    		flag = permission;
    	}
    
    	/**
    	 *  添加一项或多项权限
    	 */
    	public void enable(int permission) {
    		flag |= permission;
    	}
    
    	/**
    	 *  删除一项或多项权限
    	 */
    	public void disable(int permission) {
    		flag &= ~permission;
    	}
    
    	/**
    	 *  是否拥某些权限
    	 */
    	public boolean isAllow(int permission) {
    		return (flag & permission) == permission;
    	}
    
    	/**
    	 *  是否禁用了某些权限
    	 */
    	public boolean isNotAllow(int permission) {
    		return (flag & permission) == 0;
    	}
    
    	/**
    	 *  是否仅仅拥有某些权限
    	 */
    	public boolean isOnlyAllow(int permission) {
    		return flag == permission;
    	}
    }


以上代码中，用四个常量表示了每个二进制位代码的权限项。例如：ALLOW_SELECT = 1 << 0 转成二进制就是0001，二进制第一位表示Select权限。ALLOW_INSERT = 1 << 1 转成二进制就是0010，二进制第二位表示Insert权限。private int flag存储了各种权限的启用和停用状态，相当于代替了Permission中的四个boolean类型的变量。用flag的四个二进制位来表示四种权限的状态，每一位的0和1代表一项权限的启用和停用，下面列举了部分状态表示的权限：

| flag       | 删除 | 修改 | 新增 | 查询 |
| ---------- | ---- | ---- | ---- | ---- |
| 1（0001）  | 0    | 0    | 0    | 1    |
| 2（0010）  | 0    | 0    | 1    | 0    |
| 4（0100）  | 0    | 1    | 0    | 0    |
| 8（1000）  | 1    | 0    | 0    | 0    |
| 3（0011）  | 0    | 0    | 1    | 1    |
| 0（0000）  | 0    | 0    | 0    | 0    |
| 15（1111） | 1    | 1    | 1    | 1    |

使用位掩码的方式，只需要用一个大于或等于0且小于16的整数即可表示所有的16种权限的状态。

此外，还有很多设置权限和判断权限的方法，需要用到位运算，例如：


​    

    public void enable(int permission) {
    	flag |= permission; // 相当于flag = flag | permission;
    }


调用这个方法可以在现有的权限基础上添加一项或多项权限。

添加一项Update权限：


​    

    permission.enable(NewPermission.ALLOW_UPDATE);


假设现有权限只有Select，也就是flag是0001。执行以上代码，flag = 0001 | 0100，也就是0101，便拥有了Select和Update两项权限。

添加Insert、Update、Delete三项权限：


​    

    permission.enable(NewPermission.ALLOW_INSERT 
        | NewPermission.ALLOW_UPDATE | NewPermission.ALLOW_DELETE);


NewPermission.ALLOW_INSERT | NewPermission.ALLOW_UPDATE | NewPermission.ALLOW_DELETE运算结果是1110。假设现有权限只有Select，也就是flag是0001。flag = 0001 | 1110，也就是1111，便拥有了这四项权限，相当于添加了三项权限。上面的设置如果使用最初的Permission类的话，就需要下面三行代码：
    
    

    permission.setAllowInsert(true);
    permission.setAllowUpdate(true);
    permission.setAllowDelete(true);


二者对比  
设置仅允许Select和Insert权限

Permission


​    

    permission.setAllowSelect(true);
    permission.setAllowInsert(true);
    permission.setAllowUpdate(false);
    permission.setAllowDelete(false);


NewPermission


​    

    permission.setPermission(NewPermission.ALLOW_SELECT | NewPermission.ALLOW_INSERT);


判断是否允许Select和Insert、Update权限

Permission


​    

    if (permission.isAllowSelect() && permission.isAllowInsert() && permission.isAllowUpdate())


NewPermission


​    

    if (permission. isAllow (NewPermission.ALLOW_SELECT 
        | NewPermission.ALLOW_INSERT | NewPermission.ALLOW_UPDATE))  


判断是只否允许Select和Insert权限

Permission


​    

    if (permission.isAllowSelect() && permission.isAllowInsert() 
        && !permission.isAllowUpdate() && !permission.isAllowDelete())  


NewPermission


​    

    if (permission. isOnlyAllow (NewPermission.ALLOW_SELECT | NewPermission.ALLOW_INSERT))  


二者对比可以感受到MyPermission位掩码方式相对于Permission的优势，可以节省很多代码量，位运算是底层运算，效率也非常高，而且理解起来也很简单。

.

**可以替代位域的更好的方案**  
在《Effective Java》一书中，更推荐用EnumSet来代替位域：位域表示法也允许利用位操作，有效的执行像union和intersection这样的集合操作。但位域有着int枚举常量所有的缺点，甚至更多。当位域以数字形式打印时，翻译位域比翻译简单的int枚举常量要困难很多。甚至要遍历位域表示的所有元素也没有很容易的方法。


​    

    public class Text {
        public static final int STYLE_BOLD          = 1 << 0;
        public static final int STYLE_ITALIC        = 1 << 1;
        public static final int STYLE_UNDERLINE     = 1 << 2;
        public static final int STYLE_STRIKETHROUGH = 1 << 3;
    
        public void applyStyles(int styles) {...}
    }


调用：


​    

    text.applyStyles(STYLE_BOLD | STYLE_ITALIC);  


改成EnumSet的写法是：


​    

    public class Text {
    
        public enum Style {
            BOLD, ITALIC, UNDERLINE, STRIKETHROUGH
        }
    
        public void applyStyles(Set<Style> styles) {
            System.out.println(styles);
        }
    }  


调用：


​    

    text.applyStyles(EnumSet.of(Style.BOLD, Style.ITALIC));


* * *

##### 三、位运算试题

有 1000 个一模一样的瓶子，其中有 999 瓶是普通的水，有一瓶是毒药。任何喝下毒药的生物都会在一星期之后死亡。现在，你只有 10 只小白鼠和一星期的时间，如何检验出哪个瓶子里有毒药？

根据2^10=1024，所以10个老鼠可以确定1000个瓶子具体哪个瓶子有毒。具体实现跟3个老鼠确定8个瓶子原理一样。  
000=0  
001=1  
010=2  
011=3  
100=4  
101=5  
110=6  
111=7  
一位表示一个老鼠，从左到右分别代表老鼠3，老鼠2，老鼠1。0-7表示8个瓶子。也就是分别将1、3、5、7号瓶子的药混起来给老鼠1吃，2、3、6、7号瓶子的药混起来给老鼠2吃，4、5、6、7号瓶子的药混起来给老鼠3吃，哪个老鼠死了，相应的位标为1。如老鼠1死了、老鼠2没死、老鼠3死了，那么就是101=5号瓶子有毒。同样道理10个老鼠可以确定1000个瓶子

* * *

感谢

二、位运算应用 出处：叉叉哥.<https://www.zhihu.com/question/34021773/answer/118589857>

三、位运算试题 出处：黄志聪.<https://www.zhihu.com/question/19676641/answer/12613290>


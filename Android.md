# Android 开发

## Android 体系结构及开发环境

### Android 体系结构
1. **应用程序** (Application)
> 基于 Java 语言编写，基于Android 系统的 API 构建的
2. **应用程序框架** (Application Framework)
> 开发人员所使用的 API 框架
1. **函数库** (Libraries、Android Runtime)
2. **Linux 内核** (Linux Kernel)

### 开发环境的配置
#### 配置 JDK
  - [下载 JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
   > **注意记住安装路径**
  - **配置环境变量**
      - JAVA_HOME：<kbd> JDK 安装路径 </kbd>
      - PATH：&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<kbd> %JAVA_HOME%\bin </kbd>
      - CLASSPATH：<kbd> .;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar </kbd>
      > JDK1.5 后**非必要**，配置可保证向下兼容
  - 在命令行窗口中输入 <kbd> java -version </kbd> 是否配置成功
#### 安装 IDEA
  - [下载 IDEA](https://www.jetbrains.com/idea/download/)
  - [激活 IDEA](http://idea.lanyus.com/)
  - **下载 Android SDK**
    - 直接创建一个 Android Poject 按照提示安装即可
    > 此处未使用手动配置SDK，也可自行下载 [Android SDK](https://www.androiddevtools.cn/)
    - Tools - Android - SDK Manager 可进行 **SDK 管理**
    - Tools - Android - AVD Manager 可进行 **AVD 管理**
    > **AVD**：安卓虚机设备

***

## Android 应用程序开发

### Android 应用程序组成
#### Activity 组件
- 用户操作的可视化界面
- Activity 的生命周期四种状态
  - **Activity** (活动)：只有一个 Activity 处于活动状态，内容变亮
  - **Paused** (暂停)：画面变暗，禁止状态
  - **Stoped** (停止)：不可见
  - **Dead** (死亡)：被系统回收或未启动
~~~
流程图 (待补充)
~~~
#### Intent 组件
> Intent 是一种**消息机制**，用于组件之间调用的方法或所需要的数据
##### Intent 对象
- **目标组件** (Components)
- **动作** (Action)
- **数据** (Data)
- **类别** (Category)
- **附加数据** (Extras)
- **标志** (Flags)
##### 显示 Intent
> 通过 Components 实现直接调用
###### 实现方法
~~~
setComponent(ComponentName name)
Intent(Context context, Class classObjectInThatContext)
setClass(Context context, Class classObjectInThatContext)
setClassName(Context context, String classNameInThatPackage)
setClassName(String packageName, String classNameThatPackage)
~~~
##### 隐式 Intent
用于**不同应用**之间的调用
#### Broadcast Receiver 组件
广播接收器是一个专用于**接受广播通知并作出处理**的组件
#### Service 组件
Service通常位于**后台**运行，它一般不需要与用户交互，因此Service组件**没有图形用户界面**
#### Content Provider 组件
用于应用程序之间的**数据共享**

### View与ViewGroup的概念
#### View
所有可视化控件的**父类**,提供**组件描绘**和**时间处理方法**
#### ViewGroup
View类的**子类**，可以拥有子控件,可以看作是**容器**

### UI 布局
#### 公共属性
| 属性 | 描述 |
| ---- | ---- |
| **layout_width <br/> layout_height** | 全屏：&nbsp;&nbsp;match_parent <br/> 自适应：warp_conent |
| **backgroung** | 设置背景颜色/图片 |
| **gravity** | 居中：center &#124; 居上：top <br/>  居左：left &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&#124; 居右：right<br/> 垂直居中：center_vertical <br/> 水平居中：center_horizontal |
| **margin** | 外边距(marginLeft/Right/Top/Bottom) |
| **padding** | 内边距(同上，单位dp) |
#### 线性布局 (LinearLayout)
| 属性 | 描述 |
| ---- | ---- |
| **orientation** | 竖向排列：verticall <br/> 横向排列：horizonta |
| **weightSum** | 总权重 |
| **layout_weight** | 权重(元素的权重占比) |
| **layout_gravity** | 设置自己在父元素中的位置 |
#### 相对布局 (RelativeLayout)
| 属性 | 描述 |
| ---- | ---- |
| **layout_toRightOf** | 控件左边缘与某控件的右边缘对齐 |
| **layout_alignLeft** | 控件左边缘与某控件的左边缘对齐 |
| **layout_alignRight** | 与上同理，另有Top &#124; Bottom |
| **layout_alignParentLeft** | 控件左边缘与父控件左边缘对齐 |
| **layout_alignParentRight** | 与上同理，另有Top &#124; Bottom |
| **layout_centerParent** | 置于父控件中间位置 |
| **layout_centerHorizontal** | 置于父控件水平方向的中心位置 |
| **layout_centerVertical** | 置于父控件垂直方向的中心位置 |
#### 帧布局 (FrameLayout)
| 属性 | 描述 |
| ---- | ---- |
| **foreground** | 设置前景图像 |
| **foregroundGravity** | 设置前景图像位置 |
#### 表格布局 (TableLayout)
在表格布局中，TableRow 表示一行
| 属性 | 描述 |
| ---- | ---- |
| **collapseColumns** | 隐藏列 |
| **stretchColumns** | 拉伸列 |
| **shrinkColumns** | 收缩列 |
#### 绝对布局 (absoluteLayout)
| 属性 | 描述 |
| ---- | ---- |
| **layout_x** | 指定子元素的x坐标 |
| **layout_y** | 指定子元素的y坐标 |
#### 网格布局 (GridLayout)
| 属性 | 描述 |
| ---- | ---- |
| **rowCount** | 设置网格行数 |
| **columnCount** | 设置网格列数 |
| **layout_row** | 设置元素位于哪行 |
| **layout_column** | 设置元素位于哪列 |
| **layout_rowSpan** | 设置元素纵向跨越行数 |
| **layout_columnSpan** | 设置元素横向跨越行数 |
#### 约束布局 (ConstraintLayout)
**属性格式**：**layout_constraint X _to XX Of**
**格式说明**：将某控件的 X 侧与某控件的 XX 侧对齐
#### RTL布局
#### 百分比布局

### UI 控件

#### 公共属性
| 属性 | 描述 |
| ---- | ---- |
| **id** | 设置控件 id <br/> 格式：@+id/id_name |
| **layout_width** <br/> **layout_height** | 全屏：&nbsp;&nbsp;match_parent <br/> 自适应：warp_conent |
#### 有文字控件的公共属性
| 属性 | 描述 |
| ---- | ---- |
| **text** | 文本内容 |
| **textSize** | 字体大小 (单位sp，默认 14sp)
| **textColor** | 字体颜色 支持3种方式 <br/> @android:color/系统颜色 <br/> @color/自定义 &nbsp;&nbsp;&#124;&nbsp;&nbsp; #RGB |
| **textStyle** | 字体样式 (normal&nbsp;&#124;&nbsp;bold&nbsp;&#124;&nbsp;italic)
| **gravity** | 字体位置 (top&nbsp;&#124;&nbsp;bottom&nbsp;&#124;&nbsp;left&nbsp;&#124;&nbsp;right) |
| **singleLine** | 单行显示 (true &nbsp;&#124;&nbsp; false) |
| **minLine** | 最小行数 |
| **maxLine** | 最大行数 |
#### 文本框 TextView
#### 编辑框 EditView
~~~
<EditView
    android:hint          = "xxx"        // 文本提示信息
    android:password      = "true"       // 设置密码框
    android:phoneNumber   = "true"       // 只能输入数字
    android:cursorVisible = "true/false" // 设定光标显示/隐藏
/>
~~~
#### 列表 ListView
#### [RecyclerView](https://www.jianshu.com/p/bb6b029de04f)
#### 图像 ImageView
~~~
<ImageView
    android:scr                = "@drawable/项目资源 | @android:drawable/系统资源"
    android:scaleType          = "center" // 定义伸缩类型 (详见下表)
    android:backgroud          = "#RGB"
    android:contentDescription = "xxx"    // 定义内容描述
/>
~~~
| 属性 | 描述 |
| ---- | ---- |
| **matrix** | 左上角起始的矩形区域，会裁剪 |
| **fitXY** | 匹配控件宽高，会变形，不裁剪 |
| **fitStart** | 伸缩匹配控件，顶部或左侧显示，不裁剪 |
| **fitEnd** | 伸缩匹配控件，底部或右侧显示，不裁剪 |
| **fitCenter** | 伸缩匹配控件，居中显示，不裁剪 |
| **center**| 原尺寸，居中显示，会裁剪 |
| **centerCrop** | 伸缩匹配控件，居中显示，会裁剪 |
| **centerInside** | 原图或缩小匹配控件，居中显示，不裁剪 |
#### 图片浏览器 GridView
> 基于网格索引的图片浏览器
#### 按钮 Button
Button 常用子类 **CheckButton**、**RadioButton**、**ToggleButton**
#### 图片按钮 ImageButton
~~~
<ImageButton
    android:layout_width = "200dp"  // 此控件可以设置具体宽高
/>
~~~
#### 单选按钮 RadioButton 和 复选按钮 CheckButton
#### 下拉列表 Spinner
#### 进度条 ProgressBar
~~~
<ProgerssBar
    style=""         // 设置进度条样式
/>
android:max = "100"  //在进度条外设置进度条的最大值
~~~
#### 拖动条 SeekBar
#### 日期相关控件
DatePicker、TimePicker、Calendar

### 菜单
#### 菜单生成方式
##### Java 代码动态生成
~~~
final static int MENU_00 = Menu.FIRST;
final static int MENU_01 = Menu.FIRST + 1;
final static int MENU_02 = Menu.FIRST + 2;
// 创建菜单
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    menu.add(0, MENU_00, 0, "menu0").setIcon(R.drawable.pic0);
    menu.add(0, MENU_01, 1, "menu1").setIcon(R.drawable.pic1);
    menu.add(0, MENU_02, 2, "menu2").setIcon(R.drawable.pic2);
    return true;
}
~~~
###### add() 方法
    MenuItem android.view.Menu.add(int groupId, int itemId, int order, CharSequence title);
| 参数 | 描述 |
| ---- | ---- |
| **groupId** | 组 ID (不需要设置未Menu.NONE) |
| **itemID** | 每一项的ID，具有唯一性 |
| **order** | 排序信息 |
| **title** | 显示标题 |
###### setIcon() 方法
设置菜单**图标**
##### XML 菜单
~~~
<?xml version = "1.0" encoding = "utf-8">
<menu xmlns:android = "http://schemas.android.com/apk/res/android">
    <item
        android:id    = "@+id/main_menu_0"
        android:icon  = "@drawable/pic0"
        android:title = "menu0"
    />
    <item
        android:id    = "@+id/main_menu_1"
        android:icon  = "@drawable/pic1"
        android:title = "menu1"
    />
    <item
        android:id    = "@+id/main_menu_2"
        android:icon  = "@drawable/pic2"
        android:title = "menu2"
    />
</menu>
~~~

#### 选项菜单 (Option Menu)
> 图标菜单至多为 6 个，OptionsMenu被Activity或者Fragment对象持有
##### 处理方法
~~~
// 创建菜单 (加载 XML 布局)
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    getMenuInflater().inflate(R.menu.menu_main, menu);
    return true;
}

// 响应事件
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    switch (item.getIteaId()) {
        case R.id.main_menu_00:
            Toast.makeText(this, "menu1", Toast.LENGTH_SHORT). show(); break;
        case R.id.main_menu_01:
            Toast.makeText(this, "menu2", Toast.LENGTH_SHORT). show(); break;
        case R.id.main_menu_02:
            Toast.makeText(this, "memu3", Toast.LENGTH_SHORT). show(); break;
        default: break;
    }
    return super.onContextItemSelected(item);
}
~~~
#### 快捷菜单/上下文菜单 (Context Menu)
> ContextMenu是被view对象持有
~~~
// 为一个 Button 注册一个 Context Menu
registerForContextMenu(mContButton);

// 创建菜单并设置监听
@Override
public boolean onCreateOptionsMenu(Menu menu) {
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.toobar_menu,menu);
    return super.onCreateOptionsMenu(menu);
}
@Override
public boolean onContextItemSelected(MenuItem item) {
    switch (item.getItemId()){
        case R.id.share_item:
            Toast.makeText(this,"context menu add",Toast.LENGTH_SHORT).show();
    }
    return super.onContextItemSelected(item);
}
~~~
### 弹出菜单 (PopupMenu)
~~~
// 菜单逻辑封装
private void showPopupMenu() {
    //创建PopupMenu，并绑定到mContButton
    PopupMenu popupMenu = new PopupMenu(this,mContButton);
    MenuInflater menuInflater = getMenuInflater();
    menuInflater.inflate(R.menu.toobar_menu, popupMenu.getMenu());
    //popupMenu.inflate(R.menu.toobar_menu);  API 14 可以采用这种方式
    popupMenu.show();

    //设置item的点击事件
    popupMenu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
        @Override
        public boolean onMenuItemClick(MenuItem item) {
            switch (item.getItemId()){
                case R.id.add_item:
                    Toast.makeText(this,"Add button clicked",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.remo_item:
                    invalidateOptionsMenu();
                    Toast.makeText(this,"Remove button clicked",Toast.LENGTH_SHORT).show();
                    break;
                case R.id.more_item:
                    Toast.makeText(this,"More button clicked",Toast.LENGTH_SHORT).show();
                    break;
                default:
                    break;
            }
            return  true;
        }
    });
}
// mContButton 的点击事件监听
mContButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        showPopupMenu();
    }
});
~~~

### 事件处理
#### 基于监听的事件处理
##### 处理机制中涉及的对象：
1. **事件源**：事件发生的组件
2. **事件**：是对整个事件信息的封装
3. **事件处理器**：完成事件的处理
##### 常用处理机制
###### 在布局文件中指定处理函数
~~~
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <Button
        android:id="@+id/loginButton"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="loginButtonClick"  // 指定单击事件的处理函数
        android:text="登陆" />
</LinearLayout>

// 编写事件处理函数
public class MainActivity extends AppCompatActivity {
    public void loginButtonClick (View view){
        // pass
    }
}
~~~
###### 通过具体的对象实现事件处理
~~~
public class MainActivity extends AppCompatActivity {
    private Button loginButton;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        loginButton = findViewById(R.id.loginButton);  // 根据ID查找组件
        // 设置单击事件处理
        loginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                // pass
            }
        });
    }
}
~~~
#### [基于回调的事件处理机制](https://blog.csdn.net/salary/article/details/82787626)0

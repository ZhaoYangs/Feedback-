# **Android 用户反馈 SDK 5.0使用指南**
友盟用户反馈SDK帮助开发者和用户进行直接的沟通和交流。 

* <font color="red">重要通知：Android 用户反馈自4.3以后的版本在结构上有重大更新，集成接口也有变化。 提供给开发者更多的自定义灵活性。如果您集成过老版本的反馈SDK， 请先卸载老版本SDK， 将对应集成代码删除。</font>


<a name="1"></a>
## 1. 导入SDK 所需 jar包

◆`Eclipse`用户
下载最新版SDK的zip包，解压后将其中的libs/目录合并到本地工程`libs`目录。

右键工程根目录，选择`Properties -> Java Build Path -> Libraries`，然后点击`Add External JARs...` 选择指向jar的路径，点击`OK`，即导入成功。


> 注意: Eclipse ADT 17 以上版本用户，请在工程目录下建一个文件夹`libs`，把jar包直接拷贝到这个文件夹下，再在Eclipse里面刷新一下工程就好了。不要通过上述步骤手动添加jar包引用。 详情请参考[Dealing with dependencies in Android projects](http://tools.android.com/recent/dealingwithdependenciesinandroidprojects).

◆`Android Strdio`用户
可以使用maven仓库的方式集成，使用此方式不需要手动导入SDK。集成方法：

修改build.gradle文件添加代码：
```java
 repositories {
          jcenter()
          maven { url "https://raw.githubusercontent.com/umeng/mvn-repo-umeng/master/repository" }
  }
```
然后引用sdk即可
```java
dependencies {
      compile 'com.android.support:support-v4:20.+'
      compile 'com.umeng:fb:5.0.0'
  }
```
也可以使用和`Eclipse`用户相同的集成方法，下载最新版SDK的zip包，解压后将其中的libs/目录合并到本地工程`libs`目录。右键jar文件选择`Add As Libaray...`即可。
<a name="2"></a>
## 2. 添加资源文件
将SDK提供的`res`文件夹拷入工程目录下, 和工程本身`res`目录合并。 您可以更改资源内容但是请不要更改文件名和资源ID。

> 提示: 友盟SDK提供的资源文件都以`umeng_`开头，`Android Strdio`用户使用maven仓库的方式集成SDK不需要添加资源文件。


## 3. 配置 AndroidManifest.xml 

打开`AndroidManifest.xml`, 在`<application>`标签中添加Activity, APPKEY, 和权限

```xml
<manifest……>
	<application ……>
		<activity 
		     android:name="com.umeng.fb.ConversationActivity"/>
		<meta-data 
		     android:value="YOUR_APP_KEY" 
		     android:name="UMENG_APPKEY"/>
		<meta-data 
		     android:value="Channel ID" 
		     android:name="UMENG_CHANNEL"/>
	</application>
	<uses-sdk android:minSdkVersion="4"/>
	<uses-permission android:name="android.permission.INTERNET"/>
	<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
</manifest>
```

## 4. 添加代码

> 请参考SDK 下载包中的example 工程代码: `umeng-sdk/example/src/com/umeng/example/fb`.                                            
> 注：SDK使用了Android Support Library, 所以请添加最新版本 android-support-v4.jar 到工程的libs目录下。

### 4.1 基础功能设置
代码中启用`Feedback`模块，调用下面函数进入反馈界面：

```java
FeedbackAgent agent = new FeedbackAgent(context);
agent.startFeedbackActivity();
```

如果开发者需要自定义反馈会话界面，可使用SDK提供的`Fragment`，将`FeedbackFragment`添加到`Activity`中。`example`：

```java
String conversation_id = getIntent().getStringExtra(FeedbackFragment.BUNDLE_KEY_CONVERSATION_ID);
getSupportFragmentManager().beginTransaction()
                    .add(R.id.container, FeedbackFragment.newInstance(conversation_id))
                    .commit();
```

### 4.2 反馈推送功能设置

下载友盟消息推送SDK，并将jar包导入项目。按照友盟消息推送SDK集成文档增加对AndroidManifest.xml文件的配置，文档地址：
http://dev.umeng.com/message/android/integration-guide
> 提示: `Android Strdio`用户也可以直接引用友盟消息推送SDK，即在build.gradle文件添加代码：
```java
dependencies {
      compile 'com.umeng:message:1.4.1'
  }
```

在应用主`Activity`的`onCreate()`方法中开启反馈回复推送服务，同时开启友盟消息推送服务。
`example ：`
```java
agent.openFeedbackPush();
PushAgent.getInstance(this).enable();
```
> 注意：使用反馈推送功能，需要到友盟消息推送后台（http://www.umeng.com/push ）开启友盟消息推送功能，否则会影响使用

开启反馈推送功能后，开发者的回复将会被实时推送到应用。开发者实现分为三种场景，开发者应按照相应场景进行设置：

##### ◆ 场景一 : 在应用中只集成了友盟用户反馈SDK，没有集成友盟消息推送SDK，或同时集成了两个SDK，但没有使用友盟消息推送自定义消息

在`Application`的`onCreate()`方法中调用下面的方法初始化反馈推送的相关设置，参数为`false`。`example：`
```java
FeedbackPush.getInstance(this).init(false);
```
在配置文件`AndroidManifest.xml`中设置自己的`Application`。     



##### ◆ 场景二 : 在应用中同时集成了友盟用户反馈SDK和友盟消息推送SDK，且使用友盟消息推送自定义消息 

在`Application`的`onCreate()`方法中调用下面的方法初始化反馈推送的相关设置，参数为`true`。`example：`

```java           
FeedbackPush.getInstance(this).init(true);
```
在配置文件`AndroidManifest.xml`中设置自己的`Application`。  
按照友盟消息推送SDK集成文档说明，重写友盟消息推送SDK提供的消息处理类`UmengMessageHandler`的`dealWithCustomMessage(Context context, UMessage msg)`方法，在该方法中的第一行调用`dealFBMessage(UMessage msg)`方法，如果返回为`false`，开发者可继续处理该非反馈推送消息，如果为`true`，开发者应直接退出该方法，即 `return;`。 `example：`

```java 
 mMessageHandler = new UmengMessageHandler() {
            @Override
            public void dealWithCustomMessage(Context context, UMessage msg) {
                if (FeedbackPush.getInstance(context).dealFBMessage(new FBMessage(msg.custom))) {
                    return;
                }
                //此推送消息非开发者回复消息，开发者可以继续处理该消息
                /*************** 其他操作 ***************/
            }
        };
```


##### ◆ 场景三 : 在应用中同时集成了友盟用户反馈SDK和友盟消息推送SDK，使用完全自定义消息

按照友盟消息推送SDK集成文档说明，重写友盟消息推送SDK提供的`UmengBaseIntentService`类中的`onMessage(Context context, Intent intent)`方法，在调用`super.onMessage(context, intent);`之后，调用`onFBMessage(Intent intent)`方法,如果`onFBMessage(Intent intent)`方法返回为`false`，开发者可继续处理该非反馈推送消息，如果为`true`，开发者应直接退出该方法，即`return;`。`example ：`

```java
 @Override
    protected void onMessage(Context context, Intent intent) {
        super.onMessage(context, intent);
        if (FeedbackPush.getInstance(context).onFBMessage(intent)) {
            return;
        }
       //此推送消息非开发者回复消息，开发者可以继续处理该消息
       /*************** 其他操作 ***************/
    }
```

另外：开发者如果使用`FeedbackFragment`自定义反馈会话界面则应将调用`init(true)`改为调用重载的`init(Class<?> cls，boolean  isCustom)`方法,参数`cls`为自定义的`Activity`。`example：`
 
```java
FeedbackPush.getInstance(this).init(ConversationActivity.class, true);
```

在自定义的Activity中复写onNewIntent()方法，在onNewIntent()调用以下方法：

```java
mFeedbackFragment.addPushDevReply();
```

> 提示：其他涉及到友盟消息推送SDK的地方均可按照友盟消息推送SDK的要求来编写代码。


### 4.3 附加功能设置

* 设置新回复通知

当开发者回复用户反馈后，如果需要提醒用户，请在应用程序的入口`Activity`的`OnCreate()`方法中下添加以下代码

```java
agent.sync();
```

若调用该接口，反馈模块将在你程序启动后于后台检查是否有新的来自开发者的回复。 若有，我们将在通知栏提醒用户，若无，则不会打扰用户。你也可以选择不调用该接口，这样我们会在用户进入反馈界面后，再去检查是否存在新的回复。
如果你希望改变默认通知方式， 可以使用接口

```java
agent.getDefaultConversation().sync(listener);
```


* 修改反馈页面默认欢迎语

要想修改反馈页面默认欢迎语，可以修改`res/layout/umeng_fb_welcome_item.xml` 文件,找到id为`umeng_fb_welcome_info`的TextView,修改该TextView的`android:text`属性。

* 关于用户信息

如果用户没有提交联系信息， SDK提供的反馈信息界面会提供入口让用户填写信息。 在用户首次填写信息之后会将入口关闭， 用户填写信息提交之后不能修改。 如果您希望改变SDK提供的这种默认行为， 可以参考代码 https://github.com/umeng/umeng-android-sdk-theme/blob/master/fb/v4.3/src/com/umeng/fb/ 定制适合自己应用的界面。


<a name="3"></a>
## 5. 高级定制
从v4.3版本开始， 用户反馈SDK 提供更为底层的接口供开发者定制。 
### 数据模型

用户反馈中开发者和用户之间的对话通过会话 (`com.umeng.fb.model.Conversation`) 类来实现。 用户或者开发者的每一条回复是`Reply`， 用户的回复为`UserReply`, 开发者的回复为`DevReply`。 
我们建议在客户端使用单会话模式. 使用`FeedbackAgent.getDefaultConversation(); `函数获取默认的`Conversation`。 

然后开发者可以向`Conversation`中添加用户回复(`UserReply`) `con.addUserReply("user reply content here")`;

`Conversation`会自动保存在本地`SharedPreferences` 当中。可以通过`Conversation.sync()`方法将本地内容和服务器同步。 这是一个异步方法。 传入参数`SyncListener`. 实现回调接口， 从而当网络操作完成时， 可以更新UI中的元素。 

更详细的使用方法请参考SDK 提供的UI实现代码： ConversationActivity 和ContactActivity.源代码在：https://github.com/umeng/umeng-android-sdk-theme/tree/master/fb/v4.3

如果SDK 中提供的默认UI 不能满足您的需求， 您可以参考SDK 中的UI源代码利用SDK提供的数据API实现自己的界面自定义需求。 


### 设置用户信息

可以使用
```java
					UserInfo info = agent.getUserInfo();
					if (info == null)
						info = new UserInfo();
					Map<String, String> contact = info.getContact();
					if (contact == null)
						contact = new HashMap<String, String>();
					String contact_info = contactInfoEdit.getEditableText()
							.toString();
					contact.put(KEY_UMENG_CONTACT_INFO_PLAIN_TEXT, contact_info);
					
					// optional, setting user gender information.
					//contact.put("gender", "male");
					//contact.put("gender", "female");

					//optional, setting user age group information
					//contact.put("age_group", 1);
					
					info.setContact(contact);
					agent.setUserInfo(info);
```

age_group 参考。 有效值1-8. 
```xml
<string-array name="age_group_list">
		<item>Age</item>
		<item>&lt;18</item>
		<item>18~24</item>
		<item>25~30</item>
		<item>31~35</item>
		<item>36~40</item>
		<item>41~50</item>
		<item>51~59</item>
		<item>>=60</item>
	</string-array>
```

详细请参考示例 https://github.com/umeng/umeng-android-sdk-theme/blob/master/fb/v4.3/src/com/umeng/fb/ContactActivity.java. 

注意: 

> 联系人信息需要在第一次发送反馈信息时一起同步到服务器端， 不能单独同步。 使用函数`conversation.addUserReply(content); ` 添加一条用户反馈内容， 然后`sync()`同步。 


 

## FAQ
* sync 的时候报错， `java.lang.NoClassDefFoundError: android.support.v4.app.NotificationCompat$Builder`, 如何解决?

    > Log 如下:
```log
    05-30 14:25:24.175: E/AndroidRuntime(21578): java.lang.NoClassDefFoundError: android.support.v4.app.NotificationCompat$Builder
05-30 14:25:24.175: E/AndroidRuntime(21578): 	at com.umeng.fb.FeedbackAgent.a(FeedbackAgent.java:115)
05-30 14:25:24.175: E/AndroidRuntime(21578): 	at com.umeng.fb.FeedbackAgent.a(FeedbackAgent.java:21)
```
这是support library 的一个bug, 详情请参考: https://code.google.com/p/android/issues/detail?id=36502. 
解决方案: 更新android.support.v4.jar 到最新版本。 在Eclipse 中， 鼠标右击工程， `Android Tools` -> `Add Support Library`， 或者手动从官方网站下载最新的jar放到工程的libs目录下。 

* 混淆过程中遇到的问题,具体请见[这里](http://dev.umeng.com/analytics/reference/faq#proguard).



## 技术支持
论坛: [友盟开发者社区](http://bbs.umeng.com/forum-feedback-1.html)

QQ:   800083942

Email：support@umeng.com

为了能够尽快响应您的反馈，请提供您的appkey及logcat中的详细出错日志，您所提供的内容越详细越有助于我们帮您解决问题。








# framework
Xcode7打包Framework


一、打包Framework

1、新建iOS->Framework & Library->Cocoa Touch Framework

2、选择next进入下一步

3、在PROJECT->Deployment Target->iOS Deployment Target选择你需要支持的最低系统。

同样的操作在TARGETS中，Deployment Info->Deployment Target

4、由于我的framework需要支持iOS7，所以在第2、3步中进行了相应的设置。Build时会发现有️ld: warning: embedded dylibs/frameworks only run on iOS 8 or later警告，这是因为工程默认编译设置的是Dynamic Framework。这种编译只有在iOS8以后才能使用。

5、针对第4步中所出现的问题，根据需求我的工程不需要使用动态framework，以使用其动态更新的功能。动态库可以分开发布，在运行时查找并存入内存，但苹果只允许他自己用，到iOS8以后才开放给开发者。因此，我需要将Dynamic Framework更换为Static Library静态模式。设置路径为Build Settings->Linking->Mach-O Type->Static Library

6、这里要注意，在编译时，不要将图片文件放在工程里面，否则编译后framework中会出现大量的零散图片文件在里面。这时需要将图片等资源放在.bundle文件中。图片的打包在第二步介绍。

7、这样打包的framework依然有问题，如果你用了Category，别人在用你的framework时会发生崩溃。这时别人在引用时需要在工程中other linker flags中添加-objC如果依然有问题，再添加-all_load。

8、终于编译成功，但发现很多关于符号表的警告，这时需要将Generate Debug Symbols设置为NO即可关闭符号表警告。

9、但是我需要支持bitcode，以上设置后并不能使framework支持bitcode,因此还需要进行额外的设置一个命令让其支持bit code。在TAGETS的Build setting中搜索Other C Flags，添加命令“-fembed-bitcode”。同样的设置在PROJECT中。如果不进行以上操作。别人在集成你的framework时可以编译，亦可以真机测试。唯独在打包时会发出警告并打包失败。警告为framework不支持bitcode！

10、无论SDK还是Framework都需要暴露公共的头文件以供使用者读取和。在TARGETS->Build Phases->Headers里面，有三种类别。Public(公共的)，这里存放供其他人查看的header。Private(私有的)这里存放私有的Header，以上两个Headers存放位置都会暴露出来，所有人可以查看。有些Header是不想给别人看到的。这种header放在第三个类Project中。

11、打包。Edit Scheme->Build Configuration->选为Release然后Run即可.

二、打包bundle文件

1、新建OS X->Framework & Library->Bundle新建

2、在Build Settings->(null)-Deployment->iOS Deployment Target->选择自己需要支持的最低系统。

3、build后会生成一个bundle包，但在包中的图片由以前的png格式全部变成tiff格式。为了防止这种格式转变。需要在Build Settings->Architectures->Base SDK->选择iOS的SDK要支持的版本。这时TARGETS中Build Setting->User-Defined中会出现一个新的Key：COMBINE_HIDPI_DEBUG_INFO,把它设置为NO。

4、这样创建的图片资源不能使用[UIImage imageNamed:@“png”]来获取了。需要使用路径方式来读取图片。

这里我使用了一个函数来获取路径。

NSString *getKaYiKaImageBundlePath(NSString *filename);

NSString *getKaYiKaImageBundlePath(NSString *filename) {

NSBundle *libBundle = [NSBundle bundleWithPath:[[[NSBundle mainBundle] resourcePath] stringByAppendingPathComponent:@"KaYiKa.bundle"]];

if (libBundle && filename) {

NSString *path = [[libBundle resourcePath] stringByAppendingPathComponent:filename];

path = [path stringByAppendingString:@".png"];

return path;

}

return nil;

}

使用时直接用

[UIImage imageWithContentsOfFile:getKaYiKaImageBundlePath(@"tool_return_day")]获取图片。

三、创建引用工程

创建引用工程时将framework和bundle同时导入。

剩下的使用与系统framework相同。

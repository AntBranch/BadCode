# 我从烂代码中学会了什么

### 目录
   - [不要再面向字典开发了] (#不要再面向字典开发了)
   - [谁的工作谁干] (#谁的工作谁干)
   - [第三方库入侵太深] (#第三方库入侵太深)
   - [用原生控件] (#用原生控件)
   - [主色调] (#主色调)
   - [源文件分类存放] (#源文件分类存放)
   - [类名前加上前缀] (#类名前加上前缀)
   - [方法调用短一点] (#方法调用短一点)
   - [在类头文件中尽量少引入其他头文件] (#在类头文件中尽量少引入其他头文件)
   - [多用类型常量,少用#define预处理指令] (#多用类型常量,少用#define预处理指令)
   - [重写getter setter方法] (#重写getter setter方法)
   
  
### 不要再面向字典开发了
现在移动开发几乎是零门槛。注意几乎是零门槛，不是没门槛。如果你是个移动开发者如果看到这个样的代码会怎么样？
``` objc
_idea = [responseObject objectForKey:@"idea"];
_isOwner = [[responseObject objectForKey:@"is_owner"] boolValue];
_hasFollowed = [[responseObject objectForKey:@"has_followed"] boolValue];
_commentsCount = [[[responseObject objectForKey:@"idea"] objectForKey:@"comment_count"] integerValue];
NSDictionary *owner = [_idea objectForKey:@"owner"];
NSURL *coverUrl = [NSURL URLWithString:[_idea valueForKey:@"medium_cover_url"]];
```
面向字典开始真的很痛苦，面向模型开发吧。提高代码的可读性和易维护性。在字典转模型的时可以使用明杰老师的[MJExtension](https://github.com/CoderMJLee/MJExtension)会大大加快效率。

### 谁的工作谁干
不要有一万行代码都写在ViewController里，谁的工作谁干。搞个MVC。
``` objc
AppDelegate *appDelegate = (AppDelegate *)[[UIApplication sharedApplication] delegate];
NSDictionary *owner = [_idea objectForKey:@"owner"];
if(indexPath.row == 0){
    IdeaProfileTableViewCell *cell = [[IdeaProfileTableViewCell alloc] init];
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
    NSURL *coverUrl = [NSURL URLWithString:[_idea valueForKey:@"medium_cover_url"]];
    [cell.cover sd_setImageWithURL:coverUrl placeholderImage:[UIImage imageNamed:@"medium_idea_cover"]];
    cell.cover.userInteractionEnabled = YES;
    UITapGestureRecognizer *showCover = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(zoom:)];
    [cell.cover addGestureRecognizer:showCover];
    
    cell.title.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"title"]];
    [cell.title sizeToFit];
    if (cell.title.frame.size.width + 20> appDelegate.rootController.view.frame.size.width-140-25) {
        cell.title.frame = CGRectMake(140, 20, appDelegate.rootController.view.frame.size.width-140-25 -20, 40);
        [cell.qrCode setFrame:CGRectMake(cell.title.frame.origin.x +cell.title.frame.size.width, cell.title.frame.origin.y+2, 16, 16)];
    }else {
        cell.title.frame = CGRectMake(cell.title.frame.origin.x, cell.title.frame.origin.y, cell.title.frame.size.width + 20, cell.title.frame.size.height+2);
        [cell.qrCode setFrame:CGRectMake(cell.title.frame.origin.x +cell.title.frame.size.width, cell.title.frame.origin.y+2, 16, 16)];
    }
    [cell.qrCode setUserInteractionEnabled:YES];
    UITapGestureRecognizer *showQRCode = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(showQR:)];
    [cell.qrCode addGestureRecognizer:showQRCode];
    
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"yyyy-MM-dd'T'HH:mm:ssZ"];
    NSDate *date = [dateFormatter dateFromString:[_idea objectForKey:@"created_at"]];
    cell.timeAgo.text = [NSString stringWithFormat:@"%@ 发布", [date timeAgo]];
    
    if ([owner objectForKey:@"name"]) {
        NSMutableAttributedString *founder = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"创始人：%@", [owner objectForKey:@"name"]]];
        [founder addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 3)];
        [founder addAttribute:NSForegroundColorAttributeName value:TableColor range:NSMakeRange(4, [founder length] - 4)];
        cell.founder.attributedText = founder;
        cell.founder.userInteractionEnabled = YES;
        UserDataTapGestureRecognizer *showFounder = [[UserDataTapGestureRecognizer alloc] initWithTarget:self action:@selector(showOwner:)];
        showFounder.userData = [[owner objectForKey:@"server_id"] stringValue];
        showFounder.numberOfTapsRequired = 1;
        [cell.founder addGestureRecognizer:showFounder];
    }
    
    if ([[owner objectForKey:@"roles"] count] > 1) {
        NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：%@等", [[owner objectForKey:@"roles"] firstObject]]];
        [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
        [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
        cell.role.attributedText = roles;
    }else if ([[owner objectForKey:@"roles"] count] == 1) {
        NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：%@", [[owner objectForKey:@"roles"] firstObject]]];
        [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
        [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
        cell.role.attributedText = roles;
    }
    else if ([[owner objectForKey:@"roles"] count] == 0) {
        NSMutableAttributedString *roles = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"角色定位：未填写"]];
        [roles addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
        [roles addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [roles length] - 5)];
        cell.role.attributedText = roles;
    }
    
    if ([[_idea objectForKey:@"investment"] isEqual:[NSNull null]]) {
        NSMutableAttributedString *investment = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"可投入资金：未填写"]];
        [investment addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 5)];
        [investment addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(6, [investment length] - 6)];
        cell.money.attributedText = investment;
    }else if ([_idea objectForKey:@"investment"]) {
        NSMutableAttributedString *investment = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"可投入资金：%@", [_idea objectForKey:@"investment"]]];
        [investment addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 5)];
        [investment addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(6, [investment length] - 6)];
        cell.money.attributedText = investment;
    }
    
    if ([[owner objectForKey:@"experience"] isEqual:[NSNull null]]) {
        NSMutableAttributedString *experience = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"行业经验：未填写"]];
        [experience addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
        [experience addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [experience length] - 5)];
        cell.experience.attributedText = experience;
    }else if ([owner objectForKey:@"experience"]) {
        NSMutableAttributedString *experience = [[NSMutableAttributedString alloc] initWithString:[NSString stringWithFormat:@"行业经验：%@", [owner objectForKey:@"experience"]]];
        [experience addAttribute:NSForegroundColorAttributeName value:SummaryColor range:NSMakeRange(0, 4)];
        [experience addAttribute:NSForegroundColorAttributeName value:SharingColor range:NSMakeRange(5, [experience length] - 5)];
        cell.experience.attributedText = experience;
    }
    [cell.founder sizeToFit];
    [cell.role sizeToFit];
    [cell.money sizeToFit];
    cell.followed.userInteractionEnabled = YES;
    cell.followMount.userInteractionEnabled = YES;
    cell.commentMount.userInteractionEnabled = YES;
    cell.comment.userInteractionEnabled = YES;
    UITapGestureRecognizer *showFollowers = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(viewFollowers:)];
    [cell.followed addGestureRecognizer:showFollowers];
    [cell.followMount addGestureRecognizer:showFollowers];
    UITapGestureRecognizer *showComments = [[UITapGestureRecognizer alloc]initWithTarget:self action:@selector(pushToCommentView)];
    [cell.comment addGestureRecognizer:showComments];
    [cell.commentMount addGestureRecognizer:showComments];
    cell.followMount.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"followers_count"]];
    cell.commentMount.text = [NSString stringWithFormat:@"%@", [_idea objectForKey:@"comment_count"]];
    if ([[_idea objectForKey:@"recommend"] boolValue]) {
        cell.recommend.image = [UIImage imageNamed:@"ideaRecommend"];
    }
    else {
        cell.recommend.image = nil;
    }
    cell.selectionStyle = UITableViewCellSelectionStyleNone;
    cell.backgroundColor = [UIColor colorWithHexString:@"F7F7F7"];
    return cell;
}
```
很抱歉让大家看了一个这么长的代码。这是一个哥们的cell，我想告诉你这个TableView里边有10个自定义的cell，你可以想象.......此处省略10000字。用MVC模式之后
``` objc
if (indexPath.row == 0) {
    UserMainInfoCell *cell = [[UserMainInfoCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"cellOne"];
    cell.userInfoModel = self.userInfoModel;
    return cell;
}
```
ViewController你把Model给我，剩下的事情和你没关系。

MVC其实是iOS开发中最基本的模式，关于ViewController里面的tableview除了像上面这样做，其实还可以把tableview的datasource和delegate拆离出来，在MVC上加一层TableviewProtocol层。

使用如下：
``` objc
//ViewController.m

demoTableView=[[UITableView alloc] initWithFrame:CGRectMake(70, 0,ScreenWidth-70 , ScreenHeight-64-48) style:UITableViewStylePlain];
[self.view addSubview:demoTableView];
demoTableView.rowHeight=90;
demoDataSource=[[DemoDataSource alloc] init];
demoDataSource.listData=@[@"23",@"23"];
demoTableView.dataSource=demoDataSource;
    
demoDelegate=[[DemoDelegate alloc] init];
demoTableView.delegate=demoDelegate;
```

``` objc
//DemoDataSource.h

@interface DemoDataSource : NSObject<UITableViewDataSource>
@property(nonatomic,strong)NSArray *listData;// datasource

@end
```

### 第三方库入侵太深
我举一个例子AFNetworking，这个可能是每一个开发者网络请求的标配，机会每一个页面都要用到。这样就造成了AFNetworking入侵太深，最近AFNetworking升级到了3.0，很多类改了方法也改了，你一看自己的项目300个地方用到了AFNetworking怎么办。全局替换？？像这样的库最好进行二次封装。
``` objc
-(void)GET:(NSString*)url withParameters:(NSDictionary*)parameters success:(void (^)(NSDictionary *data))success failure:(void (^)(NSError *error,AFHTTPRequestOperation *operation))failure {
    AFHTTPRequestOperationManager *manager = [AFHTTPRequestOperationManager manager];
    [manager GET:url parameters:parameters success:^(AFHTTPRequestOperation *operation, id responseObject) {
        if (success) {
            success((NSDictionary*)responseObject);
        }
    } failure:^(AFHTTPRequestOperation *operation, NSError *error) {
        if (failure) {
            failure(error,operation);
        }
    }];
}
```
封装之后使用，如果AFNetworking更新了你就改此处即可。

### 用原生控件
如果没有特殊需求，尽量用苹果提供的原生控件。有个哥们的TabBarViewController不知道为啥要用。
``` objc
@interface RDVTabBarController : UIViewController <RDVTabBarDelegate>
```
没有特殊需求不要用第三方的东西，而且实现的功能是一样的。

### 主色调
开发之前确立好App的主色调不要每次用到颜色就用这个东西
``` objc
self.view.backgroundColor = [UIColor colorWithHexString:@"878787"];
titleLabel.backgroundColor = [UIColor colorWithRed:0.95f green:0.48f blue:0.00f alpha:1.00f];
```
确定颜色之后定义成宏。
``` objc
self.view.backgroundColor = DLBackgroundColor;
titleLabel.backgroundColor = DLBackgroundColor;
```

当然项目如果庞大 用宏可能会增加编译时间 ，其实可以 写出UIColor的category，需要的时候使用就更加方便了。
``` objc
@implementation UIColor (ThemeColors)

+ (instancetype)kColorWhite {
    return [UIColor whiteColor];
}

+ (instancetype)kColorBlue {
    return [UIColor blueColor];
}
```
 
### 源文件分类存放
不要把所有源文件堆放在同一个目录下。
小型应用，根据文件的功能用途，按照应用架构（Model、View、Controller等），把源文件放入不同路径分开存放。
中型、大型应用，按照功能的不同，区分开不同模块和子模块，并放入不同路径分开存放。在基层模块中，可以按照架构来分类存放文件。

### 类名前加上前缀
Objective-C是不支持命名空间的，所以同一个应用内不得出现名称相同的两个类。
给类名前面加上两三个大写字母作为前缀是一个好习惯。尤其是打算开源供他人使用的库。
比如可以取名`SKImageCache`，而不是`ImageCache`。

### 方法调用短一点
Cocoa和很多第三方库的API都非常长，而你的屏幕就那么宽，所以别在一行里把代码写得太长。
比如：SDWebImage
``` objc
[backgroundImageView sd_setImageWithURL:[NSURL URLWithString:@"http://xxxx.xxx.com/xxx.png"] placeholderImage:[UIImage imageNamed:@"backgroundTemp"] options:SDWebImageContinueInBackground completed:^(UIImage *image, NSError *error, SDImageCacheType cacheType, NSURL *imageURL) {
    // next step
}];
```
解决办法是把实参从『方法调用』中解放出来，并用换行来分割各个参数：
``` objc
NSURL *url = [NSURL URLWithString:@"http://xxxx.xxx.com/xxx.png"];
UIImage *image = [UIImage imageNamed:@"backgroundTemp"];
    
[backgroundImageView sd_setImageWithURL:url
                       placeholderImage:image
                                options:SDWebImageContinueInBackground
                              completed:^(UIImage *image,
                                          NSError *error,
                                          SDImageCacheType cacheType,
                                          NSURL *imageURL) {
                                  // next step
}];
```
写成这样是不是好多了呢？

### 在类头文件中尽量少引入其他头文件
- 除非有必要，否则不要引入头文件。一般来说，应在某个类的头文件中使用 向前声明（`@class PersonModel;`）来提及别的类，并在实现文件中引入那些类的头文件。这样做可以尽量降低类之间的耦合（couping)。
- 有时无法使用 向前声明（`@class PersonModel;`），比如要声明某个类遵循意向协议。 这种情况下，尽量把 “该类遵循某协议” 的这条声明 移至 “分类Category” 中。如果不行的话， 就把协议单独放在一个头文件中，然后将其引入。


### 多用类型常量，少用#define预处理指令
**少用** : #define ANIMATION_DURATION 0.3

**最好用** :
```objc
// 变量一定要同时用static 与 const 来声明。只在.m使用
static const NSTimeInterval YXAnimationDuration = 0.3;
```


- 若需要对外公布使用

例1 ：

```objc
// View.h
extern const NSTimeInterval YXAnimationDuration;
// View.m
const NSTimeInterval YXAnimationDuration = 0.3;
```

例2：

```objc
// View.h
extern NSString *const YXStringConstant;
// View.m
NSString *const YXStringConstant = @"VALUE”;
```

**要点**

- 不要使用预处理指令定义常量。这样定义 出来的常量不含类型信息，编译器只是会在编译前据此执行查找与替换操作。即使有人重新定义了常量值，编译器也不会产生警告信息，这将导致应用程序中的常量值不一致。

- 在实现文件中使用 `static const` 来定义“只在编译单元内可见的常量（`translation-unit-specific constant`）”。 由于此类常量不在全局常量表中，所以无需为其名称加前缀。
- 在头文件中使用`extern` 来声明全局常量，并在相关实现文件中定义其值。 这种常量要出现在全局符号表中，所以其名称应加以区隔，通常用 和他相关的类名做前缀。



### 重写getter setter方法

- setter
有些哥们儿不太喜欢重写setter方法，而是单独再提供一个api来更新数据。事实上，我们通过重写setter方法，可以给我们带来很大的便利。

举例:

```objc
- (void)setSelectedHospitalModel:(HDFJiaHaoConsultHistoryModel *)selectedHospitalModel {
  if (_selectedHospitalModel != selectedHospitalModel) {
    _selectedHospitalModel = selectedHospitalModel;
    self.hospitalTextField.text = selectedHospitalModel.hospitalName;
    self.departmentTextField.text = selectedHospitalModel.hospitalFacultyName;
  }
}
// 更新数据显示,只需要调用 set方法即可
self.selectedHospitalModel = selectedModel;
```



- getter
尽量不要使用_name这种类型的调用，而是声明为属性，直接使用self.name这样的写法。声明为属性，我们可以重写getter方法，而且就是所谓的lazy loading。

举例:

```objc
- (NSMutableArray *)yearSources {
  if (_yearSources == nil) {
    _yearSources = [[NSMutableArray alloc] init];
    int curYear = (int)[[NSDate date] hdf_year];
    for (int year = 1970; year <= curYear + 100; ++year) {
      [_yearSources addObject:@(year)];
    }
  }
  return _yearSources;
}
```














# 1. 什么是单元测试
在面向对象语言中，一个单元就是一个方法. 通过测试代码对一个方法进行正确性验证就叫单元测试

> google对测试的分类：

表头|小型测试|中型测试|大型测试
---|:--:|:---:|:---:
测试类型|单元测试|逻辑层测试|UI测试/接口测试
测试环境|mock|mock/真实|真实
网络访问|否|localhost|是

> 测试金字塔对测试的分类：最底层是单元测试，然后是业务逻辑测试，最后是端到端的测试（GUI或CLI）。—《Succeeding With Agile》

![avatar](./asset/test-pyramid.png)

单元测试依赖少，耗时底，解决问题成本低。
大型集成测试是在真实环境测试的，更能保证整个应用的可靠性，但这个阶段解决问题成本相对比较高。
小型测试是是大型测试的根基，各种测试阶段是相辅相成，而非互斥。

### 为什么要写单元测试？
- 提前发现代码问题：边界条件难以手工验证，代码可能有逻辑错误
- 保证方法的正确性：为重构打好基础
- 优化代码设计


# 2. iOS单元测试框架使用 (XCTest + OCMock)
> XCTest相关tips
- 单测类继承自XCTestCase类
- 单测方法以test开头
- setUp/tearDown有类方法与实例方法
- 单测文件命名与目录建议仿照业务代码
- 单测执行顺序与编码前后无关，每个单测都是相互独立的.
- Xcode集成，简单易用，学习成本低
- 支持同步/异步流程测试
- 支持单元测试，性能测试，UI测试
- 提供丰富的Assert方法

> 添加单元测试
- 创建项目点的时候，勾选添加单元测试.
- 手动创建单元测试target，添加单元测试文件.

> 常见的断言
    
    - XCTFail
    - XCTAssert
    - XCTAssertTrue
    - XCTAssertEqual
    - XCTAssertEqualObjects
    - XCTAssertEqualWithAccuracy
    - XCTAssertNil
    - XCTAssertNotNil
    - XCTAssertThrows

> XCTest demo

```objc
// 被测方法
+ (NSString *)descForCount:(NSInteger)count;
// 单元测试
- (void) testDescForCount {
    // arragne
    NSInteger count = 0;
    // action
    NSString *desc = [CDataUtil descForCount:count];
    // assert
    XCTAssertEqualObjects(desc, @"0");
}
```

> OCMock
- 支持stub并添加自定义行为
- 支持mock class/protocol/observer
- 支持参数验证、延迟验证

```objc
// 对于依赖网络或者第三方接口的函数，可以用扩展方法把它暴露出来，然后用mock替换掉.
@interface CDataUtil ()
+ (NSInteger)countThreshold;
@end


@interface CDataUtilTest2: XCTestCase
@property(nonatomic, strong) id mock;
@end

@implementation CDataUtilTest2
- (void)setUp {
    _mock = OCMClassMock([CDataUtil class]);
    OCMStub([_mock countThreshold]).andReturn(0);
}

- (void)tearDown {
    [_mock stopMocking];
}

- (void)testDescForCount {
    NSInteger count = 100;
    NSString *desc = [CDataUtil descForCount2:count];
    XCTAssertEqualObjects(desc, @"100");
}
@end
```
- *这里也可以把内部的依赖提取出来作为参数传入，这样也就不需要mock了.



# 3. 单元测试用例设计
1. 步骤
    + Arrange：准备好所需要的外部环境，数据、mock等.
    + Action：调用需要测试的方法或流程
    + Assert：判断结果是否符合预期

1. 经验
    + 运行
        - 先跑起来看看环境是否正常
    + 正反面测试
        - 对于数字的参数，正面可以是正数，负面则为负数
    + 特性测试
        - 临界值
    + 完善代码覆盖率
        - 编辑Edit scheme, 勾选Code Coverage跟target，构建结构就回展示详细的代码覆盖率。以及在Xcode编辑页面有右上角勾选Code Coverage就可以看到被覆盖的绿色部分跟没有被覆盖的红色部分.

1. 基本原则
    + 基于意图而不是实现
    + 简单、清晰、易懂 (包括函数名，函数体)
    + 避免引入条件判断，循环等
    + 测试用例完备而不重复
    + 使用Assert进行验证



# 4. 单元测试案例分析
> UI代码写单元测试？: 
- 纯UI代码不需要写单元测试
- 分离数据与ui代码，对数据部分编写单元测试
- 复杂代码需要进行合理的拆分.
- 通过单测优化代码架构

> 异步代码写单元测试？:
- 确定是否需要验证异步逻辑
- 单元测试是同步执行的，对于异步代码需要使用Expectation，并设置合理的超时时间，到达时间waitForExpectations会触发超时错误. 

```objc
// 有效的单元测试，通过XCTestExpectation验证异步代码
- (void)testAsyncFetchCount2 {
    // 添加预期
    XCTestExpectation *expect = [self expectationWithDescription:@"asyncFetchCount"];

    [CDemoUtil asyncFetchCountCompletion:^(NSInteger count) {
        XCTAssertEqual(count, 0);
        // 通知执行完成
        [expect fulfill];
    }];
    // 开始等待并设置超时时间
    [self waitForExpectations:@[expect] timeout:1];
}
```
> OCMock常用功能
- 测试方法是否执行/未执行：使用OCMExpect/OCMReject添加预期，然后执行代码，最后用OCMVerifyAll验证预期.
- 测试方法的参数：使用OCMArg.checkWithBlock对参数进行验证.
- 测试方法的执行顺序：setExpectationOrderMatter：YES, 之后会按照添加预期的顺序，验证执行顺序
- 忽略参数类型：OCMArg.any（自定义类型），ignoringNonObjectArgs（基本类型）
- 验证异步逻辑：OCMVerifyAllWithDelay设置超时时间

>#### 提升代码的可测性
- 明确测试用例是否需要Mock
- 思考从diam设计上能否避免使用Mock
- 使用依赖注入等方法提升代码的可测试性：如果代码需要大量的mock才能写好单测，那么这块代码的外部依赖是比较多的。在使用mock之前应该相信是不是可以优化代码结构来提升代码的可测试性。
以前面的countThreshold为例：
```objc
// 将threshold作为参数传入，避免内部产生依赖
+ (NSString *)descForCount2:(NSInteger)count withThreshold:(NSInteger)threshold;
```

> 小结
- 逻辑层代码都需要单测
- 保证没问题可以不写，但除了文档一定要补充单元测试
- 重构之前先补充单元测试
- 发现不好写单测的情况及时优化diam结构
- 合理使用Mock辅助编写单元测试

# 5. 单元测试自动化
做好自动化才能更好的发挥单元测试的优势, 真正做到通过单元测试来保证代码质量. xcodebuild提供了test命令，执行完毕之后会生成结果文件以及覆盖率文件，通过解析结果文件可以获取单测的成功数、失败数、覆盖率等详细数据。
以下是搭建自动化流程需要关注的点：

- 单元测试用例个数、失败个数、执行时长监控
- 全量代码覆盖率、各模块代码覆盖率监控
- MR增量代码覆盖率检测
- 版本增量代码覆盖率检测
- 个人增量代码覆盖率检测


```shell
# 执行单元测试
xcodebuild test \
-workspace Xxx.xcworkspace (or xx.project file) \
-scheme Xxx \
-derivedDataPath "build/" \
-destination "platform=XXX,OS=XX.X,name=xx" \
-resultBundlePath "result/" \
-resultBundleVersion x

# 获取覆盖率文件
xcrun xccov view --report --json path/to/xcresult
```

> 单元测试自动化监控小结
- 用例数合代码覆盖率是单元测试水平的重要体现.
- 不应盲目追求用例数和代码覆盖率，单测的目的是发现问题保证质量.
- 有数据度量才能推动整体单元测试水平不断提升.


# 6. demo
```objc
#import <XCTest/XCTest.h>
#import "OCMock.h"
#import "UTDataUtil.h"

@interface UTDataUtil ()
+ (NSInteger)minThreshold;
+ (NSInteger)maxThreshold;

+ (void)handleLoadSuccessWithInfo:(NSDictionary *)info;
+ (void)handleLoadFailWithInfo:(NSDictionary *)info;
+ (void)showError:(BOOL)show;
@end

@interface UTDataUtilTest : XCTestCase
@property(nonatomic, strong) id mock;
@end

@implementation UTDataUtilTest

- (void)setUp {
    _mock = OCMClassMock([UTDataUtil class]);
    OCMStub([_mock minThreshold]).andReturn(10);
    OCMStub([_mock maxThreshold]).andReturn(100);
}

- (void)tearDown {
    [_mock stopMocking];
}

/// 将数字转化为字符串，验证转换逻辑是否正确
/// @discussion 大于等于10万时，展示xx万，不带小数点
/// @discussion 大于等于1万时，展示1.x万，保留一位小数点
/// @discussion 低于1万时，展示实际数字
- (void)testDescForCount {
    {
        NSInteger count = 100000;
        NSString *desc = [UTDataUtil descForCount:count];
        XCTAssertEqualObjects(desc, @"10万");
    }
    {
        NSInteger count = 10000;
        NSString *desc = [UTDataUtil descForCount:count];
        XCTAssertEqualObjects(desc, @"1.0万");
    }
    {
        NSInteger count = 1;
        NSString *desc = [UTDataUtil descForCount:count];
        XCTAssertEqualObjects(desc, @"1");
    }
}

- (void)testAsyncFetchCount {
    XCTestExpectation *expect = [self expectationWithDescription:@"asyncFetchCount"];

    [UTDataUtil asyncFetchCountCompletion:^(NSInteger count) {
        XCTAssertEqual(count, 1);
        [expect fulfill];
    }];
    [self waitForExpectations:@[expect] timeout:0.5];
}

/// 获取处理后的数字，验证数字处理逻辑是否正确
/// @discussion 大于阈值时，返回 原始数字 + 阈值
/// @discussion 小于阈值时，返回 0
/// @discussion 否则，返回原始数字
- (void)testProcessedCount {
    {
        NSInteger origCount = 9;
        NSInteger result = [UTDataUtil processedCount:origCount];
        XCTAssertEqual(result, 0);
    }
    {
        NSInteger origCount = 101;
        NSInteger result = [UTDataUtil processedCount:origCount];
        XCTAssertEqual(result, 201);
    }
    {
        NSInteger origCount = 50;
        NSInteger result = [UTDataUtil processedCount:origCount];
        XCTAssertEqual(result, 50);
    }
}

///// 使用OCMock验证代码是否执行
- (void)testHandleLoadFinished {
    {
        NSDictionary *info = @{@"testKey": @"data"};
        id mock = OCMClassMock([UTDataUtil class]);

        OCMExpect([mock handleLoadSuccessWithInfo:[OCMArg any]]);
        OCMReject([mock handleLoadFailWithInfo:[OCMArg any]]);

        [UTDataUtil handleLoadFinished:info];

        OCMVerifyAll(mock);
        [mock stopMocking];
    }
    {
        NSDictionary *info = @{};
        id mock = OCMClassMock([UTDataUtil class]);

        OCMExpect([mock handleLoadFailWithInfo:[OCMArg any]]);
        OCMReject([mock handleLoadSuccessWithInfo:[OCMArg any]]);

        [UTDataUtil handleLoadFinished:info];

        OCMVerifyAll(mock);
        [mock stopMocking];
    }
}
@end
```
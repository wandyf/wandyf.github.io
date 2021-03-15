---
layout: post
title: "AFNetworking"
# background-image: ""
date: 2021-2-8
published: true
category: iOS
tags:
        - AFNetworking
---

## AFURLSessionManager/AFHTTPSessionManager解析
### AFN结构

- AFNetworking
    - 核心
        - AFURLSessionManager
        - AFHTTPSessionManager
    - 序列化
        - AFURLRequestSerialization
            - AFHTTPSerializer
            - AFJSONRequestSerializer
            - AFPropertyRequestSerializer
        - AFURLResponseSerialization
            - AFHTTPResponseSerializer
            - AFJSONResponseSerializer
            - AFXMLParserResponseSerializer
            - AFXMLDocumentResponseSerializer(macOS)
            - AFPropertyListResponseSrializer
            - AFImageResponseSerializer
            - AFCompoundResponseSerializer
    - 辅助类
        - AFSecurityPolicy
        - AFNetworkReachabilityManager
    - UI类
        - ...

### AFHTTPSessionManager的初始化

*注意此方法是工厂方法，而不是单例，只看外层写法容易误解*  
`AFHTTPSessionManager *manager = [AFHTTPSessionManager manager];`

### AFHTTPSessionManager初始化的内部流程
 
- 1.层层调用至父类(AFURLSessionManager)的初始化方法  
    `- (instancetype)initWithSessionConfiguration:(NSURLSessionConfiguration *)configuration;`
    - 获取或创建默认configuration并持有
    ```ObjC
    if (!configuration) {
        configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
    }
    self.sessionConfiguration = configuration;
    ```
    - 初始化串行队列(serial queue，苹果文档要求，为了确保回调顺序正确)，用于生成session
    ```ObjC
    self.operationQueue = [[NSOperationQueue alloc] init];
    self.operationQueue.maxConcurrentOperationCount = 1;
    ```
    - 初始化session并持有,session会产生不止一个task
    ```ObjC
    self.session = [NSURLSession sessionWithConfiguration:self.sessionConfiguration delegate:self delegateQueue:self.operationQueue];
    ```
    - 设置解析(默认JSON)，证书(默认无条件信任https)，网络状态监听
    ```ObjC
    self.responseSerializer = [AFJSONResponseSerializer serializer];
    self.securityPolicy = [AFSecurityPolicy defaultPolicy];
    self.reachabilityManager = [AFNetworkReachabilityManager sharedManager];
    ```
    - 初始化一个NSMutableDictionary,用于存储绑定task和delegate(@{key:delegate]})
    ```ObjC
    self.mutableTaskDelegatesKeyedByTaskIdentifier = [[NSMutableDictionary alloc] init];
    ```
    - 初始化线程锁
    ```ObjC
    self.lock = [[NSLock alloc] init];
    self.lock.name = AFURLSessionManagerLockName;
    ```
    - 异步获取当前session所有未完成task(如从后台回来后初始化session，可能会有之前的任务)
    ```ObjC
    [self.session getTasksWithCompletionHandler:^(NSArray *dataTasks, NSArray *uploadTasks, NSArray *downloadTasks) {
        for (NSURLSessionDataTask *task in dataTasks) {
            [self addDelegateForDataTask:task uploadProgress:nil downloadProgress:nil completionHandler:nil];
        }

        for (NSURLSessionUploadTask *uploadTask in uploadTasks) {
            [self addDelegateForUploadTask:uploadTask progress:nil completionHandler:nil];
        }

        for (NSURLSessionDownloadTask *downloadTask in downloadTasks) {
            [self addDelegateForDownloadTask:downloadTask progress:nil destination:nil completionHandler:nil];
        }
    }];
    ```
- 2.处理url，给url拼接"/"
  ```ObjC
  if ([[url path] length] > 0 && ![[url absoluteString] hasSuffix:@"/"]) {
        url = [url URLByAppendingPathComponent:@""];
    }
  ```
- 3.持有baseURL
  ```ObjC
  self.baseURL = url;
  ```
- 4.设置序列化默认值
    ```ObjC
    self.requestSerializer = [AFHTTPRequestSerializer serializer];
    self.responseSerializer = [AFJSONResponseSerializer serializer];
    ```

### 发起请求,生成NSURLSessionDataTask

- 生成request
  ```ObjC
  NSMutableURLRequest *request = [self.requestSerializer requestWithMethod:method URLString:[[NSURL URLWithString:URLString relativeToURL:self.baseURL] absoluteString] parameters:parameters error:&serializationError];
  ```
  - 生成NSMutableURLRequest并设置请求方式
    ```ObjC   
    NSMutableURLRequest *mutableRequest = [[NSMutableURLRequest alloc] initWithURL:url];
    mutableRequest.HTTPMethod = method;
    ```
  - 遍历mutableObservedChangedKeyPaths，给mutableRequest进行KVC赋值
    在requestSerializer初始化时，会对mutableObservedChangedKeyPaths进行KVO操作
    ```ObjC
    for (NSString *keyPath in AFHTTPRequestSerializerObservedKeyPaths()) {
        if ([self.mutableObservedChangedKeyPaths containsObject:keyPath]) {
            [mutableRequest setValue:[self valueForKeyPath:keyPath] forKey:keyPath];
        }
    }
    ```
  - 将参数进行编码并传递给mutableRequest
    ```ObjC
    mutableRequest = [[self requestBySerializingRequest:mutableRequest withParameters:parameters error:error] mutableCopy];
    ```
    - 执行拷贝
        ```ObjC
        NSMutableURLRequest *mutableRequest = [request mutableCopy];
        ```
    - 设置请求头
        ```ObjC
        [self.HTTPRequestHeaders enumerateKeysAndObjectsUsingBlock:^(id field, id value, BOOL * __unused stop) {
        if (![request valueForHTTPHeaderField:field]) {
            [mutableRequest setValue:value forHTTPHeaderField:field];
        }
        }];
        ``` 
    - 将参数字典转为字符串，支持自定义解析方式
        ```ObjC
        1.先通过递归NSArray * AFQueryStringPairsFromKeyAndValue(NSString *key, id value)获得AFQueryStringPair参数数组
        2.然后遍历参数数组对AFQueryStringPair进行URLEncodedStringValue百分号编码操作
        3.最后对AFQueryStringPair参数数组进行componentsJoinedByString:@"&"元素分割 获得AFQueryStringFromParameters参数字符串
        ```
        ```ObjC
        NSString * AFQueryStringFromParameters(NSDictionary *parameters) {
        NSMutableArray *mutablePairs = [NSMutableArray array];
        for (AFQueryStringPair *pair in AFQueryStringPairsFromDictionary(parameters)) {
            [mutablePairs addObject:[pair URLEncodedStringValue]];
        }
        return [mutablePairs componentsJoinedByString:@"&"];
        }
        ```
        ```ObjC
        NSArray * AFQueryStringPairsFromDictionary(NSDictionary *dictionary) {
            return AFQueryStringPairsFromKeyAndValue(nil, dictionary);
        }
        ```

        ```ObjC
        NSArray * AFQueryStringPairsFromKeyAndValue(NSString *key, id value) {
        NSMutableArray *mutableQueryStringComponents = [NSMutableArray array];
        NSSortDescriptor *sortDescriptor = [NSSortDescriptor sortDescriptorWithKey:@"description" ascending:YES selector:@selector(compare:)];
            if ([value isKindOfClass:[NSDictionary class]]) {
                 NSDictionary *dictionary = value;
                 for (id nestedKey in [dictionary.allKeys sortedArrayUsingDescriptors:@[ sortDescriptor ]]) {
                     id nestedValue = dictionary[nestedKey];
                     if (nestedValue) {
                        [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue((key ? [NSString stringWithFormat:@"%@[%@]", key, nestedKey] : nestedKey), nestedValue)];
                    }
                }
            } else if ([value isKindOfClass:[NSArray class]]) {
                NSArray *array = value;
                    for (id nestedValue in array) {
                        [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue([NSString stringWithFormat:@"%@[]", key], nestedValue)];
                    }
            } else if ([value isKindOfClass:[NSSet class]]) {
                NSSet *set = value;
                for (id obj in [set sortedArrayUsingDescriptors:@[ sortDescriptor ]]) {
                     [mutableQueryStringComponents addObjectsFromArray:AFQueryStringPairsFromKeyAndValue(key, obj)];
                }
            } else {
                 [mutableQueryStringComponents addObject:[[AFQueryStringPair alloc] initWithField:key value:value]];
            }
            return mutableQueryStringComponents;
        }
        ```
    - 若是GET,HEAD,DELETE请求，将参数拼接到url后面，若是PUT,POST请求，先判断是否有设置`Content-Type`，未设置则设置为默认值`application/x-www-form-urlencoded`,然后将参数拼接到http body中
        ```ObjC
        if ([self.HTTPMethodsEncodingParametersInURI containsObject:[[request HTTPMethod] uppercaseString]]) {
             if (query && query.length > 0) {
                mutableRequest.URL = [NSURL URLWithString:[[mutableRequest.URL absoluteString] stringByAppendingFormat:mutableRequest.URL.query ? @"&%@" : @"?%@", query]];
            }
         } else {
            if (!query) {
                query = @"";
            }
            if (![mutableRequest valueForHTTPHeaderField:@"Content-Type"]) {
                [mutableRequest setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
            }
            [mutableRequest setHTTPBody:[query dataUsingEncoding:self.stringEncoding]];
        }
        ``` 
- 通过request生成task
  - 1.使用串行队列创建dtaTask，解决iOS8以下的一个bug，详情见[issue 2093](https://github.com/AFNetworking/AFNetworking/issues/2093)
    ```ObjC
    __block NSURLSessionDataTask *dataTask = nil;
    url_session_manager_create_task_safely(^{
        dataTask = [self.session dataTaskWithRequest:request];
    });
    ```
  - 给task添加代理
    ```ObjC
    [self addDelegateForDataTask:dataTask uploadProgress:uploadProgressBlock downloadProgress:downloadProgressBlock completionHandler:completionHandler];
    ```
    - 
        ```ObjC 
        以taskId为key将delegate存入mutableTaskDelegatesKeyedByTaskIdentifier，每个task都有自己的delegate
        ```
        ```ObjC
        manager > session > task <> delegate > /weak，防止循环引用/ manager 
        manager持有task，task和delegate是绑定的，相当于manager是持有delegate的
        ```
        ```ObjC
        1.用KVO监听AFURLSessionManagerTaskDelegate的downloadProgress和uploadProgress
        2.用KVO监听Task的progress，将监听到的completedUnitCount和totalUnitCount赋给downloadProgress和uploadProgress
        3.监听到downloadProgress和uploadProgress的变化，调用AFURLSessionManagerTaskDelegate的downloadProgressBlock和uploadProgressBlock回调
        ```
- `resume()`
  - task的resume()被替换为af_resume()  
    
    ```ObjC
    - (void)af_resume {
        NSAssert([self respondsToSelector:@selector(state)], @"Does not respond to state");
        NSURLSessionTaskState state = [self state];
        [self af_resume];
        if (state != NSURLSessionTaskStateRunning) {
            [[NSNotificationCenter defaultCenter] postNotificationName:AFNSURLSessionTaskDidResumeNotification object:self];
         }
    }
    ```
  - 当task状态发生改变时，通知到manager  
    
    ```ObjC
     - (void)addNotificationObserverForTask:(NSURLSessionTask *)task {
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(taskDidResume:) name:AFNSURLSessionTaskDidResumeNotification object:task];
        [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(taskDidSuspend:) name:AFNSURLSessionTaskDidSuspendNotification object:task];
    }
    ```
  - manager发布task状态改变的广播  
    
    ```ObjC
    - (void)taskDidResume:(NSNotification *)notification {
        NSURLSessionTask *task = notification.object;
        if ([task respondsToSelector:@selector(taskDescription)]) {
          if ([task.taskDescription isEqualToString:self.taskDescriptionForSessionTasks]) {
              dispatch_async(dispatch_get_main_queue(), ^{
                  [[NSNotificationCenter defaultCenter] postNotificationName:AFNetworkingTaskDidResumeNotification object:task];
                });
            }
        }
    }
    ```
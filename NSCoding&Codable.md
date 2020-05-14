### NSCoding & Codable & NSKeyedArchiver

##### 缓存符合NSCoding协议的任意对象
	
	- (void)encodeWithCoder:(NSCoder *)coder;
	- (nullable instancetype)initWithCoder:(NSCoder *)coder; // NS_DESIGNATED_INITIALIZER

##### Example

	override func encode(with coder: NSCoder) {
	  super.encode(with: coder)
	}
	
	required init?(coder: NSCoder) {
	  super.init(coder: coder)
	}
	
	override init(nibName nibNameOrNil: String?, bundle nibBundleOrNil: Bundle?) {
	  super.init(nibName: nibNameOrNil, bundle: nil)
	}
	
	convenience init() {
	  self.init(nibName: nil, bundle: nil)
	}
	
	override func viewDidAppear(_ animated: Bool) {
	  super.viewDidAppear(animated)
	  
	  if let libraryPath = NSSearchPathForDirectoriesInDomains(.libraryDirectory, .userDomainMask, true).last {
	    let directory = libraryPath + "/archiver_directory"
	    if !FileManager.default.fileExists(atPath: directory) {
	      try? FileManager.default.createDirectory(atPath: directory, withIntermediateDirectories: true, attributes: nil)
	    }
	    let data = NSMutableData()
	    let archiver = NSKeyedArchiver(forWritingWith: data)
	    archiver.encode(UIViewController(), forKey: "archiver_key")
	    archiver.finishEncoding()
	    data.write(to: URL.init(fileURLWithPath: directory + "/file_name"), atomically: true)
	  }
	  
	  GCDQueue.executeInMainQueue({
	    if let libraryPath = NSSearchPathForDirectoriesInDomains(.libraryDirectory, .userDomainMask, true).last {
	      let directory = libraryPath + "/archiver_directory"
	      if let data = NSData(contentsOfFile: directory + "/file_name") as Data? {
	        let unarchiver =  NSKeyedUnarchiver.init(forReadingWith: data)
	        if let vc = unarchiver.decodeObject(forKey: "archiver_key") as? UIViewController {
	          self.present(vc, animated: true, completion: nil)
	        }
	      }
	    }
	  }, afterDelay: .seconds(10))
	}
	  
  
  
### Codable协议

	typealias Codable = Decodable & Encodable
	
	public protocol Decodable {
	    init(from decoder: Decoder) throws
	}
	
	public protocol Encodable {
	    func encode(to encoder: Encoder) throws
	}
	
#### JSONEncoder

	struct GroceryProduct: Codable {
      var name: String
      var points: Int
      var description: String?
	}

	let pear = GroceryProduct(name: "Pear", points: 250, description: "A ripe pear.")
	
	let encoder = JSONEncoder()
	encoder.outputFormatting = .prettyPrinted
	
	let data = try encoder.encode(pear)
	print(String(data: data, encoding: .utf8)!)
	
	/* Prints:
	 {
	   "name" : "Pear",
	   "points" : 250,
	   "description" : "A ripe pear."
	 }
	*/
	
	
	struct GroceryProduct: Codable {
       var name: String
       var points: Int
      var description: String?
	}
#### JSONDecoder
	let json = """
	{
	    "name": "Durian",
	    "points": 600,
	    "description": "A fruit with a distinctive scent."
	}
	""".data(using: .utf8)!
	
	let decoder = JSONDecoder()
	let product = try decoder.decode(GroceryProduct.self, from: json)
	
	print(product.name) // Prints "Durian"
	
####  自定义嵌入键	

	class User: Codable {
	  
	  var name: String?
	  
	  // 自定义嵌入键
	  enum CodingKeys: String, CodingKey {
	    case memberName
	  }
	  
	  required init(from decoder: Decoder) throws {
	    let container = try decoder.container(keyedBy: CodingKeys.self)
	    name = try container.decode(String.self, forKey: .memberName)
	  }
	  
	  func encode(to encoder: Encoder) throws {
	    var container = encoder.container(keyedBy: CodingKeys.self)
	    try container.encode(name, forKey: .memberName)
	  }
	
	  init() {
	    
	  }
	}
	
	let jsonData = """
	{
	  "memberName": "Janine",
	}
	""".data(using: .utf8)!
	
	
	func test() {
	  do {
	    let user = try? JSONDecoder().decode(User.self, from: jsonData)
	    let data = try? JSONEncoder().encode(user)
	    print(user.debugDescription)
	    print(data?.description)
	  } catch {
	    print(error.localizedDescription)
	  }
	}
	
	
NSCoding & NSKeyedArchiver &  Data Parsing

	
	#import "DataModel.h"
	#import <objc/runtime.h>
	
	@implementation DataModel
	
	- (NSArray *)getAllPropertyNames       //获取类的所有属性名称
	{
	    u_int count;
	    objc_property_t *properties  =class_copyPropertyList([self class], &count);
	    NSMutableArray *propertiesArray = [NSMutableArray arrayWithCapacity:count];
	    for (int i = 0; i < count ; i++)
	    {
	        const char* propertyName =property_getName(properties[i]);
	        [propertiesArray addObject: [NSString stringWithUTF8String: propertyName]];
	    }
	    free(properties);
	    return propertiesArray;
	}
	
	- (id)bundingData:(NSDictionary *)dict{          //绑定数据
	    static Class oldClass;
	    if ([self class]!=oldClass) {
	        DLog(@"   >>>>>>>  绑定模型 >>>>>>>>>> ： %@\n\n",[self class]);
	        oldClass = [self class];
	    }
	    if (dict==nil) {
	        return nil;
	    }
	    NSArray *propertyNames = [self getAllPropertyNames];
	    for (NSString *key in propertyNames) {
	        id value = nil;
	        if ([key isEqualToString:@"pid"]) {
	            value = [dict objectForKeyOrNil:@"id"];
	        }else{
	            value = [dict objectForKeyOrNil:key];
	        }
	        if (value) {
	            [self setValue:value forKey:key];   //使用KVC给属性赋值
	        }
	    }
	    return self;
	}
	
	- (NSDictionary *)toDictionary{
	    NSArray *propertyKeyArray = [self getAllPropertyNames];
	    NSMutableDictionary *dict = [NSMutableDictionary dictionary];
	    for (NSString *key in propertyKeyArray) {
	        id obj = SafeObj([self valueForKey:key]);
	        //对象为空则不转换到Dictionary中
	        if ((NSNull *)obj == [NSNull null]) {
	            continue;
	        }
	        if ([key isEqualToString:@"pid"]) {
	            [dict setObject:obj forKey:@"id"];
	        }else{
	            [dict setObject:obj forKey:key];
	        }
	    }
	    return dict;
	}
	
	- (void)clearData{          //清除对象的值
	    //   DLog(@"%@",self);                   //输出调试信息
	    NSArray *propertyNames = [self getAllPropertyNames];
	    for (NSString *key in propertyNames) {
	        [self setValue:[NSNull null] forKey:key];   //使用KVC给属性赋值,清除对象的值
	    }
	    DLog(@"清除类 ------ > %@ (%@)",NSStringFromClass([self class]),self);
	}
	
	
	- (NSString *)description               //类描述方法
	{
	    NSMutableString *desc = [NSMutableString string];
	    [desc appendString:@"\n\n"];
	    [desc appendFormat:@"Class name: %@\n", NSStringFromClass([self class])];
	    NSArray *propertyNames = [self getAllPropertyNames];
	    for (NSString *key in propertyNames) {
	        [desc appendFormat: @"%@: %@\n", key, [self valueForKey:key]];
	    }
	    return [NSString stringWithString:desc];
	}
	
	
	#pragma -mark 对象归档协议<NSCoding>方法
	- (void)encodeWithCoder:(NSCoder *)encoder {
	    NSArray *propertyNames = [self getAllPropertyNames];
	    for (NSString *key in propertyNames) {
	        [encoder encodeObject:[self valueForKey:key] forKey:key];
	    }
	}
	
	- (id)initWithCoder:(NSCoder *)decoder {
	    NSArray *propertyNames = [self getAllPropertyNames];
	    for (NSString *key in propertyNames) {
	        [self setValue:[decoder decodeObjectForKey:key] forKey:key] ;
	    }
	    return self;
	}
	
	#pragma -mark 对象归档
	- (void)archiverToFile:(NSString *)fileName{                        //对象归档保存
	    NSString *fullPath = [self getDocumentsPath:fileName];
	    [NSKeyedArchiver archiveRootObject:self toFile:fullPath];
	}
	
	- (id)unArchiverFromFile:(NSString *)fileName{                      //反归档对象
	    NSString *fullPath = [self getDocumentsPath:fileName];
	    if ([[NSFileManager defaultManager] fileExistsAtPath:fullPath]){
	        DataModel *model = [NSKeyedUnarchiver unarchiveObjectWithFile: fullPath];
	        return model;
	    }else{
	        return nil;
	    }
	}
	#pragma -mark 辅助方法
	- (NSString *)getDocumentsPath:(NSString *)fileName{          //获取文档目录
	    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
	    NSString *docPath = [paths objectAtIndex:0];
	    return [docPath stringByAppendingPathComponent:fileName];
	}
	
	@end
	
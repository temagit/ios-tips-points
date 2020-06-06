# FaceID + TouchID + KeychainAccess Biometric Authentication


### FaceID Or Touch ID 

	enum LAPolicy : Int  {
	
	  case deviceOwnerAuthenticationWithBiometrics = 1
	  case deviceOwnerAuthentication = 2
	}

##### deviceOwnerAuthenticationWithBiometrics	
+ 只有指纹验证功能
+ 设备所有者使用生物识别方法进行认证。TouchID验证是必须的，如果TouchID不可用或者没有注册，则策略评估将会失败。如果TouchID是锁被锁定，则需要输入密码作为解锁TouchID的第一步。
+ TouchID对话框包括一个取消按钮，默认标题为“取消”，可以使用“localizedCancelTitle”属性去修改。有一个fallback按钮默认标题为“输入密码”，可以通过“localizedFallbackTitle”属性去修改。
+ fallback按钮最初是隐藏的，并且在首次touchID尝试失败之后显示。
+ 点击取消按钮或者输入密码按钮后会导致 evaluatePolicy: 方法调用失败，返回一个不同的错误代码。
+ 5次验证失败，生物识别将被锁定。之后，用户必须输入密码才可以解锁验证。

##### deviceOwnerAuthentication

+ 包含指纹认证与输入密码认证方式
+ 设备所有者使用生物识别后者设备密码进行验证。
+ TouchID或者密码验证是必须的，如果TouchID可用，注册并且未被锁定，首先使用TouchID验证。否则直接要求输入设备密码。
+ 如果密码未被启用，则策略评估失败。
+ Touch ID身份验证对话框的行为与LAPolicyDeviceOwnerAuthenticationWithBiometrics使用的行为相似。当点击输入密码按钮的时候，切换验证方法，并允许用户输入设备密码。
+ 密码锁定会在6次失败尝试之后被锁定，并且逐步增加退避延迟。

##### Face ID 需要增加权限申请说明，不加不会崩溃，最好添加
`NSFaceIDUsageDescription` 

##### biometryType

	enum LABiometryType : Int {
	    case none = 0
	    public static var LABiometryNone: LABiometryType { get }
	    case touchID = 1
	    case faceID = 2
	}


+ canEvaluatePolicy:生物识别策略成功之后才会去设置这个属性的值
+ 这个属性的值只有在你调用canEvaluatePolicy:方法之后并且返回是YES没有错误的情况下才会设置，才会有值。在调用 canEvaluatePolicy: 方法前，或者调用后但是有Error的情况下，该属性均无任何有意义的值，验证之后实际为空

#### LAError

	typedef NS_ENUM(NSInteger, LAError)
	{
	    //身份验证不成功，因为用户无法提供有效的凭据。
	    LAErrorAuthenticationFailed = kLAErrorAuthenticationFailed,
	
	    //认证被用户取消(例如了取消按钮)。
	    LAErrorUserCancel = kLAErrorUserCancel,
	
	    //认证被取消了,因为用户利用回退按钮(输入密码)。
	    LAErrorUserFallback = kLAErrorUserFallback,
	
	    //身份验证被系统取消了(如另一个应用程序去前台)。
	    LAErrorSystemCancel = kLAErrorSystemCancel,
	
	    //身份验证无法启动,因为设备没有设置密码。
	    LAErrorPasscodeNotSet = kLAErrorPasscodeNotSet,
	
	    //身份验证无法启动,因为触摸ID不可用在设备上。
	    LAErrorTouchIDNotAvailable NS_ENUM_DEPRECATED(10_10, 10_13, 8_0, 11_0, "use LAErrorBiometryNotAvailable") = kLAErrorTouchIDNotAvailable,
	
	    //身份验证无法启动,因为没有登记的手指触摸ID。
	    LAErrorTouchIDNotEnrolled NS_ENUM_DEPRECATED(10_10, 10_13, 8_0, 11_0, "use LAErrorBiometryNotEnrolled") = kLAErrorTouchIDNotEnrolled,
	
	    //验证不成功,因为有太多的失败的触摸ID尝试和触///摸现在ID是锁着的。
	    //解锁TouchID必须要使用密码，例如调用LAPolicyDeviceOwnerAuthenti//cationWithBiometrics的时候密码是必要条件。
	    //身份验证不成功，因为有太多失败的触摸ID尝试和触摸ID现在被锁定。
	    LAErrorTouchIDLockout NS_ENUM_DEPRECATED(10_11, 10_13, 9_0, 11_0, "use LAErrorBiometryLockout")
	    __WATCHOS_DEPRECATED(3.0, 4.0, "use LAErrorBiometryLockout") __TVOS_DEPRECATED(10.0, 11.0, "use LAErrorBiometryLockout") = kLAErrorTouchIDLockout,
	
	    //应用程序取消了身份验证（例如在进行身份验证时调用了无效）。
	    LAErrorAppCancel NS_ENUM_AVAILABLE(10_11, 9_0) = kLAErrorAppCancel,
	
	    //LAContext传递给这个调用之前已经失效。
	    LAErrorInvalidContext NS_ENUM_AVAILABLE(10_11, 9_0) = kLAErrorInvalidContext,
	
	    //身份验证无法启动,因为生物识别验证在当前这个设备上不可用。
	    LAErrorBiometryNotAvailable NS_ENUM_AVAILABLE(10_13, 11_0) __WATCHOS_AVAILABLE(4.0) __TVOS_AVAILABLE(11.0) = kLAErrorBiometryNotAvailable,
	
	    //身份验证无法启动，因为生物识别没有录入信息。
	    LAErrorBiometryNotEnrolled NS_ENUM_AVAILABLE(10_13, 11_0) __WATCHOS_AVAILABLE(4.0) __TVOS_AVAILABLE(11.0) = kLAErrorBiometryNotEnrolled,
	
	    //身份验证不成功，因为太多次的验证失败并且生物识别验证是锁定状态。此时，必须输入密码才能解锁。例如LAPolicyDeviceOwnerAuthenticationWithBiometrics时候将密码作为先决条件。
	    LAErrorBiometryLockout NS_ENUM_AVAILABLE(10_13, 11_0) __WATCHOS_AVAILABLE(4.0) __TVOS_AVAILABLE(11.0) = kLAErrorBiometryLockout,
	
	    //身份验证失败。因为这需要显示UI已禁止使用interactionNotAllowed属性。  据说是beta版本
	    LAErrorNotInteractive API_AVAILABLE(macos(10.10), ios(8.0), watchos(3.0), tvos(10.0)) = kLAErrorNotInteractive,
	} NS_ENUM_AVAILABLE(10_10, 8_0) __WATCHOS_AVAILABLE(3.0) __TVOS_AVAILABLE(10.0);


#### Error codes
 
    #define kLAErrorAuthenticationFailed                       -1
    #define kLAErrorUserCancel                                 -2
    #define kLAErrorUserFallback                               -3
    #define kLAErrorSystemCancel                               -4
    #define kLAErrorPasscodeNotSet                             -5
    #define kLAErrorTouchIDNotAvailable                        -6
    #define kLAErrorTouchIDNotEnrolled                         -7
    #define kLAErrorTouchIDLockout                             -8
    #define kLAErrorAppCancel                                  -9
    #define kLAErrorInvalidContext                            -10
    #define kLAErrorNotInteractive                          -1004
    #define kLAErrorBiometryNotAvailable              kLAErrorTouchIDNotAvailable
    #define kLAErrorBiometryNotEnrolled               kLAErrorTouchIDNotEnrolled
    #define kLAErrorBiometryLockout                   kLAErrorTouchIDLockout
    
+   FaceID TouchID  使用时, 必须确保app已经是活动状态
+   同一个文件路径下的同一份代码，打包编译成多个ipa安装包，就算各个包的Bundle Identifier不同，安装到同一个设备后，也只有最后生成的那个ipa包可以启用TouchID，其他包会报Code: -1004 NSLocalizedDescription: User interaction is required的错误。


### FaceID + TouchID + KeychainAccess Biometric Authentication
  
    import KeychainAccess
       
		typealias BiometricsFallBack = () -> Void
		typealias BiometricsCallBack = (_ useable: Bool, _ success: Bool, _ error: Error?) -> Void
		
		class BiometricsHelper {
		
		  static let DEVICEFINGERPRINT = "com.keychain.com"
		  static let SERVICE = "com.keychain.service.com"
		
		  class func isFingerprintChange() -> Bool {
		    let data = self.getDeviceFingerprint()
		    if data.isNone && self.isLockout() {
		      return false
		    } else {
		      let oldData = self.getSavedDeviceFingerprint()
		      if oldData.isNone {
		        return false
		      } else if oldData == data {
		        return false
		      } else {
		        return true
		      }
		    }
		  }
		
		  open class func deviceOwnerAuthenticationWithBiometrics(title: String, fallbackTitle: String?, fallbackBlock: BiometricsFallBack?, resultBlock: BiometricsCallBack?) {
		    let context = LAContext();
		    context.localizedFallbackTitle = fallbackTitle
		    var useableError: NSError?
		    if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &useableError) {
		      context.evaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, localizedReason: title) { (success, error) in
		        DispatchQueue.main.async {
		          if success {
		            if resultBlock.isSome {
		              resultBlock!(true, success, error)
		              _ = saveDeviceFingerprint()
		            }
		          } else {
		            guard let error = error else { return }
		            print("ErrorCode:\(error._code) errorMsg:\(error.localizedDescription)")
		            if error._code == LAError.userFallback.rawValue {
		              if fallbackBlock.isSome {
		                fallbackBlock!()
		              }
		            } else if error._code == LAError.biometryLockout.rawValue {
		              self.deviceOwnerAuthentication(title: title, resultBlock: resultBlock)
		            } else {
		              if resultBlock.isSome {
		                resultBlock!(false, success, error)
		              }
		            }
		          }
		        }
		      }
		    } else {
		      print("ErrorCode:\(String(describing: useableError?._code)) errorMsg:\(String(describing: useableError?.localizedDescription))")
		      if useableError?.code == LAError.biometryLockout.rawValue {
		        self.deviceOwnerAuthentication(title: title, resultBlock: resultBlock)
		      } else {
		        if resultBlock.isSome {
		          resultBlock!(false, false, useableError)
		        }
		      }
		    }
		  }
		
		  private class func deviceOwnerAuthentication(title: String, resultBlock: BiometricsCallBack?) {
		    let context = LAContext();
		    context.localizedFallbackTitle = ""
		    var useableError: NSError?
		    if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthentication, error: &useableError) {
		      context.evaluatePolicy(LAPolicy.deviceOwnerAuthentication, localizedReason: title) { (success, error) in
		        DispatchQueue.main.async {
		          if resultBlock.isSome {
		            resultBlock!(true, success, error)
		          }
		        }
		        guard let error = error else{ return }
		        print("ErrorCode:\(error._code) errorMsg:\(error.localizedDescription)")
		      }
		    } else {
		      print("ErrorCode:\(String(describing: useableError?._code)) errorMsg:\(String(describing: useableError?.localizedDescription))")
		      if resultBlock.isSome {
		        resultBlock!(false, false, useableError)
		      }
		    }
		  }
		
		  private class func getSavedDeviceFingerprint() -> Data? {
		    let keychain = Keychain(service: SERVICE)
		    return keychain[data: DEVICEFINGERPRINT]
		  }
		  
		  private class func saveDeviceFingerprint() -> Bool {
		    if let data = self.getDeviceFingerprint(){
		      let keychain = Keychain(service: SERVICE)
		      keychain[data: DEVICEFINGERPRINT] = data
		      return true;
		    } else {
		      return false;
		    }
		  }
		
		  private class func getDeviceFingerprint() -> Data? {
		    let context = LAContext()
		    var error: NSError? = nil;
		    if context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &error) {
		      return context.evaluatedPolicyDomainState
		    }
		    print("ErrorCode:\(String(describing: error?._code)) errorMsg:\(String(describing: error?.localizedDescription))")
		    return nil
		  }
		
		  private class func isLockout() -> Bool {
		    let context = LAContext()
		    var error: NSError?
		    context.canEvaluatePolicy(LAPolicy.deviceOwnerAuthenticationWithBiometrics, error: &error)
		    guard error != nil else { return false }
		    if error!.code == LAError.biometryLockout.rawValue {
		      return true
		    } else {
		      return false
		    }
	  }
	}


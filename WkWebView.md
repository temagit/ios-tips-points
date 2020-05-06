### WKWebView 多Cookie更新同步
+ HTTPCookieStorage与websiteDataStore.httpCookieStore的Cookie同步有时间差，并且同步时间不能确定，需手动同步，websiteDataStore httpCookieStore iOS11之后添加，所以iOS11之前使用document.cookie设置同步cookie。

+ websiteDataStore httpCookieStore的cookie同步必须在主线程，并且是异步回调，所以使用递归逐一同步，最终回调。webview的同步时间必须在Request之前，否则不起作用。可以对UIView增加addSubview的交换方法，在addSubview时，对webview进行一次所有Cookies同步。

+ Cookie存储在HTTPCookieStorage 当中，它是同步执行的，可以使用UserDefaults初始化持久化操作。

		
		
		extension WKWebView {
		
		  func synchronizeCookie(domain: String, name: String, completionHandler: (() -> Void)? = nil) {
		    guard let cookie = HTTPCookieStorage.shared.cookies?.filter({ $0.domain == domain && $0.name == name }).first else { return }
		    if #available(iOS 11.0, *) {
		      self.configuration.websiteDataStore.httpCookieStore.setCookie(cookie) { [weak self] in
		        guard let _ = self, let completionHandler = completionHandler else { return }
		        completionHandler()
		      }
		      return
		    }
		    let script = String.init(format: "document.cookie='%@=%@;expires=%@;domain=%@;path=%@;'", cookie.name, cookie.value, Date().addingTimeInterval(14515200).GMT(), cookie.domain, cookie.path)
		    self.evaluateJavaScript(script) { (result, error) in
		      if let completionHandler = completionHandler {
		        completionHandler()
		      }
		    }
		  }
			
		  func synchronizeCookies(completionHandler: (() -> Void)? = nil) {
		    guard let cookies = HTTPCookieStorage.shared.cookies else { return }
		    if #available(iOS 11.0, *) {
		      synchronizeIOS11Later(cookies: cookies, index: 0, completionHandler: completionHandler)
		      return
		    }
		    var script = ""
		    HTTPCookieStorage.shared.cookies?.forEach({ (cookie) in
		      script += String.init(format: "document.cookie='%@=%@;expires=%@;domain=%@;path=%@;'", cookie.name, cookie.value, Date().addingTimeInterval(14515200).GMT(), cookie.domain, cookie.path)
		    })
		    if !script.isEmpty {
		      self.evaluateJavaScript(script) { (result, error) in
		        if let completionHandler = completionHandler {
		          completionHandler()
		        }
		      }
		    }
		  }
			
		  private func synchronizeIOS11Later(cookies: [HTTPCookie], index: Int, completionHandler: (() -> Void)? = nil) {
		    if #available(iOS 11.0, *) {
		      self.configuration.websiteDataStore.httpCookieStore.setCookie(cookies[index]) { [weak self] in
		        guard let self = self else { return }
		        if index + 1 < cookies.count {
		          self.synchronizeIOS11Later(cookies: cookies, index: index + 1, completionHandler: completionHandler)
		        } else if index + 1 == cookies.count, let completionHandler = completionHandler {
		          completionHandler()
		        }
		      }
		    }
		  }
		}
			
		private extension Date {
		  func GMT() -> String {
		    let dateFormatter = DateFormatter()
		    dateFormatter.timeZone = NSTimeZone.local
		    dateFormatter.dateFormat = "EEE, dd MMM yyyy HH:mm:ss 'GMT'"
		    return dateFormatter.string(from: self)
		  }
}


### WKWebView 清除缓存
WKWebView,在iOS9以后提供了缓存管理类WKWebsiteDataStore

获取WKWebView web缓存数据类型

    [WKWebsiteDataStore allWebsiteDataTypes];
    
支持的web缓存类型有

    WKWebsiteDataTypeDiskCache,
    WKWebsiteDataTypeOfflineWebApplicationCache,
    WKWebsiteDataTypeMemoryCache,
    WKWebsiteDataTypeLocalStorage,
    WKWebsiteDataTypeCookies,
    WKWebsiteDataTypeSessionStorage,
    WKWebsiteDataTypeIndexedDBDatabases,
    WKWebsiteDataTypeWebSQLDatabases
    
根据缓存类型和记录时间移除
    
    WKWebsiteDataStore.default().removeData(ofTypes: WKWebsiteDataStore.allWebsiteDataTypes(), modifiedSince: Date.init(timeIntervalSince1970: 0)) {}

### WebView 禁止3DTouch手势
    webView.evaluateJavaScript("document.documentElement.style.webkitTouchCallout='none';", completionHandler: nil)
    webView.evaluateJavaScript("document.documentElement.style.webkitUserSelect='none';", completionHandler: nil)
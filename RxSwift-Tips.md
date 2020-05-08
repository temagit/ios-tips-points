####1、debounce
   过滤掉高频产生的元素，例如定位改变时刷新接口,短时间多次调会造成服务器压力过大，可以使用debounce限流，下面例子10s刷新一次
   ![debounce.png](https://upload-images.jianshu.io/upload_images/1419920-cf1d9e1282c1f09c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```  
NotificationCenter.default.rx.notification(Notification.Name(rawValue: "kLocationUpdated"))
      .debounce(10, scheduler: MainScheduler.instance)
      .subscribe(onNext: {[weak self] (notification) in
        guard let strongSelf = self else { return }
        // 刷新数据
}).disposed(by: disposeBag)
```


####2、takeUntil
忽略掉在第二个 Observable 产生事件后发出的那部分元素，例如cell进行复用时，忽略之前的订阅

![takeUntil.png](https://upload-images.jianshu.io/upload_images/1419920-3532181accb046f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

先扩展cell的复用序列


	extension Reactive where Base: UITableViewCell {
	  public var prepareForReuse: RxSwift.Observable<Void> {
	        var prepareForReuseKey: Int8 = 0
	        if let prepareForReuseOB = objc_getAssociatedObject(base, &prepareForReuseKey) as? Observable<Void> {
	            return prepareForReuseOB
	        }
	     //将cell复用和deinit转换为序列
	        let prepareForReuseOB = Observable.of(
	            sentMessage(#selector(Base.prepareForReuse)).map { _ in }
	            , deallocated)
	            .merge()
	        objc_setAssociatedObject(base, &prepareForReuseKey, prepareForReuseOB, objc_AssociationPolicy.OBJC_ASSOCIATION_RETAIN)
	
	        return prepareForReuseOB
	    }
	}


takeUntil当cell复用或者销毁cell button 回来复用之前的订阅

	cell.button.rx.tap.takeUntil(cell.rx.prepareForReuse)
	    .subscribe(onNext: { () in
	        print("点击了 \(indexPath)")
	    })

####3、cell重复订阅问题

除了刚刚上面的takeUntil方法，也可以对cell扩展一个专用DisposeBag


	extension Reactive where Base: UITableViewCell {
	  public var reuseBag: DisposeBag {
	        MainScheduler.ensureExecutingOnScheduler()
	        var prepareForReuseBag: Int8 = 0
	        if let bag = objc_getAssociatedObject(base, &prepareForReuseBag) as? DisposeBag {
	            return bag
	        }
	        
	        let bag = DisposeBag()
	        objc_setAssociatedObject(base, &prepareForReuseBag, bag, objc_AssociationPolicy.OBJC_ASSOCIATION_RETAIN)
	        
	        _ = sentMessage(#selector(Base.prepareForReuse))
	            .subscribe(onNext: { [weak base] _ in
	                let newBag = DisposeBag()
	                guard let base = base else {return}
	                objc_setAssociatedObject(base, &prepareForReuseBag, newBag, objc_AssociationPolicy.OBJC_ASSOCIATION_RETAIN)
	            })
	        return bag
	    }
	}


cell使用扩展的reuseBag 防止重复订阅 

```
cell.button.rx.tap
    .subscribe(onNext: { () in
        print("点击了 \(indexPath)")
    })
 .disposed(by: cell.rx.reuseBag)
```





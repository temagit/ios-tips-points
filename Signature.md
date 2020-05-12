因为苹果的安全策略，通过签名保证安装在设备上都是苹果认证的，APP的安装方法有四种方式：

1.  通过App Store 安装
2.  开发者通过 Xcode 安装
3.  内测分发 ad-hoc，设备数量限制100
4.  In-House企业证书分发，设备信任证书后可以使用App，无设备数量限制

#### 通过App Store 安装

1.  由苹果生成一对公钥私钥，私钥保存在苹果后台，公钥保存在iOS设备中
2.  开发者上传App审核，由苹果使用私钥进行数据签名，发布至App Store
3.  用户App Store 下载应用后使用公钥进行数据验证，验证通过则表明是苹果认证通过的

#### 通过Xcode安装

苹果采用双重签名机制。Xcode安装使用Mac电脑的一对公钥私钥进行签名，苹果还是原来的一对公钥私钥签名。

1. 开发时需要真机测试时，需要从钥匙串中的证书中心创建证书请求文件CSR（CertificateSigningRequest.certSigningRequest，用于绑定电脑，文件中应该有Mac电脑的公钥
2. Apple 使用私钥对CSR签名，生成一份包含公钥信息以及Apple对它的签名信息，被称为证书（CER，开发证书，发布证书）
3. 编译完一个APP时，Mac使用私钥对App进行签名
4. 在安装App 时， 根据当前配置把CER证书一起打包进APP里面。 
5. iOS社保通过内置的Apple的公钥验证，CER是否正确，证书验证确保Mac公钥时是经过Apple认证的。
6. 使用CER文件中的Mac公约去验证App的签名是否正确，确保安装行为是苹果允许的。

> `Apple 只确定安装是否合法，不会验证App内容是否修改。 `

通过Ad-Hoc 方法打包安装

Xcode打包APP 生成ipa文件，通过iTunes或者第三方发布平台，安装到设备上。流程与Xcode真机调试基本相同。区别于第四步

1. 开发时需要测试打包或者法布施。需要从钥匙串证书中心 创建证书文件（CSR）,并且上传至Apple服务器。
2. Apple 使用私钥对CSR签名。生成一份包含Mac公钥以及Apple对它的签名，被称之为（开发证书，发布证书）
3. 编译完一个APP 后，Mac使用私钥对App进行签名。
4. 编译签名完后，要导出ipa文件时，导出时，需要选择一个保存的方法（App Store/ Ad Hoc/ Enterprise / Development）,就是选择将上一步生成的CER一起打包进APP
5. iOS 设备通过内置的Apple的公钥验证CER是否正确，证书验证确保Mac是经过苹果认证的。
6. 再使用CER文件中MAC的公钥去验证APP签名是否正确。确保安装行为是苹果允许的。

#### 通过In-House 方法打包安装

企业版证书签名验证流程和Ad-Hoc差不多。只是企业版不限制设备数，而且需要用户在iOS设备上手动点击信任证书。

+ In-House 与 Ad-Hoc 第四步是打包证书进入APP时,需要加上允许安装的设备ID和App对应的APPID等数据（Profile文件）
+ 根据数字签名的原理。只要数字签名通过验证。这里第五步设备IDs/AppleID/Mac公钥都是苹果认证的，无法被修改，苹果就是通过这个限制设备和App,避免滥用。
+ 苹果还要控制iCloud/push/后台运行等，这些都需要苹果授权签名，苹果把这些权限开关统称为：Entitlements，去让苹果授权。
+ 因此证书中可能包含很多东西，不符合规定的格式规范，所以有了Provisioning Profile（描述文件），描述中包含了证书以及其他所有的信息及信息的签名。

> App Store	用于发布到App Store。使用的是发布证书（Cer）。

> Ad Hoc	安装到指定设备上，用于测试。使用的是发布证书（Cer）。

> Enterprise	企业版证书签名。

> Development	安装到指定设备，用于测试。使用的是开发证书（Cer）。


### 流程总结

1. Mac电脑和苹果分别有一套公钥私钥， 苹果的私钥在后台，公钥存放在每个iOS设备，Mac的私钥存放在电脑，公钥是要发送给苹果服务器。
2. Mac从钥匙串生成CSR(包含mac公钥)，上传至苹果服务器。
3. 苹果服务器使用私钥对CSR进行签名，得到包含Mac公钥以及其签名的数据，称之为证书（cer文件）
4. 从苹果后台申请Apple ID , 配置好设备ID列表及App的其他权限，使用苹果的私钥进行签名。生成描述文件， 第3步的证书，一起下载到Mac安装，钥匙串会自动将Cer与之前生成的CSR文件的私钥关联（公钥私钥对应）。
5. 使用Mac编译APP后，使用Mac私钥进行签名，并且描述文件打包进入APP，文件名为`embedded.mobileprovision`
6. 安装App时，iOS设备取得证书，使用内置的Apple私钥去验证Cer及`embedded.mobileprovision`文件。
7. 保证Cer及`embedded.mobileprovision`是经过苹果认证之后，从Cer中取出Mac公钥，验证App签名，以及设备列表，权限开关是否对应

+ 其他人想要编译签名App时应怎么做?
简单就是把私钥给他。私钥也是从 钥匙串 中导出，就是.p12文件，其他Mac导入私钥后就可以正常使用了

+ 解压.ipa文件，得到App数据包，显示包内容，找到embedded.mobileprovision文件所在目录，运行命令security cms -D -i embedded.mobileprovision

> 证书请求文件（CertificateSigningRequest.certSigningRequest）	本地公钥。
> 
> 证书（Cer）	公钥及苹果签名后的信息。
> 
> Entitlements	包含了 App 权限开关列表。
> 
> p12	本地私钥，可以导入到其他电脑。
> 
> rovisioning Profile	包含了 证书 / Entitlements 等数据，并由苹果后台私钥签名的数据包。



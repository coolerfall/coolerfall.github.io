---
title: 使用gradle将项目上传到maven中央库
category: android
tags: [android studio, maven]
---

用AS开发也有一段时间了，发现AS可以使用maven，在gradle中可以很方便的解决依赖问题，再也不用去下载相应的jar包了。gradle添加一些依赖库比较方便，但是如果想把自己的一些开源项目上传到maven中央库给别人使用，就稍微有些麻烦了。现在比较简单的办法是先将自己在github上的项目提交到Sonatype，然后再同步到maven中央库去，大致说一下流程。

<!-- more -->

#### 注册JIRA账号
要想把项目提交到Sonatype，首先得得去[Sonatype Issue][1]去注册一个JIRA账号，注册流程很简单，挨着填写就ok了，这个账号在后面配置的时候要用到。
#### 创建一个issue
账号注册好之后，就开始创建一个issue，这个issue对应你的一个项目，在上面导航栏中有个Create按钮：
![img][6]
然后再弹出的对话框中填写项目的信息：
![img][7]
![img][8]
其中Group Id很重要，一般为自己的域名，这个在后面提交项目的时候要用到。创建好issue后，接下来就是等待，然后工作人员会给你发送邮件。这个时候可以在导航栏中的Issue中找到自己创建的issue，到下面的comments可以看到工作人员问你之前填写的group id这个域名是否是你的，回答是，然后工作人员就会让你提交你的项目到Sonatype:
![img][9]
#### 生成GPG密钥，用于上传的文件加密和签名
linux下可以直接使用gpg。windows上则可以使用git自带的gpg(如果没有安装git，可以直接安装[gpg4win][3])，在cmd中执行：
```shell
$ gpg --gen-key
```
姓名，邮箱以及备注要认真填写，最后要求输入passphase，这个是密码，上传maven要用到。

```shell
$ gpg --list-keys --keyid-format SHORT
/home/xxxx/.gnupg/pubring.kbx
-------------------------------
pub   rsa4096/BB090204 2019-01-03 [SC]
      7B0A1A36C235E95636B75C53D781EED2BB090204
uid           [ultimate] Vincent Cheung (coolerfall.com) <coolingfall@gmail.com>
sub   rsa4096/9123FD1A 2019-01-03 [E]
```

其中BB090204是key id，接下来将公钥上传至服务器：
```shell
$ gpg --keyserver hkp://pool.sks-keyservers.net --send-keys BB090204
```

gpg 2.1之后需要手动导出secring：
```shell
gpg --keyring secring.gpg --export-secret-keys > ~/.gnupg/secring.gpg
```

#### 使用gradle提交项目到Sonatype
在网上找到了Chris Banes写的[gradle-mvn-push.gradle][4]脚本，参照这个基本就可以了，然后将这个脚本在你项目的`build.gradle`的最后加入`apply from: '../maven_push.gradle'`，如果想发布jar，需要在脚本中加入：
```groovy
artifacts {
	archives packageReleaseJar
}
```

然后需要在项目下的`gradle.properties`中加入相应参数的值：
```text
VERSION_NAME=1.3.0
POM_GROUP_ID=com.coolerfall
POM_NAME=Http Download Manager
POM_ARTIFACT_ID=android-http-download-manager
POM_DESCRIPTION=An useful and effective http/https download manager for Android, support breakpoint downloading
POM_URL=https://github.com/Coolerfall/Android-HttpDownloadManager
POM_SCM_URL=https://github.com/Coolerfall/Android-HttpDownloadManager
POM_SCM_CONNECTION=scm:git@github.com:Coolerfall/Android-HttpDownloadManager.git
POM_SCM_DEV_CONNECTION=scm:git@github.com:Coolerfall/Android-HttpDownloadManager.git
POM_LICENCE_NAME=The Apache Software License, Version 2.0
POM_LICENCE_URL=http://www.apache.org/licenses/LICENSE-2.0.txt
POM_LICENCE_DIST=repo
POM_PACKAGING=jar

POM_DEVELOPER_ID=coolerfall
POM_DEVELOPER_NAME=Vincent Cheung
POM_INCEPTION_YEAR=2014
```

VERSION_NAME如果后面带有SNAPSHOT字符串将会提交到[snapshots][5]，这个不需要同步就可以下载jar以及源码等，如果不加则是发布release版本，这个就需要同步到maven center了。然后还需要在`C:/Users/xxx/.gradle/gradle.properties`中添加Sonatype账号、密码以及签名信息
```text
NEXUS_USERNAME=username
NEXUS_PASSWORD=password

signing.keyId=BB090204
signing.password=password
signing.secretKeyRingFile=C:/Users/xxx/AppData/Roaming/gnupg/secring.gpg
```
其中signing.password就是刚刚生成gpg时候的passphase。接下来就可以使用gralde提交项目：
```shell
$ %GRADLE_HOME%/bin/gradle uploadArchives
```
我使用的是AS最新版本，gradle home在AS目录下（如果不是请自行找到gradle home）
登陆到[Sonatype Nexus][2]，查看Staging Repositories，然后在搜索中过滤出你的Group Id，就可以看到你刚刚提交的项目，接下来发布release版本了，先close再Release:
![img][11]
然后在[Sonatype Issue][1]给工作人员回复，你已经成功发布，让他们帮你同步到maven中央库去：
![img][10]
最后就是等待工作人员给你同步，成功同步后，一般10几分钟左右就可以在[maven center][12]搜索到你的项目了，这样就可以在AS中使用gradle来添加依赖了。如果对[gradle-mvn-push.gradle][4]配置还有什么疑问，请参考我的项目：[Android-HttpDownloadManger][13]。
ps:如果创建issue的时候，工作人员回复`Only one JIRA issue per top-level groupId is necessary. You should already have all the necessary permissions to deploy and new artifacts to this groupId or to any sub-groups thanks to OSSRH-14919.`, 说明之前已经成功的release了一个项目，就不需要再创建一个新的issue了，直接将项目提交到[snapshots][5]，并进行closed和realease操作即可，过10分钟左右就可以在gradle中使用了。

[1]: https://issues.sonatype.org
[2]: https://oss.sonatype.org/
[3]: http://gpg4win.org/
[4]: https://raw.githubusercontent.com/chrisbanes/gradle-mvn-push/master/gradle-mvn-push.gradle
[5]: https://oss.sonatype.org/content/repositories/snapshots
[6]: /images/gradle-maven/new-issue.jpg
[7]: /images/gradle-maven/create_issue1.jpg
[8]: /images/gradle-maven/create_issue2.jpg
[9]: /images/gradle-maven/comments1.jpg
[10]: /images/gradle-maven/comments2.jpg
[11]: /images/gradle-maven/release.jpg
[12]: http://search.maven.org/
[13]: https://github.com/Coolerfall/Android-HttpDownloadManager
---
layout: post
title: "Mac OS下搭建APNS推送服务器(Apache+PHP)"
date: 2013-12-19 21:16:04 +0800
comments: true
categories: [iOS, APNS, 推送, PHP, Apache]
---
<h3>配置Apache+PHP</h3>
Mac OS自带Apache服务器和PHP，所以只需少许配置便可将它派上用场
```
    1）启动Apache服务器
    打开“终端（terminal）”，然后（注意，sudo需要的密码就是系统的root帐号密码）运行“sudo apachectl start”
    再输入帐号密码这样Apache就运行了。如此在浏览器中输入“http://localhost”就可以看到一个内容为“It works!”
    的页面。其位于“/Library/WebServer/Documents/”下，这就是Apache的默认根目录

    2）配置Apache支持PHP
    在终端中运行“sudo vi /etc/apache2/httpd.conf”，打开Apache的配置文件。
    找到“#LoadModule php5_module libexec/apache2/libphp5.so”，把前面的#号去掉，
    保存（在命令行输入:w）并退出vi（在命令行输入:q）。运行“sudo apachectl restart”，
    重启Apache，这样PHP就可以用了。
    
    3）测试PHP在Apache中运行是否正常
    运行“sudo cp /Library/WebServer/Documents/index.html.en /Library/WebServer/Documents/info.php”
    即在Apache的根目录下复制index.html.en文件并重命名为info.php。
    在终端中运行“sudo vi /Library/WebServer/Document/info.php”，这样就可以在vi中编辑info.php文件了。
    在“It’s works!”后面加上“<?php phpinfo(); ?>”，然后保存之。如此就可以在http://localhost/info.php中看到
    有关PHP的信息，比如10.8中内置PHP版本号是5.3.13。
```
<!--more-->
<h3>制作Pem文件</h3>
```
    1）打开keychain，选择左上Keychains区域中的login，再选择左下Category区域中的My Certificates，
    然后在右边找到项目使用的Push证书(注意：当有developement和production时一定要记住你导出的是哪个)
    后面做测试时无论是打包还是直接build在device的App都要使用对应的codesign.

    2）分别把Certificate和key的p12导出，命名为cert.p12和key.p12
    
    3）打开terminal运行以下代码：
        openssl pkcs12 -clcerts -nokeys -out cert.pem -in cert.p12
        openssl pkcs12 -nocerts -out key.pem -in key.p12
        openssl rsa -in key.pem -out key-noenc.pem
        cat cert.pem key-noenc.pem > apns.pem
    把最后生成的apns.pem保管好
```

<h3>编写PHP页面</h3>
在Apache根录下新建一个index.php文件，代码如下：
```
    <html>
	<head>
		<title>Push Simulator</title>
		<meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
		<meta id="viewport" name="viewport"content="width=320; initial-scale=1.0;maximum-scale=1.0;user-scalable=no;"/>
		<script language="javascript">
			function onTokenChange(value) {
				if (value == "other") {
					var token_other = document.getElementById("token_other");
					token_other.style.display = "inline"
				}
				else {
					var token_other = document.getElementById("token_other");
					token_other.style.display = "none"
				}
			};
		</script>
	</head>
	<body>
		<h2 style="border-bottom:red solid 1px;">Push Simulator</h2>
		<form action="a.php" method="post">
			<div>Badge number</div>
			<div>
				<input style="width:40px;" value="1" name="badgeNumber"></input>
			</div>
			<br />
	
			<div>Alert message</div>
			<div>
				<input style="width:150px;" value="新消息内容" name="msg"></input>
			</div>
			<br />
	
			<div>Link2</div>
			<div>
				<input style="width:450px;" value="video://vid=1605673&mid=10348504&columnId=117&channelId=1" name="link2"></input>
			</div>
			<br />

			<div>
				Token:
				<select name="token" id="tokenSelect" onchange="onTokenChange(this.options[this.options.selectedIndex].value);">
					<option value="4e52bb2a215643c0da52477177ba19793720ebc17ec5a38aa853a68a22ab7f93">token1</option>
					<option value="other">Other</option>
				</select>
				<input style="width:450px;display:none;" value="" id="token_other" name="token_other">
			</div>
			<div>
				Schema:
				<select name="schema">
					<option value="apns.pem">apns.pem</option>
				</select>
			</div>
			<br />
	
			<div>
				<input type="submit" value="发送"></input>
			</div>
		</form>
	</body>
    </html>
```
再在Apache根录下新建一个a.php文件(这个文件才是真正用来发推送的)，代码如下：
```
    <?php
	/*
	* 文件名称：Lib_Push_IPhone_Server.php
	* 摘    要：苹果PUSH发送类
	* 版    本：1.0
	*/
	class Lib_Push_IPhone_Server  {
		//发布版的server：gateway.push.apple.com 开发版：gateway.sandbox.push.apple.com
		private $apnsHost = 'gateway.sandbox.push.apple.com';
		private $apnsPort = '2195';
		private $sslPem = ''; //开发版本和发布版的pem不一样，用不同的证书生成
		private $passPhrase = 'sohu';
		private $apnsConnection = null ;
 
		public function __destruct(){
		//退出时，关闭到苹果服务器的连接
			$this->closeConnections();
			}
 
		//建立到苹果服务器的连接
		function connectToAPNS($pem) {
			$streamContext = stream_context_create();
			stream_context_set_option($streamContext, 'ssl', 'local_cert', $pem);
			$this->apnsConnection = stream_socket_client('ssl://'.$this->apnsHost.':'.$this->apnsPort, $error, $errorString, 60, STREAM_CLIENT_CONNECT, $streamContext);
			if($this->apnsConnection == false) {
				$this->Nslog("Failed to connect {".$error."} {".$errorString."}");
				$this->closeConnections();
			}
		}
 
		//发送消息到苹果
		function sendNotification($deviceToken, $message) {
			if(!$this->apnsConnection){
				$this->connectToAPNS();
			}
			$apnsMessage = chr(0) . chr(0) . chr(32) . pack('H*', str_replace(' ', '', $deviceToken)) . chr(0) . chr(strlen($message)) . $message;
			return fwrite($this->apnsConnection, $apnsMessage);
		}
 
		//关闭到苹果服务器的连接
		function closeConnections() {
			fclose($this->apnsConnection);
		}
 
		function NsLog($logmsg) {
			print date("Y-m-d H:i:s")."\t".$logmsg."\n";
		}
	}
    ?>

    <?php
        //=========================
        $schema			= $_POST['schema'];
        $badgeNumber	= $_POST['badgeNumber'];
        $msg 			= $_POST['msg'];
        $link2 			= $_POST['link2'];
        $token			= $_POST['token'];
        $token_other	= $_POST['token_other'];
        if ($token == 'other') {
            $token = $token_other;
        }

        $body = array();
        $body['aps'] = array('alert' => $msg);
        $body['aps']['badge'] = $badgeNumber;
        $body['url'] = $link2;
        $payload = json_encode($body);
        $push = new Lib_Push_IPhone_Server();

        $push->connectToAPNS($schema);
        $push->sendNotification($token, $payload);
        $push->closeConnections();
    ?>

    <title>iOS PushNotification Simulator</title>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
    <meta id="viewport" name="viewport"content="width=320; initial-scale=1.0;maximum-scale=1.0;user-scalable=no;"/>
    <body>
        <div><a href="javascript:history.back();"><<<<返回</a></div>
        <div>Schema</div>
        <div><?=$schema?></div>
        <br />
        
        <div>Badge Number</div>
        <div><?=$badgeNumber?></div>
        <br />
        
        <div>Alert Message</div>
        <div><?=$msg?></div>
        <br />

        <div>Link2</div>
        <div><?=$link2?></div>
        <br />
        
        <div>Token</div>
        <div><?=$token?></div>
        <br />
    </body>
```
再把apns.pem拷到Apache根目录下，然后重启Apche: sudo apachectl restart

<h3>配置index.php为默认首页</h3>
```
    找到
    <IfModule dir_module>
        DirectoryIndex index.html
    </IfModule>
    修改为
    <IfModule dir_module>
        DirectoryIndex index.php index.html
    </IfModule>
```
重启Apche: sudo apachectl restart

<h3>测试</h3>
```
    在浏览器地址栏里输入http://apache服务器地址，如:http://localhost，便能看到index.php页面。
    在这个页面填写相关参数便可以发送。
```

<h3>注意要点</h3>
```
    1）导出cert.p12和key.p12时的p12一定要是最后测试的App对应用的codesign
    即：developent的p12对应Debug模式的App，production的p12对应AppStore模式的App
    
    2）生成pem时一定要小心
    
    3）在index.php页面测试时，一定要选择对的schema（即pem，目前只有一种，
    我的项目中有多种：debug和debug-inhouse）填对token，
    这个token要以xcode输出的为准，因为它会变化
```

<h3>GOOD LUCK!!!</h3>

<h3>参考</h3>
<ol style="margin-top:-18px; padding-left:28px;">
	<li style="padding-bottom:10px;">
	    <a target="_blank" href="http://dancewithnet.com/2010/05/09/run-apache-php-mysql-in-mac-os-x/">在Mac OS X中配置Apache＋PHP＋MySQL</a><br/>
	</li>
	<li>
	    <a target="_blank" href="http://stackoverflow.com/questions/11536587/creating-pem-file-for-push-notification">Creating.pem file for push notification</a><br/>
	</li>
</ol>


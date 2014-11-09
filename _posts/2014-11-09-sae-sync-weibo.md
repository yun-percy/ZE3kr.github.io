---
layout: post
category: post

title: "本网站成功实现 RSS 自动同步新浪微博"
subtitle: "通过新浪 App Engine 以及 PHP 和新浪微博提供的 API，完全可以实现 RSS 数据自动同步到微博"
photo-file: screenshot/2014/1109.png

date: "2014-11-09 19:06:00+08:00"

tags: 
- SAE
- PHP
- 开发
- 新浪微博
---

首先，需要注册成为新浪微博开放平台的开发者，然后就可以获取到应用的 App key 和 App secret。

在 SAE 中创建 PHP 网页，然后添加以下内容在根目录下的 `oauth.php` 中
{% highlight php %}
<?php
header('Content-Type: text/html; charset=utf-8');
/* ------------------------- *
* -------------------------
POST 请求 */
function request_by_curl($remote_server,$post_string){
$ch = curl_init();
curl_setopt($ch,CURLOPT_URL,$remote_server);
curl_setopt($ch,CURLOPT_POSTFIELDS,'mypost='.$post_string);
curl_setopt($ch,CURLOPT_RETURNTRANSFER,true);
$data = curl_exec($ch);
curl_close($ch);
return $data;
}

/* ------------------------- *
* -------------------------
OAuth 2.0 */
if($_GET['auth'] == SAE_SECRETKEY)
{
    if(isset($_GET['code']))
    {
        $kv = new SaeKV();
		$kv -> init(); 

		$post_data = '&client_id=1963081010&client_secret=XXXXXXXXXXXXXXXXXXXXXXX&grant_type=authorization_code&redirect_uri=https%3A%2F%2Fze3kr.com%2Findex.html&code='.$_GET['code'];

		$json_string=request_by_curl('https://api.weibo.com/oauth2/access_token',$post_data); 
		$obj=json_decode($json_string);

		$ret = $kv->set('oauthkey', $obj->access_token);
    }
}
{% endhighlight %}

从 SAE 中找到你程序的 SAE_SECRETKEY ， 访问 `https://api.weibo.com/oauth2/authorize?client_id=1963081010&redirect_uri=[[ oauth.php 的网页 ]]?auth=[[ SAE_SECRETKEY ]]` 完成授权。

添加以下内容在根目录下的 `update.php` 中
{% highlight php %}
<?php
header('Content-Type: text/html; charset=utf-8');
/* ------------------------- *
* -------------------------
POST 请求 */
function request_by_curl($remote_server,$post_string){
$ch = curl_init();
curl_setopt($ch,CURLOPT_URL,$remote_server);
curl_setopt($ch,CURLOPT_POSTFIELDS,'mypost='.$post_string);
curl_setopt($ch,CURLOPT_RETURNTRANSFER,true);
$data = curl_exec($ch);
curl_close($ch);
return $data;
}

/* ------------------------- *
* -------------------------
开始 */
$doc = new DOMDocument;
$doc->load('http://t.ze3kr.com/rss'); 
$rssid = $doc->getElementsByTagName('guid');
$rsstitle = $doc->getElementsByTagName('title');

$kv = new SaeKV();
$kv -> init(); 

if ($ret = $kv->get('rss') == $rssid->item(0)->nodeValue )
{
    echo '没有新内容';
}

else
{
    echo '发布中<br />';

	$post_data = '&access_token='.$ret = $kv->get('oauthze3kr').'&status='.urlencode($rsstitle->item(1)->nodeValue).' '.urlencode($rssid->item(0)->nodeValue);

    $json_string=request_by_curl('https://api.weibo.com/2/statuses/update.json',$post_data); 
	$obj=json_decode($json_string);
    if(isset($obj->created_at))
    {
        echo '已发布';
    }

}
$ret = $kv->set('rss', $rssid->item(0)->nodeValue);
{% endhighlight %}

最后，不要忘记添加以下内容在 `config.yaml` 中
{% highlight yaml %}
cron:
    - description: 微博更新
      url: update.php
      schedule: every 10 mins
      timezone: Beijing
{% endhighlight %}

这就完成了
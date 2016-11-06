---
layout: post
title: "OAuth以及Android自定义Authenticator"
excerpt: "Custom written post descriptions are the way to go... if you're not lazy."
categories: android
tags: [android]
comments: true
share: true
comments: true
share: true
---


## 一，简述

使用AccountManager的作用在于，你可以使用android自带的账号系统，进一步可以使用账号同步。最直观的特点如下图

<figure class="half">
  <img src="{{ site.url }}/images/account1.jpg" alt="search screenshot">
   <img src="{{ site.url }}/images/account2.jpg" alt="search screenshot">
  <figcaption></figcaption>
</figure>

上图中，点击添加帐户，然后选择具体某个应用，接着会跳转到该应用的登陆界面。
你可以在手机的设置中添加帐号，然后账号信息将保存于此，这里我们一般都是保存token和账号，而不会保存密码，因为越狱过的手机可能使密码泄露。

另外讲到这个token，这是关于OAuth2.0的知识，下面就称呼为OAuth。



## 二，OAuth

首先得分清Authentication（认证）以及Authorization（授权），我们这里要讲的，OAuth关注的，就是Authorization（授权）。

token的作用是授权给第三方应用访问资源，使其不用每次请求都携带账号密码，而只需携带token，另外，通过token可以选择授权哪部分资源的权限。


### 介绍OAuth中的四个角色

+ 资源拥有者（resource owner）

  资源拥有者是拥有对资源进行授权的一方，经常就是永远账号密码的用户

+ 资源服务器（resource server）

  资源服务器持有某些资源，负责处理对这些资源的请求以及响应这些请求。

+ 客户端（client）

  这里的客户端就是第三方应用，它获得资源拥有者的授权后，就可以去请求某些受保护的资源

+ 授权服务器（authorization server）

  授权服务器负责发放token，当然，发放token的前提是客户端已经被授权。


### 四个角色的关系


<figure>
  <img src="{{ site.url }}/images/oauth2.png" alt="search screenshot">
  <figcaption>OAuth授权流程</figcaption>
</figure>




(A)  客户端（client）向资源拥有者请求授权


(B)  客户端（client）收到授权许可，


(C)  客户端（client）使用刚刚获得的授权去请求token。（此时你可能需要一次账号密码的登陆认证）


(D)  授权服务器（authorization server）检查客户端的认证请求，成功则返回一个授权的token给客户端。


(E)  客户端（client）使用刚刚获取的token向资源服务器（resource server）请求资源


(F)  资源服务器（resource server）检查token的有效性，有效则返回资源给客户端。


## 三，自定义Authenticator

如果使用android手机的账号系统，app的启动activity必然不会是登陆界面，而应该直接就是app的主页（我们此处称为MainActivity），


使用android账号系统的app，生成token有两种情况，

+ app启动进入MainActivity后请求token，如果获取失败则挑战到LoginActivity，输入账号密码重新请求token。

+ 通过设置－>添加帐户的方式，然后跳转到登陆界面。


如果想让你的应用使用android的账号系统，你需要做如下几步

+ 1，自定义AccountAuthenticator（继承重写AbstractAccountAuthenticator），它负责获取token，以及获取token失败时，跳转到登陆界面

+ 2，定义AccountService，它会与你自定义的AccountAuthenticator绑定，主要是为了实现“设置－>添加帐户”可以添加帐户，另外，定义该Service时需要指定一个account-authenticator资源

+ 3，定义继承自AccountAuthenticatorActivity的LoginActivity


此处不一一贴出代码，只记录一下需要注意的代码片段


点击“设置－>添加帐户”跳转到登陆界面，是通过自定义AccountAuthenticator中的addAccount方法，如下

{% highlight java %}

@Override
public Bundle addAccount(AccountAuthenticatorResponse response, String accountType, String authTokenType, String[] requiredFeatures, Bundle options) throws NetworkErrorException {
    final Intent intent = new Intent(context, LoginActivity.class);
    intent.putExtra(ARG_ACCOUNT_TYPE, accountType);
    intent.putExtra(ARG_AUTH_TYPE, authTokenType);
    intent.putExtra(ARG_IS_ADDING_NEW_ACCOUNT, true);
    intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);
    final Bundle bundle = new Bundle();
    bundle.putParcelable(AccountManager.KEY_INTENT, intent);
    return bundle;
}


{% endhighlight %}

在你的MainActivity中尝试获取token，是通过一下代码实现，注意，请求和结果的返回是异步的，所以该过程不能再UI线程中执行。下面这段代码会触发自定义AccountAuthenticator中的getAuthToken方法的执行（后面会讲）


{% highlight java %}

final AccountManagerFuture<Bundle> future = manager.getAuthToken(account, ACCOUNT_TYPE, null, (Activity)context, null, null);

      try {
          Bundle result = future.getResult();
          return result.getString(AccountManager.KEY_AUTHTOKEN);
      } catch (AccountsException e) {
          Log.e(TAG, "Auth token lookup failed", e);
          return null;
      } catch (IOException e) {
          Log.e(TAG, "Auth token lookup failed", e);
          return null;
      }
}

{% endhighlight %}

上面的代码，MainActivity试图获取本地保存的token时，可能获取成功，成功则直接返回token，也可能获取失败，失败则跳转到登陆界面，登陆界面输入账号密码重新向服务器请求token。该过程由自定义AccountAuthenticator中的getAuthToken方法实现。


{% highlight java %}


@Override
public Bundle getAuthToken(AccountAuthenticatorResponse response, Account account, String authTokenType, Bundle options) throws NetworkErrorException {
    final AccountManager am = AccountManager.get(context);
    //第一步：试图获取本地的token
    String authToken = am.peekAuthToken(account, authTokenType);
    //第二步：获取失败的话，检查是否密码，有的的话直接使用账号密码请求token，这样就不用跳转到登陆界面
    if (TextUtils.isEmpty(authToken)) {
        final String password = am.getPassword(account);
        if (password != null) {
            Github github = new GithubImpl(context);
            try {
                github.createToken(account.name,password);
            } catch (GithubError githubError) {
                githubError.printStackTrace();
                authToken = "";
            } catch (AuthError authError) {
                authError.printStackTrace();
                authToken = "";
            }
        }
    }
    //第三步：第二步失败了，只能跳转到登陆界面了
    if (!TextUtils.isEmpty(authToken)) {
        final Bundle result = new Bundle();
        result.putString(AccountManager.KEY_ACCOUNT_NAME, account.name);
        result.putString(AccountManager.KEY_ACCOUNT_TYPE, account.type);
        result.putString(AccountManager.KEY_AUTHTOKEN, authToken);
        return result;
    }
    // If we get here, then we couldn't access the user's password - so we
    // need to re-prompt them for their credentials. We do that by creating
    // an intent to display our AuthenticatorActivity.
    final Intent intent = new Intent(context, LoginActivity.class);
    intent.putExtra(AccountManager.KEY_ACCOUNT_AUTHENTICATOR_RESPONSE, response);
    intent.putExtra(ARG_ACCOUNT_TYPE, account.type);
    intent.putExtra(ARG_AUTH_TYPE, authTokenType);
    final Bundle bundle = new Bundle();
    bundle.putParcelable(AccountManager.KEY_INTENT, intent);
    return bundle;
}

{% endhighlight %}



参考链接

[http://blog.udinic.com/2013/04/24/write-your-own-android-authenticator/](http://blog.udinic.com/2013/04/24/write-your-own-android-authenticator/)

[http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

[http://tools.ietf.org/html/rfc6749](http://tools.ietf.org/html/rfc6749)

另外，我已经将其应用于我的项目

[https://github.com/Leaking/GithubKnife](https://github.com/Leaking/GithubKnife)

title: 一个由于环境变量产生的bug
date: 2016-03-07 01:37:18
tags: node.js
---
## 起因

事情的起因主要是一个 Java 的同学通过 `sudo -u admin` 来执行基于 **Egg** 实现的服务挂了，然后在各种调试之后，终于找到了问题的所在。

Egg 会在用户路径下记录 Log 文件，而这个用户路径是由 `process.env.HOME` 获取到的。很神奇的一点是，通过它获取到的用户路径依然是原用户的路径，而不是期望中的 `/home/admin`，于是就出现了权限问题，导致写入失败。

也就是类似下面的步骤：

```
$ sudo -u admin node
> var fs = require('fs')
undefined
> var path = require('path')
undefined
> fs.writeFile(path.join(process.env.HOME, '/log'), 'test')
undefined
> Error: EACCES, open '/home/yilin.fyl/log'
  at Error (native)
```

## process.env

  那么 `process.env`它的值是怎么得到的呢？(･ω´･ )是不是取值的时候有什么奇怪的设定呢？

  接着我找到了 node.cc 里初始化 node 里 process 的函数--[SetupProcessObject函数实现](https://github.com/nodejs/node/blob/v5.x/src/node.cc#L2775)，以及 `process.env`的[定义](https://github.com/nodejs/node/blob/v5.x/src/node.cc#L2939):

```
 // create process.env
  Local<ObjectTemplate> process_env_template =
      ObjectTemplate::New(env->isolate());
  process_env_template->SetNamedPropertyHandler(EnvGetter,
                                                EnvSetter,
                                                EnvQuery,
                                                EnvDeleter,
                                                EnvEnumerator,
                                                env->as_external());
  Local<Object> process_env = process_env_template->NewInstance();
  process->Set(env->env_string(), process_env);
```

> 顺带一提，SetNamedPropertyHandler的定义：
void SetNamedPropertyHandler	(	NamedPropertyGetterCallback 	getter,
NamedPropertySetterCallback 	setter = 0,
NamedPropertyQueryCallback 	query = 0,
NamedPropertyDeleterCallback 	deleter = 0,
NamedPropertyEnumeratorCallback 	enumerator = 0,
Handle< Value > 	data = Handle< Value >()
)		
所以上面的 `process_env_template->SetNamedPropertyHandler` 相当于创建了一个拥有 getter, setter, query...方法的对象

(% ﾟーﾟ)那么再来看 `EnvGetter` 的[实现](https://github.com/nodejs/node/blob/v5.x/src/node.cc#L2451):

```
static void EnvGetter(Local<String> property,
                      const PropertyCallbackInfo<Value>& info) {
  Isolate* isolate = info.GetIsolate();
#ifdef __POSIX__
  node::Utf8Value key(isolate, property);
  const char* val = getenv(*key); // 根据 key 获取环境变量
  if (val) {
    return info.GetReturnValue().Set(String::NewFromUtf8(isolate, val));
  }
#else  // _WIN32 下的获取环境变量的有兴趣的可以自己去看下
  ...
#endif
}

```

  最后定位到 `char* getenv (const char* name)` 这个函数， 也就是在 UNIX 下的 C 语言通过 [getenv()](http://www.cplusplus.com/reference/cstdlib/getenv/) 这个方法来获取当前的环境变量。

  也就是说 `process.env` 拿的直接是当前系统的环境变量，( ˘･з･)那为什么用 `sudo -u admin xxx` 启动后的环境变量却不是 admin 下环境变量的呢 ？

## sudo -u

  接下来看了下 `sudo -h` 里面对 `-u` 的解释：

```
  -u user       run command (or edit file) as specified user //指定用户运行命令（或编辑文件）
```

  那么，问题又来了( ˘•ω•˘ )，这个命令(`sudo -u admin`)和真正切到该用户下(`sudo su admin`)去执行命令有什么区别呢？是不是因为这些区别所以才引起一些奇怪的权限问题呢？

  首先先来试下 `sudo -u xxx`，由于本地电脑上没有其他的用户，就直接用 `root` 代替了。

```
  ➜  ~  sudo -u root node
  > process.env
  {
    HOME: '/Users/minary',
    LOGNAME: 'root',
    USER: 'root',
    USERNAME: 'root',
    SUDO_COMMAND: '/Users/Minary/.nvm/versions/node/v4.2.1/bin/node',
    SUDO_USER: 'minary',
    SUDO_UID: '501',
    SUDO_GID: '20'
  }
```
  可以看到这里的 `HOME` 字段并不是 `/var/root` 而是 `/Users/当前用户`，用了 `root` 的权限来执行 `node`, 但是它的类似 `HOME` 字段的环境变量仍未被改变，还是在当前状态，(´-ω-｀)这是为什么呢？

  我们可以执行 `sudo -l` 来看当前的 sudo 配置或者直接查看 `/etc/sudoers`：

```
  ➜  ~  sudo -l   
  Matching Defaults entries for minary on this host:
      env_reset, env_keep+=BLOCKSIZE, env_keep+="COLORFGBG COLORTERM", env_keep+=__CF_USER_TEXT_ENCODING, env_keep+="CHARSET LANG LANGUAGE
      LC_ALL LC_COLLATE LC_CTYPE", env_keep+="LC_MESSAGES LC_MONETARY LC_NUMERIC LC_TIME", env_keep+="LINES COLUMNS", env_keep+=LSCOLORS,
      env_keep+=SSH_AUTH_SOCK, env_keep+=TZ, env_keep+="DISPLAY XAUTHORIZATION XAUTHORITY", env_keep+="EDITOR VISUAL", env_keep+="HOME MAIL",
      lecture_file=/etc/sudo_lecture

  User minary may run the following commands on this host:
      (ALL) ALL

```
  可以看到这里有很多的 `env_keep`，也就是当使用 `sudo` 时，这些环境变量保持不变，所以如果我们在配置里把 `env_keep+="HOME MAIL"` 给注释了再用上面的方法执行的时候就不应该依旧是 `HOME: '/Users/minary'` 了，来验证下吧( ¯•ω•¯ )：

```
sudo -l             
Matching Defaults entries for minary on this host:
    env_reset, env_keep+=BLOCKSIZE, env_keep+="COLORFGBG COLORTERM", env_keep+=__CF_USER_TEXT_ENCODING, env_keep+="CHARSET LANG LANGUAGE
    LC_ALL LC_COLLATE LC_CTYPE", env_keep+="LC_MESSAGES LC_MONETARY LC_NUMERIC LC_TIME", env_keep+="LINES COLUMNS", env_keep+=LSCOLORS,
    env_keep+=SSH_AUTH_SOCK, env_keep+=TZ, env_keep+="DISPLAY XAUTHORIZATION XAUTHORITY", env_keep+="EDITOR VISUAL",
    lecture_file=/etc/sudo_lecture

User minary may run the following commands on this host:
    (ALL) ALL

```
  可以看到上面已经没有了 `env_keep+="HOME MAIL"`，那么结果怎么样呢？

```
➜  ~  sudo -u root node   
> process.env
{
  LOGNAME: 'root',
  USER: 'root',
  USERNAME: 'root',
  HOME: '/var/root',
  SUDO_COMMAND: '/Users/Minary/.nvm/versions/node/v4.2.1/bin/node',
  SUDO_USER: 'minary',
  SUDO_UID: '501',
  SUDO_GID: '20'
}
>

```
  ヽ(✿ﾟ▽ﾟ)ノ `HOME` 变成了 `/var/root`，验证成功！

  但是在内部的服务器上，`/etc/sudoers` 被单独配置过，怎么可以不修改这个配置但依旧获取到 `root` 的环境变量呢？
  其实也很简单，直接用 `sudo -u root -i xxx` 执行就可以了, 来看下 `sudo -h` 里面对 `-i` 的解释：

```
-i [command]  run a login shell as target user
```

  相当于直接把整个环境切到了 admin 下去执行，所以获得的环境变量也就肯定是 root 下的了，再来做下验证：

```
➜  ~  sudo -u root -i node       
Password:
> process.env
{
  USER: 'root',
  SUDO_USER: 'minary',
  SUDO_UID: '501',
  USERNAME: 'root',
  HOME: '/var/root',
  LOGNAME: 'root',
  SUDO_GID: '20',
  _: '/Users/Minary/.nvm/versions/node/v4.2.1/bin/node'
}

```

  可以看到这里的 `HOME` 也变成了 `/var/root`, 切换完成～ヾ(✿❛ω❛ฺฺ）ﾉ，撒花～

## os.homedir

  那么 node 下有没有直接可以 *获取当前用户的 HOME 路径* 的方法呢？
  ヽ(•̀ω•́ )ゝ这时候突然想到了 node.js 中还有个方法，也就是 `os.homedir()`,
  看了下官网对这个函数的[定义](https://nodejs.org/api/os.html#os_os_homedir):

    os.homedir() #Returns the home directory of the current user.

  翻译过来是获取的是当前用户的 home 路径，感觉这个靠谱像是我需要的，(｡˘•ε•˘｡) 所以迫不及待的试了下：

```
➜  ~  sudo -u root node
> os.homedir()
'/Users/minary'
>

```
  然后...=͟͟͞͞( •̀д•́)，教练，这和想象中的不一样啊，于是...又去翻了 `os.homedir()` 的实现：

```
// node/blob/master/lib/os.js

const binding = process.binding('os');
exports.homedir = binding.getHomeDirectory;

```

```
// node/blob/master/src/node_os.cc

static void GetHomeDirectory(const FunctionCallbackInfo<Value>& args) {
  Environment* env = Environment::GetCurrent(args);
  char buf[PATH_MAX];

  size_t len = sizeof(buf);
  const int err = uv_os_homedir(buf, &len); //通过调用 uv_os_homedir 方法来将 HOME 赋值给 buf

  if (err) {
    return env->ThrowUVException(err, "uv_os_homedir");
  }

  Local<String> home = String::NewFromUtf8(env->isolate(), buf, String::kNormalString, len);
  args.GetReturnValue().Set(home);
}
```

接下来再来看看 libuv 中 `uv_os_homedir` 的[实现](https://github.com/libuv/libuv/blob/v1.x/src/unix/core.c#L1016)的时候突然发现了这个:

```
/* Check if the HOME environment variable is set first */
buf = getenv("HOME");

...

/* HOME is not set, so call getpwuid() */
 initsize = sysconf(_SC_GETPW_R_SIZE_MAX);
...

```

(ﾟдﾟ≡ﾟдﾟ), 为什么，为什么第一步还是去拿当前的环境变量里的`HOME`！当拿不到的时候才去获取当前用户的`HOME`路径, 为什么要故意加一步？

然后我开始找这个特性的pr，果然看到了关于这个的[讨论](https://github.com/libuv/libuv/pull/350#issuecomment-102701904)

> I wonder if this should check $HOME first? It's something of a UNIX tradition to be able to change your home directory on the fly. A problem with that is that getenv("HOME") is not MT-safe.

担心某些 UNIX 系统在运行的时候会把环境变量的`HOME`路径给改掉了，会导致出现一系列的不安全的问题。

咦，那不是和 `process.env.HOME` 功能基本一样了么？然后找到了这个功能诞生前的[讨论](https://github.com/nodejs/node/pull/1670), 大致的意思就是 `os.homedir()` 是 `process.env.HOME` 的升级版

## 总结

  所以，最后的总结就是在执行的时候注意这些细节，最好的执行方法还是直接用 `sudo su xxx` 去切换用户后去执行比较好，当然还是可以选择 `sudo -u xxx -i` 去执行，但是不推荐直接使用 `sudo -u xxx` 的方法来启动服务, 会导致一系列的因为环境变量引起的问题。

  在 node.js 层上，也最好用`os.homedir()`来替代`process.env.HOME`，因为当遇到 `getenv('PATH')`为空的时候，`os.homedir()`会使用当前的运行用户的目录，降低bug率。

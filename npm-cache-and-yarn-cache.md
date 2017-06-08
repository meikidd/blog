# npm 和 yarn 缓存策略对比

前几天 npm@5 发布，其中最大的改进是对缓存策略的更新，本文对比一下 npm@5 和 yarn 的缓存策略。

## npm

### 缓存命令
[npm cache](https://docs.npmjs.com/cli/cache) 提供了三个命令，分别是`npm cache add`, `npm cache clean`, `npm cache verify`。

#### `npm cache add`
官方解释说这个命令主要是 npm 内部使用，但是也可以用来手动给一个指定的 package 添加缓存。(This command is primarily intended to be used internally by npm, but it can provide a way to add data to the local installation cache explicitly.)

#### `npm cache clean`
删除缓存目录下的所有数据。从 npm@5 开始，为了保证缓存数据的有效性和完整性，需要加上 `--force` 参数。

#### `npm cache verify`
验证缓存数据的有效性和完整性，清理垃圾数据。

### 缓存策略
npm 的缓存目录是通过 cache 变量指定的，一般默认是在 `~/.npm` 文件夹（Windows 系统在 `%AppData%/npm-cache` 文件夹），可以执行下面的命令查看
```
npm config get cache
```

在 npm@5 以前，每个缓存的模块在 `~/.npm` 文件夹中以模块名的形式直接存储，例如 koa 模块存储在 `~/.npm/koa` 文件夹中。而 npm@5 版本开始，数据存储在 `~/.npm/_cacache` 中，并且不是以模块名直接存放。

npm 的缓存是使用 [pacote](https://www.npmjs.com/package/pacote) 模块进行下载和管理，基于 [cacache](https://www.npmjs.com/package/cacache) 缓存存储。由于 npm 会维护缓存数据的完整性，一旦数据发生错误，就回重新获取。因此不推荐手动清理缓存，除非需要释放磁盘空间，这也是要强制加上 `--force` 参数的原因。

目前没有提供用户自己管理缓存数据的命令，随着你不断安装新的模块，缓存数据也会越来越多，因为 npm 不会自己删除数据。


### 离线安装
npm 提供了离线安装模式，使用 `--offline`, `--prefer-offline`, `--prefer-online` 可以指定离线模式。

#### `--prefer-offline` / `--prefer-online`
“离线优先/网络优先”模式。

- 如果设置为 `--prefer-offline` 则优先使用缓存数据，如果没有匹配的缓存数据，则从远程仓库下载。
- 如果设置为 `--prefer-online` 则优先使用网络数据，忽略缓存数据，这种模式可以及时获取最新的模块。

#### `--offline`
完全离线模式，安装过程不需要网络，直接使用匹配的缓存数据，一旦缓存数据不存在，则安装失败。


## yarn

### 缓存命令
[yarn cache](https://yarnpkg.com/en/docs/cli/cache) 提供了三个命令，分别是`yarn cache ls`, `yarn cache dir`, `yarn cache clean`。

#### `yarn cache ls`

列出当前缓存的包列表。

#### `yarn cache dir`

显示缓存数据的目录。

#### `yarn cache clean`

清除所有缓存数据。

### 缓存策略
官方文档没有详细介绍缓存策略，不过进入缓存目录也可以看出一些端倪。在 `~/Library/Caches/Yarn` 文件夹中，每个缓存的模块被存放在独立的文件夹，文件夹名称包含了模块名称、版本号等信息。

### 离线安装
yarn 默认会使用 “prefer-online” 的模式，也就是先尝试从远程仓库下载，如果连接失败则尝试从缓存读取。yarn 也提供了 `--offline` 参数，即通过 `yarn add --offline` 安装依赖。

另外 yarn 还支持配置离线镜像，通过以下命令设置离线缓存仓库。具体细节参照官方博客[《Running Yarn offline》](https://yarnpkg.com/blog/2016/11/24/offline-mirror/)。
```
yarn config set yarn-offline-mirror ./npm-packages-offline-cache
```

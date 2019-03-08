## 站点结构

| 文件 / 目录                                       | 描述                                                         |
| ------------------------------------------------- | ------------------------------------------------------------ |
| `_config.yml`                                     | 保存[配置](https://www.jekyll.com.cn/docs/configuration/)数据。很多配置选项都会直接从命令行中进行设置，但是如果你把那些配置写在这儿，你就不用非要去记住那些命令了。 |
| `_drafts`                                         | drafts 是未发布的文章。这些文件的格式中都没有 `title.MARKUP` 数据。学习如何使用 [drafts](https://www.jekyll.com.cn/docs/drafts/). |
| `_includes`                                       | 你可以加载这些包含部分到你的布局或者文章中以方便重用。可以用这个标签 `{% include file.ext %}` 来把文件 `_includes/file.ext` 包含进来。 |
| `_layouts`                                        | layouts 是包裹在文章外部的模板。布局可以在 [YAML 头信息](https://www.jekyll.com.cn/docs/frontmatter/)中根据不同文章进行选择。 这将在下一个部分进行介绍。标签`{{ content }}` 可以将content插入页面中。 |
| `_posts`                                          | 这里放的就是你的文章了。文件格式很重要，必须要符合:`YEAR-MONTH-DAY-title.MARKUP`。 The [permalinks](https://www.jekyll.com.cn/docs/permalinks/) 可以在文章中自己定制，但是数据和标记语言都是根据文件名来确定的。 |
| `_data`                                           | 格式化好的站点数据将放在这里，jekyll引擎将自动加载此路径中的所有yaml文件（后缀为.yml或.yaml）。 如果路径中存在`members.yml`文件，则可以通过 `site.data.members`访问该文件的内容 |
| `_site`                                           | 一旦 Jekyll 完成转换，就会将生成的页面放在这里（默认）。最好将这个目录放进你的 `.gitignore` 文件中。 |
| `index.html`和 `HTML`, `Markdown`, `Textile` 文件 | 如果这些文件中包含 [YAML 头信息](https://www.jekyll.com.cn/docs/frontmatter/) 部分，Jekyll 就会自动将它们进行转换。当然，其他的如 `.html`， `.markdown`， `.md`，或者 `.textile` 等在你的站点根目录下或者不是以上提到的目录中的文件也会被转换。 |
| 其他文件/文件夹                                   | 其他一些未被提及的目录和文件如 `css` 还有 `images` 文件夹，`favicon.ico` 等文件都将被完全拷贝到生成的 site 中。 这里有一些[使用 Jekyll 的站点](https://www.jekyll.com.cn/docs/sites/)，如果你感兴趣就来看看吧。 |

## 配置设置

### 全局配置

下表中列举了所有 Jekyll 可用的设置，和多种多样的 `配置项` (配置文件中) 及 `参数` (命令行中)。

| 设置                                                         | 配置项 和 参数                            |
| ------------------------------------------------------------ | ----------------------------------------- |
| **Site Source**改变 Jekyll 读取文件的目录                    | `source: DIR``-s, --source DIR`           |
| **Site Destination**改变 Jekyll 写入文件的目录               | `destination: DIR``-d, --destination DIR` |
| **Safe**禁用 [自定义插件](https://www.jekyll.com.cn/docs/plugins/)。 | `safe: BOOL``--safe`                      |
| **Exclude**转换时排除某些文件夹或文件。被排除的文件或文件夹的路径是相对于网站源码目录的，源码目录以外不受影响。 | `exclude: [DIR, FILE, ...]`               |
| **Include**转换时强制包含某些文件、文件夹。 `.htaccess` 是个典型的例子，因为默认排除 .（dot，英文中的句号） 开头的文件。 | `include: [DIR, FILE, ...]`               |
| **Time Zone**设置时区，这个设置作用于 `TZ` 变量， Ruby 用它来处理日期和时间。 [IANA Time Zone Database](http://en.wikipedia.org/wiki/Tz_database) 里边的都有效，比如 `America/New_York` 。默认值为操作系统的时区。 | `timezone: TIMEZONE`                      |
| **Encoding**设置文件的编码，仅 Ruby 1.9 以上可用。默认值为 nil ，使用 Ruby 默认的 `ASCII-8BIT`。可以用命令`ruby -e 'puts Encoding::list.join("\n")'` 查看 Ruby 可用的编码。 | `encoding: ENCODING`                      |

### 编译选项

| 设置                                                         | 配置项 和 参数                        |
| ------------------------------------------------------------ | ------------------------------------- |
| **Regeneration**允许文件修改时自动重新生成网站。             | `-w, --watch`                         |
| **Configuration**手动设置配置文件并替代`_config.yml`，可设置多个，且后边的设置会覆盖前边的。 | `--config FILE1[,FILE2,...]`          |
| **Drafts**处理草稿                                           | `--drafts`                            |
| **Future**用将来的日期发布文章                               | `future: BOOL``--future`              |
| **LSI**为相关文章生成索引                                    | `lsi: BOOL``--lsi`                    |
| **Limit Posts**限制文章的数量                                | `limit_posts: NUM``--limit_posts NUM` |

### 服务选项

除了下边的选项， `serve` 命令还可以接收 `build` 的选项，当运行网站服务之前的编译时候使用。

| 设置                                      | 配置项 和 参数                    |
| ----------------------------------------- | --------------------------------- |
| **Local Server Port**监听所给的端口       | `port: PORT``--port PORT`         |
| **Local Server Hostname**监听所给的主机名 | `host: HOSTNAME``--host HOSTNAME` |
| **Base URL**网站的根路径                  | `baseurl: URL``--baseurl URL`     |
| **Detach**从终端命令行中分离出来          | `detach: BOOL``-B, --detach`      |
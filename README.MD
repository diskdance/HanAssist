# HanAssist ![Linter](https://github.com/diskdance/HanAssist/actions/workflows/lint.yml/badge.svg)
[代码文档](https://diskdance.github.io/HanAssist/)

HanAssist 是帮助中文维基百科及其他 MediaWiki 网站上的用户脚本和小工具更优雅地处理中文变体消息的实用程序。

本程序的目标是取代``wgULS()``和``wgUVS()``小工具。

```javascript
HanAssist.localize( { hans: '一天一苹果，医生远离我。', hant: '一天一蘋果，醫生遠離我。' } );
// => 界面语言为简中：“一天一苹果，医生远离我。”；繁中：“一天一蘋果，醫生遠離我。”

HanAssist.vary( { hans: '一天一苹果，医生远离我。', hant: '一天一蘋果，醫生遠離我。' } );
// => 页面变体为简中：“一天一苹果，医生远离我。”；繁中：“一天一蘋果，醫生遠離我。”
```

## 为什么使用 HanAssist？
``wgULS()``和``wgUVS()``几乎是中文维基百科每个用户脚本和小工具都要使用到的功能。它们极大方便了中文变体消息的处理，但随着 JavaScript 的发展，它们的问题也逐渐显现：

1. 仍然使用旧式的``wgXXX``类名称，污染全局空间。现今 MediaWiki 中的此类变量全部通过``mw.config.get()``获取，但这两个函数并未也无法跟进。
2. 使用意义不明的缩写，影响代码可读性。
3. 参数列表过长，影响阅读。设计良好的 JavaScript 函数[最多仅应包含两个参数](https://github.com/ryanmcdermott/clean-code-javascript#function-arguments-2-or-fewer-ideally)。
   设想一下，如果像这样调用``wgULS()``：
   ```javascript
   wgULS( undefined, undefined, '显示%s的用户日志', '顯示%s的使用者日誌', '顯示%s的用戶日誌' );
   ```
   难道不会使代码非常难以阅读及维护？
4. 并非类型安全。``wgULS()``和``wgUVS()``允许任何类型的参数传入，这可能会导致非预期的行为发生，并且使得代码难以维护。
5. 没有代码文档。这使得不了解这些函数的人必须通过直接阅读代码来确定它们的用法。

为了解决这些问题，我们制作了 HanAssist。它具有如下优点：

1. 仅占用``HanAssist``一个全局名称，提供原小工具的全部功能；
2. 采用命名参数语法，显著提升代码可读性；
3. 兼容旧版``wgULS``、``wgUVS``语法，方便迁移；
4. 完善的代码文档及类型定义；
5. 支持批量转译消息，在代码大量依赖中文变体消息时可极大减少代码复杂程度。

## 用法
``HanAssist.localize()``与``HanAssist.vary()``接受一种称作“候选消息列表”的对象，其原型如下所示：
```typescript
interface Candidates {
	zh?: string, // 中文（不转换）消息
	hans?: string, // 简体中文消息
	hant?: string, // 繁體中文消息
	cn?: string, // 大陆简体消息
	tw?: string, // 台灣正體消息
	hk?: string, // 香港繁體消息
	mo?: string, // 澳門繁體消息
	my?: string, // 大马简体消息
	sg?: string, // 新加坡简体消息
	other?: string // 非中文语言（如英语）消息
}
```

两者的用法：
```javascript
// 等同于 wgULS( '一天一苹果，医生远离我。', '一天一蘋果，醫生遠離我。' );
HanAssist.localize( '一天一苹果，医生远离我。', '一天一蘋果，醫生遠離我。' );
HanAssist.localize( { hans: '一天一苹果，医生远离我。', hant: '一天一蘋果，醫生遠離我。' } );

// 等同于 wgULS( undefined, undefined, 'IP用户', 'IP使用者', 'IP用戶' );
HanAssist.localize( { cn: 'IP用户', tw: 'IP使用者', hk: 'IP用戶' } );

// 等同于 wgUVS( '一天一苹果，医生远离我。', '一天一蘋果，醫生遠離我。' );
HanAssist.vary( '一天一苹果，医生远离我。', '一天一蘋果，醫生遠離我。' );
HanAssist.vary( { hans: '一天一苹果，医生远离我。', hant: '一天一蘋果，醫生遠離我。' } );

// 配合占位符号使用
mw.format(
    HanAssist.localize( { hans: '页面$2的修订版本$1', hant: '頁面$2的修訂版本$1' } ),
    '123456',
    'Apple'
); // => 界面语言为简中：“页面Apple的修订版本123456”；繁中：“頁面Apple的修訂版本123456”
```

批量转译消息：
```javascript
HanAssist.parse( {
	'article': { hans: '条目', hant: '條目' },
	'category': { hans: '分类', hant: '分類' },
	'categories': { hans: '分类', hant: '分類' },
	'image': { hans: '文件', hant: '檔案' },
	'images': { hans: '文件', hant: '檔案' },
	'minute': '分',
	'minutes': '分',
	'second': '秒',
	'seconds': '秒',
	'week': '周',
	'weeks': '周',
	'search': { hans: '搜索', hant: '搜尋' },
	'SearchHint': { hans: '搜索包含$1的页面', hant: '搜尋包含$1的頁面' },
	'web': { hans: '站点', hant: '站點' },
} ); // => { 'article': '条目', 'category': '分类', 'categories': '分类', ... }

// 推荐配合 mw.messages 使用
mw.messages.set( HanAssist.parse( {
	'article': { hans: '条目', hant: '條目' },
	'category': { hans: '分类', hant: '分類' },
	'categories': { hans: '分类', hant: '分類' },
	'image': { hans: '文件', hant: '檔案' },
	'images': { hans: '文件', hant: '檔案' },
	'minute': '分',
	'minutes': '分',
	'second': '秒',
	'seconds': '秒',
	'week': '周',
	'weeks': '周',
	'search': { hans: '搜索', hant: '搜尋' },
	'SearchHint': { hans: '搜索包含$1的页面', hant: '搜尋包含$1的頁面' },
	'web': { hans: '站点', hant: '站點' },
} ) );

mw.msg( 'categories' ); // => 界面语言为简中：“分类”；繁中：“分類”
mw.msg( 'SearchHint', 'Apple' ); // => 界面语言为简中：“搜索包含Apple的页面”；繁中：“搜尋包含Apple的頁面”
// 其他用法详见 MediaWiki 文档
```

## 局限
在软件领域，[没有银弹](https://zh.wikipedia.org/wiki/%E6%B2%A1%E6%9C%89%E9%93%B6%E5%BC%B9)，因此 HanAssist 也并非完美。在一些使用场景下，您应该使用其他更合适的工具而非 HanAssist。

如果您的小工具或用户脚本需要多国语言支持，而非仅限于中文（汉语）一种，请考虑使用其他支持多语言处理的库，如 [``jQuery.i18n``](https://github.com/wikimedia/jquery.i18n)。

## 授权 & 致谢
本程序由 Diskdance 等制作，采用 [3-Clause BSD License](./LICENSE) 授权协议。

特别感谢中文维基百科用户 [Роу Уилсон Фредериск Холм](https://zh.wikipedia.org/wiki/User:Роу_Уилсон_Фредериск_Холм) 和 [SunAfterRain](https://zh.wikipedia.org/wiki/User:SunAfterRain) 在制作过程中给予的支持，尤其是关于 TypeScript 方面问题的解答。
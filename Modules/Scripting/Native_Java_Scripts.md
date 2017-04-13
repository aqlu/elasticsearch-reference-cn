# 原生（Java）脚本

有时候`groovy`和[表达式](./Lucene_Expressions_Language.md)是不够的。这种时候可以实现原生脚本。

实现原生脚本，最好的方法是编写一个插件并安装它。如何编写一个插件并让Elasticsearch正确的加载它，在[插件文档](https://www.elastic.co/guide/en/elasticsearch/plugins/5.3/plugin-authors.html)章节有更多的介绍。

要注册一个新的脚本，你需要通过实现`NativeScriptFactory`来构建它。可以通过继承`AbstractExecutableScript`或`AbstractSearchScript`。 另外一种可能是最有用的方式，通过扩展`AbstractLongSearchScript`和`AbstractDoubleSearchScript`。最后，你还需要通过实现`ScriptPlugin`接口来注册原生脚本插件。

下面展示的是如何在一个类中来完成所有事情，它会是这样的：

```java
public class MyNativeScriptPlugin extends Plugin implements ScriptPlugin {

    @Override
    public List<NativeScriptFactory> getNativeScripts() {
        return Collections.singletonList(new MyNativeScriptFactory());
    }

    public static class MyNativeScriptFactory implements NativeScriptFactory {
        @Override
        public ExecutableScript newScript(@Nullable Map<String, Object> params) {
            return new MyNativeScript();
        }
        @Override
        public boolean needsScores() {
            return false;
        }
        @Override
        public String getName() {
            return "my_script";
        }
    }

    public static class MyNativeScript extends AbstractDoubleSearchScript {
        @Override
        public double runAsDouble() {
            double a = (double) source().get("a");
            double b = (double) source().get("b");
            return a * b;
        }
    }
}
```

您可以通过指定脚本的`lang`字段值为`native`，以及`inline`字段的值为脚本的名称来执行它：

```js
POST /_search
{
  "query": {
    "function_score": {
      "query": {
        "match": {
          "body": "foo"
        }
      },
      "functions": [
        {
          "script_score": {
            "script": {
                "inline": "my_script",
                "lang" : "native"
            }
          }
        }
      ]
    }
  }
}
```
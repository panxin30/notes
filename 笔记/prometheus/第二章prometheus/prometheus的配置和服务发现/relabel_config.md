## **relabel_config**

重新标记是一种强大的工具，可以在**抓取目标之前动态重写目标的标签**集。 每个抓取配置**可以配置多个重新标记步骤**。 它们按照它们**在配置文件中的出现顺序应用于每个目标的标签**集。

最初，除了配置的每目标标签之外，目标的作业标签设置为相应的scrape配置的`job_name`值。`__address__`标签设置为目标的`<host>：<port>`地址。 重新标记后，如果在重新标记期间未设置实例标签，则实例标签默认设置为`__address__`的值。`__scheme__`和`__metrics_path__`标签分别设置为目标的方案和度量标准路径。`__param_ <name>`标签设置为名为`<name>`的第一个传递的URL参数的值。

在重新标记阶段，可以使用带有`__meta_`前缀的附加标签。 它们由提供目标的服务发现机制设置，并在不同机制之间变化。

在目标重新标记完成后，将从标签集中删除以`__`开头的标签。

如果重新标记步骤仅需临时存储标签值（作为后续重新标记步骤的输入），请使用`__tmp`标签名称前缀。 保证Prometheus本身不会使用此前缀。
```
# The source labels select values from existing labels. Their content is concatenated
# using the configured separator and matched against the configured regular expression
# for the replace, keep, and drop actions.
# 源标签从现有标签中选择值。 它们的内容使用已配置的分隔符进行连接，并与已配置的正则表达式进行匹配，以进行替换，保留和删除操作。
[ source_labels: '[' <labelname> [, ...] ']' ]

# Separator placed between concatenated source label values.
# 分隔符放置在连接的源标签值之间。
[ separator: <string> | default = ; ]

# Label to which the resulting value is written in a replace action.
# It is mandatory for replace actions. Regex capture groups are available.
# 在替换操作中将结果值写入的标签。
# 替换操作是强制性的。 正则表达式捕获组可用。
[ target_label: <labelname> ]

# Regular expression against which the extracted value is matched.
# 与提取的值匹配的正则表达式。
[ regex: <regex> | default = (.*) ]

# Modulus to take of the hash of the source label values.
# 采用源标签值的散列的模数。
[ modulus: <uint64> ]

# Replacement value against which a regex replace is performed if the
# regular expression matches. Regex capture groups are available.
# 如果正则表达式匹配，则执行正则表达式替换的替换值。 正则表达式捕获组可用。
[ replacement: <string> | default = $1 ]

# Action to perform based on regex matching.
# 基于正则表达式匹配执行的操作。
[ action: <relabel_action> | default = replace ]
```
`<regex>`是任何有效的[RE2正则表达式](https://github.com/google/re2/wiki/Syntax)。这是必需的`replace`，`keep`，`drop`，`labelmap`，`labeldrop`和`labelkeep`行动。正则表达式固定在两端。要取消锚定正则表达式，请使用`.*<regex>.*`。

`<relabel_action>`确定要采取的重新贴标签行动：

*   `replace`：匹配`regex`连接`source_labels`。然后，设置`target_label`于`replacement`与匹配组的引用（`${1}`，`${2}`，...）中`replacement`可以通过值取代。如果`regex`不匹配，则不进行替换。
*   `keep`：删除`regex`与连接不匹配的目标`source_labels`。
*   `drop`：删除`regex`与连接匹配的目标`source_labels`。
*   `hashmod`：设置`target_label`为`modulus`连接的哈希值`source_labels`。
*   `labelmap`：匹配`regex`所有标签名称。然后复制匹配标签的值来标记给出的名字`replacement`与之相匹配的组引用（`${1}`，`${2}`，...）在`replacement`由他们的价值取代。
*   `labeldrop`：匹配`regex`所有标签名称。匹配的任何标签都将从标签集中删除。
*   `labelkeep`：匹配`regex`所有标签名称。任何不匹配的标签都将从标签集中删除。

必须注意`labeldrop`并`labelkeep`确保在删除标签后仍然对标准进行唯一标记。
labelmap会根据regex去匹配Target实例所有标签的名称（注意是名称），并且将捕获到的内容作为为新的标签名称，regex匹配到标签的的值作为新标签的值。
 其中action定义了当前relabel_config对Metadata标签的处理方式，默认的action行为为replace。

replace是根据regex的配置匹配source_labels标签的值（多个source_label的值会按照separator进行拼接），并且将匹配到的值写入到target_label当中，如果有多个匹配组，则可以使用${1}, ${2}确定写入的内容。如果没匹配到任何内容则不对target_label进行重新。

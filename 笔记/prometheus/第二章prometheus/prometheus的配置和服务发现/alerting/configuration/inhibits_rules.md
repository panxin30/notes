当存在与另一组匹配器匹配的警报（源）时，禁止规则将匹配一组匹配器的警报（目标）静音。目标和源警报必须具有`equal`列表中标签名称的相同标签值。

从语义上讲，缺少标签和具有空值的标签是相同的。因此，如果`equal`源和目标警报中都缺少所列的所有标签名称，则将应用禁止规则。

为了防止提醒抑制本身，即相匹配的警报*既*目标和规则的源极侧不能由警报其中相同为真（包括其本身）的抑制。但是，我们建议以警报永远不会匹配双方的方式选择目标和源匹配器。理由更容易，并且不会触发这种特殊情况。
~~~
# Matchers that have to be fulfilled in the alerts to be muted.
target_match:
  [ <labelname>: <labelvalue>, ... ]
target_match_re:
  [ <labelname>: <regex>, ... ]

# Matchers for which one or more alerts have to exist for the
# inhibition to take effect.
source_match:
  [ <labelname>: <labelvalue>, ... ]
source_match_re:
  [ <labelname>: <regex>, ... ]

# Labels that must have an equal value in the source and target
# alert for the inhibition to take effect.
[ equal: '[' <labelname>, ... ']' ]
~~~
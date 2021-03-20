#### Form 组件之字段

```python
Field
    required=True,               是否允许为空
    widget=None,                 HTML插件
    label=None,                  用于生成Label标签或显示内容
    initial=None,                初始值
    help_text='',                帮助信息(在标签旁边显示)
    error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
    show_hidden_initial=False,   是否在当前插件后面再加一个隐藏的且具有默认值的插件（可用于检验两次输入是否一直）
    validators=[],               自定义验证规则
    localize=False,              是否支持本地化
    disabled=False,              是否可以编辑
    label_suffix=None            Label内容后缀
    
CharField(Field)
    max_length=None,             最大长度
    min_length=None,             最小长度
    strip=True                   是否移除用户输入空白
 
IntegerField(Field)
    max_value=None,              最大值
    min_value=None,              最小值
 
FloatField(IntegerField)
    ...
 
DecimalField(IntegerField)
    max_value=None,              最大值
    min_value=None,              最小值
    max_digits=None,             总长度
    decimal_places=None,         小数位长度
 
BaseTemporalField(Field)
    input_formats=None          时间格式化   
 
DateField(BaseTemporalField)    格式：2015-09-01
TimeField(BaseTemporalField)    格式：11:12
DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
 
DurationField(Field)            时间间隔：%d %H:%M:%S.%f
    ...
 
RegexField(CharField)
    regex,                      自定制正则表达式
    max_length=None,            最大长度
    min_length=None,            最小长度
    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
 
EmailField(CharField)      
    ...
 
FileField(Field)
    allow_empty_file=False     是否允许空文件
 
ImageField(FileField)      
    ...
    注：需要PIL模块，pip3 install Pillow
    以上两个字典使用时，需要注意两点：
        - form表单中 enctype="multipart/form-data"
        - view函数中 obj = MyForm(request.POST, request.FILES)
 
URLField(Field)
    ...
 
 
BooleanField(Field)  
    ...
 
NullBooleanField(BooleanField)
    ...
 
ChoiceField(Field)
    ...
    choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
    required=True,             是否必填
    widget=None,               插件，默认select插件
    label=None,                Label内容
    initial=None,              初始值
    help_text='',              帮助提示
 
 
ModelChoiceField(ChoiceField)
    ...                        django.forms.models.ModelChoiceField
    queryset,                  # 查询数据库中的数据
    empty_label="---------",   # 默认空显示内容
    to_field_name=None,        # HTML中value的值对应的字段
    limit_choices_to=None      # ModelForm中对queryset二次筛选
     
ModelMultipleChoiceField(ModelChoiceField)
    ...                        django.forms.models.ModelMultipleChoiceField
 
 
     
TypedChoiceField(ChoiceField)
    coerce = lambda val: val   对选中的值进行一次转换
    empty_value= ''            空值的默认值
 
MultipleChoiceField(ChoiceField)
    ...
 
TypedMultipleChoiceField(MultipleChoiceField)
    coerce = lambda val: val   对选中的每一个值进行一次转换
    empty_value= ''            空值的默认值
 
ComboField(Field)
    fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
                               fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])
 
MultiValueField(Field)
    PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用
 
SplitDateTimeField(MultiValueField)
    input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
    input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']
 
FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中
    path,                      文件夹路径
    match=None,                正则匹配
    recursive=False,           递归下面的文件夹
    allow_files=True,          允许文件
    allow_folders=False,       允许文件夹
    required=True,
    widget=None,
    label=None,
    initial=None,
    help_text=''
 
GenericIPAddressField
    protocol='both',           both,ipv4,ipv6支持的IP格式
    unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用
 
SlugField(CharField)           数字，字母，下划线，减号（连字符）
    ...
 
UUIDField(CharField)           uuid类型
    ...
```

#### Form组件之部件

一个 widget 对应一个 HTML form 输入元素，有两个功能，一是用于 HTML 渲染，二是 从 GET/POST 字典抽出数据。

`Field`类表现* 校验逻辑* ，而部件表现* 显示逻辑* 。

##### 1. 如何为form类指定部件

若不为form的属性指定部件时，使用默认的， [Built-in Field classes](https://docs.djangoproject.com/en/1.6/ref/forms/fields/#built-in-fields) 。然而，若想指定不同的部件时，只需要让 widget 作为参数即可

```python
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField(widget=forms.Textarea)
```

这里的 comment 会变成 Textarea，而不是 CharField

##### 2.如何给部件指定参数

widgets 具有可选的参数，可以在给 form field 定义 widget 时设置。如下所示， years 属性被指定为 [SelectDateWidget](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#django.forms.extras.widgets.SelectDateWidget) 的参数。

```python
from django import forms
from django.forms.extras.widgets import SelectDateWidget

BIRTH_YEAR_CHOICES = ('1980', '1981', '1982')
FAVOfRITE_COLORS_CHOICES = (('blue', 'Blue'),
                            ('green', 'Green'),
                            ('black', 'Black'))

class SimpleForm(forms.Form):
    birth_year = forms.DateField(widget=SelectDateWidget(years=BIRTH_YEAR_CHOICES))
    favorite_colors = forms.MultipleChoiceField(required=False,
        widget=forms.CheckboxSelectMultiple, choices=FAVORITE_COLORS_CHOICES)
```

参考官方文档 [Built-in widgets](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#built-in-widgets) 获得更多关于可用部件及部件参数的信息

##### 3.继承选择部件

继承自选择部件（Select widget）的类可以人性化为用户提供选项列表。不同的部件代表不同选项类别。例如，选择部件自身生成HTML代码 <select> ，而RadioSelect 生成HTML代码 radio button。

ChoiceField 的默认部件是Select 。其所显示的HTML正是继承自 ChoiceField ，更新  ChoiceField.choices 也会更新 Select.choices 的值（两者有绑定关系）。例如：

```python
>>> from django import forms
>>> CHOICES = (('1', 'First',), ('2', 'Second',))
>>> choice_field = forms.ChoiceField(widget=forms.RadioSelect, choices=CHOICES)
>>> choice_field.choices
[('1', 'First'), ('2', 'Second')]
>>> choice_field.widget.choices
[('1', 'First'), ('2', 'Second')]
>>> choice_field.widget.choices = ()
>>> choice_field.choices = (('1', 'First and only',),)
>>> choice_field.widget.choices
[('1', 'First and only')]
```

forms.RadioSelect 正是继承自 Select

##### 4. 如何订制部件

Django 会把部件渲染成 HTML，这个渲染过程只会执行最小的工作量 -- 不会添加类名，或者其它具体的属性。这意味着，例如，所有 TextInput 部件会在你所有的Web页面上具有一样的外观。

有两种办法可以订制部件：一是定制部件的实例对象（订制 field 属性）；二是继承部件，定义内部类（订制 css 和 js 文件的链接）。


1）定制部件的实例对象
     若想让一个部件看起来和其它的不一样，只需要在部件对象被指定给form域时在部件类里添加额外的属性即可。在 Form 类里，对 field 指定widget，并传递一个参数 attrs，这个参数的类型是一个字典。

例如，有一个 Form 类：

```python
from django import forms

class CommentForm(forms.Form):
    name = forms.CharField()
    url = forms.URLField()
    comment = forms.CharField()
```

此 form 包含三个默认的 TextInput 部件，默认没有 CSS 类渲染，没有额外的属性。这意味着每个部件会具有同样的外观。

```python
>>> f = CommentForm(auto_id=False)
>>> f.as_table()
<tr><th>Name:</th><td><input type="text" name="name" /></td></tr>
<tr><th>Url:</th><td><input type="url" name="url"/></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" /></td></tr>
```

在实际应用中，你可能会不想每个部件看起来一样（让comment有更大的输入框或给name部件添加额外的CSS）。可以指定 ‘type’ 属性来利用 HTML5的新HTML元素。想要实现这些，你只需要在创建部件时，使用  [`Widget.attrs`](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#django.forms.Widget.attrs)  作为参数。

```python
class CommentForm(forms.Form):
    name = forms.CharField(
                widget=forms.TextInput(attrs={'class':'special'}))
    url = forms.URLField()
    comment = forms.CharField(
               widget=forms.TextInput(attrs={'size':'40'}))
```

其所渲染的 HTML 会包含额外的属性：

```python
>>> f = CommentForm(auto_id=False)
>>> f.as_table()
<tr><th>Name:</th><td><input type="text" name="name" class="special"/></td></tr>
<tr><th>Url:</th><td><input type="url" name="url"/></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" size="40"/></td></tr>
```

2）继承部件，定义内部类

   第二种方法：继承部件类，或定义内部类 Media，或创建一个 media 属性。

（1）定义内部类 Media

```python
from django import forms

class CalendarWidget(forms.TextInput):
    class Media:
        css = {
            'all': ('pretty.css',)
        }
        js = ('animations.js', 'actions.js')
        
>>> w = CalendarWidget()
>>> print(w.media)
<link href="http://static.example.com/pretty.css" type="text/css" media="all" rel="stylesheet" />
<script type="text/javascript" src="http://static.example.com/animations.js"></script>
<script type="text/javascript" src="http://static.example.com/actions.js"></script>
```

（2）创建一个 Media 属性

```python
class CalendarWidget(forms.TextInput):
    def _media(self):
        return forms.Media(css={'all': ('pretty.css',)},
                           js=('animations.js', 'actions.js'))
    media = property(_media)
```

##### 5 部件类基类

所有内置的部件类都是继承自 [`Widget`](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#django.forms.Widget) 和[`MultiWidget`](https://docs.djangoproject.com/en/1.6/ref/forms/widgets/#django.forms.MultiWidget) ，可以继承它们实现自己的部件功能。

##### 6 内置部件类

- 处理 input 的部件
  TextInput   
  NumberInput
  EmailInput
  URLInput
  PasswordInput
  HiddenInput
  DateInput
  DateTimeInput
  TimeInput
  Textarea

- Selector 和 checkbox 部件
  CheckboxInput
  Select
  NullBooleanSelect
  RadioSelect
  CheckboxSelectMultiple

- File upload 部件
  FileInput
  ClearableFileInput

- 合成部件
  MultipleHiddenInput
  SplitDateTimeWidget
  SplitHiddenDateTimeWidget
  SelectDateWidget
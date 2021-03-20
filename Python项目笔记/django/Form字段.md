[toc]



# 表单字段

当你创建一个 `Form` 类时，最重要的部分是定义表单的字段。每个字段都有自定义的验证逻辑，以及其他一些钩子。

`Field.``clean`**(***value***)**

虽然你使用 `Field` 类的主要方式是在 `Form` 类中，但你也可以将它们实例化并直接使用，以更好地了解它们的工作方式。每个 `Field` 实例都有一个 `clean()` 方法，它接收一个参数，并引发一个 `django.core.exceptions.ValidationError` 异常或返回干净的值：

```python
>>> f = forms.EmailField()
>>> f.clean('foo@example.com')
'foo@example.com'
>>> f.clean('invalid email address')
Traceback (most recent call last):
...
ValidationError: ['Enter a valid email address.']
```

## 核心字段参数

每个 `Field` 类的构造函数至少需要这些参数。有些 `Field` 类需要额外的、特定的字段参数，但以下参数应 *始终* 接受：

### `required`

Field.required

默认情况下，每个 `Field` 类都假定值是必填的，所以如果你传递一个空值—— `None` 或空字符串（`""`）——那么 `clean()` 将引发一个 `ValidationError` 异常：

```python
>>> from django import forms
>>> f = forms.CharField()
>>> f.clean('foo')
'foo'
>>> f.clean('')
Traceback (most recent call last):
...
ValidationError: ['This field is required.']
>>> f.clean(None)
Traceback (most recent call last):
...
ValidationError: ['This field is required.']
>>> f.clean(' ')
' '
>>> f.clean(0)
'0'
>>> f.clean(True)
'True'
>>> f.clean(False)
'False'
```

要指定一个字段是 *不必填* 的，传递 `required=False` 给 `Field` 构造函数：

```python
>>> f = forms.CharField(required=False)
>>> f.clean('foo')
'foo'
>>> f.clean('')
''
>>> f.clean(None)
''
>>> f.clean(0)
'0'
>>> f.clean(True)
'True'
>>> f.clean(False)
'False'
```

如果一个 `Field` 有 `required=False`，而你给 `clean()` 传递一个空值，那么 `clean()` 将返回一个 *规范化* 的空值，而不是引发 `ValidationError`。对于 `CharField`，将返回 [`empty_value`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.CharField.empty_value)，默认为一个空字符串。对于其他 `Field` 类，它可能是 `None`。（这因字段而异。）

必填表单字段的部件有 `required` HTML 属性。将 [`Form.use_required_attribute`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.use_required_attribute) 属性设置为 `False` 就可以禁用。由于在添加和删除表单集时，浏览器的验证可能不正确，所以表单集的表单中不包含 `required` 属性。

### `label`

- Field.label

`label` 参数让你指定该字段的“人类友好”标签。当 `Field` 在 `Form` 中显示时，会用到这个标签。

如上文“将表格输出为 HTML”中所解释的，`Field` 的默认标签是由字段名通过将所有下划线转换为空格并将第一个字母大写而生成的。如果默认行为不能产生一个适当的标签，请指定 `label`。

这是一个完整的 `Form` 例子，它的两个字段实现了 `label`。我们指定 `auto_id=False` 以简化输出：

```python
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(label='Your name')
...     url = forms.URLField(label='Your website', required=False)
...     comment = forms.CharField()
>>> f = CommentForm(auto_id=False)
>>> print(f)
<tr><th>Your name:</th><td><input type="text" name="name" required></td></tr>
<tr><th>Your website:</th><td><input type="url" name="url"></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" required></td></tr>
```

### `label_suffix`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#label-suffix)

- `Field.label_suffix`

`label_suffix` 参数可以让你在每个字段的基础上覆盖表单的 [`label_suffix`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.label_suffix)：

```python
>>> class ContactForm(forms.Form):
...     age = forms.IntegerField()
...     nationality = forms.CharField()
...     captcha_answer = forms.IntegerField(label='2 + 2', label_suffix=' =')
>>> f = ContactForm(label_suffix='?')
>>> print(f.as_p())
<p><label for="id_age">Age?</label> <input id="id_age" name="age" type="number" required></p>
<p><label for="id_nationality">Nationality?</label> <input id="id_nationality" name="nationality" type="text" required></p>
<p><label for="id_captcha_answer">2 + 2 =</label> <input id="id_captcha_answer" name="captcha_answer" type="number" required></p>
```

### `initial`

- `Field.``initial`

`initial` 参数让你指定在未绑定的 `Form` 中渲染这个 `Field` 时要使用的初始值。

要指定动态初始数据，请参见 [`Form.initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 参数。

这个用法是当你想显示一个“空”的表单，其中一个字段被初始化为一个特定的值。例如：

```python
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='Your name')
...     url = forms.URLField(initial='http://')
...     comment = forms.CharField()
>>> f = CommentForm(auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="Your name" required></td></tr>
<tr><th>Url:</th><td><input type="url" name="url" value="http://" required></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" required></td></tr>
```

你可能会想，为什么不在显示表单时传递一个初始值的字典作为数据呢？好吧，如果你这样做，你会触发验证，HTML 输出将包括任何验证错误：

```python
>>> class CommentForm(forms.Form):
...     name = forms.CharField()
...     url = forms.URLField()
...     comment = forms.CharField()
>>> default_data = {'name': 'Your name', 'url': 'http://'}
>>> f = CommentForm(default_data, auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="Your name" required></td></tr>
<tr><th>Url:</th><td><ul class="errorlist"><li>Enter a valid URL.</li></ul><input type="url" name="url" value="http://" required></td></tr>
<tr><th>Comment:</th><td><ul class="errorlist"><li>This field is required.</li></ul><input type="text" name="comment" required></td></tr>
```

这就是为什么 `initial` 值只在未绑定的表单中显示。对于绑定的表格，HTML 输出将使用绑定的数据。

还请注意，`initial` 值在验证中 *不* 用作“后备”数据，如果某个字段的值没有给出。`initial` 值 *仅* 用于初始表格显示：

```python
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='Your name')
...     url = forms.URLField(initial='http://')
...     comment = forms.CharField()
>>> data = {'name': '', 'url': '', 'comment': 'Foo'}
>>> f = CommentForm(data)
>>> f.is_valid()
False
# The form does *not* fall back to using the initial values.
>>> f.errors
{'url': ['This field is required.'], 'name': ['This field is required.']}
```

你也可以通过任何可调用对象来代替常量。

```python
>>> import datetime
>>> class DateForm(forms.Form):
...     day = forms.DateField(initial=datetime.date.today)
>>> print(DateForm())
<tr><th>Day:</th><td><input type="text" name="day" value="12/23/2008" required><td></tr>
```

只有在显示未绑定的表单时，而不是在定义表单时，才会对可调用对象表单执行。

### `widget`

- `Field.``widget`

`widget` 参数让你指定一个 `Widget` 类，以便在渲染这个 `Field` 时使用。参见 [部件](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/) 了解更多信息。

### `help_text`

- `Field.``help_text`

`help_text` 参数让你为这个 `Field` 指定描述性文本。如果你提供了 `help_text`，当 `Field` 被一个方便的 `Form` 方法（例如 `as_ul()`）渲染时，它将显示在 ``Field`` 旁边。

就像模型字段的 [`help_text`](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#django.db.models.Field.help_text) 一样，这个值在自动生成的表单中并没有被 HTML 封装。

这里是一个完整的例子 `Form`，它的两个字段实现了 `help_text`。我们指定 `auto_id=False` 以简化输出：

```python
>>> from django import forms
>>> class HelpTextContactForm(forms.Form):
...     subject = forms.CharField(max_length=100, help_text='100 characters max.')
...     message = forms.CharField()
...     sender = forms.EmailField(help_text='A valid email address, please.')
...     cc_myself = forms.BooleanField(required=False)
>>> f = HelpTextContactForm(auto_id=False)
>>> print(f.as_table())
<tr><th>Subject:</th><td><input type="text" name="subject" maxlength="100" required><br><span class="helptext">100 characters max.</span></td></tr>
<tr><th>Message:</th><td><input type="text" name="message" required></td></tr>
<tr><th>Sender:</th><td><input type="email" name="sender" required><br>A valid email address, please.</td></tr>
<tr><th>Cc myself:</th><td><input type="checkbox" name="cc_myself"></td></tr>
>>> print(f.as_ul()))
<li>Subject: <input type="text" name="subject" maxlength="100" required> <span class="helptext">100 characters max.</span></li>
<li>Message: <input type="text" name="message" required></li>
<li>Sender: <input type="email" name="sender" required> A valid email address, please.</li>
<li>Cc myself: <input type="checkbox" name="cc_myself"></li>
>>> print(f.as_p())
<p>Subject: <input type="text" name="subject" maxlength="100" required> <span class="helptext">100 characters max.</span></p>
<p>Message: <input type="text" name="message" required></p>
<p>Sender: <input type="email" name="sender" required> A valid email address, please.</p>
<p>Cc myself: <input type="checkbox" name="cc_myself"></p>
```

### `error_messages`

- ``Field.error_messages``

`error_messages` 参数可以让你覆盖该字段将引发的默认消息。传递一个与你想覆盖的错误信息相匹配的键值的字典。例如，以下是默认的错误信息：

```python
>>> from django import forms
>>> generic = forms.CharField()
>>> generic.clean('')
Traceback (most recent call last):
  ...
ValidationError: ['This field is required.']
```

而这里是一个自定义的错误信息：

```python
>>> name = forms.CharField(error_messages={'required': 'Please enter your name'})
>>> name.clean('')
Traceback (most recent call last):
  ...
ValidationError: ['Please enter your name']
```

在下面的 [内置字段类](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#built-in-field-classes) 一节中，每个 `Field` 定义了它所使用的错误信息键。

### `validators`

- ``Field.validators``

`validators` 参数让你为这个字段提供一个验证函数列表。

更多信息请参见 [验证器文档](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/)。



### `localize`

- ``Field.localize``

`localize` 参数可以实现表单数据输入和渲染输出的本地化。

更多信息请参见 [格式本地化](https://docs.djangoproject.com/zh-hans/3.1/topics/i18n/formatting/) 文档。



### `disabled`

- ``Field.disabled``

`disabled` 布尔参数设置为 `True` 时，使用 `disabled` HTML 属性禁用表单字段，使其不能被用户编辑。即使用户篡改了提交给服务器的字段值，也会被忽略，而采用表单初始数据的值。

## 检查字段数据是否有变化

### `has_changed()`

- `Field.has_changed`()

`has_changed()` 方法用于确定字段值是否与初始值发生了变化。返回 `True` 或 `False`。

更多信息请参见 [`Form.has_changed()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.has_changed) 文档。

## 内置 `Field` 类

当然，`forms` 库附带了一组 `Field` 类，代表了常见的验证需求。本节将对每个内置字段进行说明。

对于每个字段，我们描述了在你没有指定 `widget` 时使用的默认部件。我们还指定了当你提供一个空值时返回的值（参见上文 `required` 一节以了解其含义）。



### `BooleanField`

- *class* `BooleanField`(***kwargs*)

  - 默认部件：[`CheckboxInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.CheckboxInput) 

  - 空值：`False`

  - 规范化为：Python 的 `True` 或 `False` 值。

  - 如果字段有 `required=True`，则验证该值是否为 `True` （例如，复选框被选中）。

  - 错误信息键：`required`

    > 注解由于所有 `Field` 子类默认都有 `required=True`，这里的验证条件很重要。如果你想在你的表单中包含一个布尔值，这个布尔值可以是 `True` 或 `False` （例如一个选中或未选中的复选框），你必须记得在创建 `BooleanField` 时传递 `required=False`。

  

### `CharField`

- *class* `CharField`(***kwargs*)

  - 默认部件：[`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)

  - 空值：不管你给 [`empty_value`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.CharField.empty_value) 的是什么。

  - 规范化为：一个字符串。

  - 如果提供了 `max_length` 和 `min_length`，则使用 [`MaxLengthValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MaxLengthValidator) 和 [`MinLengthValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MinLengthValidator)。否则，所有输入都有效。

  - 错误信息键：`required`、`max_length`、`min_length`

    需要四个可选参数进行验证：

    `max_length`

    `min_length`

    ​	如果提供了这些参数，这些参数确保字符串的长度最多或至少是给定的长度。

    `strip`

    ​	如果 `True` （默认），该值将被去掉前导和尾部的空白。

    `empty_value`	

    ​	用来表示“空”的值。默认为空字符串。



### `ChoiceField`

- *class* `ChoiceField`(***kwargs*)

  - 默认部件：[`Select`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Select)
  - 空值：`''` （空字符串）
  - 规范化为：一个字符串。
  - 验证给定值是否存在于选择列表中。
  - 错误信息键：`required`、``invalid_choice``

  `invalid_choice` 错误信息可能包含 `%(value)s`，该信息将被替换为选定的选择。

  需要一个额外的参数：

  `choices`

  > 或者是一个 [iterable](https://docs.python.org/3/glossary.html#term-iterable) 的二元元组作为这个字段的选择，或者是一个 [enumeration](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#field-choices-enum-types) 的选择，或者是一个返回这样一个迭代器的可调用对象。这个参数接受的格式与模型字段的 `choices` 参数相同。更多细节请参见 [模型字段引用文档中的选择](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#field-choices)。如果这个参数是可调用的，那么除了在渲染过程中，每次初始化字段的表单时，它都会被执行。默认为空列表。



### `TypedChoiceField`

- *class* `TypedChoiceField`(***kwargs*)

  就像 [`ChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ChoiceField) 一样，除了 [`TypedChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.TypedChoiceField) 需要两个额外的参数 [`coerce`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.TypedChoiceField.coerce) 和 [`empty_value`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.TypedChoiceField.empty_value)。

  - 默认部件：[`Select`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Select)
  - 空值：不管你给 [`empty_value`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.TypedChoiceField.empty_value) 的是什么。
  - 规范化为：[`coerce`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.TypedChoiceField.coerce) 参数提供的类型的值。
  - 验证给定的值是否存在于选择列表中，并且可以被强制执行。
  - 错误信息键：`required`、`invalid_choice`

  需要额外的参数：

  `coerce`

  接受一个参数并返回一个强制值的函数。例子包括内置的 `int`、`float`、`bool` 和其他类型。默认为身份函数。请注意，强制执行发生在输入验证之后，所以可以强制执行到一个不存在于``choices``中的值。

​	`empty_value`

​		用来表示“空”的值。默认为空字符串；`None` 是另一种常见的选择。请注意，这个值不会被 `coerce` 参数中给出的函数强制执行，所		以要据此选择。



### `DateField`

- *class* `DateField`(***kwargs*)

  - 默认部件：[`DateInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.DateInput)
  - 空值：`None`
  - 规范化为：Python 的 `datetime.date` 对象。
  - 验证给定值是 `datetime.date`、`datetime.datetime` 或以特定日期格式化的字符串。
  - 错误信息键：`required`、`invalid`

  需要一个可选的参数：

  `input_formats`

  ​	用于将字符串转换为有效的 `datetime.date` 对象的格式列表。

  如果没有提供 `input_formats` 参数，如果 [`USE_L10N`](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-USE_L10N) 为 `False`，则默认输入格式来自 [`DATE_INPUT_FORMATS`](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-DATE_INPUT_FORMATS)，如果启用了本地化，则默认输入格式来自激活的的本地格式 `DATE_INPUT_FORMATS` 键。也请参见 [格式本地化](https://docs.djangoproject.com/zh-hans/3.1/topics/i18n/formatting/)。



### `DateTimeField`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#datetimefield)

- *class* `DateTimeField`(***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.DateTimeField)

  - 默认部件：`DateTimeInput`
  - 空值：`None`
  - 规范化为：Python 的 `datetime.datetime` 对象。
  - 验证给定的值是 `datetime.datetime`、`datetime.date` 或以特定日期时间格式化的字符串。错误信息键：`required`、`invalid`

  需要一个可选的参数：

  `input_formats`

  ​	除 ISO 8601 格式外，还列出了用于将字符串转换为有效的 `datetime.datetime` 对象的格式。该字段总是接受 ISO 8601 格式的日期或类似的 [`parse_datetime()`](https://docs.djangoproject.com/zh-hans/3.1/ref/utils/#django.utils.dateparse.parse_datetime) 识别的字符串。一些例子是：

  ```python
  * '2006-10-25 14:30:59'
  * '2006-10-25T14:30:59'
  * '2006-10-25 14:30'
  * '2006-10-25T14:30'
  * '2006-10-25T14:30Z'
  * '2006-10-25T14:30+02:00'
  * '2006-10-25'
  ```

  如果没有提供 `input_formats` 参数，默认的输入格式来自 [`DATETIME_INPUT_FORMATS`](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-DATETIME_INPUT_FORMATS) 和 [`DATE_INPUT_FORMATS`](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-DATE_INPUT_FORMATS)，如果: setting:USE_L10N 为 `False`，如果启用了本地化，则从激活的本地格式 `DATETIME_INPUT_FORMATS` 和 `DATE_INPUT_FORMATS` 键中获取。也请参见 [格式本地化](https://docs.djangoproject.com/zh-hans/3.1/topics/i18n/formatting/)。

  **Changed in Django 3.1:**

  增加了对 ISO 8601 日期字符串解析的支持（包括可选的时区）。

  在默认的 `input_formats` 中增加了 `DATE_INPUT_FORMATS` 的后备功能。



### `DecimalField`

- *class* `DecimalField`(***kwargs*)

  - 当 [`Field.localize`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.localize) 为 `False` 时是 [`NumberInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.NumberInput) 否则，该字段的默认表单部件是 [`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)。
  - 空值：`None`
  - 规范化为：Python 的 `decimal`。
  - 验证给定值是否为十进制。如果提供了 `max_value` 和 `min_value`，则使用 [`MaxValueValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MaxValueValidator) 和 [`MinValueValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MinValueValidator)。前面和后面的空格会被忽略。
  - 错误信息键：`required`、`invalid`、`max_value`、`min_value`、`max_digits`、`max_decimal_places`、`max_whole_digits`

  ``max_value` 和 `min_value` 错误信息可能包含 `%(limit_value)s`，将用适当的限制代替。同样，`max_digits`、`max_decimal_places` 和 `max_whole_digits` 错误信息可能包含 `%(max)s`。

  需要四个可选的参数：

  `max_value`

  `min_value`

  ​	这些控制着字段中允许的数值范围，应以 `decimal.Decimal` 值的形式给出。

  `max_digits`

  ​	值中允许的最大数字（小数点前的数字加上小数点后的数字，去掉前导零）。

  `decimal_places`

  ​	允许的最大小数位数。



### `DurationField`

- *class* `DurationField`(***kwargs*)

  - 默认部件：TextInput`
  - 空值：`None`
  - 规范化为：Python 的 [`timedelta`](https://docs.python.org/3/library/datetime.html#datetime.timedelta)。
  - 验证给定值是否是一个字符串，可以转换成 `timedelta`。该值必须在 [`datetime.timedelta.min`](https://docs.python.org/3/library/datetime.html#datetime.timedelta.min) 和 [`datetime.timedelta.max`](https://docs.python.org/3/library/datetime.html#datetime.timedelta.max) 之间。
  - 错误信息键：`required`、`invalid`、`overflow`

  接受 [`parse_duration()`](https://docs.djangoproject.com/zh-hans/3.1/ref/utils/#django.utils.dateparse.parse_duration) 理解的任何格式。

### `EmailField`

*class* `EmailField`(***kwargs*)

- 默认部件：[`EmailInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.EmailInput)
- 空值：不管你给 `empty_value` 的是什么。
- 规范化为：一个字符串。
- 使用 [`EmailValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.EmailValidator) 来验证给定的值是一个有效的电子邮件地址，使用一个适度复杂的正则表达式。
- 错误信息键：`required`、`invalid`

有三个可选参数 `max_length`、`min_length` 和 `empty_value`，它们的工作原理与 [`CharField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.CharField) 一样。

### `FileField`

*class* `FileField`(***kwargs*)

- 默认部件：[`ClearableFileInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.ClearableFileInput)
- 空值：`None`
- 规范化为：一个 `UploadedFile` 对象，它将文件内容和文件名包装成一个单一对象。
- 可以验证非空文件数据已经绑定到表单中。
- 错误信息键：`required`、`invalid`、`missing`、`empty`、`max_length`

有两个可选的验证参数，`max_length` 和 `allow_empty_file`。如果提供了这两个参数，将确保文件名的长度最多为给定长度，即使文件内容为空，验证也会成功。

要了解更多关于 `UploadedFile` 对象的信息，请看 [文件上传文档](https://docs.djangoproject.com/zh-hans/3.1/topics/http/file-uploads/)。

当你在表单中使用 `FileField` 时，你还必须记住 [将文件数据绑定到表单中](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#binding-uploaded-files)。

`max_length` 错误指的是文件名的长度。在该键的错误信息中，`%(max)d` 将被替换为最大文件名长度，`%(length)d` 将被替换为当前文件名长度。

### `FilePathField`

*class* `FilePathField`(***kwargs*)

- 默认部件：[`Select`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Select)
- 空值：`''` （空字符串）
- 规范化为：一个字符串。
- 验证选择是否存在于选择列表中。
- 错误信息键：`required`、`invalid_choice`

该字段允许从某个目录内的文件中选择。它需要五个额外的参数；只有 `path` 是必须的。

- `path`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.path)

  你想要列出的内容的目录的绝对路径。该目录必须存在。

- `recursive`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.recursive)

  如果 `False` （默认），只提供 `path` 的直接内容作为选择。如果 `True`，目录将被递归递进，所有的子目录将被列为选择。

- `match`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.match)

  正则表达式模式；只允许将名称与此表达式相匹配的文件作为选择。

- `allow_files`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.allow_files)

  可选。 可选 `True` 或 `False`。 默认值是 `True`。 指定是否应该包含指定位置的文件。 此项或 [`allow_folders`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.allow_folders) 必须为 `True`。

- `allow_folders`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.allow_folders)

  可选。 可选 `True` 或 `False`。 默认为 `False`。 指定是否应包括指定位置的文件夹。 此项或 [`allow_files`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FilePathField.allow_files) 必须为 `True`。`

### `FloatField`

*class* `FloatField`(***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FloatField)

- 当 [`Field.localize`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.localize) 为 `False` 时是 [`NumberInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.NumberInput) 否则，该字段的默认表单部件是 [`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)。
- 空值：`None`
- 规范化为：Python 的浮点数。
- 验证给定的值是否为浮点数。如果提供了 `max_value` 和 `min_value`，则使用 [`MaxValueValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MaxValueValidator) 和 [`MinValueValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.MinValueValidator)。允许使用前导空格和后导空格，就像 Python 的 `float()` 函数一样。
- 错误信息键：`required`、`invalid`、`max_value`、`min_value`

取两个可选参数：`max_value` 和 `min_value`。这两个参数控制该字段允许的值的范围。

### `ImageField`

*class* `ImageField`(***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ImageField)

- 默认部件：[`ClearableFileInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.ClearableFileInput)
- 空值：`None`
- 规范化为：一个 `UploadedFile` 对象，它将文件内容和文件名包装成一个单一对象。
- 验证文件数据是否已经绑定到表单中。同时使用 [`FileExtensionValidator`](https://docs.djangoproject.com/zh-hans/3.1/ref/validators/#django.core.validators.FileExtensionValidator) 来验证 Pillow 是否支持文件扩展名。
- 错误信息键：`required`、`invalid`、`missing`、`empty`、`invalid_image`

使用 `ImageField` 需要安装的 [Pillow](https://pillow.readthedocs.io/en/latest/) 支持你使用的图片格式。如果你在上传图片时遇到 `corrupt image` 错误，通常意味着 Pillow 不理解图片格式。要解决这个问题，请安装相应的库并重新安装 Pillow。

当你在表单中使用 `ImageField` 时，你还必须记住 [将文件数据绑定到表单](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#binding-uploaded-files)。

在字段被清理和验证后，`UploadedFile` 对象将有一个额外的 `image` 属性，其中包含 Pillow [Image](https://pillow.readthedocs.io/en/latest/reference/Image.html) 实例，用于检查文件是否是一个有效的图像。Pillow 在验证图像后会关闭底层的文件描述符，所以虽然非图像数据属性，如 `format`、`height` 和 `width`，是可用的，但访问底层图像数据的方法，如 `getdata()` 或 `getpixel()`，在不重新打开文件的情况下不能使用。例如：

```python
>>> from PIL import Image
>>> from django import forms
>>> from django.core.files.uploadedfile import SimpleUploadedFile
>>> class ImageForm(forms.Form):
...     img = forms.ImageField()
>>> file_data = {'img': SimpleUploadedFile('test.png', <file data>)}
>>> form = ImageForm({}, file_data)
# Pillow closes the underlying file descriptor.
>>> form.is_valid()
True
>>> image_field = form.cleaned_data['img']
>>> image_field.image
<PIL.PngImagePlugin.PngImageFile image mode=RGBA size=191x287 at 0x7F5985045C18>
>>> image_field.image.width
191
>>> image_field.image.height
287
>>> image_field.image.format
'PNG'
>>> image_field.image.getdata()
# Raises AttributeError: 'NoneType' object has no attribute 'seek'.
>>> image = Image.open(image_field)
>>> image.getdata()
<ImagingCore object at 0x7f5984f874b0>
```

此外，`UploadedFile.content_type` 如果 Pillow 能够确定图片的内容类型，则会以图片的内容类型进行更新，否则会设置为 `None`。

### `IntergeFiled`

### `JSONFiled`

### `GenericIPAddressFiled`

### `MultipleChoiceFiled`

### `TypedMultipleChoiceFiled`

### `NullBooleanField`

### `RegexField`

### `SlugField`

### `TimeField`

### `URLField`

### `UUIDField`

## 稍复杂的内置 `Field` 类

### `ComboField`

*class* `ComboField`(***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ComboField)

- 默认部件：[`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)
- 空值：`''` （空字符串）
- 规范化为：一个字符串。
- 根据 `ComboField` 参数指定的每个字段验证给定值。
- 错误信息键：`required`、`invalid`

需要一个额外的必要参数。

- `fields`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ComboField.fields)

  应该用来验证字段值的字段列表（按照提供的顺序）。

  ```python
  >>> from django.forms import ComboField
  >>> f = ComboField(fields=[CharField(max_length=20), EmailField()])
  >>> f.clean('test@example.com')
  'test@example.com'
  >>> f.clean('longemailaddress@example.com')
  Traceback (most recent call last):
  ...
  ValidationError: ['Ensure this value has at most 20 characters (it has 28).']
  ```

### `MultiValueField`

*class* `MultiValueField`(*fields=()*, ***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField)

- 默认部件：[`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)
- 空值：`''` （空字符串）
- 规范化为：子类的 `compress` 方法返回的类型。
- 根据作为 `MultiValueField` 参数指定的每个字段验证给定值。
- 错误信息键：`required`、`invalid`、`incomplete`

将多个字段的逻辑聚合在一起产生一个值。

这个字段是抽象的，必须被子类化。与单值字段相反， [`MultiValueField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField) 的子类不能实现 [`clean()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.clean)，而是——实现 [`compress()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.compress)。

需要一个额外的必要参数。

- `fields`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.fields)

  字段组成的元组，其值经清理后合并为一个值。 字段的每个值都由 `fields` 中的相应字段进行清理——第一个值由第一个字段清理，第二个值由第二个字段清理，等等。当所有字段清理完毕后，通过 [`compress()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.compress) 将清理后的值列表合并为一个值。

还需要一些可选的参数：

`require_all_fields`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.require_all_fields)

默认值为 `True`，在这种情况下，如果没有为任何字段提供值，就会出现 `required` 验证错误。

[`Field.required`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.required) 属性设置为 `False` 时，可将单个字段设置为 `False`，使其成为可选字段。如果没有为必填字段提供任何值，就会出现 `incomplete` 的验证错误。

可以在 [`MultiValueField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField) 子类上定义一个默认的 `incomplete` 错误信息，也可以在每个单独的字段上定义不同的信息。例如：

```python
from django.core.validators import RegexValidator

class PhoneField(MultiValueField):
    def __init__(self, **kwargs):
        # Define one message for all fields.
        error_messages = {
            'incomplete': 'Enter a country calling code and a phone number.',
        }
        # Or define a different message for each field.
        fields = (
            CharField(
                error_messages={'incomplete': 'Enter a country calling code.'},
                validators=[
                    RegexValidator(r'^[0-9]+$', 'Enter a valid country calling code.'),
                ],
            ),
            CharField(
                error_messages={'incomplete': 'Enter a phone number.'},
                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid phone number.')],
            ),
            CharField(
                validators=[RegexValidator(r'^[0-9]+$', 'Enter a valid extension.')],
                required=False,
            ),
        )
        super().__init__(
            error_messages=error_messages, fields=fields,
            require_all_fields=False, **kwargs
        )
```

- `widget`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.widget)

  必须是 [`django.forms.MultiWidget`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.MultiWidget) 的子类。默认值是 [`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput)，在这种情况下可能不是很有用。

- `compress`(*data_list*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.MultiValueField.compress)

  取一个有效值的列表，并返回这些值的“压缩”版本——在一个单一值中。例如，[`SplitDateTimeField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.SplitDateTimeField) 是一个子类，它将一个时间字段和一个日期字段合并成一个 `datetime` 对象。这个方法必须在子类中实现。

## 处理关系的字段

有两个字段可用于表示模型之间的关系： [`ModelChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField) 和 [`ModelMultipleChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelMultipleChoiceField)。 这两个字段都需要一个 `queryset` 参数，用于创建字段的选择。 在表单验证后，这些字段将把一个模型对象（对于 `ModelChoiceField`）或多个模型对象（对于 `ModelMultipleChoiceField`）放入表单的 `cleaned_data` 字典中。

对于更复杂的用途，你可以在声明表单字段时指定 `queryset=None`，然后在表单的 `__init__()` 方法中填充 `queryset`：

```python
class FooMultipleChoiceForm(forms.Form):
    foo_select = forms.ModelMultipleChoiceField(queryset=None)

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['foo_select'].queryset = ..
```

`ModelChoiceField` 和 `ModelMultipleChoiceField` 都有一个 `iterator` 属性，它指定了在生成选择时用于迭代查询集的类。详见 [迭代关系选择](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#iterating-relationship-choices)。

### `ModelChoiceField`

*class* `ModelChoiceField`(***kwargs*)[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField)

- 默认部件：[`Select`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Select)
- 空值：`None`
- 规范化为：一个模型实例。
- 验证给定的 id 是否存在于查询集中。
- 错误信息键：`required`、`invalid_choice`

允许选择一个单一的模型对象，适合代表一个外键。请注意，当条目数量增加时，`ModelChoiceField` 的默认部件变得不实用。你应该避免将其用于超过 100 个项目。

需要一个参数：

- `queryset`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField.queryset)

  由模型对象组成的 `QuerySet`，从中得出字段的选择，用于验证用户的选择。它在表单渲染时被执行。

`ModelChoiceField` 也需要两个可选参数：

- `empty_label`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField.empty_label)

  默认情况下，`ModelChoiceField` 使用的 `<select>` 小组件将在列表顶部有一个空的选择。你可以用 `empty_label` 属性来改变这个标签的文本（默认是 `"---------"`），或者你可以通过将 `empty_label` 设置为 `None` 来完全禁用空标签。

```python
# A custom empty label
field1 = forms.ModelChoiceField(queryset=..., empty_label="(Nothing)")

# No empty label
field2 = forms.ModelChoiceField(queryset=..., empty_label=None)
```

请注意，如果 `ModelChoiceField` 是必需的，并且有一个默认的初始值，则不会创建空的选择（无论 `empty_label` 的值如何）

`to_field_name`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField.to_field_name)

这个可选参数用于指定字段，作为字段的小组件中选择的值。请确保它是模型的唯一字段，否则所选的值可能会匹配多个对象。默认情况下，它被设置为 `None`，在这种情况下，将使用每个对象的主键。例如：

```python
# No custom to_field_name
field1 = forms.ModelChoiceField(queryset=...)
```

会产生：

```python
<select id="id_field1" name="field1">
<option value="obj1.pk">Object1</option>
<option value="obj2.pk">Object2</option>
...
</select>
```

和：

```python
# to_field_name provided
field2 = forms.ModelChoiceField(queryset=..., to_field_name="name")
```

会产生：

```python
<select id="id_field2" name="field2">
<option value="obj1.name">Object1</option>
<option value="obj2.name">Object2</option>
...
</select>
```

`ModelChoiceField` 也有属性：

- `iterator`[¶](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField.iterator)

  用于从 `queryset` 中生成字段选择的迭代器类。默认情况下， [`ModelChoiceIterator`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceIterator)。

模型的 `__str__()` 方法将被调用，以生成用于字段选择的对象的字符串表示。要提供自定义的表示，请将 `ModelChoiceField` 子类化，并覆盖 `label_from_instance`。该方法将接收一个模型对象，并应返回一个适合表示它的字符串。例如：

```python
from django.forms import ModelChoiceField

class MyModelChoiceField(ModelChoiceField):
    def label_from_instance(self, obj):
        return "My Object #%i" % obj.id
```

### 迭代关系选择

默认情况下， [`ModelChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceField) 和 [`ModelMultipleChoiceField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelMultipleChoiceField) 使用 [`ModelChoiceIterator`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceIterator) 来生成它们的字段 `choices`。

当迭代时，`ModelChoiceIterator` 产生 2 个包含 [`ModelChoiceIteratorValue`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.ModelChoiceIteratorValue) 实例的选择，作为每个选择中的第一个 `value` 元素。`ModelChoiceIteratorValue` 包装选择值，同时维护对源模型实例的引用，该实例可用于自定义部件实现，例如，将 [data-* attributes](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/data-*) 添加到 `<option>` 元素。

例如，考虑以下模型：

```python
from django.db import models

class Topping(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(decimal_places=2, max_digits=6)

    def __str__(self):
        return self.name

class Pizza(models.Model):
    topping = models.ForeignKey(Topping, on_delete=models.CASCADE)
```

你可以使用 [`Select`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.Select) 部件子类将 `Topping.price` 的值作为 HTML 属性 `data-price`，包含在每个 `<option>` 元素中：

```python
from django import forms

class ToppingSelect(forms.Select):
    def create_option(self, name, value, label, selected, index, subindex=None, attrs=None):
        option = super().create_option(name, value, label, selected, index, subindex, attrs)
        if value:
            option['attrs']['data-price'] = value.instance.price
        return option

class PizzaForm(forms.ModelForm):
    class Meta:
        model = Pizza
        fields = ['topping']
        widgets = {'topping': ToppingSelect}
```

这将使 `Pizza.topping` 选择为：

```python
<select id="id_topping" name="topping" required>
<option value="" selected>---------</option>
<option value="1" data-price="1.50">mushrooms</option>
<option value="2" data-price="1.25">onions</option>
<option value="3" data-price="1.75">peppers</option>
<option value="4" data-price="2.00">pineapple</option>
</select>
```

对于更高级的用法，你可以将 `ModelChoiceIterator` 子类化，以自定义产生的二元元组选择。
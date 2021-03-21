[toc]

## 绑定和非绑定表单

一个 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例要么是 **绑定** 到一组数据，要么是 **未绑定**。

- 如果是 **绑定** 了一组数据，它就能够验证这些数据，并将表单渲染成 HTML，并在 HTML 中显示数据。
- 如果是 **未绑定**，它就不能进行验证（因为没有数据可验证！），但它仍然可以将空白表单渲染为 HTML。

*class*  `Form`

要创建一个未绑定的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例，实例化类：

```python
>>> f = ContactForm()
```

要将数据绑定到表单中，将数据以字典的形式传递给你的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类构造函数的第一个参数：

```python
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
```

在这个字典中，键是字段名，对应于 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类中的属性。值是你要验证的数据。这些数据通常是字符串，但不要求它们是字符串；你传递的数据类型取决于 [`Field`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field)，我们稍后会看到。

- `Form.is_bound`

如果你需要在运行时区分绑定和未绑定的表单实例，请检查表单的 [`is_bound`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_bound) 属性的值：

```python
>>> f = ContactForm()
>>> f.is_bound
False
>>> f = ContactForm({'subject': 'hello'})
>>> f.is_bound
True
```

需要注意的是，传递一个空的字典会创建一个空数据的 *绑定* 表单：

```python
>>> f = ContactForm({})
>>> f.is_bound
True
```

如果你有一个绑定的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例，并想以某种方式改变数据，或者你想将一个未绑定的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例绑定到一些数据，请创建另一个 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例。没有办法改变一个 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例中的数据。一旦创建了一个 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例，你应该认为它的数据是不可改变的，不管它是否有数据。

## 使用表单来验证数据

- `Form.clean`()

  当你必须为相互依赖的字段添加自定义验证时，在你的 `Form` 上实现一个 `clean()` 方法。参见 [清理和验证相互依赖的字段](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/validation/#validating-fields-with-clean) 的用法示例。

- `Form.is_valid`()

[`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 对象的主要任务是验证数据。有了一个绑定的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例，调用 [`is_valid()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_valid) 方法来运行验证，并返回一个布尔值，指定数据是否有效：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
True
```

让我们用一些无效的数据试试。在这种情况下，`subject` 是空白的（这是一个错误，因为所有的字段都是默认的），`sender` 不是一个有效的电子邮件地址：

```
>>> data = {'subject': '',
...         'message': 'Hi there',
...         'sender': 'invalid email address',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
False
```

- `Form.errors`

访问 [`errors`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors) 属性来获取错误信息的字典：

```
>>> f.errors
{'sender': ['Enter a valid email address.'], 'subject': ['This field is required.']}
```

在这个字典中，键是字段名，值是代表错误信息的字符串列表。错误信息存储在列表中，因为一个字段可以有多个错误信息。

你可以访问 [`errors`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors)，而不必先调用 [`is_valid()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_valid)。无论是调用 [`is_valid()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_valid) 还是访问 [`errors`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors)，表单的数据都会首先被验证。

验证例程只会被调用一次，无论你访问 [`errors`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors) 或调用 [`is_valid()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_valid) 多少次。这意味着，如果验证有副作用，这些副作用将只被触发一次。

- `Form.errors.as_data`()

返回一个 `dict`，将字段映射到它们的原始 `ValidationError` 实例。

```python
>>> f.errors.as_data()
{'sender': [ValidationError(['Enter a valid email address.'])],
'subject': [ValidationError(['This field is required.'])]}
```

当你需要通过错误 `code` 来识别错误时，请使用此方法。这样就可以在给定的错误出现时，重写错误信息或在视图中编写自定义逻辑。它还可以用来以自定义格式（如 XML）将错误序列化；例如， `as_json()` 依赖于 `as_data()`。

需要使用 `as_data()` 方法是由于向后兼容性。以前，`ValidationError` 实例一旦被添加到 `Form.errors` 字典中，其 **渲染的** 错误信息就会丢失。理想情况下，`Form.errors` 会存储 `ValidationError` 实例，并且带有 `as_` 前缀的方法可以渲染它们，但为了不破坏那些期望在 `Form.errors` 中渲染错误信息的代码，必须反过来做。

- `Form.errors.as_json`(*escape_html=False*)

返回以 JSON 格式序列化的错误。

```
>>> f.errors.as_json()
{"sender": [{"message": "Enter a valid email address.", "code": "invalid"}],
"subject": [{"message": "This field is required.", "code": "required"}]}
```

默认情况下，`as_json()` 不会转义其输出。如果你使用它来处理类似 AJAX 请求的表单视图，客户端解释响应并将错误插入到页面中，你会希望确保在客户端转义结果，以避免跨站点脚本攻击的可能性。你可以在 JavaScript 中使用 `element.textContent = errorText` 或者使用 jQuery 的 `$(el).text(errorText)` （而不是它的 `.html()` 函数）来实现。

如果出于某些原因你不想使用客户端转义，你也可以设置 `escape_html=True`，错误信息将被转义，这样你就可以直接在 HTML 中使用它们。

- `Form.errors.get_json_data`(*escape_html=False*)

`Form. errors.as_json()` 将返回序列化的 JSON，而这个则是返回序列化之前的错误数据。

`escape_html` 参数的行为如 [`Form.errors.as_json()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors.as_json) 中所述。

- `Form.add_error`(*field*, *error*)

该方法允许在 `Form.clean()` 方法中添加错误到特定字段，或者从表单外部添加错误，例如从视图中添加。

`field` 参数是应该添加错误的字段名。如果它的值是 `None`，错误将被视为非字段错误，就像 [`Form.non_field_errors()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.non_field_errors) 返回的那样。

`error` 参数可以是一个字符串，或者最好是 `ValidationError` 的实例。关于定义表单错误的最佳做法，请参见 [引发 ValidationError](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/validation/#raising-validation-error)。

注意，`Form.add_error()` 会自动从 `cleaned_data` 中删除相关字段。

- `Form.has_error`(*field*, *code=None*)

本方法返回一个布尔值，表示一个字段是否有特定错误 `code` 的错误。如果 `code` 是 `None`，如果字段包含任何错误，它将返回 `True`。

要检查非字段错误，使用 [`NON_FIELD_ERRORS`](https://docs.djangoproject.com/zh-hans/3.1/ref/exceptions/#django.core.exceptions.NON_FIELD_ERRORS) 作为 `field` 参数。

- `Form.non_field_errors`()

这个方法从 [`Form.errors`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.errors) 中返回没有与特定字段关联的错误列表。这包括在 [`Form.clean()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.clean) 中引发的 `ValidationError` 和使用 [`Form.add_error(None, "...")`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.add_error) 添加的错误。

### 非绑定表单的行为

在没有数据的情况下验证表单是没有意义的，但是，记录一下，以下是未绑定表单的情况：

```python
>>> f = ContactForm()
>>> f.is_valid()
False
>>> f.errors
{}
```

## 动态初始值

- `Form.initial`

使用 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 在运行时声明表单字段的初始值。例如，你可能想用当前会话的用户名来填写 `username` 字段。

要实现这一目标，请使用 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 参数到 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form)。如果给定这个参数，它应该是一个将字段名映射到初始值的字典。只包含你要指定初始值的字段，没有必要包含表单中的每个字段。例如：

```
>>> f = ContactForm(initial={'subject': 'Hi there!'})
```

这些值只对未绑定的表单显示，如果没有提供特定的值，它们不会被用作后备值。

如果一个 [`Field`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field) 定义了 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.initial) *并且* 你在实例化 `Form` 时包含了 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial)，那么后者 `initial` 将具有优先权。在这个例子中，`initial` 既在字段级别又在表单实例级别提供，后者具有优先权：

```python
>>> from django import forms
>>> class CommentForm(forms.Form):
...     name = forms.CharField(initial='class')
...     url = forms.URLField()
...     comment = forms.CharField()
>>> f = CommentForm(initial={'name': 'instance'}, auto_id=False)
>>> print(f)
<tr><th>Name:</th><td><input type="text" name="name" value="instance" required></td></tr>
<tr><th>Url:</th><td><input type="url" name="url" required></td></tr>
<tr><th>Comment:</th><td><input type="text" name="comment" required></td></tr>
```

- `Form.get_initial_for_field`(*field*, *field_name*)

使用 [`get_initial_for_field()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.get_initial_for_field) 来获取表单字段的初始数据。它按顺序从 [`Form.initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 和 [`Field.initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.initial) 中获取数据，并执行任何可调用的初始值。

## 检查哪些表格数据已经改变

- `Form.has_changed`()

当你需要检查表单数据是否与初始数据发生变化时，请使用 `has_changed()` 方法。

```python
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data, initial=data)
>>> f.has_changed()
False
```

当表单提交后，我们会重新构建表单，并提供原始数据，以便进行对比。

```python
>>> f = ContactForm(request.POST, initial=data)
>>> f.has_changed()
```

如果来自 `request.POST` 的数据与 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 中提供的数据不同，那么 `has_changed()` 将为 `True`，否则为 `False`。结果是通过调用 [`Field.has_changed()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field.has_changed) 对表单中的每个字段进行计算。

- `Form.changed_data`

`changed_data` 属性返回表单绑定数据（通常是 `request.POST`）中与 [`initial`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.initial) 中提供的数据不同的字段名列表。如果没有不同的数据，则返回一个空列表。

```python
>>> f = ContactForm(request.POST, initial=data)
>>> if f.has_changed():
...     print("The following fields changed: %s" % ", ".join(f.changed_data))
>>> f.changed_data
['subject', 'message']
```

## 访问表单中的字段

- `Form.fields`

你可以从 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例的 `fields` 属性中访问其字段：

```python
>>> for row in f.fields.values(): print(row)
...
<django.forms.fields.CharField object at 0x7ffaac632510>
<django.forms.fields.URLField object at 0x7ffaac632f90>
<django.forms.fields.CharField object at 0x7ffaac3aa050>
>>> f.fields['name']
<django.forms.fields.CharField object at 0x7ffaac6324d0>
```

你可以改变 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例的字段，以改变它在表单中的显示方式：

```
>>> f.as_table().split('\n')[0]
'<tr><th>Name:</th><td><input name="name" type="text" value="instance" required></td></tr>'
>>> f.fields['name'].label = "Username"
>>> f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="instance" required></td></tr>'
```

请注意不要修改 `base_fields` 属性，因为这个修改会影响同一 Python 进程中所有后续的 `ContactForm` 实例：

```
>>> f.base_fields['name'].label = "Username"
>>> another_f = CommentForm(auto_id=False)
>>> another_f.as_table().split('\n')[0]
'<tr><th>Username:</th><td><input name="name" type="text" value="class" required></td></tr>'
```



## 访问“干净的”数据

- `Form.cleaned_data`

[`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类中的每个字段不仅负责验证数据，还负责“清理”数据——将其规范为一致的格式。这是一个很好的功能，因为它允许以各种方式输入特定字段的数据，并总是产生一致的输出。

例如， [`DateField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.DateField) 将输入规范化为 Python 的 `datetime.date` 对象。无论你传递给它的是格式为 `'1994-07-15'` 的字符串，还是 `datetime.date` 对象，或者其他一些格式，只要它是有效的，`DateField` 都会将它规范化为 `datetime.date` 对象。

当你用一组数据创建了一个 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 实例并对其进行验证后，你就可以通过它的 `cleaned_data` 属性来访问干净的数据：

```python
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data
{'cc_myself': True, 'message': 'Hi there', 'sender': 'foo@example.com', 'subject': 'hello'}
```

请注意，任何基于文本的字段——如 `CharField` 或 `EmailField`——总是将输入清理成一个字符串。我们将在本文档后面介绍编码的含义。

如果你的数据 *没有* 验证，`cleaned_data` 字典只包含有效字段：

```python
>>> data = {'subject': '',
...         'message': 'Hi there',
...         'sender': 'invalid email address',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> f.is_valid()
False
>>> f.cleaned_data
{'cc_myself': True, 'message': 'Hi there'}
```

`cleaned_data` 总是只包含 `Form` 中定义的字段的键，即使你在定义 `Form` 时传递了额外的数据。在这个例子中，我们传递了一堆额外的字段给 `ContactForm` 构造函数，但是 `cleaned_data` 只包含了表单的字段：

```
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True,
...         'extra_field_1': 'foo',
...         'extra_field_2': 'bar',
...         'extra_field_3': 'baz'}
>>> f = ContactForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data # Doesn't contain extra_field_1, etc.
{'cc_myself': True, 'message': 'Hi there', 'sender': 'foo@example.com', 'subject': 'hello'}
```

当 `Form` 有效时，`cleaned_data` 将包括 *所有* 字段的键和值，即使数据没有包括一些可选字段的值。在这个例子中，数据字典没有包含 `nick_name` 字段的值，但 `cleaned_data` 包含了它，并使用一个空值：

```
>>> from django import forms
>>> class OptionalPersonForm(forms.Form):
...     first_name = forms.CharField()
...     last_name = forms.CharField()
...     nick_name = forms.CharField(required=False)
>>> data = {'first_name': 'John', 'last_name': 'Lennon'}
>>> f = OptionalPersonForm(data)
>>> f.is_valid()
True
>>> f.cleaned_data
{'nick_name': '', 'first_name': 'John', 'last_name': 'Lennon'}
```

在上面这个例子中，`nick_name` 的 `cleaned_data` 值被设置为一个空字符串，因为 `nick_name` 是 `CharField`，而 `CharField` 将空值视为空字符串。每个字段类型都知道它的“空”值是什么——例如，对于 `DateField`，它是 `None` 而不是空字符串。关于每个字段在这种情况下的行为的全部细节，请参阅下面“内置 `Field` 类”一节中每个字段的“空值”说明。

你可以编写代码来对特定的表单字段（基于其名称）或整个表单（考虑各种字段的组合）进行验证。更多关于这方面的信息请参见 [表单和字段验证](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/validation/)。

## 将表单输出为 HTML

`Form` 对象的第二个任务是将其本身渲染为 HTML。为此，`print` 对象:

```python
>>> f = ContactForm()
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" required></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" required></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" required></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself"></td></tr>
```

如果表单与数据绑定，HTML 输出将适当地包含该数据。例如，如果一个字段用 `<input type="text">` 表示，数据将在 `value` 属性中。如果一个字段用 `<input type="checkbox">` 表示，那么 HTML 将适当地包括 `checked`：

```python
>>> data = {'subject': 'hello',
...         'message': 'Hi there',
...         'sender': 'foo@example.com',
...         'cc_myself': True}
>>> f = ContactForm(data)
>>> print(f)
<tr><th><label for="id_subject">Subject:</label></th><td><input id="id_subject" type="text" name="subject" maxlength="100" value="hello" required></td></tr>
<tr><th><label for="id_message">Message:</label></th><td><input type="text" name="message" id="id_message" value="Hi there" required></td></tr>
<tr><th><label for="id_sender">Sender:</label></th><td><input type="email" name="sender" id="id_sender" value="foo@example.com" required></td></tr>
<tr><th><label for="id_cc_myself">Cc myself:</label></th><td><input type="checkbox" name="cc_myself" id="id_cc_myself" checked></td></tr>
```

这个默认输出是一个两列的 HTML 表单，每个字段都有一个 `<tr>`。请注意以下几点：

- 为灵活起见，输出 *不* 包括 `<table>` 和 `</table>` 标签，也不包括 `<form>` 和 `</form>` 标签或 `<input type="submit">` 标签。这是你的工作。
- 每个字段类型都有一个默认的 HTML 表示。`CharField` 用 `<input type="text">` 表示，`EmailField` 用 `<input type="email">` 表示。`BooleanField(null=False)` 由一个 `<input type="checkbox">`。请注意，这些只是合理的默认值；你可以通过使用部件来指定一个给定字段使用的 HTML，我们将在后面解释。
- 每个标签的 HTML `name` 直接从 `ContactForm` 类中的属性名中提取。
- 每个字段的文本标签——例如 `'Subject:'`、`'Message:'` 和 `'Cc myself:'`，是根据字段名将所有下划线转换为空格并将第一个字母大写而产生的。同样，请注意这些只是合理的默认值；你也可以手动指定标签。
- 每个文本标签都被一个 HTML `<label>` 标签包围，该标签通过其 `id` 指向相应的表格字段。而它的 `id` 则是通过在字段名前加上 `'id_'` 生成的。`id` 属性和 `<label>` 标签默认包含在输出中，以遵循最佳实践，但你可以改变这种行为。
- 输出使用 HTML5 语法，目标是 `<!DOCTYPE html>`。例如，它使用布尔属性，如 `checked` 而不是 XHTML 风格的 `checked='checked'`。

虽然当你 `print` 表单时，`<table>` 输出是默认的输出样式，但还有其他的输出样式。每种样式都可以作为表单对象的一个方法，每个渲染方法都返回一个字符串。

### `as_p()`

### `as_ul()`

### `as_table()`

### 样式化必填或错误的表单行

- `Form.error_css_class`

- `Form.required_css_class`

对必填或有错误的表单行和字段进行样式设计是很常见的。例如，你可能想用粗体显示必填的表格行，用红色突出显示错误。

[`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类有几个钩子，你可以用来给必填行或有错误的行添加 `class` 属性：设置 [`Form.error_css_class`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.error_css_class) 和／或 [`Form.required_css_class`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.required_css_class) 属性：

```python
from django import forms

class ContactForm(forms.Form):
    error_css_class = 'error'
    required_css_class = 'required'

    # ... and the rest of your fields here
```

一旦你这样做了，根据需要，行将被赋予 `"error"` 和／或 `"required"` 类。HTML 将看起来像：

```python
>>> f = ContactForm(data)
>>> print(f.as_table())
<tr class="required"><th><label class="required" for="id_subject">Subject:</label>    ...
<tr class="required"><th><label class="required" for="id_message">Message:</label>    ...
<tr class="required error"><th><label class="required" for="id_sender">Sender:</label>      ...
<tr><th><label for="id_cc_myself">Cc myself:<label> ...
>>> f['subject'].label_tag()
<label class="required" for="id_subject">Subject:</label>
>>> f['subject'].label_tag(attrs={'class': 'foo'})
<label for="id_subject" class="foo required">Subject:</label>
```

### 设置表单元素的 HTML `id` 属性和 `<label>` 标签。

- `Form.auto_id`

默认情况下，表单渲染方法包括：

- 表单元素的 HTML `id` 属性。
- 标签周围对应的 `<label>` 标签。HTML `<label>` 标签指定了哪个标签文本与哪个表单元素相关联。这个小小的改进使表单更加可用，也更容易被辅助设备访问。使用 `<label>` 标签总是一个好主意。

`id` 属性值是通过将 `id_` 预置到表单字段名后生成的。 如果你想改变 `id` 惯例或完全删除 HTML `id` 属性和 `<label>` 标签，这种行为是可以设置的。

使用 `Form` 构造函数的 `auto_id` 参数来控制 `id` 和标签行为。这个参数必须是 `True`、`False` 或一个字符串。

如果 `auto_id` 是 `False`，那么表单输出将不包含 `<label>` 标签或 `id` 属性：
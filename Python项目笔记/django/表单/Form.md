[toc]



## 使用表单

> Django提供了一系列的工具和库来帮助您构建表单来接收网站访客的输入，然后处理以及响应这些输入

### Django在表单中的角色

- 准备并重组数据，以便下一步的渲染
- 为数据创建HTML 表单
- 接收并处理客户端提交的表单及数据

Web应用中所说的'表单'，可能指的是HTML `<form>` ，或者是生成了它的Django [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) ，再或者是提交时返回的结构化数据，亦或是这些端到端作业的合集。

### Django的 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类

Django表单系统的核心组件是 [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类。它与Django模型描述对象的逻辑结构、行为以及它呈现给我们内容的形式的方式大致相同， [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类描述一张表单并决定它如何工作及呈现。

### 实例化、处理和渲染表单

1. 在视图中获取它（例如从数据库中取出）
2. 将它传递给模板上下文
3. 使用模板变量将它扩展为HTML标记

### 构建一张表单

假设您希望在您的网站上创建一张简易的表单，用来获取用户的名字。您需要在模板中使用类似代码：

```python
<form action="/your-name/" method="post">
    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
    <input type="submit" value="OK">
</form>
```

### 在Django 中构建一张表单

#### 1.Form类

forms.py[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#id1)

```python
from django import forms

class NameForm(forms.Form):
    your_name = forms.CharField(label='Your name', max_length=100)
```

字段的最大长度由 [`max_length`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.CharField.max_length) 来定义。它做了两件事情。首先它在HTML的 `<input>` 上增加了 `maxlength="100"` （这样浏览器会在第一时间阻止用户输入超过这个数量的字符串）。其次它还会在Django收到浏览器传过来的表单时，对数据长度进行验证（也就是服务器端验证）。

A [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) instance has an [`is_valid()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_valid) method, which runs validation routines for all its fields. When this method is called, if all fields contain valid data, it will:

- 返回 `True`
- 将表单的数据放到它的属性 [`cleaned_data`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.cleaned_data) 中。

这样整个表单在第一次渲染时，会显示如下：

```python
<label for="your_name">Your name: </label>
<input id="your_name" type="text" name="your_name" maxlength="100" required>
```

注意它 **没有** 包含 `<form>` 标签和提交按钮。我们必须自己在模板中提供。

#### 2.视图

发回Django网站的表单数据由视图来处理，一般和发布这个表单用的是同一个视图。这允许我们重用一些相同的逻辑。

为了处理表单，我们需要将它实例化到我们希望发布的URL的对应的视图中：

views.py[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#id2)

```python
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import NameForm

def get_name(request):
    # if this is a POST request we need to process the form data
    if request.method == 'POST':
        # create a form instance and populate it with data from the request:
        form = NameForm(request.POST)
        # check whether it's valid:
        if form.is_valid():
            # process the data in form.cleaned_data as required
            # ...
            # redirect to a new URL:
            return HttpResponseRedirect('/thanks/')

    # if a GET (or any other method) we'll create a blank form
    else:
        form = NameForm()

    return render(request, 'name.html', {'form': form})
```

如果我们访问这个视图用的是 `GET` 请求，它会创建一个空的表单实例并将其放置在模板上下文中进行渲染。这是我们在首次访问这个URL时能预料到会发生的情况。

如果表单提交用的是 `POST` 请求，那么该视图将再次创建一个表单实例并使用请求中的数据填充它： `form = NameForm(request.POST)` 这叫“绑定数据到表单” （现在它是一张 *绑定的* 表单）。

我们调用表单的 `is_valid()` 方法；如果不为 `True` ，我们带着表单返回到模板。这次表单不再为空（ *未绑定* ），所以HTML表单将用之前提交的数据进行填充，放到可以根据需要进行编辑和修正的位置。

如果 `is_valid()` 为 `True` ，我们就能在其 `cleaned_data` 属性中找到所有通过验证的表单数据。我们可以在发送一个HTTP重定向告诉浏览器下一步去向之前用这些数据更新数据库或者做其他处理。

#### 3.模板

我们没有必要在模板 `name.html` 中做过多的操作：

```python
<form action="/your-name/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```

所有的表单字段及其属性都将通过Django模板语言从 `{{ form }}` 中被解包成HTML标记。

表格和跨站请求伪造保护

Django自带一个简单易用的 [跨站请求伪造防护](https://docs.djangoproject.com/zh-hans/3.1/ref/csrf/) 。当通过 `POST` 方法提交一张启用了CSRF防护的表单时，您必须使用上例中这样的模板标签 [`csrf_token`](https://docs.djangoproject.com/zh-hans/3.1/ref/templates/builtins/#std:templatetag-csrf_token) 。但是，由于CSRF防护在模板中没有与表单直接绑定，因此这个标签在本页文档之后的示例中都将被忽略。

HTML5输入类型和浏览器验证

如果您的表单包含 [`URLField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.URLField) ， [`EmailField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.EmailField) 或者其他整数字段类型，Django将使用 `url` ， `email` 和 `number` HTML5输入类型。默认情况下，浏览器可能会在这些字段上应用他们自己的验证，这也许比Django的验证更加严格。如果您想禁用这个行为，请在 `form` 标签上设置 novalidate 属性，或者在字段上指定一个不同的控件，比如 [`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput) 

现在我们有了一个可以工作的web表单，它通过一张Django [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 描述，由一个视图来处理并渲染成一个HTML `<form>` 。

以上是您入门需要了解的所有内容，但是表单框架提供了更多垂手可得的内容。一旦您理解了上述过程的基础知识，您应该再了解下表单系统的其他功能，然后学习更多的底层机制。

## 详解Django [`Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 类

所有表单类都作为 [`django.forms.Form`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form) 或者 [`django.forms.ModelForm`](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/modelforms/#django.forms.ModelForm) 的子类来创建。您可以把 `ModelForm` 想象成 `Form` 的子类。实际上 `Form` 和 `ModelForm` 从（私有） `BaseForm` 类继承了通用功能，但是这个实现细节不怎么重要。

### 绑定的和未绑定的表单实例

- 未绑定的表单没有与其关联的数据。当渲染给用户的时候，它会是空的或者包含默认值。
- 绑定的表单拥有已提交的数据，因此可以用来判断数据是否合法。如果渲染了一张非法的绑定的表单，它将包含内联的错误信息，告知用户要纠正哪些数据。

表单的 [`is_bound`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.is_bound) 属性将告诉您一张表单是否具有绑定的数据。

### 字段详解

forms.py[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#id3)

```python
from django import forms

class ContactForm(forms.Form):
    subject = forms.CharField(max_length=100)
    message = forms.CharField(widget=forms.Textarea)
    sender = forms.EmailField()
    cc_myself = forms.BooleanField(required=False)
```

#### 控件[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#widgets)

每个表单字段都有一个相对应的 [控件类](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/) ，这个控件类又有对应的HTML表单控件，比如 `<input type="text">` 。

多数情况下，字段都有合适的默认控件。比如，默认情况下， [`CharField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.CharField) 有个 [`TextInput`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/widgets/#django.forms.TextInput) 控件，它会在HTML中生成一个 `<input type="text">` 。如果您想要的是 `<textarea> `` ，您要在定义表单字段的时候指定控件，就像我们对 ``message` 字段那样处理

#### 字段数据[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#field-data)

无论用表单提交了什么数据，一旦通过调用 `is_valid()` 验证成功（ `is_valid()` 返回 `True` ），已验证的表单数据将被放到 `form.cleaned_data` 字典中。这里的数据已经很好的为你转化为Python类型。

> 此时您依然能够直接从 `request.POST` 中访问到未验证的数据，但最好还是使用经验证的数据

在上面的联系表单示例中， `cc_myself` 会被转化成一个布尔值。同样的，字段 [`IntegerField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.IntegerField) 和 [`FloatField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.FloatField) 的值分别会被转化为Python的 `int` 和 `float` 类型。

下面例举了如何在视图中处理表单数据：

views.py[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#id4)

```python
from django.core.mail import send_mail

if form.is_valid():
    subject = form.cleaned_data['subject']
    message = form.cleaned_data['message']
    sender = form.cleaned_data['sender']
    cc_myself = form.cleaned_data['cc_myself']

    recipients = ['info@example.com']
    if cc_myself:
        recipients.append(sender)

    send_mail(subject, message, sender, recipients)
    return HttpResponseRedirect('/thanks/')
```

有些字段类型需要一些额外的处理。例如，使用表单上传文件就要以不同的方式处理（它们可以从 `request.FILES` 获取，而不是 `request.POST` 中）。有关如何使用表单处理文件上传的详细信息，请参见 [将上传的文件绑定到表单中](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#binding-uploaded-files) 。

## 使用表单模板

您只需将表单实例放到模板的上下文中即可。因此，如果您的表单在上下文中叫 `form` ，那么 `{{ form }}` 将渲染它相应的 `<label>` 和 `<input>` 元素。

### 表单渲染选项

额外表单模板标签

不要忘记，一张表单的输出 *不* 包含外层 `<form>` 标签以及 `submit` 控件。这些必须由你自己提供。

对于 `<label>` / `<input>` 对，还有其他输出选项：

- `{{ form.as_table }}` will render them as table cells wrapped in `<tr>` tags
- `{{ form.as_p }}` will render them wrapped in `<p>` tags
- `{{ form.as_ul }}` will render them wrapped in `<li>` tags

注意，您必须自己提供外层的 `<table>` 或 `<ul>` 元素。

下面是我们 `ContactForm` 实例用 `{{ form.as_p }}` 的输出：

```python
<p><label for="id_subject">Subject:</label>
    <input id="id_subject" type="text" name="subject" maxlength="100" required></p>
<p><label for="id_message">Message:</label>
    <textarea name="message" id="id_message" required></textarea></p>
<p><label for="id_sender">Sender:</label>
    <input type="email" name="sender" id="id_sender" required></p>
<p><label for="id_cc_myself">Cc myself:</label>
    <input type="checkbox" name="cc_myself" id="id_cc_myself"></p>
```

请注意，每个表单字段都有一个 `id_<field-name>` 这样的ID属性，它被附带的label标签引用。这对于确保表单可供屏幕阅读软件这样的辅助技术访问非常重要。您还可以 [自定义Label和ID的生成方式](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#ref-forms-api-configuring-label) 。

更多相关信息，请参阅 [将表单输出为 HTML](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#ref-forms-api-outputting-html) 。

### 手动渲染字段

我们没有必要非要让Django来解包表单字段；如果我们喜欢，可以手动来处理（比如，让我们对字段重新排序）。每个字段都可以用 `{{ form.name_of_field }}` 作为表单的一个属性，并被相应的渲染在Django模板中。例如：

```python
{{ form.non_field_errors }}
<div class="fieldWrapper">
    {{ form.subject.errors }}
    <label for="{{ form.subject.id_for_label }}">Email subject:</label>
    {{ form.subject }}
</div>
<div class="fieldWrapper">
    {{ form.message.errors }}
    <label for="{{ form.message.id_for_label }}">Your message:</label>
    {{ form.message }}
</div>
<div class="fieldWrapper">
    {{ form.sender.errors }}
    <label for="{{ form.sender.id_for_label }}">Your email address:</label>
    {{ form.sender }}
</div>
<div class="fieldWrapper">
    {{ form.cc_myself.errors }}
    <label for="{{ form.cc_myself.id_for_label }}">CC yourself?</label>
    {{ form.cc_myself }}
</div>
```

完整的 `<label>` 元素还可以使用 [`label_tag()`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.BoundField.label_tag) 来生成。例如：

```python
<div class="fieldWrapper">
    {{ form.subject.errors }}
    {{ form.subject.label_tag }}
    {{ form.subject }}
</div>
```

#### 渲染表单错误信息

这种灵活性的代价需要多做一点工作。到目前为止，我们不必担心如何显示表单的错误信息，因为它们已经帮我们处理好了。下面的例子中，我们需要自己处理每个字段的错误信息以及表单整体的所有错误信息。注意表单顶部的 `{{ form.non_field_errors }}` 以及模板中对每个字段查找错误信息。

使用 `{{ form.name_of_field.errors }}` 显示该字段的错误信息列表，它被渲染成无序列表。看起来如下：

```python
<ul class="errorlist">
    <li>Sender is required.</li>
</ul>
```

该列表有一个CSS class `errorlist` ，允许您自定义其样式。如果你想进一步自定义错误信息的显示，您可以通过遍历它们来实现：

```python
{% if form.subject.errors %}
    <ol>
    {% for error in form.subject.errors %}
        <li><strong>{{ error|escape }}</strong></li>
    {% endfor %}
    </ol>
{% endif %}
```

非字段验证错误信息（或者通过使用像 `form.as_p()` 这样的辅助方法渲染产生在表单顶部的隐藏错误信息）渲染后会额外带上一个class `nonfield` 以便与字段验证错误信息区分。例如， `{{ form.non_field_errors }}` 渲染后会像这样：

```python
<ul class="errorlist nonfield">
    <li>Generic validation error</li>
</ul>
```

### 遍历表单字段[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/forms/#looping-over-the-form-s-fields)

如果您要给每个表单字段使用相同的HTML，您可以用 `{% for %}` 依次循环遍历每个字段来减少重复代码

```python
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
        {% if field.help_text %}
        <p class="help">{{ field.help_text|safe }}</p>
        {% endif %}
    </div>
{% endfor %}
```

Useful attributes on `{{ field }}` include:

- `{{ field.label }}`

  字段的label，比如 `Email address`。

- `{{ field.label_tag }}`

  该字段的label封装在相应的HTML `<label>` 标签中。它包含表单的 [`label_suffix`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.Form.label_suffix) 。例如，默认的 `label_suffix` 是一个冒号：`<label for="id_email">Email address:</label> `

- `{{ field.id_for_label }}`

  用于该字段的 ID（像上面的例子中的 `id_email` ）。如果您要手动构建label，您可能要用这个来替换 `label_tag` 。例如，如果你有一些内嵌的JavaScript并且想要避免硬编码字段的ID，这也很有用。

- `{{ field.value }}`

  字段的值。例如 `someone@example.com` 。

- `{{ field.html_name }}`

  字段名称：用于其输入元素的name属性中。如果设置了表单前缀，它也会被加进去。

- `{{ field.help_text }}`

  与该字段关联的帮助文本。

- `{{ field.errors }}`

  输出一个 `<ul class="errorlist">` ，其中包含这个字段的所有验证错误信息。你可以使用 `{% for error in field.errors %}` 循环来自定义错误信息的显示。在这种情况下，循环中的每个对象是包含错误信息的字符串。

- `{{ field.is_hidden }}`

  如果是隐藏字段，这个属性为 `True` ，否则为 `False` 。它作为模板变量没多大作用，但可用于条件测试，例如：

```
{% if field.is_hidden %}
   {# Do something special #}
{% endif %}
```

- `{{ field.field }}`

  表单类中的 [`Field`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field) 实例由 [`BoundField`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/api/#django.forms.BoundField) 封装。您可以用它来访问 [`Field`](https://docs.djangoproject.com/zh-hans/3.1/ref/forms/fields/#django.forms.Field) 的属性，比如 `{{ char_field.field.max_length }}` 。

#### 遍历隐藏字段和可见字段

如果您在手动布置模板中的表单，而不是依靠Django的默认表单布局，您可能希望将 `<input type="hidden">` 字段与非隐藏字段区别开来。例如，因为隐藏字段不显示任何内容，将错误消息“放到”该字段旁边可能会导致用户混淆——所以这些字段的错误应该以不同的方式处理。

Django在表单上提供了两种方法，允许您独立地遍历隐藏和可见的字段： `hidden_fields()` 和 `visible_fields()` 。以下是使用这两种方法对之前示例的修改：

```python
{# Include the hidden fields #}
{% for hidden in form.hidden_fields %}
{{ hidden }}
{% endfor %}
{# Include the visible fields #}
{% for field in form.visible_fields %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

这个示例没有处理隐藏字段中的任何错误信息。通常，隐藏字段中的错误象征着表单被篡改，因为正常的表单交互不会去改变它们。但是，您也可以轻松地为这些表单错误插入一些错误信息显示出来。

### 可复用的表单模板

如果您的网站在多个位置对表单使用相同的渲染逻辑，您可以通过将表单的循环保存到独立的模板中，然后在其他模板中使用 [`include`](https://docs.djangoproject.com/zh-hans/3.1/ref/templates/builtins/#std:templatetag-include) 标签来减少代码重复

```python
# In your form template:
{% include "form_snippet.html" %}

# In form_snippet.html:
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

如果传递给模板的表单对象在上下文中具有不同的名称，您可以使用 [`include`](https://docs.djangoproject.com/zh-hans/3.1/ref/templates/builtins/#std:templatetag-include) 标签的 `with` 属性来给它取别名。

```python
{% include "form_snippet.html" with form=comment_form %}
```
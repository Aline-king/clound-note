# 组合的妙用

## 提示块嵌套折叠块

{% hint style="info" %}
## 这是提示快标题

<details>

<summary>折叠块标题1</summary>

内容

</details>

<details>

<summary>折叠块标题2</summary>

内容

</details>
{% endhint %}

```
{% raw %}
{% hint style="info" %}
**提示快标题**
# 这是主标题
<details>
<summary>这是折叠块标题提</summary>
</details>
{% endhint %}
{% endraw %}
```

## Expandable blocks

<details>

<summary>Expandable block</summary>



</details>

```
<details>

<summary>1111</summary>

</details>
```

```
<details>

<summary>Expandable block</summary>



</details>
```

## 选项卡嵌套折叠块

{% tabs %}
{% tab title="Windows" %}
<details>

<summary>这是折叠块标题提</summary>



</details>
{% endtab %}

{% tab title="OSX" %}
Here are the instructions for macOS
{% endtab %}

{% tab title="Linux" %}
Here are the instructions for Linux
{% endtab %}
{% endtabs %}

```
{% raw %}
{% tabs %}

{% tab title="Windows" %} 
<details>
<summary>这是折叠块标题提</summary>
</details>
{% endtab %}

{% tab title="OSX" %} Here are the instructions for macOS {% endtab %}

{% tab title="Linux" %} Here are the instructions for Linux {% endtab %}

{% endtabs %}
{% endraw %}
```

## 选项卡嵌套提示快与折叠块的

{% tabs %}
{% tab title="Windows" %}
{% hint style="info" %}
**提示快标题**

## 这是主标题

<details>

<summary>这是折叠块标题提</summary>



</details>
{% endhint %}
{% endtab %}

{% tab title="OSX" %}
Here are the instructions for macOS
{% endtab %}

{% tab title="Linux" %}
Here are the instructions for Linux
{% endtab %}
{% endtabs %}

```
{% raw %}
{% tabs %}

{% tab title="Windows" %} 
{% hint style="info" %}
**提示快标题**
# 这是主标题
<details>
<summary>这是折叠块标题提</summary>
</details>
{% endhint %}
{% endtab %}

{% tab title="OSX" %} Here are the instructions for macOS {% endtab %}

{% tab title="Linux" %} Here are the instructions for Linux {% endtab %}

{% endtabs %}
{% endraw %}
```

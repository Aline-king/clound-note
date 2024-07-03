# 组合的妙用

## 提示块嵌套折叠块

{% hint style="info" %}
## 这是提示块标题

<details>

<summary>折叠块标题1</summary>



</details>

<details>

<summary>折叠块标题2</summary>

内容

</details>
{% endhint %}

```
{% raw %}
{% hint style="info" %}
**提示块标题**
# 这是主标题
<details>
<summary>这是折叠块标题提</summary>
</details>
{% endhint %}
{% endraw %}
```



```
{% raw %}
{% hint style="info" %}
**提示块标题**
# 这是主标题
<details>
<summary>2323</summary>
</details>
{% endhint %}
{% endraw %}
```

{% hint style="info" %}
**提示块标题**

## 这是主标题

<details>

<summary>2323</summary>

小标题

</details>
{% endhint %}

## Expandable blocks

<details>

<summary>1111</summary>

***

dwadwadwa

dwad



```
{% raw %}
{% tabs %}

{% tab title="Windows" %} 1111 {% endtab %}
{% tab title="Windows" %} 112211 {% endtab %}
{% endtabs %}
{% endraw %}
```



<img src="../.gitbook/assets/镜像服务02.png" alt="" data-size="original">dawdawddddddddddddddddddddddddddwd











</details>

```jsx
<details>

<summary>Expandable block</summary>


</details>
```

<details>

<summary>Expandable block</summary>

1111112211

\-2222

</details>

## 提示块内嵌选项卡

{% hint style="info" %}
**提示块标题**

## 这是主标题

{% tabs %}
{% tab title="Windows" %}

{% endtab %}

{% tab title="Windows" %}
2222
{% endtab %}
{% endtabs %}
{% endhint %}

```
{% raw %}
{% hint style="info" %}
**提示块标题**
# 这是主标题
{% tabs %}

{% tab title="Windows" %} 1111 {% endtab %}
{% tab title="Windows" %} 2222 {% endtab %}
{% endtabs %}
{% endhint %}
{% endraw %}
```

## 提示快内嵌选项卡和折叠块

{% hint style="info" %}
**提示块标题**

## 这是主标题

{% tabs %}
{% tab title="undefined" %}
<details>

<summary>32323</summary>



</details>
{% endtab %}

{% tab title="Windows" %}
1111
{% endtab %}

{% tab title="Windows" %}
2222
{% endtab %}
{% endtabs %}
{% endhint %}

```
{% raw %}
{% hint style="info" %}
**提示块标题**
# 这是主标题
{% tabs %}
<details>
<summary>32323</summary>
</details>
{% tab title="Windows" %} 1111 {% endtab %}
{% tab title="Windows" %} 2222 {% endtab %}
{% endtabs %}
{% endhint %}
{% endraw %}
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

## 选项卡嵌套选项卡

```
{% raw %}
{% tabs %}
{% tab title="Windows" %} 
{% hint style="info" %}
**提示快标题**
{% tabs %}
{% tab title="Windows" %} 
{% tab title="OSX" %} Here are the instructions for macOS {% endtab %}
{% endtabs %}

{% endhint %}
{% endtab %}

{% tab title="OSX" %} Here are the instructions for macOS {% endtab %}

{% tab title="Linux" %} Here are the instructions for Linux {% endtab %}

{% endtabs %}
{% endraw %}
```

{% tabs %}
{% tab title="Windows" %}
{% hint style="info" %}
**提示快标题**

{% tabs %}
{% tab title="Windows" %}

{% endtab %}

{% tab title="Linux" %}
Here are the instructions for macOS
{% endtab %}
{% endtabs %}
{% endhint %}
{% endtab %}

{% tab title="Linux" %}
Here are the instructions for Linux
{% endtab %}
{% endtabs %}


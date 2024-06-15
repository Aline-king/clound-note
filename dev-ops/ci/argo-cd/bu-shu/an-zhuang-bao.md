# 安装包

{% hint style="info" %}
安装包，任选其一

manifests&#x20;

helm chart
{% endhint %}

## 安装包区别

<table><thead><tr><th width="187">资源对象</th><th width="86" data-type="checkbox">install</th><th width="169" data-type="checkbox">namespace-install</th><th width="109" data-type="checkbox">ha/install</th><th data-type="checkbox">ha/namespace-install</th></tr></thead><tbody><tr><td>crd</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>sa</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>role</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>clusterrole</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>clusterrolebinding</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>configmap</td><td>false</td><td>false</td><td>false</td><td>false</td></tr><tr><td>secret</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>service</td><td>false</td><td>false</td><td>false</td><td>false</td></tr><tr><td>deployment</td><td>true</td><td>true</td><td>true</td><td>true</td></tr><tr><td>statefulset</td><td>false</td><td>false</td><td>false</td><td>false</td></tr><tr><td>networkPolicy</td><td>false</td><td>false</td><td>false</td><td>false</td></tr><tr><td>redis</td><td>true</td><td>true</td><td>false</td><td>false</td></tr><tr><td>ha</td><td>false</td><td>false</td><td>false</td><td>false</td></tr><tr><td>rolebinding</td><td>true</td><td>true</td><td>true</td><td>true</td></tr></tbody></table>


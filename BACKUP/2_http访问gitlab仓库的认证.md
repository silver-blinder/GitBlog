# [http访问gitlab仓库的认证](https://github.com/silver-blinder/GitBlog/issues/2)

<p>git config中的这几项作用：</p>

配置项 | 是否参与 git clone 过程 | 作用
-- | -- | --
credential.helper | ✅ | 提供认证凭据
http.proxy / https.proxy | ✅ | 决定是否通过代理发起请求
http.sslverify | ✅ | 决定是否验证 GitLab 的 SSL 证书
http.version | ✅ | 决定通信协议版本
user.name / user.email | ❌（不参与认证） | 仅用于 Git 提交记录，不参与 clone 认证过程、


<p>其中：
credential.helper（表示 Git 使用 macOS 自带的 Keychain（钥匙串） 来保存你的 Git 凭据）</p>
<p>假设你执行：</p>
<pre><code class="language-bash">git clone &lt;https://gitlab.example.com/project.git&gt;

</code></pre>
<ol>
<li>Git 检查本地有没有保存凭据（Keychain）；</li>
<li>如果有，就自动使用；</li>
<li>没有的话，会弹出让你输入用户名和密码/Token；</li>
<li>成功后，会将凭据保存到 Keychain（凭借 <code>credential.helper=osxkeychain</code>）；</li>
<li>下次就会自动使用，无需再次输入。</li>
</ol>
<!-- notionvc: 26714734-7f8b-4660-98c0-38af3da030b5 -->
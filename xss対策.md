# XSS (クロスサイトスクリプティング) 対策まとめ

XSS (Cross-Site Scripting) は、Webサイトにユーザーが入力した悪意のJavaScriptを埋め込み、別のユーザーに実行させる攻撃手段です。

## 【対策基本方針】

### 1. HTMLエスケープ (Escape)

悪意のスクリプトをHTMLとして解釈させないため、入力の出力時に以下の文字を置換します:

| 文字  | 置換形式     |
| --- | -------- |
| `<` | `&lt;`   |
| `>` | `&gt;`   |
| `"` | `&quot;` |
| `'` | `&#x27;` |
| `&` | `&amp;`  |

**Python**:

```python
import html

user_input = "<script>alert('XSS!')</script>"
safe_output = html.escape(user_input)

print(safe_output)
# 結果: &lt;script&gt;alert(&#x27;XSS!&#x27;)&lt;/script&gt;
```

**PHP**:

```php
htmlspecialchars($input, ENT_QUOTES, 'UTF-8');
```

**Django (Jinja2)**:

```jinja2
{{ user_input | e }}
```

---

### 2. JavaScript DOMでの安全な出力

* `innerHTML` は使わない → 正確な表示でもXSSを起こすもとに
* 代わりに `textContent` または `innerText`

```javascript
// 安全
element.textContent = userInput;

// 危険
element.innerHTML = userInput; // 攻撃者のスクリプトが実行される
```

---

### 3. Content Security Policy (CSP)

CSPを利用すると、外部スクリプトや不安全なスタイルのコード実行を防ぐことができます:

```http
Content-Security-Policy: default-src 'self'; script-src 'self'; object-src 'none';
```

---

### 4. 入力値の検証 (バリデーション)

* 文字数制限
* 許可された文字のみを可接受
* HTMLタグの補正はしない

```python
import re
if not re.match(r'^[a-zA-Z0-9_]{1,20}$', user_input):
    raise ValueError("Invalid input")
```

---

## 【よくある脆弱性の例】

1. 検索ページでの URL パラメータ:

```
https://example.com/search?q=<script>alert(1)</script>
```

2. コメント欄:

```html
<p>{{ comment }}</p>  ← エスケープしないと危険
```

---

## 【まとめ】

* **XSSは、表示段階で防ぐこと**
* コード内に「出力は繰り返すな」の思想を持つ
* 読み込みスクリプトも、依存CDNも注意
* CSP + HTMLエスケープ + DOMセキュア の3本柱

---



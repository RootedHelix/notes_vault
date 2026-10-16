✅ Yes — `application/x-www-form-urlencoded` is the **MIME type** that tells the server:

> “This request body is a **URL-encoded string**.”

---

### 🔍 What is a URL-encoded string?

It’s the format used when form data is encoded like a URL query string:

```
username=John+Doe&email=john%40example.com
```

* Key-value pairs separated by `&`
* Keys and values joined by `=`
* Spaces encoded as `+`
* Special characters percent-encoded (e.g., `@` → `%40`)

---

### 📤 When is it used?

It's the **default** when:

* You submit a form with `method="POST"` or `method="GET"`
* And you do **not** set `enctype` or set it to:

  ```html
  enctype="application/x-www-form-urlencoded"
  ```

---

### ✅ Example Form:

```html
<form action="/submit" method="POST">
  <input name="username" value="Alice">
  <input name="email" value="alice@example.com">
  <button type="submit">Send</button>
</form>
```

Sends:

```
username=Alice&email=alice%40example.com
```

With HTTP header:

```
Content-Type: application/x-www-form-urlencoded
```

---

Would you like to see how this compares with `multipart/form-data` or how it's parsed server-side?

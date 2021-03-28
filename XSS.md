# XSS

## About

XSS (Cross-site scripting) attacks enable attackers to inject client-side scripts into web pages viewed by other users.

The main effects of this vulnerability are the possibility of:

- execution of any actions in the context of the logged in user
- reading any data in the context of the logged in user

## Attack scenario

1. The attacker locates the XSS vulnerability on a website used by the victim, e.g. a bank's website
2. The victim is currently logged on to this page
3. The attacker sends the victim a crafted URL
4. The victim clicks on the URL
5. On the victim's bank website, JavaScript code starts executing to intercept the user's data or execute a transfer on his behalf to the attacker's account

It is worth noting that performing operations on behalf of the victim may be invisible to the victim, as it may take place in the background using the bank's API or the attacker may perform it in some time with the data needed for authentication, tokens, cookies, etc.

## XSS types

1. Reflected XSS

   It is one where HTML/JS code contained in any parameter (eg GET, POST or cookie) is displayed in response.

   A page with a text input to search for something that puts the parameter `?search=foo` in the URL ending when querying the API. After entering any phrase, in case of not finding it, we get a return message placed in HTML ex.

   ```html
   <div>No result found for <b>foo</b></div>
   ```

   We can try put in URL `?search=<script>alert('XSS')</script>`.

2. DOM XSS

   This is when its execution is possible due to the use of dangerous functions in JavaScript, such as `eval` or` innerHtml`. Below "Live example" shows DOM XSS attack based on `innerHtml` function.

3. Stored XSS

   It is one where the malicious code gets written on the server side. For example, we may send comment with malicious code to a blog post that is uploaded to the server. His task is, for example, to wait for the administrator's moderation to steal his session data, etc.

## Injection methods

1.  In the tag content

    `onerror=alert('XSS')` into

    ```html
    <img src onerror=alert('XSS') />
    ```

2.  In the content of the attribute

    `" onmouseover=alert('XSS')` into

    ```html
    <div class="" onmouseover=alert('XSS')"></div>
    ```

3.  In the content of the attribute without the quotes

    `x onclick=alert('XSS')` into

    ```html
    <div class=x onclick=alert('XSS')></div>
    ```

4.  In the `href` attribute

    `javascript:alert('XSS')` into

    ```html
    <a href="javascript:alert('XSS')"></a>
    ```

5.  In a string inside JS code

    `";alert('XSS')//` into

    ```js
    <script>let username="";alert('XSS')//";</script>
    ```

6.  In the attribute with the JS event

    `&#39;);alert('XSS')//` where `&#39` is apostrophe, into

    ```js
    <div onclick="change('&#39;);alert('XSS')//')">John</div>
    ```

7.  In the href attribute inside the JS protocol

    `%27);alert(1)//` where `%27` is apostrophe, into

    ```html
    <a href="javascript:change('%27);alert(1)//')">click</a>
    ```

## Live example

https://codesandbox.io/s/xss-vulnerability-iedok

## Defense methods

1. Data encoding using built-in functions found in many programming languages.
2. Using template systems with automatic encoding. Most of the popular framework that use such systems protect us from XSS injection (Django, Templates, Vue, React etc.).
3. Do not use functions like `eval` or `Function` by passing untrusted user data to them.
4. Do not use functions and properties that assign HTML code directly to the DOM tree elements, eg `innerHTML`, `outerHTML`, `insertAdjacentHTML`, `document.write`. Instead, you can use functions that assign text directly to these elements, such as `textContent` or `innerText`.
5. Be careful when you redirect the user to a URL that is under his control. Risk of injection `location = 'javascript('XSS')'`.
6. Filter HTML using libraries such as `DOMPurify`.
7. Be careful about uploading `.html` or `.svg` files. You can create a separate domain from which the uploaded files will be served.
8. Use the `Content-Security-Policy` mechanism.
9. Take look at anti-XSS filters built into most popular browsers.

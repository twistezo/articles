# JavaScript Security Vulnerabilities

## target="\_blank"

### Attack scenario

1. Suppose the victim uses facebook which is known for opening links via `target="_blank"`
2. Create a fake viral page
3. Create a phishing website that looks like facebook sign in
4. Put below code on viral page
   ```
   window.opener.location = 'https://phishing-website/facebook.com';
   ```
5. The victim clicks on link on FB to the viral page
6. The viral page redirects the FB tab to phishing website asking user to sign in again

So we can change the "parent" tab from infected "children" page by `window` object from Web API.

Note that modern browsers makes `window.opener` function in child tab as `null` to prevent this behaviour.

### Example code

```html
<a href="https://github.com" target="_blank">Go to GitHub - infected link</a>
```

```js
const link = document.getElementsByTagName('a')
if (link)
  link[0].onclick = () => {
    if (window) window.opener.location = 'https://stackoverflow.com'
  }
```

Above is the infected link which originally opens new tab with GitHub page but meanwhile it changes our "parent" page to Stackoverflow site.

### Live example

https://codesandbox.io/s/targetblank-vulnerability-r8uij

### Solutions

- HTML - add `rel="noopener noreferrer"` to `<a>` tag.
- JS - add `window.opener = null` after `window.open()`.

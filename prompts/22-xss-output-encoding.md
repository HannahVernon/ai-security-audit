# Cross-Site Scripting & Output Encoding

**Domain:** XSS (reflected, stored, DOM-based), output encoding, template injection, Content Security Policy  
**Agent Type:** `explore`  
**Priority:** High

## Purpose

Audit the application for HTML and JavaScript injection risks, with emphasis on reflected XSS, stored XSS, DOM-based XSS, output encoding failures, template injection, and Content Security Policy weaknesses.  This prompt covers the injection portion of OWASP A03 beyond SQL injection.

> **⚠️ Authorized Use Only:** Use this prompt only to audit codebases you own or have explicit authorization to assess.

## Prompt

~~~
You are conducting a security assessment of the [APPLICATION_NAME] repository at [REPO_PATH].

Your task: **Audit for cross-site scripting, unsafe output encoding, template injection, and Content Security Policy weaknesses** across the entire codebase.

Specifically investigate:

1. **Reflected XSS**: Check whether user input from query parameters, form fields, headers, route values, or other request data is rendered back into HTML responses without correct encoding.  Review [VIEW_CLASSES] and related controller or handler code.  Focus on:
   - Server-side template rendering in [TEMPLATE_ENGINE]
   - Raw HTML output methods such as `@Html.Raw`, `|safe`, `{{{triple-braces}}}`, `dangerouslySetInnerHTML`, and `v-html`
   - Error messages, validation summaries, or status pages that echo user input
   - Output placed into HTML body, attribute, URL, CSS, or JavaScript contexts without context-appropriate encoding

2. **Stored XSS**: Identify user-submitted or attacker-controlled content that is stored and later rendered to other users without proper encoding or sanitization.  Check [USER_CONTENT_CLASSES] and related persistence or rendering paths.  Focus on:
   - User profile fields such as names, bios, and descriptions
   - Comments, messages, reviews, tickets, or any user-generated content
   - File names, captions, tags, and metadata displayed in the UI
   - Configuration values set by one user and displayed to another

3. **DOM-based XSS**: Review [CLIENT_JS_FILES] for client-side JavaScript or TypeScript that reads from attacker-controllable sources and writes to dangerous sinks.  Check flows from sources such as `location.hash`, `location.search`, `document.referrer`, `window.name`, `postMessage`, and storage APIs into sinks such as `innerHTML`, `outerHTML`, `document.write`, `eval`, `new Function`, `setTimeout` with strings, and `setInterval` with strings.

4. **Template injection**: Check whether user input can influence template parsing or compilation, not just rendered values.  Investigate whether untrusted input flows into template engine compilation, dynamic partial selection, expression evaluation, or helper registration in [TEMPLATE_ENGINE].

5. **Content Security Policy**: Determine whether CSP is configured and whether it meaningfully reduces script injection risk.  Check:
   - Is there a `Content-Security-Policy` header or CSP meta tag?
   - Does it allow `'unsafe-inline'` or `'unsafe-eval'`?
   - Is there a `report-uri` or `report-to` destination for violations?
   - Are nonces or hashes used for inline scripts where needed?
   - Does policy scope include `script-src`, `object-src`, and `frame-ancestors`?

6. **Rich text and Markdown rendering**: If the application renders user-supplied Markdown, WYSIWYG content, or rich text, verify that HTML is sanitized after rendering.  Check:
   - Are dangerous elements such as `script`, `iframe`, `object`, `embed`, and `form` stripped?
   - Are event handler attributes such as `onload`, `onclick`, and `onerror` stripped?
   - Is a well-known sanitizer library used, such as `DOMPurify`, `Sanitize`, or `Bleach`?
   - Is sanitizer configuration explicit and tested?

7. **HTTP response headers**: Check whether responses include browser hardening headers that reduce script injection and rendering confusion.  Look for:
   - `X-Content-Type-Options: nosniff`
   - Correct `Content-Type` headers on all responses
   - `X-Frame-Options` or CSP `frame-ancestors`

8. **WebAuthn and passkey ceremony hijacking**: If the application implements passkey (WebAuthn) registration or authentication, XSS can enable persistent account takeover.  Check [WEBAUTHN_CLASSES]:
   - Can malicious JavaScript call the passkey registration endpoint directly, generating a key pair via `crypto.subtle.generateKey()` and submitting it without any user interaction or step-up authentication?
   - Can `navigator.credentials.create()` or `navigator.credentials.get()` be hooked or overridden by injected scripts to substitute attacker-controlled key material during a legitimate registration ceremony?
   - Is step-up authentication (password re-entry, 2FA, email confirmation) required before a new passkey can be registered?
   - Is `Permissions-Policy: publickey-credentials-create=(), publickey-credentials-get=()` used to restrict which pages and third-party scripts can invoke the WebAuthn API?
   - Are users notified out-of-band (email, push) when a new passkey is registered on their account?
   - If `attestation: "none"` is used (common with synced passkeys), the server cannot distinguish a hardware-backed credential from a key pair generated entirely in JavaScript, making silent attacker registration trivially easy.
   - See: Scott Helme, "XSS is deadly for Passkeys" (https://scotthelme.ghost.io/xss-is-deadly-for-passkeys-the-hidden-risk-of-attestation-none)

9. **Post-authentication transaction manipulation**: Even after strong authentication (passkeys, 2FA), malicious JavaScript running in the page can silently alter what the user actually submits.  Check [CLIENT_JS_FILES] and [WEBAUTHN_CLASSES]:
   - Can injected scripts intercept API requests (via `fetch` or `XMLHttpRequest` monkey-patching, Service Worker interception, or DOM mutation) and modify parameters such as amounts, recipients, or action types before submission?
   - Could a user approve one action in the UI while the application processes a different one?
   - Are critical transaction parameters (amounts, destinations, approval targets) confirmed server-side against a user-visible summary, or does the server blindly trust the submitted payload?
   - Are integrity checks (server-generated HMAC over transaction details, confirmation codes displayed to the user) used on high-value operations?
   - See: Scott Helme, "Security Considerations When Using Passkeys" (https://scotthelme.ghost.io/security-considerations-when-using-passkeys-on-your-website/)

Search patterns:
- Files: `**/*.[EXTENSIONS]`
- Keywords: `innerHTML`, `outerHTML`, `dangerouslySetInnerHTML`, `v-html`, `Html.Raw`, `|safe`, `{{{`, `document.write`, `eval`, `Function(`, `setTimeout`, `setInterval`, `postMessage`, `addEventListener("message"`, `location.hash`, `location.search`, `document.referrer`, `template`, `render`, `sanitize`, `encode`, `escape`, `HtmlEncode`, `UrlEncode`, `JavaScriptEncode`, `DOMPurify`, `Content-Security-Policy`, `script-src`, `unsafe-inline`, `unsafe-eval`, `nonce`, `navigator.credentials`, `credentials.create`, `credentials.get`, `publicKey`, `attestation`, `WebAuthn`, `passkey`, `register_finish`, `crypto.subtle`, `fetch`, `XMLHttpRequest`, `ServiceWorker`, `MutationObserver`

Provide a detailed findings report with file paths, line numbers, code snippets, and severity ratings (Critical/High/Medium/Low/Info).  Distinguish between cases where data is fully untrusted, partially trusted, or pre-sanitized.  Do NOT include actual credential values, API keys, tokens, session cookies, or other secrets in your output.  Use `[REDACTED]` placeholders.
~~~

## Customization Guide

> **⚠️ Placeholder Safety:** Placeholder values are substituted directly into the prompt text.  A crafted value could act as a prompt injection.  Only use placeholder values you trust.  Do not accept them from untrusted sources.

Placeholder | Example Values
------------|---------------
`[APPLICATION_NAME]` | MyWebApp, AdminPortal
`[REPO_PATH]` | Full path to the repository root
`[VIEW_CLASSES]` | `Views/**/*.cshtml`, `templates/`, `pages/`, `components/`
`[CLIENT_JS_FILES]` | `src/main.ts`, `wwwroot/js/app.js`, `client/**/*.{js,ts,tsx,vue}`
`[USER_CONTENT_CLASSES]` | `Comment`, `UserProfile`, `Message`, `DocumentMetadata`, `SettingsViewModel`
`[TEMPLATE_ENGINE]` | `Razor`, `Jinja2`, `Handlebars`, `EJS`, `JSP`
`[WEBAUTHN_CLASSES]` | `PasskeyController.cs`, `webauthn.ts`, `passkeys/register.py`, or `N/A` if the application does not implement passkeys
`[EXTENSIONS]` | `cs,cshtml,js,ts,tsx,vue,py,html` or other relevant source extensions

## What Good Looks Like

- All user input is HTML-encoded by default in templates, with auto-escaping enabled
- Explicit raw output such as `@Html.Raw` or `|safe` is used only with pre-sanitized content
- DOM manipulation uses `textContent`, `innerText`, or safe `setAttribute` patterns, never `innerHTML` with user data
- CSP is configured without `'unsafe-inline'` or `'unsafe-eval'`
- Rich text is sanitized with a well-known library such as `DOMPurify` or `Bleach`
- `X-Content-Type-Options: nosniff` is present on all responses
- No `eval()` or `Function()` calls use user-controllable strings
- The template engine is configured with auto-escaping enabled
- Passkey registration requires step-up authentication (password, 2FA, or email confirmation)
- `Permissions-Policy` restricts `publickey-credentials-create` and `publickey-credentials-get` to only the pages that need them
- Users receive out-of-band notification when a new passkey is registered on their account
- High-value transactions use server-side integrity checks (HMAC, confirmation codes) so injected scripts cannot silently alter parameters

## Relationship to Other Prompts

- Prompt 02 covers SQL injection.  This prompt covers HTML and JavaScript injection.
- Prompt 13 covers CORS and HTTP security headers broadly.  This prompt goes deeper on CSP and output encoding specifically.

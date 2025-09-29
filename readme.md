# Unintended Form Submission Issue During IME Composition

This repository demonstrates a common issue where a form is submitted unintentionally when a user presses the `Enter` key to confirm a character conversion while using an Input Method Editor (IME), such as those for Japanese, Chinese, or Korean. It also provides the solution.

## Problem Summary

In many web forms, pressing the `Enter` key is a convenient shortcut to submit the form. However, for users of IMEs, the `Enter` key also serves the critical function of finalizing a character conversion (e.g., converting phonetic input like "nihongo" to 「日本語」).

This creates a conflict. When a user presses `Enter` to confirm their text conversion, the browser may also interpret it as a form submission event. This leads to a premature submission with incomplete data, creating a frustrating user experience.

This is especially problematic in UIs like chat applications or search bars, where `Enter`-to-submit is a common pattern.

**Problematic Demo:** `index.html`

Open this file, enable a Japanese IME, type "nihongo", and press `Enter` to confirm the conversion. You will see that the form submits immediately, even though you were still in the middle of typing.

```html
<!-- Problematic code in index.html -->
<script>
  // ...
  input.addEventListener('keydown', (event) => {
    if (event.key === 'Enter') {
      // Because there is no check for IME composition status,
      // the form submits even when Enter is pressed for conversion.
      event.preventDefault();
      form.requestSubmit();
    }
  });
  // ...
</script>
```

## The Solution

This issue can be solved by checking the `isComposing` property of the `KeyboardEvent` object.

- `event.isComposing` returns `true` if the event occurs during an active IME composition session.
- `event.isComposing` returns `false` otherwise, meaning the input is not in a pre-composition state.

By adding a condition to check that `event.isComposing` is `false` when the `Enter` key is pressed, we can cleanly distinguish between an IME confirmation and a form submission action.

**Fixed Demo:** `fixed.html`

This file includes the logic to check for `isComposing`. If you follow the same steps, pressing `Enter` will only confirm the text conversion. The form will not submit. Pressing `Enter` a second time will then submit the form as expected.

```html
<!-- Fixed code in fixed.html -->
<script>
  // ...
  input.addEventListener('keydown', (event) => {
    if (event.key === 'Enter') {
      event.preventDefault();

      // Only submit the form if an IME is NOT actively composing.
      if (!event.isComposing) {
        form.requestSubmit();
      }
    }
  });
  // ...
</script>
```

## Setup and Testing

1.  Open `index.html` and `fixed.html` from this repository in your browser.
2.  Enable a Japanese input method (e.g., Google Japanese Input, Microsoft IME) in your OS.
3.  Type in the input field on each page and observe the difference in behavior when you press the `Enter` key to confirm your text.